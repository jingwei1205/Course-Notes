## 第一章 网络爬虫之规则

#### 一、Requests库入门
1. request库:http://www.python-requests.org
2. 安装方法：pip install requests
3. 抓取百度
```
import requests
r=requests.get("http://www.baidu.com")
r.status_code
r.encoding='utf-8'
r.text
```
4. requests库的7个主要方法

方法 | 说明
---|---
requests.request() | 构造一个请求，支撑以下各方法的基础方法
requests.get() | 获取HTML网页的主要方法，对应于HTTP的GET
requests.head | 获取网页头信息的方法，对应于HTTP的HEAD
requests.post() | 向HTML网页提交POST请求的方法，对应于HTTP的POST
requests.put() | 向HTML网页提交PUT请求的方法，对应于HTTP的PUT
requests.patch | 向HTML网页提交局部修改请求，对应于HTTP的PATCH
requests.delete | 向HTML页面提交删除请求，对应于HTTP的DELETE
##### requests库的get()方法
1. 获取网页最简单的方法：r=requests.get(url)
    - 构造一个向服务器请求资源的Request对象
    - 返回一个包含服务器资源的Response对象
2. 形式
    ```
    requests.get(url,params=None,**kwargs)
    ```
    - url:拟获取页面的url链接
    - params：url中的额外参数，字典或字节流格式，可选
    - kwargs：12个控制访问的参数
3. Response对象：包含爬虫返回的内容
    
    - 常用的五个属性

属性 | 说明
--- | ---
r.status_code | HTTP请求的返回状态，200表示连接成功，404表示失败
r.text | HTTP响应内容的字符串形式，即，url对应的页面内容
r.encoding | 从HTTP header中猜测的响应内容编码方式
r.apparent_encoding | 从内容中分析出的响应内容编码方式（备选编码方式）
r.content | HTTP响应内容的二进制形式
4. 基本流程
```
graph TD
A(r.status_code)-->|200| B(r.text,r.encoding,r.apparent_encoding,r.content)
A-->|404或其他| C(某些原因出错将产生异常)
```
5. 理解Response的编码
    - r.encoding:如果header中不存在charset，则认为编码为ISO-8859-1
    - r.apparent_encoding:根据网页内容分析出的编码方式  
##### 理解Requests库的异常  
异常|说明
---|---
requests.ConnectionError|网络连接错误异常，如DNS查询失败、拒绝连接等
requests.HTTPError|HTTP错误异常
requests.URLRequired|URL缺失异常
requests.TooManyRedirects|超过最大重定向次数，产生重定向异常
requests.ConnectTimeout|连接远程服务器超时异常
requests.Timeout|请求URL超时，产生超时异常
r.raise_for_status()|如果不是200，产生异常requests.HTTPError
1. 通用代码框架
```python
import requests
def getHTMLText(url):
    try:
        r=requests.get(url,timeout=30)
        r.raise_for_status()
        r.encoding=r.apparent_encoding
        return r.text
    except:
        return "Error"
if __name__=="__main__":
    url=""
    print(getHTMLText(url)
```
##### HTTP协议及Requests库方法
1. HTTP：超文本传输协议，基于请求与响应模式的无状态的应用层协议，采用URL作为定位网络资源的标识。
2. URL格式：http://host[:port][path]
    - host:合法的Internet主机域名或IP地址
    - port：端口号，缺省端口为80
    - 请求资源的路径  
3. HTTP协议对资源的操作

方法|说明
---|---
GET|请求获取URL位置的资源
HEAD|请求获取URL位置资源的响应消息报告，即获得该资源的头部信息
POST|请求向URL位置的资源后附加新的数据
PUT|请求向URL位置存储一个资源，覆盖原URL位置的资源
PATCH|请求局部更新URL位置的资源，即改变该资源的部分内容
DELETE|请求删除URL位置存储的资源
4. HTTP协议对资源的操作
```
graph LR
A(人)-->|PUT POST PATCH DELETE| B(URL)
B-->|GET HEAD| A
```
5. 理解PATCH和PUT的区别
    - 假设URL位置有一组数据UserInfo，包括UserID、UserName等20个字段，需求是用户修改了UserName，其他不变
        - 采用Patch，仅向URL提交UserName的局部更新请求
        - 采用PUT，必须将所有20个字段一并提交到URL，未提交字段被删除
    - PATCH节省网络带宽  
