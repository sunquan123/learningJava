# Spring笔记

## Spring bean初始化步骤

userService.class->反射调用无参构造方法->普通对象初始化->实例化bean->属性值设置->检查aware接口实现->调用beanpostprocess的前置处理方法->执行afterPropertiesSet方法->调用自定义初始化方法->调用beanpostprocess的后置处理方法：aop->注册Destruction方法回调->放入容器map<对象名,对象>

### Spring初始化bean用类的哪个构造方法？

有无参的构造方法，就用无参的；没有无参的，有唯一的一个有参构造方法，就用唯一的；有多个有参的构造方法，直接报错，找不到用哪个构造方法初始化；有多个有参的构造方法，用户在有参构造方法上加autowired注解，就用这个指定的构造方法。

## 单例bean是真的一个类型只有一个吗？

单例bean是对象名相同的只能有一个，orderService名的只有一个，orderService1名的只有一个。例如可以写几个返回OrderService的方法，加上bean注解后，都可以实例化。

## Spring创建一个依赖对象bean的方式

先通过依赖对象的类来找，找到只有一个bean，则依赖对象找到；找到多个bean，则根据依赖对象的属性名来找，相同属性名的是匹配的；找不到则会报错。

## 如何理解 Spring Boot 中的 Starter

使用spring + springmvc使用，如果需要引入mybatis等框架，需要到xml中定义mybatis需要的bean。starter就是定义一个starter的jar包，写一个@Configuration配置类、将这些bean定义在里面，然后在starter包的META-INF/spring.factories中写入该配置类，springboot会按照约定来加载该配置类。开发人员只需要将相应的starter包依赖进应用，进行相应的属性配置（使用默认配置时，不需要配置），就可以直接进行代码开发，使用对应的功能了，比如mybatis-spring-boot--starter，spring-boot-starter-redis。

## 如何理解Spring IOC的设计方式？

所谓的IOC（inversion of control），就是控制反转的意思。何为控制反转？

在传统的软件设计中，应用程序通常控制着对象的创建和管理。例如，一个对象依赖另一个对象，那么它会直接new一个对象出来，这是控制流程的设计思想。

在Spring中，控制关系得到了反转。对象的控制权掌握在Spring容器中，容器负责对象的创建和管理，并在需要的时候将它们注入到容器中。

简单来说就是，我们原先创建对象会new一个出来，这个对象如果依赖其他对象，我们也需要new出来，并进行构造。这样循环往复，直到全部对象都构造完毕，我们才可以开始使用原来的第一个对象。

现在有了Spring容器，我们只需要给类加上相应注解@Component等等，并给依赖加上@Autowired即可注入依赖对象。

### 使用IOC的好处

使用者不需要关心依赖对象的实现细节，直接通过注解注入对象既可以开始使用。如果我依赖一个对象，还需要知道并构造全部的实现细节才可以使用，那相应的开发成本就要提高不少。

不用创建多个对象造成浪费。如果大部分场景下的对象只要单例即可，那每次依赖对象时都new一遍将造成浪费。Spring默认构造的bean都是单例的，引用时将不会浪费。

Bean的修改不需要依赖方感知。如果采用new的方式得到一个beanA，对beanA的修改还需要通知到所有依赖beanA的对象进行相应修改。但是如果采用IOC的方式，其他bean将不需要感知到这个改变。

### Spring IOC的启动流程

对于Spring IOC来说，在容器启动的时候，它会根据每个bean的设置将bean注入到Spring容器中，如果需要依赖其他的bean，Spring会从容器中直接拿对应的bean。即：

1. 从配置元数据中获取要DI的业务POJO（这里的配置元数据包括xml，注解，configuration类等）

2. 将业务POJO形成BeanDefinition注入到Spring Container中

3. 使用方通过ApplicationContext从Spring Container直接获取即可。

## 描述一下Spring AOP

AOP(Aspect-Oriented Programming)，即面向切面编程，用人话说就是把公共的逻辑抽出来，让开发者可以更专注于业务逻辑开发。

和IOC一样，AOP也指的是一种思想。AOP思想是OOP（Object-Oriented Programming）的补充。OOP是面向类和对象的，但是AOP则是面向不同切面的。一个切面可以横跨多个类和对象去操作，极大的丰富了开发者的使用方式，提高了开发效率。

譬如，一个订单的创建，可能需要以下步骤： 

1. 权限校验

2. 事务管理

3. 创建订单

4. 日志打印

如果使用AOP思想，我们就可以把这四步当成四个“切面”，让业务人员专注开发第三个切面，其他三个切面则是基础的通用逻辑，统一交给AOP封装和管理。

Spring AOP有如下概念（列举下，不用刻意记）：

| 术语                            | 翻译   | 释义                                                                                                      |
| ----------------------------- | ---- | ------------------------------------------------------------------------------------------------------- |
| Aspect                        | 切面   | 切面由切入点和通知组成，它既包含了横切逻辑的定义，也包括了切入点的定义。切面是一个横切关注点的模块化，一个切面能够包含同一个类型的不同增强方法，比如说事务处理和日志处理可以理解为两个切面。          |
| PointCut                      | 切入点  | 切入点是对连接点进行拦截的条件定义，决定通知应该作用于截哪些方法。（充当where角色，即在哪里做）                                                      |
| Advice                        | 通知   | 通知定义了通过切入点拦截后，应该在连接点做什么，是切面的具体行为。（充当what角色，即做什么）                                                        |
| Target                        | 目标对象 | 目标对象指将要被增强的对象，即包含主业务逻辑的类对象。或者说是被一个或者多个切面所通知的对象。                                                         |
| JoinPoint                     | 连接点  | 连接点是程序在运行时的执行点，这个点可以是正在执行的方法，或者是正在抛出的异常。因为Spring只支持方法类型的连接点，所以在Spring中连接点就是运行时刻被拦截到的方法。连接点由两个信息确定：<br/> |
| ●                             |      |                                                                                                         |
| 方法(表示程序执行点，即在哪个目标方法)          |      |                                                                                                         |
| <br/>●                        |      |                                                                                                         |
| 相对点(表示方位，即目标方法的什么位置，比如调用前，后等) |      |                                                                                                         |
| Weaving                       | 织入   | 织入是将切面和业务逻辑对象连接起来, 并创建通知代理的过程。织入可以在编译时，类加载时和运行时完成。在编译时进行织入就是静态代理，而在运行时进行织入则是动态代理。                       |

对于通知类型来说：

| Before Advice          | 连接点执行前执行的逻辑                     |
| ---------------------- | ------------------------------- |
| After returning advice | 连接点正常执行（未抛出异常）后执行的逻辑            |
| After throwing advice  | 连接点抛出异常后执行的逻辑                   |
| After finally advice   | 无论连接点是正常执行还是抛出异常，在连接点执行完毕后执行的逻辑 |
| Around advice          | 该通知可以非常灵活的在方法调用前后执行特定的逻辑        |

