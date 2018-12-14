## moveToThread

### 用法

1. 新建Object对象_worker
2. 新建objThread线程_workThread
3. 将object对象迁移到objThread线程中（无需run函数重写）

连接object的信号与创建object对象的实例（_controller）的槽

_worker信号 ——>_ _ controller中的槽 ——> _controller发送信号——>主界面中的槽

该object的槽都运行在子线程objThread中

在使用moveToThread时，继承自Object的类都运行在子线程objThread中

**Caution: 'Can't move objects with parent to thread**