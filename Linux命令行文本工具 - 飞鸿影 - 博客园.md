# Linux命令行文本工具 - 飞鸿影 - 博客园
浏览文件[#](#浏览文件)
--------------

```null
cat      查看文件内容
more     以翻页形式查看文件内容（只能向下翻页）
less     以翻页形式查看文件内容（可以上下翻页）
head     查看文件的头几行（默认10行）
tail     查看文件的尾几行（默认10行）

```

示例：  
1、查看前10行

```null
$ head  -n 10 test.log

```

2、跟踪查看最后100行

```null
$ tail -f -n 100 test.log

```

wc[#](#wc)
----------

命令 `wc` 用于统计文件的行数、单词数、字符数等。

不带参数时默认输出一行，字段格式为：

```null
行数 单词数 字符数 文件名

```

常用参数：

```null
-l	只统计行数
-w	只统计单词数
-c	只统计字节数
-m	只统计字符数
-L	最长的一行包含了多少个字符

```

grep[#](#grep)
--------------

grep (global search regular expression(RE) and print out the line,全面搜索正则表达式并把行打印出来)是一种强大的文本搜索工具，它能使用正则表达式搜索文本，并把匹配的行打印出来。常用来在结果中搜索特定的内容。

一般格式：

```null
 grep [选项] 基本正则表达式 [文件]

```

### 选项[#](#选项)

```null
    -c    只输出匹配行的计数
    -i    不区分大小写（单字符）
    -h    不显示文件名（多文件时）
    -l    只输出文件名（多文件时）
    -n    显示匹配行及行号
    -s    不显示错误信息
    -v    显示不包含匹配文本的所有行
    -r    递归在子目录里文件查找

    --color=auto 自动高亮找到的关键词

```

示例

1.  将/etc/passwd，有出现 root 的行取出来:

```null
$ grep 'root' /etc/passwd

root:x:0:0:root:/root:/bin/bash
operator:x:11:0:operator:/root:/sbin/nologin

# 或者
$ cat /etc/passwd | grep 'root'

```

2)将/etc/passwd，有出现 root 的行取出来,同时显示这些行在/etc/passwd的行号:

```null
$ grep -n root /etc/passwd

1:root:x:0:0:root:/root:/bin/bash
30:operator:x:11:0:operator:/root:/sbin/nologin

```

3)将/etc/passwd，将没有出现 root 的行取出来

```null
$ grep -v root /etc/passwd

root:x:0:0:root:/root:/bin/bash
operator:x:11:0:operator:/root:/sbin/nologin

```

4)将/etc/passwd，将没有出现 root 和nologin的行取出来

```null
$ grep -v root /etc/passwd | grep -v nologin

root:x:0:0:root:/root:/bin/bash
operator:x:11:0:operator:/root:/sbin/nologin

```

5.  查找nginx是否运行：

```null
$ ps aux | grep nginx

www       1576  0.0  2.7  71652 28232 ?        S    Aug14   0:21 nginx: worker process

```

### 根据文件内容递归查找目录[#](#根据文件内容递归查找目录)

6)在当前目录里文件查找字符串'math'

```null
$ grep 'math' *

grep: my: Is a directory
grep: my1: Is a directory
s.txt:lisi 1989 male math 99
s.txt:wangxuebing 1978 male math 89
s.txt:lichang 1989 male math 99

```

7)在当前目录及其子目录下搜索'math'行的文件

```null
$ grep -r 'math' * 

```

8)当前目录及其子目录下搜索'math'行的文件，但是不显示匹配的行，只显示匹配的文件

```null
$ grep -l -r 'math' * 

s.txt

```

显示行号：

```null
$ grep -nr 'swoole' --color=auto /work/www/mixphp/*

```

### 正则表达式[#](#正则表达式)

支持正则语法，单引号里面写正则。

正则示例：

```null
't[ae]st' #查找test或者tast
'[^g]oo' #字符串不含有g。注意中括号里是不包含，不是以其开头
'[^a-z]oo' #字符串前不包含a-z小写字母
'[0-9]' #包含数字0-9
'^the' #匹配字母t开始的字符
'the$' #匹配字母e结尾的字符

```

示例：

```null
$ grep '^xu' s.txt 

xuliang 1977 male economic 89
xuxin 1986 female english 99

```

更多的正则知识请查看正则表达式相关知识。

### 扩展grep(grep -E 或者 egrep)[#](#扩展grepgrep--e-或者-egrep)

使用扩展grep的主要好处是增加了额外的正则表达式元字符集。

示例：

查找包含1990和1989的行：

```null
$ grep -E '1990|1989' s.txt 

lisi 1989 male math 99
wanglijiang 1990 female chinese 78
lichang 1989 male math 99
wanglijiang 1990 female chinese 78
lisibao 1989 male math 99
xiaobao 1990 female chinese 78


```