## Spring AOP如何实现的？

从Bean的初始化流程中来讲，Spring的AOP会在bean实例的实例化已完成，进行初始化后置处理时创建代理对象，即下面代码中的applyBeanPostProcessorsAfterInitialization部分。

```java
protected Object initializeBean(final String beanName, final Object bean, RootBeanDefinition mbd) {

    //...
    //检查Aware
    invokeAwareMethods(beanName, bean);

    //调用BeanPostProcessor的前置处理方法
    Object wrappedBean = bean;
    if (mbd == null || !mbd.isSynthetic()) {
        wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
    }

    //调用InitializingBean的afterPropertiesSet方法或自定义的初始化方法及自定义init-method方法
    try {
        invokeInitMethods(beanName, wrappedBean, mbd);
    }
    catch (Throwable ex) {
        throw new BeanCreationException(
                (mbd != null ? mbd.getResourceDescription() : null),
                beanName, "Invocation of init method failed", ex);
    }
    //调用BeanPostProcessor的后置处理方法
    if (mbd == null || !mbd.isSynthetic()) {
        wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
    }
    return wrappedBean;
}
```

applyBeanPostProcessorsAfterInitialization中会遍历所有BeanPostProcessor，然后调用其postProcessAfterInitialization方法，而AOP代理对象的创建就是在AbstractAutoProxyCreator这个类的postProcessAfterInitialization中：

```java
@Override
public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
    if (bean != null) {
        Object cacheKey = getCacheKey(bean.getClass(), beanName);
        if (this.earlyProxyReferences.remove(cacheKey) != bean) {
            return wrapIfNecessary(bean, beanName, cacheKey);
        }
    }
    return bean;
}
```

这里面最重要的就是wrapIfNecessary方法了：

```java
/**
 * 如果需要，对bean进行包装。
 *
 * @param bean 要包装的目标对象
 * @param beanName bean的名称
 * @param cacheKey 缓存键
 * @return 包装后的对象，可能是原始对象或代理对象
 */
protected Object wrapIfNecessary(Object bean, String beanName, Object cacheKey) {
    // 如果beanName不为null且在目标源bean集合中，则直接返回原始对象
    if (beanName != null && this.targetSourcedBeans.contains(beanName)) {
        return bean;
    }

    // 如果缓存键对应的值为Boolean.FALSE，则直接返回原始对象
    if (Boolean.FALSE.equals(this.advisedBeans.get(cacheKey))) {
        return bean;
    }

    // 如果bean的类型为基础设施类，或者应跳过该类型的代理，则将缓存键对应的值设置为Boolean.FALSE并返回原始对象
    if (isInfrastructureClass(bean.getClass()) || shouldSkip(bean.getClass(), beanName)) {
        this.advisedBeans.put(cacheKey, Boolean.FALSE);
        return bean;
    }

    // 如果存在advice，为bean创建代理对象
    Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(bean.getClass(), beanName, null);
    if (specificInterceptors != DO_NOT_PROXY) {
        // 将缓存键对应的值设置为Boolean.TRUE
        this.advisedBeans.put(cacheKey, Boolean.TRUE);
        // 创建代理对象
        Object proxy = createProxy(
                bean.getClass(), beanName, specificInterceptors, new SingletonTargetSource(bean));
        // 将代理对象的类型与缓存键关联起来
        this.proxyTypes.put(cacheKey, proxy.getClass());
        return proxy;
    }

    // 如果没有advice，将缓存键对应的值设置为Boolean.FALSE并返回原始对象
    this.advisedBeans.put(cacheKey, Boolean.FALSE);
    return bean;
}
```

createProxy的主要作用是根据给定的bean类、bean名称、特定拦截器和目标源，创建代理对象：

```java
/**
 * 根据给定的bean类、bean名称、特定拦截器和目标源，创建代理对象。
 *
 * @param beanClass 要代理的目标对象的类
 * @param beanName bean的名称
 * @param specificInterceptors 特定的拦截器数组
 * @param targetSource 目标源
 * @return 创建的代理对象
 */
protected Object createProxy(
        Class<?> beanClass, String beanName, Object[] specificInterceptors, TargetSource targetSource) {

    // 如果beanFactory是ConfigurableListableBeanFactory的实例，将目标类暴露给它
    if (this.beanFactory instanceof ConfigurableListableBeanFactory) {
        AutoProxyUtils.exposeTargetClass((ConfigurableListableBeanFactory) this.beanFactory, beanName, beanClass);
    }

    // 创建ProxyFactory实例，并从当前代理创建器复制配置
    ProxyFactory proxyFactory = new ProxyFactory();
    proxyFactory.copyFrom(this);

    // 如果不强制使用CGLIB代理目标类，根据条件决定是否使用CGLIB代理
    if (!proxyFactory.isProxyTargetClass()) {
        if (shouldProxyTargetClass(beanClass, beanName)) {
            proxyFactory.setProxyTargetClass(true);
        } else {
            // 根据bean类评估代理接口
            evaluateProxyInterfaces(beanClass, proxyFactory);
        }
    }

    // 构建advisor数组
    Advisor[] advisors = buildAdvisors(beanName, specificInterceptors);
    // 将advisors添加到ProxyFactory中
    proxyFactory.addAdvisors(advisors);
    // 设置目标源
    proxyFactory.setTargetSource(targetSource);
    // 定制ProxyFactory
    customizeProxyFactory(proxyFactory);

    // 设置代理是否冻结
    proxyFactory.setFrozen(this.freezeProxy);
    // 如果advisors已经预过滤，则设置ProxyFactory为预过滤状态
    if (advisorsPreFiltered()) {
        proxyFactory.setPreFiltered(true);
    }

    // 获取代理对象，并使用指定的类加载器
    return proxyFactory.getProxy(getProxyClassLoader());
}
```

Spring AOP 是通过代理模式实现的，具体有两种实现方式，一种是基于Java原生的动态代理，一种是基于cglib的动态代理。对应到代码中就是，这里面的Proxy有两种实现，分别是CglibAopProxy和JdkDynamicAopProxy。

Spring AOP默认使用标准的JDK动态代理进行AOP代理。这使得任何接口可以被代理。但是JDK动态代理有一个缺点，就是它不能代理没有接口的类。

所以Spring AOP就使用CGLIB代理没有接口的类。

## Spring Bean的生命周期是什么样的？

一个Spring的Bean从出生到销毁的全过程就是他的整个生命周期，那么经历以下几个阶段：

![](./pic/Spring/SpringBean生命周期.png) 

整个生命周期可以大致分为3个大的阶段，分别是：创建、使用、销毁。还可以进一步分为6个小的阶段：实例化、设置属性值、初始化、注册Destruction回调、Bean的正常使用以及Bean的销毁。