##### Requests库主要方法解析  
1. *.request(method,url,\*\*kwargs)*
    - method:请求方式，对应get、put、post等7种
    - url：拟获取页面的url链接
    - **kwargs：控制访问的参数，共13个
        - params:字典或字节序列，作为参数增加到url中
        ```
        import requests
        kv={'key1':'value1','key2':'value2'}
        r=requests.request('GET','http://python123.io/ws',params=kv)
        print(r.url)
        结果：https://python123.io/ws?key1=value1&key2=value2
        ```
        - data：字典、字节序列或文件对象，作为Request的内容
        ```
        kv={'key1':'value1','key2':'value2'}#也可以字符串
        r=requests.request('POST','http://python123.io/ws',data=kv)
        ```
        - json：JSON格式的数据，作为Request的内容
        ```
        kv={'key1':'value1'}
         r=requests.request('POST','http://python123.io/ws',json=kv)
        ```
        - headers:字典，HTTP定制头
        ```
        hd={'user-agent':'Chrome/10'}
         r=requests.request('POST','http://python123.io/ws',headers=hd)
        ```
        - cookies：字典或CookieJar，Request中的cookie
        - auth：元组，支持HTTP认证功能
        - files：字典类型，传输文件
        ```
        fs={'file':open('data.xls','rb')}
         r=requests.request('POST','http://python123.io/ws',files=fs)
        ```
        - timeout：设定超时时间，秒为单位
        ```
        pxs={'http':'http://user:pass@10.10.10.1:!234'
             'https':'https://10.10.10.1:4321'}
         r=requests.request('POST','http://python123.io/ws',proxies=pxs)
        ```
        
        - proxies：字典类型，设定访问代理服务器，可以增加登录认证
        - allow_redirects：True/False，默认T，重定向开关
        - stream：T/F,默认T，获取内容立即下载开关
        - verify：T/F，默认T，认证SSSL证书开关
        - cert：本地SSL证书路径
2. *.get(url,params=None,\*\*kwargs)*
    - url：拟获取页面的url链接
    - params：url中的额额外参数，字典或字节流格式，可选
    - **kwargs：控制访问的参数，共12个
3. *.head(url,\*\*kwargs)*
    - url：拟获取页面的url链接
    - **kwargs：控制访问的参数，共13个
4. *.post(url,data=None,json=None,\*\*kwargs)*
    - url：拟获取页面的url链接
    - data：字典、字节序列或文件对象，作为Request的内容
    - json：JSON格式的数据，作为Request的内容
    - **kwargs：控制访问的参数，共11个
5. *.put(url,data=None,\*\*kwargs)*
    - url：拟获取页面的url链接
    - data：字典、字节序列或文件对象，作为Request的内容
    - **kwargs：控制访问的参数，共12个
6. *.patch(url,data=None,\*\*kwargs)*
    - url：拟获取页面的url链接
    - data：字典、字节序列或文件对象，作为Request的内容
    - **kwargs：控制访问的参数，共12个
7. *.delete(url,\*\*kwargs)*
    - url：拟获取页面的url链接
    - **kwargs：控制访问的参数，共13个

#### 二、网络爬虫的“盗亦有道”
##### 网络爬虫引发的问题
1. 小规模，数据量小，爬取速度不敏感，使用Requests库即可，爬取网页，玩转网页。（使用率>90%)
2. 中规模，数据规模较大，爬取速度敏感Scrapy库，爬取网站，爬取系列网站。
3. 大规模，搜索引擎爬取速度关键定制开发，爬取全网。
4. 网络爬虫的侵扰：受限于编写水平和目的，对网站形成骚扰。
5. 网络爬虫的法律风险：服务器上的数据有产权归属，网络爬虫获取数据后牟利带来法律风险。
6. 个人隐私泄露。
7. 网络爬虫的限制：来源审查，判断USER-AGENT进行限制，检查来访HTTP协议头的USER-AGENT域，只响应浏览器或友好爬虫的访问；发布协议，ROBOTS协议。
##### Robots协议
1. 网络爬虫排除标准（Robots Exclusion Standard）
    - 作用：网站告知哪些页面可以抓取，哪些不行。
    - 形式：在网站根目录下的robots.txt文件。
    - 基本语法：
        - User-agent：*任何爬虫
        - Disallow：/
##### Robots协议的使用
- 网络爬虫：自动或人工识别robots.txt,再进行内容爬取。
- 约束性：Robots协议是建议但非约束性，网络爬虫可以不遵守，但存在法律风险。
- 类人行为可不参考Robots协议
#### 三、Requests库网络爬虫实战
##### 京东商品页面爬取实例
```
import requests
url="https//item.jd.com/2967929.html"
try:
    r=requests.get(url)
    r.raise_for_status()#如果返回200不产生异常，否则产生异常
    r.encoding=r.apparent_encoding
    print(r.text[:1000])
except:
    print("爬取失败")
```
##### 网络图片的爬取与存储
```
import requests
import os
url='http://img0.dili360.com/pic/2019/12/25/5e0326307c5bd9515777642_t.jpg'
root='D://pics//'
path=root+url.split('/')[-1]
try:
    if not os.path.exists(root):
        os.mkdir(root)
    if not os.path.exists(path):
        r=requests.get(url)
        f=open(path,'wb')
        f.close()
        print("sucessful")
    else:
        print("failed")
except:
    print("spider can not run")
```
---
## 第二章 网络爬虫之提取
---
#### 一、Beautiful Soup库入门
##### 安装

