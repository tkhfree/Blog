---
title: Linux的shell脚本
tags:
  - Linux
comments: true
toc: true
mathjax: true
abbrlink: aa9b661c
date: 2020-05-22 11:15:37
urlname:
categories:
thumbnail:
---

shell脚本的一些方法，想到什么命令就记什么命令了。

### set

```shell
set -u -x -e -o pipefail
开局使用
-u是使用未定义的变量时报错并终止。
-x是先打印脚本里执行的命令再执行
-e只要遇到错误就终止执行 +e关闭这个功能
-o pipefail是针对管线的，因为对于管线命令来说，只要管线最后输出exit 0/执行正确，则-e -u等就不会终止执行，使用这个可以对管线每个命令进行判断
```

还有另一种针对单个命令执行判断的方法

```shell
command || exit 1
```

### ||与&& 

|| 与 &&的区别

```shell
command1 || command2
执行完command1不成功才执行command2

command1 && command2 
执行完command1成功之后执行command2
```

```shel
` `之内的内容是需要执行的命令，取执行命令的结果
```

```shell
grep -c ^head <file>
输出file文件head开头的字符串个数
```

### 运算符

```shell
取变量名用$,像c语言的指针概念。
$0 脚本本身的名称
$1 脚本后跟的第一个参数
$# 参数个数
$？最后命令的退出状态
$* 所有参数
$$ 脚本当成进程id号
$@ 与$*作用一样
```

```shel
expr是一款表达式计算器，可以计算数值运算的值
`expr 1 + 1 `
或者
$[ 1+1 ]
```

```shell
关系运算符（只适用于数字）
-eq 检查是否相等，相等返回true    : equals
-ne 检查是否相等，不相等返回true  : not equals
-gt 左边大于右边，返回true       : greater than
-lt 右边大于左边，返回true       : less than
-ge 左边大于等于右边，返回true    : greater equals
-le 左边小于等于右边，返回true    : less equals


! 非运算
-o 或运算
-a 与运算

-e 文件或者目录是否存在  ： -e $file
```

### 脚本输出EOF用法

EOF通常都配合<<使用，unix环境下<<之后的字符是一个标志符，标识符之间的内容被输入，所以标识符可以是任何内容，只是通常用EOF（End Of File）代替。

```shell
<< EOF
...
EOF
```

...内容都被输入进去。

输入到哪用cat >>来指定文件

< :输入重定向
\> :输出重定向
\>> :输出重定向,进行追加,不会覆盖之前内容
<< :标准输入来自命令行的一对分隔号的中间内容.

```shell
xxxxxxxxxx tangkaifeideMacBook-Pro:Documents tang$ cat <<EOF >>test.sh "this is a cat&EOF demo" EOF
```

### echo用法

结果定向至文件

```shell
echo "It is a test" > myfile
```