这里的小阶段如何划分只是我一家之言，实际上可以有所变动。这里重要的部分是理解Bean的各阶段执行内容，具体的概念划分并不重要。实际上在Spring代码中，Bean的创建阶段也是基于多个初始化方法来进行分层的。

具体到代码方面，可以参考以下这个更加详细的过程介绍，我把具体实现的代码位置列出来了。

1. 实例化Bean：
   
   - Spring容器首先创建Bean实例。
   
   - 在AbstractAutowireCapableBeanFactory类中的createBeanInstance方法中实现

2. 设置属性值：
   
   - Spring容器注入必要的属性到Bean中。
   
   - 在AbstractAutowireCapableBeanFactory类的populateBean方法中处理

3. 检查Aware：
   
   - 如果Bean实现了BeanNameAware、BeanClassLoaderAware等这些Aware接口，Spring容器会调用它们。
   
   - 在AbstractAutowireCapableBeanFactory的initializeBean方法中调用

4. 调用BeanPostProcessor的前置处理方法：
   
   - 在Bean初始化之前，允许自定义的BeanPostProcessor对Bean实例进行处理，如修改Bean的状态。BeanPostProcessor的postProcessBeforeInitialization方法会在此时被调用。
   
   - 由AbstractAutowireCapableBeanFactory的applyBeanPostProcessorsBeforeInitialization方法执行。

5. 调用InitializingBean的afterPropertiesSet方法：
   
   - 提供一个机会，在所有Bean属性设置完成后进行初始化操作。如果Bean实现了InitializingBean接口，afterPropertiesSet方法会被调用。
   
   - 在AbstractAutowireCapableBeanFactory的invokeInitMethods方法中调用。

6. 调用自定义init-method方法：
   
   - 提供一种配置方式，在XML配置中指定Bean的初始化方法。如果Bean在配置文件中定义了初始化方法，那么该方法会被调用。
   
   - 在AbstractAutowireCapableBeanFactory的invokeInitMethods方法中调用。

7. 调用BeanPostProcessor的后置处理方法：
   
   - 在Bean初始化之后，再次允许BeanPostProcessor对Bean进行处理。BeanPostProcessor的postProcessAfterInitialization方法会在此时被调用。
   
   - 由AbstractAutowireCapableBeanFactory的applyBeanPostProcessorsAfterInitialization方法执行

8. 注册Destruction回调：
   
   - 如果Bean实现了DisposableBean接口或在Bean定义中指定了自定义的销毁方法，Spring容器会为这些Bean注册一个销毁回调，确保在容器关闭时能够正确地清理资源。
   
   - 在AbstractAutowireCapableBeanFactory类中的registerDisposableBeanIfNecessary方法中实现

9. Bean准备就绪：
   
   - 此时，Bean已完全初始化，可以开始处理应用程序的请求了。

10. 调用DisposableBean的destroy方法：
    
    - 当容器关闭时，如果Bean实现了DisposableBean接口，destroy方法会被调用。
    
    - 在DisposableBeanAdapter的destroy方法中实现

11. 调用自定义的destory-method
    
    - 如果Bean在配置文件中定义了销毁方法，那么该方法会被调用。
    
    - 在DisposableBeanAdapter的destroy方法中实现

可以看到，整个Bean的创建的过程都依赖于AbstractAutowireCapableBeanFactory这个类，而销毁主要依赖DisposableBeanAdapter这个类。

AbstractAutowireCapableBeanFactory 的入口处，doCreateBean的核心代码如下，其中包含了实例化、设置属性值、初始化Bean以及注册销毁回调的几个核心方法。

```java
protected Object doCreateBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
            throws BeanCreationException {
        BeanWrapper instanceWrapper = null;
        if (instanceWrapper == null) {
            // 实例化bean
            instanceWrapper = createBeanInstance(beanName, mbd, args);
        }
        // ...
        Object exposedObject = bean;
        try {
            // 设置属性值
            populateBean(beanName, mbd, instanceWrapper);
            // 初始化bean
            exposedObject = initializeBean(beanName, exposedObject, mbd);
        }

        // ...
        try {
            // 注册bean的Destruction回调
            registerDisposableBeanIfNecessary(beanName, bean, mbd);
        }

        return exposedObject;
    }
```

而DisposableBeanAdapter的destroy方法中核心内容如下：

```java
@Override
    public void destroy() {
        if (this.invokeDisposableBean) {
            //...
            try {
                // 调用这个bean实现disposableBean的destory方法
                ((DisposableBean) this.bean).destroy();
            }
        //...
        // 调用自定义的destory-method
        if (this.destroyMethod != null) {
            invokeCustomDestroyMethod(this.destroyMethod);
        }
        else if (this.destroyMethodName != null) {
            Method destroyMethod = determineDestroyMethod(this.destroyMethodName);
            if (destroyMethod != null) {
                invokeCustomDestroyMethod(ClassUtils.getInterfaceMethodIfPossible(destroyMethod, this.bean.getClass()));
            }
        }
    }
```

## Spring Bean的初始化是什么样的？

前面已经谈到了Spring Bean的初始化可以分为6个小的阶段：实例化、设置属性值、初始化、注册Destruction回调、Bean的正常使用以及Bean的销毁。

我们再把初始化的的这个过程单独拿出来展开介绍一下。

首先先看一下初始化和实例化的区别是什么？

在Spring框架中，初始化和实例化是两个不同的概念：

实例化（Instantiation）：

- 实例化是创建对象的过程。在Spring中，这通常指的是通过调用类的构造器来创建Bean的实例。这是对象生命周期的开始阶段。对应doCreateBean中的createBeanInstance方法。

初始化（Initialization）：

- 初始化是在Bean实例创建后，进行一些设置或准备工作的过程。在Spring中，包括设置Bean的属性，调用各种前置&后置处理器。对应doCreateBean中的populateBean和initializeBean方法。

```java
protected Object doCreateBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
            throws BeanCreationException {
        BeanWrapper instanceWrapper = null;
        if (instanceWrapper == null) {
            // 实例化bean
            instanceWrapper = createBeanInstance(beanName, mbd, args);
        }
        // ...
        Object exposedObject = bean;
        try {
            // 设置属性值
            populateBean(beanName, mbd, instanceWrapper);
            // 初始化bean
            exposedObject = initializeBean(beanName, exposedObject, mbd);
        }

        // ...
        return exposedObject;
    }
```

下面是SpringBean的实例化+初始化的完整过程：

### 实例化Bean

Spring容器在这一步创建Bean实例。其主要代码在AbstractAutowireCapableBeanFactory类中的createBeanInstance方法中实现：