awk[#](#awk)
------------

### awk简介[#](#awk简介)

awk是一个强大的文本分析工具，相对于grep的查找，sed的编辑，awk在其对数据分析并生成报告时，显得尤为强大。简单来说awk就是把文件(或其他方式的输入流, 如重定向输入)逐行的读入（看作一个记录集）, 把每一行看作一条记录，以空格(或\\t,或用户自己指定的分隔符)为默认分隔符将每行切片（类似字段），切开的部分再进行各种分析处理。

awk有3个不同版本: awk、nawk和gawk，未作特别说明，一般指gawk，gawk 是 AWK 的 GNU 版本。

Awk基本语法:　

```null
awk 'pattern1 {command1;command 2…; command 3}  pattern2 { command …}'

```

pattern表示用来过滤记录的模式,可是是正则表达式，关系运算表达式，也可以什么也没有(表示选中所有记录)。

每个pattern选中的行记录会被花括号括起来的命令command操作一遍, command之间用`;`分割。 花括号里面可以什么也没有, 则默认为print输出整行记录。 Comamnd可以是输出， 可以是算术运算，逻辑运算，循环控制等等。

### 示例[#](#示例)

s.txt

```null
zhangsan 1977 male computer 83
lisi 1989 male math 99
wanglijiang 1990 female chinese 78
xuliang 1977 male economic 89
xuxin 1986 female english 99
wangxuebing 1978 male math 89
lichang 1989 male math 99
wanglijiang 1990 female chinese 78
zhangsansan 1977 male computer 83 
langxuebing 1978 male math 89
lisibao 1989 male math 99
xiaobao 1990 female chinese 78

```

一行中的5个字段分别表示`姓名, 出生年, 性别,科目,分数`, 是一个很传统很典型的报表文件。

现在演示awk是如何查找的：

1)直接输出1990年出生的同学:

```null
$ awk '/1990/' s.txt

wanglijiang 1990 female chinese 78
wanglijiang 1990 female chinese 78
xiaobao 1990 female chinese 78
 

```

或者：

```null
$ awk '/1990/{print $0}' s.txt

```

awk默认把输入的内容以空格拆分出每列。`$0`表示匹配所有列，`print $0`将输出所有列，每列分隔符是空格。

2）对chinese的课程的行输出"语文"：

```null
$ awk '/chinese/{print "语文"}' s.txt

语文
语文
语文


```

3）记录的头部和结尾加上一段说明：

```null
$ awk 'BEGIN{print "Result of the quiz:\n"}{print $0}END{print "------"}' s.txt
Result of the quiz:

zhangsan 1977 male computer 83
lisi 1989 male math 99
wanglijiang 1990 female chinese 78
xuliang 1977 male economic 89
xuxin 1986 female english 99
wangxuebing 1978 male math 89
lichang 1989 male math 99
wanglijiang 1990 female chinese 78
zhangsansan 1977 male computer 83
langxuebing 1978 male math 89
lisibao 1989 male math 99
xiaobao 1990 female chinese 78
------


```

AWK工作流程：**逐行扫描文件，从第一行到最后一行，寻找匹配特定模式的行，并在这些行上进行用户想要到的操作**。

BEGIN只会在最开始执行；END只会在扫描所有行数之后执行。BEGIN和END之间的花括号的内容每扫描一行都会执行。

4)查找女生的成绩且只输出姓名、学科、成绩：

```null
$ awk '$3=="female"{print $1,$4,$5}' s.txt
wanglijiang chinese 78
xuxin english 99
wanglijiang chinese 78
xiaobao chinese 78


```

`$1`表示第1列，`$n`类推。这里条件是表达式，而不是正则。print里`,`表示空格分隔符。

5)找出1990年出生的学生姓名，并要求匹配正则:

```null
$ awk '$2~/1990/{print $1}' s.txt
wanglijiang
wanglijiang
xiaobao


```

这里`~`表示匹配正则表达式。`!~`表示不匹配正则表达式。

如果需要多选，则改成：

```null
$ awk '$2~/(1990|1991)/{print $1}' s.txt

```

6)找出大于1985年出生的学生姓名,年龄，使用表达式：

```null
$ awk '$2>"1985"{print $1,$2}' s.txt
lisi 1989
wanglijiang 1990
xuxin 1986
lichang 1989
wanglijiang 1990
lisibao 1989
xiaobao 1990


```

### awk内置变量[#](#awk内置变量)

awk有许多内置变量用来设置环境信息，这些变量可以被改变，下面给出了最常用的一些变量。

```null
ARGC      命令行参数个数
ARGV      命令行参数排列
ENVIRON   支持队列中系统环境变量的使用
FILENAME  awk浏览的文件名
FNR       浏览文件的记录数

FS        设置输入列分隔符，等价于 -F选项。默认是空格
OFS       输出列分隔符。默认是空格

NF        列的字段总数，$NF指当前列最后一个的值
NR        已读的记录数（当前行数）

ORS       行输出分隔符，默认为\n
RS        行输入分隔符，默认分隔符为\n
RT        指定的那个分隔符

$0        指整条记录
$1, $2, … $n   分别是第1,2,...n列的字段值

```

