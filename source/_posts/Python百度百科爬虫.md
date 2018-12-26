---
title: Python百度百科爬虫
date: 2016-12-08 09:59:12
tags: [Python,爬虫]
subtitle: 最近想用搭建一个服务器，于是决定用python来实现。在经过将近一个月的python学习后，想要做点什么实际的东西。而python最有趣的就是可以很简洁的写一个很有意思的爬虫。
---

​	最近想用搭建一个服务器，于是决定用python来实现。在经过将近一个月的python学习后，想要做点什么实际的东西。而python最有趣的就是可以很简洁的写一个很有意思的爬虫.

## 一.框架

​	这是一个最为初级的爬虫框架。分为：主函数，URL管理器，html下载器，html解析器，html输出器。

### URl管理器

​	URL管理器的作用就是收集新的URL地址，并回收爬虫爬过的URL地址，有两个成员，一个new_urls，一个old_urls，都为list()类型。其他的几个函数都是很简单的判断与添加数据。

```python
class UrlManager(object):
    def __init__(self):
        self.new_urls = list()
        self.old_urls = list()

    def add_new_url(self, url):
        if url is None:
            return
        if url not in self.new_urls and url not in self.old_urls:
            self.new_urls.append(url)
            self.new_urls = list(set(self.new_urls))

    def add_new_urls(self, urls):
        if urls is None:
            return
        for url in urls:
            self.add_new_url(url)

    def has_new_url(self):
        return len(self.new_urls)

    def get_new_url(self):
        new_url = self.new_urls.pop(0)
        self.old_urls.append(new_url)
        return new_url
```

#### HTML下载器

​	HTML下载器需要用到一个库urllib2，这个模块的主要作用个就是下载目标URL的HTML文件调用` response = urllib2.urlopen(url)` 就可以将目标html装入response对象中，通过getcode()就可以知道是否下载成功。

```python
class HtmlDownloader(object):
    def download(self, url):
        if url is None:
            return

        response = urllib2.urlopen(url)

        if response.getcode() != 200:
            return None
        return response.read()

```

#### HTML解析器

​	这个模块是爬虫中最为关键的，它使用BeautifulSoup库和正则表达式来匹配收集想要继续爬取的URL网址和数据来保存到URL管理器中和data变量。

1.收集url。由于从分析python百度百科页面的HTML原代码知道`<a target="_blank" href="/view/20965.htm">自由软件</a>` 网址都是以/view/数字 的形式保存在a开头的代码中，其中网址保存在href属性中，那么就调用beautifulsoup的函数对其进行筛选。`links = soup.find_all('a', href=re.compile(r"/view/\d+\.htm"))`	函数会返回links这个字典类型的变量，接下来用一个循环提取href字段中的网址，并调用urlparse中的urljoin函数按照标准网址进行url补全，接着将其存入数组就行了，该数组最后会传给URL管理器进行管理。 	

2.收集数据。再次观察HTML源代码，发现数据都是这么储存的。标题为  `<dd class="lemmaWgt-lemmaTitle-title"><h1>Python</h1>` 那么还是调用soup的函数。而简介内容为在 `<div class="lemma-summary" label-module="lemmaSummary">`  的text中。用一个字典把收集的数据都保存下来。

```python
class HtmlPraser(object):
    def prase(self, page_url, html_cont):
        if page_url is None or html_cont is None:
            return
        soup = BeautifulSoup(html_cont, 'html.parser', from_encoding='utf-8')
        new_urls = self._get_new_urls(page_url, soup)
        new_data = self._get_new_data(page_url, soup)

        return new_urls, new_data

    def _get_new_urls(self, page_url, soup):
        new_urls = set()
        links = soup.find_all('a', href=re.compile(r"/view/\d+\.htm"))
        print len(links)
        for link in links:
            new_url = link['href']
            new_full_url = urlparse.urljoin(page_url, new_url)
            new_urls.add(new_full_url)
        return new_urls

    def _get_new_data(self, page_url, soup):
        res_data = {}
        # url
        res_data['url'] = page_url

        # <dd class="lemmaWgt-lemmaTitle-title">  <h1>Python</h1>

        title_node = soup.find('dd', class_="lemmaWgt-lemmaTitle-title").find('h1')
        res_data['title'] = title_node.get_text()

        # <div class="lemma-summary" label-module="lemmaSummary">
        summary_node = soup.find('div', class_="lemma-summary")
        res_data['summary'] = summary_node.get_text()

        return res_data
```