```java
protected BeanWrapper createBeanInstance(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) {
        // 解析Bean的类，确保Bean的类在这个点已经被确定
        Class<?> beanClass = resolveBeanClass(mbd, beanName);
        // 检查Bean的访问权限，确保非public类允许访问
        if (beanClass != null && !Modifier.isPublic(beanClass.getModifiers()) && !mbd.isNonPublicAccessAllowed()) {
            throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                    "Bean class isn't public, and non-public access not allowed: " + beanClass.getName());
        }

        Supplier<?> instanceSupplier = mbd.getInstanceSupplier();
        if (instanceSupplier != null) {
            return obtainFromSupplier(instanceSupplier, beanName);
        }
        // 如果Bean中指定了工厂方法，则使用工厂方法创建Bean实例
        if (mbd.getFactoryMethodName() != null) {
            return instantiateUsingFactoryMethod(beanName, mbd, args);
        }

        // 当重新创建相同的Bean时的快捷路径
        boolean resolved = false;
        boolean autowireNecessary = false;
        if (args == null) {
            synchronized (mbd.constructorArgumentLock) {
                // 如果构造方法或工厂方法已经被解析，直接使用解析结果
                if (mbd.resolvedConstructorOrFactoryMethod != null) {
                    resolved = true;
                    autowireNecessary = mbd.constructorArgumentsResolved;
                }
            }
        }
        if (resolved) {
            if (autowireNecessary) {
                // 如果需要自动装配构造函数参数，则调用相应方法进行处理
                return autowireConstructor(beanName, mbd, null, null);
            }
            else {
                // 否则使用无参构造函数或默认构造方法创建实例
                return instantiateBean(beanName, mbd);
            }
        }

        // 通过BeanPostProcessors确定构造函数候选
        Constructor<?>[] ctors = determineConstructorsFromBeanPostProcessors(beanClass, beanName);
        // 如果bean有合适的构造函数或需要通过构造函数自动装配，则主动使用相应的构造函数创建实例
        if (ctors != null || mbd.getResolvedAutowireMode() == AUTOWIRE_CONSTRUCTOR ||
                mbd.hasConstructorArgumentValues() || !ObjectUtils.isEmpty(args)) {
            return autowireConstructor(beanName, mbd, ctors, args);
        }

        // Preferred constructors for default construction?
        ctors = mbd.getPreferredConstructors();
        if (ctors != null) {
            return autowireConstructor(beanName, mbd, ctors, null);
        }

        // 没有特殊处理，使用默认的无参构造函数创建Bean实例
        return instantiateBean(beanName, mbd);
    }
```

其实就是先确保这个Bean对应的类已经被加载，然后确保它是public的，然后如果有工厂方法，则直接调用工厂方法创建这个Bean，如果没有的话就调用它的构造方法来创建这个Bean。

这里需要注意的是，在Spring的完整Bean创建和初始化流程中，容器会在调用createBeanInstance之前检查Bean定义的作用域。如果是Singleton，容器会在其内部单例缓存中查找现有实例。如果实例已存在，它将被重用；如果不存在，才会调用createBeanInstance来创建新的实例。

```java
protected Object doCreateBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
        throws BeanCreationException {

    // 如果bean是单例的，则先去单例缓存中查找现有实例
    BeanWrapper instanceWrapper = null;
    if (mbd.isSingleton()) {
        instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
    }
    // 如果查找到实例，则可以重用
    // 如果查找不到实例，才会调用createBeanInstance创建实例
    if (instanceWrapper == null) {
        instanceWrapper = createBeanInstance(beanName, mbd, args);
    }
    //...
}
```

### 三级缓存解决循环依赖问题

#### 先简单介绍一下循环依赖问题

Spring中Bean的创建过程其实可以分成两步，第一步叫做实例化，第二步叫做初始化。实例化的过程只需要调用构造函数把对象创建出来并给他分配内存空间，而初始化则是给对象的属性进行赋值。而Spring之所以可以解决循环依赖就是因为对象的初始化是可以延后的，也就是说，当我创建一个Bean ServiceA的时候，会先把这个对象实例化出来，然后再初始化其中的serviceB属性。

```java
@Service
public class ServiceA{
    @Autowired
    private ServiceB serviceB;
}

@Service
public class ServiceB{
    @Autowired
    private ServiceA serviceA;
}
```

而当一个对象只进行了实例化，但是还没有进行初始化时，我们称之为半成品对象。所以，所谓半成品对象，其实只是 bean 对象的一个空壳子，还没有进行属性注入和初始化。

当两个Bean在初始化过程中互相依赖的时候，如初始化A发现他依赖了B，继续去初始化B，发现他又依赖了A，那这时候怎么办呢？大致流程如下：

#### 三级缓存流程图

![](./pic/Spring/三级缓存.png)

通过以上方式，就通过引入三级缓存，解决了循环依赖的问题，在上述流程执行完之后，ServiceA和ServiceB都被成功的完成了实例化和初始化。

以下是DefaultSingletonBeanRegistry#getSingleton方法，代码中，包括一级缓存、二级缓存、三级缓存的处理逻辑，该方法是获取bean的单例实例对象的核心方法：

```java
@Nullable
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    // 首先从一级缓存中获取bean实例对象，如果已经存在，则直接返回
    Object singletonObject = this.singletonObjects.get(beanName);
    if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
        // 如果一级缓存中不存在bean实例对象，而且当前bean正在创建中，则从二级缓存中获取bean实例对象
        singletonObject = this.earlySingletonObjects.get(beanName);
        if (singletonObject == null && allowEarlyReference) {
            // 如果二级缓存中也不存在bean实例对象，并且允许提前引用，则需要在锁定一级缓存之前，
            // 先锁定二级缓存，然后再进行一系列处理
            synchronized (this.singletonObjects) {
                // 进行一系列安全检查后，再次从一级缓存和二级缓存中获取bean实例对象
                singletonObject = this.singletonObjects.get(beanName);
                if (singletonObject == null) {
                    singletonObject = this.earlySingletonObjects.get(beanName);
                    if (singletonObject == null) {
                        // 如果二级缓存中也不存在bean实例对象，则从三级缓存中获取bean的ObjectFactory，并创建bean实例对象
                        ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                        if (singletonFactory != null) {
                            singletonObject = singletonFactory.getObject();
                            // 将创建好的bean实例对象存储到二级缓存中
                            this.earlySingletonObjects.put(beanName, singletonObject);
                            // 从三级缓存中移除bean的ObjectFactory
                            this.singletonFactories.remove(beanName);
                        }
                    }
                }
            }
        }
    }
    return singletonObject;
}
```

#### 二级缓存流程图

理论上只用二级缓存也可以解决循环依赖的问题，如图所示：

![](./pic/Spring/二级缓存.png)

那么，为什么还需要引入三级缓存呢？

如果完全依靠二级缓存解决循环依赖，意味着当我们依赖了一个代理类的时候，就需要在Bean实例化之后完成AOP代理。而在Spring的设计中，为了解耦Bean的初始化和代理，是通过AnnotationAwareAspectJAutoProxyCreator这个后置处理器来在Bean生命周期的最后一步来完成AOP代理的。

我给大家标出来了，如果使用三级缓存，在实例化之后，初始化之前，向三级缓存中保存的是ObjectFactory。而如果使用二级缓存，那么在这个步骤中保存的就是具体的Object。

