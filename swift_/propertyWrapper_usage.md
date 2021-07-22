## 属性包装器

<details>
<summary>summary</summary>

```
就是给包裹的存储属性，添加 getter&setter， getter&setter 具体实现在 propertyWrapper 定义的结构中

so propertyWrapper:
1. 核心点在 wrapperValue 中实现 getter & setter
2. 实现各种业务相关的 init 方法
    * init() 默认值初始化
    * init(wrapperValue:) 表达式初始化
    * init(wrapperValue:, num0:, num1:) 其他参数初始化，（用来定义属性时，使用方式有点意思
3. 结构中一定要有 wrapperValue
4. 如果有投影必要添加 projectedValue
```

[TOC]

</details>

属性包装器在管理属性存储方式的代码和定义属性的代码之间添加了一层分离——添加 getter&setter。

要定义 Property Wrappers ，**需要创建一个包含`wrappedValue` 属性的** structure, enumeration, or class 。
在下面的代码中，`TwelveOrLess`结构确保它包装的值始终包含小于或等于 12

```swift
@propertyWrapper
struct TwelveOrLess {
  private var number = 0
  var wrappedValue: Int {
    get { return number }
    set { number = min(newValue, 12) }
  }
}
```

setter 确保新值小于 12，getter 返回存储的值。

注意：

> 上例中的 for 声明`number`将变量标记为`private`，这确保`number`仅在 TwelveOrLess 的实现中使用。在其他任何地方编写的代码使用 getter 和 setter for 访问该值`wrappedValue`，并且不能直接使用`number`

可以通过在属性之前写入包装器的名称作为属性来将包装器应用于属性。这是一个存储矩形的结构，该矩形使用`TwelveOrLess`属性包装器来确保其尺寸始终为 12 或更少：

```swift
struct SmallRectangle {
  @TwelveOrLess var height: Int
  @TwelveOrLess var width: Int
}

var rectangle = SmallRectangle()
print(rectangle.height)
// Prints "0"

rectangle.height = 10
print(rectangle.height)
// Prints "10"

rectangle.height = 24
print(rectangle.height)
// Prints "12"
```

在`height`和`width`来自定义性能得到它们的初始值`TwelveOrLess`，它设置`TwelveOrLess.number`为 0（`TwelveOrLess` 结构中默认值）。setter in`TwelveOrLess`将 10 视为有效值，因此将数字 10 存储`rectangle.height`为写入的值。但是，24 大于`TwelveOrLess`允许值，因此尝试存储 24 最终设置`rectangle.height`为 12，即最大允许值。

**当将包装器应用于属性时，编译器会合成为包装器提供存储的代码和通过包装器提供对属性的访问的代码。**（属性包装器负责存储包装的值）可以编写使用属性包装器行为的代码，而无需利用特殊的属性语法。例如，下面是上一个`SmallRectangle`代码清单中的一个版本，它`TwelveOrLess`显式地将其属性包装在结构中，而不是编写`@TwelveOrLess`为属性：

```swift
struct SmallRectangle {
  private var _height = TwelveOrLess()
  private var _width = TwelveOrLess()
  var height: Int {
    get { return _height.wrappedValue }
    set { _height.wrappedValue = newValue }
  }
  var width: Int {
    get { return _width.wrappedValue }
    set { _width.wrappedValue = newValue }
  }
}
```

在`_height`和`_width`属性存储属性包装的一个实例，`TwelveOrLess`。用于`height`和`width`包装对`wrappedValue`属性的访问的 getter 和 setter 。

### 设置包装属性的初始值

上面示例中的代码通过在的定义中提供 `number` 初始值来设置包装属性的初始值。使用此属性包装器的代码不能为被包装的属性指定不同的初始值——例如，`SmallRectangle`不能给出`height`或`width`初始值的定义。

为了支持设置初始值或其他自定义，属性包装器需要添加一个初始化函数。这是`TwelveOrLess`调用的扩展版本，`SmallNumber`它定义了设置包装和最大值的初始值设定项：