1. 了解更多：https://www.crummy.com/software/BeautifulSoup/
2. 使用方法
```
from bs4 import BeautifulSoup
soup=BeautifulSoup('<>','html.parser')
```
##### BS4库的基本元素
1. BS库解析器

解析器|使用方法|条件
---|---|---
bs4的HTML解析器|BeautifulSoup(mk,'html.parser')|安装bs4库
lxml的HTML解析器|BeautifulSoup(mk,'lxml')|pip install lxml
lmxl的XML解析器|BeautifulSoup(mk,'xml')|pip install lxml
html5lib的解析器|BeautifulSoup(mk,'html5lib')|pip install html5lib  
2. Beautiful Soup类的基本元素

基本元素|说明
---|---
Tag|标签，最基本的信息组织单元，分别用<></>标明开头和结尾
Name|标签的名字，\<p>……\</p>的名字是'p',格式\<tag>.name
Attributes|标签的属性，字典形式组织，格式：\<tag>.attrs
NavigableString|标签内非属性字符串，<>……</>的字符串，格式\<tag>.string
Comment|标签内字符串注释部分，一种特殊的Comment类型  
##### 基于bs4库的HTML内容遍历方法
1. 标签树的下行遍历

属性|说明
---|---
.contents|子节点的列表，将tag所有儿子节点存入列表
.children|子节点的迭代类型，与.contents类似，用于循环遍历
.descebdants|子孙节点的迭代类型，包含所有子孙节点，用于循环遍历
2. 标签树的上行遍历

属性|说明
---|---
.parent|节点的父亲标签
.parents|节点先辈标签的迭代类型，用于循环遍历先辈节点
3. 标签树的平行遍历

属性|说明
:-:|:-:
.next_sibling|返回按照HTML文本顺序的下一个平行节点标签
.previous_sibling|返回按照HTML文本顺序的上一个平行节点标签
.next_siblings|迭代类型，返回按照HTML文本顺序的后续所有平行节点标签
.previous_siblings|迭代类型，返回按照HTML文本顺序的前序所有平行节点标签
##### 基于bs4库的HTML的格式与编码
1. bs4库的prettify()方法
2. 默认utf-8编码格式的

#### 二、信息标记
##### 信息标记的三种形式
1. 标记后的信息可形成信息组织结构，增加了信息维度
2. 标记后的信息可用于通信、存储或展示
3. 标记的结构与信息一样具有重要价值
4. 标记后的信息更利于程序理解和运用
5. json：有类型键值对
```
"key":"value"
"key":["value1","value2"]
"key":{"subkey":"subvalue"}
```
6. yaml:无类型键值对，利用缩进表示
```
name:xxx
name:
    newname:xxx
    oldname:xxx
name:并列关系
-xxx
-xxx
text: | #表示注释 |表达整块数据
```
##### 三种信息标记形式的比较
- XML：Internet上的信息交互与传递。
- JSON:移动应用云端和节点的信息通信，无注释。
- YAML：各类系统的配置文件，有注释易读。
##### 信息提取的一般方法
- 方法一：完整解析信息的标记形式，再提取关键信息。
    - XML JSON YAML需要标记解析器 例如bs4库的标签树遍历
    - 优点：信息解析准确
    - 缺点：提取过程繁琐，速度慢。
- 方法二：无视标记形式，直接搜索关键信息。
    - 搜索：对信息的文本查找函数即可。
    - 优点：提取过程简洁，速度较快。
    - 缺点：提取结果准确性与信息内容相关。
- 融合方法：结合形式解析与搜索方法，提取关键信息。
    - XML JSON YAML 搜索需要标记解析器及文本查找函数。
##### 基于bs4库的HTML内容查找方法
1. *\<>.find_all(name,attrs,recursive,string,\*\*kwargs)*
    - 返回一个列表类型，存储查找的结果
    - name：对标签名称的检索字符串。
    - attrs:对标签属性值的检索字符串，可标注属性检索。
    - recursive：是否对子孙全部检索，默认True。
    - string：\<>……\</>中字符串区域的检索字符串。
    ```
    for tag in soup.find_all(True):
        print(tag.name)#所有标签信息
    for tag in soup.find_all(re.compile('b'):
        print(tag.name)#标签以b开头
    soup.find_all('a')#查找a标签
    soup.find_all(['a','b'])#查找ab标签
    soup.find_all('p','course')#带有course属性值的p标签
    soup.find_all(id='link1')#查找id属性link1的标签
    soup.find_all(id=re.compile('link'))#查找ab标签
    soup.find_all('a',recursive=False)#在儿子节点查有无a标签的存在
    soup.find_all(string='Basic Python')#检索带有字符串的标签
    soup.find_all(string=re.compile('python'))#检索所有带有python的字符串域
    ```