这里如果我们只用二级缓存，对于普通对象的循环依赖问题是都可以正常解决的，但是如果是代理对象的话就麻烦多了，并且AOP又是Spring中很重要的一个特性，代理又不能忽略。

我们都知道，我们是可以在一个ServiceA中注入另外一个ServiceB的代理对象的，那么在解决循环依赖过程中，如果需要注入ServiceB的代理对象，就需要把ServiceB的代理对象创建出来，但是这时候还只是ServiceB的实例化阶段，代理对象的创建要等到初始化之后，在后置处理的postProcessAfterInitialization方法中对初始化后的Bean完成AOP代理的。

那怎么办好呢？Spring想到了一个好的办法，那就是使用三级缓存，并且在这个三级缓存中，并没有存入一个实例化的对象，而是存入了一个匿名类ObjectFactory（其实本质是一个函数式接口() -> getEarlyBeanReference(beanName, mbd, bean)），具体代码如下：

```java
public abstract class AbstractAutowireCapableBeanFactory extends AbstractBeanFactory
        implements AutowireCapableBeanFactory {
  protected Object doCreateBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
    throws BeanCreationException {
    ....

    // 如果允许循环引用，且beanName对应的单例bean正在创建中，则早期暴露该单例bean，以便解决潜在的循环引用问题
        boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
                isSingletonCurrentlyInCreation(beanName));
        if (earlySingletonExposure) {
            if (logger.isTraceEnabled()) {
                logger.trace("Eagerly caching bean '" + beanName +
                        "' to allow for resolving potential circular references");
            }
            // 向singletonFactories添加该beanName及其对应的提前引用对象，以便解决潜在的循环引用问题
            addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
        }

    ...
    }
}
```

#### Spring真的完全解决了循环依赖吗？

Spring默认不会开启循环依赖，所以想要测试循环依赖需要先配置上这个：

spring.main.allow-circular-references=true

看以下的例子：

```java
@Service
public class A {
  @Autowired private B b;

  @Async
  public void test() {
    System.out.println("A.test start");
  }
}


@Service
public class B {
  @Autowired private A a;

  public void test() {
    a.test();
  }
}
```

程序运行结果：

```log
2023-11-17 20:05:57.856  WARN 16032 --- [           main] ConfigServletWebServerApplicationContext : Exception encountered during context initialization - cancelling refresh attempt: org.springframework.beans.factory.BeanCurrentlyInCreationException: Error creating bean with name 'a': Bean with name 'a' has been injected into other beans [b] in its raw version as part of a circular reference, but has eventually been wrapped. This means that said other beans do not use the final version of the bean. This is often the result of over-eager type matching - consider using 'getBeanNamesForType' with the 'allowEagerInit' flag turned off, for example.
2023-11-17 20:05:57.857  INFO 16032 --- [           main] com.alibaba.druid.pool.DruidDataSource   : {dataSource-1} closed
2023-11-17 20:05:57.859  INFO 16032 --- [           main] o.apache.catalina.core.StandardService   : Stopping service [Tomcat]
2023-11-17 20:05:57.889 ERROR 16032 --- [           main] o.s.boot.SpringApplication               : Application run failed

org.springframework.beans.factory.BeanCurrentlyInCreationException: Error creating bean with name 'a': Bean with name 'a' has been injected into other beans [b] in its raw version as part of a circular reference, but has eventually been wrapped. This means that said other beans do not use the final version of the bean. This is often the result of over-eager type matching - consider using 'getBeanNamesForType' with the 'allowEagerInit' flag turned off, for example.
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:649) ~[spring-beans-5.3.29.jar:5.3.29]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:542) ~[spring-beans-5.3.29.jar:5.3.29]
	at org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:335) ~[spring-beans-5.3.29.jar:5.3.29]
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:234) ~[spring-beans-5.3.29.jar:5.3.29]
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:333) ~[spring-beans-5.3.29.jar:5.3.29]
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:208) ~[spring-beans-5.3.29.jar:5.3.29]
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingletons(DefaultListableBeanFactory.java:955) ~[spring-beans-5.3.29.jar:5.3.29]
	at org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:921) ~[spring-context-5.3.29.jar:5.3.29]
	at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:583) ~[spring-context-5.3.29.jar:5.3.29]
	at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.refresh(ServletWebServerApplicationContext.java:147) ~[spring-boot-2.7.14.jar:2.7.14]
	at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:731) [spring-boot-2.7.14.jar:2.7.14]
	at org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:408) [spring-boot-2.7.14.jar:2.7.14]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:307) [spring-boot-2.7.14.jar:2.7.14]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1303) [spring-boot-2.7.14.jar:2.7.14]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1292) [spring-boot-2.7.14.jar:2.7.14]
	at com.sun.fileconverter.FileConverterApplication.main(FileConverterApplication.java:16) [classes/:na]

Disconnected from the target VM, address: '127.0.0.1:10700', transport: 'socket'

```

这里可以看到，A有async方法，是一个需要被代理的bean，B是个简单的bean。启动程序后，实例化A，然后将A加入到三级缓存中，A初始化，需要依赖B；B不在一级、二级、三级缓存中，则实例化B，将B加入到三级缓存中，初始化B，需要依赖A，A在三级缓存中有，直接返回缓存对象到二级缓存中，所以三级缓存返回对象是A原始对象，此时初始化的逻辑出问题了：从二级缓存中能拿到A的原始对象，A自身初始化完成后会生成代理对象（@Async注解会在postProcessAfterInitialization阶段将A代理生成对象），Spring此时判断这个A又没有配置允许此bean的原始对象注入到其他bean中，则判断二级缓存和生成对象不一致，抛出了异常。

Spring中对应代码处理部分：