```swift
@propertyWrapper
struct SmallNumber {
  private var maximum: Int
  private var number: Int
  
  var wrappedValue: Int {
    get { return number }
    set { number = min(newValue, maximum) }
  }
  
  init() {// 提供默认初始值
    maximum = 12
    number = 0
  }
  init(wrappedValue: Int) {//
    maximum = 12
    number = min(wrappedValue, maximum)
  }
  init(wrappedValue: Int, maximum: Int) {
    self.maximum = maximum
    number = min(wrappedValue, maximum)
  }
}
```

`SmallNumber`的定义中包括三个initializers - `init()`，`init(wrappedValue:)`和`init(wrappedValue:maximum:)` 下面使用实例来设置 wrappedValue 和 maximum。

1. 当将包装器应用于属性并且您没有指定初始值时，Swift 使用`init()`初始化器来设置包装器。例如：

```swift
struct ZeroRectangle {
  @SmallNumber var height: Int
  @SmallNumber var width: Int
}

var zeroRectangle = ZeroRectangle()
print(zeroRectangle.height, zeroRectangle.width)
// Prints "0 0"
```

`SmallNumber`包裹的`height 和 width`通过调用`SmallNumber()`。使得 0 和 12 设置初始包装值和初始最大值。

1. 当你为属性指定一个初始值时，Swift 使用`**init(wrappedValue:)**`初始化器来设置包装器。例如：

```swift
struct UnitRectangle {
    @SmallNumber var height: Int = 1
    @SmallNumber var width: Int = 1
}

var unitRectangle = UnitRectangle()
print(unitRectangle.height, unitRectangle.width)
// Prints "1 1"
```

当写下  “= 1” 的时候，包裹器会调用 `init(wrappedValue:)` 调用，根据其内部实现 wrappedValue = 1，maximum 在`init(wrappedValue:)` 内部是 12

1. 当在自定义属性后的括号中写入参数时，Swift 使用接受这些参数的初始值设定项来设置包装器。例如，如果您提供初始值和最大值，Swift 将使用`**init(wrappedValue:maximum:)**`初始化程序：

```swift
struct NarrowRectangle {
    @SmallNumber(wrappedValue: 2, maximum: 5) var height: Int
    @SmallNumber(wrappedValue: 3, maximum: 4) var width: Int
}

var narrowRectangle = NarrowRectangle()
print(narrowRectangle.height, narrowRectangle.width)
// Prints "2 3"

narrowRectangle.height = 100
narrowRectangle.width = 100
print(narrowRectangle.height, narrowRectangle.width)
// Prints "5 4"
```

`SmallNumber`的实例包装`height`是通过调用`SmallNumber(wrappedValue: 2, maximum: 5)` 创建，以及`width` 包装的实例是通过调用`SmallNumber(wrappedValue: 3, maximum: 4)`创建。

通过在属性包装器中包含参数，您可以在包装器中设置初始状态或在包装器创建时将其他选项传递给包装器。此语法是使用属性包装器的最通用方法。您可以为属性提供所需的任何参数，并将它们传递给初始化程序。

当您包含属性包装器参数时，您还可以使用赋值指定初始值。Swift 将赋值视为`wrappedValue`参数并使用接受您包含的参数的初始化程序。例如：

```swift
struct MixedRectangle {
  @SmallNumber var height: Int = 1
  @SmallNumber(maximum: 9) var width: Int = 2
}

var mixedRectangle = MixedRectangle()
print(mixedRectangle.height)
// Prints "1"

mixedRectangle.height = 20
print(mixedRectangle.height)
// Prints "12"
```

`SmallNumber`包装的实例`height`是通过调用`SmallNumber(wrappedValue: 1)` 创建的，它使用默认最大值 12。包装的实例`width`是通过调用`SmallNumber(wrappedValue: 2, maximum: 9)`创建的。

### 从属性包装器投射值

除了包装的值之外，属性包装器还可以通过定义*投影值*来公开其他功能——例如，管理对数据库的访问的属性包装器可以暴露一个`flushDatabaseConnection()方法在`投影值上。投影值的名称与包装值一样，只是它以美元符号 ( `$`)开头。因为您的代码无法定义以`$`投影值开头的属性，所以不会干扰您定义的属性。