2. 简写形式：
    - <tag>(...)=<tag>.find_all(...)
    - soup(...)=soup.find_all(...)
3. 扩展方法

方法|说明
---|---
.find|搜索且只返回一个结果，字符串类型，参数同.find_all()参数
.find\_parents()|在先辈节点中搜索，返回列表类型，参数同.find_all()参数
.find\_parent()|在先辈节点中返回一个结果，返回列表类型，参数同.find_all()参数
.find\_next\_siblings()|在后续平行节点中搜索，返回列表类型，参数同.find_all()参数。
.find\_next\_sibling()|在后续平行节点中返回一个结果，字符串类型，参数同.find_all()参数。
.find\_previous\_siblings()|在前序平行节点中搜索，返回列表类型，参数同.find_all()参数。
.find\_previous\_sibling()|在前序平行节点中返回一个结果，字符串类型，参数同.find_all()参数。


#### 中国大学排名爬虫
1. 功能描述
    - 输入：大学排名URL链接
    - 输出：大学排名信息的屏幕输出（排名，大学名称，总分）
    - 技术路线：requests-bs4
    - 定向爬虫：仅对输入URL进行爬取，不扩展爬取。
2. 程序的结构设计
    - 步骤1：从网络上获取大学排名网页内容——从网络上获取大学排名网页内容GetHTMLText()
    - 步骤2：提取网页内容中信息到合适的数据结构——提取网页内容中信息到合适的数据结构fillUnivList()
    - 步骤三：利用数据结构展示并输出结果printUnivList()
```
import requests
from bs4 import BeautifulSoup
import bs4

def getHTMLText(url):
    try:
        r=requests.get(url,timeout=30)
        r.raise_for_status
        r.encoding=r.apparent_encoding
        return r.text
    except:
        return 'Errors!'

def fillUnivList(ulist,html):
    soup=BeautifulSoup(html,'html.parser')
    for tr in soup.find('tbody').children:
        if isinstance(tr,bs4.element.Tag):
            tds=tr('td')
            ulist.append([tds[0].string,tds[1].string,tds[2].string])

def printUnivList(ulist,num):
    tplt="{0:^10}\t{1:{3}^10}\t{2:^10}"
    print(tplt.format("排名","学校名称","地区",chr(12288)))
    for i in range(num):
        u=ulist[i]
        print(tplt.format(u[0],u[1],u[2],chr(12288)))

    
def main():
    unifo=[]
    url="http://www.zuihaodaxue.cn/zuihaodaxuepaiming2016.html"
    html=getHTMLText(url)
    fillUnivList(unifo,html)
    printUnivList(unifo,20)
    
main()
```
---
## 第三章 网络爬虫之实战
---
#### 一、Re库入门
##### 正则表达式的概念
1. 正则表达式
    - 通用额字符串表达框架
    - 简洁表达一组字符串的表达式
    - 针对字符串表达“简洁”和“特征”思想的工具
    - 判断某字符串的特征归属
2. 正则表达式在文本处理中十分常用
    - 表达文本类型的特征
    - 同时查找和替换一组字符串
    - 匹配字符串的全部或部分
3. 正则表达式的使用
    - 编译：将符合正则表达式语法的字符串转换成正则表达式特征。
##### 正则表达式的语法
1. 正则表达式语法由字符和操作符构成

操作符|说明|实例
---|---|---
.|表示任何单个字符|
[]|字符集，对单个字符给出取值范围|[abc]表示a、b、c,[a-z]表示a-z的单个字符
[^]|非字符集，对单个字符给出排除范围|[^abc]表示非a或b或c的单个字符
\*|前一个字符0次或无限次扩展|abc\*表示ab、abc、abcc、abccc等
\+|前一个字符0次或无限次扩展|abc\+表示abc、abcc、abccc等
?|前一个字符0次或1扩展|abc\+表示ab、abc
\||左右表达式任意一个|abc\|def表示abc、def
{m}|扩展前一个字符m次|ab{2}c表示abbc
{m,n}|扩展前一个字符m至n次（含n）|ab{1,2}表示abc、abbc
^|匹配字符串开头|^abc表示abc且在一个字符串的开头
$|匹配字符串结尾|abc$表示abc且在一个字符串的结尾
（）|分组标记，内部只能使用\|操作符|(abc|def)表示abc、def
\d|数字，等价[0-9]|
\w|单词字符，等价于[A-Za-z0-9]|
2. 正则表达式语法实例

正则表达式|对应字符串
---|---
P(Y\|YT\|YTH\|YTHO)?N|PN、PYN、PYYN、PYTHN、 PYTHON
3. 经典正则表达式实例

