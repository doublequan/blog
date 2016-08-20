---

layout: post
title: 'Scrapy-redis'
date: '2016-8-20'
header-img: "img/post-bg-web.jpg"
tags:
     - scrapy
     - redis
author: 'Bill Quan'
---



## Introduction to Scrapy-Redis

The official document is [here](https://scrapy-redis.readthedocs.io/en/stable/index.html "scrapy-redis document").

一个非常好的scrapy-redis简单原理介绍：（转自知乎[here](https://www.zhihu.com/question/32302268 "scrapy-redis 和 scrapy 有什么区别")）

作者：杨恒

来源：知乎

著作权归作者所有

我刚刚接触scrapy的时候，也看过这个项目，奈何对scrapy本身就不怎么熟悉，所以当时怎么也想不明白，直到后来开始看scrapy 的源代码，才渐渐明白。这里提一下我的看法，水平有限，不敢保证完全正确，欢迎指正。
一、scrapy和scrapy-redis的主要区别在哪里？
个人认为，scrapy和scrapy-redis不应该讨论区别。
scrapy 是一个通用的爬虫框架，其功能比较完善，可以帮你迅速的写一个简单爬虫，并且跑起来。scrapy-redis是为了更方便地实现scrapy分布式爬取，而提供了一些以redis为基础的组件（注意，scrapy-redis只是一些组件，而不是一个完整的框架）。你可以这么认为，scrapy是一工厂，能够出产你要的spider。而scrapy-redis是其他厂商为了帮助scrapy工厂更好的实现某些功能而制造了一些设备，用于替换scrapy工厂的原设备。
所以要想跑分布式，先让scrapy工厂搭建起来，再用scrapy-redis设备去更换scrapy的某些设备。
那么这些scrapy-redis组件有什么突出特点呢？他们使用了redis数据库来替换scrapy原本使用的队列结构（deque），换了数据结构，那么相应的操作当然都要换啦，所以与队列相关的这些组件都做了更换。
二、scrapy-redis提供了哪些组件？
Scheduler、Dupefilter、Pipeline、Spider
提供了哪些组件具体见[darkrho/scrapy-redis · GitHub**](//link.zhihu.com/?target=https%3A//github.com/darkrho/scrapy-redis)。
三、为什么要提供这些组件？
这要从哪哪哪说起（喝口水，默默地望着远方......）
我们先从scrapy的“待爬队列”和“Scheduler”入手：
咱们玩过爬虫（什么玩过，是学习过，研究过，爱过也被虐过）的同学都多多少少有些了解，在爬虫爬取过程当中，有一个主要的数据结构是“待爬队列”，以及能够操作这个队列的调度器（也就是Scheduler啦）。scrapy官方文档对这二者的描述不多，基本上没提。
scrapy使用什么样的数据结构来存放待爬取的request呢？
其实没用高大上的数据结构，就是python自带的collection.deque，不过当然是改造过后的啦（后面所说到的deque均是指scrapy改造之后的队列，至于如何改造的，就去看代码吧）。
详见源代码[queuelib/queue.py at master · scrapy/queuelib · GitHub**](//link.zhihu.com/?target=https%3A//github.com/scrapy/queuelib/blob/master/queuelib/queue.py)
不过咱们用一用deque就会意识到，该怎么让两个以上的Spider共用这个deque呢？答案是，我水平不够，不知道。那分布式怎么实现呢，待爬队列都不能共享，还玩个泥巴呀。scrapy-redis提供了一个解决方法，把deque换成redis数据库，我们从同一个redis服务器存放要爬取的request，这样就能让多个spider去同一个数据库里读取，这样分布式的主要问题就解决了嘛。
那么问题又来了，我们换了redis来存放队列，哪scrapy就能直接分布式了么？（当初天真的我呀~
当然不能。我们接着往下说。scrapy中跟“待爬队列”直接相关的就是调度器“Scheduler”：[scrapy/scheduler.py at master · scrapy/scrapy · GitHub**](//link.zhihu.com/?target=https%3A//github.com/scrapy/scrapy/blob/master/scrapy/core/scheduler.py)，它负责对新的request进行入列操作（加入deque），取出下一个要爬取的request（从deque中取出）等操作。
在scrapy中，Scheduler并不是直接就把deque拿来就粗暴的使用了，而且提供了一个比较高级的组织方法，它把待爬队列按照优先级建立了一个字典结构，比如：

```python
{
	priority0:队列0
	priority1:队列2
	priority2:队列2
}
```

然后根据request中的priority属性，来决定该入哪个队列。而出列时，则按priority较小的优先出列。为了管理这个比较高级的队列字典，Scheduler需要提供一系列的方法。说这么多有什么意义呢？最主要的指导意义就是：你要是换了redis做队列，这个scrapy下的Scheduler就用不了，所以自己写一个吧。
于是就出现了scrapy-redis的专用scheduler：[scrapy-redis/scheduler.py at master · darkrho/scrapy-redis · GitHub**](//link.zhihu.com/?target=https%3A//github.com/darkrho/scrapy-redis/blob/master/scrapy_redis/scheduler.py)，其实可以对比一下看，操作什么的都差不太多。主要是操作的数据结构变了。


那么既然使用了redis做主要数据结构，能不能把其他使用自带数据结构关键功能模块也换掉呢？（关键部分使用自带数据结构简直有种没穿小内内的危机感，这是本人的想法~）
在我们爬取过程当中，还有一个重要的功能模块，就是request去重（已经发送过得请求就别再发啦，也要考虑一下服务器君的感受好伐）。
scrapy中是如何实现这个去重功能的呢？用集合~
scrapy中把已经发送的request指纹放入到一个集合中，把下一个request的指纹拿到集合中比对，如果该指纹存在于集合中，说明这个request发送过了，如果没有则继续操作。这个核心的判重功能是这样实现的：

```
def request_seen(self, request):
    #self.figerprints就是一个指纹集合
    fp = self.request_fingerprint(request)
    if fp in self.fingerprints:#这就是判重的核心操作。
        return True
    self.fingerprints.add(fp)
    ......
```

详见源代码：[scrapy/dupefilters.py at master · scrapy/scrapy · GitHub**](//link.zhihu.com/?target=https%3A//github.com/scrapy/scrapy/blob/master/scrapy/dupefilters.py)
为了分布式，把这个集合也换掉吧，换了redis，照样也得把去重类给换了。
于是就有了scrapy-redis的dupefilter:[scrapy-redis/dupefilter.py at master · darkrho/scrapy-redis · GitHub**](//link.zhihu.com/?target=https%3A//github.com/darkrho/scrapy-redis/blob/master/scrapy_redis/dupefilter.py)

那么依次类推，接下来的其他组件（Pipeline和Spider），我们也可以轻松的猜到，他们是为什么要被修改呢。对，都是因为redis，全怪redis，redis是罪魁祸首。（其实我不知道，我只是在凑字数而已）



### Run Scarpy-Redis in Docker

[scrapy redis example project in docker](https://github.com/rolando/scrapy-redis/tree/master/example-project)

上述example project可以运行，但是docker-compose.yml是v1版的，自行修改了v2版的如下：

```
version: '2'
services:
  redis:
    image: redis
    ports:
      - "6379:6379" # added port for external db provisioning
  scrapy_redis:
    build: 
      context: .
      dockerfile: scrapy_redis_dockerfile
    command: tail -f /code/t.log
    volumes:
      - ./scrapy_redis:/code
    depends_on:
      - redis
```

这里redis和scrapy_redis 两个service在同一个docker network下，所以不需要在scrapy_redis中申明link，但是scrapy_redis默认的redis服务器地址是localhost:6379，需在scrapy的setting.py中添加如下设定替换默认设定：

```python
# Specify the host and port to use when connecting to Redis (optional).
REDIS_HOST = 'redis'
REDIS_PORT = 6379
```