```java
    protected Object doCreateBean( ... ){
    	...
    	boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences && isSingletonCurrentlyInCreation(beanName));
    	if (earlySingletonExposure) {
    		addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
    	}
    	...
    
    	// populateBean这一句特别的关键，它需要给A的属性赋值，所以此处会去实例化B~~
    	// 而B我们从上可以看到它就是个普通的Bean（并不需要创建代理对象），实例化完成之后，继续给他的属性A赋值，而此时它会去拿到A的早期引用
    	// 也就在此处在给B的属性a赋值的时候，会执行到上面放进去的Bean A流程中的getEarlyBeanReference()方法  从而拿到A的早期引用~~
    	// 执行A的getEarlyBeanReference()方法的时候，会执行自动代理创建器，但是由于A没有标注事务，所以最终不会创建代理，so B合格属性引用会是A的**原始对象**
    	// 需要注意的是：@Async的代理对象不是在getEarlyBeanReference()中创建的，是在postProcessAfterInitialization创建的代理
    	// 从这我们也可以看出@Async的代理它默认并不支持你去循环引用，因为它并没有把代理对象的早期引用提供出来~~~（注意这点和自动代理创建器的区别~）
    
    	// 结论：此处给A的依赖属性字段B赋值为了B的实例(因为B不需要创建代理，所以就是原始对象)
    	// 而此处实例B里面依赖的A注入的仍旧为Bean A的普通实例对象（注意  是原始对象非代理对象）  注：此时exposedObject也依旧为原始对象
    	populateBean(beanName, mbd, instanceWrapper);
    	
    	// 标注有@Async的Bean的代理对象在此处会被生成~~~ 参照类：AsyncAnnotationBeanPostProcessor
    	// 所以此句执行完成后  exposedObject就会是个代理对象而非原始对象了
    	exposedObject = initializeBean(beanName, exposedObject, mbd);
    	
    	...
    	// 这里是报错的重点~~~
    	if (earlySingletonExposure) {
    		// 上面说了A被B循环依赖进去了，所以此时A是被放进了二级缓存的，所以此处earlySingletonReference 是A的原始对象的引用
    		// （这也就解释了为何我说：如果A没有被循环依赖，是不会报错不会有问题的   因为若没有循环依赖earlySingletonReference =null后面就直接return了）
    		Object earlySingletonReference = getSingleton(beanName, false);
    		if (earlySingletonReference != null) {
    			// 上面分析了exposedObject 是被@Aysnc代理过的对象， 而bean是原始对象 所以此处不相等  走else逻辑
    			if (exposedObject == bean) {
    				exposedObject = earlySingletonReference;
    			}
    			// allowRawInjectionDespiteWrapping 标注是否允许此Bean的原始类型被注入到其它Bean里面，即使自己最终会被包装（代理）
    			// 默认是false表示不允许，如果改为true表示允许，就不会报错啦。这是我们后面讲的决方案的其中一个方案~~~
    			// 另外dependentBeanMap记录着每个Bean它所依赖的Bean的Map~~~~
    			else if (!this.allowRawInjectionDespiteWrapping && hasDependentBean(beanName)) {
    				// 我们的Bean A依赖于B，so此处值为["b"]
    				String[] dependentBeans = getDependentBeans(beanName);
    				Set<String> actualDependentBeans = new LinkedHashSet<>(dependentBeans.length);
    
    				// 对所有的依赖进行一一检查~	比如此处B就会有问题
    				// “b”它经过removeSingletonIfCreatedForTypeCheckOnly最终返返回false  因为alreadyCreated里面已经有它了表示B已经完全创建完成了~~~
    				// 而b都完成了，所以属性a也赋值完成儿聊 但是B里面引用的a和主流程我这个A竟然不相等，那肯定就有问题(说明不是最终的)~~~
    				// so最终会被加入到actualDependentBeans里面去，表示A真正的依赖~~~
    				for (String dependentBean : dependentBeans) {
    					if (!removeSingletonIfCreatedForTypeCheckOnly(dependentBean)) {
    						actualDependentBeans.add(dependentBean);
    					}
    				}
    	
    				// 若存在这种真正的依赖，那就报错了~~~  则个异常就是上面看到的异常信息
    				if (!actualDependentBeans.isEmpty()) {
    					throw new BeanCurrentlyInCreationException(beanName,
    							"Bean with name '" + beanName + "' has been injected into other beans [" +
    							StringUtils.collectionToCommaDelimitedString(actualDependentBeans) +
    							"] in its raw version as part of a circular reference, but has eventually been " +
    							"wrapped. This means that said other beans do not use the final version of the " +
    							"bean. This is often the result of over-eager type matching - consider using " +
    							"'getBeanNamesOfType' with the 'allowEagerInit' flag turned off, for example.");
    				}
    			}
    		}
    	}
    	...
    }
```

所以这里的解决方法是：

1. 将A的允许此bean的原始对象注入到其他bean配置设置为true，即将B中注入的A加上@Lazy注解

2. 配置Spring的allowRawInjectionDespiteWrap为true，允许全局bean都可以将原始对象注入到其他bean中
   
   ```java
   public class MyBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
     @Override
     public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory)
         throws BeansException {
       ((AbstractAutowireCapableBeanFactory) beanFactory).setAllowRawInjectionDespiteWrapping(true);
     }
   }
   ```

### 设置属性值

populateBean方法是Spring Bean生命周期中的一个关键部分，负责将属性值应用到新创建的Bean实例。它处理了自动装配、属性注入、依赖检查等多个方面。代码如下：

```java
protected void populateBean(String beanName, RootBeanDefinition mbd, BeanWrapper bw) {
    // 获取Bean定义中的属性值
    PropertyValues pvs = mbd.getPropertyValues();

    // 如果BeanWrapper为空，则无法设置属性值
    if (bw == null) {
        if (!pvs.isEmpty()) {
            throw new BeanCreationException(
                    mbd.getResourceDescription(), beanName, "Cannot apply property values to null instance");
        }
        else {
            // 对于null实例，跳过设置属性阶段
            return;
        }
    }

    // 在设置属性之前，给InstantiationAwareBeanPostProcessors机会修改Bean状态
    // 这可以用于支持字段注入等样式
    boolean continueWithPropertyPopulation = true;

    // 如果Bean不是合成的，并且存在InstantiationAwareBeanPostProcessor，执行后续处理
    if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
        for (BeanPostProcessor bp : getBeanPostProcessors()) {
            if (bp instanceof InstantiationAwareBeanPostProcessor) {
                InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
                if (!ibp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
                    continueWithPropertyPopulation = false;
                    break;
                }
            }
        }
    }

    // 如果上述处理后决定不继续，则返回
    if (!continueWithPropertyPopulation) {
        return;
    }

    // 根据自动装配模式（按名称或类型），设置相关的属性值
    if (mbd.getResolvedAutowireMode() == RootBeanDefinition.AUTOWIRE_BY_NAME ||
            mbd.getResolvedAutowireMode() == RootBeanDefinition.AUTOWIRE_BY_TYPE) {
        MutablePropertyValues newPvs = new MutablePropertyValues(pvs);

        // 如果是按名称自动装配，添加相应的属性值
        if (mbd.getResolvedAutowireMode() == RootBeanDefinition.AUTOWIRE_BY_NAME) {
            autowireByName(beanName, mbd, bw, newPvs);
        }

        // 如果是按类型自动装配，添加相应的属性值
        if (mbd.getResolvedAutowireMode() == RootBeanDefinition.AUTOWIRE_BY_TYPE) {
            autowireByType(beanName, mbd, bw, newPvs);
        }

        pvs = newPvs;
    }

    // 检查是否需要进行依赖性检查
    boolean hasInstAwareBpps = hasInstantiationAwareBeanPostProcessors();
    boolean needsDepCheck = (mbd.getDependencyCheck() != RootBeanDefinition.DEPENDENCY_CHECK_NONE);

    // 如果需要，则进行依赖性检查
    if (hasInstAwareBpps || needsDepCheck) {
        PropertyDescriptor[] filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
        if (hasInstAwareBpps) {
            for (BeanPostProcessor bp : getBeanPostProcessors()) {
                if (bp instanceof InstantiationAwareBeanPostProcessor) {
                    InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
                    pvs = ibp.postProcessPropertyValues(pvs, filteredPds, bw.getWrappedInstance(), beanName);
                    if (pvs == null) {
                        return;
                    }
                }
            }
        }
        if (needsDepCheck) {
            checkDependencies(beanName, mbd, filteredPds, pvs);
        }
    }

    // 应用属性值
    applyPropertyValues(beanName, mbd, bw, pvs);
}
```