表达式|含义
---|---
\^[A\-Za\-z]\+$|由26个字母组成的字符串
\^[A-Za-z0-9]+$|由26个字母和数字组成的字符串
\^\-?\d\+$|整数形式的字符串
\^[0-9]\*[1\-9][0\-9]\*$|正整数形式的字符串
[1-9]\d{5}|中国境内邮政编码，6位
[\u4e00-\u9fa5]|匹配中文字符
\d{3}-\d{8}\|\d{4}-\d{7}|国内电话号码，021-12345678
##### Re库的基本使用
1. Re库介绍：Re库是Python的标准库，主要用于字符串匹配,import re
2. 正则表达式的表示类型：
    - raw string类型（原生字符串类型）
        - re库采用raw  string类型表示正则表达式，表示为：r'text'
        - raw string是不包含转义字符串的字符串
    - string类型，更繁琐，不推荐
3. Re库主要功能函数

函数|说明
---|---
re.search()|在一个字符串中搜索匹配正则表达式的第一个位置，返回match对象
re.match()|从一个字符串的开始位置起匹配正则表达式，返回match对象
re.findall()|搜索字符串，以列表形式返回全部能匹配的子串
re.split()|将一个字符串按照正则表达式匹配结果进行分割，返回列表类型
re.finditer()|搜索字符串，返回一个匹配结果的迭代类型，每个迭代元素是match对象
re.sub()|在一个字符串中替换所有匹配正则表达式的子串，返回替换后的子串
4. *re.search(pattern,string,flags=0)*
    - pattern：正则表达式的字符串或原生字符串表示
    - string：待匹配字符串
    - flags：正则表达式使用时的控制标记
        - re.I,re.IGNORECASE:忽略正则表达式的大小写，[A-Z]能够匹配小写字符
        - re.M,re.MULTILINE：正则表达式中的^操作符能够将给定字符串的每行当作匹配开始
        - re.S,re.DOTALL：正则表达式中的.操作符能够匹配包含换行符的所有字符，默认匹配除换行外的所有字符
5. *re.split(pattern,string,maxsplit=0，flags=0)*
    -maxsplit：最大分割数，剩余部分作为最后一个元素输出
6. *re.sub(pattern,repl,string,count=0,maxsplit=0，flags=0)*
    - repl:替换匹配字符串的字符串
    - count:匹配的最大替换次数
7. Re库的另一种等价用法
    - 函数式用法：一次性操作
    ```
    rst=re.search(r'[1-9]\d{5}','BIT 100081')
    ```
    - 面向对象用法：编译后的多次操作
    ```
    pat=re.compile(r'[1-9]\d{5}')
    rst=pat.search('BIT 100081')
    ```
8. re.compile(pattern,flags=0)
    - 将正则表达式的字符串形式编译成正则表达式对象
    - pattern：正则表达式的字符串或原生字符串表示
    - flags：正则表达式使用时的控制标记
##### Re库的match对象
1. match对象的属性

属性|说明
---|---
.string|待匹配的文本
.re|匹配时使用的pattern对象（正则表达式）
.pos|正则表达式搜索文本的开始位置
.endpos|正则表达式搜索文本的结束位置
2. match对象的方法

方法|说明
---|---
.group(0)|获得匹配后的字符串
.start()|匹配字符串在原始字符串的开始位置
.end()|匹配字符串在原始字符串的结束位置
.span()|返回(.start(),.end())
##### Re库的贪婪匹配和最小匹配
1. 贪婪匹配：Re库默认采用贪婪匹配，即输出匹配最长的子串。
2. 最小匹配操作符

操作符|说明
---|---
*?|前一个字符0次或无限次扩展，最小匹配
+?|前一个字符1次或无限次扩展，最小匹配
??|前一个字符0次或1次扩展，最小匹配
{m,n}?|扩展前一个字符m次至n次（含n），最小匹配
#### 二、淘宝商品比价定向爬虫
1. 功能描述
    - 目标：获取淘宝搜索页面的信息，提取其中的商品名称和价格
    - 理解：淘宝的搜索借口，翻页的处理
    - 技术路线：requests-re
2. 程序结构设计
    - 步骤1：提交商品搜索请求，循环获取界面。
    - 步骤2：对于每个页面，提取商品名称和价格信息。
    - 将信息输出到屏幕上。
