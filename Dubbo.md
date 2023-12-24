# Dubbo

## 什么是Dubbo的优雅停机，怎么实现的？

### 优雅上下线

关于"优雅上下线"这个词，我没找到官方的解释，我尝试解释一下这是什么。  

首先，上线、下线大家一定都很清楚，比如我们一次应用发布过程中，就需要先将应用服务停掉，然后再把服务启动起来。这个过程就包含了一次下线和一次上线。  

那么，"优雅"怎么理解呢？  

先说什么情况我们认为不优雅：  

1、服务停止时，没有关闭对应的监控，导致应用停止后发生大量报警。  

2、应用停止时，没有通知外部调用方，很多请求还会过来，导致很多调用失败。  

3、应用停止时，有线程正在执行中，执行了一半，JVM进程就被干掉了。  

4、应用启动时，服务还没准备好，就开始对外提供服务，导致很多失败调用。  

5、应用启动时，没有检查应用的健康状态，就开始对外提供服务，导致很多失败调用。  

以上，都是我们认为的不优雅的情况，那么，反过来，优雅上下线就是一种避免上述情况发生的手段。  

一个应用的优雅上下线涉及到的内容其实有很多，从底层的操作系统、容器层面，到编程语言、框架层面，再到应用架构层面，涉及到的知识很广泛。  

其实，优雅上下线中，最重要的还是优雅下线。因为如果下线过程不优雅的话，就会发生很多调用失败了、服务找不到等问题。所以很多时候，大家也会提优雅停机这样的概念。  

本文后面介绍的优雅上下线也重点关注优雅停机的过程。

### 操作系统&容器的优雅上下线

我们知道，kill -9之所以不建议使用，是因为kill -9特别强硬，系统会发出SIGKILL信号，他要求接收到该信号的程序应该立即结束运行，不能被阻塞或者忽略。  

这个过程显然是不优雅的，因为应用立刻停止的话，就没办法做收尾动作。而更优雅的方式是kill -15（kill命令默认使用-15）。  

当使用kill -15时，系统会发送一个SIGTERM的信号给对应的程序。当程序接收到该信号后，具体要如何处理是自己可以决定的。  

kill -15会通知到应用程序，这就是操作系统对于优雅上下线的最基本的支持。  

以前，在操作系统之上就是应用程序了，但是，自从容器化技术推出之后，在操作系统和应用程序之间，多了一个容器层，而Docker、k8s等容器其实也是支持优雅上下线的。  

如Docker中同样提供了两个命令， docker stop 和 docker kill  

docker stop就像kill -15一样，他会向容器内的进程发送SIGTERM信号，在10S之后（可通过参数指定）再发送SIGKILL信号。  

而docker kill就像kill -9，直接发送SIGKILL信号。

### JVM的优雅上下线

在操作系统、容器等对优雅上下线有了基本的支持之后，在接收到docker stop、kill -15等命令后，会通知应用进程进行进程关闭。  

而Java应用在运行时就是一个独立运行的进程，这个进程是如何关闭的呢？  

Java程序的终止运行是基于JVM的关闭实现的，JVM关闭方式分为正常关闭、强制关闭和异常关闭3种。  

这其中，正常关闭就是支持优雅上下线的。正常关闭过程中，JVM可以做一些清理动作，比如删除临时文件。  

当然，开发者也是可以自定义做一些额外的事情的，比如通知应用框架优雅上下线操作。  

而这种机制是通过JDK中提供的shutdown hook实现的。JDK提供了Java.Runtime.addShutdownHook(Thread hook)方法，可以注册一个JVM关闭的钩子。  

例子如下：  

```java
package com.hollis;

public class ShutdownHookTest {

    public static void main(String[] args) {
        boolean flag = true;
        Runtime.getRuntime().addShutdownHook(new Thread(() -> {
            System.out.println("hook execute...");
        }));

        while (flag) {
            // app is runing
        }

        System.out.println("main thread execute end...");
    }
}
```

控制台输出内容：  

```log
hook execute...
Process finished with exit code 143 (interrupted by signal 15: SIGTERM)
```

可以看到，当我们使用kill（默认kill -15）关闭进程的时候，程序会先执行我注册的shutdownHook，然后再退出，并且会给出一个提示：interrupted by signal 15: SIGTERM

### Spring的优雅上下线

有了JVM提供的shutdown hook之后，很多框架都可以通过这个机制来做优雅下线的支持。  

比如Spring，他就会向JVM注册一个shutdown hook，在接收到关闭通知的时候，进行bean的销毁，容器的销毁处理等操作。  

同时，作为一个成熟的框架，Spring也提供了事件机制，可以借助这个机制实现更多的优雅上下线功能。  

