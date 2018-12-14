**QJsonDocument:: fromJson**，返回一个QJsonDocument对象

Parses a UTF-8 encoded JSON document （此处是.px4文件）and creates a QJsonDocument from it.

调用QJsonDocument对象的object()，返回一个QJsonObject对象，

调用QJsonObject的contains()，value()方法处理JSon文档