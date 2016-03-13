title: iOS内存管理
date: 2015-07-10 08:43:21
tags: iOS
---
# 手动管理内存原则

## 引用计数工作原理
Objective-C内存管理不像Java和C#有垃圾回收机制，而是依赖引用计数器。在Objective-C对象中有一个retainCount来统计当前引用数。

虽然苹果正在用ARC取代MRC，但是了解手动内存管理的基本原理还是必要的。手动管理虽然难度较大，但是只要遵守一个法则：==谁创建，谁释放==，以及注意以下几个知识。

- 当一个对象在创建之后它的引用计数器为1
- 当调用alloc、retain、new、copy方法递增引用计数器
- 当调用release方法后递减引用计数器
- 当调用autorelease方法待稍后清理“自动释放池”时，再递减引用计数器
- 如果一个对象的引用计数器为0，则系统会自动调用这个对象的dealloc方法来销毁这个对象。
<!-- more -->
注意：  
如果对象被释放之后，最后引用它的变量应该手动设置为nil，否则可能造成野指针错误，且给空对象发送消息是不会引起错误的，一般人很容易忽略。

示例代码:

```
NSNumber *number = [[NSNumber alloc] initWithInt:1337];
[array addObject:number];
[number release];
number = nil;
```

## 属性存取方法中的内存管理
上述示例代码中数组的addObject方法中内部会调用retain方法来保留number对象。一般在属性的设置方法中用到相同的机制。
例如:

```
- (void)setFoo:(id)foo {
    [foo retain];
    [_foo release];
    _foo = foo;
}
```

这里需要注意的是方法调用的顺序，先retain新值，再进行release。否则当两个指针在一开始就指向同一块内存，而先执行release会将内存释放，此对象永久回收，之后的retain也就没有意义了。  
记住养成习惯：==先保留新值，再释放旧值，最后设置实例变量==。

## 自动释放池
当一个方法 返回对象时，想稍后递减计数器，避免对象被提前释放。我们就要用到autorelease了。  
示例代码:

```
- (NSString *)stringValue {
    NSString *str = [[NSString alloc] initWithFormat:@"I am this: %@", self];
    return [str autorelease];
}
```

我们需要这样调用该方法:

```
NSString *str = [[self stringValue] retain];
// ...
[str release];
```

我们需要了解的是:

- autorelease方法不会改变对象的引用计数器，只是将这个对象放到自动释放池中。
- 自动释放池实质是当自动释放池销毁后调用对象的release方法，不一定就能销毁对象（例如如果一个对象的引用计数器>1则此时就无法销毁）
- 由于自动释放池最后统一销毁对象，因此如果一个操作比较占用内存（对象比较多或者对象占用资源比较多），最好不要放到自动释放池或者考虑放到多个自动释放池
- Objective-C类库中的静态方法一般都不需要手动释放，内部已经调用了autorelease方法

# 自动内存管理（ARC）
ARC技术让保留和释放操作自动添加，把引用计数的工作丢给编译器。实际上，ARC在调用retain、release等方法并不通过普通的Objective-C消息派发机制，而是直接调用底层的C语言版本。

## 使用ARC必须遵循的方法命名规则
若方法以下列单词开头，则其返回对象归调用者所有:

- alloc
- new
- copy
- mutableCopy

若方法名不以上述四个单词开头，则表示其返回对象不归调用者所有。在这种情况下，返回的对象会自动释放，也就是说其值在跨越方法调用边界后依然有效。要想使对象继续存活，必须令调用者保留它。

下面是示例代码:

```
// 该方法以new开头，为了让计数器保持为1，ARC不会继续添加retain、release、autorelease等方法
+ (EOCPerson *)newPerson {
    EOCPerson *person = [[EOCPerson alloc] init];
    return person;
}

// 该方法没有用所规定的单词开头，ARC会将return语句修改为return [person autorelease];
+ (EOCPerson *)somePerson {
    EOCPerson *person = [[EOCPerson alloc] init];
    return person;	
}

- (void)doSomething {
    EOCPerson *personOne = [EOCPerson newPerson];
    // ...
    // 此时ARC会在这里添加 [personOne release];
    EOCPerson *personTwo = [EOCPerson somePerson];
    // ...
    // 由于personTwo为autorelease，将在释放池中释放
}
```

ARC带来的好处不仅仅只是自动帮忙添加保留和释放，还带来性能上极大的优化。ARC能够抵消同一个对象上多次保留和释放操作。在运行期同样得到优化。

```
// _myPerson为强引用类型，调用personWithName:方法返回对象前会执行autorelease操作
_myPerson = [EOCPerson personWithName:@"Bob Smith"];
```
此时，为了保留对象，应该随后添加[\_myPerson retain]操作，不过autorelease和retain都是多余的。为了提升性能，应该将两者抵消删去。  
为了优化代码，ARC在返回一个自动释放的对象时，不会调用autorelease方法，而是调用objc\_autoreleaseReturnValue。此函数会监视当前方法返回之后的后续代码。
若发现后续代码要在返回的对象上执行retain操作，则设置全局变量中的一个标志位，而不执行autorelease操作。与此同时，也不执行retain操作，而是改为执行objc\_retainAutoreleasedReturnValue函数。此函数要检测刚才设置的标志位，若已经设置，则不执行retain操作，并且重置标志位。

```
+ (EOCPerson)personWithName:(NSString *)name {
    EOCPerson *person = [[EOCPerson alloc] init];
    person.name = name;
    objc_autoreleaseReturnValue(person);
}

_myPerson = [EOCPerson personWithName:@"Bob Smith"];
_myPerson = objc_retainAutoReleasedReturnValue(tmp);
```

## 变量的内存管理关键字

- \_\_strong: 默认关键字，保留此值
- \_\_unsafe\_unretained: 不保留此值，不安全可能会出现野指针
- \_\_weak: 不保留此值，安全，对象被回收那么该变量指向nil
- \_\_autoreleasing: 把对象按引用传递给方法时，使用这个特殊的修饰符。此值在方法返回时自动释放