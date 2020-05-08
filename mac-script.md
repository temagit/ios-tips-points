####ls　显示文件或目录

-l          列出文件详细信息l(list)

-a          列出当前目录下所有文件及目录，包括隐藏的a(all)

####mkdir        创建目录

-p          创建目录，若无父目录，则创建p(parent)



####touch          创建空文件

####echo          创建带有内容的文件。

####cat            查看文件内容

####cp            拷贝

####mv            移动或重命名

####rm              删除文件

-r            递归删除，可删除子目录及文件

-f            强制删除



####find              在文件系统中搜索某文件

####wc                统计文本中行数、字数、字符数

####grep            在文本文件中查找某个字符串

####rmdir          删除空目录

####tree            树形结构显示目录，需要安装tree包

####pwd              显示当前目录

####ln                  创建链接文件

####more、less  分页显示文本文件内容

####head、tail    显示文件头、尾内容

####ctrl+alt+F1  命令行全屏模式



## Linux中常用文件指令

####stat              显示指定文件的详细信息，比ls更详细

####who              显示在线登陆用户

####whoami          显示当前操作用户

####hostname      显示主机名

####uname          显示系统信息

####top                动态显示当前耗费资源最多进程信息

####ps                  显示瞬间进程状态 ps -aux

####du                  查看目录大小 du -h /home带有单位显示目录信息

####df              查看磁盘大小 df -h 带有单位显示磁盘信息

####ifconfig          查看网络情况

####ping                测试网络连通

####netstat          显示网络状态信息

####man                命令不会用了，找男人  如：man ls

####clear              清屏

####alias              对命令重命名 如：alias showmeit="ps -aux" ，另外解除使用unaliax showmeit

#### kill                杀死进程，可以先用ps 或 top命令查看进程的id，然后再用kill命令杀死进程

##打包压缩相关命令

####gzip：

####bzip2：

####tar:                打包压缩

-c              归档文件

-x              压缩文件

-z              gzip压缩文件

-j              bzip2压缩文件

-v              显示压缩或解压缩过程 v(view)

-f              使用档名

tar -cvf abc.tar abc              只打包，不压缩

tar -zcvf abc.tar.gz abc        打包，并用gzip压缩

tar -jcvf abc.tar.bz2 abc      打包，并用bzip2压缩

####当然，如果想解压缩，就直接替换上面的命令  tar -cvf  / tar -zcvf  / tar -jcvf 中的“c” 换成“x” 就可以了

tar -xvf abc.tar abc

tar -xcvf abc.tar.gz abc

tar -xcvf abc.tar.bz2 abc



##关机/重启机器

####shutdown

sudo shutdown [-h | -r | -s] [time]

-h ：关机（halt）

-r ：重启（reboot）

-s ：休眠（sleep）

time ：执行操作的时间



-r            关机重启

-h            关机不重启

now          立刻关机

####halt              关机

####reboot          重启


示例
sudo shutdown -h 1656055737 //在 2022-06-24 15:28:57 关机

sudo shutdown -h now  //立刻关机

sudo reboot

sudo shutdown -r now

##vi/vim 的使用

基本上 vi/vim 共分为三种模式，分别是命令模式（Command mode），插入模式（Insert mode）和底线命令模式（Last line mode）

####命令模式：

用户刚刚启动 vi/vim，便进入了命令模式。

i 　切换到插入模式，以输入字符。

x  删除当前光标所在处的字符。

:  切换到底线命令模式，以在最底一行输入命令。



若想要编辑文本：启动Vim，进入了命令模式，按下i，切换到输入模式

####输入模式：

在命令模式下按下 i 就进入了输入模式。

在输入模式中，可以使用以下按键：

ENTER(回车键)      　　换行

BACK SPACE(退格键) 　　删除光标前一个字符

方向键        　　　　　在文本中移动光标

HOME/END            移动光标到行首/行尾

Page Up/Page Down    上/下翻页

ESC                  退出输入模式，切换到命令模式



####底线命令模式：

在命令模式下按下 :（英文冒号）就进入了底线命令模式。

底线命令模式可以输入单个或多个字符的命令，可用的命令非常多。

在底线命令模式中，基本的命令有（已经省略了冒号）：



q 　　退出程序

w 　　保存文件

按ESC键可随时退出底线命令模式。







常用命令 一般模式切换到编辑模式

i  从目前光标所在处插入

I  在目前所在行的第一个非空格符处开始插入

a  从目前光标所在的下一个字符处开始插入

A  从光标所在行的最后一个字符处开始插入

o  在目前光标所在的下一行处插入新的一行

O  在目前光标所在处的上一行插入新的一行

r  只会取代光标所在的那一个字符一次

R  会一直取代光标所在的文字，直到按下 ESC 为止



一般模式切换到指令行模式

:w      将编辑的数据写入硬盘档案中

:w!    强制将编辑的数据写入硬盘档案中

:q      离开

:q!    为强制离开不储存档案

:wq    储存后离开

:wq!    强制储存后离开

:set nu    　 显示行号，设定之后，会在每一行的前缀显示该行的行号

:set nonu  　  取消行号

##文件权限

终端输入可查看文件操作权限

ls -l

####三种基本权限

r          读        数值表示为4

w          写        数值表示为2

x          可执行      数值表示为1


文件权限'
![20191550.png](https://upload-images.jianshu.io/upload_images/1419920-66fcb44dae400729.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![20191548.png](https://upload-images.jianshu.io/upload_images/1419920-e0b261c25427839a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



-rw-r--r--一共十个字符，分成四段。 1 表示可执行，2 表示可写，4 表示可读。每种类型数字相加所得到的值表示交叉部分的公共类型
第一个字符“-”表示普通文件；这个位置还可能会出现“l”链接；“d”表示目录
第二三四个字符“rw-”表示当前所属用户的权限。   所以用数值表示为4+2=6
第五六七个字符“r--”表示当前所属组的权限。      所以用数值表示为2
第八九十个字符“r--”表示其他用户权限。              所以用数值表示为2
所以操作此文件的权限用数值表示为622 

九个字母分为三组，从前到后每组分别对应所属用户（user）、所属用户所在组（group）和其他用户（other）对该文件的访问权限；
每组中的三个字符 “rwx” 分别表示对应用户对该文件拥有的可读／可写／可执行权限，没有相应权限则使用 “-” 符号替代。

####更改权限

sudo chmod [u所属用户  g所属组  o其他用户  a所有用户]  [+增加 
权限  -减少权限]  [r  w  x]   目录名 

例如：有一个文件filename，权限为“-rw-r----x” ,将权限值改为"- 
rwxrw-r-x"，用数值表示为765

sudo chmod u+x g+w o+r  filename

复杂一点操作的话，可以同时使用多种操作符添加和取消权限，并且可以使用 “,” 符号同时对不同用户范围修改权限，比如
chmod g+x,o+x-w shard.sh





修改所有用户的访问权限均为可读可写可执行（rwx）的话，使用数值修改如下:

sudo chmod 777 [文件名]





