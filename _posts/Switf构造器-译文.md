title: Switf 构造器-译文
date: 2016-03-13 18:40:05
tags:
- Swift
---

# 初始化

初始化是类、结构体或者枚举类型在被使用前的准备过程。在一个新的实例准备被使用前，这个过程为每一个存储属性设置初始值并且执行执行其它设置或者初始化工作。  

实现这个初始化过程被定义为构造器，就像一个被用来创造指定实例的特殊方法。和 Objective-C 构造器不同，Swift 的构造器不返回任何值。他们主要的确保新实例在使用前能够被正确地初始化。  

类的实例也可以实现析构器，在实例被回收前执行一些自定义的清理工作。更多信息请参考析构器一章。  
<!--more-->
## 为存储属性设置初始值

类和结构体被创建时，必须为所有的存储属性设置初始值。存储属性不能处于一个不确定的状态。  

可以在构造器内为存储属性设置初始值，或者在属性定义时赋默认值。接下来将描述这一系列行为。

> 注意
> 当你给存储属性一个默认值，或者在构造器内给它初始值，属性的值是直接被设置的，不会触发任何属性观察器。

## 构造器

构造器在创建特定类型的新实例时被调用。最简单的形式，一个构造器就像一个无参的实例方法，用 init 关键字描述：

```
init() {
    // 执行一些初始化操作
}
```

下面的例子中定义了一个 Fahrenheit 结构体来保存华氏温度。Fahrenheit 结构体有一个叫 temperature 的 Double 类型的存储属性：

```
struct Fahrenheit {
    var temperature: Double
    init() {
        temperature = 32.0
    }
}
var f = Fahrenheit()
print("The default temperature is \(f.temperature)° Fahrenheit")
// 输出 "The default temperature is 32.0° Fahrenheit”
```

这个结构体定义了一个简单的无参构造器，init。将存储属性 temperature 设置为 32。（华氏温度下水的结冰点）。

## 默认属性值

如上所示，你可以在构造器中为存储属性设定初始值。同样，在属性声明时也可以设置一个默认的值。  

> 注意
> 如果一个属性总是获取相同的初始值，就提供一个默认值而不是在构造器内初始化。这样最终结果是一样的，不过属性声明和默认值初始化将更加紧密。这使得构造器更加简单、清晰，并且可以通过默认值的类型推断。默认值也让你更简单地领会到默认构造器和构造器继承的好处。这将在后续章节介绍。

你可以通过默认值用更简单的形式编写 Fahrenheit 结构体的 temperature 属性：

```
struct Fahrenheit {
    var temperature = 32.0
}
```

## 自定义初始化

通过输入的参数和可选参数属性类型初始化，或者在初始化期间给常量属性赋值。

### 初始化参数

通过提供初始化参数作为一个构造器的定义的一部分，这个过程定义参数的类型和值。初始化参数的语法和用法与函数和方法的参数一致。  

接下来的例子定义了用来变现摄氏度的名为 Celsius 的结构体。Celsius 结构体实现两个自定义的构造器，分别为 init(fromFahremheit:) 和 init(fromKelvin:) ，这些构造器通过来自不同温度度量的值来创建一个结构体实例：

```
struct Celsius {
    var temperatureInCelsius: Double
    init(fromFahrenheit fahrenheit: Double) {
        temperatureInCelsius = (fahrenheit - 32.0) / 1.8
    }
    init(fromKelvin kelvin: Double) {
        temperatureInCelsius = kelvin - 273.15
    }
}
let boilingPointOfWater = Celsius(fromFahrenheit: 212.0)
// boilingPointOfWater.temperatureInCelsius 是 100.0
let freezingPointOfWater = Celsius(fromKelvin: 273.15)
// freezingPointOfWater.temperatureInCelsius 是 0.0”
```

第一个构造器包含一个外部变量名为 fromFahrenheit 和一个内部变量名 fahrenheit 的变量。第二个构造器包含一个外部变量名为 fromKelvin 和内部变量名为 kelvin 的变量。两个构造器将传递来的参数转换为对应的摄氏度值，并将值存储到 temperatureInCelsius 的属性中。  

### 内部变量名和外部变量名

构造器参数可以同时拥有两个参数名作为变量参数，一个内部变量名用于构造函数内部使用，一个外部变量名用于当构造函数被调用时使用。  

但是和方法函数不同，构造器没有方法名。因此构造器参数的名字和类型作为构造器唯一标识来调用时扮演特殊的角色。如果你没有提供一个外部变量名，Swift 会自动提供为每个参数提供外部变量名。  

接下来的例子定义了一个叫 Color 的结构体，有三个常量属性分别是 red，green 和 blue。属性存储的值在 0.0 至 1.0 之间，代表三原色。  

Color 提供了一个包含三个类型为 Double 的参数的构造函数，同时也提供了一个接受单个参数同时设置三个属性为相同值的构造函数。

```
struct Color {
    let red, green, blue: Double
    init(red: Double, green: Double, blue: Double) {
        self.red   = red
        self.green = green
        self.blue  = blue
    }
    init(white: Double) {
        red   = white
        green = white
        blue  = white
    }
}
```