ApplicationListener是Spring事件机制的一部分，与抽象类ApplicationEvent类配合来完成ApplicationContext的事件机制。  

开发者可以实现ApplicationListener接口，监听到 Spring 容器的关闭事件（ContextClosedEvent），来做一些特殊的处理：

```java
@Component
public class MyListener implements ApplicationListener<ContextClosedEvent> {
    @Override
    public void onApplicationEvent(ContextClosedEvent event) {
        // 做容器关闭之前的清理工作
    }
}
```

### Dubbo的优雅上下线

因为Spring中提供了ApplicationListener接口，帮助我们来监听容器关闭事件，那么，很多web容器、框架等就可以借助这个机制来做自己的优雅上下线操作。  

如tomcat、dubbo等都是这么做的。  

应用在停机时，接收到关闭通知时，会先把自己标记为不接受（发起）新请求，然后再等待10s（默认是10秒）的时候，等执行中的线程执行完。  

那么，之所以他能做这些事，是因为从操作系统、到JVM、到Spring等都对优雅停机做了很好的支持。  

关于Dubbo各个版本中具体是如何借助JVM的shutdown hook机制、或者说Spring的事件机制做的优雅停机，我的一位同事的一篇文章介绍的很清晰，大家可以看下：  

[一文聊透 Dubbo 优雅停机 - 徐靖峰|个人博客](https://www.cnkirito.moe/dubbo-gracefully-shutdown/)  

在从Dubbo 2.5 到 Dubbo 2.7介绍了历史版本中，Dubbo为了解决优雅上下线问题所遇到的问题和方案。  

目前，Dubbo中实现方式如下，同样是用到了Spring的事件机制：

```java
public class SpringExtensionFactory implements ExtensionFactory {
    public static void addApplicationContext(ApplicationContext context) {
        CONTEXTS.add(context);
        if (context instanceof ConfigurableApplicationContext) {
            ((ConfigurableApplicationContext) context).registerShutdownHook();
            DubboShutdownHook.getDubboShutdownHook().unregister();
        }
        BeanFactoryUtils.addApplicationListener(context, SHUTDOWN_HOOK_LISTENER);
    }
}
```

## Dubbo支持哪些调用协议？

dubbo支持多种协议，主要由以下几个：

1. dubbo 协议 (默认) 
   默认就是走dubbo协议的，基于hessian作为序列化协议，单一长连接，TCP协议传输，NIO异步通信，适合大并发小数据量的服务调用，以及消费者远大于提供者，传输数据量很小（每次请求在100kb以内），但是并发量很高。

2. rmi 协议
   采用JDK标准的rmi协议实现，传输参数和返回参数对象需要实现Serializable接口，使用java标准序列化机制，使用阻塞式短连接，传输数据包大小混合，消费者和提供者个数差不多，可传文件，传输协议TCP。

3. hessian 协议
   集成Hessian服务，基于HTTP通讯，采用Servlet暴露服务，Dubbo内嵌Jetty作为服务器时默认实现，提供与Hession服务互操作。
   hessian序列化协议，多个短连接，同步HTTP传输，传入参数较大，提供者大于消费者，提供者压力较大，适用于文件的传输，一般较少用；

4. http 协议
   基于Http表单提交的远程调用协议，使用Spring的HttpInvoke实现。

5. webservice 协议
   
   基于WebService的远程调用协议，集成CXF实现，提供和原生WebService的互操作。

6. thrift 协议
   当前 dubbo 支持的 thrift 协议是对 thrift 原生协议 的扩展，在原生协议的基础上添加了一些额外的头信息，比如 service name，magic number 等。

7. memcached 协议
   基于 memcached实现的 RPC 协议。

8. redis 协议
   基于 Redis实现的 RPC 协议。

9. restful
   基于标准的Java REST API——JAX-RS 2.0（Java API for RESTful Web Services的简写）实现的REST调用支持

## Dubbo服务发现与路由的概念有什么不同？

服务发现是指在Dubbo注册中心中查找提供某个服务的服务提供者，以便服务消费者可以调用它们。  

Dubbo的注册中心可以是ZooKeeper、Redis等，服务提供者在启动时会将自己的地址信息注册到注册中心中，服务消费者在调用服务时会从注册中心中获取服务提供者的地址信息。  

服务路由是指根据一定的规则将服务请求路由到指定的服务提供者上。Dubbo提供了多种路由策略，如随机路由、轮询路由、一致性哈希路由等。路由规则可以在Dubbo的配置文件中进行配置，也可以在运行时通过API进行动态修改。  

因此，服务发现是获取服务提供者的地址信息，路由则是将服务请求路由到指定的服务提供者上。两者都是Dubbo中非常重要的概念，但是它们的作用是不同的。

## Dubbo的缓存机制了解吗？

Dubbo提供了缓存机制，其主要作用是缓存服务调用的响应结果，减少重复调用服务的次数，提高调用性能。  

Dubbo支持了服务端结果缓存和客户端结果缓存。  

服务端缓存是指将服务端方法的返回结果缓存到内存中，以便下次请求时可以直接从缓存中获取结果，而不必再调用服务方法。服务端缓存可以提高响应速度和系统吞吐量。Dubbo提供了三种服务端缓存的实现方式：

- LRU Cache: 使用基于LRU(最近最少使用)算法的缓存，当缓存空间满时，会将最近最少使用的缓存清除掉。

- Thread Local Cache: 使用线程本地缓存，即每个线程都拥有一个缓存实例，缓存结果只对当前线程可见。

- Concurrent Map Cache: 使用基于ConcurrentMap的缓存，支持并发读写，相对LRU Cache和Thread Local Cache来说，缓存效率更高。  

客户端缓存是指客户端将调用远程服务方法的返回结果缓存到内存中，以便下次请求时可以直接从缓存中获取结果，而不必再调用远程服务方法。消费端缓存可以提高系统的响应速度和降低系统的负载。Dubbo提供了两种消费端缓存的实现方式：

- LRU Cache: 使用基于LRU算法的缓存，当缓存空间满时，会将最近最少使用的缓存清除掉。 

- Thread Local Cache: 使用线程本地缓存，即每个线程都拥有一个缓存实例，缓存结果只对当前线程可见。  

需要注意的是，缓存虽好用，使用需谨慎，过度依赖缓存可能会出现数据不一致的问题。

### 服务端缓存

接口维度的服务端缓存配置方式支持XML和注解两种：
XML配置：

```java
<bean id="demoService" class="org.apache.dubbo.demo.provider.DemoServiceImpl"/>
<dubbo:service interface="com.foo.DemoService" ref="demoService" cache="lru" /> 
```

注解配置方式：

```java
@DubboService(cache = "lru")
public class DemoServiceImpl implements DemoService {

    private static final Logger logger = LoggerFactory.getLogger(DemoServiceImpl.class);
    @Override
    public String sayHello(String name) {
        logger.info("Hello " + name + ", request from consumer: " + RpcContext.getContext().getRemoteAddress());
        return "Hello " + name;

    }

}
```

还支持方法维度的缓存配置，同样支持XML和注解两种：

```java
<bean id="demoService" class="org.apache.dubbo.demo.provider.DemoServiceImpl"/>
<dubbo:service interface="com.foo.DemoService" ref="demoService" cache="lru" />
    <dubbo:method name="sayHello" cache="lru" />
</dubbo:service>
```

```java
@DubboService(methods = {@Method(name="sayHello",cache = "lru")})
public class DemoServiceImpl implements DemoService {

    private static final Logger logger = LoggerFactory.getLogger(DemoServiceImpl.class);
    @Override
    public String sayHello(String name) {
        logger.info("Hello " + name + ", request from consumer: " + RpcContext.getContext().getRemoteAddress());
        return "Hello " + name;
    }
}
```

### 客户端缓存

接口维度：

```java
<dubbo:reference interface="com.foo.DemoService" cache="lru" />
```

或者：

```java
@DubboReference(cache = "lru")
private DemoService demoService;
```

方法维度：

```java
<dubbo:reference interface="com.foo.DemoService">
    <dubbo:method name="sayHello" cache="lru" />
</dubbo:reference>
```

或者：

```java
@DubboReference(methods = {@Method(name="sayHello",cache = "lru")})
private DemoService demoService;
```

## Dubbo如何实现像本地方法一样调用远程方法的？

Dubbo 实现像本地方法一样调用远程方法的核心技术是动态代理。Dubbo 使用 JDK 动态代理或者字节码增强技术，生成一个代理类，该代理类实现了本地接口，具有本地接口的所有方法。在调用本地接口方法时，会通过代理类的 invoke 方法将请求转发到远程服务提供者上。  

生成代理类，Dubbo 在启动时会扫描配置文件（注解）中指定的服务接口，并根据服务接口生成一个代理类。这个代理类实现了服务接口，并且在调用服务接口的方法时，会将参数封装成请求消息，然后通过网络传输给服务提供方。  

网络通信，Dubbo 支持多种通信协议，包括 Dubbo 协议、HTTP 协议、Hessian 协议等。在配置文件中指定了要使用的通信协议后，Dubbo 会根据协议的不同，选择不同的序列化方式，将请求消息序列化成二进制流并发送给服务提供方。  

负载均衡，Dubbo 支持多种负载均衡算法，包括轮询、随机、加权随机、最小活跃数等。在客户端发起调用时，Dubbo 会根据负载均衡算法选择一台服务提供方进行调用。  

远程服务执行，当客户端发起远程调用后，服务提供方接收到请求后，会根据请求中的服务接口名和方法名，找到对应的实现类和方法，并将请求消息反序列化成参数列表，最终调用服务实现类的方法，并将执行结果序列化成响应消息返回给客户端。

## Dubbo实现服务调用的过程是什么样的？

Dubbo的整体架构中，有多个角色，分别是服务提供者，服务调用者以及服务注册中心。一次完整的服务调用过程其实要分为服务注册、服务发现和服务调用三个过程。

![](./pic/dubbo/dubbo-1.png)

1、服务注册：服务提供者在启动时，会向注册中心注册自己提供的服务，并将服务相关的信息（如服务名称、版本号、IP地址、端口号、协议、权重等）一并注册。Dubbo支持多种注册中心，包括ZooKeeper、Redis、Multicast、Simple等。一旦服务注册成功，服务提供者就可以等待服务调用请求的到来。  

2、服务发现：服务调用者在启动时，需要向注册中心订阅自己所需的服务，注册中心会将服务提供者列表返回给服务调用者。当过程中，如果服务提供者列表发生变化，那么Dubbo会通知客户端进行变更。  

3、服务调用：服务调用者需要根据负载均衡策略选择一个服务提供者，之后，就可以发送服务调用请求了。Dubbo支持多种通信协议和序列化方式。Dubbo客户端将调用请求序列化成二进制数据，并使用网络协议发送给服务提供者，服务提供者将调用请求反序列化后，调用目标方法并将结果序列化成二进制数据返回给服务调用者。在整个调用过程中 ，Dubbo会对服务调用进行监控，包括调用次数、调用时间、响应时间、异常次数、异常信息等，以便于服务提供者和服务调用者进行故障排查和性能调优。

### 连接方式

Dubbo提供了多种通信协议和通信方式，包括dubbo、http、hessian等，对于不同的通信方式，Dubbo也提供了不同的通信模型。对于服务提供者和服务消费者之间的通信，可以使用长连接模式或短连接模式，可以使用NIO或者Netty等高性能的通信框架，还可以设置心跳等机制来保持连接的稳定性。  

对于服务提供者和注册中心之间的通信，Dubbo同样提供了多种注册中心实现，包括ZooKeeper、Redis、Multicast和Simple等，这些注册中心实现了不同的通信协议和通信方式。  

在使用ZooKeeper作为注册中心时，Dubbo会通过ZooKeeper的长连接来注册服务和订阅服务提供者列表，并通过ZooKeeper提供的watch机制来监听服务提供者列表的变化。而在使用Redis作为注册中心时，则会使用Redis提供的订阅-发布机制来进行服务注册和发现。

## 为什么RPC要比HTTP更快一些？

其实，RPC的设计目的就是用于高效的内部服务通信，他通常优化了数据传输和序列化过程，目的是减少网络延迟和提高性能。而HTTP的设计是一种更通用的协议，用于Web文档传输，它在灵活性和可访问性上进行了优化，而不是仅仅专注于性能。  

展开来说主要在以下几个方面，RPC做出了很多事情。  

轻量级序列化协议，RPC通常使用更高效的数据序列化格式（如Protocol Buffers、Thrift等），这些格式专门为性能和效率设计，它们比HTTP标准使用的文本格式（如JSON、XML）更紧凑、解析更快。  

网络协议更优，RPC的网络通信协议通常被设计的更轻量（如Netty），他一般不需要像HTTP那样有很复杂的Header信息，从而不需要传输太多的数据。  

长连接，虽然RPC和HTTP都是基于TCP的，但是RPC可以使用长连接和更有效的连接管理策略，如gRPC还是基于HTTP/2实现，这可以减少建立连接的开销，并允许多个请求在同一连接上有效地复用。虽然HTTP/1.1也有keep-alive机制，HTTP/2也有很多优化。但是RPC只在企业内部用，所以兼容性更好，而HTTP新版的普及程度并不太高。  

定制优化，RPC框架通常允许更深层次的定制和优化，比如调整底层传输细节、序列化方式和错误处理机制。而HTTP作为一个标准化的Web协议，其灵活性和定制能力可能较低，特别是在面向性能的场景中。  

内部网络，RPC通常应用于企业内部，内部网络交互链路更短，而HTTP在公网上进行通信，一次交互需要经过多个中间节点的转换。