```
import requests
import re
 
def getHTMLText(url):
    try:
        r = requests.get(url, timeout=30)
        r.raise_for_status()
        r.encoding = r.apparent_encoding
        return r.text
    except:
        return ""
     
def parsePage(ilt, html):
    try:
        plt = re.findall(r'\"view_price\"\:\"[\d\.]*\"',html)
        tlt = re.findall(r'\"raw_title\"\:\".*?\"',html)
        for i in range(len(plt)):
            price = eval(plt[i].split(':')[1])
            title = eval(tlt[i].split(':')[1])
            ilt.append([price , title])
    except:
        print("")
 
def printGoodsList(ilt):
    tplt = "{:4}\t{:8}\t{:16}"
    print(tplt.format("序号", "价格", "商品名称"))
    count = 0
    for g in ilt:
        count = count + 1
        print(tplt.format(count, g[0], g[1]))
         
def main():
    goods = '书包'
    depth = 3
    start_url = 'https://s.taobao.com/search?q=' + goods
    infoList = []
    for i in range(depth):
        try:
            url = start_url + '&s=' + str(44*i)
            html = getHTMLText(url)
            parsePage(infoList, html)
        except:
            continue
    printGoodsList(infoList)
     
main()
```
#### 三、股票数据定向爬虫
1. 功能描述
    - 技术路线：bs4-re-requests
    - 目标：获取上交所和深交所所有股票的名称和交易信息
    - 输出：保存到文件中
2. 候选数据网站的选择
    - 选取原则：股票信息静态存在于HTML页面中，非js代码生成，没有Robots协议限制。
    - 选取方法：浏览器F12，源代码查看等。
    - 选取心态：不要纠结于某个网站，多找信息源尝试。
3. 程序的结构设计
    - 步骤1：从东方财富网获取股票列表
    - 步骤2：根据股票列表逐个到百度股票获取个股信息
    - 步骤3：将结果存储到文件
```
import requests
from bs4 import BeautifulSoup
import traceback
import re
 
def getHTMLText(url, code="utf-8"):
    try:
        r = requests.get(url)
        r.raise_for_status()
        r.encoding = code
        return r.text
    except:
        return ""
 
def getStockList(lst, stockURL):
    html = getHTMLText(stockURL, "GB2312")
    soup = BeautifulSoup(html, 'html.parser') 
    a = soup.find_all('a')
    for i in a:
        try:
            href = i.attrs['href']
            lst.append(re.findall(r"[s][hz]\d{6}", href)[0])
        except:
            continue
 
def getStockInfo(lst, stockURL, fpath):
    count = 0
    for stock in lst:
        url = stockURL + stock + ".html"
        html = getHTMLText(url)
        try:
            if html=="":
                continue
            infoDict = {}
            soup = BeautifulSoup(html, 'html.parser')
            stockInfo = soup.find('div',attrs={'class':'stock-bets'})
 
            name = stockInfo.find_all(attrs={'class':'bets-name'})[0]
            infoDict.update({'股票名称': name.text.split()[0]})
             
            keyList = stockInfo.find_all('dt')
            valueList = stockInfo.find_all('dd')
            for i in range(len(keyList)):
                key = keyList[i].text
                val = valueList[i].text
                infoDict[key] = val
             
            with open(fpath, 'a', encoding='utf-8') as f:
                f.write( str(infoDict) + '\n' )
                count = count + 1
                print("\r当前进度: {:.2f}%".format(count*100/len(lst)),end="")
        except:
            count = count + 1
            print("\r当前进度: {:.2f}%".format(count*100/len(lst)),end="")
            continue
 
def main():
    stock_list_url = 'https://quote.eastmoney.com/stocklist.html'
    stock_info_url = 'https://gupiao.baidu.com/stock/'
    output_file = 'D:/BaiduStockInfo.txt'
    slist=[]
    getStockList(slist, stock_list_url)
    getStockInfo(slist, stock_info_url, output_file)
 
main()
```
---
## 第四章 网络爬虫之框架
---
#### 一、Scrapy爬虫框架介绍
1. Scrapy的安装：执行pip install srapy
2. 安装后测试：执行scrapy -h
3. 爬虫框架
    - 爬虫框架是实现爬虫功能的一个软件结构和功能组件集合。
    - 爬虫框架是一个半成品，能够帮助用户实现专业网络爬虫。