通过提供给每个参数传值，两个构造器都可以创建一个新的 Color 实例。

```
let magenta = Color(red: 1.0, green: 0.0, blue: 1.0)
let halfGray = Color(white: 0.5)
```

需要注意的是，如果调用构造函数而没有使用外部变量名是不可能成功的。如果构造函数定义了外部变量名你就必须使用它，否则会在编译期报错：

```
let veryGreen = Color(0.0, 1.0, 0.0)
// 报编译时错误，需要外部名称
```

### 没有外部变量名的初始化参数

如果你不想让初始化参数使用外部变量名，使用下划线 _ 代替外部变量名来覆盖上述的默认创建外部变量名的行为。  

这里有一个之前 Celsius 例子的扩展版本，有一个额外的构造函数提供一个 Double 值创建一个新的 Celsius 实例。

```
struct Celsius {
    var temperatureInCelsius: Double
    init(fromFahrenheit fahrenheit: Double) {
        temperatureInCelsius = (fahrenheit - 32.0) / 1.8
    }
    init(fromKelvin kelvin: Double) {
        temperatureInCelsius = kelvin - 273.15
    }
    init(_ celsius: Double){
        temperatureInCelsius = celsius
    }
}
let bodyTemperature = Celsius(37.0)
// bodyTemperature.temperatureInCelsius 为 37.0
```

Celsius(37.0) 这个构造函数的调用意图非常清楚，不需要使用额外的外部变量名。因此这是种十分恰当的方式编写构造函数，init(_ celsius: Double)，这个构造函数可以通过提供一个 Double 值来调用。  


### 可选属性类型

如果自定义的类型有一个存储属性，逻辑上是允许没有值的，可能是因为该值在初始化期间不能被设定，或者因为被声明为可选类型从而允许空值。可选属性自动在初始化时被设置为 nil 值，代表目前这个属性在初始化期间是没有值。  

下面的例子定义了一个叫做 SurveyQuestion 的类，这个类有一个可选的 String 属性名为 response：

```
class SurveyQuestion {
    var text: String
    var response: String?
    init(text: String) {
        self.text = text
    }
    func ask() {
        print(text)
    }
}
let cheeseQuestion = SurveyQuestion(text: "Do you like cheese?")
cheeseQuestion.ask()
// 输出 "Do you like cheese?"
cheeseQuestion.response = "Yes, I do like cheese."
```

问题调查的结果在回复前是无法知道的，所以 response 属性被声明为 String? 类型，或者 optional String。可选属性被自动设为 nil，意味着当 SurveyQuestion 被实例化时值为空。  

### 初始化时期的常量赋值

你可以在初始化期间给一个常量属性赋值，只要在构造结束前确定一个值。一旦常量属性被赋值，将再也不会被修改。

注意  
对于类实例来说，一个常量属性可以在初始化过程被修改，不能在子类中被修改。  

上面 SurveyQuestion 例子中，你能设置 text 属性为常量属性而不是变量属性，这说明 SurveyQuestion 类一旦被实例化就不能修改问题。尽管 text 属性现在是常量，但是仍然可以在构造器中被设置。

```
class SurveyQuestion {
    let text: String
    var response: String?
    init(text: String) {
        self.text = text
    }
    func ask() {
        print(text)
    }
}
let beetsQuestion = SurveyQuestion(text: "How about beets?")
beetsQuestion.ask()
// 输出 "How about beets?"
beetsQuestion.response = "I also like beets. (But not with cheese.)"
```

## 默认构造器

如果结构体或者类没有一个构造函数，Swift 会提供一个为所有未初始化的属性设置默认值的构造函数。  

这个例子定义了一个叫做 ShoppingListItem 类，封装了 name，quantity 和 pruchase state 属性。  

```
class ShoppingListItem {
    var name: String?
    var quantity = 1
    var purchased = false
}
var item = ShoppingListItem()
```

因为所有的 ShoppingListItem 属性都有默认值，同时也没有父类，ShoppingListItem 自动得到一个默认的构造器实现为所有属性设值并创建新实例（name 属性是一个可选的 String 属性，所以会自动收到一个 nil 的默认值，即使这个值在代码中没有写到）。上面的例子使用默认构造器为 ShoppingLsitItem 类创建一个新的类实例。通过调用 ShoppingListItem() 并赋值给 item 变量。  

### 结构体类型的逐一成员构造器

如果没有自定义构造器，结构体类型自动收到一个逐一成员构造器。不同于默认构造器，即使构造器的存储属性没有默认值，结构体也会收到逐一成员构造器。逐一成员构造器是一个快捷方式来初始化新的结构体实例的成员属性。新实例属性的初始化值通过名字逐一传给逐一构造器。  

下面的例子定义了一个叫 Size 的结构体，拥有 width 和 height 两个属性。两个属性被推断为 Double 类型并设置默认值为 0.0。  

Size 结构体自动获得一个 init(width:height:) 逐一成员构造器，你可以用这个构造器来实例化 Size 结构体：

```
struct Size {
	var width = 0.0, height = 0.0
}

let twoByTwo = Size(width:2.0, height:2.0)
```