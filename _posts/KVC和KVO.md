title: KVC和KVO
date: 2015-07-12 08:44:55
tags: iOS
---
在iOS的面试中KVC和KVO是经常被问到的内容。  
明天本人恰好有一个iOS实习面试，于是准备了有关KVC和KVO的详细资料。

---

# KVC
KVC的全称为Key Value Coding，即键-值编码。以字符串形式间接操作对象的属性。KVC会自动判断对象属性的类型，完成相应的转换

我们首先定义一个User类，接口部分代码:

```
#import <Foundation/Foundation.h>

@interface User : NSObject

@property (nonatomic, copy) NSString * name;
@property (nonatomic, copy) NSString * pass;
@property (nonatomic, copy) NSDate *birth;

@end
```
<!-- more -->
我们在main函数中使用KVC:

```
#import <Foundation/Foundation.h>
#import "User.h"

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        User *user = [[User alloc] init];
        // 使用KVC方式设置user对象的属性
        [user setValue:@"Season" forKey:@"name"];
        [user setValue:@"1234" forKey:@"pass"];
        [user setValue:[[NSDate alloc] init] forKey:@"birth"];
        // 使用KVC方式获取user对象的属性
        NSLog(@"name is : %@", [user valueForKey:@"name"]);
        NSLog(@"pass is : %@", [user valueForKey:@"pass"]);
        NSLog(@"birth is : %@", [user valueForKey:@"birth"]);
    }
    return 0;
}
```

输出结果为:

```
name is : Season
pass is : 1234
birth is : 2015-07-12 07:45:46 +0000
```

对于setValue:forKey:方法，底层的实现机制如下:

1. 先调用对应键值的setter方法完成设置
2. 如果没有setter方法，则搜索对应以下划线开头的成员变量，_keyName
3. 如果既没有setter方法，也没有对应的成员变量，那么就搜索无下划线的成员变量，keyName
4. 如果上述3条都没有找到，系统会执行该对象的setValue:forUndefinedKey:方法，引发一个异常

对于valueForKey:方法，底层的实现机制同样也是通过`getter方法 -> _keyName -> keyName -> valueForUndefinedKey:方法`

需要注意的是，如果使用setValue:forKey:方法时，value的值为nil，如果理想的属性类型是NSString类型，则可以正常运行；但是如果为其它类型，程序会引发`NSInvalidArgumentException`异常。  

当程序尝试为某个属性设置为nil时，如果该属性不接受nil值，那么程序会自动执行该对象的setNilValueForKey:方法。如果我们不重写该方法，那么就会抛出上面的异常。

对于复合类型，我们可以使用setValue:forKeyPath:和valueForKeyPath:方法，这里不再赘述。  

为什么我们要使用KVC呢？  
KVC的性能比setter和getter更差，使用KVC的优势在于简洁，适合提炼一些通用性质的代码，同时字符串的形式操作对象属性，具有极高的灵活性。

# KVO
MVC框架中，在数据模型组件的状态发生改变时，视图组件能动态更新自己，及时显示最新的数据模型组件的内容。

为了解决上述需求，通常有两种实现方案:

- 让数据组件持有一个视图组件的引用，数据模型组件可以回调视图组件的特定方法，视图组件通过回调方法即使更新内容
- 利用iOS的消息中心（NSNotificationCenter），让数据模型组件作为消息发送者，视图组件作为消息接收者。每当数据模型组件的内部状态发生改变时，数据模型组件向消息中心发送消息，视图组件接收到消息中心的消息后动态更新自己

但是以上两个方案存在不足:

- 第一种方案：视图模型组件和数据模型组件相互存在依赖，增加了耦合，是一种非常糟糕的设计
- 第二种方案：视图模型组件和数据模型组件与iOS的消息中心耦合，每当数据模型组件的状态改变时都要向消息中心发送消息

KVO（Key Value Observing）机制提供了更优秀的解决方案。

NSKeyValueObserving协议提供了KVO的支持，NSObject遵守了该协议，因此iOS所有继承NSObject的子类都支持KVO。

KVO主要有以下几个常用方法:

- addObserver:forKeyPath:options:context: 注册一个监听器用于监听指定的key路径
- removeObserver:forKeyPath: 为key路径删除指定的监听器
- removeObserver:forKeyPath:context: 为key路径删除指定的监听器。多了个Context参数
- observeValueForKeyPath:ofObject:change:context: key路径对应属性值状态改变时，激发回调此函数

KVO编程的编程步骤也非常简单:

1. 为被监听对象（一般为数据模型组件）注册监听器
2. 重写监听器的observeValueForKeyPath:ofObject:change:context:方法

KVO的优点有:

1. 减少代码量
2. KVO是观察者设计模式中的一种，有利于业务逻辑于视图控制之间的解耦。

一般KVO设计思路如下：

1. 定义一个对象和一些该对象的属性
2. 监听对象的属性
3. 设置一个视图组件的触发事件来改变该对象的属性
4. 收到属性改变时发出来的通知
5. 对象销毁的时，移除通知。