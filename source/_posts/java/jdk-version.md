---
title: JDK各版本主要特性汇总
comments: true
mp3: /music/blog.mp3
categories: 
  - JAVA
keywords: 
  - JDK
date: 2021-09-14 10:51:48
updated: 2021-09-14 10:51:48
tags:
---

JDK各版本主要特性汇总
## JDK 17(LTS)

+ 特定于上下文的反序列化过滤器
+ Vector API 
+ 外部函数和内存 API
+ 弃用安全管理器以进行删除
+ 删除实验性 AOT 和 JIT 编译器
+ 密封类
+ 删除 RMI 激活
+ 模式匹配switch
+ JDK 内部的强封装
+ 弃用 Applet API 以进行删除
+ JDK 移植到 MacOS/AArch64
+ 新的 macOS 渲染管线
+ 增强型伪随机数生成器
+ 恢复始终严格的浮点语义

## JDK 16

+ 允许在 JDK C ++源代码中使用 C ++ 14功能
+ 掉ZGC线程堆栈处理从安全点到并发阶段
+ 增加 Unix 域套接字通道
+ 弹性元空间能力
+ 提供用于打包独立 Java 应用程序的 jpackage 工具
## JDK 15

+ EdDSA 数字签名算法
+ Sealed Classes（封闭类，预览）
+ Hidden Classes（隐藏类）
+ 移除 Nashorn JavaScript引擎
+ 改进java.net.DatagramSocket 和 java.net.MulticastSocket底层实现

## JDK 14

+ instanceof模式匹配
+ Record类型，类似于Lombok 的@Data注解
+ Switch 表达式-标准化
+ 改进 NullPointerExceptions提示信息
+ 删除 CMS 垃圾回收器
## JDK 13

+ Switch 表达式扩展（引入 yield 关键字）
+ 文本块升级 """
+ SocketAPI 重构
+ FileSystems.newFileSystem新方法
+ 增强 ZGC 释放未使用内存
## JDK 12

+ Switch 表达式扩展
+ 新增NumberFormat对复杂数字的格式化
+ 字符串支持transform、indent操作
+ 新增方法Files.mismatch(Path, Path)
+ Teeing Collector
+ 支持unicode 11

## JDK 11(LTS)

+ 增加一些符串处理方法
+ 用于 Lambda 参数的局部变量语法
+ Http Client重写，支持HTTP/1.1和HTTP/2 ，也支持 websockets
+ 可运行单一Java源码文件，如：java Test.java
+ ZGC：可伸缩低延迟垃圾收集器
+ 支持 TLS 1.3 协议
## JDK 10

+ 局部变量类型推断
+ 不可变集合的改进
+ 并行全垃圾回收器 G1
+ 线程本地握手
+ Optional新增orElseThrow()方法
+ 类数据共享
+ Unicode 语言标签扩展
+ 根证书
## JDK 9

+ 模块化
+ 提供了List.of()、Set.of()、Map.of()和Map.ofEntries()等工厂方法
+ 接口支持私有方法
+ Optional 类改进
+ 多版本兼容Jar包
+ JShell工具
+ try-with-resources的改进
+ Stream API的改进
## JDK 1.8(LTS)

+ lambada表达式
+ 函数式接口
+ 方法引用
+ 默认方法
+ Stream API 对元素流进行函数式操作
+ Optional 解决NullPointerException
+ Date Time API
+ 重复注解 @Repeatable
+ Base64
+ 使用元空间Metaspace代替持久代（PermGen space）
## JDK 1.7

+ switch 支持String字符串类型
+ try-with-resources，资源自动关闭
+ 整数类型能够用二进制来表示
+ 数字常量支持下划线
+ 泛型实例化类型自动推断,即”<>”
+ catch捕获多个异常类型，用（|）分隔开
+ 全新的NIO2.0 API
+ Fork/join 并行执行任务的框架
## JDK 1.6

+ java.awt新增Desktop类和SystemTray类
+ 使用JAXB2来实现对象与XML之间的映射
+ 轻量级 Http Server API
+ 插入式注解处理API(lombok使用该特性来实现的)
+ STAX，处理XML文档的API
+ Compiler API
+ 对脚本语言的支持（ruby, groovy, javascript）
## JDK 1.5

+ 泛型(本质是参数化类型，解决不确定具体对象类型的问题)
+ 增强的for循环（for-each）
+ 自动装箱和自动拆箱
+ 类型安全的枚举(enum)
+ 可变长度参数
+ 静态引入（import static）
+ 元数据（注解）
+ 线程并发库（java.util.concurrent）