title: 从UIView开始
date: 2015-09-26 09:34:00
tags: 
- iOS
- Swift
---

你在 iOS 设备上看到的美丽的界面要感谢 UIView 这个类。创建和配置 UIView 也非常简单。  
如果是代码，只要实例化后确定其 frame，然后添加到父视图中，运行 app 就可以看到预期的效果。  
如果是 xib 或者 storyboard，只需直接拖动控件到指定的 view 上，更加方便简单。  
UIView 更强大的之处在于，你可以控制何时显示何时隐藏，移动到指定位置，改变自身的大小形状，并且可以模拟碰撞，重力加速，弹性等物理效果。  
UIView 是继承自 UIResponder，这也是区别于 CALayer 的要点之一：UIView 具备交互功能，它能够响应用户的点击、滑动等操作。  
<!-- more -->
View 的层级关系（view hierarchy）也是重要的知识点，一个 View 可以有多个子视图，但一个 View 只有一个父视图。因此可以把这个层级关系想象成数据结构中的树。  
当一个父视图把移除，那么它下面的所有子视图也伴随着一起被移除。同理包括对父视图隐藏、移动等操作，同样会影响到其子视图。  
与此同时，层级关系也是响应链（responder chain）的基础。  

# UIWindow
window 处于这颗层级树的最顶部，是 UIWindow 的实例，继承自 UIView。在 iOS 中，你的 app 在运行时，window 将被创建，而且永远不会被销毁和替代。  
一个良好的编程习惯就是，至始至终只维护这一个 UIWindow 的实例，如果你想在界面上显示更多内容，请实例化 UIView 而不是 UIWindow。  

UIWindow 必须填满整个屏幕，所以它的大小和屏幕的尺寸是一致的。  
如果使用了 Main Storyboard，UIApplicationMain 方法在 app 启动时会自动帮你完成此工作。  
但是如果你是使用代码的形式，并且持续存活在 app 的生命周期里，那么就需要在 App Delegate 中 application:didFinishLaunchingWithOptions: 的方法中添加如下代码：

```Swift
self.window = UIWindow(frame: UIScreen.mainScreen().bounds)
```

你不能想当然地将随后的一些视图直接添加到这个 window 上，你需要做的是将一个 View Controller 实例传个这个 window 的 rootViewController 属性。  
如果你使用了 Main StroyBoard 那么上面的工作也不需要你关心。这个 View Controller 的实例就是 Main Stroyboard 中箭头所指的 initial view controller。  
一旦确定了 rootViewController 的值，那么这个 view controller 的 view 将被作为这个 window 唯一的子视图。随后所有的其它视图都将被作为这个 view 的子视图。  

但是此时此刻，你的 app 仍然不能看到任何界面。我们需要将这个 window 作为 app 的 key window。并且让它可见。

```Swift
self.window?.makeKeyAndVisible()
```
## Note
现在来看看从启动 app 到看到的视图所发生的事情。Main Storyboard 自动将这些工作完成，所以我们只从代码方面总结。
当 app 启动时，UIApplicationMain 函数会实例化 App Delegate，然后访问 App Delegate 的 window 属性。如果该值为 nil，UIApplicationMain 会实例化一个 UIWindow 并将该值赋给 AppDelegate 的 window 属性；如果该值不为 nil，UIApplicationMain 会保留这个 window，并将其值作为 app 的主 window。

# UIView

## 层级关系
我们先看看 UIView 层级关系的一些特点：

- 如果一个 View 在父视图中被移除或者被移动，子视图也会受到其影响而被移除或者移动。
- 一个 View 的透明度取决于其子视图
- 你所看到的子视图可以超出父视图的边界，如果你不想显示超出的部分，可以设置 clipsToBounds 属性。
- 父视图是强拥有子视图，可以想象数组管理内存的形式。
- 一个视图的 size 被改变，其子视图会自动被重新设置 size。

### 一些常用的方法：  

isDescendantOfView: 可以检查一个 View 是否是另一个 View 的后代。（即任何深度的子视图）  
addSubview: 最常用的添加子视图的方法。  
removeFromSuperview 将当前视图从父视图中移除。  

### 一些改变层级结构的常用方法：

- insertSubview:atIndex:
- insertSubview:belowSubview:, insertSubview:aboveSubview:
- exchangeSubviewAtIndex:withSubviewAtIndex:
- bringSubviewToFront:, sendSubviewToBack:

View 的 subviews 数组是 immutable copy 的。这意味着数组的值是无法改变的，也就是说我们无法正常地移除一个 View 的所有子视图。  
此时我们需要遍历数组，移除子视图：

```Swift
for tempView in myView.subviews as [UIView] {
	tempView.removeFromSuperview()
}

```

在 Swift 中，利用其函数式编程的优势，更简洁的方法如下

