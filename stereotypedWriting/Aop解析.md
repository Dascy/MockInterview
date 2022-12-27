### Aop解析

#### 简介

AOP(Aspect Oriented Programming) 意为面向切面变成。通过预编译方式和运行期间动态代理实现程序功能的统一维护的一种技术,通过动态代理的技术来实现的在运行期间对植入增强代码。

#### 代理模式（实现原理）

代理模式分为静态代理和动态代理。

- 静态代理

​        在运行前已经有相关代码，通过对代码的编译，在程序运行前生成代理类的.class文件

- 动态代理

​       通过反射机制来实现的。主要的代理有两种：JDK的动态代理和CGLIB的动态代理。JDK的代理只支持代理接口。不支持类的代理。

#### AOP的使用场景

- 事务的管理控制
- 记录日志  
- 监控性能   在方法调用前后记录调用时间，方法执行太长或超时报警
- 权限控制  方法执行前验证是否有权限执行当前方法，没有则抛出没有权限执行异常，由业务代码捕捉
- 缓存    缓存某方法的返回值，下次执行该方法时，直接从缓存里获取

#### AOP的相关技术点

| Joinpoint        | 连接点,运行程序的一个方法或者异常处理。                     |
| ---------------- | ---------------------------------------- |
| **Pointcut**     | 切入点，进行advice的位置。程序的所有方法都是连接点。对程序的操作是以某一个连接点进行的。这个连接点就是切点 |
| **Advice**       | 通知，对切面连接点进行的操作。                          |
| **Introduction** | 增强，通过introduction可以对切面新增方法或者字段           |
| **Target**       | 目标                                       |
| **Weaving**      | 织入，切面连接到应用程序的过程                          |
| **Proxy**        | 代理                                       |
| **Aspect**       | 切面。由切点组成的对象                              |

#### AOP的失效场景

- 权限问题，未使用public修饰方法
- 方式使用了final修饰
- 内部调用方法
- 未被Spring对象管理
- 多线程调用   @Async 
- 手动处理异常
- 手动抛出异常与AOP处理的不同
- 抛出自定义异常

#### Spring AOP 的代码实现

配置方法一

```java
package com.dascy.learn.aop.aspect;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;

/**
 * @author djz
 * @Title: Myaspect
 * @ProjectName djz-nacos
 * @Description: TODO
 * @date 2022/5/3013:18
 */
//切面类
@Component
@Aspect
public class Myaspect {

    //前置通知
    @Before("execution(* com.dascy.learn.aop..*dao..*(..))")
    public void  sayHellow(){
        System.out.println("前置方法执行");
    }

  //后置通知
    @After("execution(* com.dascy.learn.aop..*dao..*(..))")
    public void sayGoodBye(){
        System.out.println("后置方法执行");
    }
  //环绕通知
    @Around("execution(* com.dascy.learn.aop..*dao..*(..))")
    public void Around(ProceedingJoinPoint point){

        System.out.println("环绕前置");

        try {
            Object proceed = point.proceed();
        } catch (Throwable throwable) {
            throwable.printStackTrace();
        }
        System.out.println("环绕后置");
    }
}
```

配置方法二

```java
@Component
@Aspect
public class Myaspect {


    @Pointcut(value = "execution(* com.dascy.learn.aop..*dao..*(..))")
    public void log(){
        System.out.println("切点方法执行中");
    }

    @Before("log()")
    public void  sayHellow(){
        System.out.println("前置方法执行");
    }

    @After("log()")
    public void sayGoodBye(){
        System.out.println("后置方法执行");
    }

    @Around("log()")
    public void Around(ProceedingJoinPoint point){

        System.out.println("环绕前置");

        try {
            Object proceed = point.proceed();
        } catch (Throwable throwable) {
            throwable.printStackTrace();
        }
        System.out.println("环绕后置");
    }
}
```

#### 关于JoinPoint的相关说明

| Signature getSignature(); | 获取签名， |
| ------------------------- | ----- |
|                           |       |
|                           |       |
|                           |       |

#### Spring AOP内部调用失效及解决方法

##### 关于为什么想写这个的原因

​      看新的项目组代码的时候，遇到了类内部调用方法。没有直接使用`this` 去调用，而是使用了beanFactory注入。虽然之前知道this的调用会导致失效，想要测试一下都有哪些方法可以解决。

##### 简单的测试代码

###### Dao

```java
@Component
public class daoImpl {

    public void  sayHello(){
        System.out.println("目标方法体执行");
    }

    public void aopTest() {
        System.out.println("目标方法执行中");
        this.say();
    }

    public void say(){
        System.out.println("say方法执行中");
    }
}
```

###### controller

```java
/**
 * @author djz
 * @Title: AopInvalidTestController
 * @ProjectName djz-nacos
 * @Description: TODO
 * @date 2022/11/2915:07
 */
@RestController
@RequestMapping("/aop/test")
public class AopInvalidTestController {

    @Autowired
    daoImpl dao;

    @GetMapping("demo")
    public String demo(){
        dao.aopTest();
        dao.say();
        return "success";
    }
}
```

###### AOP

```java
package com.dascy.learn.aop.aspect;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;

/**
 * @author djz
 * @Title: Myaspect
 * @ProjectName djz-nacos
 * @Description: TODO
 * @date 2022/5/3013:18
 */
@Component
@Aspect
public class Myaspect {


    @Pointcut(value = "execution(* com.dascy.learn.aop..*dao..*(..))")
    public void log(){
        System.out.println("切点方法执行中");
    }

    @Before("log()")
    public void  sayHellow(){
        System.out.println("前置方法执行");
    }

    @After("log()")
    public void sayGoodBye(){
        System.out.println("后置方法执行");
    }

    @Around("log()")
    public void Around(ProceedingJoinPoint point){

        System.out.println("环绕前置");

        try {
            Object proceed = point.proceed();
        } catch (Throwable throwable) {
            throwable.printStackTrace();
        }
        System.out.println("环绕后置");
//    }

//    @Before("execution(* com.dascy.learn.aop..*dao..*(..))")
//    public void  sayHellow(){
//        System.out.println("前置方法执行");
//    }
//
//    @After("execution(* com.dascy.learn.aop..*dao..*(..))")
//    public void sayGoodBye(){
//        System.out.println("后置方法执行");
//    }
//
//    @Around("execution(* com.dascy.learn.aop..*dao..*(..))")
//    public void Around(ProceedingJoinPoint point){
//
//        System.out.println("环绕前置");
//
//        try {
//            Object proceed = point.proceed();
//        } catch (Throwable throwable) {
//            throwable.printStackTrace();
//        }
//        System.out.println("环绕后置");
    }
}

```

###### 执行结果

```txt
环绕前置
前置方法执行
目标方法执行中
say方法执行中
后置方法执行
环绕后置
环绕前置
前置方法执行
say方法执行中
后置方法执行
环绕后置
```

可以看到。内部调用没有触发切面。this调用的是实例类。不是代理类。所以没有触发。明白了这点。我们可以通过使用代理类来完成切面触发

###### 改写dao的设计

```java
@Component
public class daoImpl {


    @Lazy
    @Autowired
    private daoImpl dao;

    public void  sayHello(){
        System.out.println("目标方法体执行");
    }


    public void aopTest() {
        System.out.println("目标方法执行中");
        this.dao.say();
    }

    public void say(){
        System.out.println("say方法执行中");
    }
}

```

这里做了内部注入。会有循环依赖的问题。所以使用了`@lazy` 注解来解决。也可以通过ApplicationContext获取bean，通过bean调用内部方法，就使用了bean的代理类。