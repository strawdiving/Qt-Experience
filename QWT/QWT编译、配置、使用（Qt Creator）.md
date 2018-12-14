## QWT编译、配置、使用（Qt Creator）

环境： **Win7 32 bit / Qt Creator 3.3.1 / Qt 5.4.1 (msvc2013_opengl, 32 bit) / QWT 6.1.2**

QWT， Qt Widgets for Technical Applications，是一个基于LGPL版权协议的开源项目，可生成各种统计图。它为具有技术专业背景的程序提供GUI组件和一组实用类，其目标是以基于2D方式的窗体部件来显示数据，数据源以数值，数组或一组浮点数等方式提供， 输出方式可以是Curves（曲线），Slider（滚动条），Dials（圆盘），Compasses（仪表盘）等等。该工具库基于Qt开发，所以也继承了Qt的跨平台特性。

![QWT](QWT.png)

### 下载安装

QWT官方网址：[QWT官方网址][1]

稳定版下载地址： [稳定版下载地址][2]

[1]: http://qwt.sourceforge.net/
[2]: http://sourceforge.net/projects/qwt/files/qwt

选择.zip文件下载，解压

![qwt-6.1.2](qwt-6.1.2.png)

- designer文件夹： qwt插件的源码，用于生成Qt Designer插件，插件可以在Qt Designer中直接拖拽使用
- doc文件夹：帮助文档
- examples文件夹： qwt的示例（源码、可执行程序）, 这些工程的生成需要src或designer目录下工程生成的qwt.lib/qwt.dll
- src文件夹： qwt的源码
- textengines目录：存放数学指标语言的文本驱动引擎代码
- pro等工程文件等。

使用Qt Creator打开qwt.pro，进行编译（qmake->build），编译完后会在lib文件夹下生成 **qwt.dll** 和 **qwt.lib（release版）** ， 以及 **qwtd.dll** 和 **qwtd.lib（debug版）** 。

![build](build.png)

同时会生成qt creator使用的插件 **qwt_designer_plugin.dll** 和 **qwt_designer_plugin.lib** 。

![plugin](plugin.png)

### 配置

1. 本例【QT安装目录】为D:\Qt\Qt5.4.1\5.4\msvc2013_opengl
2. 将 **qwtd.dll、qwt.dll** 拷贝到【QT安装目录】\bin下，将 **qwtd.lib、qwt.lib** 拷贝到【QT安装目录】\lib下。
3. 将 **qwt_designer_plugin.dll** 和 **qwt_designer_plugin.lib** 拷贝到【QT安装目录】\include目录下。
4.  将解压得到的qwt-6.1.2\src文件夹拷贝到【QT安装目录】\include目录下，改名为 QtQWT。

### 使用

到这里，就基本配置完成了。

在Creator中新建带GUI的Qt项目，使用qwt插件和基类完成图表类设计。

Qwt的基类有以下几个：

- QwtAbstractScale： 包含刻度尺的所有类的抽象基类
- QwtAbstractScaleDraw：绘制刻度尺的抽象基类
- QwtAbstractSlider：滑块部件的抽象基类
- QwtAnalogClock：时钟的模拟类
- QwtArrayData：包含2个QwtArray<double>实例的数据类
- QwtArrowButton：箭头按钮
- QwtClipper：剪贴板类
- QwtColorMap：提供数值到颜色的映射功能
- QwtCompass：指南针部件
- QwtCompassMagnetNeedle：指南针部件的磁针
- QwtCompassRose：罗盘部件的抽象基类
- QwtCompassWindArrow：风向标的指示器

注：

需要在pro中进行配置：

1.   **LIBS += -L"D:/Qt/Qt5.4.1/5.4/msvc2013_opengl/lib" –lqwtd**

   **或 LIBS += -L"D:/Qt/Qt5.4.1/5.4/msvc2013_opengl/lib" -lqwt** 

2.   **INCLUDEPATH += D:/Qt/Qt5.4.1/5.4/msvc2013_opengl/include/QtQwt** 

然后就可以在Designer中进行设计了。如果直接双击打开.ui文件，找不到qwt插件，则选择用Qt Designer打开。

![open-ui](open-ui.png)

左边栏中出现Qwt的插件，可以直接拖拽使用。

![qwt-plugin](qwt-plugin.png)

#### 参考：

[QWT编译、配置、使用（Qt Creator）][1]

[ 详解 Qwt 安装、使用、示例][2]

[WIN7 下 Qt Creator 安装 QWT][3]

[1]: http://blog.sina.com.cn/s/blog_a6fb6cc90102v25w.html
[2]: http://blog.csdn.net/ymc0329/article/details/7865339
[3]: http://blog.chinaunix.net/uid-26815567-id-4064185.html

