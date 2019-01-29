---
title: 开放平台的设计
date: 2019-01-28 18:15:01
tags: 开放平台 Spring Cloud Gateway
---
                         


  ### 搭建开放平台的目的       

​         旅居平台与基础平台提出了要做SAAS平台，接入第三方公寓的诉求。优品也提出要接入不同的商家，提供平台让第三方在此平台上销售产品。遂打算做一套开放平台来实现不同业务线的诉求。

​        开放平台通过开放自身产品服务的各种API功能，让其他开发者在开发应用时根据需求直接调用。为第三方的开发者提供了基础服务。

​        

### 开放平台的诉求

开放平台的设计需要满足以下几个需求：

1.  提供一套对内的管理系统，方便业务方添加服务、管理第三方注册的用户、对第三方注册密钥的管控。
2.  提供一套对第三方使用的系统，类似于CMS，可以注册用户，管理服务，服务里还需要提供密钥申请，文档  管理等功能。
3.  开放平台本身承担的功能类似于API  Gateway,  需要转发请求到对应的后台服务，还要承担接口鉴权验签的责任，100%保障业务接口的安全。监控的维度上，还需要获取每个API调用频次，方便日后根据API来计费使用。自身也需要具备限流，熔断等基础功能，提供高可用性。



  在调研的过程中，参考了别的公司对于开放平台的设计。下图是易宝开放平台的设计：

![1548644022989](https://techimg.ziroom.com/1548644022989.png)

  

![1548644033842](https://techimg.ziroom.com/1548644033842.png)





### 开放平台的技术设计



​    博主在五年前接触到的第一款开放平台叫[rop](https://github.com/itstamen/rop) , rop的设计非常优秀，基于spring mvc的架构，RopServlet类似于*DispatcherServlet*, 负责截获http请求然后转由rop处理。

![img](https://techimg.ziroom.com/TIM%E5%9B%BE%E7%89%8720190128183138.png)



这套架构放在目前依然有很高的学习价值。

但放眼现在，满足条件的框架已经不胜枚举，有基于OpenResty的成熟产品kong,还有基于java的zuul,spring cloud gateway。从可维护性和性能上来说，spring cloud gateway都更胜一筹，spring cloud gateway基于spring5的webflux模块开发，使用的服务器为reactor  netty,  异步化的架构对于这种场景来说在合适不过。

![Spring Cloud Gateway Diagram](https://raw.githubusercontent.com/spring-cloud/spring-cloud-gateway/master/docs/src/main/asciidoc/images/spring_cloud_gateway_diagram.png)



介绍一下spring cloud gateway的相关概念：



- Route（路由）：这是网关的基本构建块。它由一个 ID，一个目标 URI，一组断言和一组过滤器定义。如果断言为真，则路由匹配。
- Predicate（断言）：这是一个 Java 8 的 Predicate。输入类型是一个 ServerWebExchange。我们可以使用它来匹配来自 HTTP 请求的任何内容，例如 headers 或参数。
- Filter（过滤器）：这是`org.springframework.cloud.gateway.filter.GatewayFilter`的实例，我们可以使用它修改请求和响应。

客户端向 Spring Cloud Gateway 发出请求。如果 Gateway Handler Mapping 中找到与请求相匹配的路由，将其发送到 Gateway Web Handler。Handler 再通过指定的过滤器链来将请求发送到我们实际的服务执行业务逻辑，然后返回。 过滤器之间用虚线分开是因为过滤器可能会在发送代理请求之前（“pre”）或之后（“post”）执行业务逻辑。

Spring Cloud Gateway 的特征：

- 基于 Spring Framework 5，Project Reactor 和 Spring Boot 2.0
- 动态路由
- Predicates 和 Filters 作用于特定路由
- 集成 Hystrix 断路器
- 集成 Spring Cloud DiscoveryClient
- 易于编写的 Predicates 和 Filters
- 限流
- 路径重写



具体的文档可以参考

https://github.com/spring-cloud/spring-cloud-gateway/blob/master/docs/src/main/asciidoc/spring-cloud-gateway.adoc

在具体的使用过程中，博主扩展了router，将路由相关的信息存放到mysql中，5分钟刷新一次cache，便于对路由的管理。

使用过程中遇到了版本的问题，导致form表单的形式转发请求失败，在issue中发现了相关的解决方案。所以这里也要提醒大家有问题的时候多去github找找相关的issue，说不定你踩过的坑就有别人踩过。







