## Qt与JS交互

### 问题

**web应用不能操作本地文件，这是不安全，不允许的** 。Javascript前端无法操作本地文件，无法将网页的内容导出保存。

IE能通过Active控件操作本地文件，这其实是一种退步。

html5 提供了filesystem的api，也是向系统申请一块空间，只能作为沙盒，可以由你进行存储应用数据。也就是说在沙盒里面可以任意操作，可以读取本地的文件内容。但其他应用是不能访问的，是独立的一块空间。

考虑到各个浏览器的兼容性，要向本地写文件的需求，以及生成的代码的可访问性。

### 解决

**Qt可以利用javascript访问网页元素**

注： Qt 5.4以前用webkit库，之后逐渐转到了QWebEngine，但5.8 MSVC版本中默认的没有QWebEngine库。

C++中初始化了一个webengine（qt的web引擎组件已经基于chromium了）将ui组件和WebEngineView和WebEngine绑定， 之后创建了一个QWebChannel对象， 并通过WebChannel对象注册了一个名字为content的对象， 并绑定一个MsgHandler的Qobject的对象用来处理和页面的交互。

例：

1. 建一个类，表示要和JS交互的对象

```c++
// document.h
class document: public QObject
{
    Q_OBJECT                             
public:
public slots:
	void receiveText(const QString &r_text);
signals:
    void sendText(const QString &text);
};
```

```c++
// document.cpp
void document::receiveText(const QString &r_text)
{
    qDebug()<<(QObject::tr("Received message: %1").arg(r_text));
}
```

2. 建立通道，从QT传类对象到JS

```c++
// mainwindow.cpp
// web引擎可以显示网页
QWebEngineView *view = new QWebEngineView(parent);
view->load(QUrl("file:///E:/Web%20Blockly/test/blockly/tests/playground.html"));
ui->gridLayout1->addWidget(view,1,1,1,1);

// 通过QWebChannel传递给Javascript的对象
m_content = new document(this);

// 辅助Qt和javascript交互的类对象，从Qt传类对象到JS
QWebChannel *channel = new QWebChannel(this);

// 注册对象 id "content"和js中的对象名一致,即将m_content对象传递给JS，在JS中名称是content
channel->registerObject(QStringLiteral("content"), &m_content);
view->page()->setWebChannel(channel);
```

3.  JS文件里，把对象赋值到JS中，和qt进行信号与槽的交互

```javascript
//send generated command to Qt
function sendCommand() {
    var textarea = document.getElementById('importExport');
    var text = textarea.value;
    new QWebChannel(qt.webChannelTransport,function(channel){
    //make dialog object accessible globally
    //把对象赋值到JS中
    var content = channel.objects.content;
    //receiveText 是一个c++中的slot 函数
    content.receiveText(text);
  });
}			
content.sendText.connect(function(message) {           
//连接QT中发出的信号，里面的message是参数;只需要在QT中调用此函数也就是sendText就可以调用JS函数
     alert("Received message: " + message);    
}
```

js文件中发生错误：  **error: js:Uncaught ReferenceError: qt is not defined ** 

解决： 在javascript程序里，要加入

<script
src="qrc:///qtwebchannel/qwebchannel.js"></script> 

![mainwindow]()

![test]()

