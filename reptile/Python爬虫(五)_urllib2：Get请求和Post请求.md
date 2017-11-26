>本篇将介绍urllib2的Get和Post方法，更多内容请参考:[python学习指南]()  

**urllib2默认只支持HTTP/HTTPS的GET和POST方法**  

#### urllib.urlencode()
urllib和urllib2都是接受URL请求的相关参数，但是提供了不同的功能。两个最显著的不同如下：  
>+ urllib仅可以接受URL，不能创建设置了headers的Request类实例；
>+ 但是urllib提供了`urlencode`方法用来GET查询字符串的产生，而urllib2则没有。(这是urllib和urllib2经常一起使用的主要原因)  
>+ 编码工作使用urllib的`urlencode()`函数，帮我们将`key:value`这样的键值对转换成`"key=value"`这样的字符串，解码工作可以使用urllib的`unquote()`函数。(注意，不是urllib.urlencode())

```python
#-*- coding:utf-8 -*-
#06.urllib2_urlencode.py
import urllib2
import urllib

word = {"wd":"传智播客"}

#通过urllib.urlencode()方法，将字典键值对按URL编码转换，从而能被web服务器接受
encode = urllib.urlencode(word)

print(encode)
#通过urllib.unquote()方法，把URL编码字符串，转换回原始字符串

print(urllib.unquote("wd=%E4%BC%A0%E6%99%BA%E6%92%AD%E5%AE%A2"))
```
>一般HTTP请求提交数据，需要编码成URL编码格式，然后作为url的一部分，或者作为参数传到Request对象中。  

### Get方式
Get请求一般用于我们向服务器获取数据，比如说，我们用百度搜索`传智播客`；https://www.baidu.com/s?wd=传智播客  

