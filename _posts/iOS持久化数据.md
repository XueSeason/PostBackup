title: iOS持久化数据
date: 2015-07-11 08:44:08
tags: iOS
---
首先明确两个概念数据结构和存储方式。数据结构指数据存在的形式，简单的如Foundation框架中的数组，字典集合等，复杂的如关系模型，对象图和属性列表等；存储形式指数据存储的媒介，主要为内存和闪存，内存的数据是临时的，程序重启后数据将丢失，但读取速度快，闪存的数据是持久化的，I/O消耗造成读取速度不及内存。将内存中的数据保留到闪存中的过程称之为归档（Archive）。  

在iOS中常用的四个存储方式:

- NSUserDefaults，属性列表，用于存储配置信息
- SQLite，用于存储查询需求较多的数据
- CoreData，用于规划应用中的对象
- 使用基本对象类型定制的个性化缓存方案
<!-- more -->
接下来介绍这四种存储方式

# NSUserDefaults
NSUserDefaults通过单例模式获取对象，也就是说每个应用程序只有一个NSUserDefaults对象，开发者可以通过如下代码获取NSUserDefaults对象:

```
NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];
```

获取NSUserDefaults对象之后，就可以通过调用如下方法获取和设置参数:
![](/images/iOS持久化数据/NSUserDefaults.png)

NSUserDefaults所有数据操作都在内存中进行，速度快，设置完后需要调用一个归档方法：- (void)synchronize。

归档会以plist格式将数据保存到应用目录的/Library/Preferences/[App_Bundle_Identifier].plist中。  

# SQLite
SQLite不是客户端/服务器架构的数据库引擎，而是被集成在用户程序中的数据库引擎，适用于资源有限的设备上。作为关系型数据库，SQLite支持SQL标准，并且SQLite数据库只是一个文件。

iOS的SQLite编程使用的是C语言的原生组件，为了使用iOS的SQLite编程API，需要完成一下几步:

- 为项目添加libsqlite3.dylib函数库，可以在Target选中Build Phases中的Link With Libraries中添加
- 在需要使用SQLite API的类文件中导入libsqlite3.dylib，添加代码`#import <sqlite3.h>`

设置好以上几步就可以使用SQLite的API对数据库进行操作了。

以下为创建和插入数据库操作的示例代码:

```
// 获取数据库的路径
NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
NSString *documentsDirectory = [paths objectAtIndex:0];
NSString *dbPath = [NSString stringWithFormat:@"%@/mySqlite.db", documentsDirectory];
NSLog(@"%@", dbPath);
// 打开指定路径的数据库，如果不存在，则在该目录下创建对应的数据库
sqlite3 *database;
sqlite3_open([dbPath UTF8String], &database);

// 使用SQL语句创建表
const char * createSQL = "create table if not exists fruits \
                        (_id integer primary key autoincrement, \
                        name, \
                        color)";
char * errMsg;
int result = sqlite3_exec(database, createSQL, NULL, NULL, &errMsg);

if (result == SQLITE_OK) {
    NSLog(@"创建表成功");
    const char * insertSQL = "insert into fruits values(null, ?, ?)";
    sqlite3_stmt *stmt;
    // 预编译SQL语句，stmt变量保存预编译结果的指针
    int insertResult = sqlite3_prepare_v2(database, insertSQL, -1, &stmt, nil);
    // 如果预编译成功
    if (insertResult == SQLITE_OK) {
        NSLog(@"预编译成功");
        // 为?占位符绑定参数
        sqlite3_bind_text(stmt, 1, "apple", -1, NULL);
        sqlite3_bind_text(stmt, 2, "red", -1, NULL);
        // 执行SQL语句
        sqlite3_step(stmt);
    }
    sqlite3_finalize(stmt);
}
// 关闭数据库
sqlite3_close(database);
```

下面为查询示例代码:

```
// 获取数据库的路径
NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
NSString *documentsDirectory = [paths objectAtIndex:0];
NSString *dbPath = [NSString stringWithFormat:@"%@/mySqlite.db", documentsDirectory];
NSLog(@"%@", dbPath);
// 打开指定路径的数据库，如果不存在，则在该目录下创建对应的数据库
sqlite3 *database;
sqlite3_open([dbPath UTF8String], &database);

// 使用SQL语句查询
const char * selectSQL = "select * from fruits where name like ?";
sqlite3_stmt *stmt;
//预编译SQL语句，stmt保存预编译结果的引用
int queryResult        = sqlite3_prepare_v2(database, selectSQL, -1, &stmt, nil);
NSMutableArray *result = [[NSMutableArray alloc] init];
// 如果预编译成功
if (queryResult == SQLITE_OK) {
    NSLog(@"预编译成功");
    // 为?占位符绑定数据为%red%
    sqlite3_bind_text(stmt, 1, "%%apple%%", -1, NULL);
    // 迭代执行sqlite3_step()函数获取查询结果
    while (sqlite3_step(stmt) == SQLITE_ROW) {
        int fruit_id = sqlite3_column_int(stmt, 0);
        char * fruit_name = (char *)sqlite3_column_text(stmt, 1);
        char * fruit_color = (char *)sqlite3_column_text(stmt, 2);

        Fruit * fruit = [[Fruit alloc] initWithId:fruit_id andName:[NSString stringWithUTF8String:fruit_name] andColor:[NSString stringWithUTF8String:fruit_color]];
        [result addObject:fruit];
    }
}

// 关闭数据库
sqlite3_close(database);

NSLog(@"%@", result);
```