逻辑也比较清晰，就是把各种属性进行初始化。

### initializeBean方法

InitializingBean阶段主要包括afterPropertiesSet方法和自定义的初始化方法，具体实现是invokeInitMethods方法。

```java
protected Object initializeBean(String beanName, Object bean, @Nullable RootBeanDefinition mbd) {
        //...
        //检查Aware    else {
        invokeAwareMethods(beanName, bean);
        //...
        //调用BeanPostProcessor的前置处理方法
        Object wrappedBean = bean;
        if (mbd == null || !mbd.isSynthetic()) {
            wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
        }
        //调用InitializingBean的afterPropertiesSet方法或自定义的初始化方法及自定义init-method方法
        try {
            invokeInitMethods(beanName, wrappedBean, mbd);
        }
        catch (Throwable ex) {
            throw new BeanCreationException(
                    (mbd != null ? mbd.getResourceDescription() : null),
                    beanName, "Invocation of init method failed", ex);
        }
        //调用BeanPostProcessor的后置处理方法
        if (mbd == null || !mbd.isSynthetic()) {
            wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
        }

        return wrappedBean;
    }
```

#### 检查Aware

```java
private void invokeAwareMethods(String beanName, Object bean) {
        if (bean instanceof Aware) {
            if (bean instanceof BeanNameAware) {
                ((BeanNameAware) bean).setBeanName(beanName);
            }
            if (bean instanceof BeanClassLoaderAware) {
                ClassLoader bcl = getBeanClassLoader();
                if (bcl != null) {
                    ((BeanClassLoaderAware) bean).setBeanClassLoader(bcl);
                }
            }
            if (bean instanceof BeanFactoryAware) {
                ((BeanFactoryAware) bean).setBeanFactory(AbstractAutowireCapableBeanFactory.this);
            }
        }
    }
```

就是检查这个Bean是不是实现了BeanNameAware、BeanClassLoaderAware等这些Aware接口，Spring容器会调用它们的方法进行处理。

这些Aware接口提供了一种机制，使得Bean可以与Spring框架的内部组件交互，从而更灵活地利用Spring框架提供的功能：

- BeanNameAware: 通过这个接口，Bean可以获取到自己在Spring容器中的名字。这对于需要根据Bean的名称进行某些操作的场景很有用。

- BeanClassLoaderAware: 这个接口使Bean能够访问加载它的类加载器。这在需要进行类加载操作时特别有用，例如动态加载类。

- BeanFactoryAware：通过这个接口可以获取对 BeanFactory 的引用，获得对BeanFactory 的访问权限

#### 调用BeanPostProcessor的前置处理方法

BeanPostProcessor是Spring IOC容器给我们提供的一个扩展接口，他的主要作用主要是帮我们在Bean的初始化前后添加一些自己的逻辑处理，Spring内置了很多BeanPostProcessor，我们也可以定义一个或者多个 BeanPostProcessor 接口的实现，然后注册到容器中。

调用BeanPostProcessor的前置处理方法是在applyBeanPostProcessorsBeforeInitialization这个方法中实现的，代码如下：

```java
@Override
    public Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName)
            throws BeansException {

        Object result = existingBean;
        for (BeanPostProcessor processor : getBeanPostProcessors()) {
            Object current = processor.postProcessBeforeInitialization(result, beanName);
            if (current == null) {
                return result;
            }
            result = current;
        }
        return result;
    }
```

其实就是遍历所有的BeanPostProcessor的实现，执行他的postProcessBeforeInitialization方法。

##### @PostConstruct注解标注的初始化方法就是在这里反射调用完成的

#### afterPropertiesSet方法

InitializingBean接口的afterPropertiesSet方法被各种组件使用，无论是Spring内置的组件还是外部的组件集成到Spring中，有很多实现类实现了InitializingBean接口，用来完成bean初始化的这一阶段操作。

#### 自定义初始化方法

@Bean(initMethod = "initMethod")是Spring提供给我们的指定一个bean的初始化方法的入口。具体实现逻辑在invokeCustomInitMethod方法里：

```java
protected void invokeCustomInitMethod(String beanName, Object bean, RootBeanDefinition mbd)
            throws Throwable {
        // 获取配置的 initMethod
        String initMethodName = mbd.getInitMethodName();
        // 通过反射获取方法对象
        Method initMethod = (mbd.isNonPublicAccessAllowed() ?
                BeanUtils.findMethod(bean.getClass(), initMethodName) :
                ClassUtils.getMethodIfAvailable(bean.getClass(), initMethodName));
        // 如果方法不存在 , 抛出异常或返回
        if (initMethod == null) {
            if (mbd.isEnforceInitMethod()) {
                throw new BeanDefinitionValidationException("Could not find an init method named '" +
                        initMethodName + "' on bean with name '" + beanName + "'");
            }
            else {
                if (logger.isTraceEnabled()) {
                    logger.trace("No default init method named '" + initMethodName +
                            "' found on bean with name '" + beanName + "'");
                }
                return;
            }
        }

        // 如果可能，为给定的方法句柄确定相应的接口方法
        Method methodToInvoke = ClassUtils.getInterfaceMethodIfPossible(initMethod, bean.getClass());
        // 通过方法反射的方式执行method，需要先获取权限管理
        if (System.getSecurityManager() != null) {
            AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
                ReflectionUtils.makeAccessible(methodToInvoke);
                return null;
            });
            try {
                 // 由相应的管理者去执行初始化方法
                AccessController.doPrivileged((PrivilegedExceptionAction<Object>)
                        () -> methodToInvoke.invoke(bean), getAccessControlContext());
            }
            catch (PrivilegedActionException pae) {
                InvocationTargetException ex = (InvocationTargetException) pae.getException();
                throw ex.getTargetException();
            }
        }
        else {
            try {
                ReflectionUtils.makeAccessible(methodToInvoke);
                 // 不需要许可的情况下可以直接调用反射执行初始化方法
                methodToInvoke.invoke(bean);
            }
            catch (InvocationTargetException ex) {
                throw ex.getTargetException();
            }
        }
    }
```

### 调用BeanPostProcessor的后置处理方法

调用BeanPostProcessor的后置处理方法是在applyBeanPostProcessorsAfterInitialization这个方法中实现的，代码如下：