浏览器的url会跳转如图所示
![百度传智搜索](http://oyl9rg5dr.bkt.clouddn.com/image/baidu_itcast.png)  

[https://www.baidu.com/s?wd=%E4%BC%A0%E6%99%BA%E6%92%AD%E5%AE%A2](https://www.baidu.com/s?wd=%E4%BC%A0%E6%99%BA%E6%92%AD%E5%AE%A2)  

在其中我们可以看到在请求部分里，`http://www.baidu.com/s`之后出现一个长长的字符串，其中就包含我门要查询的关键词传智播客，于是我们可以尝试使用默认的Get方式来发送请求。

```python
#-*- coding:utf-8 -*-
#07.urllib2_get.py

import urllib
import urllib2

url = "http://www.baidu.com/s"
word = {"wd":"传智播客"}
word = urllib.urlencode(word) #转换成url编码格式(字符串)

newurl = url + "?" + word

headers = {"User-Agent":"Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/51.0.2704.103 Safari/537.36"}

request = urllib2.Request(newurl,headers = headers)

response = urllib2.urlopen(request)

print(response.read())
```

#### 批量爬取贴吧页面数据
首先我们创建一个python文件，tiebaSpider.py，我们要完成的是，输入一个百度贴吧的地址，比如：  
百度贴吧LOL吧第一页：[http://tieba.baidu.com/f?kw=lol&ie=utf-8&pn=0](http://tieba.baidu.com/f?kw=lol&ie=utf-8&pn=0

第二页：[http://tieba.baidu.com/f?kw=lol&ie=utf-8&pn=50](http://tieba.baidu.com/f?kw=lol&ie=utf-8&pn=50

第二页：[http://tieba.baidu.com/f?kw=lol&ie=utf-8&pn=100](http://tieba.baidu.com/f?kw=lol&ie=utf-8&pn=100

发现规律了吧，贴吧中每个页面不同之处，就是url最后的pn的值，其余的都是一样的，我们可以抓住这个规律。  

**简单写一个小爬虫程序，来爬虫百度LOL吧的所有网页**  
+ 先写一个main，提示用户输入要爬取的贴吧名，并用urllib.urlencode()进行转码，然后组合url，假设是lol吧，那么组合的url就是: `http://tieba.baidu.com/f?kw=lol`  

```python
# 模拟 main 函数
if __name__ == "__main__":

    kw = raw_input("请输入需要爬取的贴吧:")
    # 输入起始页和终止页，str转成int类型
    beginPage = int(raw_input("请输入起始页："))
    endPage = int(raw_input("请输入终止页："))

    url = "http://tieba.baidu.com/f?"
    key = urllib.urlencode({"kw" : kw})

    # 组合后的url示例：http://tieba.baidu.com/f?kw=lol
    url = url + key
    tiebaSpider(url, beginPage, endPage)
```

+ 接下来，我们写一个百度贴吧爬虫接口，我们需要传递3个参数给这个接口，一个是main里组合url地址，以及起始页和终止页码
，表示要爬取页码的范围。  

```python
def tiebaSpider(url, beginPage, endPage):
    """
        作用：负责处理url，分配每个url去发送请求
        url：需要处理的第一个url
        beginPage: 爬虫执行的起始页面
        endPage: 爬虫执行的截止页面
    """


    for page in range(beginPage, endPage + 1):
        pn = (page - 1) * 50

        filename = "第" + str(page) + "页.html"
        # 组合为完整的 url，并且pn值每次增加50
        fullurl = url + "&pn=" + str(pn)
        #print fullurl

        # 调用loadPage()发送请求获取HTML页面
        html = loadPage(fullurl, filename)
        # 将获取到的HTML页面写入本地磁盘文件
        writeFile(html, filename)
```

+ 我们已经之前写出一个爬取一个网页的代码。现在，我们可以把它封装成一个loadPage,供我们使用。  

```python
def loadPage(url, filename):
    '''
        作用：根据url发送请求，获取服务器响应文件
        url：需要爬取的url地址
        filename: 文件名
    '''
    print "正在下载" + filename

    headers = {"User-Agent": "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Trident/5.0;"}

    request = urllib2.Request(url, headers = headers)
    response = urllib2.urlopen(request)
    return response.read()
```

+ 最后如果我们希望将爬取到了每页的信息存储在本地磁盘上，我们可以简单写一个存储文件的接口。  

```python
def writeFile(html, filename):
    """
        作用：保存服务器响应文件到本地磁盘文件里
        html: 服务器响应文件
        filename: 本地磁盘文件名
    """
    print "正在存储" + filename
    with open(filename, 'w') as f:
        f.write(html)
    print "-" * 20
```

>其实很多网站都是这样的，同类网站下的html页面编号，分别对应网址后的网页序号，只要发现规律就可以批量爬取页面了。  

### POST方式：  
上面我们说了Request请求对象里有data参数，它就是用在POST里，我们要传送的数据就是这个参数data，data是一个字典，里面要匹配键值对。  

#### 有道词典翻译网站：  

输入测试数据，再通过使用Fiddler观察，其中有一条是POST请求，而向服务器发送的请求数据并不是在url里，那么我们可以试着模拟这个POST请求。  
![有道翻译](http://oyl9rg5dr.bkt.clouddn.com/image/youdaopost.png)  
于是，我们可以尝试用POST方式发送请求。  
```python
#-*- coding:utf-8 -*-
#09.urllib2_post.py

import urllib
import urllib2

#POST请求的目标URL
url = "http://fanyi.youdao.com/translate?smartresult=dict&smartresult=rule&smartresult=ugc&sessionFrom=null"

headers = {"User-Agent":"Mozilla...."}

formate = {
    "type":"AUTO",
    "i":"i love python",
    "doctype":"json",
    "xmlVersion":"1.8",
    "keyform":"fanyi.web",
    "ue":"utf-8",
    "action":"FY_BY_ENTER",
    "typoResult":"true"
}

data = urllib.urlencode(formate)

request = urllib2.Request(url, data=data, headers = headers)

response = urllib2.urlopen(request)

print("-"*30)
print(response.read())
```

发送POST请求时，需要特别注意headers的一些属性：
>`Content-Length`：是指发送的表单数据长度为144，也就是字符个数是144个；  
`X-Requested-With`：表示Ajax异步请求。  
`Content-Type: application/x-www-form-urlencoded`：表示浏览器提交web表单时，表单数据会按照name1=value1&name2=value2键值对形式进行编码。  

#### 获取AJAX加载的内容

有些网页内容使用AJAX加载，只要记得，AJAX一般返回的是JSON，直接对AJAX地址进行post或get,就返回JSON数据了。  
"作为一名爬虫工程师，你最需要关注的，是数据的来源了。"  
```python
#-*- coding:utf-8 -*-
#10.urllib2_ajax.py
import urllib
import urllib2

# demo1

url = "https://movie.douban.com/j/chart/top_list?type=11&interval_id=100%3A90&action"

headers={"User-Agent": "Mozilla...."}

# 变动的是这两个参数，从start开始往后显示limit个
formdata = {
    'start':'0',
    'limit':'10'
}
data = urllib.urlencode(formdata)

request = urllib2.Request(url, data = data, headers = headers)
response = urllib2.urlopen(request)

print(response.read())


#demo2
url = "https://movie.douban.com/j/chart/top_list?"
headers={"User-Agent": "Mozilla...."}

# 处理所有参数
formdata = {
    'type':'11',
    'interval_id':'100:90',
    'action':'',
    'start':'0',
    'limit':'10'
}
data = urllib.urlencode(formdata)

request = urllib2.Request(url, data = data, headers = headers)
response = urllib2.urlopen(request)

print response.read()
```

#### 问题：为什么有时候POST也能在URL内看到数据?  
>+ GET方式是以直接以链接形式访问，链接中包含了所有参数，服务器端用Request.QueryString获取变量的值。如果包含了密码的话是一种不安全的选择，不过你可以直观地看到自己提交了什么内容。  
>+ POST则不会在网址上显示所有的参数，服务器端用Request.Form获取提交的数据，在Form提交的时候。但是HTML代码里如果不指定method属性，则默认为GET请求，Form中提交的数据将会附加在url之后，以`?`与url分开   
>+ 表单数据可以作为URL字段(method="get")或者HTTP POST(method="post")的方式来发送。比如在下面的HTML代码中，表单数据将因为(method="get")而附加到URL上；
```python
<form action="form_action.asp" method="get">
    <p>First name: <input type="text" name="fname" /></p>
    <p>Last name: <input type="text" name="lname" /></p>
    <input type="submit" value="Submit" />
</form>
```

![豆瓣ajax](http://oyl9rg5dr.bkt.clouddn.com/image/doubanajax.png)

####处理HTTPS请求SSL证书验证
现在随处可见https开头的网站，urllib2可以为HTTPS请求验证SSL证书，就像web浏览器一样，如果网站的SSL证书是经过CA认证的，则能够正常访问，如`https://www.baidu.com/`...  
如果SSL证书验证不通过，或者操作系统不信任服务器的安全证书，比如浏览器在访问12306网站如:"https://www.12306.cn/normhweb/"的时候，会警告用户证书不受信任。(据说 12306网络证书是自己的，没有通过CA认证)  
![12306](http://oyl9rg5dr.bkt.clouddn.com/image/12306zhengshu.png)  
urllib2在访问的时候则会报出SSLError:  
```python
import urllib2

url = "https://www.12306.cn/mormhweb/"

headers = {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.99 Safari/537.36"}

request = urllib2.Request(url, headers = headers)

response = urllib2.urlopen(request)

print(response.read())
```
运行结果：  
`urllib2.URLError: <urlopen error [SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed (_ssl.c:590)>`  
所以，如果以后遇到这种网站，我们需要单独处理SSL证书，让程序忽略SSL证书验证错误，即可正常访问。  
```python
import urllib
import urllib2
# 1. 导入Python SSL处理模块
import ssl

# 2. 表示忽略未经核实的SSL证书认证
context = ssl._create_unverified_context()

url = "https://www.12306.cn/mormhweb/"

headers = {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.99 Safari/537.36"}

request = urllib2.Request(url, headers = headers)

# 3. 在urlopen()方法里 指明添加 context 参数
response = urllib2.urlopen(request, context = context)

print response.read()
```

### 关于CA
CA(Certificate Authority)是数字证书认证中心的简称，是指发放、管理、废除数字证书的受信任的第三方机构，如北京数字认证股份有限公司、上海市数字证书认证中心有限公司等...  

CA的作用是检查证书持有者身份的合法性，并签发证书，以防证书被伪造或篡改，以及对证书和密钥进行管理。  

现实生活中可以用身份证来证明身份， 那么在网络世界里，数字证书就是身份证。和现实生活不同的是，并不是每个上网的用户都有数字证书的，往往只有当一个人需要证明自己的身份的时候才需要用到数字证书。  

普通用户一般是不需要，因为网站并不关心是谁访问了网站，现在的网站只关心流量。但是反过来，网站就需要证明自己的身份了。  

比如说现在钓鱼网站很多的，比如你想访问的是www.baidu.com，但其实你访问的是www.daibu.com”，所以在提交自己的隐私信息之前需要验证一下网站的身份，要求网站出示数字证书。  

一般正常的网站都会主动出示自己的数字证书，来确保客户端和网站服务器之间的通信数据是加密安全的。  


### 参考
1. [破解有道翻译反爬虫机制](http://www.jianshu.com/p/2a99329d565b)  
2. [浏览器验证网站数字证书的流程](http://blog.csdn.net/hejjiiee/article/details/53443357)