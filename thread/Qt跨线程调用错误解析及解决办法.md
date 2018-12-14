## Qt跨线程调用错误解析及解决办法

**错误提示：Error: Cannot create children for a parent that is in a different thread.**

### 错误案例分析

新建SerialLink子线程，继承QThread，并重写它的run()，调用 start()函数时自动调用重载的run()函数。在主线程中创建SerialLink类的对象。

串口_port在SerialLink的头文件中定义，在_hardwareConnect()函数中初始化。在_connect()函数中调用start()函数启动线程，自动调用重载的run()函数。

```c++
Class SerialLink: public QThread
{
……
QSerialPort* _port;
 ……
}
```

```c++
void SerialLink:: _hardwareConnect( )
{
……
_port = new SerialPort( );
……
} 
void SerialLink:: _connect( )
{
……
_hardwareConnect( );
start( );
……
} 
void SerialLink:: run( )
{
……
}
```

该错误是 **跨线程调用** 引起的。

**原因：** 

_ port 的初始化是在SerialLink类中，SerialLink子线程是在主线程中创建的子线程，SerialLink对象本身工作在主线程下，其中定义的所有东西都属于创建SerialLink的线程，即主线程，所以 _ port应该是属于主线程（父线程）的；

只有run()范围内的代码工作在次线程中，所以在SerialLink的run()中调用_port属于跨线程调用，就会出现异常。

**解决办法： **

**将_ port在run函数中初始化，** _ port就是在同一子线程中创建和调用，不会出现跨线程调用错误。

```c++
void SerialLink* run( )
{
	_hardwareConnect( );
	……
}
void SerialLink* _connect( )
{
	……
	start( );
	……
}
void _hardwareConnect( )
{
	……
	_port = new SerialPort( );
	……
}
```

### 总结

继承自QThread实现的线程本身工作在主线程下，在类中定义的对象或对象的指针都是属于主线程的，即使调用其定义的变量和方法，也是工作在主线程下。线程对象的this指针也是属于主线程的。

子线程真正意义上的实体内容是在run()函数中实现的，也就是说，只有run()函数内的代码是在子线程中执行的。为避免跨线程调用引起异常，一个对象的创建和调用要放在同一线程中。

推荐的工作方式：利用Qt的事件驱动特性，将需要在次线程中处理的业务放在独立的模块（类）中，由主线程创建完该对象后，将其移交给指定的线程（ **moveToThread** ），且可以将多个类似的对象移交给同一个线程。

**connect连接类型：**

1. 自动连接(AutoConnection)，默认的连接方式，如果信号与槽，也就是发送者与接受者在同一线程，等同于直接连接；如果发送者与接受者处在不同线程，等同于队列连接。

2. 直接连接(DirectConnection)，当信号发射时，槽函数立即直接调用。无论槽函数所属对象在哪个线程，槽函数总在发送者所在线程执行。

3. 队列连接(QueuedConnection)，当控制权回到接受者所在线程的事件循环时，槽函数被调用。槽函数在接受者所在线程执行。

在GUI程序中，主线程也叫GUI线程，因为它是唯一被允许执行GUI相关操作的线程。对于一些耗时的操作，如果放在主线程中，就会出现界面无法响应的问题。这种问题的一种解决方式是，把这些耗时操作放到子线程中。