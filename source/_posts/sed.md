---
title: 'sed'
date: "2018/04/20"
tags: [ubuntu]
categories: [ubuntu]
copyright: true
---
sed文本处理，可以配合正则使用。处理时，把当前处理的行存储在临时缓冲区中，称为“模式空间”（pattern space），接着用sed命令处理缓冲区中的内容，处理完成后，把缓冲区的内容送往屏幕。接着处理下一行，这样不断重复，直到文件末尾。文件内容并没有 改变，除非你使用重定向存储输出（除了-i命令）。
简化对文件的反复操作。

# 定界符
可以使用任意定界符，如/，：，|等。
定界符出现在内部时，用\转义。如把/bin替换成/bin/local：
```
sed 's/\/bin/\/bin\/local/'
```
# test.txt
以下的命令都基于文件 test.txt
```
Tom　　 2012-12-11 car 53000
John　　 2013-01-13 bike 41000
vivi 2013-01-18 car 42800
Tom　　 2013-01-20 car 32500
John　　 2013-01-28 bike 63500
```
# s 替换命令
把‘bike’替换成‘train’
```
sed 's/bike/train/' test.txt
输出：
Tom　　 2012-12-11 car 53000
John　　 2013-01-13 train 41000
vivi 2013-01-18 car 42800
Tom　　 2013-01-20 car 32500
John　　 2013-01-28 train 63500
```
# -n 和 p一起只打印发生替换的行。 p是输出的功能。
```
sed -n 's/bike/train/p' test.txt
输出：
John　　 2013-01-13 train 41000
John　　 2013-01-28 train 63500
```
# -i会直接编辑原文件并保存
```
sed -i 's/bike/train/g' test.txt 
cat test.txt 结果：
Tom　　 2012-12-11 car 53000
John　　 2013-01-13 train 41000
vivi    2013-01-18 car 42800
Tom　　 2013-01-20 car 32500
John　　 2013-01-28 train 63500
```
# g会替换每一行所有 Ng可以直接从匹配到的第N个开始替换。
```
sed 's/0/7/3g' test.txt
输出：
Tom　　 2012-12-11 car 53077
John　　 2013-01-13 train 41777
vivi 2013-01-18 car 42877
Tom　　 2013-01-27 car 32577
John　　 2013-01-28 train 63577
```
# d删除 Nd删除第N行  N，Md删除第N到M行（^是以...开头，$是以...结尾）
```
sed '/^vivi/d' test.txt #搜索以vivi开头的行并删除
输出：
Tom　　 2012-12-11 car 53000
John　　 2013-01-13 train 41000
Tom　　 2013-01-20 car 32500
John　　 2013-01-28 train 63500
sed '/500$/d' ./test.txt #搜索以500结尾的行并删除
输出：
Tom　　  2012-12-11      car     53000
John　　 2013-01-13      train    41000
vivi    2013-01-18      car     42800
```
# & 表示已匹配字符串
```
sed 's/car\|train/[&]/g' test.txt
输出：
Tom　　 2012-12-11 [car] 53000
John　　 2013-01-13 [train] 41000
vivi 2013-01-18 [car] 42800
Tom　　 2013-01-20 [car] 32500
John　　 2013-01-28 [train] 63500
#sed 's/\w\+/[&]/g' test.txt
sed 's/\w\+/[&]/g' test.txt
输出：
[Tom]　　 [2012]-[12]-[11] [car] [53000]
[John]　　 [2013]-[01]-[13] [train] [41000]
[vivi] [2013]-[01]-[18] [car] [42800]
[Tom]　　 [2013]-[01]-[20] [car] [32500]
[John]　　 [2013]-[01]-[28] [train] [63500]
```
# \1子串匹配
```
sed 's/2\([0-9]\)/\1/' test.txt    #\1是正则括号匹配的第一个
输出：
Tom　　 012-12-11 car 53000
John　　 013-01-13 train 41000
vivi 013-01-18 car 42800
Tom　　 013-01-20 car 32500
John　　 013-01-28 train 63500

sed 's/2\([0-9]\)\([0-9]\)/\2\1/' test.txt  #把匹配到的第一个和第二个位置交换
输出：
Tom　　  102-12-11      car     53000
John　　 103-01-13      train    41000
vivi    103-01-18      car     42800
Tom　　  103-01-20      car     32500
John　　 103-01-28      train    63500

sed 's/2\([0-9]\)1\([0-9]\)/2\21\1/' test.txt
输出：
Tom　　 2210-12-11 car 53000
John　　 2310-01-13 train 41000
vivi 2310-01-18 car 42800
Tom　　 2310-01-20 car 32500
John　　 2310-01-28 train 63500
```
# 引用shell变量
# sed表达式可以使用单引号，但是表达式中有引用字符串（即下面的$test）时就必须用双引号。
```
#!/bin/sh

test=hello
echo hello | sed "s/$test/HELLO/"
```
# 逗号 分割行范围
```
sed -n '2,/^Tom/p' test.txt   #输出第2行到以Tom开头的行（没有Tom就到文件结尾）
输出：
John　　 2013-01-13 train 41000
vivi    2013-01-18 car 42800
Tom　　 2013-01-20 car 32500

sed '/^John/,/^vivi/s/$/ add/' test.txt #从以John开头的行到以vivi开头的行 结尾加add（因为s/$/ add，s是替换，$是以...为结尾）
输出：
Tom　　 2012-12-11 car 53000
John　　 2013-01-13 train 41000 add
vivi    2013-01-18 car 42800 add
Tom　　 2013-01-20 car 32500
John　　 2013-01-28 train 63500 add

sed -e '1,3d' -e 's/car/train/' test.txt  #删除第1到3行，剩下的用car替换train。
输出：
Tom　　 2013-01-20 train 32500
John　　 2013-01-28 train 63500
```
# -e 同一行里执行多条命令
#（更高级的还可以用分号；分割多个命令）
```
sed -e '1,3d' -e 's/car/train/' test.txt
输出：
Tom　　 2013-01-20 train 32500
John　　 2013-01-28 train 63500
```
# w 写文件
```
sed -n '/^vivi/w test1.txt' test.txt   #把test.txt中以vivi开头的行写到test1.txt中
结果：
cat test1.txt
vivi 2013-01-18 car 42800
```
# r 读文件
```
sed '/^vivi/r test.txt' test1.txt     #匹配test1.txt中以vivi开头的行，把test.txt的内容跟在后面
输出：
vivi 2013-01-18 car 42800
Tom　　 2012-12-11 car 53000
John　　 2013-01-13 train 41000
vivi 2013-01-18 car 42800
Tom　　 2013-01-20 car 32500
John　　 2013-01-28 train 63500
```
# a 追加文件
```
sed '/^vivi/a\this is vivi' test.txt   #把‘this is vivi’追加到test.txt中以vivi开头的行后面
Tom　　 2012-12-11 car 53000
John　　 2013-01-13 train 41000
vivi    2013-01-18 car 42800
this is vivi
Tom　　 2013-01-20 car 32500
John　　 2013-01-28 train 63500
```
# i 追加到行前面
（如果要直接修改原文件，就写上-i）
```
sed '/^vivi/i this is vivi' test.txt
输出
Tom　　 2012-12-11 car 53000
John　　 2013-01-13 train 41000
this is vivi
vivi          2013-01-18 car 42800
Tom　　 2013-01-20 car 32500
John　　 2013-01-28 train 63500


sed -i '/^vivi/i this is vivi' test.txt  #没有输出，但是直接修改了test.txt文件
```
# n;  匹配到换行
```
sed '/vivi/{n; s/car/bike/}' test.txt   #匹配到vivi后换行用bike替换car
输出：
Tom　　 2012-12-11 car 53000
John　　 2013-01-13 train 41000
vivi 2013-01-18 car 42800
Tom　　 2013-01-20 bike 32500
John　　 2013-01-28 train 63500
```
# y一对一替换
```
sed '1,10y/abcr/ABCR/' test.txt   #所有的abcr字符都替换成对应大写
输出：
Tom　　 2012-12-11 CAR 53000
John　　 2013-01-13 tRAin 41000
vivi 2013-01-18 CAR 42800
Tom　　 2013-01-20 CAR 32500
John　　 2013-01-28 tRAin 63500
```
# q 退出
```
sed '2q' test.txt  #输出两行后退出
输出：
Tom　　 2012-12-11 car 53000
John　　 2013-01-13 train 41000
```
# 常用例子
```
grep oldString -rl /path | xargs sed -i "s/oldString/newString/g"    #替换一个目录下的所有文件中字符串oldString成newString
```

# 保存sed处理结果的方法
可以用-i命令（比较危险）或者>重定向到另一个文件或者用w命令