---
layout: post
title: 「Shell脚本学习指南笔记」第01-05章-正则表达式-文本查找替换排序-管道
filename: 2014-04-15-notes-Classic-Shell-Scripting-chapter-01-05.md
category: Unix
tags: [shell]

---

《Shell脚本学习指南》，机工09版。li2摘抄笔记，2014年4月14日 ~ 2014年5月15日。
正则表达式是人为定义的规则，了解它，使用它，掌握它，熟能生巧。然后用它做更多的事情。
做笔记的好处之一：敲下来需要时间，就可以多想几遍；解困。
使用一年多，然后系统的看书。精读，记录。
给你时间的工作，很喜欢。
  
## 第1,2章 背景知识与入门

------

软件工具的原则： 输出格式必须与可接受的输入格式一致。目的是容易将一个程序的执行结果交给另一个程序处理。P20
 
 
`|` 管道符号可以在两个程序之间建立管道（pipeline）。P24
 
 
小型shell脚本典型的开发周期： 命令行测试以寻找完成工作的适当组合和语法；写入独立脚本；设置脚本执行权限；使用该脚本。P24
 
 
当shell执行一个程序时，会要求UNIX内核启动一个新的进程（process），以便在该进程里执行所指定的程序。P24
 
 
位于第一行的 #! 允许引用除shell之外的任意解释器.比如： ` #! /bin/sh -`， ` #! /bin/awk -f`
 
 
内核会扫描 `#!` 后的其余部分，以获取解释器的完整路径，解释器选项。几个初级陷阱：
 
- 脚本的可移植性依赖于解释器的完整路径名称；
- 选项之后不要有空格，否则也会传递给解释器；
- `#!` 与其后的解释器名称之间是否可以有空白，在较旧的系统上可能有不同的解释。P24
 
 
Shell识别三种基本命令：内建命令、Shell函数、外部命令。P28
 
 
Shell变量可用来保存字符串值，字符数无限制。遵循不限制（no arbitrary limit）设计原则。P29
变量赋值的方式：` 变量名称=字符串  或者"字符串1 字符串2"`（包含空格需要加双引号）
变量引用的方式：`$变量名称`
 
 
echo命令用来产生输出，可以用来提示用户，或是产生数据供进一步处理。
POSIX未定义echo的行为模式，因此导致echo版本差异，但标准保留了实现时定义（implementation-difined）。只要使用echo最简单的形式，可移植性不会有问题，相对复杂的输出使用printf，其格式 ` printf  "format-string"  [argumens ...] `  P32
 
标准输入/输出（standard I/O）的概念：程序应该有数据的来源端、数据的目的地端、报告问题的地方，它们被称为标准输入、标准输出、标准错误输出。当你登录时，UNIX把终端设置为标准输入、输出、错误。I/O重定向就是重新安排从哪里输入、输出到哪里。P33

``` bash
<     重定向标准输入
>     重定向标准输出，新建或者覆盖
>>    重定向标准输出，新建或者追加
|     建立管道，program1 | program2 将前者的标准输出与后者的标准输入相连。
``` 

管道可以把程序衔接起来，执行速度比文件I/O快上十倍。 后续将讨论如何将各类工具串起，通过管道以完成复杂的功能。P34
 
 
编写脚本时，你通常已有某种格式的原始数据，处理（排序、求和、平均、格式化打印等等）这些数据以得到期望的结果。
构造管道时，应该减少每个阶段的数据量。例如，使用sort排序前，先以grep找出相应的行，这样可以减少sort的工作量。P35
 
UNIX提供了对Shell特别有用的特殊文件：
 
-  /dev/null      位桶（bit bucket），写入此文件的数据会被丢掉。如果需要命令的退出状态，而非它的输出，此功能会很有用。读取/dev/null立即返回文件结束符（end-of-file）。
- /dev/tty     当程序打开此文件时，UNIX会自动将它重定向到一个终端。P35
 
 
所谓位置参数（positional parameters）指的就是shell脚本的命令行参数（command-line arguments）。在shell函数里，它们同时也可以是函数的参数。参数由整数命名。由于历史原因，当它超过9时，需要大括号括起来。P37
 
 
执行跟踪（execution tracing）脚本正在做的事情：
 