示例：  
6)第四个字段科目为chinese的记录编号, 学生姓名, 科目:

```null
$ awk '$4=="chinese"{print NR, $1, $4, $5}' s.txt

3 wanglijiang chinese 78
8 wanglijiang chinese 78
12 xiaobao chinese 78


```

7)统计数学成绩大于90的个数：

```null
$ awk 'BEGIN{goodMath=0;}($4=="math" && $5>90){goodMath++}END{print goodMath}' s.txt

3

```

8)更换输入换行符

```null
echo "11 22|12 23" | awk 'BEGIN{RS="|"}{print $0}'

```

等价于：

```null
echo "11 22|12 23" | awk -v RS='|' '{print $0}'

```

输出：

```null
11 22
12 23

```

> 注：文本内容（例如"11 22\\n12 23"）里的`\n`不是换行符，实际是`\\n`。shell里字符串的`\n`要生效，需要使用`echo -e`。示例：`echo -e "11 22\n12 23" | awk '{print $0}'` 。

9)更换列输入、输出分隔符：

```null
$ cat /etc/passwd |awk  -F ':' -v OFS='\t'  '{print $1}'  
root
daemon
bin
sys

```

`-F`指定输入域分隔符为`:`。`-F`等价于`-v FS`。  
`-v OFS`指定输出域分隔符为`\t`。

注：`-F`和`-v OFS`在处理MySQL数据导出导入时有非常大的作用：我们可以使用`-F`指定每列是以`\t`或者`,`(csv格式)分隔的；输出的时候我们可以用`-v OFS`指定每列分隔符，默认的空格经常不足以方便使用。如果使用了`-v OFS`，使用`print $0`是改变不了输出分隔符的，需要手动指定列，例如`print $1,$2`。

10)批量操作

```null

docker ps | awk  '{print $1}' | xargs docker stop


docker ps -a | awk  '{print $1}' | xargs docker rm

```

11)文件切割

```null
awk '{filename = "sub." int((NR-1)/5000) ".csv"; print >> filename}' history.csv

```

每5W行切割为一个文件。

12)分组合并

例如test.txt文本内容是：

```null
yjc 1 20170118
yjc 1 20170118
lisi 1 20170223

```

需要整理成(姓名、日期相同的计数累加)：

```null
yjc 2 20170118
lisi 1 20170223

```

脚本：

```null
cat test.txt | awk '{a[$1$3]["c"]+=$2;a[$1$3]["u"]=$1;a[$1$3]["d"]=$3;}END{for(i in a)print a[i]["u"],a[i]["c"],a[i]["d"]}'

```

### awk函数[#](#awk函数)

awk还支持内置函数。这里只列举部分。

```null
int(x)	返回 x 的截断至整数的值
rand()	返回任意数字 n，其中 0 <= n < 1。
sqrt(x)	 返回 x 平方根。

sub(Ere, Repl, [In])    字符串替换
gsub(Ere, Repl, [In])    正则替换
index(str, str2)   str2在str中出现的位置，从1开始编号。不存在返回0
substr(str, M, [N]) 返回具有N参数指定的字符数量子串。如果未指定 N 参数，则子串的长度将是M参数指定的位置到str参数的末尾的长度。
length [(str)]	返回 str 参数指定的字符串的长度（字符形式）。如果未给出 str 参数，则返回整个记录的长度（$0 记录变量）。
match(str, Ere) 返回Ere匹配的字符串在str中出现的位置，从1开始编号。不匹配返回 -1
tolower(str) 字符串转小写
toupper(str) 字符串转大写
split(str, A, [Ere] )	将str参数指定的参数分割为数组元素 A[1], A[2], . . ., A[n]，并返回n变量的值。分隔符由正则表达式Ere匹配得出。

mktime(YYYY MM DD HH MM SS[DST]) 根据日期生成时间戳。失败返回-1 
strftime([format [, timestamp]])  格式化时间输出，将时间戳转为时间字符串
systime() 得到时间戳

```

13.  int函数

```null
$ echo "10.22元" | awk '{$1=int($1);print $1}'
10

```

如果只是想转换为数字，可以使用乘法运算：

```null
$ echo "10.22元" | awk '{$1=$1*1;print $1}'
10.22

```

14.  数学函数

```null
$ echo 9 | awk '{$1=sqrt($1);print $1}'
3

```

15.  字符串函数