#### HTML输出器

​	输出器的功能就只有一个将获得的数据出输出为HTML格式。

```python
class HtmlOutputer(object):
    def __init__(self):
        self.datas = []

    def collect_data(self, data):
        if data is None:
            return
        self.datas.append(data)

    def output_html(self):
        fout = open('test.html', 'w')
        fout.write("<html>")
        fout.write("<head><meta http-equiv='content-type' content='text/html;charset=utf-8'></head>")
        fout.write("<body>")
        fout.write("<table>")
        for data in self.datas:
            fout.write("<tr>")
            fout.write("<td>%s</td>" % data['url'])
            fout.write("<td>%s</td>" % data['title'].encode('utf-8'))
            fout.write("<td>%s</td>" % data['summary'].encode('utf-8'))
        fout.write("</tr>")

        fout.write("</table>")
        fout.write("</body>")
        fout.write("</html>")
        fout.close()
```

#### 主函数

​	主函数先定义一个SpiderMain类，在构造函数中把之前的四个模块中的类都初始化，因为后面都要用到。主函数的思路是首先接受一个root_url。先将其存入url管理器，接着使用一个循环不断地从url管理器中提取未爬取的url，一开始当然只有一个url，调用HTML解析器将HTML下载器获得的字符串解析为数据和new_urls，new_urls就又存入URL管理器，数据就存入HTML输出器暂时储存。就这样不断的从URL管理器中提起未爬去的url，又从这个url中提取新的url和数据。不断地循环下去，知道到达预先设定的循环次数，就将数据输出为HTML文件，用浏览器打开即可。

```python
class SpiderMain(object):
    def __init__(self):
        self.urls = url_manager.UrlManager()
        self.praser = html_praser.HtmlPraser()
        self.downloader = html_downloader.HtmlDownloader()
        self.outputer = html_outputer.HtmlOutputer()

    def craw(self, root_url):
        count = 1
        # 添加新的url进入url管理器
        self.urls.add_new_url(root_url)
        # 循环检查url管理器是否还有未爬完的url
        while self.urls.has_new_url():
            try:
                new_url = self.urls.get_new_url()
                print 'craw %d :%s ' % (count, new_url)
                # 将目标html文件下载下来成字符串保存到html_cont变量中
                html_cont = self.downloader.download(new_url)

                # 获取新获得的多个url，和data
                new_urls, new_data = self.praser.prase(new_url, html_cont)
                # 将urls添加进入url管理器,数据添加进入数据管理器
                self.urls.add_new_urls(new_urls)
                self.outputer.collect_data(new_data)
                if count == item_num:
                    break
                count += 1
            except:
                print '爬取失败'

        self.outputer.output_html()


if __name__ == '__main__':
    print '欢迎使用百度百科超级爬虫!!!--------'
    item_name = raw_input("请输入你要搜索的字段：")
    item_num = input("请输入你要搜索的网站数目（最好不要超过1000）:")
    root_url = "http://baike.baidu.com/item/" + item_name
    obj_spider = SpiderMain()
    obj_spider.craw(root_url)

```

## 二.爬虫爬取效果

![爬虫截图2](/image/爬虫截图2.PNG)

![爬虫截图](/image/爬虫截图.PNG)

## 三.总结与反思

​	当把HTML文件打印出来后，会发现许多信息与Python的关联性很小。思考后觉得由于爬虫爬取的url集合实际上是一个树，如果使用普通的集合来保存的话就相当于一个深度优先遍历，优先遍历与主题关联性最小的条目。可以考虑用树形结构保存url，并使用广度优先遍历来爬取，就可以大大增加其关联性。