4. scrapy爬虫框架结构
##### Scrapy主要包括了以下组件：
- 引擎(Scrapy):用来处理整个系统的数据流处理, 触发事务(框架核心)
- 调度器(Scheduler):用来接受引擎发过来的请求, 压入队列中, 并在引擎再次请求的时候返回.可以想像成一个URL（抓取网页的网址或者说是链接）的优先队列,由它来决定下一个要抓取的网址是什么, 同时去除重复的网址。
- 下载器(Downloader)：用于下载网页内容,并将网页内容返回给蜘蛛(Scrapy下载器是建立在twisted这个高效的异步模型上的)
- 爬虫(Spiders)：爬虫是主要干活的,用于从特定的网页中提取自己需要的信息,即所谓的实体(Item)。用户也可以从中提取出链接,让Scrapy继续抓取下一个页面
- 项目管道(Pipeline)：负责处理爬虫从网页中抽取的实体，主要的功能是持久化实体、验证实体的有效性、清除不需要的信息。当页面被爬虫解析后，将被发送到项目管道，并经过几个特定的次序处理数据。
- 下载器中间件(Downloader Middlewares)：位于Scrapy引擎和下载器之间的框架，主要是处理Scrapy引擎与下载器之间的请求及响应。
- 爬虫中间件(SpiderMiddlewares)：介于Scrapy引擎和爬虫之间的框架，主要工作是处理蜘蛛的响应输入和请求输出。
- 调度中间件(Scheduler Middewares)
介于Scrapy引擎和调度之间的中间件，从Scrapy引擎发送到调度的请求和响应。
##### Scrapy运行流程大概如下：
引擎从调度器中取出一个链接(URL)用于接下来的抓取引擎把URL封装成一个请求(Request)传给下载器
下载器把资源下载下来，并封装成应答包(Response)
爬虫解析Response，解析出实体（Item）,则交给实体管道进行进一步的处理，解析出的是链接（URL）,则把URL交给调度器等待抓取。
##### Scrapy爬虫框架解析
1. Engine
    - 控制所有模块之间的数据流
    - 根据条件触发时间
    - 不需要用户修改
2. Downloader
    - 根据请求下载网页
    - 不需要用户修改
3. Schedualer
    - 对所有爬取请求进行调度管理
    - 不需要用户修改
4. Downloader Middleware
    - 目的：实施Engine、Schedualer和Downloader之间进行用户可配置的控制
    - 功能：修改、丢弃、请求或响应
    - 用户可以编写配置代码
5. Spider
    - 解析Downloader返回的响应（Response）
    - 产生爬取项（scraped item）
    - 产生额外的爬去请求（Request）
    - 需要用户编写配置代码
6. Item Pipelines
    - 以流水线方式处理Spider产生的爬取项。
    - 由一组操作顺序组成，类似流水线，每个操作是一个Item Pipeline类型。
    - 可能操作包括：清理、校验和查重爬取项中的HTML数据、将数据存储到数据库。
    - 需要用户编写配置代码
7. Spider Middleware
    - 目的：对请求和爬取项的再处理
    - 功能：修改、丢弃、新增请求或爬取项
    - 用户可以编写配置代码
##### Request库和Scrapy爬虫的比较
1. 相同点
    - 两者都可以进行页面请求和爬取，Python爬虫的两个重要技术路线。
    - 两者可用性都好，文档丰富，入门简单。
    - 两者都没有处理js、提交表单、应对验证码等功能（可扩展）。
2. requests vs. Scrapy

requests|Scrapy
---|---
页面级爬虫|网站级爬虫
功能库|框架
并发性考虑不足，性能较差|并发性好，性能较高
重点在于页面下载|重点在于爬虫结构
定制灵活|一般定制灵活，深度定制困难
上手十分简单|入门稍难
3.  选用哪个技术路线开发爬虫
    - 非常小的需求，requests库。
    - 不太晓得需求，Scrapy框架。
    - 定制程度很高的需求（不考虑规模），自搭框架，requests>Scrapy。
##### Scrapy爬虫的常用命令
1. Scrapy命令行
    
    - Scrapy是为持续运行设计的专业爬虫框架，提供操作的Scrapy命令行。
2. Scrapy命令行格式
    
    >\>Scrapy<command>[options]
3. Scrapy常用命令

命令|说明|格式
---|---|---
startproject|创建一个新工程|scrapy startproject <name> [dir]
genspider|创建一个爬虫|scrapy genspider [options] <name> <domain>
settings|获得爬虫配置信息|scarpy settings [options]
crawl|运行一个爬虫|scrapy crawl <spider>
list|列出工程中所有爬虫|scrapy list
shell|启动URL调试命令行|scrapy shell [url]
4. Scrapy爬虫的命令行逻辑
    -  命令行更容易自动化，适合脚本控制。
    -  本质上，Scrapy是给程序员使用的，功能比界面更重要。
#### 二、Scrapy爬虫基本使用
##### Scrapy爬虫的第一个实例
1. 生成的工程目录
    - python123demo:外层目录
    -  scrapy.cfg:部署Scrapy爬虫的配置文件
    -  python123demo/Scrapy框架的用户自定义Python代码
    -  \_\_init\_\_.py:初始化脚本
    -  items.py:Items代码模块（继承类）
    -  middlewares.py：Middlewares代码模块（继承类）
    -  pipelines.py:Pipelines代码模块（继承类）
    -  settings.py:Scrapy爬虫的配置文件
    -  spiders/:Spiders代码模块目录（继承类）
        - \_\_init\_\_.py:初始文件，无需修改
        - \_\_pycache\_\_:缓存目录，无需修改
2. 产生步骤
    - 步骤1：建立一个Scrapy爬虫工程
    - 步骤2：在工程中产生一个Scrapy爬虫
    - 步骤3：配置产生的spider
    - 步骤4：运行爬虫，获取网页
