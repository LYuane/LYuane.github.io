# 工作流DB空间专项治理

DB空间优化方案

- 重构工作流存储逻辑，通过工厂模式和责任链将各种方案链式调度
- 在1的基础上，实现context jsondiff优化
- 大context分表存储
- 新加工作流模型的标签解析，检测到该标签时不存储该工作流/活动。
- 业务侧推动合理使用工作流

未来优化建议

- binlog消费
- 对于一个Activity不存
- 开关存储优化
- 提供spring注解，对于注解的字段不存储



## 重构工作流存储逻辑 

方案1:

直接修改DBManger



### 方案2

通过工厂模式和责任链将各种方案链式调度，可以低成本的进行方案选择和迭代

初始化PersistFactory：读用户配置-->初始化DB Persist--> 根据用户写配置链式构造PersistChain-->注入到DBManger

![未命名文件 (1)](https://github.com/LYuane/LYuane.github.io/tree/master/doc/images/1.png)



### 方案3

进一步抽象出CustomerActivity，规范用户实现接口。

![未命名文件 (2)](https://github.com/LYuane/LYuane.github.io/tree/master/doc/images/2.png)