```null
$ awk 'BEGIN{info="test2010test";gsub("2010"," ",info);print info}'
test test

$ awk 'BEGIN{info="test2010test";gsub(/[0-9]+/," ",info);print info}'
test test

$ awk 'BEGIN{info="test2010test";print index(info, 2010);}'
5

$ awk 'BEGIN{info="test2010test";print substr(info, 5);}'
2010test

$ awk 'BEGIN{info="test2010test";print length(info);}'
12

$ awk 'BEGIN{info="test2010test";print toupper(info);}'
TEST2010TEST

$ awk 'BEGIN{info="test2010test";print match(info, /[0-9]+/);}'
5

$ echo "10:20" | awk '{split($1,arr,":");print arr[1];print arr[1];print arr[1]*60+arr[2];}'
10
10
620

$ awk 'BEGIN{info="hello shell";split(info,arr," ");print length(arr);for(k in arr){print
 k,arr[k];}}'
2
1 hello
2 shell


```

> `awk for …in` 循环，是一个无序的循环。 并不是从数组下标`1…n` ，因此使用时候需要注意。split生成的数组下标从1开始。

16.  时间戳函数

```null
$  awk 'BEGIN{print systime();}'
1543668202

$ awk 'BEGIN{print strftime("%Y-%m-%d %H:%M:%S",1543668202);}'
2018-12-01 20:43:22

$ awk 'BEGIN{print mktime("2018 12 01 20 43 22");}'
1543668202

$ awk 'BEGIN{$1="2018-12-20";gsub(/[-:]/," ",$1);print mktime($1." 20 43 22");}'
1545309802

```

### printf格式化[#](#printf格式化)

格式和C语言的一样。支持`%d %s %u %f %c %o %x`等。

| 格式符 | 说明 |
| --- | --- |
| %d | 十进制有符号整数 |
| %u | 十进制无符号整数 |
| %f | 浮点数 |
| %s | 字符串 |
| %c | 单个字符 |
| %p | 指针的值 |
| %e | 指数形式的浮点数 |
| %x | %X 无符号以十六进制表示的整数 |
| %o | 无符号以八进制表示的整数 |
| %g | 自动选择合适的表示法 |

```null
$ awk 'BEGIN{x=12.12; printf("%.2f,%.2u,%d,%s,%o\n",x,x,x,x,x);}'
12.12,12,12,12.12,14

```

其它详见：[https://www.cnblogs.com/chengmo/archive/2010/10/08/1845913.html](https://www.cnblogs.com/chengmo/archive/2010/10/08/1845913.html)

17.  if...else

if后面的条件判断需要使用括号括起来。支持 ==, >, <, >=, <= 等判断。

示例: 将下列文件格式转换为新的格式：  
原格式：route.csv

```null

'/api/user/info' => 'User::getUserInfo',
'/api/user/info_batch' => 'User::getUserInfoBatch', 

```

新格式：route2.csv

```null

'/api/user/info' => ["route" => 'User::getUserInfo', "tag" => 'user'],
'/api/user/info_batch' => ["route" => 'User::getUserInfoBatch', "tag" => 'user'], 

```

脚本：

```null
cat route.csv | awk -F '=>' -v OFS='\t' '{print $1,$2}' | awk -F ',' -v OFS='\t' '{print $1,$2}'  |  awk -F '\t'  '{if($2>"") print $1, " => [ \"route\" => "$2", \"tag\" => "user" ], "$3; else print $1 }' >> route2.csv

```

sed[#](#sed)
------------

和grep、awk不同，sed更侧重对搜索文本的处理，如修改、删除、替换等等。

> sed工作原理：sed会一次处理一行内容。处理时，把当前处理的行存储在临时缓冲区中，成为"模式空间"，接着用sed命令处理缓冲区中的内容，处理完成后，把缓冲区的内容送往屏幕。接着处理下一行，这样不断重复，直到文件末尾。文件内容并没有改变，除非你使用重定向存储输出。

**语法**

```null
sed [options] 'command' file(s)
sed [options] -f scriptfile file(s)

```

**参数说明：** 

```null
-n, --quiet, --silent  安静模式，也就是不会输出默认打印信息
-e <script>或--expression=<script> 以选项中指定的script来处理输入的文本文件。
-f <script文件>, --file=<script文件>   以选项中指定的script文件来处理输入的文本文件。
-i  直接编辑文件而不是显示在屏幕上

-h, --help 显示帮助。
-V, --version 显示版本信息。

```

> mac上`-i`需要改成`-i ""`。或者使用`gsed`替换sed：`brew install gsed`

**动作说明：** 

```null
a   表示在指定行下边插入指定行的内容。
i   命令i和a使用上基本上一样，只不过是在指定行上边插入指定行的内容。
d   表示删除指定的行内容。
c   c是表示把指定的行内容替换为自己需要的行内容。注意是整行替换
y   字符替换，可以替换多个字符，只能替换字符不能替换字符串，且不支持正则表达式
s   字符串替换，是平时sed使用的最多的子命令。支持正则表达式
p  打印，亦即将某个选择的数据印出。通常 p 会与参数 sed -n 一起运行
r   类似于a，也是将内容追加到指定行的后边，只不过r是将指定文件内容读取并追加到指定行下边。 

```

sed命令必须跟一个动作。

新建文件t.txt:

```null
zhangsan 1977 male computer 83
lisi 1989 male math 99
wanglijiang 1990 female chinese 78

```

1)新增一行：第3行后新增：

```null
$ sed -e '3a newline'

zhangsan 1977 male computer 83
lisi 1989 male math 99
wanglijiang 1990 female chinese 78
newline

```

2)插入一行：第3行前插入：