```
import scrapy
class DemoSpider(scrapy.Spider):
    name = "demo"#当前爬虫名字为demo
    #allowed_domains = ["python123.io"]#只能爬取这个域名以下的链接
    start_urls = ['https://python123.io/ws/demo.html']#爬取的初始界面
    def parse(self, response):
        fname = response.url.split('/')[-1]
        with open(fname, 'wb') as f:
            f.write(response.body)
        self.log('Saved file %s.' % name)
```
##### yield关键词的使用
1. yield关键字
    - yield=生成器
    - 生成器是一个不断产生值的函数
    - 包含yield语句的函数是一个生成器
    - 生成器每次产生一个值（yield语句），函数被冻结，被唤醒后在产生一个值。
2. 优势
    - 更节省存储空间
    - 响应更迅速
##### Scrapy爬虫的基本使用

1. Scrapy爬虫的使用步骤
    - 步骤1：创建一个工程和Spider模板
    - 步骤2：编写Spider
    - 步骤3：编写Item Pipeline
    - 步骤4：优化配置策略

2. Scrapy爬虫的数据类型
    - Request类
        - class scrapy.Request()
        - Request对象表示一个HTTP请求。
        - 由Spider生成，由Downloader执行。
            属性或方法|说明
            ---|---
            .url|Request对应的请求URL地址
            .method|对应的请求方法，‘GET’‘POST’等
            .headers|字典类型风格的请求头
            .body|请求内容主体，字符串类型
            .meta|用户添加的扩展信息，在Scrapy内部模块间传递信息使用
            .copy()|复制该请求

    - Response类
        - class scrapy.http.Response()
        - Response对象表示一个HTTP响应。
        - 由Downloader生成，由Spider处理。
          
            属性或方法|说明
            ---|---
            .url|Response对应的请求URL地址
            .status|HTTP状态码，默认是200
            .headers|Response对应的头部信息
            .body|Response对应的内容信息，字符串类型 .flags|一组标记
            .request|产生Response类型对应的Request对象
            .copy()|复制该响应
        
    - Item类
       - class scrapy.item.Item()
        - Item对象表示一个从HTML页面中提取的内容信息。
        - 由Spider生成，由Item Pipeline处理。
        - 类字典类型，可以按照字典类型操作
    
3. Scrapy爬虫提取信息的方法
    - BS4
    - lxml
    - re
    - Xpath Selector
    - CSS Selector
        - \<HTML>.css('a::attr(href)').extract()
##### 股票数据Scrapy爬虫
```
下面是stocks.py文件源代码

# -*- coding: utf-8 -*-
import scrapy
import re
 
class StocksSpider(scrapy.Spider):
    name = "stocks"
    start_urls = ['https://quote.eastmoney.com/stocklist.html']
 
    def parse(self, response):
        for href in response.css('a::attr(href)').extract():
            try:
                stock = re.findall(r"[s][hz]\d{6}", href)[0]
                url = 'https://gupiao.baidu.com/stock/' + stock + '.html'
                yield scrapy.Request(url, callback=self.parse_stock)
            except:
                continue
 
    def parse_stock(self, response):
        infoDict = {}
        stockInfo = response.css('.stock-bets')
        name = stockInfo.css('.bets-name').extract()[0]
        keyList = stockInfo.css('dt').extract()
        valueList = stockInfo.css('dd').extract()
        for i in range(len(keyList)):
            key = re.findall(r'>.*</dt>', keyList[i])[0][1:-5]
            try:
                val = re.findall(r'\d+\.?.*</dd>', valueList[i])[0][0:-5]
            except:
                val = '--'
            infoDict[key]=val
 
        infoDict.update(
            {'股票名称': re.findall('\s.*\(',name)[0].split()[0] + \
             re.findall('\>.*\<', name)[0][1:-1]})
        yield infoDict

下面是pipelines.py文件源代码：

# -*- coding: utf-8 -*-
 
# Define your item pipelines here
#
# Don't forget to add your pipeline to the ITEM_PIPELINES setting
# See: https://doc.scrapy.org/en/latest/topics/item-pipeline.html

class BaidustocksPipeline(object):
    def process_item(self, item, spider):
        return item
 
class BaidustocksInfoPipeline(object):
    def open_spider(self, spider):
        self.f = open('BaiduStockInfo.txt', 'w')
 
    def close_spider(self, spider):
        self.f.close()
 
    def process_item(self, item, spider):
        try:
            line = str(dict(item)) + '\n'
            self.f.write(line)
        except:
            pass
        return item

下面是settings.py文件中被修改的区域：

# Configure item pipelines
# See https://scrapy.readthedocs.org/en/latest/topics/item-pipeline.html
ITEM_PIPELINES = {
    'BaiduStocks.pipelines.BaidustocksInfoPipeline': 300,
}
```