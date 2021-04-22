---
title: http
date: 2021-04-22 11:08:17
tags: 网络
---
# 报文格式
## 请求报文
一个 http 请求报文由四个部分组成：

- 请求行（Request-Line）
    - 示例 POST / HTTP/1.1
    - 请求行分为了三个部分：
        - 请求方法（Method）
        - 请求URI
        - HTTP 协议版本
- 请求头部（Request Header Fields）
    - 这部分由成对的请求头部组成，用来告知服务端请求的更多信息。
- 回车换行（CRLF）
- 消息体（Message Body）
    - 这部分携带了本次请求需要发往服务端的信息，有的 Method 有这部分，而有的 Method 不需要这部分。
    比如 get 方法就没有消息体，get 方法一般都是通过 query 来传递参数。而 post 方法一般就有消息体。
## 响应报文
一个 http 响应报文也由四个部分组成：

- 状态行（Status-Line）
    - HTTP/1.1 200 OK
    - 上面是一个典型的 http 响应状态行，我们可以看到也是由三部分组成的：

        - http 协议版本
        - 状态码（Status Code）
        - 状态码的文本描述（Reason-Phrase）

- 响应头部（Response Header Fields）
    - 和请求头部类似，就是两者之间有一些不同的头部字段。
- 回车换行（CRLF）
- 消息体（Message Body）
    - 返回给客户端的具体消息