

### 设计1

通过**工厂模式和责任链模式**将各种方案链式调度，可以低成本的进行方案选择和迭代

初始化PersistFactory：读用户配置-->初始化DB Persist--> 根据用户写配置链式构造PersistChain-->注入到DBManger

![未命名文件 (1)](https://github.com/LYuane/LYuane.github.io/blob/master/doc/images/1.png)



### 设计2

进一步抽象出CustomerActivity，规范用户实现接口。

![未命名文件 (2)](https://github.com/LYuane/LYuane.github.io/blob/master/doc/images/2.png)






