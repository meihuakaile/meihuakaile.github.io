---
title: python 二进制、十六进制、ascii码互转
tags: ['python', '进制转换']
categories: ['cnn图片数据处理、显示']
copyright: true
---
1.bin(数字)十进制-》二进制，有0b，用replace('0b','')

  

    
    
    <span style="font-size:18px;">eg:>>> print bin(97).replace('0b','')
         1100001</span>

  
  
2.int(浮点型数字) float-》int  

    
    
    <span style="font-size:18px;">eg:>>> print int(12.3)
         12</span>

  
  
3.chr(数字a)a在0~255之间。int-》ascii码（即只有8位）  

    
    
    <span style="font-size:18px;">eg:>>> print chr(97)
         a</span>

  
  
4.ard(字符a)ascii码-》int。3的反向  

    
    
    <span style="font-size:18px;">eg:>>> print ord('a')
         97</span>

  
  
5.hex(数字a)十进制-》十六进制  

    
    
    <span style="font-size:18px;"> eg:>>> print hex(10)
         0xa</span>

  
  
6.binascii.b2a_hex(字符串)字符串-》十六进制  
7.binascii.a2b_hex(十六进制数)十六进制-》字符串。6的反向  

    
    
    <span style="font-size:18px;">eg:>>> import binascii as ba
         >>> print ba.b2a_hex('nihao')
         6e6968616f
         >>> import binascii as ba
         >>> b =  ba.b2a_hex('nihao')
         >>> a = ba.a2b_hex(b)
         >>> print b
         6e6968616f
         >>> print a
         nihao
    
    </span>

  
6,7我不常用，其他常用。  

