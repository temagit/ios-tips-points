## 崩溃分析
### 1.1 Crash日志（crash log）
#### 1.1.1 在Xcode中查看崩溃日志
Xcode -> Window -> Organizer-> Crashs

![](xcode-crash_1.webp)

#### 1.1.2 根据符号表来检查崩溃位置
#### 什么是符号表
###### 符号表是在Xcode项目编译后，在Build目录下与 xxx.app统计目录的文件后缀名为，dYSM的文件

###### .dSYM文件是个目录文件，在子目录中包含了16进制的保存函数地址映射的中转文件，所有的Debug的symbols都在这个文件中（包括文件名，函数名行号等）。
#### 符号表有什么用
###### 符号表就是用来符号化crash log 崩溃日志的。crash log 中有一些方法16进制的内存数据等，通过符号表就能对应的直观的看到方法名，类等。

#### 如何得到.dsYM文件

###### Archive的时候会生成。.xcarchive文件，显示包内内容就可以找到 .dsYM文件。

#### 如何使用.dsYM
##### 1、第三方日志框架会使用，Firebase, 友盟(有时候自动上传缺失需要手动上传)
#####  2、 或者自己找到.xcarchive文件和错误内存地址(友盟错误详情里标绿色的为错误内存地址)。然后通过一个小应用来分析出对应的函数。[应用下载地址](https://github.com/answer-huang/dSYMTools)
#####  3、命令行工具symbolicatecrash
###### symbolicatecrash是xcode的一个符号化crash log的命令行工具。使用方法也就是导出.crash文件（crash log）和找到.dsYM文件，然后进行分析。
如何使用[symbolicatecrash使用](Symbolicatecrash.md)
#####  4、命令行工具atos
###### 如果你有多个 .ipa 文件， 多个 .dSYM文件，你并不太确定 dSYM文件对应哪个 .ipa 文件，就可以用atos命令行工具。
如何使用[命令行工具atos](atos.md) 

