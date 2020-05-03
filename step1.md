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

下面是一个HTTP响应的例子：

```text
HTTP/1.1 200 OK
Server: Apache Tomcat/5.0.12
Date: Mon,6Oct2003 13:23:42 GMT
Content-Length: 112

<html>
...
</html>
```

因为HTTP协议是个比较庞大的协议，不可能在本章完全把其阐述清楚，所以以后用到的时候再详细介绍。

## Go网络编程

由于WEB服务器一般使用 `TCP协议` 作为传输层协议，所以本节主要介绍怎么使用Go语言的 `net` 包来进行TCP编程。

### TCP 的 C/S 架构

一般来说，编写 C/S（Client/Server） 架构的程序都有比较统一的模式，如下图所示：

![Alt text](./1_13.png)

TCP服务器首先调用 `net` 包的 `Listen()` 函数创建一个监听端口的 `Listener` 对象，然后通过调用 `Listener` 对象的 `Accept()` 方法来接收客户端连接。`Accept()` 方法会返回一个 `Conn` 对象（客户端连接），然后就可以通过调用 `Conn` 对象的 `Read() / Write()` 方法来对客户端连接进行读写操作。

下面通过一个例子来介绍怎么使用 `net` 包来编写一个TCP服务端程序：
```go
package main

import (
	"fmt"
	"net"
)

func main() {
	// 监听8080端口
	listen, err := net.Listen("tcp", "127.0.0.1:8080")
	if err != nil {
		fmt.Println("err = ", err)
		return
	}

	defer listen.Close()

	// 阻塞等待用户链接
	conn, err := listen.Accept()
	if err != nil {
		fmt.Println("err = ", err)
		return
	}

	buf := make([]byte, 1024)

	n, err := conn.Read(buf) // 读取客户端请求数据
	if err != nil {
		fmt.Println("err = ", err)
		return
	}

	fmt.Println("buf = ", string(buf[:n])) // 打印客户端请求
	defer conn.Close()
}
```

我们把上面程序编译后运行，然后通过浏览器访问 `http://127.0.0.1:8080` ，服务器会打印以下结果：

```text
buf =  GET / HTTP/1.1
Host: localhost:8080
Connection: keep-alive
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.130 Safari/537.36
Sec-Fetch-User: ?1
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-TW,zh;q=0.9,en-US;q=0.8,en;q=0.7
```
上面的输出结果就是HTTP请求的数据，WEB服务器就是通过解析HTTP请求，然后根据不同的HTTP请求来进行相应的操作。

## 一个简单WEB服务器

最后，我们通过一个编写一个简单的WEB服务器来结束本章。这个WEB服务器只返回一条信息：“`This is simple WEB server`”。

按照上面C/S架构的例子，我们先编写大概的服务端骨架：

```go
package main

import (
	"fmt"
	"net"
)

func connResp(conn net.Conn) {
}

func main() {
	// 监听8080端口
	listen, err := net.Listen("tcp", "127.0.0.1:8080")
	if err != nil {
		fmt.Println("err = ", err)
		return
	}

	defer listen.Close()

    // 无限循环
	for {
		// 阻塞等待用户链接
		conn, err := listen.Accept()
		if err != nil {
			fmt.Println("err = ", err)
			return
		}

		buf := make([]byte, 1024)

		n, err := conn.Read(buf) // 读取客户端请求数据
		if err != nil {
			fmt.Println("err = ", err)
			return
		}

		fmt.Println("buf = ", string(buf[:n])) // 打印客户端请求

		connResp(conn) // 返回数据给客户端连接
	}
}
```
在上面的代码中，我们首先通过调用 `net.Listen()` 方法来创建一个 `Listener` 对象来监听 `8080` 端口，然后在一个无限循环中调用 `Listener` 对象的 `Accept()` 方法来接收客户端连接，`Accept()` 方法返回一个 `Conn` 对象。接着通过调用 `Conn` 对象的 `Read()` 方法来读取客户端连接的HTTP请求，然后通过调用 `connResp()` 函数来返回数据给客户端请求。

> 注意：为什么要在无限循环中接收客户端连接呢？因为如果不在无限循环中接收客户端连接，那么程序处理完一个请求后便会退出进程。

注意到上面的 `connResp()` 函数还没有进行任何处理，所以我们需要继续编程返回数据的逻辑代码，如下：

```go
func connResp(conn net.Conn) {
	httpResp := "HTTP/1.1 200 OK\r\n"       // 状态行
	httpResp += "Connection: closed\r\n"    // 响应头
	httpResp += "\r\n"                      // 空行
	httpResp += "This is simple WEB server" // 响应正文

	conn.Write([]byte(httpResp))

	conn.Close()
}
```

上面的代码构建了HTTP响应数据，然后通过调用 `Conn` 对象的 `Write()` 方法把响应数据发送给客户端。我们编译并运行上面的程序，然后通过浏览器访问 `http://127.0.0.1:8080`，服务器就会返回 “`This is simple WEB server`” 消息给浏览器。

当然，现在这个服务器并没有什么作用，因为并不能根据我们的HTTP请求来进行不同的处理，但我们可以通过这个程序来了解到浏览器和WEB服务器之间是怎么通讯的，接下来的章节主要在这个程序的基础上不断完善，从而实现一个完整的WEB服务器。


