### 《Java并发编程实战》读书笔记

#### 线程安全性

##### 什么是线程安全性？

```txt
  当多个线程访问某个类时，不管运行时环境采取何种调度方式或者这些线程将如何交替执行，并且在主调代码中不需要做额外的同步或者协同，这个类都能表现出正确的行为，那么就称这个类是线程安全的。
```

​             