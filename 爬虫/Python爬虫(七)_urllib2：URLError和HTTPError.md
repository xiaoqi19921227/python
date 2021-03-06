### urllib2的异常错误处理
在我们用`urlopen`或`opener.open`方法发出一个请求时，如果`urlopen或opener.open`不能处理这个response，就产生错误。  

这里主要说的是URLError和HTTPError,以及对它们的错误处理。  

### URLError

URLError产生的原因主要有： 

> 1. 没有网络连接
> 2. 服务器链接失败
> 3. 找不到指定的服务器

我们可以用`try except`语句来补货相应的异常。下面的例子里我们访问了一个不存在的域名。  

```python
#urllib2_urlerror.py
import urllib2

request = urllib2.Request("http://www.sdfsdfsf.com")

try:
    urllib2.urlopen(request, timeout= 5)
except urllib2.URLError, err:
    print err

```
运行结果如下：  
```python
<urlopen error [Errno 8] nodename nor servname provided, or not known>
```
>urlopen error,错误代码8.错误原因是没有找到指定的服务器。  

### HTTPError

HTTPError是URLError的子类，我们发出一个请求时，服务器都会对应一个response应答对象，其中它包含一个数字"响应状态码"  

如果`urlopen`或`opener.open`不能处理，会发出一个HTTPError，对应相应的状态码，HTTP状态码表示HTTP协议所返回的响应的状态。  

**注意，urllib2可以为我们处理重定向的页面(也就是3开头的响应码)，100-299范围的号码表示成功，所以我们只能看到400-599的错误号码**

```python
#-*- coding:utf-8 -*-
#23.urllib2_urllib2_httperror.py

import urllib2

request = urllib2.Request("http://blog.baidu.com/itcast")  

try:
    urllib2.urlopen(request)
except urllib2.HTTPError, err:
    print err.code
    print err
```

运行结果如下：
```python
404
HTTP Error 404:Not Found
```
>HTTP Error，错误代号是404，错误原因是Not Found,说明服务器无法找到被请求的页面。 


#### 改进版
由于HTTPError的父类是URLError,所以父类的异常应当写到子类异常的后面，所以上述的代码可以这么改写：  
```python
#-*- coding:utf-8 -*-
#24.urllib2_botherror.py

import urllib2

request = urllib2.Request("http://blog.baidu.com/itcast")

try:
    urllib2.urlopen(request)
except urllib2.HTTPError,err:
    print err.code
except urllib2.URLError,err:
    print err

else:
    print "Good Job"
```

运行结果如下：  
```python
404
```

`这样我们就可以做到，首先捕捉子类的异常，如果子类捕捉不到，那么可以捕捉父类的异常。`  

