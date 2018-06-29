---
title: 'awk'
date: "2018/04/20"
tags: [ubuntu]
categories: [linux]
copyright: true
---

把文件逐行的读入，以空格（还可以指定别的字符）为默认分隔符将每行切片，切开的部分再进行各种分析处理。

参考：https://www.cnblogs.com/emanlee/p/3327576.html

# 使用方法
（1）awk 'pattern{action}' input-file(s)  
pattern匹配行，并对匹配做出action操作。{action}可省。
pattern 表示 AWK 在数据中查找的内容，而 action 是在找到匹配内容时所执行的一系列命令。
pattern常用的有：
- /正则表达式/：使用通配符的扩展集。
- 关系表达式：可以用下面运算符表中的关系运算符进行操作，可以是字符串或数字的比较，如$2>%1选择第二个字段比第一个字段长的行。
- 模式匹配表达式：用运算符~(匹配)和~!(不匹配)。
- 模式，模式：指定一个行的范围。该语法不能包括BEGIN和END模式。
- BEGIN：让用户指定在第一条输入记录被处理之前所发生的动作，通常可在这里设置全局变量。
- END：让用户在最后一条输入记录被读取之后发生的动作。

action用{}括起来。
例1：搜索/etc/passwd中有root的关键词的行
```
awk -F: '/root/' /etc/passwd
输出：
root:x:0:0:root:/root:/bin/bash
```
例2：搜索/etc/passwd中以‘r’开头的行，并输出匹配行的用户名和对应的shell（注意：正则表达式中，^出现在[]里时是非的意思，其他是以...开头的意思）
```
awk -F: '/^r/{print $1, $7}' /etc/passwd
输出：
root /bin/bash
rtkit /bin/false
```
在例2中pattern 是 ^r  ，用以匹配以 r 开头的行； action是 {print $1, $7}'   ，输出第1、7个域。-F: 是每行按照冒号分割。
合起来的意思就是：搜索/etc/passwd中以‘r’开头的行，行按   冒号： 分割，输出这些行第1、7个分割值.

# awk [-F ”field-separator“] 'commands' input-file(s)       
其中commands=BEGIN{} {} END{}
commands 对文件的真正处理操作，[-F 域分隔符]是可选的，有默认值。 input-file(s) 是待处理的文件。
在awk中，文件的每一行中，由域分隔符分开的每一项称为一个域。通常，在不指名-F域分隔符的情况下，默认的域分隔符是空格。commands的执行流程：
1. 先执行BEGING，
2. 然后读取文件，读入有/n换行符分割的一条记录，然后将记录（行）按指定的域分隔符划分域，填充域，
3. $0则表示所有域,$1表示第一个域,$n表示第n个域,随后开始执行模式所对应的动作action。
4. 接着开始读入第二条记录······直到所有的记录都读完，最后执行END操作。
例如：
```
awk -F: 'BEGIN {count=0;} {name[count] = $1; count++;} END{for (i=0; i<NR; i++) {print i, name[i]}}' /etc/passwd
部分输出：
0 root
1 daemon
2 bin
3 sys
4 sync
5 games
6 man
7 lp
8 mail
......
```
# BEGIN/END
BEGIN/END后面的第一个{}的内容是跟随begin和end的含义，BEGIN后的第二个{}是每行都会执行。如：
```
ls -l |awk 'BEGIN {size=0;print "[begin]size is ", size;} {size=size+$5;} END{print "[end]size is ", size}'
输出：
[begin]size is 0
[end]size is 0
```
```
ls -l |awk 'BEGIN {size=0;} {print "[begin]size is ", size;size=size+$5;} END{print "[end]size is ", size}
输出：
[begin]size is 0
[begin]size is 0
[begin]size is 4096
[begin]size is 8192
[begin]size is 12288
......
[end]size is 232404
```
## begin
BEGIN模块后紧跟着动作块，这个动作块在awk处理任何输入文件之前执行。所以它可以在没有任何输入的情况下进行测试。它通常用来改变内建变量的值，如OFS,RS和FS等，以及打印标题。
```
awk 'BEGIN{FS=":"; OFS="\t"; ORS="\n\n"}{print $1,$2,$3}' test 
#在处理输入文件test以前，域分隔符(FS)被设为冒号，输出文件分隔符(OFS)被设置为制表符，输出记录分隔符(ORS)被设置为两个换行符
awk 'BEGIN{print "TITLE TEST"}'     #只打印标题
```
# awk的语法
if判断语法、循环如while、for、break等都是借鉴与c语言。