- 在命令行中：`解释器 -x 脚本`；
- 在脚本中：`set -x` 打开跟踪功能，`set +x` 关闭跟踪功能。
 
 
我们缩写的shell脚本常受到locale的影响，尤其是排序规则（collation order），正则表达式（regular expression）的“方括弧表示式”（bracket-expression）里的字符范围。P42
 
 
 
## 第3章 文本查找（Searching）与文本替换（Substitution）

------

查找文本，UNIX的专业术语叫匹配文本（matching text），有三种：P45
 
- grep     最早的匹配文本程序。使用POSIX定义的基本正常表达式（Basic Regular Expression, BRE）。
- egrep   扩展grep（Extended grep）。使用扩展正则表达式（Extended Regular Expression, ERE）。
- grep     快速grep（Fast grep）。匹配固定字符串而非正则表达式；可以并行（in parallel）地匹配多个字符串。
 
 
下面的段落将说明如何使用正则表达式完成具有可移植的shell脚本。
正则表达式是UNIX工具使用和构建模型上的基础，花些时间学习如何使用它们并且好好利用它们，你会不断从各个层面得到充分的回报。
正则表达式由两个基本组成部分建立：一般字符、特殊字符。P47
 
正则表达式入门资料： 《learning the UNIX operating system》（O'Reilly）， 《sed & awk》 （O'Reilly）
正则表达式进阶资料： 《mastering regular expression》 （O'Reilly）
 
正则表达式是一种表示方法，可以把诸如“以字母a开头的字符串”，转换成程序可以理解的表达式，从而查找特定的文本。
 
 
#### 使用正则表达式以强化本身功能的UNIX程序
- 匹配文本的grep工具族；     P47
- 改变输入流的sed流编辑器（stream editor）；
- 字符串处理程序语言：awk、Icon、Perl、Python、Ruby、Tcl；
- 文件查看程序（又名分页程序pager），more、page、pg、less；
- 文本编辑器，ed行编辑器、vi屏幕编辑器，还有一些插件（add-on）编辑器，emacs、vim等。
 
 
####  元字符 metacharacter
特殊字符称为“元字符”metacharacter. 本书以meta表示。
 
 
#### 简单的正则表达式匹配范例

``` bash
P49， 表3-2
tolstoy             位于一行上任何位置的7个字母 tolstoy
^tolstoy            出现在一行开头的7个字母 tolstoy
tolstoy$            出现在一行结尾的7个字母 tolstoy
^tolstoy$           一行只包含 7个字母 tolstoy
[Tt]olstoy          位于一行上任何位置的7个字母 tolstoy或Tolstoy
tol.toy             位于一行上任何位置，3个字母tol，加上1个任意字符，再加上3个字母toy
tol.*toy            位于一行上任何位置，3个字母tol，加上0个或多个任意字符，再加上3个字母toy
```
 
####  方括号表达式（bracket expression）
POSIX标准定义了方括号表达式（bracket expression），以扩大字符集范围，匹配非英文字符。
匹配方括号内的任一字符。连字符`-`指的是连续字符的范围。（范围因local而不同，因此不可移植）。`^`位于方括号的第一个字符，表示不在方括号指定字符列表内的任何字符。P48
除了一般字符外，还定义了如下：P50
 
- 字符集（character class），描述不同的字符集，`[:字符集:]`
- 排序符号（collating symbol），将多字符视为一个整体，`[.排序符号.]`
- 等价字符集（ equivalence  class），可以视为相同的一组字符，`[=等价字符集=]`
 
 
#### POSIX字符集

```bash
P50， 表3-3
[:alnum:]           数字字符
[:digit:]           数字字符
[:xdigit:]          十六进制数字
[:alpha:]           字母字符
[:lower:]           小写字母字符
[:upper:]           大写字母字符
[:punct:]           标点符号字符
[:blank:]           空格（space）与定位（tab）字符
[:space:]           空白（whitespace）字符
[:graph:]           非空格（nonspace）字符
[:cntrl:]           控制字符
[:print:]           可显示的字符
```
 
