## opencv配置过程 （cmake，vs2013，qt 5.4）

平台及软件：Windows 7 X86

Visual Studio 2013，OpenCV3.0.0，Cmake3.3

### 新版opencv配置方法

 新版OpenCV将以往多个库集成到2个库文件：  **F:\opencv\build\x86\vc12\lib\opencv_ts300.lib** ** 和 **F:\opencv\build\x86\vc12\lib\opencv_ts300.lib** 中。仅将这两个库文件加载到Qt的.pro文件中即可，不用再进行Cmake编译生成QT/VS可用的库。以往常用的大部分库文件可以在**F:\opencv\build\x86\vc12\staticlib**  中找到，也可以将此文件夹中的相应库文件加入.pro文件中。用opencv自带的库，运行没有出现问题。

```c++
 INCLUDEPATH += F:/opencv/build/include\
                F:/opencv/build/include/opencv \
                F:/opencv/build/include/opencv2
 
CONFIG(debug, debug|release):
{
LIBS += F:\opencv\build\x86\vc12\lib\opencv_ts300d.lib\
        F:\opencv\build\x86\vc12\lib\opencv_world300d.lib
 }

CONFIG(release, debug|release):
{
LIBS +=F:\opencv\build\x86\vc12\lib\opencv_ts300.lib \
       F:\opencv\build\x86\vc12\lib\opencv_world300.lib
}
```

### 老版opencv的配置过程

#### 下载

下载Windows下的安装文件OpenCV-3.0.0.exe，解压，选择需要的安装目录即可。（本文为F:\opencv）。注意相应的目录不能包含中文。

#### Cmake编译

执行CMake,用于把OpenCV的源码生成对应的VS工程。

![cmake](https://github.com/strawdiving/Qt-Experience/blob/master/opencv/images/cmake.png)

1. 设置OpenCV的安装文件路径（Where is the source code）和想要生成的文件路径（Where to build the binaries）, **  安装文件路径必须包括cmakelists文件，想要生成的文件路径任意。** 

2. 点击左下方Configure，在弹出的窗口中选择Visual Studio 2012 （VS2013 可用），其他默认

**在configure前需要先配置环境变量，将QT的D:\Qt\Qt5.4.1\5.4\msvc2013_opengl\bin加入，否则无法cmake**

点击Finish即开始配置，配置完成如图，原来的设置不动，再选择（勾选）需要加入的文件WITH_QT和WITH_OPENGL,再次configure。

配置完成后，如无错误，红色消失。

3.  按Generate。Generate完成后，会有完成提示。

#### VS编译版本库

 以上操作完成后，就可以在生成的目录下找到对应的工程文件，打开，进行如下操作。

1. 在Debug下，打开解决方案“OpenCV.sln”，重新生成解决方案；
2. 生成成功后，选择INSTALL项目，右键运行生成；
3. 在Release下进行1-2步的操作；
4. 以上操作完成后，针对当前的系统的OpenCV库就生成了。

#### 包含目录

用Qt Creator编译opencv的时候，在创建一个新工程后，还需要在该工程的工程文件.pro文件内添加下列语句：

```c++
INCLUDEPATH+=  F:\OpenCV\install\include
               F:\OpenCV\install\include\opencv
               F:\OpenCV\install\include\opencv2
// 添加需要使用的相关库
CONFIG(debug, debug|release):
{
LIBS +=F:\opencv\cmake\install\x86\vc12\lib\opencv_core300d.dll \
       F:\opencv\cmake\install\x86\vc12\lib\opencv_calib3d300d.dll \
       F:\opencv\cmake\install\x86\vc12\lib\opencv_highgui300d.dll \
       F:\opencv\cmake\install\x86\vc12\lib\opencv_imgproc300d.dll\
       F:\opencv\cmake\install\x86\vc12\lib\opencv_objdetect300d.dll \
       F:\opencv\cmake\install\x86\vc12\lib\opencv_photo300d.dll \
       F:\opencv\cmake\install\x86\vc12\lib\opencv_video300d.dll \
       F:\opencv\cmake\install\x86\vc12\lib\opencv_videoio300d.dll \
       F:\opencv\cmake\install\x86\vc12\lib\opencv_flann300d.dll \
       F:\opencv\cmake\install\x86\vc12\lib\opencv_features2d300d.dll
}

CONFIG(release, debug|release):
{
LIBS +=F:\opencv\cmake\install\x86\vc12\lib\opencv_core300.dll\
       F:\opencv\cmake\install\x86\vc12\lib\opencv_calib3d300.dll\
       F:\opencv\cmake\install\x86\vc12\lib\opencv_highgui300.dll\
       F:\opencv\cmake\install\x86\vc12\lib\opencv_imgproc300.dll\
       F:\opencv\cmake\install\x86\vc12\lib\opencv_objdetect300.dll\
       F:\opencv\cmake\install\x86\vc12\lib\opencv_photo300.dll\
       F:\opencv\cmake\install\x86\vc12\lib\opencv_video300.dll\
       F:\opencv\cmake\install\x86\vc12\lib\opencv_videoio300.dll\
       F:\opencv\cmake\install\x86\vc12\lib\opencv_flann300.dll\
       F:\opencv\cmake\install\x86\vc12\lib\opencv_features2d300.dll
}
```

#### 参考

[opencv + cmake + vs2010 配置过程][1]

[Cmake配置opencv][2]

[1]: http://blog.sina.com.cn/s/blog_8b6c17eb0101l7zd.html
[2]: https://wenku.baidu.com/view/f8dec74902768e9950e73808.html

