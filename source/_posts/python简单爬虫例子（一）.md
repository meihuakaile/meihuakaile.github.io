---
title: python简单爬虫例子（一）
date: "2017/04/08"
tags: ['爬虫', 'python']
categories: ['python']
copyright: true
---
环境与上一篇一样windows，editplus，python-2.7.6（且我前面文章有介绍过配置过程）

另外介绍一个抓包工具fiddler，超级好用的，特别是在以后你需要爬一些很复杂网站时。（不要它是英文就接受不了，上手很快的）  

以前都是用beautifulsoup，现在想从头尝试用urllib2.

urllib2是python提供的抓取网页的组件。

1.最简单例子：

    
    
    import urllib2
    response = urllib2.urlopen("http://www.baidu.com/")
    html = response.read()
    print html

  
输出就是百度首页的编码。

  

2.下面是一个需要发送数据的爬虫简单例子。发送方式时get。（其实我自己也不知道为什么，在浏览器的网站栏里，网站的url中的中文是正常显示的，但是我把url
拷到editplus里之后就变了，好吧，拷到其他地方也是这样。。。不知道是为什么，开始还担心请求会不成功的，后来还是有数据的。看来是我的web开发学的不到位
，如果有知道原因的，请留言告诉我一声，虽然这件事和这个例子没什么关系。。。）

    
    
    #coding=utf-8
    import urllib
    import urllib2
    
    #http://dujia.com/pq/list_%E5%AE%9C%E6%98%8C?searchfrom=around&arounddep=%E6%AD%A6%E6%B1%89&tf=Ihot_01
    data = {}
    data['searchfrom'] = 'around'
    data['arounddep'] = '%E6%AD%A6%E6%B1%89'
    data['tf'] = 'Ihot_01'
    
    value = urllib.urlencode(data)
    print value
    url = 'http://dujia.com/pq/list_%E5%AE%9C%E6%98%8C' + '?' + value
    
    response = urllib2.urlopen(url)
    print response.read()

  
3.也是需要发送数据的爬虫例子。这个是post方式的。

    
    
    import urllib
    import urllib2
    
    #http://dujia.com/pq/list_%E5%AE%9C%E6%98%8C?searchfrom=around&arounddep=%E6%AD%A6%E6%B1%89&tf=Ihot_01
    data = {}
    data['searchfrom'] = 'around'
    data['arounddep'] = '%E6%AD%A6%E6%B1%89'
    data['tf'] = 'Ihot_01'
    
    value = urllib.urlencode(data)
    print value
    
    url = 'http://dujia.com/pq/list_%E5%AE%9C%E6%98%8C'
    response = urllib2.urlopen(url,value)
    print response.read()

  
貌似两个也没大差哈~  

