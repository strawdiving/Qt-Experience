## Qt与COM交互

以Qt调用TTS朗读文本为例。

**需求：** 利用TTS朗读文本

**方法：** 必须借助Com组件，QAxObject调用windows的SAPI

**环境：**  Win764bit，Qt 5.4.1，Qt Creator

### 原理

Qt中提供了QtActiveX模块来支持微软ActiveX的开发，ActiveQt包含了两个组件QAxContainer和QAxServer。Qt的ActiveX和COM的开发支持两种方式：

- 支持将已有的COM或者ActiveX空间引入到Qt的应用程序中。

  QAxContainer允许我们使用COM对象。通过QAxObject和QAxWidget分别支持COM对象和ActiveX控件的开发，可以通过这两个对象将外部的COM或者ActiveX组件接入到Qt应用程序。

- 支持将Qt应用程序或者Qt的对象导出成COM对象或者ActiveX控件供他人使用。

  QAxServer可以将我们写的Qt控件导出为COM对象或者是ActiveX控件。通过QAxAggregated、QAxBindable和QAxFactory类，通过了进程内和可执行程序exe两种方式的COM Server模式，用来将Qt写的内容导出为COM或者ActiveX供他人使用。

  ![Qt&COM]()

QAxContainer是由三个类组成的。分别是：

- QAxObject，封装了COM对象
- QAxWidget，封装了ActiveX控件
- QAxBase，是QAxObject和QAxWidget的父类，它实现了封装COM的核心函数。

###  使用QtActiveX创建COM或ActiveX Server

COM（Component Object Model）是微软提出的一种技术，它定义了一种规范，通过COM可以轻松实现一种语言（如C#）调用另一种语言（如C++、VB等）开发的功能模块。

ActiveX是微软主要针对互联网客户端设计的以COM为技术基础的一种实现。

一般来说二者并没有本质的区别，仅有一些概念上的差异，一般来说：

1. ActiveX一般包含一个窗体界面，COM对象一般并没有界面

2. COM对象一般作为一个可调用的模块来使用，ActiveX一般嵌入在网页中使用

上述仅仅是一种使用上的惯例，但是并未强制一定这样。

#### 流程

1. 创建一个QAxObject对象

```c++
mutable QAxObject *voiceObj;
```

2. 通过setControl设置这个ActiveX控件的class_id

```c++
voiceObj->setControl("sapi.spvoice");
```

这时就会调用CoCreateInstance创建ActiveX控件的实例。这时这个ActiveX控件的所有的属性、方法、事件将通过QAxWidget转换为Qt的properties、signals和slots。

既然可以调用ActiveX控件的方法属性，还需要 **COM中的数据类型和Qt中使用的数据类型的转换** 。下面就是COM中数据类型和Qt中的数据类型对应的表格：

![types](types.jpeg)

Qt的ActiveX框架会将Qt类中的要素转换为COM中的标准要素供其他调用者使用，具体来说：

- Qt类中的属性和公有的槽函数(slots)会被转换为COM中的属性和方法
- Qt类中的信号(signals)会被转换成为COM组件中的事件

3. 通过dynamicCall方法来调用ActiveX控件的方法, 通过dynamicCall调用COM对象的方法的时候需要提供完整的函数声明。

```c++
voiceObj->dynamicCall("Speak(QString,uint)",text,1);
activeX->dynamicCall("Navigate(const QString&)", "qt.nokia.com");
```

例： 在Qt中使用shockwaveFlash这个ActiveX控件

```c++
#include <QApplication>
#include <QtGui>
#include <QAxWidget>
int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
     QAxWidget *flash = new QAxWidget(0,0);
    flash->setControl(QString::fromUtf8("{d27cdb6e-ae6d-11cf-96b8-444553540000}"));
    flash->dynamicCall("LoadMovie(long,string)",0,"c:/1.swf");
    flash->show();
    return a.exec();
}
```

### 方法

1. QAxContainer不包含在QtCore里面，所以要使用QAxContainer的话还必须要在pro文件里加上

```c++
QT += axcontainer
```

2. 头文件中：添加需要的头文件，并声明QAxObject对象和朗读函数

   ```c++
   #include <ActiveQt/QAxObject>
   ...
   void say(QString text);
   mutable QAxObject *voiceObj;
   ...
   ```

3. 源文件中：初始化QAxObject对象，并定义朗读函数

```c++
voiceObj = new QAxObject();
...
void Audio_Worker::say(QString text)
{
    voiceObj->setControl("sapi.spvoice");
    voiceObj->dynamicCall("Speak(QString,uint)",text,1);
}
```

4. 中、英文朗读

Win7语音库，

- lili中文女声且支持中英文混读（默认），CLSID: {F51C7B23-6566-424C-94CF-2C4F83EE96FF}
- Anna英文女声，CLSID: {F51C7B23-6566-424C-94CF-2C4F83EE96FF}

使用lili中文女声朗读中文：

读中文时，可能会遇到QString str(tr(“中文”))无效，一般改为QString::fromLocal8Bit(“中文”)。

如遇到错误： **errorC2001：常量中有换行符**

**原因解析：** 中文字符引起，只有部分使用中文的语句会提示该错误

**解决办法：** 中文的个数为偶数个，或结尾加英文符号，或字符转换GBK To UTF8

**注：** 每个系统设置不同，使用中文的时候需要注意。

### 参考

[Qt中使用ActiveX（一）][1]

[在Qt中使用ActiveX控件][2]

[QT TTS中、英文朗读][3]

[1]: http://blog.csdn.net/csxiaoshui/article/details/47333989
[2]: http://blog.csdn.net/tingsking18/article/details/5403038
[3]: http://blog.csdn.net/li235456789/article/details/50767262

