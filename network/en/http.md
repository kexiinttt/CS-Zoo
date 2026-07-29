# HyperText Transfer Protocol (HTTP)

## Features

* HTTP is **connectionless**: each connection is limited to one request. After the server processes the client's request and receives the client's response, it disconnects. This approach can save transfer time.
* HTTP is **media independent**: any type of data can be sent over HTTP as long as the client and server know how to process it. The client and server specify a suitable MIME type for the content.
* HTTP is **stateless**: each HTTP request is independent and the protocol does not remember previous transactions.

## Status Codes

| Category | Description |
| :---: | :---: |
| 1xx | Continue processing the request |
| 2xx | Success |
| 3xx | Redirection |
| 4xx | Client error |
| 5xx | Server error |

## `POST` vs. `GET`

* Browsers use `GET` to retrieve an HTML page, image, CSS file, JavaScript file, or other resource. They use `POST` to submit a `<form>` and receive a result page.
* `GET` sends the header and body together in one TCP packet; `POST` sends the header first, waits for the server's `100` response, and then sends the body in a second TCP packet.
* `GET` includes parameters in the URL, such as `www.google.com/search?q=xxx`; `POST` places parameters in the form, such as `www.google.com`.
* `GET` can be executed repeatedly, while `POST` submits the form again. A browser may show an alert such as `alert('Refresh requires resubmission')`.
* A `GET` URL can be bookmarked, while a `POST` request cannot be treated the same way because each submission is not idempotent.
* `GET` parameters are retained in full in browser history, while `POST` parameters are not retained there.
* Parameters sent in a `GET` URL have a length limit, while `POST` has no equivalent URL length limit.
* Browsers actively cache `GET` requests, while they do not normally cache `POST` requests.
* `GET` uses URL encoding, while `POST` supports multiple encoding methods.

---

# Cookie &amp; Session

* Session &rarr; a data structure stored on the server to track user state. It can be stored in a cluster, database, or file.
* Cookie &rarr; a mechanism that stores user information on the client. It records user information and is also one way to implement a Session.

HTTP is stateless, so when a server needs to track a user's state it needs a mechanism to identify that user. That mechanism is the Session. Typically, when a server first responds with a page, it asks the browser to record some information in a Cookie. Later HTTP requests can then be matched between the server and client.

## Cookie vs. Session

* A Cookie is stored on the client; a Session is stored on the server.
* A Cookie can store only a small amount of ASCII data; a Session can store arbitrary data.
* A Cookie generally has a longer lifetime and is kept in the browser for a relatively long time. A Session generally has a shorter lifetime and expires when the user session ends.
* Cookies have weaker security because they are stored on the user's side.
