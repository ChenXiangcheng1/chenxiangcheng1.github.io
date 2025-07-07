

# OOP面向对象编程



## POJO

plain ordinary java object 普通 Java 对象，作为 PO/VO/DTO 使用。声明周期由new创建，由 GC 回收。

* 没有继承任何类
* 没有实现任何接口
* 不包含业务逻辑



**PO**(persistant object)

持久化对象，对应数据库中的数据实体作为与数据库交互时的中间对象，可以带有各种逻辑操作。生命周期存在于与数据库连接后由数据库 insert 创建，由 delete 删除。

* POJO+持久化方法(Insert、Update、Delete…)



**VO**(view object)

视图，作为 UI Model 对象

* POJO+数据绑定功能



**DTO**(data transform object)

数据转换对象

* POJO+业务逻辑方法



**JavaBean**

符合特定规范的 Java 类

* 所有属性为 private
* 提供无参构造方法
* 提供 getter 和 setter
* 实现 serializable 接口



**Spring Bean**

符合 Spring 规范的 Java 类



