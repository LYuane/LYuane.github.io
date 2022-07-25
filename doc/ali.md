# 工作流DB空间专项治理

## 整体方案

DB空间优化方案

1. **大context分表存储**
2. **重构工作流存储逻辑，通过工厂模式和责任链将各种存储方案链式调度**
3. **在2的基础上，实现context jsondiff优化**
4. **新加工作流模型的标签解析，检测到该标签时不存储该工作流/活动。**
5. **推动业务端合理使用工作流**

未来优化建议

- binlog消费
- 对于一个Activity不存
- 开关存储优化
- 提供spring注解，对于注解的字段不存储



## 1. 大context分表存储

### 背景

部分context字段过大，存到数据库text字段时出现异常

### 目标

解决大context导致工作流无法正常执行的问题

减少对工作流框架性能、稳定性的影响

### 方案

❌将原有字段text改为mediumtext： **表碎片问题，锁表不可接受**。

&#x2705; 增加关联表：有利于保护非大context工作流，业务不受影响。  增加框架逻辑复杂度。



## 2. 重构工作流存储逻辑 

### 目标

1.使用户可以自定义存储方案。

2.工作流引擎提供默认存储方案。 

 3.便于客户进行存储方案选择 

4.便于维护和迭代



### 设计1

通过**工厂模式和责任链模式**将各种方案链式调度，可以低成本的进行方案选择和迭代

初始化PersistFactory：读用户配置-->初始化DB Persist--> 根据用户写配置链式构造PersistChain-->注入到DBManger

![未命名文件 (1)](https://github.com/LYuane/LYuane.github.io/blob/master/doc/images/1.png)



### 设计2

进一步抽象出CustomerActivity，规范用户实现接口。

![未命名文件 (2)](https://github.com/LYuane/LYuane.github.io/blob/master/doc/images/2.png)



## 3.实现context jsondiff优化

在1的基础上对ActivityContext做diff操作再进行存储



