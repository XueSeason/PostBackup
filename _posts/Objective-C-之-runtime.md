title: Objective-C 之 runtime
date: 2015-10-05 10:28:57
tags:
- Objective-C
- runtime
- iOS
---

Objective-C 作为一门动态语言，其精华在于 runtime 机制。  
简单举个栗子：

```
[receiver message];
```

以上代码会被编译时转化为 C 代码：

```
objc_msgSend(receiver, selector, arg1, arg2, ...)
```

此时，我们并不能确定 reveiver 是否能响应指定的消息。  
只有在运行时，才能确定方法是否实现，所以容易导致运行时崩溃的问题。
<!--more-->
# 基本概念

首先，我们要明确几个关键词的意思：

- 消息（message），函数名和参数列表的抽象，没有真正意义上的代码描述。
- 方法（method），.m 文件内实现的真正意义上的方法。
- 方法选择器（selector），SEL 类型，描述一个特定的方法或者消息。

## SEL
我们可以打开 objc.h 文件中查看 SEL 到底是个什么鬼。  

```
/// An opaque type that represents a method selector.
typedef struct objc_selector *SEL;
```

可以看出 SEL 代表指向 objc_selector 结构体指针的类型。

我们看看下面的代码输出的结果：

```
SEL selector = @selector(hello);
NSLog(@"%s", (char *)selector);
```

没错，大胆地想象，其实 SEL 可以被视为一个 char* 类型的指针，所指向的字符串内容就是方法的名称。  
这简直是坑爹啊有没有？如果两个方法名相同但是参数类型不同，这在其它语言里再普通不过的重载，在 Objective-C 中被视为编译错误。  
原因就是 SEL 只认方法名称作为唯一标识，根本无视你的参数类型。所以编译器这关直接将你 block 掉来避免后续错误。  
所以我们就不难理解为什么会有 Objective-C 中一些奇葩的方法命名规则：

```
- (void)setNumberWithIntValue:(int)temp;
- (void)setNumberWithFloatValue:(float)temp;
```

既然 SEL 这么弱智的玩意，设计出来是为了什么？  

每个方法确定一个唯一的一个 SEL 变量，这些许许多多的方法也就有了许许多多的 SEL，这些 SEL 组成一张 Hash 表。  
不懂 Hash 表的原理，可以看下数据结构相关的知识，这里姑且看做一个高查询效率的表格。  
现在我们就可以知道，SEL 的存在其实是优化查询效率。  

## IMP
继续找到 objc.h 中的定义：

```
/// A pointer to the function of a method implementation. 
typedef id (*IMP)(id, SEL, ...); 
```
IMP 是一个函数指针，这个被指向的函数包含一个接受消息的对象id（self 指针），调用方法的 SEL 指针，以及可变个数的方法参数。  
可以想象，IMP 是消息最终调用的执行代码，是方法真正的实现代码。  

NSObject 类中的 methodForSelector: 方法返回一个指向相应 SEL 所对应方法的指针，即返回的指针和赋值的变量类型完全一致，包括方法的参数类型和返回值类型。  
用 IMP 的方式，省去了 runtime 消息传递的开销，比直接向对象发送消息高效一些。  
