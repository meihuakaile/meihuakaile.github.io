---
title: 'linux常用命令'
date: "2018/04/20"
tags: [ubuntu]
categories: [linux]
copyright: true
---
### 防火墙
```
sudo ufw status   #查看防火墙状态
sudo ufw enable|disable   #开启/关闭防火墙 (默认设置是’disable’)
```
### SecureCRT上传下载
rz上传  sz下载
在SecureCRT中使用 rz -be 可以打开上传界面，把本地文件上传到服务器
sz 文件1 文件2    可下载多个文件 文件在本地的保存地址看：options — session options — X/Y/Zmodem。
rz 上传文件
总结自：http://blog.csdn.net/lioncode/article/details/7921525

### vim搜索
/字符串  第一个出现的字符串
?字符串 最后出现的字符串
n跳到下一个匹配  N跳到上一个匹配
### zcat
查看压缩文件。

### alias
主要是有些经常用又很长比较难记的命令。
输入alias可以看到已有的别名。
添加别名的两种办法：
（1）打开 .bashrc文件 vim .bashrc
\# some more ls aliases
alias ll='ls -alF'
添加别名。如alias d2='ssh data2.cn'就是只输入d2就相当与输入了ssh登陆“data2.cn”的命令。

（2）新建文件vim ~/.bash_aliases，然后添加：alias d2='ssh data2.cn'。最后退出输入source .bashrc就可以用了。

详细可参看：http://blog.csdn.net/a746742897/article/details/52228422

### 读取数据库中信息。
mysql -h 127.0.0.1 -u root -p XXXX -P 3306 -e "select * from table"  > /tmp/test/txt
-h 后面是主机； -u后面是用户名； -p后面是数据库名字 ； -P后面是端口号 ； -e就是mysql的select语句 ； 最后是重定向到一个文件。

参看：https://www.cnblogs.com/emanlee/p/4233602.html

### 查看端口号占用
```
netstat -apn | grep 8080
ps -ef | grep 8080   
```
pid是进程id（一般kill的时候用） ppid是父进程id  c是cpu占用率 其他看http://blog.csdn.net/lg632/article/details/52556139

### lsof查看文件被什么进程占用
lsof=list open files
`lsof 文件名`  查看某个文件被哪些进程在读写

### set -e/set -o pipefail
`set -e` 表示一旦脚本中有命令的返回值为非0，则脚本立即退出，后续命令不再执行;
`set -o pipefail` 表示在管道连接的命令序列中，只要有任何一个命令返回非0值，则整个管道返回非0值，即使最后一个命令返回0. 
### 解压纯 .gz文件
方法（1）：gunzip 文件       方法（2）：zcat输出 之后用 > 保存

### date
```
date -d today +"%Y-%m-%d"   输出2017-12-08  按照格式“%Y-%m-%d”输出今天的日期。
date +%Y-%m-%d --date="-1 day"  输出2017-12-07 按照格式“%Y-%m-%d”输出昨天的日期。 “-1”可以修改 “day”可以是month/year。
date -d "last sunday" +%Y-%m-%d   输出上周日的日期。
date -d "2018-03-12 1 days ago" +%Y-%m-%d  输出2018-03-11.此办法可以得到字符串2018-03-12的前一天
date --date="2018-03-12 1 days ago" +%Y-%m-%d  和上面那条效果一样。
```
注意：
（1）加号“+”跟要输出的格式   -s 可以设置时间；
（2）最后两条的 `ago`可以省略，是加的效果；
（3）最后两条的`days`可以换成hours等；
（4）最后两条的`2018-03-12`可以省略，把时间默认成当前时间；
参考：http://www.linuxidc.com/Linux/2016-11/137559.htm

### if [ $? -ne 0 ];then
$? -ne 0表示前面的命令的执行结果是不是0（0代表成功）   $?就是前面命令的执行结果。  
另外切记，shell里的`` 是执行里面内容的命令。 -ne是不等时true； -eq是相等时true。

### sort
sort 命令对 File 参数指定的文件中的行排序，并将结果写到标准输出。如果 File 参数指定多个文件，那么 sort 命令将这些文件连接起来，并当作一个文件进行排序。
`-r` 从大到小
`-n` 按照数字排序，否则可能会有`11 112 12`这样的结果 
`-k 2` 以第2列排序
`-t 字符` 以字符分割，和`-k`一起用，可以自定义分割并排序.
`-o` 指定输出文件，类似于重定向的效果
`-f` 会将小写字母都转换为大写字母来进行比较，亦即忽略大小写
`-c` 会检查文件是否已排好序，如果乱序，则输出第一个乱序的行的相关信息，最后返回1
`-C` 会检查文件是否已排好序，如果乱序，不输出内容，仅返回1
`-M` 会以月份来排序，比如JAN小于FEB等等
`-b` 会忽略每一行前面的所有空白部分，从第一个可见字符开始比较。

eg：
```
test.txt:

banana:30:5.5
apple:20:2.5
pear:90:2.3
orange:20:3.4
```
第一列表示水果类型，第二列表示水果数量，第三列表示水果价格.
以水果数量排序 ：`sort -n -r -t : -k 2 test.txt`

作用较大且复杂的是`-k`参数，参数有`[ FStart [ .CStart ] ] [ Modifier ] [ , [ FEnd [ .CEnd ] ][ Modifier ] ]`
这个语法格式可以被其中的逗号（“，”）分为两大部分，Start部分和End部分。如果不设定End部分，那么就认为End被设定为行尾.
Start部分也由三部分组成，其中的Modifier部分就是我们之前说过的类似n和r的选项部分。我们重点说说Start部分的FStart和C.Start。
C.Start也是可以省略的，省略的话就表示从本域的开头部分开始。之前例子中的-k 2和-k 3就是省略了C.Start的例子喽。
FStart.CStart，其中FStart就是表示使用的域，而CStart则表示在FStart域中从第几个字符开始算“排序首字符”。


