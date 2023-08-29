## scrapy使用教程

### 框架简介

```md
Scrapy是用纯Python实现一个为了爬取网站数据、提取结构性数据而编写的应用框架，用途非常广泛。
Scrapy框架：用户只需要定制开发几个模块就可以轻松的实现一个爬虫，用来抓取网页内容以及各种图片，非常之方便。
Scrapy 使用了Twisted(其主要对手是Tornado)多线程异步网络框架来处理网络通讯，可以加快我们的下载速度，不用自己去实现异步框架，并且包含了各种中间件接口，可以灵活的完成各种需求。
```

![scrapy组件](./scrapy组件.png "scrapy组件")

```md
scrapy包含一下组件：
scrapy引擎：负责Spider、ItemPipeline、Downloader、Scheduler之间的通讯，信息、数据的传递。
scheduler调度器：它负责接收引擎发送过来的Requests请求，并按照一定方式整理排列，入队，当引擎需要时，归还给引擎。
Downloader下载器：它负责下载引擎发送的所有Requests请求，并将其获取到的Response对象交还给scrapy引擎，由引擎交给spider处理。
spider爬虫：它负责处理所有的response，从中分析提取数据，获取Item字段需要的数据，并将需要跟进的URL提交给引擎，再次进入调度器。
Item Pipeline管道：它负责处理在spider获取的Item，并进行后期处理（详细分析、过滤、存储等）的地方。
Downloader Middlewares下载器中间件：你可以当作是一个可以自定义扩展下载功能的组件
Spider Middlewares爬虫中间件：你可以理解为可自定义扩展引擎和爬虫之间通信的功能组件，比如进入spider的response和从spider出去的requests
```



### 数据处理流程

```md
scrapy的整个数据处理流程由引擎进行控制，通常的运转流程包括以下的步骤：
1. 引擎询问蜘蛛需要处理哪个网站，并让蜘蛛将第一个需要处理的URL交给它。
2. 引擎让调度器将需要处理的URL放入到队列中。
3. 引擎从调度器那获取下一个需要爬取的页面。
4. 调度器把下一个爬取的URL发送给引擎，引擎通过下载中间件将它发送到下载器。
5. 当网页被下载完成后，响应内容通过下载中间件被发送到引擎；如果下载失败了，引擎会通知调度器记录这个URL，待会儿重新下载。
6. 引擎收到下载器的响应并将它通过蜘蛛中间件发送到蜘蛛进行处理。
7. 蜘蛛处理响应并返回爬到的数据条目，此外还要将需要跟进的新的URL发送给引擎。
8. 引擎将抓取到的数据条目送入数据管道，把新的URL发送给调度器放入队列中。
上述操作中的第2-8步会一直重复执行，直到调度器中没有需要请求的URL，爬虫就停止工作。
```

### 安装scrapy

使用pip命令进行安装

```md
pip install scrapy
```





### scrapy爬虫的创建方法

1. 创建项目：

```python 
scrapy startproject <project name>
```

创建好的项目的目录结构

```md
demo
|-demo
|	|-spiders
|	|	|-__init__.PY
|	|-__init__.py
|	|-items.py
|	|-middlewares.py
|	|-pipelines.py
|	|-settings.py
|-scrapy.cfg
```



2. 创建spider程序

```md
1. 进入到demo目录下
2. 使用 scrpay genspider <spider name> <domain> 命令创建spider程序
spide name为爬虫名称
domain为允许爬虫爬取域的范围
```



3. 用pycharm打开，设置好项目的虚拟环境

```md
windows：
找到file -> settingS -> project -> python Interpreter,设置好虚拟环境，最好放到项目目录下
```



4. 确定爬取目标，编辑items.py文件

```md
Item定义结构化数据字段，用来保存爬取到的数据，有点像python中的dict，但是提供了一些额外的保护来减少错误。
可以通过创建一个scrapy.Item类，并且定义类型为Scrapy.Field 的类属性来定义一个Item（可以理解成类似于ORM的映射关系）。

import scrapy

class MovieItem(scrapy.Item):
	name = scrapy.Field()
	rank = scrapy.Field()
	subject = scrapy.Field()
```