# awk的内置变量
ARGC 命令行参数个数
ARGV 命令行参数排列
ENVIRON 支持队列中系统环境变量的使用
FILENAME awk浏览的文件名
FNR 浏览文件的记录数
FS 设置输入域分隔符，等价于命令行 -F选项
NF 浏览记录的域的个数
NR 已读的记录数（当前记录编号）
OFS 输出域分隔符
ORS 输出记录分隔符
RS 控制记录分隔符

# 多个分隔符
（1）width:720 height: 360
```
echo "width::720 height: 360" | awk -F '[ :]+' '{print $2, $4} '
输出：720 360
```
 命令用两个分隔符空格和分号，一次把720和360提出来。
‘+’的作用是遇到两个相邻的分隔符时视为一个，如height后面的冒号后面紧跟了一个空格，或者是width后面的冒号后面紧跟了另一个冒号。
弊端、约束：多个字符只能是单字符，无法处理多个字符串的情况。

（2）i have two apples and one banana
```
echo "i have two apples and one banana" | awk -F 'one|two' '{for(i = 1; i <= NF; i++) {print "i= ", $i}}' 
输出：
i=  i have 
i=   apples and 
i=   banana
```
命令用one、two分割。
使用的是正则表达式， -F后面的分割处可以使用正则。
# 常用字符串内置函数
（1）.split（a，b，c）以字符c分割字符串a并保存在数组b中
```
cat test.txt
输出：
Tom　　  2012-12-11      car     53000
John　　 2013-01-13      train    41000
vivi    2013-01-18      car     42800
Tom　　  2013-01-20      car     32500
John　　 2013-01-28      train    63500
```
计算cat.txt中每个人一月份的工资总和 （注意，语法for ind in b的ind是数组下标，而不是java中理解的数组元素） ：
```
cat test.txt|awk '{split($2,a,"-");if(a[2]==01){b[$1]+=$4}}END{for(i in b)print i,b[i]}'
输出：
vivi 42800
Tom　　 32500
John　　 104500
```
（2）.substr(a, b[, c])从b下标开始截取a，取长度c;c可选，没有默认到末尾。
（3）length每个记录的长度。如awk '{print length}' ./test.txt 是test中每行长度。
（4）sub（a，b[[，c]）目标字符串c中的所有a子串用b串替换，只替换每个匹配到的第一个。
（5）gsub（a，b[，c]）目标字符串c中的所有a子串用b串替换，替换所有匹配到的字符串。
（6）index（string， substring）substring在string中第一次被匹配到的下标。
```
awk '{ print index("mytest", "test") }' testfile
输出：
3
```
# 匹配操作符～
test.txt的内容如上
```
cat test.txt | awk '$3 ~/^ca/'    #匹配第三列以ca开头的行
输出：
Tom　　  2012-12-11      car     53000
vivi    2013-01-18      car     42800
Tom　　  2013-01-20      car     32500
```
# 比较表达式
conditional expression1 ? expression2: expression3，例如：
```
awk '{max = {$1 > $3} ? $1: $3: print max}' test。如果第一个域大于第三个域，$1就赋值给max，否则$3就赋值给max。
awk '$1 + $2 < 100' test。如果第一和第二个域相加大于100，则打印这些行。
awk '$1 > 5 && $2 < 10' test,如果第一个域大于5，并且第二个域小于10，则打印这些行。
```
# next
next 语句从输入文件中读取一行，然后执行awk脚本。  通过next在某条件时跳过该行，对下一行执行操作(类似于其他语言的continue)。如：
```
if ($4>3000) 
	next; 
else c4+=$4;
```
# 数组注意点
（1）语法for ind in b的ind是数组下标，而不是java中理解的数组元素。
（2）下标可以是字符串。可以上面计算工资的例子。
（3）delete(b[ind])。删除元素

# 多文件例子
```
# cat account
张三|000001
李四|000002
# cat cdr 
000001|10
000001|20
000002|30
000002|15
# awk -F | 'NR==FNR{a[$2]=$0;next}{print a[$1]"|"$2}' account cdr
张三|000001|10
张三|000001|20
李四|000002|30
李四|000002|15
```
多个文件时，NR=FNR为真时,判断当前读入的是第一个文件。

# 外部变量
（1）-v 参数 （不要使用内置函数的名字做变量名，会报错）
```
awk -v a="test" -v b="mytest" '{print a,b}' ./test.txt 
```
（2）awk运行在shell环境中。所以，写在awk中的命令，要先经过shell解析后，再交由awk来解释和执行.

  awk为了防止shell解析自己的命令 ，在命令外面用了单引号（最前面的commands）。

'{print "'$b'",$2, $4}'跟java调用mysql的非注入方式类似，在拼接sql语句。准确理解是三部分:`'{print "'    $b     '",$2, $4} '`
```
b="begin";echo "width::720 height: 360" | awk -F '[ :]+' '{print "'$b'",$2, $4} '
输出：
begin 720 360
```