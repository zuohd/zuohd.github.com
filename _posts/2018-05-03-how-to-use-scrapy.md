---
layout: post
title:  "Scrapy 爬虫使用小结"
date:   2018-05-03 20:35:48 +0800
categories: crawler
tags: [Scrapy]

---

## 创建一个新的Scrapy项目

``` shell
scrapy startproject mySpider
```

## 编写items.py文件定义model，即明确需要提取的数据

``` python
import scrapy

class MyspiderItem(scrapy.Item):
    positionName = scrapy.Field()
    positionLink = scrapy.Field()
    companyName = scrapy.Field()
    workLocation = scrapy.Field()
    salary = scrapy.Field()
    publishTime = scrapy.Field()

```

## 编写spiders/myspider.py爬虫文件，处理请求和响应，以及使用'yield item'提取数据
此处以爬取51job北京python工程师职位信息列表为例，我们只提取前20页信息：

``` python
import scrapy

from mySpider.items import MyspiderItem


class fiveOneJobSpider(scrapy.Spider):
    name = '51job'
    allowed_domains = ['51job.com']

    # just search Beijing python jobs
    baseURL = "https://search.51job.com/list/010000,000000,0000,00,9,99,python,2,%s.html"
    page = 1
    start_urls = [baseURL % (page)]

    def parse(self, response):
        node_list = response.xpath("//div[@id='resultList']/div[@class='el']")
        for node in node_list:
            item = MyspiderItem()
           
            item['positionName'] = node.xpath("./p/span/a/text()").extract_first(default="").strip()

            item['positionLink'] = node.xpath("./p/span/a/@href").extract_first(default="").strip()

            item['companyName'] = node.xpath("./span[1]/a/text()").extract_first(default="").strip()

            item['workLocation'] = node.xpath("./span[2]/text()").extract_first(default="").strip()

            item['salary'] = node.xpath("./span[3]/text()").extract_first(default="").strip()

            item['publishTime'] = node.xpath("./span[4]/text()").extract_first(default="").strip()

            yield item

            if self.page < 20:
                self.page += 1
                url = self.baseURL % (self.page)
                yield scrapy.Request(url, callback=self.parse)           

```
我们知道页数总是在动态变化，所以有了第二种方式依据其下一页的状态来爬数据：

``` python
    if len(response.xpath("//li[@class='bk'][2]/span/text()")) == 0:
        url = response.xpath("//li[@class='bk'][2]/a/@href").extract_first()
        yield scrapy.Request(url.split('?')[0], callback=self.parse)
```
## 编写管道文件pipelines.py,处理spider返回的item数据

这里我们可以持久化存储为json文件或数据库当中：
``` python
import json


class MyspiderPipeline(object):
    def __init__(self):
        self.f = open("51job.json", "w")

    def process_item(self, item, spider):
        content = json.dumps(dict(item), ensure_ascii=False) + ",\n"
        self.f.write(content)
        return item

    def close_spider(self, spider):
        self.f.close()
```

## 设置settings.py,启用管道以及其他相关配置

``` python
ITEM_PIPELINES = {
   'mySpider.pipelines.MyspiderPipeline': 300,
}
```

## 执行爬虫

``` shell
scrapy crawl 51job

```