其它更详细的SQLite API可以查阅[SQLite官方文档](https://www.sqlite.org/c3ref/intro.html)

# CoreData
由于SQLite API的操作不符合面向对象思维，类似于J2EE的Hibernate框架，iOS提供了CoreData框架，一个支持持久化的，对象图和生命周期的自动化管理方案。  
CoreData底层的持久化存储方式可以是SQLite数据库，也可以是XML文档，甚至是内存。  
CoreData的使用场景在于：整个应用使用CoreData规划，把应用内的数据通过CoreData建模，完全基于CoreData架构应用。

CoreData的核心概念为实体，必须继承自NSManagedObject类。  
CoreData的核心对象为托管对象上下文（NSManagedObjectContext），所有实体都处于上下文管理中，CRUD操作必须通过上下文完成。而NSManagedObjectContext底层与持久化存储协调器衔接，持久化存储协调器负责管理底层的存储形式（如SQLite）。

CoreData中常用类:

- NSManagedObject，托管对象模型，负责管理整个应用中的所有实体以及实体之间的关系
- NSPersistentStoreCoordinator，持久化存储协调器，负责管理底层的存储文件
- NSManagedObjectContext，托管对象上下文，对实体进行增删改查操作
- NSEntityDescription，实体描述，实体的描述信息，相当于实体的抽象
- NSFetchRequest，抓去请求，封装了查询实体的请求

使用CoreData持久化操作的步骤大致如下:

1. 创建NSManagedObjectModel对象，加载管理应用的托管对象模型
2. 以NSManagedObjectModel对象为基础，根据实际需求创建NSPersistentStoreCoordinator对象，该对象确定CoreData底层数据的存储形式
3. 以NSManagedObjectModel对象为基础，创建NSManagedObjectContext
4. 对于普通的增、删、改操作，需分别新建实体、删除实体、修改实体，然后调用NSManagedObjectContext对象的save:方法保存到底层存储设备
5. 对于查询操作，需要先创建NSFetchRequest对象，再调用NSManagedObjectContext的executeFetchRequest:error:方法执行查询，该方法返回所有匹配条件的实体组成的NSArray。

## 创建CoreData项目
我们不选择Xcode自带的Use Core Data选项，首先创建一个普通的项目。  
接下来我们将这个普通的项目改造为一该CoreData项目:

1. 在Target的Build Phases中添加CoreData.framework框架
2. 为项目添加一个实体模型文件。File -> New -> Core Data -> Data Model
3. 初始化NSManagedObjectModel、NSPersistentStoreCoordinator、NSManagedObjectContext对象。这些核心API对象都属于全局可用对象，因此程序一般会在应用程序委托类中执行初始化

修改应用程序委托类的接口文件，定义三个核心API属性，并增加一个对NSManagedObjectContext对象执行存储的方法，一个获取应用Documents目录下的方法。

```
#import <UIKit/UIKit.h>
#import <CoreData/CoreData.h>

@interface AppDelegate : UIResponder <UIApplicationDelegate>

@property (strong, nonatomic) UIWindow *window;
// CoreData三个核心API属性
@property (readonly, strong, nonatomic) NSManagedObjectContext * managedObjectContext;
@property (readonly, strong, nonatomic) NSManagedObjectModel * managedObjectModel;
@property (readonly, strong, nonatomic) NSPersistentStoreCoordinator * persistentStoreCoordinator;

- (void)saveContext;
- (NSURL *)applicationDocumentsDirectory;

@end

```

接下来修改应用程序委托的实现文件:

```
#pragma -mark CoreData
@synthesize managedObjectContext       = _managedObjectContext;
@synthesize managedObjectModel         = _managedObjectModel;
@synthesize persistentStoreCoordinator = _persistentStoreCoordinator;

- (void)saveContext {
    NSError *error = nil;
    // 获取应用程序的托管对象上下文
    NSManagedObjectContext *managedObjectContext = self.managedObjectContext;
    if (managedObjectContext != nil) {
        // 如果托管对象的上下文包含了未保存的修改，则执行保存
        if ([managedObjectContext hasChanges] && ![managedObjectContext save:&error]) {
            NSLog(@"保存错误信息: %@, %@", error, [error userInfo]);
            abort();
        }
    }
}

// 初始化应用的托管对象上下文
- (NSManagedObjectContext *)managedObjectContext {
    if (_managedObjectContext != nil) {
        return _managedObjectContext;
    }

    // 获取持久化存储协调器
    NSPersistentStoreCoordinator *coordinator = [self persistentStoreCoordinator];
    if (coordinator != nil) {
        _managedObjectContext = [[NSManagedObjectContext alloc] init];
        // 为NSManagedObjectContext对象设置持久化存储协调器
        [_managedObjectContext setPersistentStoreCoordinator:coordinator];
    }

    return _managedObjectContext;
}

- (NSManagedObjectModel *)managedObjectModel {
    if (_managedObjectModel != nil) {
        return _managedObjectModel;
    }

    // 获取实体模型文件对应的NSURL
    NSURL *modelURL = [[NSBundle mainBundle] URLForResource:@"Model" withExtension:@"momd"];
    // 加载应用的实体模型文件，并初始化NSManagedObjectModel对象
    _managedObjectModel = [[NSManagedObjectModel alloc] initWithContentsOfURL:modelURL];

    return _managedObjectModel;
}

- (NSPersistentStoreCoordinator *)persistentStoreCoordinator {
    if (_persistentStoreCoordinator != nil) {
        return _persistentStoreCoordinator;
    }

    // 获取SQLite数据库文件的存储目录
    NSURL *storeURL = [[self applicationDocumentsDirectory] URLByAppendingPathComponent:@"Demo.sqlite"];
    NSError *error = nil;
    // 以持久化对象模型为基础，创建NSPersistentStoreCoordinator对象
    _persistentStoreCoordinator = [[NSPersistentStoreCoordinator alloc] initWithManagedObjectModel:[self managedObjectModel]];
    // 设置持久化存储协调器底层采用SQLite存储机制
    if (![_persistentStoreCoordinator addPersistentStoreWithType:NSSQLiteStoreType configuration:nil URL:storeURL options:nil error:&error]) {
        NSLog(@"设置持久化存储失败: %@ %@", error, [error userInfo]);
        abort();
    }

    return _persistentStoreCoordinator;
}

- (NSURL *)applicationDocumentsDirectory {
    return [[[NSFileManager defaultManager] URLsForDirectory:NSDocumentDirectory inDomains:NSUserDomainMask] lastObject];
}
```

## 设计实体模型
我们先在Model.xcdatamodeld中添加一个Fruit的实体，并设置name和color属性，类型为String。  
然后点击Editor -> Create NSManagedObject Subclass将实体生成为NSManagedObject的子类。  
此时，我们就为实体模型添加类一个简单的Fruit实体，该实体包含两个属性，但不包含任何关联关系。  

如果在后续编译中出现如下信息:

```
warning: Unable to load class named 'PRODUCT_MODULE_NAME.Fruit' for entity 'Fruit'.  Class not found, using default NSManagedObject instead.
```

请在Model.xcdatamodeld将Fruit实体的name和Class的名字都设置为Fruit。

## 使用CoreData对数据进行CRUD的操作
获取托管对象上下文就可以对对象进行增删改查的操作。

## 添加实体

```
// 获取应用的委托对象
AppDelegate *appDelegate = (AppDelegate *)[[UIApplication sharedApplication] delegate];
// 托管上下文中创建新实体
Fruit *apple = [NSEntityDescription insertNewObjectForEntityForName:@"Fruit" inManagedObjectContext:appDelegate.managedObjectContext];
// 为实体设置属性
apple.color = @"green";
apple.name  = @"apple";

NSError *error;
// 设置完实体属性后，调用托管对象上下文的save:方法将实体写入数据库
if ([appDelegate.managedObjectContext save:&error]) {
    NSLog(@"保存实体成功");
} else {
    NSLog(@"保存实体出错: %@, %@", error, [error userInfo]);
}

```

后续的修改和删除代码大同小异，请读者自行研究。

## 查询实体

```
// 获取应用的委托对象
AppDelegate *appDelegate = (AppDelegate *)[[UIApplication sharedApplication] delegate];
// 创建抓取数据的请求对象
NSFetchRequest *request = [[NSFetchRequest alloc] init];
// 设置抓取实体的类型
NSEntityDescription *entity = [NSEntityDescription entityForName:@"Fruit" inManagedObjectContext:appDelegate.managedObjectContext];
// 设置抓取实体
[request setEntity:entity];
// 定义抓取条件
request.predicate = [NSPredicate predicateWithFormat:@"name=%@", @"apple"];
NSError *error;
NSArray *array = [[appDelegate.managedObjectContext executeFetchRequest:request error:&error] mutableCopy];

```

# 使用基本对象类型定制的个性化缓存方案

自己实现一套存储方案。首先要明确，这个所谓的定制方案适用于互联网应用中对远程数据的缓存，几个限制条件缺一不可。

从需求出发分析缓存数据有哪些要求：按Key查找，快速读取，写入不影响正常操作，不浪费内存，支持归档。这些都是基本需求，那么再进一步或许还需要固定缓存项数量，支持队列缓存，缓存过期等。从这些需求入手设计一个缓存方案并不十分复杂。

---

本次iOS持久化数据知识点也就罗列这么多，当然如果深入，这些完全是不够的。但作为整体了解，快速上手还是可以的。

# 参考资料
[对比iOS中的四种数据存储](http://www.infoq.com/cn/articles/data-storage-in-ios)