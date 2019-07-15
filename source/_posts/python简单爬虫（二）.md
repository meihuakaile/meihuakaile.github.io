---
title: python简单爬虫（二）
date: "2017/04/08"
tags: ['python', '爬虫']
categories: ['python']
copyright: true
---
首先必须要会用python中处理异常的语法。看了一个很好，字体感觉很舒服（
![敲打](http://static.blog.csdn.net/xheditor/xheditor_emot/default/knock.gif)
原谅我的挑剔）且很详细，python异常： [ 点击打开链接
](http://www.cnblogs.com/dkblog/archive/2011/06/24/2089026.html)

二者环境仍和上篇一样，版本2.X

最后虽然上面那个异常处理讲的很清楚，但是我们这里主要用的两个异常没有具体说，我这里再提一下：URLError、HTTPError。URLError是HTTP
Error的基类，HTTPError的错误中会返回相应的code，如我们最常见的404，其他的可以自己去查。  

1.原本平时访问网站时都会出现错误，请求不到等等，更遑论我们这是爬虫了（且可能是漏洞百出）。且网站是被人的，说不定人家什么时候就把文件的存放位置改了呢，是不
是，仔细想想，状况百出呀，然你总不能做电脑旁边一直看着你的程序吧。。。这个时候就需要try来解决了。

首先来个小栗子：

    
    
    import urllib2
    
    url = 'http://bbs.csdn.net/WhereAreYou'
    req = urllib2.Request(url)
    try:
    	response = urllib2.urlopen(req)
    except urllib2.URLError, e:
    	print e.reason
    	print e.code
    #print response.read()

你执行这个栗子，结果是：

\---------- Python ----------  
Forbidden  
403  
  
输出完成 (耗时 0 秒) - 正常终止

然大家千万不要被他骗了，你把那个url地址在浏览器中尝试，你会发现csdn给你的是404错误。403错误是csdn服务器禁止了你的请求，为毛咧？因为他发现了
你不是正常访问。解决方法是让它以为你是正常访问，这里假装我们是chrome浏览器，在req里添加头，也就是它：

    
    
    req.add_header('User-Agent','Chrome')
    

现在再执行就是404（找不到你要的文件）

这是个HTTPError的栗子，如果把url换成“http://wwwbalibali.com/”就成了URLError（那就不能有code）里的了。我的理
解是如果服务器存在一般都是HTTPError，否则就是除了HTTPError的URLError了。  

2.下面是一个可以体现URLError是HTTPError父类的栗子，以及异常处理的语法，一旦捕捉到异常即跳出：

    
    
    #coding=utf-8
    import urllib
    import urllib2
    
    #url = 'http://www.balabala.com/'
    url = 'http://bbs.csdn.net/WhereAreYou'
    req = urllib2.Request(url)
    req.add_header('User-Agent','Chrome')
    try:
    	request = urllib2.urlopen(req)
    except urllib2.HTTPError,e:
    	print 'the server can not fullfill our request'
    	print 'the return code is: {0}'.format(e.code)
    except urllib2.URLError,e:
    	print 'we can not catch the server'
    	print 'the reason is: {0}'.format(e.reason)
    else :
    	print 'success!'

  
3.这是一个和上面处理结果一样的栗子，方式略不同

    
    
    import urllib2
    
    url = 'http://www.baibai.com/'
    #url = 'http://bbs.csdn.net/WhereAreYou'
    req = urllib2.Request(url)
    req.add_header('User-Agent','Chrome')
    try:
    	request = urllib2.urlopen(req)
    except urllib2.URLError, e:
    	if hasattr(e,'code'):
    		print 'the server can not fullfill our request'
    		print 'the return code is: {0}'.format(e.code)
    	elif hasattr(e,'reason'):
    		print 'we can not catch the server'
    		print 'the reason is: {0}'.format(e.reason)
    	else :
    		print 'success!'

其实就我自身小经验觉得，很多地方解决可能出现的且你知道什么样的异常选择if-elif处理也不失为一个好办法。举个栗子吧，如果你在分析一个页面的成分时，可能将
来你分析的那个节点没有了，这就可以轻易的停止你的程序，而这时如果用try解决明显很累赘。所以，你懂得。  

