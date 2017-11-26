>本章将结合先前所学的爬虫和正则表达式知识，做一个简单的爬虫案例，更多内容请参考:[Python学习指南]()  

现在拥有了正则表达式这把神兵利器，我们就可以进行对爬取到的全部网页源代码进行筛选了。  

下面我们一起尝试一下爬取内涵段子网站：  
http://www.neihan8.com/article/list_5_1.html

打开之后，不难看出里面一个一个非常有内涵的段子，当你进行翻页的时候，注意url地址的变化：

+ 第一页url: http: //www.neihan8.com/article/list_5_1 .html  
+ 第二页url: http: //www.neihan8.com/article/list_5_2 .html  
+ 第三页url: http: //www.neihan8.com/article/list_5_3 .html  
+ 第四页url: http: //www.neihan8.com/article/list_5_4 .html  

这样我们的url规律找到了，要想爬取所有的段子，只需要修改一个参数即可。  
我们就开始一步一步将所有的段子爬取下来吧。  

### 第一步：获取数据  

####1. 按照我们之前的用法，我们需要一个加载页面的方法。  
这里我们统一定义一个类，将url请求作为一个成员方法处理。  
我们创建了一个文件，叫duanzi_spider.py
然后定义一个Spider类，并且添加一个加载页面的成员方法。  
```python
import urllib2

class Spider:
    """
        内涵段子爬虫类
    """
    def loadPage(self, page):
        """
            @brief 定义一个url请求网页的方法
            @param page需要请求的第几页
            @returns 返回的页面url
        """
        url = "http://www.neihan8.com/article/list_5_" + str(page)+ ".html"
        #user-Agent头
        user_agent = "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT6.1; Trident/5.0"
        headers = {"User-Agent":user_agent}
        req = urllib2.Request(url, headers = headers)
        response = urllib2.urlopen(req)
        print html
```

>以上的loadPage的实现思想想必大家都应该熟悉了，需要注意定义python类的成员方法需要额外添加一个参数self.  

#### 2.写main函数测试一个loadPage方法
```python
if __name__ == "__main__":
    """
        =====================
            内涵段子小爬虫
        =====================
    """
    print("请按下回车开始")
    raw_input()
    
    #定义一个Spider对象
    mySpider = Spider()
    mySpider.loadPage(1)
```

