###### Spring bean初始化步骤

userService.class->反射调用无参构造方法->普通对象->依赖注入->执行aftgerpropertiesset方法->aop->放入容器map<对象名,对象>

###### Spring初始化bean用类的哪个构造方法？

有无参的构造方法，就用无参的；没有无参的，有唯一的一个有参构造方法，就用唯一的；有多个有参的构造方法，直接报错，找不到用哪个构造方法初始化；有多个有参的构造方法，用户在有参构造方法上加autowired注解，就用这个指定的构造方法。

###### 单例bean是真的一个类型只有一个吗？

单例bean是对象名相同的只能有一个，orderService名的只有一个，orderService1名的只有一个。例如可以写几个返回OrderService的方法，加上bean注解后，都可以实例化。

###### Spring创建一个依赖对象bean的方式

先通过依赖对象的类来找，找到只有一个bean，则依赖对象找到；找到多个bean，则根据依赖对象的属性名来找，相同属性名的是匹配的；找不到则会报错。

Spring怎么注入一个属性对象

set注入也可以加autowired、构造方法、属性加autowired

###### @resource是先按照name找bean，找不到再根据类找。