### HTTP响应状态码参考:
```html
1xx:信息

100 Continue
服务器仅接收到部分请求，但是一旦服务器并没有拒绝该请求，客户端应该继续发送其余的请求。
101 Switching Protocols
服务器转换协议：服务器将遵从客户的请求转换到另外一种协议。



2xx:成功

200 OK
请求成功（其后是对GET和POST请求的应答文档）
201 Created
请求被创建完成，同时新的资源被创建。
202 Accepted
供处理的请求已被接受，但是处理未完成。
203 Non-authoritative Information
文档已经正常地返回，但一些应答头可能不正确，因为使用的是文档的拷贝。
204 No Content
没有新文档。浏览器应该继续显示原来的文档。如果用户定期地刷新页面，而Servlet可以确定用户文档足够新，这个状态代码是很有用的。
205 Reset Content
没有新文档。但浏览器应该重置它所显示的内容。用来强制浏览器清除表单输入内容。
206 Partial Content
客户发送了一个带有Range头的GET请求，服务器完成了它。



3xx:重定向

300 Multiple Choices
多重选择。链接列表。用户可以选择某链接到达目的地。最多允许五个地址。
301 Moved Permanently
所请求的页面已经转移至新的url。
302 Moved Temporarily
所请求的页面已经临时转移至新的url。
303 See Other
所请求的页面可在别的url下被找到。
304 Not Modified
未按预期修改文档。客户端有缓冲的文档并发出了一个条件性的请求（一般是提供If-Modified-Since头表示客户只想比指定日期更新的文档）。服务器告诉客户，原来缓冲的文档还可以继续使用。
305 Use Proxy
客户请求的文档应该通过Location头所指明的代理服务器提取。
306 Unused
此代码被用于前一版本。目前已不再使用，但是代码依然被保留。
307 Temporary Redirect
被请求的页面已经临时移至新的url。



4xx:客户端错误

400 Bad Request
服务器未能理解请求。
401 Unauthorized
被请求的页面需要用户名和密码。
401.1
登录失败。
401.2
服务器配置导致登录失败。
401.3
由于 ACL 对资源的限制而未获得授权。
401.4
筛选器授权失败。
401.5
ISAPI/CGI 应用程序授权失败。
401.7
访问被 Web 服务器上的 URL 授权策略拒绝。这个错误代码为 IIS 6.0 所专用。
402 Payment Required
此代码尚无法使用。
403 Forbidden
对被请求页面的访问被禁止。
403.1
执行访问被禁止。
403.2
读访问被禁止。
403.3
写访问被禁止。
403.4
要求 SSL。
403.5
要求 SSL 128。
403.6
IP 地址被拒绝。
403.7
要求客户端证书。
403.8
站点访问被拒绝。
403.9
用户数过多。
403.10
配置无效。
403.11
密码更改。
403.12
拒绝访问映射表。
403.13
客户端证书被吊销。
403.14
拒绝目录列表。
403.15
超出客户端访问许可。
403.16
客户端证书不受信任或无效。
403.17
客户端证书已过期或尚未生效。
403.18
在当前的应用程序池中不能执行所请求的 URL。这个错误代码为 IIS 6.0 所专用。
403.19
不能为这个应用程序池中的客户端执行 CGI。这个错误代码为 IIS 6.0 所专用。
403.20
Passport 登录失败。这个错误代码为 IIS 6.0 所专用。
404 Not Found
服务器无法找到被请求的页面。
404.0
没有找到文件或目录。
404.1
无法在所请求的端口上访问 Web 站点。
404.2
Web 服务扩展锁定策略阻止本请求。
404.3
MIME 映射策略阻止本请求。
405 Method Not Allowed
请求中指定的方法不被允许。
406 Not Acceptable
服务器生成的响应无法被客户端所接受。
407 Proxy Authentication Required
用户必须首先使用代理服务器进行验证，这样请求才会被处理。
408 Request Timeout
请求超出了服务器的等待时间。
409 Conflict
由于冲突，请求无法被完成。
410 Gone
被请求的页面不可用。
411 Length Required
"Content-Length" 未被定义。如果无此内容，服务器不会接受请求。
412 Precondition Failed
请求中的前提条件被服务器评估为失败。
413 Request Entity Too Large
由于所请求的实体的太大，服务器不会接受请求。
414 Request-url Too Long
由于url太长，服务器不会接受请求。当post请求被转换为带有很长的查询信息的get请求时，就会发生这种情况。
415 Unsupported Media Type
由于媒介类型不被支持，服务器不会接受请求。
416 Requested Range Not Satisfiable
服务器不能满足客户在请求中指定的Range头。
417 Expectation Failed
执行失败。
423
锁定的错误。



5xx:服务器错误

500 Internal Server Error
请求未完成。服务器遇到不可预知的情况。
500.12
应用程序正忙于在 Web 服务器上重新启动。
500.13
Web 服务器太忙。
500.15
不允许直接请求 Global.asa。
500.16
UNC 授权凭据不正确。这个错误代码为 IIS 6.0 所专用。
500.18
URL 授权存储不能打开。这个错误代码为 IIS 6.0 所专用。
500.100
内部 ASP 错误。
501 Not Implemented
请求未完成。服务器不支持所请求的功能。
502 Bad Gateway
请求未完成。服务器从上游服务器收到一个无效的响应。
502.1
CGI 应用程序超时。　·
502.2
CGI 应用程序出错。
503 Service Unavailable
请求未完成。服务器临时过载或当机。
504 Gateway Timeout
网关超时。
505 HTTP Version Not Supported
服务器不支持请求中指明的HTTP协议版本
```