以下就是笔者学习java代码审计的路线，其实大部分教程会告诉你某种漏洞的原理和审计方法，对于新手来说很难吸收这种学习方法，因为实战中，你需要看的代码量很大，而且你也需要理解前端和后端是怎么进行交互的。所以我是侧身触底的去先学习java开发，然后再结合java代码审计。两者互相结合。


环境准备：下载jdk，下载编译环境软件IDEA
第一阶段 
javaJ编程入门：包括基础编程和高级编程，这个阶段一天学2小时大概一个月学完
https://www.bilibili.com/video/BV1Kb411W75N?p=648
基础弱就从p29 027开始看，开2倍速看，可以跳过复习章节，可以全程都用idea，看一遍是很难记住的，养成好习惯记好笔记
重点内容包括以下：
流畅控制、数组、类、java三大特性：封装、多态、继承、抽象类、接口、实现类、枚举类、注解、容器 list和map、常用类、包装类、泛型 泛型类、泛型方法、反射、静态和动态代理、java8新特性
 、lamba表达式、方法引用、流式编程

第二阶段
JDBC：主要是熟悉怎么和数据库进行交互，一天学2小时一周学完
https://www.bilibili.com/video/BV1eJ411c7rf?p=52&spm_id_from=pageDriver


第三阶段 
javaweb：熟悉前端和后端的数据是怎么进行交互的，包括重要知识：filter、servlet、listen、EL表达式、AJAX
https://www.bilibili.com/video/BV1Y7411K7zz?p=129
跳过html、css、js部分，从p99 开始

第四阶段
SSM框架：现在后端开发都是用框架进行开发的，框架开发会比原生java开发要简化很多，很有很多注解和底层的概念：IOC、AOP
SSM指的就是：  mybatis、spring、springMVC，当然还有SSH框架
底层源码讲解：https://www.bilibili.com/video/BV1yg411T7mv?from=search&seid=16222435368457372851&spm_id_from=333.337.0.0
SSM框架简单使用：https://www.bilibili.com/video/BV1WZ4y1P7Bp?p=47
这两个视频是不一样，一个是讲源码的，一个是教你怎么入门使用，建议先看入门使用，后续有时间了再看看源码

第五阶段
springboot：其实也就是整合了ssm框架，让开发更简单了
https://www.bilibili.com/video/BV1PE411i7CV?p=4


看到第四阶段就可以开始审一些CMS了，这里推荐几个：
https://gitee.com/aaluoxiang/oa_system?_from=gitee_search
https://gitee.com/macrozheng/mall?_from=gitee_search
https://gitee.com/oufu/ofcms?_from=gitee_search
其他框架jfnal：https://gitee.com/jflyfox/jfinal_cms?_from=gitee_search
原生java开发：https://gitee.com/o2oa/O2OA


后续阶段：
1.需要接触spring cloud看着来吧，这块涉及到的框架太多了，毕竟也不是走开发
  注册中心     
  配置中心        
  服务熔断        
  服务调用        
  分布式消息      
  消息总线        
  服务路由（网关） 
  分布式事务
 
 2.可以自己开发一个系统，熟悉一些代码思想，有利于自己更快看懂别人的代码
 3.可以学习一下java安全相关的，反序列利用链、内存马、漏洞回显、fastjson反序列化漏洞、shiro反序列化漏洞等等
 4.关于自动化代码审计，但代码审计做久了，追求的就是自动化了，我自己也学习了codeql，编辑自己的审计规则，后续大部分漏洞可以先自动化审计一遍，提高自己的审计效率

