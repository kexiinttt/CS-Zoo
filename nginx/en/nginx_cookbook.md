# Nginx

Nginx is a high-performance web server, reverse proxy server, and load balancer. It is commonly used to improve website performance and serve static files.

When we enter `http://www.domain.com` in a browser, DNS resolves the domain to an IP address and port. The request is therefore effectively sent to `http://192.0.0.1:80`. When many users visit the site, port `80` on `192.0.0.1` can become extremely busy. Distributing these requests across multiple servers or different ports on one server would help. Nginx is designed to solve these problems.

## What Are Forward and Reverse Proxies?

### Forward Proxy

In a forward proxy, the proxy represents the client. The client asks the proxy to contact the server on its behalf. The client sends a request to the forward proxy, which forwards it to the target server; the response travels back through the proxy in the opposite direction.

![Forward Proxy](../pic/forward_proxy.png)

A forward proxy has these characteristics:
1. Bypassing network restrictions: the client cannot access the server directly and needs the proxy.
2. Network security: the proxy can hide the client's real IP address.
3. Access control: configuration on the proxy can control whether a client is allowed to request the server.
4. Resource caching: if the proxy has cached the requested content, it can return it directly without sending another network request.

### Reverse Proxy

In a reverse proxy, the proxy represents the server. The server asks the proxy to process and distribute requests. The client sends a request to the reverse proxy, which forwards it to a backend server according to configured rules. The backend response returns to the proxy, which forwards it to the client.

![Reverse Proxy](../pic/reverse_proxy.png)

A reverse proxy has these characteristics:
1. Load balancing: after receiving a request, the reverse proxy dynamically assigns it to a server according to the current load.
2. Resource caching: it can cache static resources and reduce backend load, as a CDN (Content Delivery Network) does.
3. Network security: it hides the backend server's real IP address and reduces attacks. It can also provide SSL encryption, access control, and other features.

> [!NOTE]
> CDN: when a user visits a website, the website server must transfer data over the Internet. If website resources are stored on a CDN server close to the user, data-transfer time can be reduced.

## Nginx Reverse Proxy

**`conf/nginx.conf`**
```conf
http {
    # The application runs on port 8080 of 127.0.0.1 (localhost).
    upstream local_server {
        server 127.0.0.1:8080;
    }

    server {
        # The Nginx server listens on port 80 for external requests.
        listen       80;

        # Access the site with www.domain.com.
        server_name  www.domain.com;

        # Reverse-proxy path.
        location / {
            proxy_pass http://local_server;
        }
    }
}
```

With this configuration, a request to `http://www.domain.com` reaches `192.0.0.1:80`, and Nginx reverse-proxies it to `localhost:8080`.

## Nginx Load Balancing

**`conf/nginx.conf`**
```conf
http {
    # The application runs on these servers and ports.
    upstream load_balance_server {
        server 192.0.0.1:8080;
        server 192.0.0.2:8080;
        server 192.0.0.3:8080;
    }

    server {
        # The Nginx server listens on port 80 for external requests.
        listen       80;

        # Access the site with www.domain.com.
        server_name  www.domain.com;

        # Load-balance across available servers.
        location / {
            proxy_pass http://load_balance_server;
        }
    }
}
```

With this configuration, a request to `http://www.domain.com` reaches `192.0.0.1:80`. Nginx reverse-proxies it to `load_balance_server`, which contains multiple possible backend servers, so Nginx load-balances across that group.

### Load-Balancing Strategies

#### Round Robin

Requests are assigned to servers in sequence.
```conf
upstream load_balance_server {
    server 192.0.0.1:8080;
    server 192.0.0.2:8080;
    server 192.0.0.3:8080;
}
```

#### Weighted

A server with a higher weight receives requests more frequently. A server with better performance should generally receive a larger weight.
```conf
upstream load_balance_server {
    server 192.0.0.1:8080 weight=1;
    server 192.0.0.2:8080 weight=2;
    server 192.0.0.3:8080 weight=3;
}
```

#### IP Hash

The same IP address is assigned to the same server, which can preserve session affinity for one user.
```conf
upstream load_balance_server {
    ip_hash;
    server 192.0.0.1:8080;
    server 192.0.0.2:8080;
    server 192.0.0.3:8080;
}
```

#### URL Hash

The same URL is assigned to the same server. This can be useful when each server provides a specific function or stores specific data.
```conf
upstream load_balance_server {
    hash $request_uri;
    server 192.0.0.1:8080;
    server 192.0.0.2:8080;
    server 192.0.0.3:8080;
}
```

#### Least Connections

Forward the request to the server with the fewest active connections.
```conf
upstream load_balance_server {
    least_conn;
    server 192.0.0.1:8080;
    server 192.0.0.2:8080;
    server 192.0.0.3:8080;
}
```

## Multiple Applications in Nginx

Suppose a website has several functional areas. We could treat `http://www.domain.com` as the base URL, run the application on port `8080`, and route every feature through the backend framework.

For easier maintenance, we can separate the features into independent applications. They cannot all run on the same port `8080`, so they run on different ports:
```
http://www.domain.com           -> http://192.0.0.1:8080
http://www.domain.com/finance   -> http://192.0.0.1:8081
http://www.domain.com/education -> http://192.0.0.1:8082
```

**`conf/nginx.conf`**
```conf
http {
    # The base URL uses port 8080.
    upstream base_server {
        server 192.0.0.1:8080;
        server 192.0.0.2:8080;
        server 192.0.0.3:8080;
    }
    # The finance application uses port 8081.
    upstream finance_server {
        server 192.0.0.1:8081;
        server 192.0.0.2:8081;
        server 192.0.0.3:8081;
    }
    # The education application uses port 8082.
    upstream education_server {
        server 192.0.0.1:8082;
        server 192.0.0.2:8082;
        server 192.0.0.3:8082;
    }

    server {
        listen       80;
        server_name  www.domain.com;

        location / {
            proxy_pass http://base_server;
        }

        location /finance/ {
            proxy_pass http://finance_server;
        }

        location /education/ {
            proxy_pass http://education_server;
        }
    }
}
```

> [!IMPORTANT]
> ⚠️ **Path forwarding rules**
> * `location /finance/ { proxy_pass http://finance_server; }` &rarr; the original URL path is forwarded, such as `/finance/xxxx`.
> * `location /finance/ { proxy_pass http://finance_server/; }` &rarr; the prefix is ignored, such as `/xxxx`.

## Nginx Static Files

Sometimes we want a URL to return a static resource from the server, such as an image or video. For example, given `http://domain.com/animal/cat/orangecat.jpg`, we want to return `/home/src/jpg/orangecat.jpg` from the server.

**`conf/nginx.conf`**
```conf
http {
    upstream base_server {
        server 192.0.0.1:8080;
    }

    server {
        listen       80;
        server_name  www.domain.com;

        location /animal/cat/ {
            alias /home/src/jpg;
        }
    }
}
```

> [!IMPORTANT]
> ⚠️ Pay attention to the keywords: `root` adds the URL path, while `alias` replaces it. Given `http://domain.com/animal/cat/orangecat.jpg`:
> * `alias` &rarr; `/home/src/jpg/orangecat.jpg`
> * `root` &rarr; `/home/src/jpg/animal/cat/orangecat.jpg`