BRE与ERE共享一些常见的特性，不过仍有些重要差异。我们会从BRE的说明开始，再介绍ERE附加的meta字符，最后针对使用相同或类似的meta字符、但拥有不同语义的情况进行说明。P51
 
 
 
### 基本正则表达式
 
------
 
#### 匹配单个字符
- 一般字符；
- 转义的meta字符，`\*` 匹配字面上的 *；
- meta字符.（点号），`a.c`匹配abc、aac等；
- 方括号表达式，`c[aeiouy]t` 匹配 cat、cot、cut等；` c[^aeiouy]t` 匹配cbt等，`^`在方括号内表示取反（complement）；`[0-9]`表示0123456789所有数字；`[0-9a-fA-F]`表示所有十六进制数字。 P51
 
 
**范围表示法仅当程序运行在locale设置为POSIX时，才具可移植性。 字符集（比如` [:xdigit:]` ）提供了可移植的表示方法，而` [0-9a-fA-F]`不建议使用。 **
 
 
#### 排序字符
在部分非英语系语言里，为了匹配的需要，某些成对的字符必须视为“单个”字符，并且在其语系里与单个字符比较时，有其排序的定义方式。假如“ch”是这样的“单个”字符，那么`[a[.ch.] b ]`，匹配于字符a、b，或者ch，单独的c、h则不匹配。 P52
 
 
#### 等价字符集
让一组不同的字符，在匹配时视为相同。比如在French的locale下，如果存在`[=e=]`这样的等价字符集，那么正则表达式`[a[=e=]iouy]` 匹配所有小写英文元音字母 a、e、i、o、u、y，以及字母 ē、è 等。 P52
 
** 字符集、 排序符号、 等价字符集有效的使用：`[ [:alpha:] ]`表示所有的字母字符，而` [:alpha:]`只表示a、l、p、h、 :  **
 
 
#### 在方括弧表达式中，所有其它的meta字符失去特殊含义

```bash
[.*\]       匹配字面上的星号、反斜杠、句点；
[].*\]      要包含字面上的 ] ，需要将它放在列表最前面；
[-.*\]      要包含字面上的 - ，需要将它放在列表最前面；
[].*\-]     要包含字面上的 ]和- ，需要将 ] 放在列表最前面，- 放在最后。（因为 - 特殊含义表示范围，所以若当字面意义使用，就放在最后或最前）P53
``` 
 
