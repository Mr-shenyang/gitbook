# 1 HTTP是什么

**HTTP(HyperText Transfer Protocol)：** 超文本传输协议的简称。它确立了一种计算机之间交流通信的规范，以及相关的各种控制和错误处理方式，负责两点之间传输数据。

# 2 HTTP发展简史

![http-history](/images/read_notes/http-history.png)

# 3 HTTP知识点预览

![http-xmind](/images/read_notes/http-xmind.jpeg)

# 4 HTTP报文解析

Http报文主要由头部，实体两部分组成，具体如下图
![http-xmind](/images/read_notes/http-msg.jpeg)
这里重点介绍下头部的格式，http报文从形式上分成请求报文，和响应报文。他们的头部（header）分布如下：

## 4.1 请求报文头
![http-request](/images/read_notes/http-request.jpeg)
请求报文是由

## 4.2 响应报文头
![http-response](/images/read_notes/http-response.jpeg)

### 4.2.1 状态码

目前RFC标准里把状态码分成了五类，分别如下
|分类|说明|
|:-:|:-:|
|1××|提示信息，表示目前是协议处理的中间状态，还需要后续的操作|
|2××|成功，报文已经收到并被正确处理|
|3××|重定向，资源位置发生变动，需要客户端重新发送请求|
|4××|客户端错误，请求报文有误，服务器无法处理|
|5××|服务器错误，服务器在处理请求时内部发生了错误|

常见状态码：
|code|简介|说明|
|:-:|:-:|:-:|
|200|OK|表示一切正常|
|204|No Content|含义与“200 OK”基本相同，但响应头后没有body数据|
|206|Partial Content|HTTP分块下载或断点续传的基础|
|301|Moved Permanently|永久重定向|
|302|Found|临时重定向|
|400|Bad Request|请求报文有错误|
|403|Forbidden|表示服务器禁止访问资源|
|404|Not Found|资源在本服务器上未找到|
|405|Method Not Allowed|不允许使用某些方法操作资源，例如不允许POST只能GET|
|405|Method Not Allowed|不允许使用某些方法操作资源，例如不允许POST只能GET|
|406|Not Acceptable|资源无法满足客户端请求的条件，例如请求中文但只有英文|
|408|Request Timeout|请求超时，服务器等待了过长的时间|
|500|Internal Server Error|服务器错误|
|501|Not Implemented|客户端请求的功能还不支持|
|502|Bad Gateway|服务器作为网关或者代理时返回的错误码，表示服务器自身工作正常，访问后端服务器时发生了错误，但具体的错误原因也是不知道的|
|503|Service Unavailable|服务器当前很忙，暂时无法响应服务|

# cookie
