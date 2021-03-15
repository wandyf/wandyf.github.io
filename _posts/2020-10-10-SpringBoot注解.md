---
layout: post
title: SpringBoot注解
# background-image: 
date: 2020-10-10
published: true
category: Language
tags:
        - springboot
---

# JAVA系统注解

1. @Override 重写父类方法
2. @Deprecated 方法已废弃
3. @SuppressWarnings 忽略某类警告
    - unchecked:   未检查的转化
    - unused:      未使用的变量
    - resource:    泛型（未指定类型）
    - path:        原文件路径中有不存在的路径
    - deprecation: 使用了某些不赞成使用的类或方法
    - fallthrough: switch语句执行到底，不会遇到break语句
    - serial:      实现了Serializable,但未定义serialVersionUID
    - rawtypes:    没有传递带有泛型的参数
    - all:         代表全部类型警告


# SpringBoot常用注解

-  @RestController: 作用于类，相当于@ResponseBody + @RestController
    >- 用于返回JSON、XML等数据
    >- 不能返回HTML页面
-  @Controller:     作用于类，声明此类是一个SpringMVC Controller对象
    >- 主要用于构建MVC模式的程序，标注控制器层
-  @Service:        作用于类，声明业务处理类（实现非接口类）
    >- 用于标注服务层、处理业务逻辑
-  @Respository:    作用于类，声明数据库访问类（实现非接口类）
    >- 用于标注数据访问层
-  @Component:      作用于类，代表是Spring管理的类，常用于无法@Service、@Respository的Spring管理的类
    >- 将普通POJO(Plain Ordinary Java Object，简单Java对象)实例化到Spring容器中
    >- 当类不属于@Controllerhe @Service等时，就可以使用@Component标注
    >- 可以配合CommandLineRunner使用，在程序启动后执行一些基础任务
-  @Configuration:  作用于类，声明此类是一个配置类，常与@Bean配合使用
    >- 用于标注配置类，可以由Spring容器自动处理
    >- 作为Bean的载体，用来指示一个类声明一个或多个@Bean方法，运行时为Bean生成BeanDefinition和服务请求
-  @Resource:       作用于类、属性或构造函数参数，默认按byName自动注入
    >- 用来装配Bean，也可以写在字段上或Setter方法上
-  @Autowired:      作用于类、属性、或构造函数参数，默认按byType自动注入
    >- 表示被修饰的类需要注入对象
    >- Spring会扫描所有被@Autowired标注的类，然后根据类型在loC容器中找到匹配的类进行注入
    >- 被@Autowired标注的类不需要再导入文件
-  @RequestMapping: 作用于类或方法，若作用于类，则表示所有响应请求的方法都以该地址作为父路径
    >- 用来处理请求地址映射，共有六个属性
    >- Params: 指定Request中必须包含某些参数，才让该方法处理
    >- Headers:指定Request中必须包含某些指定的Header值，才让该方法处理
    >- Value: 指定请求的实际地址，指定的地址可以是URI Template模式
    >- Method: 指定请求的Method类型，如GET、POST、PUT、DELETE
    >- Consumes: 指定处理请求的提交内容类型Content-Type,如application/json，text，html
    >- Produces: 指定返回的内容类型，只有当Request请求头中的Accept类型中包含指定类型时才返回
- @Transactional:  作用于类或方法，用于处理事务
    >- 可以用在接口、接口方法、类、类方法上
    >- Spring不建议在接口方法上使用，因为该注解只有在使用基于接口的代理时才会生效
    >- 若异常被捕获(try{}catch{})，则事务不回滚，若想回滚，则须再抛出异常(try{}catch{throw Exception})
- @Qualifier       作用于类或属性，为Bean指定名称，随后再通过名字引用Bean
    >- 用于标注哪个实现类是需要注入的
    >- @Qualifier的参数名称为被注入的类中的注解@Service标注的名称
    >- 和@Autowired一起使用 `@Autowired  @Qualifier("someService")`
    >- 和@Resource不同，@Resource自带name属性


# 使用在方法上的注解

- @RequestBody: 作用于方法参数前
    >- 用来处理application/json application/xml等Content-Type类型的数据
    >- 意味着HTTP消息是JSON/XML格式的，需将其转化为指定类型参数
    >- 可以将请求体中的JSON/XML字符串绑定到相应的Bean上，也可以将其分别绑定到对应字符串上
- @PathVariable: 作用于方法参数前
    >- 将URL获取的参数映射到方法参数上
    >- 用于获取路路径中的参数
- @Bean: 作用于方法上
    >- 声明该方法返回结果是一个由Spring容器管理的Bean
    >- 代表产生一个Bean，并交给Spring管理
    >- 用于封装数据，有Getter、Setter方法，对应Model层
- @ResponseBody: 作用于方法上
    >- 通常用来返回JSON/XML数据
    >- 通过HttpMessageConverter转换器将控制器中方法返回的对象转换为指定格式后写入Response对象的body区


# 其他注解

- @EnableAutoConfiguration: 入口类/类名上，用于提供自动配置
- @SpringBootApplication:   入口类/类名上，用来启动入口类Application
- @EnableScheduling:        入口类/类名上，用于开启计划任务，Spring通过@Scheduled支持cron,fixDelay,fixRate等类型计划任务
- @EnableAsync:             入口类/类名上，用于开启异步注解
- @ComponentScan:           入口类/类名上，用于扫描组件，可以自动发现和装配Bean
- @Aspec:                   入口类/类名上，标注切面，可以用来配置事务、日志、权限验证、在用户请求时做一些处理
- @ControllerAdvice:        类名上，包含@Component，可以被扫描到，统一处理异常
- @ExcepitionHandler：      方法上，表示遇到这个异常就执行该方法
- @Value:                   属性上，用于获取配置文件中的值
