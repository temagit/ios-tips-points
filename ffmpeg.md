## iOS基于FFmpeg 4.1 的命令编译
### 1、iOS集成FFmpeg
下载脚本[FFmpeg脚本地址](https://github.com/kewlbear/FFmpeg-iOS-build-script.git)

这里目前使用的是最新的4.1（分支ffmpeg-3.0 编译FFmpeg 3.0）
### 2、编译FFmpeg-iOS-build-script，获得FFmpeg静态库文件
打开脚步地址终端执行


    ./build-ffmpeg.sh


编译的时间略长，请耐心等待
### 3、iOS项目集成FFmpeg
+ 上步操作执行成功后，会生成FFmpeg-iOS文件，将该文件直接拖到项目中
+ 配置头文件搜索路径：在工程文件->Bulid Setting->Search Paths->Header Search Paths添加$(SRCROOT)/$(PRODUCT_NAME)/FFmpeg-iOS/include,(请根据自己实际路径更改)
+ 使用FFmpeg工具命令，添加以下文件

##### config.h文件scratch目录下的任意架构的config.h，其余的在/ffmpeg-4.1/fftools目录下，之前版本有的直接在ffmpeg-4.1目录下


     config.h
     cmdutils.c
     cmdutils.h
     ffmpeg_cuvid.c
     ffmpeg_filter.c
     ffmpeg_hw.c
     ffmpeg_opt.c
     ffmpeg_videotoolbox.c
     ffmpeg.c
     ffmpeg.h
    
     

#####  编译后会报错，然后根据提示挨个修复，还需要导入相应的依赖库的头文件（ffmpeg-4.1目录下相应的缺失.h文件，拷贝到工程里面）
#### 最后会报错 duplicate symbols for architecture arm64 是因为main函数有两个，在ffmpeg.h中添加一个函数声明：

    int ffmpeg_main(int argc, char **argv);
#### 在ffmpeg.c 文件，将main改为名为ffmpeg_main的入口函数，编译OK
 
 
 