5. 编辑爬虫文件

   爬虫文件有两个功能，一个是在响应中解析出新的URL，并将其返回给引擎；另外一个是解析出需要保存的数据。

   ```python
   import scrapy
   from scrapy import Selector, Request
   from spider0713.items import MovieItem   # 引用自定义的Item类
   
   # 必须继承自 scrapy.Spider 类
   class DoubanSpider(scrapy.Spider)：
   	# 爬虫名
   	name = "douban"
   	# 允许爬取的域名
   	allowed_domains = ["movie.douban.com"]
   	# 开始爬取的URL
   	start_urls = ['https://movie.douban.com/top250']
   	
   	# 使用start_requests方法给引擎提供待爬取页面的URL
     def start_requests(self):
     	for page in range(10):
     		url = f"https://movie.douban.com/top250?start={page * 25}&filter="
         # 可以在此处增加代理
         meta = {'proxy':'https://127.0.0.1:7890'}
     		yield Request(url, meta=meta)
   	
     def parse(self, response, **kwargs):
     	# scrapy 自带的解析选择器  scrapy.Selector
     	# sel = Selector(response)
     	# list_items = sel.css(".grid_view > li")
     	# for list_item in list_items:
     	#     movie_item = MovieItem()
       #     movie_item["title"] = list_item.css("span.title::text").extract_first()
       #     movie_item["rank"] = list_item.css("span.rating_num::text").extract_first()
       #     movie_item["subject"] = list_item.css("span.inq::text").extract_first()
       #     yield movie_item
       
       # 使用xpath解析所需要保存的数据
       html = etree.HTML(response.text)
       list_items = html.xpath("//ol[@class='grid_view']/li")
     	for list_item in list_items:
     		movie_item = MovieItem()
     		movie_item['title'] = list_item.xpath('.//span[@class="title"]/text()')[0]
     		movie_item['rank'] = list_item.xpath('.//span[@class="rating_num"]/text()')[0]
     		if list_item.xpath('.//span[@class="inq"]/text()'):
     			movie_item['subject'] = list_item.xpath('.//span[@class="inq"]/text()')[0]
     			yield movie_item
   		
   		# 构造Request对象，并返回给引擎
   		hrefs_list = html.xpath("//div[@class='paginator']/a/@href")
   		for href in hrefs_list:
   			url = response.urljoin(href)
   			# print(url)
   			yield Request(url=url)	
   ```

   

6. settings设置爬虫爬取的一些参数

```
USER_AGENT 设置请求头中的user-agent，将程序伪装成一个浏览器
ROBOTSTXT_OBEY 是否遵守网站爬虫协议
CONCURRENT_REQUESTS 支持多少个并发请求
DOWNLOAD_DELAY 设置下载延迟
RANDOMIZE_DOWNLOAD_DELAY 随即设置下载延迟
```



7. 爬虫运行

```
scrapy crawl douban -o douban.csv    (-o 输出文件，默认支持csv、json、xml)
如果不想看到日志 可以加 --nolog
scrapy crawl douban --nolog -o douban.csv
```



8. 将依赖项清单导出为 requirements.txt 文件

```
查看安装的第三方库方法：
  1. pip list
  2. pip freeze

将依赖项清单导出为 requirements.txt 文件
	pip freeze > requirements.txt

在其他设备安装依赖项清单
	pip install -r requirements.txt
```



### 将数据保存到Excel

修改pipelines.py文件

1. 创建初始化方法，生成excel工作簿，设置好sheet名称和字段名称

```python
class Spider0713Pipeline(object):
    def __init__(self):
      # wb 工作簿需要绑定self，因为在爬虫关闭前需要调用工作簿保存数据，如果不绑定self，在关闭爬虫前就找不到wb工作簿了。
        self.wb = openpyxl.Workbook()
        self.ws = self.wb.active
        self.ws.title = "douban movie top250"
        self.ws.append(('title', 'rank', 'subject'))
```

2. 在爬虫开始前执行的函数，open_spider，爬虫运行只执行一次

```python
		def open_spider(self, spider):
				pass
```

3. 在爬虫关闭前，将写入excel文件保存到指定文件

```python
    def close_spider(self, spider):  # 只被调用一次
        self.wb.save("电影数据.xlsx")
```

4. 将爬虫获取到的数据写入Excel文件

```python
    def process_item(self, item, spider):
    		# 使用 get 方法，如果字段为空，可以赋值为默认值 ''
        title = item.get("title", '')
        rank = item.get('rank', '')
        subject = item.get('subject', '')
        self.ws.append((title, rank, subject))
        # 返回 item 对象，框架仍可以获得 item 对象
        return item
```

4. 放开管道参数

```
放开 settings.py 中的管道参数
管道：可以写多个管道，数字越小，越先执行
ITEM_PIPELINES = {
   'spider0713.pipelines.Spider0713Pipeline': 300,
}
```

注释：以上__init__、open_spider、close_spider 等方法被创建，但是我们并不会自己去调用，而是供框架去调用，这样的方法被称为“钩子函数”，或者“回调函数(fallback函数)”。



### 将数据保存到MYSQL数据库

1. 首先创建好数据库，创建数据表

```sql
use spider;

drop table if exists 	`tb_top_movie`;

create table `tb_top_movie` (
		`mov_id` int unsigned auto_increment comment "编号" ,
		`title` varchar(50) not null comment "标题",
		`rating` decimal(3, 1) not null comment "评分",
		`subject` varchar(200) default '' comment "主题",
		PRIMARY key (`mov_id`)
) engine=innodb comment="TOP电影表"
```

2. 在pipelines.py中重新创建一个DBpipeline的类，连接数据库，插入数据

