# HyperText Transfer Protocol (HTTP)

## 特点
  * HTTP是**无连接**：限制每次连接只处理一个请求。服务器处理完客户的请求，并收到客户的应答后，即断开连接。采用这种方式可以节省传输时间。
  * HTTP是**媒体独立**的：只要客户端和服务器知道如何处理的数据内容，任何类型的数据都可以通过HTTP发送。客户端以及服务器指定使用适合的MIME-type内容类型。
  * HTTP是**无状态**：每个HTTP请求都是独立的，对于事务处理没有记忆能力。

## 状态码
> 分类 | 描述
> :---: | :---:
> 1xx | 请求继续执行
> 2xx | 成功
> 3xx | 重定向
> 4xx | 客户端出错
> 5xx | 服务器出错

## `POST` vs `GET`
* 浏览器用`GET`请求来获取一个html页面/图片/css/js等资源；用`POST`来提交一个`<form>`表单，并得到一个结果的网页
* `GET`通过TCP直接把header+body组合成一个TCP包一起发出；而`POST`先发送header，服务器返回100后再发送body，即有两个TCP包
* 使用`GET`会将各种参数包含在url中，如`www.google.com/search?q=xxx`;而`POST`会将参数包含在表单里，更加安全，如`www.google.com`
* `GET`可以反复执行，而`POST`会再次提交表单（浏览器可能弹出alert('刷新后需要重新提交表单')) 
* `GET`可以bookmark，而`POST`不行，因为每次`POST`都是不幂等的
* `GET`请求参数会被完整保留在浏览器历史记录里，而`POST`中的参数不会被保留
* `GET`请求在URL中传送的参数是有长度限制的，而`POST`没有
* `GET`请求会被浏览器主动cache，而`POST`不会
* `GET`请求只能进行url编码，而`POST`支持多种编码方式

---

# Cookie &amp; Session

* Session &rarr; 是在服务端保存的一个数据结构，用来跟踪用户的状态，这个数据可以保存在集群、数据库、文件中；
* Cookie &rarr; 是客户端保存用户信息的一种机制，用来记录用户的一些信息，也是实现Session的一种方式;

由于HTTP协议是无状态的协议，所以服务端需要记录用户的状态时，就需要用某种机制来识具体的用户，这个机制就是Session。通常，在服务器第一次响应某个页面时，会让浏览器使用Cookie记录一些信息，这样之后的HTTP请求就能让服务端和客户端匹配上。

## Cookie Vs Session

* Cookie在客户端，Session在服务器端
* Cookie只能存储较少的ASCII数据，Session可以存任意数据。
* Cookie有效期比较⻓，在浏览器中一般⻓期保存。Session有效期比较短，在用户会话结束之后立刻失效。
* Cookie的安全性比较差，因此存放在用户端。