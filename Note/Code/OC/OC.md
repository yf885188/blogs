# OC与Foundation
- 写OC程序需要使用Foundation框架
- 框架是由很多类组成的库

## import与include的区别
- include与C/C++中类似，看成宏，直接展开
- import则会检查是否已经导入过

## 方法和消息
在OC中，需要执行方法里的代码，首先需要发送一条消息给包含这个方法的对象或类。

## 消息发送
![][MethodCall]

[MethodCall]: ./MethodCall.png

- 传递实参：[接收方 选择器:实参]
- 多个实参：类似Python中使用 形参:实参的方式
- 消息嵌套：由内到外
- alloc和init必须嵌套发送

## 编程规范
使用驼峰法：
- 类名首字母要大写

## 变量
- nil：OC中不用刻意检查nil的情况，nil的方法调用直接返回0
- id：指向任意类型OC对象的指针

# 对象与内存
## 内存管理——ARC
ARC：自动引用计数，automatic reference counting

# 类
跟Python中类似，一个文件算一个类模块。

- 成员变量在类中
- 方法放在类外
  - setter
  - getter
- self