title: "Scrapy初体验（一） 环境部署"
date: 2015-09-10 13:07:04
tags: Scrapy 
categories: Python
---
<!-- toc -->
系统选择centOs 7，Scrapy是一个为了爬取网站数据，提取结构性数据而编写的应用框架。 可以应用在包括数据挖掘，信息处理或存储历史数据等一系列的程序中。

其最初是为了 页面抓取 (更确切来说, 网络抓取 )所设计的， 也可以应用在获取API所返回的数据(例如 Amazon Associates Web Services ) 或者通用的网络爬虫。
<!--more-->
Linux发行版都自带Python环境，Scrapy官方推荐使用pip安装Scrapy，因此首先需要安装pip.
去github下载pip最新安装包。[pip install](https://github.com/pypa/pip)
目前版本是7.1.2下载完成得到一个`pip-7.1.2.tar.gz`的压缩包，然后执行命令解压缩
```bash
$ tar zvxf pip-7.1.2.tar.gz
```
进入解压好的pip-7.1.2目录，找到setup.py并安装执行
```bash
$ sudo python setup.py install
```
执行完成后就可以使用pip命令了。
然后使用pip命令安装Scrapy
```bash
$ sudo pip install Scrapy
```
安装过程中会出现一个报错：
```bash
编译中断。
    error: command 'gcc' failed with exit status 1
```
解决办法是执行
```bash
$ yum install gcc python-devel
```
安装完成后再次执行以上`pip install Scrapy`命令等待安装完成，直到终端出现如下文字提示，代表安装完成，即可使用Scrapy抓取数据了。
```bash
Installing collected packages: Twisted, characteristic, pyasn1-modules, service-identity, Scrapy
  Running setup.py install for Twisted
  Running setup.py install for pyasn1-modules
Successfully installed Scrapy-1.0.3 Twisted-15.4.0 characteristic-14.3.0 pyasn1-modules-0.0.7 service-identity-14.0.0

```

