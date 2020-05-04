# XML处理

XML作为一种数据交换和信息传递的格式是很普遍的，使用一系列简单的标记来描述数据，在web开发中扮演着重要的角色。

xml示例：
```xml
<?xml version="1.0" encoding="utf-8"?>

<servers version="2.0"> 
  <server> 
    <serverName>dazhairen.com</serverName>  
    <serverIp>127.0.0.1</serverIp> 
  </server>  
  <server> 
    <serverName>go.dazhairen.com</serverName>  
    <serverIp>127.0.0.1</serverIp> 
  </server> 
</servers>
```
上面示例中，描述了两个服务器信息，包含服务器名称和服务器IP。配置文件定义了这些信息，程序中也就可以针对这些xml描述的信息进行一系列的操作。