显示命令结果用   **\`   \` ** 表示或者$()

```shell
echo `ls`
echo $(ls)
```

### 输入重定向

标准输入	命令<文件1	命令把文件1的内容作为标准输入设备
标识符限定输入	命令<<标识符	命令把标准输入中读入内容，直到遇到“标识符”分解符为止
输入输出重定向（同时使用）	命令< 文件1 >文件2	命令把文件1的内容作为标准输入，把文件2作为标准输出。

还有一个进程替换<()

```shell
$ cat <(ls)    #把<(ls)当一个临时文件，文件内容是ls的结果，cat这个临时文件
```

### 输出重定向

- 0: 代表标准输入
- 1: 代表标准输出
- 2:代表标准错误输出

正常输入输出都会在控制台显示，例如：

```shell
tangkaifeideMacBook-Pro:Documents tang$ ls
tang_hexo_blog	test.sh
tangkaifeideMacBook-Pro:Documents tang$ ls -l test.sh errofile
ls: errofile: No such file or directory
-rw-r--r--  1 tang  staff  40  5 22 17:23 test.sh
tangkaifeideMacBook-Pro:Documents tang$ 
```

我们可以把正常输出定向到文件夹内：

```shell
tangkaifeideMacBook-Pro:Documents tang$ ls -l test.sh errofile >test.sh 
ls: errofile: No such file or directory
tangkaifeideMacBook-Pro:Documents tang$ cat test.sh 
-rw-r--r--  1 tang  staff  0  5 22 17:36 test.sh
```

或者一样效果的：

```shell
tangkaifeideMacBook-Pro:Documents tang$ ls -l test.sh errofile 1>test.sh 
ls: errofile: No such file or directory
tangkaifeideMacBook-Pro:Documents tang$ cat test.sh 
-rw-r--r--  1 tang  staff  0  5 22 17:36 test.sh
tangkaifeideMacBook-Pro:Documents tang$ 
```

与此对应，可以把错误输出定向到文件夹内:

```shell
tangkaifeideMacBook-Pro:Documents tang$ ls -l test.sh errofile 2>test.sh 
-rw-r--r--  1 tang  staff  0  5 22 17:37 test.sh
tangkaifeideMacBook-Pro:Documents tang$ cat test.sh 
ls: errofile: No such file or directory
tangkaifeideMacBook-Pro:Documents tang$ 
```

如果同时重定向错误信息和重定向标准输出到文件必须使用两个重定向符号，并且必须在重定向符前加上相应的文件描述符:

```shell
tangkaifeideMacBook-Pro:Documents tang$ ls -l test.sh errorfile 1>testright.txt 2>testerror.txt 
tangkaifeideMacBook-Pro:Documents tang$ cat testright.txt 
-rw-r--r--  1 tang  staff  40  5 22 17:37 test.sh
tangkaifeideMacBook-Pro:Documents tang$ cat testerror.txt 
ls: errorfile: No such file or directory
tangkaifeideMacBook-Pro:Documents tang$ 
```

如果想将标准输出和错误信息重定向到一个日志文件，Bash Shell提供了&符，就不需要使用两个重定向符了:

```shell
tangkaifeideMacBook-Pro:Documents tang$ ls -l test.sh errorfile >test.sh 2>&1
tangkaifeideMacBook-Pro:Documents tang$ cat test.sh 
ls: errorfile: No such file or directory
-rw-r--r--  1 tang  staff  0  5 22 17:43 test.sh
tangkaifeideMacBook-Pro:Documents tang$ 
```

### 重定向到垃圾桶

```shell
command >/dev/null 2>&1 #0标准输入 1标准输出 2错误输出
```

### 输出变量

```( set -o posix ; set ) | less```

<<<<<<< Updated upstream
### 查看进程

```shell
ps -ef|grep syslog|grep -v "grep"

pstree -p PID #查看进程树
top -H -p PID #查看cpu利用率
```
### inode

硬链接`ln`和软链接`ln -s`的区别，硬链接时两个文件有相同的inode号，软链接时创建一个文件，这个文件的内容是打开另一个文件的inode，所以两个文件虽然最终都是打开一个inode，但是创建两个inode。

[Linux的inode的理解](https://blog.csdn.net/xuz0917/article/details/79473562)

```shell
利用${ } 还可针对不同的变数状态赋值(沒设定、空值、非空值)： 
${file-my.file.txt} ：假如$file 沒有设定，則使用my.file.txt 作传回值。(空值及非空值時不作处理) 
${file:-my.file.txt} ：假如$file 沒有設定或為空值，則使用my.file.txt 作傳回值。(非空值時不作处理)
${file+my.file.txt} ：假如$file 設為空值或非空值，均使用my.file.txt 作傳回值。(沒設定時不作处理)
${file:+my.file.txt} ：若$file 為非空值，則使用my.file.txt 作傳回值。(沒設定及空值時不作处理)
${file=my.file.txt} ：若$file 沒設定，則使用my.file.txt 作傳回值，同時將$file 賦值為my.file.txt 。(空值及非空值時不作处理)
${file:=my.file.txt} ：若$file 沒設定或為空值，則使用my.file.txt 作傳回值，同時將$file 賦值為my.file.txt 。(非空值時不作处理)
${file?my.file.txt} ：若$file 沒設定，則將my.file.txt 輸出至STDERR。(空值及非空值時不作处理)

${file:?my.file.txt} ：若$file 没设定或为空值，则将my.file.txt 输出至STDERR。(非空值時不作处理)

${#var} 可计算出变量值的长度：

${#file} 可得到27 ，因为/dir1/dir2/dir3/my.file.txt 是27个字节
```

[一篇教会你写90%的shell脚本-zhihu](https://zhuanlan.zhihu.com/p/264346586)