+ 程序正常执行的话，我们会在皮姆上打印了内涵段子第一页的全部html代码。但是我们发现，html中的中文部分显示的可能是乱码。  
![内涵段子乱码](http://oyl9rg5dr.bkt.clouddn.com/image/%E5%86%85%E6%B6%B5%E6%AE%B5%E5%AD%90%E4%B9%B1%E7%A0%81.png)  
那么我们需要简单的将得到的网页源代码处理一下：  
```python
def loadPage(self, page):
    """
        @bridf 定义一个url请求网页的方法
        @param page 需要请求的第几页
        @returns 返回的页面html
    """

    url = "http://www.neihan8.com/article/list_5_"+str(page)+".html"
    #user-agent头
    user-agent = "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT6.1; Trident/5.0"
    headers = {"User-Agent":user-agent}
    req = urllib2.Request(url, headers = headers)
    response = urllib2.urlopen(req)
    html = response.read()
    gbk_html = html.decode("gbk").encode("utf-8")

    return gbk_html
```

>注意：对于每个网站对中文的编码各自不同，所以html.decode("gbk")的写法并不是通用的，根据网站的编码而异。  

![网站编码不同](http://oyl9rg5dr.bkt.clouddn.com/image/%E7%BD%91%E7%AB%99%E7%BC%96%E7%A0%81%E4%B8%8D%E5%90%8C.png)


### 第二步：筛选数据  
>接下来我们已经得到了整个页面的数据。但是，很多内容我们并不关心，所以下一步我们需要筛选数据。如何筛选，就用到了上一节讲述的正则表达式

+ 首先
```python
import re
```

+ 然后，我们得到的gbk_html中进行筛选匹配。  

#### 我们需要一个匹配规则  
>我们可以打开内涵段子的网页，鼠标点击右键"查看源代码"你会惊奇的发现，我们需要的每个段子的内容都是在一个`<div>`标签中，而且每个`div`标签都有一个属性`class="f18 mb20"`  
![内涵段子正则匹配](http://oyl9rg5dr.bkt.clouddn.com/image/%E5%86%85%E6%B6%B5%E6%AE%B5%E5%AD%90%E6%AD%A3%E5%88%99%E5%8C%B9%E9%85%8D.png)  

####根据正则表达式，我们可以推算出一个公式是：  
```html
<div.*?class="f18 mb20">(.*?)</div>
```
+ 这个表达式实际上就是匹配到所有`div`中`class="f18 mb20"`里面的内容(具体可以看前面介绍)  
+ 然后这个正则应用到代码中，我们会得到以下代码：  

```python
def loadPage(self, page):
    """
        @brief 定义一个url请求网页的办法
        @param page 需要请求的第几页
        @returns 返回的页面html
    """
    url = "http://www.neihan8.com/article/list_5_" +str(page) + ".html"
    #User-Agent头
    user-agent = "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT6.1; Trident/5.0"  

    headers = {"User-Agent":user-agent}
    req = urllib2.Request(url, headers=headers)
    response = urllib2.urlopen(req)

    html = response.read()

    gbk_html = html.decode("gbk").encode("utf-8")

    #找到所有的段子内容<div class="f18 mb20"></div>
    #re.S 如果没有re.S,则是只匹配一行有没有符合规则的字符串，如果没有则匹配下一行重新匹配
    #如果加上re.S,则是将所有的字符串按一个整体进行匹配
    pattern = re.compile(r'<div.*?class="f18 mb20">(.*?)</div>', re.S)
    item_list = pattern.findall(gbk_html)

    return item_list

def printOnePage(self, item_list, page):
    """
        @brief 处理得到的段子列表
        @param item_list 得到的段子列表
        @param page处理第几页
    """

    print("*********第%d页，爬取完毕...******"%page)

    for item in item_list:
        print("===============")
        print ite
```

>   + 这里需要注意一个是`re.S`是正则表达式中匹配的一个参数。  
>   + 如果没有re.S则是只匹配一行有没有符合规则的字符串，如果没有则下一行重新匹配。  
>   + 如果加上re.S则是将所有的字符串按一个整体进行匹配，findall将匹配到的所有结果封装到一个list中。  

+ 如果我们写了一个遍历`item_list`的一个方法`printOnePage()`。ok程序写到这，我们再一次执行一下。  

```python
python duanzi_spider.py
```

`我们第一页的全部段子，不包含其他信息全部的打印了出来.`  

+ 你会发现段子中有很多`<p>`,`</p>`很是不舒服，实际上这个是html的一种段落的标签。
+ 在浏览器上看不出来，但是如果按照文本打印会有`<p>`出现，那么我们只需要把我们的内容去掉即可。  
+ 我们可以如下简单修改一下printOnePage()

```python
def printOnePage(self, item_list, page):
    """
        @brief 处理得到的段子列表
        @param item_list 得到的段子列表
        @param page 处理第几页
    """
    print("******第%d页,爬取完毕*****"%page)  
    for item in item_list:
        print("============")
        item = item.replace("<p>", "").replace("</p>", "").replace("<br />", "")
        print item
```

### 第三步：保存数据  

+ 我们可以将所有的段子存放在文件中。比如，我们可以将得到的每个item不是打印出来，而是放在一个叫duanzi.txt的文件中也可以。  

```python
def writeToFile(self, text):
    """
        @brief 将数据追加写进文件中
        @param text 文件内容
    """

    myFile = open("./duanzi.txt", "a")  #a追加形式打开文件  
    myFile.write(text)
    myFile.write("-------------------------")
    myFile.close()
```

+ 然后我们将所有的print的语句改写成writeToFile(), 当前页面的所有段子就存在了本地的duanzi.txt文件中。  

```python
def printOnePage(self, item_list, page):
    """
        @brief  处理得到的段子列表
        @param item_list 得到的段子列表
        @param page 处理第几页
    """

    print("***第%d页，爬取完毕****"%page)
    for item in item_list:
        item = item.replace("<p>", "").replace("</p>", "").replace("<br />". "")

        self.writeToFile(item)
```

### 第四步：显示数据

+ 接下来我们就通过参数的传递对page进行叠加来遍历内涵段子吧的全部段子内容。  

+ 只需要在外层加上一些逻辑处理即可。  

```python
def doWork(self):
    """
        让爬虫开始工作
    """
    while self.enable:
        try:
            item_list = self.loadPage(self.page)
        except urllib2.URLError, e:
            print e.reason
            continue

    #将得到的段子item_list处理
    self.printOnePage(item_list, self.page)
    self.page += 1
    print "按回车继续...."
    print "输入quit退出"

    command = raw_input()
    if(command == "quit"):
        self.enable = False
        break
```

>   + 最后，我们执行我们的代码，完成后查看当前路径下的duanzi.txt文件，里面已经有了我们要的内涵段子。  

**以上便是一个非常精简的小爬虫程序，使用起来很是方便，如果想要爬取其它网站的信息，只需要修改其中某些参数和一些细节即可。**