#### 后向引用（backreference）
>正则表达式一个最重要的特性就是将匹配成功的模式的某部分进行存储供以后使用这一能力。称之为“后向引用”。所捕获的每个子匹配都按照在正则表达式模式中从左至右所遇到的内容存储。存储子匹配的缓冲区编号从 1 开始，连续编号直至最大 9 个子表达式，比如使用`'\2'`访问第2个子表达式。[详细参考《正则表达式简介 -后向引用 - vbs脚本之家》](http://www.jb51.net/article/4395.htm)
 
`\(why\).*\1`
`\(["']\).*\1`
更多的代码测试，参考附件。 此书上的解释简直绕口令。P53
 
 
#### 单个表达式匹配多个字符： 区间表达式（interval expression）
星号表示“匹配0或多个前面的单个字符”，`ab*c` 表示“匹配1个a、0或多个b、1个c”。
但 `*` 不能指定匹配个数，区间表达式（interval expression）可以解决此问题：P54

```bash
a\{5\}      匹配5个a
a\{5,\}     匹配至少5个a
a\{5,10\}   匹配5~10个a
``` 
 
####  锚点 （anchor） `^`, `$`
这两个meta字符 针对字符串的头、尾匹配。P54
`^$`可以用来匹配空串或者空行，`grep -v '^$'` 删除空行（-v显示不匹配模式的行）
BRE中的`^`, `$`仅在字符串的头、尾处有特殊作用。`ab^cd`, `ef$gh`中的^$就是字面上的含义。
`\^`针对出现在开头的普通^，所以使用转义字符。
`\$`针对出现在结尾的普通$.
`[$]`方括号表达式内的普通$.
`[^]`无效的正则表达式（方括号表达式内的^表示取反）TODO ` grep "[^]"  grep: Unmatched [ or [^`
 
 
#### BRE运算优先级（precedence）
 
```bash
表3-5 P55
[..] [==] [::]      方括号表达式中的符号
\meta               转义的meta字符
[]                  正则表达式
\( \)    \digit     子表达式与后向引用
*        \{ \}      字符重复
no symbol           连续
^ $                 锚点
``` 
 
 
### 扩展正则表达式
 
------

#### 匹配单个字符
ERE与BRE基本一致。P56
有一个著名的例外出现在awk里：符号`\`在方括号表达式表示其它含义。如果需要匹配左方括号、连字符、右方括号、反斜杠，应该使用`[\[\-\]\\]`。这是**使用上的经验法则**。TODO
 
 
#### 后向引用不存在
 
 
####  单个表达式匹配多个字符
 
 
| 需要匹配的      | BRE         | ERE       |
|:---------------:|:-----------:|:---------:|
| 重复5个a        | `a\{5\}`    | `a{5}`    |
| 重复5~10个a     | `a\{5,10\}` | `a{5,10}` |
 
| 表达式          | BRE            | ERE             | 说明                     |
|:---------------:|:---------------|:----------------|:-------------------------|
| `ab*c`          | ac,abc,abbc,...| ac,abc,abbc,... | 匹配>=0个前置字符        |
| `ab+c`          | 无meta字符`+`  | abc,abbc,...    | 匹配>=1个前置字符        |
| `ab?c`          | 无meta字符`?`  | 仅2个：ac, abc  | 匹配0个或1个前置字符     |
 
`*`位于ERE表达式的第一个位置时“未定义”，在BRE表达式中“字面意思的*”。 P57
 
 
#### 交替符号（alternation）`|` ，分组符号`()`
方括号表达式易于表示“匹配此字符，或其它字符，或……”；
交替符号易于表示“匹配这个序列，或其它序列，或……”；P57
 
| 表达式        |  含义                                          | 说明                                     |
|:-------------:|:-----------------------------------------------|:-----------------------------------------|
| `^ab\ef$`     | 匹配字符串的起始处是否有ad，或者结尾处是否有ef | 交替符号`\`优先级最低，类比`a*b+c*d`理解 |
| `^(ab\ef)$`   | 匹配一行仅是ab或ef的字符串                     | 分组符号`()`可以把meta符号用于“表达式”   |

**markdown表格中无法输入`|`，所以用\代替。**
 
 
 
### 文本替换（text substitution）
 
------
通过grep获取匹配的文本，称之为“原始数据 raw data”。而文本替换至少做一件事情：替换或删除匹配文本的某个部分。
sed（流编辑器stream editor）用来执行文本替换，用批处理方式而不是交互的方式编辑文件。P62
使用s命令，用replacement text替换匹配pattern的文本。
`sed 'editing command' [file ...]`, editging command如下格式：
`s/pattern/replacement/flags`
正则表达式可以用任意字符分隔（除换行符），所以当pattern包含`/`时，可以选择另外一个字符作为分隔符，比如`!`,`:`.
 
| flags | 说明                                                  |
|-------|-------------------------------------------------------|
| n     | 1~512, 对第n次匹配的情况执行替换                      |
| g     | 对匹配的情况，执行全局替换；无g只替换第一次匹配的情况 |
| p     | 打印pattern                                           |
| w     | file，把pattern写入file                               |

《sek & awk, 2nd edition, 5.1 sed命令的语法》
 
`sed 's/:.*//' /etc/passwd | sort -u`
用空串替换第一个冒号之后的文本（实际效果是删除第一个冒号之后的所有东西），排序并删除重复部分。不打印界定符`:`.
 
```bash
以下命令拷贝/home/testuser/路径下的目录结构，使用了“产生命令（generating commands）的手法”。
TODO该脚本无法处理目录含有空格的情况，将在第10章中解决。
find /home/testuser/ -type d -print         |  输出指定路径下的目录名称列表，一行一个打印
    sed 's;/home/testuser/;/home/backup/;'  |  修改目录名称，这里使用分号作为界定符
        sed 's/^/mkdir /'                   |  插入mkdir命令
            sh -x                              以shell跟踪模式执行
```
 
#### 匹配特定行
`-n p`仅打印以p指定的行，用以改变默认打印模式。P67
逗号隔开两个正则表达式称为范围表达式（range expression）。

```bash
$ grep weiyi /etc/passwd                         
weiyi:x:1000:1000:weiyi,,,:/home/weiyi:/bin/bash
$ sed -n '/weiyi/ s//WeiYi/p' /etc/passwd           仅替换第一个weiyi为WeiYi，这是不加flags的默认行为
WeiYi:x:1000:1000:weiyi,,,:/home/weiyi:/bin/bash
$ sed -n '/weiyi/ s//WeiYi/gp' /etc/passwd          替换所有的weiyi为WeiYi，使用flags g
WeiYi:x:1000:1000:WeiYi,,,:/home/WeiYi:/bin/bash
$ sed -n '\:weiyi: s//WeiYi/gp' /etc/passwd         \:: 等价于 //，演示如何使用不同的定界符
WeiYi:x:1000:1000:WeiYi,,,:/home/WeiYi:/bin/bash
$ sed -n '/weiyi/ s//WeiYi/2p' /etc/passwd          仅替换第二个weiyi为WeiYi，使用flags 数字2
weiyi:x:1000:1000:WeiYi,,,:/home/weiyi:/bin/bash
$ sed -n 's/weiyi/WeiYi/gp' /etc/passwd             另外一种匹配的方式
WeiYi:x:1000:1000:WeiYi,,,:/home/WeiYi:/bin/bash

sed -n '$p' sed.txt                    `$`指最后一行，本例仅打印最后一行
sed -n '10,20p' sed.txt                仅打印10-20行
sed '/fool/,/bar/ s/w1/w2/g' sed.txt   仅在包含fool或bar的行中，执行替换操作
sed '/fool/ !s/w1/w2/g' sed.txt        `!`是否定正则表达式，仅在不包含fool的行中，执行替换操作
```
 
#### 有多少文本会匹配
POSIX标准指出：“完全一致的匹配指的是，自最左边开始匹配、针对每一个子模式、由左至右、必须匹配到最长的可能字符串。”（子模式指的是ERE圆括号里的部分，BRE以`\(...\)`提供此功能）。P69
最长的最左边规则 longest leftmost

```bash
$ echo weiyi at ShangHai | sed 's/weiyi/li2/'           可以使用固定字符串
li2 at ShangHai
$ echo weiyi at ShangHai | sed 's/w.*i/li2/'            由于longest leftmost，会一直匹配到Hai的i
li2
$ echo weiyi at ShangHai | sed 's/w[[:alpha:]]i/li2/'   语法错误，只会匹配一个字母
li2yi at ShangHai
$ echo weiyi at ShangHai | sed 's/w[[:alpha:]]*i/li2/'  更精准的匹配
li2 at ShangHai
```
 
当开发的脚本是要执行大量文本剪贴和排列组合时，需要谨慎地测试每样东西，确认每个步骤的结果都是预期的——尤其是当你还在学习正则表达式的微妙的变化时。留意`b*`是如何匹配在abc开头和结尾的null字符串。

```bash
$ echo abc | sed 's/b*/1/'
1abc
$ echo abc | sed 's/b*/1/g'
1a1c1
```
 
#### 行 vs 字符串
大部分简易程序都是处理输入数据的行，像grep、egrep、sed工作的99%。此种情况下，不会有内嵌的换行符出现在要匹配的数据中，`^`, `$`分别表示行的开头和结尾。
对使用正则表达式的程序语言，如awk、Perl、Python，处理的多半是字符串。不仅可以处理单独的行；还允许使用不同的方式标明每条输入记录的定界符，因此记录可以内嵌换行符，比如地址簿用几行表示一条记录。此种情况下，`^`, `$`不能表示行的开头和结尾，只能表示字符串的开头和结尾。P70
 
 
### 字段处理
 
------
 
#### 文本文件惯例
UNIX系统鼓励使用文本数据，因此系统上最常用的数据存储类型是文本。在文本文件下，一行表示一条记录。P71
行内分隔字段的2种**界定符（delimitation），又称分隔符（field separator）**：
 
- 使用空白（whitespace）：空格键（space）或制表键（tab）；
  多个连续的空白符都将被看作一个界定符。
- 使用特定的界定符，比如冒号`:`;
  每个界定符都隔开一个字段。
 
尽量避免字段分隔符出现在字段内容中。
 
#### 使用cut选定字段
剪下输入字符或指定文件中的指定字段或者指定的字符范围。P72
`cut -d : -f 1,5 /etc/passwd`   取出第1，5个字段（从1开始编号）
`ls -l | cut -c 1-10`           取出第1到10个字符（从1开始编号）
`-f`指定以字段为主，使用`-d`指定的界定符（默认是TAB），执行cut操作。
`-c `指定以字符为主，cut其后跟的编号，或编号列表。
 
#### 使用join连接字段
join可以将多个文件结合在一起，输出结果包括共同的键值、来自file1的其余字段、来自file2的其余字段。
文件必须共享相同的键值（key），键值必须是惟一的（unique key），文件必须是排序的。
默认情况下，不包含键值的记录不被打印。P74
`join [options ...] file1 file2`
`-t CHAR`, use CHAR as input and output field separator，默认是空白符。
若file1为`-`, join会读取标准输入，常用于管道中。
 
#### 使用awk重新编排字段
awk的设计是要在shell中发挥所长：做一些简易的文本处理，例如取出字段并重新编排。P76
**单命令行程序 one-liners**
`awk [-option] 'pattern { action }' [file ...]`
对每条记录，awk会测试pattern，若为真（比如某条记录匹配于某正则表达式，或是一般表达式为真），则awk执行action。
 
awk设计的重点在字段与记录上：awk读取输入记录（通常是一些行），然后自动将各个记录分切为字段，并将每条记录的字段数目存储到内建变量`NF`中。
通过`$`引用字段值，`$1`引用第1个字段, `$NF`引用最后一个字段, `$0`引用整条记录。
 
```bash 
awk -F: '{print "User", $1, "path is", $6}' /etc/passwd  print的参数需要逗号隔开，否则参数之间无空白；
awk -F: '{printf "%-15s%s\n", $1,$6}' /etc/passwd        print自动提供换行，printf需要显示地输入`\n`;
awk -F: -v 'OFS=##' '{print $1,$6}' /etc/passwd          OFS改变输出域分隔符output field space; 否则默认是空格，即使输入字段分隔符是冒号。
``` 
 
 
## 第4章 文本处理工具

------
 
### 排序文本
 
------
一个可预期的记录次序，会让用户的生活更便利。如果没有次序依据鲜有价值。排序后的记录更易于程序化，也更有效率。
排序的惯例，完全视语言、国家、文化而定，且这样的规则有时会非常复杂。P81
`sort [options] [file(s)]` 将输入行按照键值字段与数据类型选项以及locale排序。
 
#### 以字段排序
`-t`选项定义界定符，若未指定，以空白分隔，且记录内头尾空白被忽略。
`-k`选项定义**排序键值字段**，其后的内容称为**字段选择器（field selector）**，可以更精确地控制排序结果。
字段以及字段里的字符编号从1开始。P84
 
`-t: -k2`       从第2个字段开始按比较，结束于记录的结尾；
`-t: -k2,2`     从第2个字段开始按比较，结束于第2个字段结尾；
`-t: -k2.3,5.6` 从第2个字段的第3个字符开始比较，结束于第5个字段的第6个字符；
`-t: -k2nr,2`   从第2个字段开始按整数、倒序比较，结束于第2个字段结尾，也可写作 `-k2, 2nr`, `-k2,2 -n -r`,
`-t: -k2n -k3n` 先从第2个字段整数排序，再第3个字段整数排序；
`-t: -k2n -u`   从第2个字段整数排序，如果键值字段匹配，仅输出一条记录；
 
| 字母 | 说明                                |
|------|-------------------------------------|
| b    | 忽略开头的空白          blank?      |
| d    | 字典顺序                dictionary? |
| f    | 不区分字母的大小写                  |
| g    | 以一般的浮点数字进行比较            |
| i    | 忽略无法打印的字符      ignore?     |
| n    | 以（整数）数字比较                  |
| r    | 倒置排序                reverse?    |
 
#### 文本块排序
多行作为一条记录，记录之间通过空行分隔，比如地址清单。
排序时，利用awk处理“行记录”分隔符的能力，识别“段落记录”的间隔。在每个段落记录内，用未使用过的字符（比如无法打印的控制字符），代替换行符。用换行符取代段落记录的间隔。P87
这样就可以把段落记录转变为行记录，然后利用sort排序，再然后恢复段落结构。
`RS`（record separator）表示输入记录分隔器；
`ORS`（output record separator）表示输出记录分隔器。

```bash 
cat address-book |                                                 地址数据
    awk -v RS="" '{ gsub("\n", "^Z"); print }' |                   转换段落记录为行记录
        sort -f |                                                  排序
            awk -v ORS="\n\n" '{ gsub("^Z", "\n"); print }' |      恢复
                grep -v '# SORTKEY'                                删除标记行
``` 
 
 
### 删除重复
uniq常用于管道中，删除已使用sort排序后的重复记录：`sort ... | uniq | ...`

```bash
uniq -c    在输出行之前加上该行的重复次数
uniq -d    仅显示重复的行
uniq -u    仅显示未重复的行
``` 
 
### 重新格式化段落
`fmt`
 
### 计算行数、字数、字符数 
`wc`
 
### 打印
  
 
### 提取开头或结尾的数行 
`head -n 25 /var/log/syslog`    提取文件的前几行，比如查看章节标题。
`tail -n 25 -f /var/log/syslog` 查看日志文件的后25行，如果此类文件是持续被写入的，可以通过选项-f监控，会每秒钟检查1次，除非ctrl+c中断。P99
 
 
### 其它 
`dd`, `file`, `od`, `strings`
 
 
 
 
## 第5章 管道的神奇魔力

------
 
UNIX工具使用原则：考虑这个问题该如何划分为更简单的工作，每个部分是否可用现成的具解决，还是写几行shell就能马上解决。P102
 
### 从结构化文本中提取数据
表达式稍微复杂。P104
 
```bash
姓名重新排序
$ echo "li2:Wei Yi. Li" |  sed 's=^\([^:]*\):\(.*\) \([^ ]*\)=\1:\3, \2='
li2:Li, Wei Yi.

取办公室号
$ echo "jones:Adrian W. Jones/OSD211/555-0123" |
> sed 's=^\([^:]*\):[^/]*/\([^/]*\)/.*$=\1:\2='
jones:OSD211

取电话号码
$ echo "jones:Adrian W. Jones/OSD211/555-0123" |
> sed 's=^\([^:]*\):[^/]*/[^\]*/\([^/]*\)=\1:\2='
jones:555-0123
``` 
 
 
### 关系数据库

数据以一对`key:value`形式构成，通过join操作生成多栏表格，用以提供选定的数据子集的视图。
处理“办公室名录表”的UNIX的软件工具，可以窥探 现代关系型数据库核心概念、 Structured Query Language（结构化查询语言）。P109
 
 
### 针对web的结构型数据

TODO



## 版本

------
li2 于上海闸北 
2014-04-15 ~ 2014-05-15, v1