```python
class DBPipeline(object):
    def __init__(self):
        host = 'your host'
        port = your port
        user = 'your name'
        password = 'your password'
        database = 'database name'
        charset = 'utf8mb4'
        # 连接数据库
        self.conn = pymysql.connect(host=host, port=port,
                                    user=user, password=password,
                                    database=database, charset=charset)
        # 创建游标
        self.cursor = self.conn.cursor()
	
    def close_spider(self, spider):
      	# 提交数据库写入内容
        self.conn.commit()
        # 关闭数据库连接
        self.conn.close()

    def process_item(self, item, spider):
        title = item.get("title", '')
        rank = item.get('rank', '')
        subject = item.get('subject', '')
        # 向数据库插入数据
        self.cursor.execute(
            'insert into tb_top_movie (title, rating, subject) values(%s, %s, %s)',
            (title, rank, subject)
        )
        return item
```

3. 将数据批量插入数据库（批处理）

```python
class DBPipeline(object):
    def __init__(self):
        host = 'your host'
        port = your port
        user = 'your name'
        password = 'your password'
        database = 'database name'
        charset = 'utf8mb4'
        # 连接数据库
        self.conn = pymysql.connect(host=host, port=port,
                                    user=user, password=password,
                                    database=database, charset=charset)
        # 创建游标
        self.cursor = self.conn.cursor()
        self.data = []
	
    def close_spider(self, spider):
      	if self.data > 0:
            self.cursor.executemany(
                  'insert into tb_top_movie (title, rating, subject) values(%s, %s, %s)',
                  self.data
              )
        		self.conn.commit()
        # 关闭数据库连接
        self.conn.close()

    def process_item(self, item, spider):
        title = item.get("title", '')
        rank = item.get('rank', '')
        subject = item.get('subject', '')
        self.data.append((title, rank, subject))
        # 向数据库批量插入数据
        if len(self.data == 100):
            self.cursor.executemany(
                'insert into tb_top_movie (title, rating, subject) values(%s, %s, %s)',
                self.data
            )
            self.cursor.commit()
            self.data.clear()
        return item
      
    # 可以将写入数据库的部分单独抽取出来，写成一个方法
    #def _write_to_db(self):
    #    self.cursor.executemany(
    #        'insert into tb_top_movie (title, rating, subject) values(%s, %s, %s)',
    #        self.data
    #    )
    #    self.conn.commit()
```

### 下载中间件的使用

我们主要会编辑下载中间件（主要是拦截请求），很少会编辑爬虫中间件

在请求之前添加代理、cookie,在Middlewares.DownloadMiddleware.ProcessRequests中添加

```python 
 # 获取cookies
  def get_cookies_dict():
    cookies_str = ''
    cookies_dict = {}
    for item in cookies_str.split('; '):
        key, value = item.split("=", maxsplit=1)
        cookies_dict[key] = value
    return cookies_dict


 COOKIES_DICT = get_cookies_dict()
  
  
  def process_request(self, request, spider):
        # Called for each request that goes through the downloader
        # middleware.

        # Must either:
        # - return None: continue processing this request
        # - or return a Response object
        # - or return a Request object
        # - or raise IgnoreRequest: process_exception() methods of
        #   installed downloader middleware will be called
        # 添加代理
        request.cookies = COOKIES_DICT # 给请求添加cookie
        # request.meta = {'proxy':'https:127.0.0.1:7890'}  # 给请求添加代理
        return None
```

中间件需要配置才能生效

```python 
# 值越小，先执行
DOWNLOADER_MIDDLEWARES = {
   'spider0713.middlewares.Spider0713DownloaderMiddleware': 543,
}
```



### 解析更多页面数据

需要将获取的新页面的URL提交给引擎，并且回调新页面的解析函数，并将已经解析的数据传给被回调的函数。

```python 
    def parse(self, response, **kwargs):
        html = etree.HTML(response.text)
        list_items = html.xpath("//ol[@class='grid_view']/li")
        for list_item in list_items:
            movie_item = MovieItem()
            movie_item['title'] = list_item.xpath('.//span[@class="title"]/text()')[0]
            movie_item['rank'] = list_item.xpath('.//span[@class="rating_num"]/text()')[0]
            detail_url = list_item.xpath(".//div[@class='hd']/a/@href")[0]
            if list_item.xpath('.//span[@class="inq"]/text()'):
                movie_item['subject'] = list_item.xpath('.//span[@class="inq"]/text()')[0]
            # print(detail_url)
            # 将新获取的URL提交给引擎，并且回调新页面的解析函数，并将已经解析的数据传给被回调的函数。
            # 传入已解析的数据使用 cb_kwargs 参数
            yield Request(url=detail_url, callback=self.parse_detail, cb_kwargs={'item': movie_item})

    # 解析新获取的页面，并添加到 item 对象中        
    def parse_detail(self, response, **kwargs):
        html = etree.HTML(response.text)
        movie_item = kwargs['item']
        movie_item['duration'] = html.xpath('//span[@property="v:runtime"]/@content')[0]
        movie_item['intro'] = html.xpath('//span[@property="v:summary"]/text()')[0]
        yield movie_item
```

