### ini文件

.ini 文件是Initialization File的缩写，即初始化文件。

除了windows现在很多其他操作系统下面的应用软件也有.ini文件，用来配置应用软件以实现不同用户的要求。一般不用直接编辑这些.ini文件，应用程序的图形界面即可操作以实现相同的功能。它可以用来存放软件信息,注册表信息等。

#### ini文件格式

INI文件由节、键、值组成。 

节[section] 

参数（键=值）

name=value

下面是一个ini文件的例子　　

```c++
[Section1 Name]
KeyName1=value1
KeyName2=value2
... 　　
[Section2 Name]
KeyName21=value21
KeyName22=value22
```

其中：[Section1 Name]用来表示一个段落。因为INI文件可能是项目中共用的，所以使用[Section Name]段名来区分不同用途的参数区。例如：[Section1 Name]表示传感器灵敏度参数区；[Section2 Name]表示测量通道参数区等等。

注解：使用分号表示（;）。在分号后面的文字，直到该行结尾都全部为注解。

### Qt写ini文件

```c++
	#include <QtCore/QCoreApplication>  
	#include <QSettings>  
	int main(int argc, char *argv[])  
	{  
	   QCoreApplication a(argc, argv);  
	   //Qt中使用QSettings类读写ini文件  
	   //QSettings构造函数的第一个参数是ini文件的路径,第二个参数表示针对ini文件,第三个参数可以缺省  
	   QSettings *configIniWrite = new QSettings("hahaya.ini", QSettings::IniFormat);  
	   //向ini文件中写入内容,setValue函数的两个参数是键值对  
	   //向ini文件的第一个节写入内容,ip节下的第一个参数  
   configIniWrite->setValue("/ip/first", "192.168.0.1");  
	   //向ini文件的第一个节写入内容,ip节下的第二个参数  
	   configIniWrite->setValue("ip/second", "127.0.0.1");  
	   //向ini文件的第二个节写入内容,port节下的第一个参数  
	   configIniWrite->setValue("port/open", "2222");  
	   //写入完成后删除指针  
	   delete configIniWrite;  
	     
	   return a.exec();  
	}  
```

运行程序后，打开程序目录下的hahaya.ini文件，结果如下图所示：

![1544775775553](C:\Users\wurenji.ZKXS\AppData\Roaming\Typora\typora-user-images\1544775775553.png)

### Qt读ini文件

```c++
1.	#include <QtCore/QCoreApplication>  
2.	#include <QSettings>  
3.	#include <QString>  
4.	#include <QDebug>  
5.	int main(int argc, char *argv[])  
6.	{  
7.	   QCoreApplication a(argc, argv);  
8.	  
9.	   QSettings *configIniRead = new QSettings("hahaya.ini", QSettings::IniFormat);  
10.	   //将读取到的ini文件保存在QString中，先取值，然后通过toString()函数转换成QString类型  
11.	   QString ipResult = configIniRead->value("/ip/second").toString();  
12.	   QString portResult = configIniRead->value("/port/open").toString();  
13.	   //打印得到的结果  
14.	   qDebug() << ipResult;   // "127.0.0.0.1"
15.	   qDebug() << portResult;  // "2222"
16.	   //读入入完成后删除指针  
17.	   delete configIniRead;  
18.	   return a.exec();  
19.	}  

```

### QSettings用法总结

用户对应用程序经常有这样的要求：要求它能记住它的settings，比如窗口大小，位置，一些别的设置，还有一个经常用的，就是recentfiles，等等这些都可以通过Qsettings来实现。

   我们知道，这些settings一般都是存在系统里的，比如windows一般都写在系统注册表或者写INI文件，mac系统一般都在XML文件里，那么按照一般的标准来说，许多应用程序是用INI文件来实现的。而Qsettings就是提供了一种方便的方法来存储和恢复应用程序的settings。

   QSettings的API是基于Qvariant，Qvariant是一种数据类型的集合，它包含了大部分通常的Qt数据类型，比如QString，QRec，QImage，等等。

   当我们创建一个Qsettings的对象时，我们需要传递给它两个参数，第一个是你公司或者组织的名称，第二个事你的应用程序的名称。比如：

   Settings = Qsettings(“MySoft”,”QtPad”)

   公司名称：MySoft，程序名称：QtPad

   假如我们在应用程序中多次要用到Qsettings，为了简单其间，我们可以在主程序中先如下声明。