```null
$ sed -e '3i newline'

zhangsan 1977 male computer 83
lisi 1989 male math 99
newline
wanglijiang 1990 female chinese 78

```

3)删除一行：删除第3行：

```null
$ sed -e '1,3d'

wanglijiang 1990 female chinese 78

```

4)替换一行：

```null
$ sed -e '3c newline'

zhangsan 1977 male computer 83
lisi 1989 male math 99
newline

```

5)行内部分内容的替换：  
格式：

```null
sed 's/要被取代的字串/新的字串/g'

```

示例：

```null
$ sed '3s/1990/2000/g' t.log 

zhangsan 1977 male computer 83
lisi 1989 male math 99
wanglijiang 2000 female chinese 78

```

一些替换规则范例：

```null
s/\t/","/g;      \t换成","
s/^/"/;      开头加上"
s/$/"/;s/\n//g     末尾加上"
s/"//g;      引号去掉 
s/,/ /g      逗号换成空格
s/,/\n/g     逗号换成空行
s/[NULL]//g;      NULL换成空白
s/[,，]/\n/g     中英文逗号换成空行

```

6)多行操作：

```null
$ sed '2,3d' t.log  
$ sed '2,$d' t.log  
$ sed '2,3a test' t.log 

```

7)r命令

```null
sed '2r t.log' message

```

将a.txt文件内容读取并插入到t1.log文件第2行的下边。

注意：  
1、上述的操作均只在输出缓冲里操作，文件并没有变化；需要直接修改文件，前面加`-i`参数；  
2、`-e`参数可以没有。

8)参数p: 打印匹配行

```null
$ sed -n ’2p’/etc/passwd 打印出第2行
$ sed -n ’1,3p’/etc/passwd 打印出第1到第3行
$ sed -n ‘$p’/etc/passwd 打印出最后一行
$ sed -n ‘/user/p’ /etc/passwd 打印出含有user的行
$ sed -n ‘/\$/p’ /etc/passwd 打印出含有$元字符的行，$意为最后一行
$ sed -n ‘$=’ ok.txt	打印总行数

```

