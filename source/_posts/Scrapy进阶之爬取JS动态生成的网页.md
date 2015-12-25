title: "Scrapy进阶 爬取JS动态生成的网页"
date: 2015-12-25 11:28:30
tags: Scrapy 
categories: Python
---
这篇文章来源于在抓取[vsco](http://vsco.co)时遇到的问题，以常规方法解析页面时，页面返回的内容只有网站的整体框架，没有页面具体信息，经过分析发现网站是采用ajax获取后台图片信息，并且对图片做了处理生成了大图和缩略图，为了能继续爬取vsco的图片，所以研究了一下Scrapy提供的图片管道以及调用浏览器执行JS
<!-- more -->

此次为了方便牺牲了一些性能采用了利用selenium调用实体浏览器的方式来加载页面，当Scrapy通过selenium调用浏览器来发起请求时，Scrapy会等待浏览器解析完所有页面才认为请求结束，并能抓取到完整的页面。

## 安装selenium
```bash
pip install selenium
```

## 下载[chrome驱动包](http://chromedriver.storage.googleapis.com/index.html?path=2.7/)
下载后放入python环境目录下的Script文件夹中，即可在代码中直接调用，本机使用chrome，也可以使用Firefox，原理相同将驱动换成Firefox的驱动即可

完成后即可以开始编写爬虫代码，项目中使用selenium的代码如下：
```python
from selenium import webdriver

brower = webdriver.Chrome() # 通过下载的chromeDriver文件启动Chrome浏览器
brower.get('http://www.baidu.com') # 将会等待页面所有JS加载完毕
page_sourece = brower.page_source # 页面加载完成后获取页面HTML代码

```

## 图片管道
Scrapy实现了图片管道，使用自带的图片管道即可下载网站图片，当然也可以实现自定义图片管道来定制一些特殊的功能，Scrapy中推荐使用[Pillow](http://pillow.readthedocs.org/en/latest/installation.html)包来处理图片，生成缩略图并将图片归一成JPEG/RGB格式，下载安装好pillow后在setting.py文件中配置打开图片管道和文件存储路径。
```python
    ITEM_PIPELINES = {'scrapy.contrib.pipeline.images.ImagesPipeline': 1}
    IMAGES_STORE = 'D:\images'  # 系统中需要存放图片的文件夹路径
```

启动爬虫后会弹出浏览器界面请求目标网站，等待页面加载完成后重复请求，效率非常慢，以后再有类似需求可以选择调用无界面的浏览器引擎来加载页面，使用中间件的形式执行网页上的JS，或使用一些开源的JS引擎。

完整的爬虫代码：
```python
import scrapy
from scrapy.selector import Selector
from selenium import webdriver
import time
from vcsoGrid.items import VcsogridItem

class ImageSpider(scrapy.spiders.Spider):
	name = 'vsco'

	def __init__(self):
		self.browser = webdriver.Chrome()


	def start_requests(self):
		base_url = 'http://grid.vsco.co/grid/'
		for i in range(1, 4081):
			url = base_url + str(i)
			yield scrapy.Request(url=url, callback=self.parse)


	def parse(self, response):
		self.browser.get(response.url)
		time.sleep(3)
		page = self.browser.page_source
		imgs = Selector(text=page).select('//img/@src').extract()
		item = VcsogridItem()

		for img in imgs:
			if '.jpg?' in img:
				src = img.split("?")
				item['image_urls'] = ['http:' + str(src[0])]
				yield item

	def __del__(self):
		self.browser.close()
```

代码开源在[github](https://github.com/lybc/ScrapySpider)