```c++
QtCore.QCoreApplication.setOrganizationName("MySoft")
QtCore.QCoreApplication.setOrganizationDomain("mysoft.com")
QtCore.QCoreApplication.setApplicationName("QtPad")
```

然后在应用程序的任何地方想要声明一个Qsettings类型的变量，便不需要书写两个参数了，直接用settings = Qsettings即可。

那么如何用它来保持应用程序的settings信息呢？我们以字典数据类型与之类比，它也有key，以及对应的value。比如下面例子：

```c++
settings= Qsettings(“MySoft”,”QtPad”)
Mainwindow = QmainWindow()
   settings.setValue(“pos”,QVariant(Mainwindow.pos())
settings.setValue(“size”,QVariant(Mainwindow.size())
```

上面两句就是把当前窗口的位置，和大小两个信息记录到了settings中，其中的key就是”pos”和”size”两个Qstring类型，而它所对应的值就是QVariant类型的。当然如果我们要写的key已在settings中存在的话，则会覆盖原来的值，写入新值。

如何读取Qsettings里的内容呢？如下：

```c++
Pos =settngs.value(“pos”).toPoint()
Size =settings.value(“size”).toSize()
```

当然如果key所对应的value是int型的，也可toInt(),如果没有我们要找的key，则会返回一个nullQVariant 如果用toInt的话会得到0。

那么实际应用中我们一般会如下：

```c++
pos=settings.value("pos", QVariant(QPoint(200,200))).toPoint()
size=settings.value("size", QVariant(QSize(400,400))).toSize()
self.resize(size)
self.move(pos)
```

意思是，如果settings里有以前存下的(用setValue设置的)pos和size的值，则读取，如果没有，不会返回null，而会使用我们给它的起始值——defaultvalue——即应用程序第一次运行时的情况。

注意：因为QVariant是不会提供所有数据类型的转化的，比如有toInt(),toPoint(),toSize(),但是却没有对Qcolor，Qimage和Qpixmap等数据类型的转化，此时我们可以用QVariant.value（），具体参看QVariant模块说明。

#### Qsettings里常用的method

```c++
   Qsettings.annKeys(self) 返回所有的key，以list的形式
   Qsettings.applicationName(self) 返回应用程序名称
   Qsettings.clear(self)  清楚此settings里的内容
   Bool Qsettings.contains(self,key) 返回真，如果存在名为key的key
   Qsettings.remove(self, keyname) 清楚key及其所对应的value
   Qsetting.fileName()  返回写入注册表地址，或者INI文件路径

```

settings在应用程序关闭以后到底存到了什么地方呢

```
writeSettings中，后面加一句话：
   Print Settings.fileName()
   这个在windows下，默认Qsettings会打印出这个程序的系统注册表所在地：
   这个结果是：HKEY_CURRENT_USERSoftwareMySoftQtPad
```

由此我们可以看出，这个writesettings其实就是个写注册表的过程。

当然，我们也可以不写注册表，我们写ini文件：

```c++
settings =QSettings("./QtPad.ini", QSettings.IniFormat)

settings.setValue("pos", QVariant(self.pos()))

settings.setValue("size", QVariant(self.size()))

就会在当前文件夹下产生一个QtPad.ini文件，打开后文件内容为：

[General]

pos=@Point(200 200)

size=@Size(400 400)
```

## 保存应用程序设置（QSettings）

**1.   QSettings** **类**

 QSettings 提供保存应用程序当前设置的接口，可以方便地保存程序的状态，例如窗口大小和位置，选项的选中状态等等。

 在 Windows 系统中，程序程序的状态信息记录在注册表中；在 Mac OS X 系统上，这些信息记录在 XML 配置文件中；在 Unix 系统中，则使用 INI text 文件记录。QSettings 则是对这些技术的一个抽象，使得保存和取得应用程序的设置状态的只得独立于操作系统。

 QSettings 的 API 是基于 QVariant 类，当创建一个 QSettings 对象时，必须传递公司或组织的名称（QString）和应用程序的名称（QString）用于构造一个 QSettings 对象。

 **2.**   **使用** **QSettings**

 （1）构造一个 QSettings 对象

 QSettings settings("MySoft", "Star Runner") ;

 （2）添加一个设置到 settings 中

 程序的设置是以“key-value”的形式，保存在 QSettings 对象中的。其中，key 由一个 QString 类型定义，value 是由 QVariant 类型定义：

 settings.setValue( "editor/wrapMargin", 68 ) ;