xargs[#](#xargs)
----------------

之所以能用到这个命令，关键是由于很多命令不支持`|`管道来传递参数，而日常工作中有有这个必要，所以就有了`xargs`命令，例如：

```null
find /sbin -perm +700 | ls -l       
find /sbin -perm +700 | xargs ls -l   

```

wget示例：

```null
cat picurl.csv | grep -E -o "http://www.test.com(.*?).jpg" |  xargs wget -c

```

sort[#](#sort)
--------------

排序，默认按照字符升序排序。

```null
-r, --reverse   逆向（倒序）排序
-n, --numeric-sort  基于数字排序
-d, --dictionary-order  字典序
-h, --human-numeric-sort    按照人类可识别的顺序，例如（1G>1M>1K）
-f, --ignore-case  忽略大小写
-u, --unique    去重复（剔除重复行）

-t, --field-separator=SEP 指定分隔符（一般配合-k参数使用，单纯分割毫无意义）
-k, --key=POS1[,POS2]   指定从第几列到第几列作为去重标准（序号n从1开始）。如果仅给出一个数，则表示从当前位置到结尾。例如 -k 3,3表示按第3个字段排序; -k 3 则是按照第3个字段及后面字段内容排序。

```

以下面的内容为例：

```null
$ cat 1.txt
456
456
123
11
34
5678
456

```

按字符降序：

```null
sort -r 1.txt 
5678
456
456
456
34
123
11

```

按数值降序：

```null
# sort -rn 1.txt 
5678
456
456
456
123
34
11

```

去除重复行：

```null
$ sort -nu 1.txt 
11
34
123
456
5678

```

`-h`这个一般可以用来排序文件大小：

```null
$ du -h
2.0G    ./test2
4.0K    ./test3
316M    ./test
2.3G    .
$ du -h |sort -hr
2.3G    .
2.0G    ./test2
316M    ./test
4.0K    ./test3

```

`-t,-k`适用于多列的情况:

```null
$ cat 2.txt
baidu,100,5000
google,110,5000
sohu,100,4500
guge,50,3000

```

分别表示公司、员工数、最低工资。  
我们先按照员工数升序，如果员工数相同，按照最低工资升序：

```null
$ sort -n -t ',' -k 2,2 -k 3,3 2.txt 
guge,50,3000
sohu,100,4500
baidu,100,5000
google,110,5000

```

按照公司人数升序排序:

```null
# sort -t ','  -k 2,2nr 2.txt
google,110,5000
baidu,100,5000
sohu,100,4500
guge,50,3000

```

等同于：

```null
sort -t ','  -k 2,2 -nr 2.txt

```

按照第2列去重：

```null
sort -t ','  -k 2,2 -u 2.txt
sohu,100,4500
google,110,5000
guge,50,3000

```

> 注：有一些sort可能不能直接使用`-t "\t"` 让tab作为分隔符，这个时候你可以转义tab，例如使用`-t $'\t'` 这样就可以让tab成为分隔符了。

uniq[#](#uniq)
--------------

用于去重，需要先执行sort。

```null
-c: 显示文件中行重复的次数
-d: 只显示重复的行（相邻的行）
-D: 把所有重复的行都显示出来（相邻的行）

```

示例：

```null
# 排序后去重
$ sort  1.txt |uniq
11
123
34
456
5678

# 显示重复次数
$ sort 1.txt | uniq  -c
      1 11
      1 123
      1 34
      3 456
      1 5678

# 仅显示重复的内容
$ sort 1.txt | uniq  -d
456

# 显示所有重复的行内容
$ sort 1.txt | uniq  -D
456
456
456

```

注：uniq仅针对换行符是`\n`，对于windows下编写的文件如果换行符是`\r\n`则无法排序。

cut[#](#cut)
------------

cut 命令和awk功能有些类似，用于列的选取。cut 以行为单位，用指定分隔符将行切分为若干字段，选取所需要的字段。

格式：

```null
cut OPTION... [FILE]...

```

参数：

```null
-d：用来定义分隔符，默认为tab键
-f ：以 -d 定义的分隔符选取列，下标从1开始
-c：以字符为单位进行分割，选取列的字符
-b：以字节为单位进行分割，选取列的字节

-s：表示不包括那些不含分隔符的行，用于去掉注释或者标题一类的信息

```

其中`-f`、`-c`、`-b`选取范围参数含义：

```null
N：只取第N项
M,N：只取第M项和N项
N-：从第N项一直到行尾
N-M：从第N项到第M项（包括M项）
-M：从第一项到第M项（包括M项）
-：从第一项开始到结束的所有项

```

例1：按指定分隔符选取列

```null
$ echo "1,2,3,4,5" | cut -d , -f2-4
2,3,4

$ echo "1,2,3,4,5" | cut -d , -f2,3
2,3

$ echo "1,2,3,4,5" | cut -d , -f2-
2,3,4,5

```

例2：按指定字符选取列

```null
$ echo "hello world" | cut -c2-4
ell

$ echo "hello world" | cut -c2,3
el

$ echo "hello world" | cut -c2-
ello world

```

split[#](#split)
----------------

split命令用于将一个文件分割成多个。格式：

```null
split [OPTION]... [INPUT [PREFIX]]

```

参数：

```null
-l : 指定每多少行切成一个小文件。其中-l可以省略，仅写后面的数字。
-b：指定每多少字节切成一个小文件
-C：与参数"-b"相似，但是在切割时将尽量维持每行的完整性
-d：默认切割后的文件后缀是aa,ab,ac...结尾的，使用该选项后会以00,01,02...结尾
-a：指定后缀长度，默认的后缀长度是 2，也就是按 aa、ab、ac 这样的格式依次编号
[PREFIX] ：设置切割后文件的文件名前缀，默认是x

```

示例：

```null
$ wc -l access_test.log 
528 access_test.log

$ split -l 200 access_test.log access_test_ -d
$ ls access*
access_test_00  access_test_01  access_test_02  access_test.log

$ wc -l access_test*
   200 access_test_00
   200 access_test_01
   128 access_test_02
   528 access_test.log
  1056 total



$ split -b 20k access_test.log access_byte -d -a 3
$  ls access_byte*
access_byte000  access_byte001  access_byte002  access_byte003  access_byte004
$  du -h access_byte*
20K	access_byte000
20K	access_byte001
20K	access_byte002
20K	access_byte003
12K	access_byte004

```

paste[#](#paste)
----------------

paste 用于把两个文件的内容按列合并一起，而不是简单在文件后面追加。类似于 MySQL里的左联查询多个表的结果。

格式：

```null
paste [OPTION]... [FILE]...

```

参数：

```null
-d 拼接时使用指定符号隔开各个文件的内容
-s 将所有换行删掉

```

示例1：

```null
$ cat a.txt 
1
2
3

$ cat b.txt 
a
b
c

$ paste -d ':' a.txt b.txt 
1:a
2:b
3:c

```

示例2：

```null
$ paste -s -d ',' a.txt
1,2,3

$ echo -e "1\n2\n3" | paste -s -d ',' 
1,2,3

```

tr[#](#tr)
----------

tr 命令常用于转换或删除文件中的字符。支持的参数：

```null
tr [-Ccsu] string1 string2
       tr [-Ccu] -d string1
       tr [-Ccu] -s string1

```

示例1：替换字符串

```null
$ echo "a|b|c" | tr "|" ","
a,b,c

$ echo "a|b|c" | tr a-z A-Z
A|B|C

```

示例2：删除指定字符串

```null
$ echo "a|b|c" | tr -d "|"
abc

```

示例3：删除指定连续重复字符串

```null
$ echo "a|b||c" | tr -s "|"
a|b|c

```

read[#](#read)
--------------

read命令用于从标准输入读取数值。  
示例1：

```null
$ read name
jack
$ echo $name
jack

```

执行read后，命令行等待用户输入。我们可以加 `-s`参数，这样命令行不会看到输入的内容：

```null
$ read -s name
$ echo $name
jack

```

注意区别。

示例2：  
read如果用于管道后面，则会读取管道里的内容：

```null
$ echo "jack jack2"  | read name name2
$ echo $name,$name2
jack,jack2

```

用于tr命令后实现批处理效果：

```null
$ echo "a|b|c" | tr "|" "\n" | while read name ;do echo $name; done

a
b
b

```

jq[#](#jq)
----------

jq 命令是用来处理json文本的。

需要先安装：

```null
yum install -y jq

```

官网示例：[https://stedolan.github.io/jq/tutorial/](https://stedolan.github.io/jq/tutorial/)

示例：

```null
$ cat jq.json 
{
    "name": "Json",
    "page": 88,
    "address": {
        "street": "科技园路.",
        "city": "江苏苏州"
    },
    "links": [
        {
            "name": "Google",
            "url": "http://www.google.com"
        },
        {
            "name": "Baidu",
            "url": "http://www.baidu.com"
        }
    ]
}

$  cat jq.json | jq .name
"Json"

$ cat jq.json | jq '.links' | jq '.[] | .name'
"Google"
"Baidu"

$ cat jq.json | jq '.links' | jq '[.[] | .name ]'
[
  "Google",
  "Baidu"
]

$ cat jq.json | jq '.links' | jq .[0].name
"Google"

```

scp[#](#scp)
------------

scp是secure copy的简写，用于在Linux下进行远程拷贝文件的命令，和它类似的命令有cp，不过cp只是在本机进行拷贝不能跨服务器，而且scp传输是加密的。

格式：

```null
scp [参数] [原路径] [目标路径]

```

示例：

```null

scp root@192.168.120.204:/opt/soft/nginx-0.5.38.tar.gz /opt/soft/


scp /opt/soft/nginx-0.5.38.tar.gz root@192.168.120.204:/opt/soft/

```

命令参数：

```null
-1  强制scp命令使用协议ssh1  
-2  强制scp命令使用协议ssh2  
-4  强制scp命令只使用IPv4寻址  
-6  强制scp命令只使用IPv6寻址  
-B  使用批处理模式（传输过程中不询问传输口令或短语）  
-C  允许压缩。（将-C标志传递给ssh，从而打开压缩功能）  
-p 保留原文件的修改时间，访问时间和访问权限。  
-q  不显示传输进度条。  
-r  递归复制整个目录。  
-v 详细方式显示输出。scp和ssh(1)会显示出整个过程的调试信息。这些信息用于调试连接，验证和配置问题。   
-c cipher  以cipher将数据传输进行加密，这个选项将直接传递给ssh。   
-F ssh_config  指定一个替代的ssh配置文件，此参数直接传递给ssh。  
-i identity_file  从指定文件中读取传输时使用的密钥文件，此参数直接传递给ssh。    
-l limit  限定用户所能使用的带宽，以Kbit/s为单位。     
-o ssh_option  如果习惯于使用ssh_config(5)中的参数传递方式，   
-P port  注意是大写的P, port是指定数据传输用到的端口号   
-S program  指定加密传输时所使用的程序。此程序必须能够理解ssh(1)的选项。

```

rsync[#](#rsync)
----------------

rsync(remote sync) 是用于同步某一位置文件和目录到另一位置的有效方法。备份的位置可以在本地服务器或远程服务器。

```null
rsync [OPTION]... SRC [SRC]... DEST

```

示例：  
将Jenkins编译生成的文件同步到远程服务器，排除git目录：

```null
rsync -azP --exclude .git /var/lib/jenkins/workspace/myapi/output/ root@172.17.17.10:/alidata/myapi/

```

参数：

```null
-a --参数，相当于-rlptgoD ，同步软链接，同步权限， 同步时间戳，同步属主和属组
-r --是递归 
-l --是链接文件，意思是拷贝链接文件
-i --列出 rsync 服务器中的文件
-p --表示保持文件原有权限
-t --保持文件原有时间 
-g --保持文件原有用户组 
-o --保持文件原有属主 
-D --相当于块设备文件 
-z --传输时压缩
-P --传输进度 
-v --传输时的进度等信息，和-P有点关系 

```

与scp的区别：

> 1、对于scp来说，除了在机器之间和一个机器目录之间进行数据同步之外，还可以在两台不同机器之间进行数据同步。比如你在A机器，可以对B、C两台机器上的数据进行同步，但是rsync就不可以；也就是说当rsync进行跨机器同步数据的时候只可以在本机与另外一台机器之间进行数据的同步。  
> 2、rsync是为了在两个机器之间进行数据的同步，既然有了scp为何还要有这个协议呢？该协议主要目的是在两台机器之间进行数据同步的时候，尽量少的传递数据。rsync可以聪明的在两台机器之间进行数据的同步，并通过合适的差分编码减少数据的传输。rsync的作用就是当要同步数据的对端已经存在部分要同步数据的情况下，通过使用rsync可以只传递对端没有的数据。假设一个文件100G，在文件末尾只加了一个句号。这时候要同步数据，如果使用scp那么要拷贝传输100G数据过去，而rsync只传输修改后的数据，整个任务就结束了。

综合示例[#](#综合示例)
--------------

### NGINX访问量统计[#](#nginx访问量统计)

查找7月10日访问log导出到10.log文件中：

```null
cat gelin_web_access.log | egrep "10/Jul/2019" | sed  -n '/00:00:00/,/23:59:59/p' > /tmp/10.log

```

查看访问量前10的IP

```null
awk '{print $1}' 10.log | sort | uniq -c | sort -nr | head -n 10 

```

根据访问IP统计UV

```null
awk '{print $1}' gelin_web_access.log | sort | uniq -c | wc -l

```

统计访问URL统计PV

```null
awk '{print $7}' gelin_web_access.log | wc -l

```

根据时间段统计查看日志

```null
cat gelin_web_access.log | sed -n '/10\/Jul\/2019:12/,/10\/Jul\/2019:13/p' | more

```

参考[#](#参考)
----------

1、linux awk命令详解 - ggjucheng - 博客园  
[http://www.cnblogs.com/ggjucheng/archive/2013/01/13/2858470.html](http://www.cnblogs.com/ggjucheng/archive/2013/01/13/2858470.html)  
2、sed入门详解教程 - 肖邦linux - 博客园  
[https://www.cnblogs.com/liwei0526vip/p/5644163.html](https://www.cnblogs.com/liwei0526vip/p/5644163.html)  
3、Linux 入门记录：十七、Linux 命令行文本/文件处理工具 - mingc - 博客园  
[https://www.cnblogs.com/mingc/p/7616206.html](https://www.cnblogs.com/mingc/p/7616206.html)  
4、linux sort 命令详解 - 孙愚 - 博客园  
[https://www.cnblogs.com/51linux/archive/2012/05/23/2515299.html](https://www.cnblogs.com/51linux/archive/2012/05/23/2515299.html)  
5、OFS-输出字段分隔符 | awk  
[https://lvs071103.gitbooks.io/awk/content/awk\_syntax\_and\_basic\_commands/ofs-.html](https://lvs071103.gitbooks.io/awk/content/awk_syntax_and_basic_commands/ofs-.html)  
6、Awk关系运算符和布尔运算符 - 自由的代价永远是警惕 - CSDN博客  
[https://blog.csdn.net/liu454638324/article/details/41578031](https://blog.csdn.net/liu454638324/article/details/41578031)  
7、linux awk 内置函数详细介绍（实例） - 程默 - 博客园  
[https://www.cnblogs.com/chengmo/archive/2010/10/08/1845913.html](https://www.cnblogs.com/chengmo/archive/2010/10/08/1845913.html)  
8、rsync 使用示例 | 《Linux就该这么学》  
[https://www.linuxprobe.com/rsync-use-sample.html](https://www.linuxprobe.com/rsync-use-sample.html)  
9、Linux之rsync数据同步服务 - 潇潇、寒 - 博客园  
[http://www.cnblogs.com/caicairui/p/8461216.html](http://www.cnblogs.com/caicairui/p/8461216.html)  
10、【Linux】Linux下同步数据scp与rsync - zwan0518的专栏 - CSDN博客  
[https://blog.csdn.net/zwan0518/article/details/45695939](https://blog.csdn.net/zwan0518/article/details/45695939)  
11、每天一个linux命令（60）：scp命令 - peida - 博客园  
[http://www.cnblogs.com/peida/archive/2013/03/15/2960802.html](http://www.cnblogs.com/peida/archive/2013/03/15/2960802.html)  
12、Linux 命令大全 | 菜鸟教程  
[https://www.runoob.com/linux/linux-command-manual.html](https://www.runoob.com/linux/linux-command-manual.html)  
13、TL;DR - Linux.cn  
[https://tldr.linux.cn/cmd/tr](https://tldr.linux.cn/cmd/tr)