总结自：https://www.cnblogs.com/51linux/archive/2012/05/23/2515299.html
### uniq
uniq命令可以去除排序过的文件中的重复行，因此uniq经常和sort合用。也就是说，为了使uniq起作用，所有的重复行必须是相邻的。

选项与参数：
-i   ：忽略大小写字符的不同；
-c  ：进行计数
-u  ：只显示唯一的行
13、例子sort+uniq
### 经典！计算ip出现次数例子

文件：
178.60.128.31 www.google.com.hk  
193.192.250.158 www.google.com  
210.242.125.35 adwords.google.com  
210.242.125.35 accounts.google.com.hk  
210.242.125.35 accounts.google.com  
210.242.125.35 accounts.l.google.com  
64.233.181.49 www.google.com  
212.188.10.167 www.google.com  
23.239.5.106 www.google.com  
64.233.168.41 www.google.com  
62.1.38.89 www.google.com  
62.1.38.89 chrome.google.com  
193.192.250.172 www.google.com  
212.188.10.241 www.google.com
命令：
cat test.txt|awk '{print $1}'|sort|uniq -c  
参考：http://james-lover.iteye.com/blog/2105795

### dir=$( cd $(dirname $0) ; pwd -P )
得到当前运行脚本实际物理地址，而非链接地址。
$0就是当前运行脚本；
`dirname`得到指定脚本所在的目录，在执行时相对的路径。
`pwd 的-P` 得到的实际物理地址，不是连接的地址；-L是链接地址，而非物理地址。
`basename` 是去除目录后剩下的名字.

### 1>/dev/null 2>&1
（1）1>/dev/null 首先表示标准输出重定向到空设备文件，也就是不输出任何信息到终端，说白了就是不显示任何信息。
（2）2>&1 接着，标准错误输出重定向等同于 标准输出，因为之前标准输出已经重定向到了空设备文件，所以标准错误输出也重定向到空设备文件。

### cat /test/* > test.txt
把test目录下的文件以行的方式追加到test.txt文件中。

### ssh转接端口
通过转接端口访问其他机器数据库
`ssh -L host1:port1:host2:port2 host3`命令必须在host1上执行，host3必须有sshd
-f 后台运行
-N 不开shell
-T 不分配tty
http://mingxinglai.com/cn/2015/09/connect-mysql-via-ssh-tunnel/

### 多行注释
使用:<<BLOCK 和 BLOCK。BLOCK是任意字符串。
更多参考：http://www.jb51.net/article/52377.htm
反引号就是1旁边的键。

### 报错：变量与空格[: too many arguments
```shell
ret="Peter Anne"
if [ $ret == "Peter Anne" ]; then
  echo "pass"
else
  echo "failed"
fi
```
这段脚本会报错`变量与空格[: too many arguments` 。原因：
`if [ $ret == "Peter Anne" ];`它的参数分别为` [，$ret， ==，"Peter Anne"，]`，一共5个参数。（`[`也是被当作参数，这就是为什么`[`一定要有空格的缘故。
如果正常5个参数，是没有问题的，但是问题出在了$ret变量里。
在Linux系统中的真实解析是，`if [ Peter Anne == "Peter Anne" ]`，参数则分为：`[，Peter，Anne， ==，"Peter Anne"，]`，一共6个参数，这时就会报上面的错。
解决办法是`if [ "${ret}" == "Peter Anne" ]`。

参考自：https://blog.csdn.net/qq_22520587/article/details/62455740

### eval command-line
其中`command－line`是在终端上键入的一条普通命令行。然而当在它前面放上eval时，其结果是shell在执行命令行之前扫描它两次。如：
```
pipe="|"
eval ls $pipe wc -l
```
shell第1次扫描命令行时，它替换出pipe的值｜，接着eval使它再次扫描命令行，这时shell把｜作为管道符号了。
如果变量中包含任何需要shell直接在命令行中看到的字符（不是替换的结果），就可以使用eval。命令行结束符`（` `；` `｜` `&` `）` ，I／o重定向符（< >）和引号就属于对shell具有特殊意义的符号，必须直接出现在命令行中。
eval其他用法参考：http://blog.51cto.com/363918/1341977

### grep
`grep -v`过滤掉某些的数据
`grep -C 5 foo file` 显示file文件中匹配foo字串那行以及上下5行
`grep -B 5 foo file` 显示foo及前5行
`grep -A 5 foo file` 显示foo及后5行

grep 参数：http://man.linuxde.net/grep

### $! 
`$!` Shell最后运行的后台Process的PID
wait PID || exit 1

### 递归查文件个数
`ls -lR ./order/ ./user/|grep "^-"|wc -l`
递归查普通文件个数
`-h` 以M格式看文件大小

### drwxrwxrwx
文件夹的所有者 所属组 其他人对这个文件夹的权限

### 逐行读文件
```bash
cat testdata | while read line
do
    echo $line
done
```

其他方法：https://www.cnblogs.com/DengGao/p/5935688.html
### 函数返回字符串
shell脚本的return只能返回数值类型，可是我们很多时候想返回字符串。在函数中直接用echo 字符串，在调用时就可以得到返回字符串。
```shell
#!/bin/sh
function getStr ()
{
    String="very good"
    echo $String
}
str=$(getStr)
echo $str
```
结果：`very good`

### 
常用参看：https://www.cnblogs.com/yu2000/p/4089011.html
shell数值比较：https://www.cnblogs.com/happyhotty/articles/2065412.html
read:https://blog.csdn.net/u012359618/article/details/52329346