​         /*  wrapMargin 是一个子 key

​         /*  如果存在相同的 key，那么已存在的 key 所对应的值将由新值代替

 （3）从 setttings 中取出设置

 同时也可以通过 key 从 settings 中取出值：

 int margin = settings.value( "editor/wrapMargin").toInt( ) ;

 **3.   QSettings** **的组织方式**

 （1）用“/”表示子 key

 QSettings 存储状态信息的形式是 key-value，其中 key 与文件路径这个概念是类似的，subkey 可以用定义文件路径的形式定义，例如 findDialog/ matchCase，其中 matchCase 就是一个 subkey；

 （2）使用 beginGroup( ) 和 endGroup( ) 

 void QSettings : : beginGroup( const QString &prefix ) 的作用是在当前的 group 后面加上 prefix。当前的 group 自动加到一个 QSettings 对象的尾部：

```c++
settings.beginGroup("mainwindow") ;
settings.setValue("size", win->size( ) ) ;
settings.setValue("fullScreen", win->isFullScreen( ) ) ;
settings.endGroup( ) ;
 
settings.beginGroup("outputpanel") ;
settings.setValue("visible", panel->isVisible( ) ) ;
settings.endGroup( ) ;
```

·      这样设置后，当前的 settings 对象看上去应该是这样的层次结构：

 mainwindow/ size

mainwindow/ fullScreen

outputpanel/ visible

（3）取得 key 与子 key

 QStringList QSettings : : childKeys( ) const 函数返回所有顶层 keys，组成一个 QStringList 作为一个返回值。例如：

```c++
QSettings settings ;
settings.setValue("fridge/color", Qt::white) ;
settings.setValue("fridge/size", QSize(32, 96) ) ;
settings.setValue("sofa", true) ;
settings.setValue("tv", false) ;
QStringList keys = settings.childKeys( ) ;
 那么这个 keys 中看上去应该是这样的：
 keys: [ "sofa", "tv" ]
 QStringList QSettings : : childGroups ( ) const 是返回所有包含有 key 的顶层 groups，组成一个 QStringList 作为返回值：
 QSettings settings ;
settings.setValue("fridge/color",Qt::white); 
settings.setValue("fridge/size",QSize(32,96)); 
settings.setValue("sofa",true); 
settings.setValue("tv",false);
QStringList groups = settings.childGroups() ;
 则 groups 看上去是：
groups : [ "fridge" ]
```

 **4.**   **保存和取得程序的设置**

 （1）在主窗口的构造函数中，readSettings( )

```c++
void MainWindow::readSettings()
{
   QSettings settings("Software Inc.", "Spreadsheet");   // 写入与读取的 settings 要一致
   restoreGeometry(settings.value("geometry").toByteArray());
   recentFiles = settings.value("recentFiles").toStringList();
   updateRecentFileActions();
   bool showGrid = settings.value("showGrid", true).toBool();
   showGridAction->setChecked(showGrid);
   bool autoRecalc = settings.value("autoRecalc", true).toBool();
   autoRecalcAction->setChecked(autoRecalc);
}
```

（2）在关闭主窗口时，writeSettings( )

```c++
void MainWindow::writeSettings()
{
   QSettings settings("Software Inc.", "Spreadsheet");

   settings.setValue("geometry", saveGeometry());
   settings.setValue("recentFiles", recentFiles);
   settings.setValue("showGrid", showGridAction->isChecked());
   settings.setValue("autoRecalc", autoRecalcAction->isChecked());
}
```

### 参考

[Qt读写ini文件][1]

[qt QSettings 用法总结][2]

[程序启动读取和关闭时保存应用程序设置(QSettings)][3]

[1]: http://blog.csdn.net/qiurisuixiang/article/details/7760828
[2]: http://blog.chinaunix.net/uid-11765716-id-3181163.html
[3]: http://www.cnblogs.com/cy568searchx/p/3653390.html