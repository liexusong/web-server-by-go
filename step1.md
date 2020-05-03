## HTTP协议简介

WEB服务器是解析HTTP协议并根据HTTP请求的信息提供服务的应用程序，所以要编写一个WEB服务器首先需要了解HTTP协议。HTTP协议是 `Hyper Text Transfer Protocol（超文本传输协议）` 的缩写，是一个基于TCP/IP协议来传递数据的应用层协议，下面简单介绍一下HTTP协议的文法。

### HTTP请求

根据RFC2616规定，HTTP请求的格式如下图：

![Alt text](./1_8.png)

在HTTP 请求中，第一行必须是一个请求行（request line），用来说明请求类型、要访问的资源以及使用的HTTP版本。紧接着是一个或者多个首部（header）小节，用来说明服务器要使用的附加信息。在首部之后是一个空行（CR+LF），再此之后可以添加任意的其他数据，称之为请求主体（body）。

下面示例是一个简单的HTTP请求：

```text
GET / HTTP/1.1
Host: github.com
User-Agent: Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.7.6)
Gecko/20050225 Firefox/1.0.1
Connection: Keep-Alive
```

#### 请求行

请求行的格式如下图：

![Alt text](./1_9.png)

* 请求行首先要指定请求的方法名，方法名有：`GET`、`POST`、`HEAD`、`OPTIONS`、`PUT`、`DELETE`、`TRACE` 和 `CONNECT` 这8种，比较常用的有GET和POST方法。
* 方法名后面是要请求的URI（统一资源标识符），是用于标识资源名称的字符串，我们可以通过URI来确认用户要请求服务器上的哪些资源。
* URI后面是HTTP的版本号，用于指定请求使用HTTP协议版本。常用的版本有HTTP/1.0、HTTP/1.1以及近年才发布的HTTP/2.0，由于HTTP/1.1是现在最流行的的版本，所以本书主要以HTTP/1.1作为实现版本。

#### 请求首部

请求首部的格式如下图：

![Alt text](./1_10.png)

请求首部是由首部字段名和字段值组成，字段名与字段值之间使用 冒号 分割。请求首部的作用是用于指定请求的行为和属性，例如：可以通过 `Connection` 请求首部来告诉Web服务器，客户端是否希望与服务端保持长连接。

```text
 Connection: Keep-Alive
```

上面示例中，首部字段名为 `Connection`，字段值为 `Keep-Alive`。其作用是告诉Web服务器客户端需要保持长连接。

#### 请求主体

请求主体主要用于发送请求的数据，比如 POST 请求的参数、上传文件的内容等。可以通过 `Content-Type` 请求首部来指定请求主体的数据类型，比如可以通过下面的请求首部来指定请求主体的数据类型为 `JSON`：
```text
Content-Type: application/json
```

### HTTP响应

HTTP 响应与 HTTP 请求相似，HTTP响应也由3个部分构成，分别是：

* 状态行
* 响应头
* 响应正文

如下图所示：

![Alt text](./1_11.jpg)

#### 状态行

状态行由协议版本、数字形式的状态码、及相应的状态描述组成，各元素之间以空格分隔。如下图：

![Alt text](./1_12.jpg)

常见的状态码有如下几种：
* `200 OK` 客户端请求成功
* `301 Moved Permanently` 请求永久重定向
* `302 Moved Temporarily` 请求临时重定向
* `304 Not Modified` 文件未修改，可以直接使用缓存的文件。
* `400 Bad Request` 由于客户端请求有语法错误，不能被服务器所理解。
* `401 Unauthorized` 请求未经授权。这个状态代码必须和WWW-Authenticate报头域一起使用
* `403 Forbidden` 服务器收到请求，但是拒绝提供服务。服务器通常会在响应正文中给出不提供服务的原因
* `404 Not Found` 请求的资源不存在，例如，输入了错误的URL
* `500 Internal Server Error` 服务器发生不可预期的错误，导致无法完成客户端的请求。
* `503 Service Unavailable` 服务器当前不能够处理客户端的请求，在一段时间之后，服务器可能会恢复正常。


