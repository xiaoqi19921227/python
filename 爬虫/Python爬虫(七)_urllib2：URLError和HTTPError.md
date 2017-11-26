### urllib2的异常错误处理
在我们用`urlopen`或`opener.open`方法发出一个请求时，如果`urlopen或opener.open`不能处理这个response，就产生错误。  

这里主要说的是URLError和HTTPError,以及对它们的错误处理。  

### URLError