```Swift
(myView.subviews as [UIView]).map{$0.removeFromSuperview()}
```

### 为了响应一些事件，可以重载如下方法：

- didAddSubview:, willRemoveSubview:
- didMoveToSuperview, willMoveToSuperview:
- didMoveToWindow, willMoveToWindow:

## 常用的属性
设置 View 是否可见只需通过设置 hidden 属性。当我们设置 hidden 属性为 YES 时，只是隐藏视图，并没有真正地移除视图。  
一个被隐藏的视图无法接受触摸事件，所以给用户一个幻觉就是这个视图真的不存在一样。  

可以通过 backgroundColor 属性为视图设置背景颜色，当 backgroundColor 的属性为 nil 时，此时的背景颜色为透明（transparent）。
你也可以通过设置 alpha 属性来设置部分或者完全透明。alpha 值的范围在0~1.0之间，0代表透明，1代表不透明，区间内的值影响透明的程度。  
一个视图的子视图的 alpha 属性的值不会大于父视图的 alpha，即使你故意这么设置，该值也只会无限接近父视图的 alpha 值。  
更复杂点，颜色也是有 alpha 的值，即使一个视图的 alpha 为1.0，但是其颜色的 alpha 为0，该视图也是透明的。  
一个视图完全透明的效果类似于其 hidden 值被设置为 YES，这个视图极其子视图将不可见也无法被触控。  

视图的 opaque 属性，从某个方面讲，可以视为一种颜色。该值并不影响一个视图的外观。设置 opaque 属性一般从性能优化方面考虑。  
该属性被设置为 YES 同时 alpha 的值为1.0，那么渲染时，因为不再考虑透明效果而更加有效率。  

## Bounds 和 Center

我们先来看下面这一段代码：

```
let v = UIView(frame: CGRectMake(0, 0, 400, 400))
v.backgroundColor = UIColor.whiteColor()

let v1 = UIView(frame: v.bounds.insetBy(dx: 10, dy: 10))
v1.backgroundColor = UIColor.redColor()

v1.bounds.size.width += 20
v1.bounds.size.height += 20

v.addSubview(v1)
```

我们可以看到 v 是一个边长为 400 的正方形，v1 作为其子视图，通过 insertBy 方法确定其位置和大小。insertBy 方法是基于 v1 的位置，内边距为 10 形成的边长为 380，center 和 v1 一致的正方形。之后，我们将其边长增长 20，发现刚好完全覆盖整个父视图。  

改变 view 的 bounds 是不会改变 center，改变 center 也不会改变 bounds。这两个属性的值是独立的，分别决定其在父视图的位置和大小。而 frame 这个属性是通过 bounds 和 center 结合产生的。frame 的改变会影响到 bounds 和 center，同样 bounds 和 center 改变也会影响 frame。所以在某些情况下，使用 frame 是毫无意义的。

此外，我们可以转换 point 和 rect 到另一个坐标系：

- convertPoint:fromView:, convertPoint:toView:
- convertRect:fromView:, convertRect:toView:

**注意，当 rect 的值不为整型，可能会造成显示错误，如果这个视图有文本，这段文本将会变得模糊。解决的办法就是整型化，Objective-C 调用方法 CGRectIntegral，Swift 调用 makeIntegralInPlace。**

iOS 8之前，window 和 screen 的坐标系是不变的。在 iOS 8 之后，UIScreen 的 bounds 随着设备的旋转和之前有所不同。  
在iOS 8 上 UIScreen 提供了两个新的属性：

```
@property (readonly) id <UICoordinateSpace> coordinateSpace NS_AVAILABLE_IOS(8_0);
@property (readonly) id <UICoordinateSpace> fixedCoordinateSpace NS_AVAILABLE_IOS(8_0);
```

我们有如下代码：

```
CGPoint point = [myView convertPoint:CGPointMake(0, 0) toCoordinateSpace:[UIScreen mainScreen].fixedCoordinateSpace];
NSLog(@"fixedCoordinateSpace %@", NSStringFromCGPoint(point)); // {50, 50}

point = [myView convertPoint:CGPointMake(0, 0) toCoordinateSpace:[UIScreen mainScreen].coordinateSpace];
NSLog(@"coordinateSpace %@", NSStringFromCGPoint(point)); // {50, 50}
```

此时我们将设备顺时针旋转 90°，

```
NSLog(@"fixedCoordinateSpace %@", NSStringFromCGPoint(point)); // {50, 518}

NSLog(@"coordinateSpace %@", NSStringFromCGPoint(point)); // {50, 50}
```

下面这张图，可以解释这种不同点：

![](http://ww3.sinaimg.cn/large/aa0fbcc4gw1f02fs7yailj20ky0b9t9f.jpg)

（未完待续）


















