title: iOS动态添加属性
date: 2015-09-05 09:03:18
tags:
- iOS
- runtime
---

我们都知道在objc中，可以通过category来动态增加方法。但是有些时候，可能需要动态地增加一些属性。这时候我们就需要一点技巧。  

我们首先查看**objc/runtime.h**中有这么一段代码（通过快捷键command＋shift＋o，输入runtime打开）：  
<!-- more -->

```objc
/** 
 * Sets an associated value for a given object using a given key and association policy.
 * 
 * @param object The source object for the association.
 * @param key The key for the association.
 * @param value The value to associate with the key key for object. Pass nil to clear an existing association.
 * @param policy The policy for the association. For possible values, see “Associative Object Behaviors.”
 * 
 * @see objc_setAssociatedObject
 * @see objc_removeAssociatedObjects
 */
OBJC_EXPORT void objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy)
    __OSX_AVAILABLE_STARTING(__MAC_10_6, __IPHONE_3_1);

/** 
 * Returns the value associated with a given object for a given key.
 * 
 * @param object The source object for the association.
 * @param key The key for the association.
 * 
 * @return The value associated with the key \e key for \e object.
 * 
 * @see objc_setAssociatedObject
 */
OBJC_EXPORT id objc_getAssociatedObject(id object, const void *key)
    __OSX_AVAILABLE_STARTING(__MAC_10_6, __IPHONE_3_1);
```

这两个C方法分别通过为对象提供的key来设置和获取对应的value。在设置的方法中，还有一个policy的参数，用于内存管理的策略。  
这种机制叫做关联引用（Associative References）。

下面来举个关联引用的参考栗子：  

```objc
#import <Foundation/Foundation.h>
#import <objc/runtime.h>

@interface Person : NSObject
@property (nonatomic, copy) NSString *name;
@end

@implementation Person
@end

@interface Person (EmailAdress)
@property (nonatomic, copy) NSString *emailAddress;
@end

@implementation Person (EmailAdress)
static char emailAddressKey;
- (NSString *)emailAddress {
    return objc_getAssociatedObject(self, &emailAddressKey);
}

- (void)setEmailAddress:(NSString *)emailAdress {
    objc_setAssociatedObject(self, &emailAddressKey, emailAdress, OBJC_ASSOCIATION_COPY);
}
@end

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        Person *p1 = [[Person alloc] init];
        p1.emailAddress = @"Ningbo";
        NSLog(@"%@", p1.emailAddress);
    }
    return 0;
}
```

如果要使用关联引用，必须引入objc/runtime.h头文件。关联引用是基于key对内存地址，我们不关心key的值是什么，它只须有一个唯一且不可改变的地址。这也就是为什么有static关键字，当然你也可以使用const关键字。  

虽然关联引用给我们带来动态增加属性的好处，但是也有不足之处。那就是无法通过**encodeWithCoder:**序列化category。

