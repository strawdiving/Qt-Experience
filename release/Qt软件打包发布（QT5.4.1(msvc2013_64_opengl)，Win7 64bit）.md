## Qt软件打包发布（QT5.4.1(msvc2013_64_opengl)，Win7 64bit）

环境：QT5.4.1(msvc2013_64_opengl)，Win7 64bit

### 编译方式

Qt开发的程序发布的时候经常采用两种方式：1）静态编译，可生成单一的可执行文件；2）动态编译，需同时附上需要的dll文件。

#### 静态编译

静态编译，是指把相关的库也一并引入exe文件，这样程序的尺寸就会很大，不过程序发布就会变得简单很多。

#### 动态编译（Qt默认）

动态编译，是指相关的库，以dll动态链接库的形式引用。动态编译的exe程序比较小，因为相关的库都没有包含进来。所以程序发布的时候要把相关的库也一并发布出去。

一般使用动态编译动态链接Qt库，尤其代码规模比较大，需要多人协作开发时，不同模块按dll划分比较方便，采用静态链接是不现实的。

#### Debug版本

Debug 通常称为调试版本，它包含调试信息，并且不作任何优化，便于程序员调试程序。

#### Release 版本

Release 称为发布版本，它往往是进行了各种优化，使得程序在代码大小和运行速度上都是最优的。一般来说，release版的可执行程序体积要比debug版小很多，而且由于剥离了许多调试信息及符号等，运行效率相对也高一些，因此一般采用release编译。 

### 打包发布

本文采用Qt动态编译，release版本的程序。需要将相应的dll跟Qt可执行程序exe文件放在一个目录下。如下例所示：

![dll文件](https://github.com/strawdiving/)

所需的dll文件包括：

1. C Runtime库msvcrt，使用VC编译的C或C++程序，都需要相关的C runtime库才能运行，如该例程中的msvcp120.dll，msvcr120.dll等；
2. icudt53.dll、icuin53.dll、icuuc53.dll、Qt5Widgets.dll、Qt5Core.dll、Qt5Gui.dll等 （Qt 的bin目录中）动态引用的Qt库；
3. platforms、imageformats、audio等运行时加载的dll 文件夹；
4. 程序中引用的第三方库，如QWT，openCV，第三方库的dll文件如qwt.dll、opencv_world300.dll

将exe文件在另一台电脑上运行时，若缺少运行所必需的dll文件，会报错——缺少dll文件。

#### C Runtime库

问题比较多的是VC的运行时库 msvcrt。使用VC编译的C或C++程序，都需要相关的C runtime库才能运行。本文采用的是VS2013编译器，对应的就是MSVCR120。进入Microsoft.VC120.CRT 目录：${VS Install Dir}\VC\redist\x64\Microsoft.VC120.CRT，就能找到C runtime库（msvcp120.dll，msvcr120.dll，vccorlib120.dll）。

从vc2005开始微软加入了manifest机制控制运行时库的加载，如果用户机器上未安装过msvcrt的distribution pack，程序就不能运行。简单的处理方法是把C runtime库一并包含进去，即将Microsoft.VC120.CRT 目录下的文件（msvcp120.dll，msvcr120.dll，vccorlib120.dll）放到exe相同的目录即可。应用程序如果找不到系统安装的msvcrt，就会加载自带的库文件。 

#### Qt库

编译Qt后，将Qt生成路径（..\build-untitled-Desktop_Qt_5_4_1_MSVC2013_OpenGL_64bit-Release\release）中的exe程序放到新建的package中，Qt程序中使用到的（在.pro文件中添加的）QtCore，QtGUI，xml，sql，multimedia等相关的动态库和调用它的exe一起放在同一个目录中。

Qt的图片解码库比如jpeg、gif解码等是以插件形式存在的，要包含imageformats文件夹中的dll文件，还有windows平台相关的platforms，windows中的语音相关的audio等文件夹中包含的dll文件。 

对于采用动态编译的Qt可执行程序，如果不确定该程序使用了哪些必要的dll，可以使用工具查看该Qt可执行程序使用了哪些dll。

##### 工具

1. 查找程序运行依赖的dll文件

最简单的方式是用Qt自带的生成必备dll文件的windepolyqt工具：

**windepolyqt  xxx.exe**

如果将Qt的bin目录加入PATH环境，就可以直接在命令行使用windeployqt调用。将生成的xxx.exe可执行文件复制到一个空的文件夹里，进入这个文件夹 ，运行windeployqt xxx.exe，则该执行文件需要的大部分依赖文件都自动拷贝到这个文件夹里了。

![windeployqt](windeployqt.png)

如果还使用了其他的第三方的SDK，如QWT，OpenCV等，就需要手动将所需dll拷贝过来，如果不知道还需要哪些dll文件，可以用Dependency Walker (depends.exe)和微软的 procexp.exe 来查看程序运行时还缺少哪些dll。

![dependency walker](dependency walker.png)

2.  一个Qt的安装包制作工具，用户打包程序，变成（桌面）安装包，如开源工具Inno Setup

　这样之后，就得到了一个在其它没有安装Qt和VS的电脑上也可以运行的Qt程序安装包了。