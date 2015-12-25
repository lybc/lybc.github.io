title: "Scrapy初体验（三） 爬爬爬！！！"
date: 2015-09-12 00:07:30
tags: Scrapy 
categories: Python
---
<!-- toc -->
在上一篇经过对目标网站的分析之后，我们现在开始进行代码的编写
<!--more-->

这时在命令行中执行`scrapy`命令，将得到得到如下提示
```bash
Scrapy 1.0.3 - no active project

Usage:
  scrapy <command> [options] [args]

Available commands:
  bench         Run quick benchmark test
  commands      
  fetch         Fetch a URL using the Scrapy downloader
  runspider     Run a self-contained spider (without creating a project)
  settings      Get settings values
  shell         Interactive scraping console
  startproject  Create new project
  version       Print Scrapy version
  view          Open URL in browser, as seen by Scrapy

  [ more ]      More commands available when run from project directory

Use "scrapy <command> -h" to see more info about a command

```
此时表示环境已部署成功，但暂未创建项目，我们可以运行scrapy自带的命令来创建项目
# 创建项目
运行命令创建项目
```bash
$ scrapy startproject movieSpider
```
完成后会在当前目录创建一个名为movieSpider的文件夹，目录结构如下
```bash
movieSpider/
    scrapy.cfg
    movieSpider/
        __init__.py
        items.py
        pipelines.py
        settings.py
        spiders/
            __init__.py
            ...
```
这些文件分别是:

- `scrapy.cfg`: 项目的配置文件
- `movieSpider/`: 该项目的python模块。之后您将在此加入代码。
- `movieSpider/items.py`: 项目中的item文件.可以在此定义需要抓取的字段。
- `movieSpider/pipelines.py`: 项目中的pipelines文件.用于数据抓取后的操作。
- `movieSpider/settings.py`: 项目的设置文件.
- `movieSpider/spiders/`: 放置spider代码的目录.

# 定义item
scrapy中的item是作为保存抓取到的数据的容器而存在，使用方法和Python的字典类似，使用它时，我们首先需要分析目标网站中有哪些我们需要的数据，如电影天堂中我们需要电影的名称，发布时间，详细信息和下载链接。于是我们可以这样定义item
```python
import scrapy

class MoviespiderItem(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    name = scrapy.Field()
    time = scrapy.Field()
    desc = scrapy.Field()
    link = scrapy.Field()
```
这样我们就可以得到一个item对象，其中包含name,time,desc,link四个字段，分别用来临时存储我们从网站中抓取到的信息

# 创建爬虫
项目创建好之后在`movieSpider/movieSpider/spider`下创建一个名为movie.py的文件，这个文件中将包括主要的程序抓取代码。
开发之前我们首先来研究一下官方文档，[scrapy官方文档](http://scrapy-chs.readthedocs.org/zh_CN/latest/intro/tutorial.html)
由官方文档可以得知在spider中有几个重要的参数
> name：定义spider名字的字符串(string)。spider的名字定义了Scrapy如何定位(并初始化)spider，所以其必须是唯一的。 不过您可以生成多个相同的spider实例(instance)，这没有任何限制。 name是spider最重要的属性，而且是必须的。


> start_urls：URL列表。当没有制定特定的URL时，spider将从该列表中开始进行爬取。 因此，第一个被获取到的页面的URL将是该列表之一。 后续的URL将会从获取到的数据中提取。

## 发起起始请求
scrapy提供了一个start_requests的函数来作为开始爬取的入口，当该函数没有被重写时，将会从你定义的start_urls列表中进行爬取，在开始编写爬虫时需要重写start_requests函数来创建一个爬虫的入口
```python
def start_requests(self):
    url = 'http://www.ygdy8.com/html/gndy/oumei/list_7_1.html'
    return [scrapy.Request(url, callback=self.sub_request)]
```
此方法定义了一个起始URL，并创建一个Request对象来对URL发起请求，爬虫经由该入口抓取到页面代码后，经过上一篇的分析，返回的响应中包含电影列表页的分页信息，将返回的响应发送到一个叫做sub_request的函数中进行进一步的处理。

## 处理第一次响应中的分页信息
这时我们在sub_request方法中接收到start_request方法传过来的响应，scrapy提取数据有一套被称为选择器（selector）的机制，他们通过特定的 XPath 或者 CSS 表达式来“选择” HTML文件中的某个部分。通过观察电影天堂页面的结构我们可以得知，在电影列表页面下有对它进行分页操作，该网站的分页操作是通过一个select元素中的option来实现，于是我们可以使用Python中的一个十分好用的解析HTML和xml的库`beautifulsoup4`来将我们需要的option数据筛选出来，得到每一页的链接后再逐一发起请求进一步获取信息
```python
def sub_request(self, response):
    '''获取分页方法，获取到该页面所有分页的链接并逐个发起请求'''
    result = response.selector.xpath('//*[@id="header"]/div/div[3]/div[3]/div[2]/div[2]/div[2]/div').extract()[0]
    bs = BeautifulSoup(result)
    options = bs.find_all('option')
    base_url = 'http://www.ygdy8.com/html/gndy/oumei/
    for option in options:
        url = base_url + option['value']
        yield scrapy.Request(url, callback=self.movie_title)
```
## 处理列表页中的元素信息
此时根据上一步获取到的URL爬虫已经进入到了电影列表页，如下图所示
![](http://7xicmj.com1.z0.glb.clouddn.com/dytt4.png)
到这一步时我们需要获取图中红框标注的URL，并再次发起请求进入我们真正想进入的页面，原理跟刚才一样，也是通过selector对象筛选出我们需要的元素
```python
def movie_title(self, response):
    '''找到页面的每一个电影列表链接并发起请求'''
    result = response.selector.xpath('//*[@id="header"]/div/div[3]/div[3]/div[2]/div[2]/div[2]/ul').extract()[0]
    bs = BeautifulSoup(result)
    links = bs.find_all('a',attrs={'class','ulink'})
    base_url = 'http://www.ygdy8.com'
    for link in links:
        url = base_url + link['href']
        # 筛选出的URL中无法区分，只好以字符串长度来区别
        if len(url) > 46:
            yield scrapy.Request(url, callback=self.parse)
```
## 处理数据
此时爬虫终于来到我们需要的数据所在的最终页面，在这个页面中，我们还是如以上一样使用选择器筛选出我们需要的数据，不同的是，这里我们定义的方法是重写spider对象中的一个叫做parse的方法，该方法被调用时，每个初始URL完成下载后生成的 Response 对象将会作为唯一的参数传递给该函数。 该方法负责解析返回的数据(response data)，提取数据(生成item)
```python
def parse(self, response):
    '''处理最终电影相关数据'''
    item = MoviespiderItem()
    item['name'] = response.selector.xpath('//*[@id="header"]/div/div[3]/div[3]/div[2]/div[2]/div[1]/h1/font/text()').extract()
    item['time'] = response.selector.xpath('//*[@id="header"]/div/div[3]/div[3]/div[2]/div[2]/div[2]/ul/text()').extract()
    desc = response.selector.xpath('//*[@id="Zoom"]').extract()[0]
    bs = BeautifulSoup(desc)
    link = bs.find_all('a')
    item['desc'] = desc
    item['link'] = link[0]['href']
    yield item
```

这时我们即完成了爬虫初步代码的编写，打开命令行执行
```bash
$ scrapy crawl name -o items.json
```
这个命令的意思是指将爬取到的数据保存在一个叫items.json的文件中，执行命令后就可以喝杯茶静静等待爬虫执行完毕了。
完整的代码放在[github](https://github.com/lybc/spider/tree/movieSpider)