在`SmallNumber`上面的示例中，如果您尝试将属性设置为一个太大的数字，则属性包装器会在存储之前调整该数字。下面的代码`projectedValue`向`SmallNumber`结构中添加了一个属性，以在存储新值之前跟踪属性包装器是否调整了该属性的新值。

```swift
@propertyWrapper
struct SmallNumber {
  private var number = 0
  var projectedValue = false
  var wrappedValue: Int {
    get { return number }
    set {
      if newValue > 12 {
        number = 12
        projectedValue = true
      } else {
        number = newValue
        projectedValue = false
      }
    }
  }
}

struct SomeStructure {
  @SmallNumber var someNumber: Int
}
var someStructure = SomeStructure()

someStructure.someNumber = 4
print(someStructure.$someNumber)
// Prints "false"

someStructure.someNumber = 55
print(someStructure.$someNumber)
// Prints "true"
```

写`someStructure.$someNumber`访问包装器的预计值。在存储一个比较小的数——4 后，`someStructure.$someNumber`是`false`。存一个比较大的数——55，projected value 是false

属性包装器可以返回任何类型的值作为其投影值。在这个例子中，属性包装器只公开一条信息——数字是否被调整——所以它公开了布尔值作为它的投影值。需要公开更多信息的包装器可以返回某个其他数据类型的实例，或者它可以返回`self`以将包装器的实例公开为其预计值。

当您从属于类型的代码（如属性 getter 或实例方法）访问投影值时，您可以`self.`在属性名称之前省略，就像访问其他属性一样。在以下示例中的代码是指围绕包装件的投影值`height`与`width`作为`$height`和`$width`：

```swift
enum Size {
  case small, large
}

struct SizedRectangle {
  @SmallNumber var height: Int
  @SmallNumber var width: Int

  mutating func resize(to size: Size) -> Bool {
    switch size {
    case .small:
      height = 10
      width = 20
    case .large:
      height = 100
      width = 100
    }
    return $height || $width
  }
}
```

因为属性包装器语法只是具有 getter 和 setter 的属性的语法糖，所以访问`height`和`width`行为与访问任何其他属性相同。例如，`resize(to:)`访问`height`和`width`使用它们的属性包装器中的代码。如果调用，则 switch case for将矩形的高度和宽度设置为 100。包装器防止这些属性的值大于 12，并将投影值设置为 true，以记录它调整了它们的值的事实。在 resize(to:) 结束时，return 语句检查 $height 和 $width 以确定属性包装器是否调整了高度或宽度。

## 全局和局部变量

上述用于计算和观察属性的功能也可用于*全局变量*和*局部变量*。全局变量是在任何函数、方法、闭包或类型上下文之外定义的变量。局部变量是在函数、方法或闭包上下文中定义的变量。

你在前几章遇到的全局变量和局部变量，都已经是*存储变量了*。存储变量，如存储属性，为特定类型的值提供存储，并允许设置和检索该值。

但是，您也可以在全局或局部范围内定义*计算变量*并为存储变量定义观察者。计算变量计算它们的值，而不是存储它，它们的编写方式与计算属性相同。

笔记

全局常量和变量总是以类似于[延迟存储属性的](https://docs.swift.org/swift-book/LanguageGuide/Properties.html#ID257)方式[延迟](https://docs.swift.org/swift-book/LanguageGuide/Properties.html#ID257)计算。与惰性存储属性不同，全局常量和变量不需要用`lazy`修饰符标记。

局部常量和变量永远不会被延迟计算。

您可以将属性包装器应用于本地存储变量，但不能应用于全局变量或计算变量。例如，在下面的代码，`myNumber`使用`SmallNumber`作为属性包装。

```swift
func someFunction() {
  @SmallNumber var myNumber: Int = 0

  myNumber = 10
  // now myNumber is 10

  myNumber = 24
  // now myNumber is 12
}
```

就像应用`SmallNumber`到属性时一样，将 的值设置`myNumber`为 10 是有效的。因为属性包装器不允许大于 12 的值，所以它设置`myNumber`为 12 而不是 24。

> https://docs.swift.org/swift-book/LanguageGuide/Properties.html