```java
@Override
public Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName)
        throws BeansException {

    Object result = existingBean;
    for (BeanPostProcessor processor : getBeanPostProcessors()) {
        result = processor.postProcessAfterInitialization(result, beanName);
        if (result == null) {
            return result;
        }
    }
    return result;
}
```

其实就是遍历所有的BeanPostProcessor的实现，执行他的postProcessAfterInitialization方法。这里面需要我们关注的就是AnnotationAwareAspectJAutoProxyCreator（继承自AspectJAwareAdvisorAutoProxyCreator，继承自AbstractAdvisorAutoProxyCreator，继承自AbstractAutoProxyCreator），他们也是BeanPostProcessor的实现，他之所以重要，是因为在他的postProcessAfterInitialization 后置处理方法。

```java
@Override
    public Object postProcessAfterInitialization(@Nullable Object bean, String beanName) {
        if (bean != null) {
            Object cacheKey = getCacheKey(bean.getClass(), beanName);
            if (this.earlyProxyReferences.remove(cacheKey) != bean) {
                return wrapIfNecessary(bean, beanName, cacheKey);
            }
        }
        return bean;
    }
```

在这里完成AOP的代理的创建。

### 注册Destruction方法回调

调用registerDisposableBeanIfNecessary方法完成对bean实现的disposable接口的destruction方法注册。用于bean销毁时回调。

```java
protected void registerDisposableBeanIfNecessary(String beanName, Object bean, RootBeanDefinition mbd) {
        AccessControlContext acc = (System.getSecurityManager() != null ? getAccessControlContext() : null);
        if (!mbd.isPrototype() && requiresDestruction(bean, mbd)) {
            if (mbd.isSingleton()) {
                //这里注册单例bean实现的disposable接口的destruction方法回调
                registerDisposableBean(beanName, new DisposableBeanAdapter(
                        bean, beanName, mbd, getBeanPostProcessorCache().destructionAware, acc));
            }
            else {
                // A bean with a custom scope...
                Scope scope = this.scopes.get(mbd.getScope());
                if (scope == null) {
                    throw new IllegalStateException("No Scope registered for scope name '" + mbd.getScope() + "'");
                }
                //这里注册非单例bean实现的disposable接口的destruction方法回调
                scope.registerDestructionCallback(beanName, new DisposableBeanAdapter(
                        bean, beanName, mbd, getBeanPostProcessorCache().destructionAware, acc));
            }
        }
    }
```

## 为什么Spring不建议使用基于字段的依赖注入？

在我们通过IDEA编写Spring的代码的时候，假如我们编写了如下代码：

```java
@Autowired
private Bean bean;
```

IDEA会给我们一个warning警告：Field injection is not recommended

翻阅[官方文档](https://docs.spring.io/spring-framework/docs/5.1.3.RELEASE/spring-framework-reference/core.html#beans-dependencies)我们会发现： 
Since you can mix constructor-based and setter-based DI, it is a good rule of thumb to use constructors for mandatory dependencies and setter methods or configuration methods for optional dependencies.

大意就是强制依赖使用构造器注入，可选依赖使用setter注入。那么这是为什么呢？使用字段注入又会导致什么问题呢？

### 单一职责问题

我们都知道，根据SOLID设计原则来讲，一个类的设计应该符合单一职责原则，就是一个类只能做一件功能，当我们使用基于字段注入的时候，随着业务的暴增，字段越来越多，我们是很难发现我们已经默默中违背了单一职责原则的。  

但是如果我们使用基于构造器注入的方式，因为构造器注入的写法比较臃肿，所以它就在间接提醒我们，违背了单一职责原则，该做重构了

### 隐藏依赖

对于一个正常的使用依赖注入的Bean来说，它应该“显式”的通知容器，自己需要哪些Bean，可以通过构造器通知，public的setter方法通知，这些设计都是没问题的。

但是对于private的field来说，从设计的角度来讲，外部的容器是不应该感知到bean内部的private属性的，所以理论上，private的field是没办法通知到容器的（不考虑反射，单从设计角度理解），所以从这个角度来看，我们最好不要通过字段注入。

### 不利于测试

很明显，使用了Autowired注解，说明这个类依赖了Spring容器，这让我们在进行UT的时候必须要启动一个Spring容器才可以测试这个类，显然太麻烦，这种测试方式非常重，对于大型项目来说，往往启动一个容器就要好几分钟，这样非常耽误时间。

不过，如果使用构造器的依赖注入就不会有这种问题，或者，我们可以使用Resource注解也可以解决上述问题

### Spring支持哪些注入方式

#### 字段注入

```java
@Autowired
private Bean bean;
```

#### 构造器注入

```java
@Component
class ServiceA {
    private final Bean bean;

    @Autowired
    public ServiceA(Bean bean) {
        this.bean = bean;
    }
}
```

setter注入

```java
@Component
class ServiceA {
    private Bean bean;

    @Autowired
    public void setBean(Bean bean) {
        this.bean = bean;
    }
}
```

## Spring 6.0和SpringBoot 3.0有什么新特性？

Spring在2022年相继推出了Spring Framework 6.0和SpringBoot 3.0，Spring把这次升级称之为新一代框架的开始，下一个10年的新开端

主要更新内容是以下几个：  

- A Java 17 baseline

- Support for Jakarta EE 10 with an EE 9 baseline

- Support for generating native images with GraalVM, superseding the experimental Spring Native project

- Ahead-Of-Time transformations and the corresponding AOT processing support for Spring application contexts

首先，前两个比较容易理解，主要说的是依赖的服务的版本升级的信息，那就是Spring Framework 6.0和SpringBoot 3.0都要求JDK的版本最低也得是JDK 17；并且底层依赖的J2EE也迁移到了Jakarta EE 9。

### AOT编译

Ahead-Of-Time，即预先编译，这是相对于我们熟知的Just-In-Time（JIT，即时编译）来说的。

相比于JIT编译，AOT指的是在程序运行前编译，这样就可以避免在运行时的编译性能消耗和内存消耗，可以在程序运行初期就达到最高性能、也可以显著的加快程序的启动。

AOT的引入，意味着Spring生态正式引入了提前编译技术，相比于JIT编译，AOT有助于优化Spring框架启动慢、占用内存多、以及垃圾无法被回收等问题。

### Spring Native

在Spring的新版本中引入了Spring Native。

有了Spring Native ，Spring可以不再依赖Java虚拟机，而是基于 GraalVM 将 Spring 应用程序编译成原生镜像（native image），提供了一种新的方式来部署 Spring 应用。这种部署Spring的方式是云原生友好的。

Spring Native的优点是编译出来的原生 Spring 应用可以作为一个独立的可执行文件进行部署，而不需要安装JVM，而且启动时间非常短、并且有更少的资源消耗。他的缺点就是构建时长要比JVM更长一些，且不支持一些动态代理功能。
