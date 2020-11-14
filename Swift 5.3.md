# Swift 5.3

## 基础

```swift
let a = "const variable" // 常量，支持推导
var b = "variable" // 变量
var a,b,c,d: String // 定义多个相同类型变量
let cat="kimi";print(cat) // 分号可用于一行多语句中
let minValue = UInt8.min // UInt的最小值 max为最大值，挺好
let anotherPi = 3 + 0.14159 // 被推断为Double

// 进制
let decimalInteger = 17 // 十进制
let binaryInteger = 0b10001 // 二进制
let octalInteger = 0o21 // 八进制
let hexadecimalInteger = 0x11 // 十六进制

// 十进制
// 1.25e2 意味着 1.25 x 102, 或者 125.0  .
// 1.25e-2  意味着 1.25 x 10-2, 或者 0.0125  .

// 十六进制
// 0xFp2  意味着 15 x 22, 或者 60.0 .
// 0xFp-2  意味着 15 x 2-2, 或者 3.75 .

// 易读性
let paddedDouble = 000123.456
let oneMillion = 1_000_000
let justOverOneMillion = 1_000_000.000_000_1

// 数值转换
let integerPi = Int(pi) // 默认为截断值为3 （ 3 + 0.14159 ）

// 布尔值
let a = true 
let b: Bool = false

// 元组
let http404Error = (404, "Not Found")
let (statusCode, statusMessage) = http404Error // 分解
let (justTheStatusCode, _) = http404Error // 可忽略

// 可索引
print("The status code is \(http404Error.0)") 
print("The status message is \(http404Error.1)")

// 元组元素命名
let http200Status = (statusCode: 200, description: "OK")
print("The status code is \(http200Status.statusCode)")
print("The status message is \(http200Status.description)")

// 可选值 (?)
var serverResponseCode: Int? = 404
serverResponseCode = nil // 可选值可赋值为 nil

// 强制展开（!）
if convertedNumber != nil {
    print("convertedNumber has an integer value of \(convertedNumber!).")
}
// 多个可选绑定
if let firstNumber = Int("4"), let secondNumber = Int("42"), firstNumber < secondNumber && secondNumber < 100 {
    print("\(firstNumber) < \(secondNumber) < 100")
}
```

## 错误处理

```swift
// 可抛错误的函数
func canThrowAnError() throws {
    // this function may or may not throw an error
}

// 使用 try 调用函数，do 语句确定一个容易范围
do {
    try canThrowAnError()
    // no error was thrown
} catch {
    // an error was thrown
}
```

## 断言 & 先决条件

```swift
let age = -3
// 如果条件判断为 true，代码运行会继续进行
assert(age >= 0, "A person's age cannot be less than zero")

// 如果条件的结果是 false 信息就会显示出来。
precondition(index > 0, "Index must be greater than zero.")
```

## 合并空值运算符

```swift
let c = a ?? b
// 如果可选 a 有值则展开，如果没有值(nil)，则返回默认值 b。a 必须为可选型，b（可以为可选型） 必须与 a 相同类型
// let c = a != nil ? a! : b
```

## 区间

```swift
// 闭区间
// 1,2,3,4,5
for index in 1...5 {}

// 半开区间
// 1，2，3，4
for index in 1..<5 {}

// 单侧区间
// c,d,e,f
let v = ["a","b","c","d","e","f"]
for idx in v[2...]{
    print(idx)
}
// [...2]
// [..<2]

// 单侧区间可以一端 无限无穷，所以在循环中需要讲去判断
let range = ...5
range.contains(7)   // false
range.contains(4)   // true
range.contains(-1)  // true
```

## 数组

```swift
var someInts = [Int]()      // 简写
var someInts = Array<Int>() // 常规写法
var threeDoubles = Array(repeating: 0.0, count: 3) // 填充默认值
var anotherThreeDoubles = Array(repeating: 2.5, count: 3)
var sixDoubles = threeDoubles + anotherThreeDoubles // 相同类型可相加来创建新数组
var shoppingList: [String] = ["Eggs", "Milk"] // 字面量创建
var shoppingList = ["Eggs", "Milk"] // 更短一些
shoppingList.append("Flour") // 常规的添加
shoppingList += ["Baking Powder"] // += 添加元素
shoppingList[4...6] = ["Bananas", "Apples"] // 3个元素 替换 2个元素 
```

### 数组遍例

```swift
// 只获取元素
for item in shoppingList {
    print(item)
}

// 获取下标与元素
for (index, value) in shoppingList.enumerated() {
    print("Item \(index + 1): \(value)")
}
```

## 集合

```swift
var letters = Set<Character>() // 没有简写
letters = [] // 但是可以用此写法清空

// 初始化
var favoriteGenres: Set<String> = ["Rock", "Classical", "Hip hop"]
var favoriteGenres: Set = ["Rock", "Classical", "Hip hop"] // 自动推导类型
```

## 字典

```swift
// 初始化
var a = Dictionary<String,String>() // 常规写法
var b = [String:String]() //推导写法

// 如果旧值在更新前存在的话，返回其值。否则返回nil (oldValue为String?)
if let oldValue = airports.updateValue("Dublin Airport", forKey: "DUB") {
    print("The old value for DUB was \(oldValue).")
}
// 同上
if let removedValue = airports.removeValue(forKey: "DUB") {
    print("The removed airport's name is \(removedValue).")
}
// 遍历字典
// 无需 enumerated 
for (airportCode, airportName) in airports {
    print("\(airportCode): \(airportName)")
}
// 初始化为两个数组
let airportCodes = [String](airports.keys)
// airportCodes is ["YYZ", "LHR"]
let airportNames = [String](airports.values)
// airportNames is ["Toronto Pearson", "London Heathrow"]


```

## 控制流

### for in

```swift
// for in 间隔
let minuteInterval = 5
let minutes = 60
for tickMark in stride(from: 0, to: minutes, by: minuteInterval) {
    print(tickMark)
  	// render the tick mark every 5 minutes (0, 5, 10, 15 ... 45, 50, 55)
}
```

### while & repeat-while

```swift
while condition {
  // statements
}

repeat{
  // statements
}while codition
```

### switch case

```swift
// 不用 break,
// 不能空执行，
// 可多个条件匹配
let anotherCharacter: Character = "a"
switch anotherCharacter {
case "a", "A":
    print("The letter A")
default:
    print("Not the letter A")
}

// 可区间匹配
// case 100..<1000:

// 元组匹配
let somePoint = (1, 1)
switch somePoint {
case (0, 0):
    print("(0, 0) is at the origin")
case (_, 0):
    print("(\(somePoint.0), 0) is on the x-axis")
case (0, _):
    print("(0, \(somePoint.1)) is on the y-axis")
case (-2...2, -2...2):
    print("(\(somePoint.0), \(somePoint.1)) is inside the box")
default:
    print("(\(somePoint.0), \(somePoint.1)) is outside of the box")
}

// let 值绑定
let anotherPoint = (2, 0)
switch anotherPoint {
case (let x, 0):
    print("on the x-axis with an x value of \(x)")
case (0, let y):
    print("on the y-axis with a y value of \(y)")
case let (x, y):
    print("somewhere else at (\(x), \(y))")
}

// where 额外条件
let yetAnotherPoint = (1, -1)
switch yetAnotherPoint {
case let (x, y) where x == y:
    print("(\(x), \(y)) is on the line x == y")
case let (x, y) where x == -y:
    print("(\(x), \(y)) is on the line x == -y")
case let (x, y):
    print("(\(x), \(y)) is just some arbitrary point")
}
```

### 控制转移

```swift
// continue，重头开始循环
// break,退出循环，退出switch域
// fallthrough,switch贯穿

testlabel : while true {
  switch 1 {
    case 1:
    	break testlabel // 跳出循环，而不是单单跳出 switch
    	continue testlabel // 跳到下一次循环
    default:
    	print("yeah")
  }
}
```

### 守护（提前退出）

```swift
// 当name赋值成功则继续执行，不然提前退出
guard let name = person["name"] else {
  return
}
```

## 函数

```swift
// from 入参（形式参数）标签
// hometown 实际参数标签
// return 可省略
func greet(person: String, from hometown: String) -> String {
    "Hello \(person)!  Glad you could visit from \(hometown)."
}
print(greet(person: "Bill", from: "Cupertino"))

// 省略实际参数标签
// 默认参数值
func someFunction(_ firstParameterName: Int, secondParameterName: Int = 2) {
    // In the function body, firstParameterName and secondParameterName
    // refer to the argument values for the first and second parameters.
}
someFunction(1, secondParameterName: 2)
```

## 闭包

```swift
// 闭包的一般形式
{ (parameters) -> (return type) in
    statements
}

// 如正常情况下
reversedNames = names.sorted(by: { (s1: String, s2: String) -> Bool in return s1 > s2 })

// 从语境中推断出闭包的类型，所以可以简化一些申明
reversedNames = names.sorted(by: { s1, s2 in return s1 > s2 } )

// 好吧，return 其实也是可以省略的，
reversedNames = names.sorted(by: { s1, s2 in s1 > s2 } )

// swift 会提供一种简写的闭包实际参数
reversedNames = names.sorted(by: { $0 > $1 } )

// > 本身就是一个函数
reversedNames = names.sorted(by: >)
```

## 尾随闭包

```swift
// 定义一个需要闭包的参数的函数  
func someFunctionThatTakesAClosure(closure:() -> Void){
      //function body goes here
}

// 常规的写法
someFunctionThatTakesAClosure({
      //closure's body goes here
})

// 尾随的写法
someFunctionThatTakesAClosure() {
       // trailing closure's body goes here
}

// 最后写成
object.someFunctionThatTakesAClosure {
  		// trailing closure's body goes here
}

// 多个闭包
func loadPicture(from server: Server, completion: (Picture) -> Void, onFailure: () -> Void) {
    if let picture = download("photo.jpg", from: server) {
        completion(picture)
    } else {
        onFailure()
    }
}
// 可省略第一个闭包的实参标签
loadPicture(from: someServer) { picture in
    someView.currentPicture = picture
} onFailure: {
    print("Couldn't download the next picture.")
}

```

### 逃逸闭包

> 如果 self 指向类的实例，那么在逃逸闭包中引用 self就需要额外注意。在逃逸闭包中捕获 self很容易不小心造成强引用循环

```swift
var completionHandlers: [() -> Void] = []
func someFunctionWithEscapingClosure(completionHandler: @escaping () -> Void) {
    completionHandlers.append(completionHandler)
}

func someFunctionWithNonescapingClosure(closure: () -> Void) {
    closure()
}
 
class SomeClass {
    var x = 10
    func doSomething() {
        // completionhandlers.append
        // {} 将来执行
        someFunctionWithEscapingClosure { self.x = 100 }
      	// 放入捕获列表 
        // someFunctionWithEscapingClosure { [self] in x = 100 }
      
      	// {} 立即执行
        someFunctionWithNonescapingClosure { x = 200 }
    }
}
 
let instance = SomeClass()
// Nonescaping set x = 200
instance.doSomething()
print(instance.x)
// Prints "200"

// Escaping self.x = 100
completionHandlers.first?()
print(instance.x)
// Prints "100"
```

### 自动闭包

```swift
var customersInLine = ["Chris", "Alex", "Ewa", "Barry", "Daniella"]
print(customersInLine.count)
// Prints "5"

// 此处只是赋值了一个闭包，并没有实际执行。
let customerProvider = { customersInLine.remove(at: 0) }
print(customersInLine.count)
// Prints "5"
 
// 此处执行了闭包
print("Now serving \(customerProvider())!")
// Prints "Now serving Chris!"
print(customersInLine.count)
// Prints "4"
```

```swift
// customersInLine is ["Ewa", "Barry", "Daniella"]
// 自动闭包 @autoclosure
func serve(customer customerProvider: @autoclosure () -> String) {
    print("Now serving \(customerProvider())!")
}
serve(customer: customersInLine.remove(at: 0))
// Prints "Now serving Ewa!"
```

## 枚数

```swift
enum CompassPoint {
    case north
    case south
    case east
    case west
}

var directionToHead = CompassPoint.west
directionToHead = .east  // 可略写

// 枚举可遍例
// CaseIterable 支持可遍例
enum Beverage: CaseIterable {
    case coffee, tea, juice
}
// allCases
let numberOfChoices = Beverage.allCases.count
print("\(numberOfChoices) beverages available")
```

### 关联值

```swift
enum Barcode {
    case upc(Int, Int, Int, Int)
    case qrCode(String)
} 
var productBarcode = Barcode.upc(8, 85909, 51226, 3)
productBarcode = .qrCode("ABCDEFGHIJKLMNOP")

// 常规写法
switch productBarcode {
case .upc(let numberSystem, let manufacturer, let product, let check):
    print("UPC: \(numberSystem), \(manufacturer), \(product), \(check).")
case .qrCode(let productCode):
    print("QR code: \(productCode).")
}

// 简略写法
switch productBarcode {
case let .upc(numberSystem, manufacturer, product, check):
    print("UPC : \(numberSystem), \(manufacturer), \(product), \(check).")
case let .qrCode(productCode):
    print("QR code: \(productCode).")
}
// Prints "QR code: ABCDEFGHIJKLMNOP."
```

### 原始值

```swift
// 原始值为 Character 类型
enum ASCIIControlCharacter: Character {
    case tab = "\t"
    case lineFeed = "\n"
    case carriageReturn = "\r"
}

// 隐式 (数值递增)
enum Planet: Int {
    case mercury = 1, venus, earth, mars, jupiter, saturn, uranus, neptune
}

// 隐式（字符串字面量）
enum CompassPoint: String {
    case north, south, east, west
}
let earthsOrder = Planet.earth.rawValue // rawValue 访问

// 原始值初始化
if let somePlanet = Planet(rawValue: 1) {
  // statements
  // somePlanet 为可选型
}else{
  // statements
}
```

### 递归枚举

```swift
// indirect 属性可递归
enum ArithmeticExpression {
    case number(Int)
    indirect case addition(ArithmeticExpression, ArithmeticExpression)
    indirect case multiplication(ArithmeticExpression, ArithmeticExpression)
}

// 整个类型可递规
indirect enum ArithmeticExpression {
    case number(Int)
    case addition(ArithmeticExpression, ArithmeticExpression)
    case multiplication(ArithmeticExpression, ArithmeticExpression)
}
```

## 结构与类

### 相同之处

- 定义属性用来存储值；
- 定义方法用于提供功能；
- 定义下标脚本用来允许使用下标语法访问值；
- 定义初始化器用于初始化状态；
- 可以被扩展来默认所没有的功能；
- 遵循协议来针对特定类型提供标准功能。

### 类的额外功能

- 继承允许一个类继承另一个类的特征;
- 类型转换允许你在运行检查和解释一个类实例的类型；
- 反初始化器允许一个类实例释放任何其所被分配的资源；
- 引用计数允许不止一个对类实例的引用。

> class 是引用类型，struct 是值类型，值类型赋值即拷贝。

### 属性与结构体

> 变量属性可修改，<u>常量属性只能在初始化中被分配，如果结构体被赋值给常量，则不能修改实例的属性，即使实例属性为变量属性</u>。

### lazy 修饰符

+ 必须配合 var 使用
+ 只会在初次使用时初始化 （例如：属性为一个类的实例）
+ 多线程初始化，无法保证属性只初始化一次。

### 计算属性

```swift
struct CompactRect {
    var origin = Point()
    var size = Size()
    var center: Point {
        get {
          	// 已简化的没有 return 的返回
            Point(x: origin.x + (size.width / 2),
                  y: origin.y + (size.height / 2))
        }
        set {
          	// 已简化了与 Point 类型相同的 newValue
            origin.x = newValue.x - (size.width / 2)
            origin.y = newValue.y - (size.height / 2)
        }
    }
}
```

```swift
struct Cuboid {
    var width = 0.0, height = 0.0, depth = 0.0
  	// 只读属性（是否可删除 return ）
    var volume: Double {
        return width * height * depth
    }
}
let fourByFiveByTwo = Cuboid(width: 4.0, height: 5.0, depth: 2.0)
print("the volume of fourByFiveByTwo is \(fourByFiveByTwo.volume)")
// prints "the volume of fourByFiveByTwo is 40.0"
```

### 属性观察者

```swift
class StepCounter {
    var totalSteps: Int = 0 {
        willSet(newTotalSteps) {
            print("About to set totalSteps to \(newTotalSteps)")
        }
        didSet {
            if totalSteps > oldValue  {
                print("Added \(totalSteps - oldValue) steps")
            }
        }
    }
}
```

### 属性包装

```swift
// 定义包装器
@propertyWrapper
struct SmallNumber {
    private var maximum: Int
    private var number: Int
    var wrappedValue: Int {  // 创建一个结构体、枚举或者定义了 wrappedValue 属性的类
        get { return number }
        set { number = min(newValue, maximum) }
    }
 
    init() {
        maximum = 12
        number = 0
    }
    init(wrappedValue: Int) {
        maximum = 12
        number = min(wrappedValue, maximum)
    }
    init(wrappedValue: Int, maximum: Int) {
        self.maximum = maximum
        number = min(wrappedValue, maximum)
    }
}

// 使用包装器 init（）方法
struct ZeroRectangle {
    @SmallNumber var height: Int
    @SmallNumber var width: Int
}

// 使用包装器 init(wrappedValue:) 方法
struct UnitRectangle {
    @SmallNumber var height: Int = 1
    @SmallNumber var width: Int = 1
}

// 显示调用包装器
struct NarrowRectangle {
    @SmallNumber(wrappedValue: 2, maximum: 5) var height: Int
    @SmallNumber(wrappedValue: 3, maximum: 4) var width: Int
}

// 较混合的调用方式
struct MixedRectangle {
    @SmallNumber var height: Int = 1
    @SmallNumber(maximum: 9) var width: Int = 2
}
```

### 包装器映射

```swift
@propertyWrapper
struct SmallNumber {
    private var number = 0
    var projectedValue = false // 通过暴击的 ${propertyName} 符访问; projectedValue 为固定关键字
    var wrappedValue: Int {    // 通过常规访问
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
  	// 似乎该属性一定要有初始化
    init(){
        projectedValue = false
    }
}
struct SomeStructure {
    @SmallNumber var someNumber: Int
}
var someStructure = SomeStructure()
 
someStructure.someNumber = 4
print(someStructure.$someNumber) // 映射值的名称和包装的值一样，除了它起始于一个美元符号（ $ ）
// Prints "false"
 
someStructure.someNumber = 55
print(someStructure.$someNumber)
// Prints "true"

///////////////////////////////////////////////// 更丰富的映射返回值 //////////////////////////////////////////
@propertyWrapper
struct SmallNumber {
    
    struct PV {
            var info = ""
            var value = ""
    }
    
    private var number = 0
    var projectedValue = PV(info:"",value:"") // 返回一个结构
    var wrappedValue: Int {
        get { return number }
        set {
            if newValue > 12 {
                number = 12
                projectedValue.info = "greater newValue"
            } else {
                number = newValue
                projectedValue.info = "less then newValue"
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
// Prints "PV(info: "less then newValue", value: "")\n"
 
someStructure.someNumber = 55
print(someStructure.$someNumber)
// Prints "PV(info: "greater newValue", value: "")\n"
```

### 类型属性（静态属性 static）不可被重写

+ static 前缀申明
+ ClassName 来调用，而不是实例

### 类型方法 （静态方法 class）可被重写

+ Class func 前缀申明
+ ClassName 来调用，而不是实例

### self 属性

+ 如果你没有显式地写出 self，Swift会在你于方法中使用已知属性或者方法的时候假定你是调用了当前实例中的属性或者方法。
+ 当一个实例方法的形式参数名与实例中某个属性拥有相同的名字的时候。在这种情况下，<u>形式参数名具有优先权</u>，并且调用属性的时候使用更加严谨的方式就很有必要了。
+ <u>静态方法中访问静态属性不需要写</u> `ClassName.somthing`

### 值类型 (struct / enum) 实例方法中修改

```swift
struct Point {
    var x = 0.0, y = 0.0
    mutating func moveBy(x deltaX: Double, y deltaY: Double) { // 增加 mutating 值类型的实例方法就可以修改值属性了。
        x += deltaX
        y += deltaY
    }
}
var somePoint = Point(x: 1.0, y: 1.0)
somePoint.moveBy(x: 2.0, y: 3.0)
print("The point is now at (\(somePoint.x), \(somePoint.y))")
// prints "The point is now at (3.0, 4.0)"
```

```swift
enum TriStateSwitch {
    case off, low, high
    mutating func next() {
        switch self {
        case .off:
            self = .low
        case .low:
            self = .high
        case .high:
            self = .off
        }
    }
}
var ovenLight = TriStateSwitch.low
ovenLight.next()
// ovenLight is now equal to .high
ovenLight.next()
// ovenLight is now equal to .off
```

## 下标语法

### 定义方式

```swift
subscript(index: Int) -> Int {
    get {
        // return an appropriate subscript value here
    }
    set(newValue) {
        // perform a suitable setting action here
    }
}
```

### 只读下标

```swift
struct TimesTable {
    let multiplier: Int
    // subscript 实现下标
    // 此处省略了 setter 与 getter 写法，为只读下标
    subscript(index: Int) -> Int {
        return multiplier * index
    }
}
let threeTimesTable = TimesTable(multiplier: 3)
print("six times three is \(threeTimesTable[6])")
// prints "six times three is 18"
```

### 自定义下标

```swift
struct Matrix {
    let rows: Int, columns: Int
    var grid: [Double]
    init(rows: Int, columns: Int) {
        self.rows = rows
        self.columns = columns
        grid = Array(repeating: 0.0, count: rows * columns)
    }
    func indexIsValid(row: Int, column: Int) -> Bool {
        return row >= 0 && row < rows && column >= 0 && column < columns
    }
  	// 接收多个参数的下标
    subscript(row: Int, column: Int) -> Double {
        get {
            assert(indexIsValid(row: row, column: column), "Index out of range")
            return grid[(row * columns) + column]
        }
        set {
            assert(indexIsValid(row: row, column: column), "Index out of range")
            grid[(row * columns) + column] = newValue
        }
    }
}
var matrix = Matrix(rows: 2, columns: 2)

// 多参数下标用例
matrix[0, 1] = 1.5
matrix[1, 0] = 3.2
```



### 类型（静态）下标

```swift
enum Planet: Int {
    case mercury = 1, venus, earth, mars, jupiter, saturn, uranus, neptune
  	// 静态
    static subscript(n: Int) -> Planet {
        return Planet(rawValue: n)!
    }
}
let mars = Planet[4]
print(mars)
```



## 继承

```swift
class SomeSubclass: SomeSuperclass {
    // subclass definition goes here
}
```

### 重写方法

> override 关键字会执行 Swift 编译器检查你重写的类的父类(或者父类的父类)是否有与之匹配的声明来供你重写。这个检查确保你重写的定义是正确的

```swift
class Train: Vehicle {
    override func makeNoise() {
        print("Choo Choo")
    }
}
```



- 一个命名为 someMethod() 的重写方法可以通过 `super.someMethod()` 在重写方法的实现中调用父类版本的 someMethod() 方法；
- 一个命名为 someProperty 的重写属性可以通过 `super.someProperty` 在重写的 getter 或 setter 实现中访问父类版本的 someProperty 属性；
- 一个命名为 someIndex 的重写下标脚本可以使用 `super[someIndex]` 在重写的下标脚本实现中访问父类版本中相同的下标脚本。

### 重写属性

> 如果你提供了一个setter作为属性重写的一部分，你也就必须为重写提供一个getter。如果你不想在重写getter时修改继承属性的值，那么你可以简单通过从 getter 返回 `super.someProperty` 来传递继承的值， someProperty 就是你重写的那个属性的名字。

```swift
class Car: Vehicle {
    var gear = 1
    override var description: String {
        return super.description + " in gear \(gear)"
    }
}
```



### 重写属性观察器

> 无法作用于继承而来的常量存储属性或者只读的计算属性

```swift
class AutomaticCar: Car {
    override var currentSpeed: Double {
        didSet {
            gear = Int(currentSpeed / 10.0) + 1
        }
    }
}
```

### 阻止重写

```swift
final var 
final func
final class func
final subscript
```

## 初始化

+ 当你给一个存储属性分配默认值，或者在一个初始化器里设置它的初始值的时候，属性的值就会被直接设置，<u>不会调用任何属性监听器。</u>
+ 在初始化的任意时刻，你都可以给常量属性赋值，只要它在初始化结束是设置了确定的值即可。一旦为常量属性被赋值，它就不能再被修改了。

#### 委托初始化

```swift
struct Rect {
    var origin = Point()
    var size = Size()
    init() {}
    init(origin: Point, size: Size) {
        self.origin = origin
        self.size = size
    }
    init(center: Point, size: Size) {
        let originX = center.x - (size.width / 2)
        let originY = center.y - (size.height / 2)
        // 此处称为委托初始化
        self.init(origin: Point(x: originX, y: originY), size: size)
    }
}
```

## [类]类型初始化

> 值类型不存在`指定`与`便捷`两种初始化器

#### 指定初始化器

> **指定初始化器**是类的主要初始化器。指定的初始化器可以初始化<u>**所有那个类引用的属性并且调用合适的父类初始化器来继续这个初始化过程给父类链**</u>。

#### 便捷初始化器

> 在为通用的初始化模式创建快捷方式以节省时间或者类的初始化更加清晰明了的时候使用便捷初始化器。<u>**(统一初始化方式，语义清晰)**</u>

- 指定初始化器必须总是向上委托。
- 便捷初始化器必须总是横向委托。
- 不像在 Objective-C 中的子类，Swift 的子类不会默认继承父类的初始化器。（<u>**只有在特定情况下才会继承父类的初始化器**</u>）

#### 区别指定与便捷

+ 规则1，指定构造器必须调用它直接父类的指定构造器
+ 规则2，便捷构造器只能调用同一个类中的其它构造器
+ 规则3，便捷构造器必须以调用一个指定构造器结束

#### 重写

> 子类 Bicycle 定义了一个自定义初始化器 init() 。这个指定初始化器和 Bicycle 的父类的指定初始化器相匹配，所以 Bicycle 中的指定初始化器需要带上 override 修饰符。
>
> Bicycle 类的 init() 初始化器以<u>**调用 super.init() 开始，这个方法作用是调用父类的初始化器。这样可以确保 Bicycle 在修改属性之前它所继承的属性 numberOfWheels 能被 Vehicle 类初始化。**</u>

```swift
class Vehicle {
    var numberOfWheels = 0
    var description: String {
        return "\(numberOfWheels) wheel(s)"
    }
}

class Bicycle: Vehicle {
    override init() {
        super.init()
        numberOfWheels = 2
    }
}
```

#### 自动继承初始化器

> RecipeIngredient 的父类是 Food ，它只有一个 init() 便捷初始化器。因此这个初始化器也被 RecipeIngredient 类继承。这个继承的 init() 函数和 Food 提供的是一样的，除了它是委托给 RecipeIngredient 版本的 init(name: String) 而不是 Food 版本。

```swift
class Food {
    var name: String
    init(name: String) {
        self.name = name
    }
    convenience init() {
        self.init(name: "[Unnamed]")
    }
}

class RecipeIngredient: Food {
    var quantity: Int
    init(name: String, quantity: Int) {
        self.quantity = quantity
      	// 提供了父类的指定初始化器的实现
      	// 所以父类的便捷初始化器被自动继承了
        super.init(name: name)
    }
  	
    override convenience init(name: String) {
        self.init(name: name, quantity: 1)
    }
}

// 被自动继承了的 Food.init()
let oneMysteryItem = RecipeIngredient()
print(oneMysteryItem.name)
let oneBacon = RecipeIngredient(name: "Bacon")
print(oneBacon.name)
let sixEggs = RecipeIngredient(name: "Eggs", quantity: 6)
print(sixEggs.name)
```

> 由于它为自己引入的所有属性提供了一个默认值并且自己没有定义任何初始化器， ShoppingListItem 会自动从父类继承*所有*的指定和便捷初始化器。

```swift
class ShoppingListItem: RecipeIngredient {
    var purchased = false
    var description: String {
        var output = "\(quantity) x \(name)"
        output += purchased ? " ✔" : " ✘"
        return output
    }
}
```

### 可失败初始化器

+ 你不能定义可失败和非可失败的初始化器为相同的形式参数类型和名称。
+ 你可以用一个非可失败初始化器重写一个可失败初始化器，但反过来是不行的。
+ **<u>init! 初始化器导致初始化失败时会触发断言。</u>**

```swift
struct Animal {
    let species: String
    init?(species: String) {
      	// species.isEmpty 等于 "" 是合法的。
        // 等于 nil 是另一个概念了
        if species.isEmpty { return nil }
        self.species = species
    }
}
```

#### 必须初始化器

> 在类的初始化器前添加 required 修饰符来表明所有该类的子类都必须实现该初始化器。
>
> <u>**在重写父类的必要初始化器时，不需要添加 override 修饰符**</u>

#### 通过闭包和函数来设置属性默认值

> 如果你使用了闭包来初始化属性，请记住闭包执行的时候，实例的其他部分还没有被初始化。这就意味着你不能在闭包里读取任何其他的属性值，即使这些属性有默认值。你也不能使用隐式 self 属性，或者调用实例的方法。

```swift
class SomeClass {
    let someProperty: SomeType = {
        // create a default value for someProperty inside this closure
        // someValue must be of the same type as SomeType
        return someValue
    }()
}
```

#### 访问可选类型的下标

```swift
var testScores = ["Dave": [86, 82, 84], "Bev": [79, 94, 81]]
testScores["Dave"]?[0] = 91
testScores["Bev"]?[0] += 1
testScores["Brian"]?[0] = 72 // 赋值失败
// the "Dave" array is now [91, 82, 84] and the "Bev" array is now [80, 94, 81]
```

## 错误处理

```swift
// 定义错误
enum VendingMachineError: Error {
    case invalidSelection
    case insufficientFunds(coinsNeeded: Int)
    case outOfStock
}

// 使用
throw VendingMachineError.insufficientFunds(coinsNeeded: 5)
```

#### 可抛出错误的函数定义

```swift
func canThrowErrors() throws -> String
```

#### do-catch

```swift
//do {
//    try expression
//    statements
//} catch pattern 1 {
//    statements
//} catch pattern 2 where condition {
//    statements
//} catch pattern 3, pattern 4 where condition {
//    statements
//} catch {
//    statements
//}

// example 1
var vendingMachine = VendingMachine()
vendingMachine.coinsDeposited = 8
do {
    try buyFavoriteSnack(person: "Alice", vendingMachine: vendingMachine)
    print("Success! Yum.")
} catch VendingMachineError.invalidSelection {
    print("Invalid Selection.")
} catch VendingMachineError.outOfStock {
    print("Out of Stock.")
} catch VendingMachineError.insufficientFunds(let coinsNeeded) { // 绑定值
    print("Insufficient funds. Please insert an additional \(coinsNeeded) coins.")
} catch {
    print("Unexpected error: \(error).")
}

// example 2
do {
    try vendingMachine.vend(itemNamed: item)
} catch is VendingMachineError { // 匹配整个类型，而不是具体的某个值
    print("Couldn't buy that from the vending machine.")
}

// example 3
func eat(item: String) throws {
    do {
        try vendingMachine.vend(itemNamed: item)
    } catch VendingMachineError.invalidSelection, VendingMachineError.insufficientFunds, VendingMachineError.outOfStock { // 多个条件
        print("Invalid selection, out of stock, or not enough money.")
    }
}
```

#### 转换错误为可选项

```swift
// 可错误，可Int
func someThrowingFunction() throws -> Int {
    // ...
}
 
// x 为可选型
let x = try? someThrowingFunction()

// y 例
let y: Int?
do {
    y = try someThrowingFunction()
} catch {
    y = nil
}
```

#### 取消错误传递

```swift
// image 跟随 app 所以，强制加载永不会出错。
// 但 LoadImage还有其它的应用场景，所以其本身还是返回可选型
let photo = try! loadImage("./Resources/John Appleseed.jpg")
```

#### defer

> 使用 defer语句来在代码离开当前代码块前执行语句合集。这个语句允许你在以<u>**任何**</u>方式离开当前代码块前执行必须要的清理工作——无论是因为抛出了错误还是因为 return或者 break这样的语句。

## 类型转换

```swift
// 父
class MediaItem {
    var name: String
    init(name: String) {
        self.name = name
    }
}
// 子
class Movie: MediaItem {
    var director: String
    init(name: String, director: String) {
        self.director = director
        super.init(name: name)
    }
} 
// 子
class Song: MediaItem {
    var artist: String
    init(name: String, artist: String) {
        self.artist = artist
        super.init(name: name)
    }
}

let library = [
    Movie(name: "Casablanca", director: "Michael Curtiz"),
    Song(name: "Blue Suede Shoes", artist: "Elvis Presley"),
    Movie(name: "Citizen Kane", director: "Orson Welles"),
    Song(name: "The One And Only", artist: "Chesney Hawkes"),
    Song(name: "Never Gonna Give You Up", artist: "Rick Astley")
]
// "library" 的类型被推断为[MediaItem]

```

### is （实例类型检查）

> 使用*类型检查操作符* （ is ）来检查一个实例是否属于一个特定的子类。如果**<u>实例</u>**是该子类类型，类型检查操作符返回 true ，否则返回 false 。

```swift
for item in library {
    if item is Movie {
        movieCount += 1
    } else if item is Song {
        songCount += 1
    }
}
```



### as  （向下类型转换）

> 某个类类型的常量或变量可能实际上在后台引用自一个子类的实例。当你遇到这种情况时你可以尝试使用*类型转换操作符*（ as? 或 as! ）将它*向下类型转换*至其子类类型。

```swift
for item in library {
    if let movie = item as? Movie {
        print("Movie: '\(movie.name)', dir. \(movie.director)")
    } else if let song = item as? Song {
        print("Song: '\(song.name)', by \(song.artist)")
    }
}
```

### Swift 为不确定的类型提供了两种特殊的类型别名：
+ AnyObject 可以表示任何类类型的实例。
+ Any 可以表示任何类型，包括函数类型。

### 类型嵌套

```swift
import Cocoa

struct BlackjackCard {
 
    // nested Suit enumeration
    enum Suit: Character {
        case spades = "♠", hearts = "♡", diamonds = "♢", clubs = "♣"
    }
 
    // nested Rank enumeration
    enum Rank: Int {
        case two = 2, three, four, five, six, seven, eight, nine, ten
        case jack, queen, king, ace
      	// 定义一个 Values 结构
        struct Values {
            let first: Int, second: Int?
        }
      	// 定义一个计算值 values 
        var values: Values {
            switch self {
            case .ace:
                return Values(first: 1, second: 11)
            case .jack, .queen, .king:
                return Values(first: 10, second: nil)
            default:
                return Values(first: self.rawValue, second: nil)
            }
        }
    }
 
    // BlackjackCard properties and methods
    let rank: Rank, suit: Suit
    var description: String {
        var output = "suit is \(suit.rawValue),"
        output += " value is \(rank.values.first)"
        if let second = rank.values.second {
            output += " or \(second)"
        }
        return output
    }
}

let theAceOfSpades = BlackjackCard(rank: .ace, suit: .spades)
print("theAceOfSpades: \(theAceOfSpades.description)")

```

## 扩展

```swift
// 使已有类型遵循一个或多个协议
extension SomeType: SomeProtocol, AnotherProtocol {
    // implementation of protocol requirements goes here
}
```

#### 扩展 Double

```swift
extension Double {
    var km: Double { return self * 1_000.0 } // 注意 self
    var m: Double { return self }
    var cm: Double { return self / 100.0 }
    var mm: Double { return self / 1_000.0 }
    var ft: Double { return self / 3.28084 }
}
let oneInch = 25.4.mm
print("One inch is \(oneInch) meters")
// Prints "One inch is 0.0254 meters"
let threeFeet = 3.ft
print("Three feet is \(threeFeet) meters")
// Prints "Three feet is 0.914399970739201 meters"
```

#### 扩展后更明确的语义

```swift
let aMarathon = 42.km + 195.m
print("A marathon is \(aMarathon) meters long")
```

#### 扩展初始化

> 如果你使用扩展为一个值类型添加初始化器，且该值类型为其所有储存的属性提供默认值，而又不定义任何自定义初始化器时，你可以在你扩展的初始化器中调用该类型默认的初始化器和成员初始化器。

```swift
extension Rect {
    init(center: Point, size: Size) {
        let originX = center.x - (size.width / 2)
        let originY = center.y - (size.height / 2)
      	// 使用默认初始化器 （ Rect 并未定义任务初始化器）
        self.init(origin: Point(x: originX, y: originY), size: size)
    }
}
```

#### 扩展方法

#### 扩展可修改方法

#### 扩展下标方法

#### 扩展内嵌类型

#### 扩展内嵌类型

## 协议

> 协议为方法、属性、以及其他特定的任务需求或功能定义蓝图。协议可被类、结构体、或枚举类型*采纳*以提供所需功能的具体实现。满足了协议中需求的任意类型都叫做*遵循*了该协议。

```swift
protocol SomeProtocol {
    var mustBeSettable: Int { get set }
    var doesNotNeedToBeSettable: Int { get }
}
```

#### 父类一起定义

```swift
class SomeClass: SomeSuperclass, FirstProtocol, AnotherProtocol {
    // class definition goes here
}
```

#### 重点：初始化要求

```swift
// 协议一个初始化要求
protocol SomeProtocol {
    init(someParameter: Int)
}
// 必须要 required
class SomeClass: SomeProtocol {
    required init(someParameter: Int) {
        // initializer implementation goes here
    }
}
```

#### 有条件遵循协议

```swift
extension Array: TextRepresentable where Element: TextRepresentable {
    var textualDescription: String {
        let itemsAsText = self.map { $0.textualDescription }
        return "[" + itemsAsText.joined(separator: ", ") + "]"
    }
}
```

#### 扩展声明（采纳协议）

```swift
struct Hamster {
    var name: String
    // 已经实现了协议方法
    var textualDescription: String {
        return "A hamster named \(name)"
    }
}
// 直接采纳协议即可
extension Hamster: TextRepresentable {}
```

#### 协议可以是集合类型

```swift
let things: [TextRepresentable] = [game, d12, simonTheHamster]
```

#### 协议可继承

```swift
protocol InheritingProtocol: SomeProtocol, AnotherProtocol {
    // protocol definition goes here
}
```

#### 类专用的协议

```swift
// AnyObject 限制了只能被类型采纳
protocol SomeClassOnlyProtocol: AnyObject, SomeInheritedProtocol {
    // class-only protocol definition goes here
}
```

#### 协议组合

> 协议组合使用 `SomeProtocol` & `AnotherProtocol` 的形式。你可以列举任意数量的协议，用和符号连接（ & ），使用逗号分隔。除了协议列表，协议组合也能包含类类型，这允许你标明一个需要的父类。（**<u>用于形式参数组合有优势</u>**）

#### 协议测试 is / as 可用

> 用于检查协议类型

#### 可选协议要求

```swift
@objc protocol CounterDataSource {
    // 定义可选函数
    @objc optional func increment(forCount count: Int) -> Int
    @objc optional var fixedIncrement: Int { get }
}
// 调用
class Counter {
    var count = 0
    var dataSource: CounterDataSource?
    func increment() {
      	// increment? 来调用可选函数
        if let amount = dataSource?.increment?(forCount: count) {
            count += amount
        } else if let amount = dataSource?.fixedIncrement {
            count += amount
        }
    }
}
```

#### 给协议的扩展

```swift
extension Collection where Iterator.Element: TextRepresentable {
    var textualDescription: String {
        let itemsAsText = self.map { $0.textualDescription }
        return "[" + itemsAsText.joined(separator: ", ") + "]"
    }
}
```

## 泛型

```swift
func swapTwoValues<T>(_ a: inout T, _ b: inout T) {
    let temporaryA = a
    a = b
    b = temporaryA
}
```

#### 扩展一个泛型类型

> 当你扩展一个泛型类型时，**<u>不需要在扩展的定义中提供类型形式参数列表。原始类型定义的类型形式参数列表在扩展体里仍然有效</u>**，并且原始类型形式参数列表名称也用于扩展类型形式参数。

```swift
extension Stack {
    var topItem: Element? {
        return items.isEmpty ? nil : items[items.count - 1]
    }
}
```

#### 类型约束语法

```swift
// T: 约束为一个类型
// U: 约束为一个协议
func someFunction<T: SomeClass, U: SomeProtocol>(someT: T, someU: U) {
    // function body goes here
}

// T 支持 Equatable 协议
func findIndex<T: Equatable>(of valueToFind: T, in array:[T]) -> Int? {
    for (index, value) in array.enumerated() {
        if value == valueToFind {
            return index
        }
    }
    return nil
}
```

### 关联类型

```swift
// 定义
protocol Container {
  	// 关联类型
    associatedtype ItemType
    mutating func append(_ item: ItemType)
    var count: Int { get }
    subscript(i: Int) -> ItemType { get }
}
// 实现
struct IntStack: Container {
    // original IntStack implementation
    var items = [Int]()
    mutating func push(_ item: Int) {
        items.append(item)
    }
    mutating func pop() -> Int {
        return items.removeLast()
    }
    // conformance to the Container protocol
    typealias ItemType = Int // 实现关联类型
    mutating func append(_ item: Int) {
        self.push(item)
    }
    var count: Int {
        return items.count
    }
    subscript(i: Int) -> Int {
        return items[i]
    }
}

```

### 关联类型约束

```swift
protocol SuffixableContainer: Container {
  	// 关联类型为：SuffixableContainer
    // SuffixableContainer 继承至 Container
    // Suffix 因为继承有 Item
    // Suffix.Item 必须和 Container.Item 一致 
    // 因为有可能 Suffix.Item == int & Container.Item == String 的情况
    associatedtype Suffix: SuffixableContainer where Suffix.Item == Item
    func suffix(_ size: Int) -> Suffix
}
```

#### 泛型 where 分句

> 被检查的两个容器不一定是相同类型的（尽管它们可以是），但是它们的元素类型必须相同。这个要求通过类型约束和泛型 Where 分句一起体现

```swift
func allItemsMatch<C1: Container, C2: Container>
    (_ someContainer: C1, _ anotherContainer: C2) -> Bool
    where C1.ItemType == C2.ItemType, C1.ItemType: Equatable {
        
        // Check that both containers contain the same number of items.
        if someContainer.count != anotherContainer.count {
            return false
        }
        
        // Check each pair of items to see if they are equivalent.
        for i in 0..<someContainer.count {
            if someContainer[i] != anotherContainer[i] {
                return false
            }
        }
        
        // All items match, so return true.
        return true
}
```

- C1 必须遵循 Container 协议（写作 C1: Container ）；
- C2 也必须遵循 Container 协议（写作 C2: Container ）；
- C1 的 ItemType 必须和 C2 的 ItemType 相同（写作 C1.ItemType == C2.ItemType ）；
- C1 的 ItemType 必须遵循 Equatable 协议（写作 C1.ItemType: Equatable ）。

进一步

- someContainer 是一个 C1 类型的容器；
- anotherContainer 是一个 C2 类型的容器；
- someContainer 和 anotherContainer 中的元素类型相同；
- someContainer 中的元素可以通过不等操作符（ != ）检查它们是否不一样。

#### 混合扩展来约束

```swift
// 元素需要支持 Equatable
extension Container where Item: Equatable {
    func startsWith(_ item: Item) -> Bool {
        return count >= 1 && self[0] == item
    }
}

// 这个栗子在 Item 是 Double 时给容器添加了 average()  方法。
extension Container where Item == Double {
    func average() -> Double {
        var sum = 0.0
        for index in 0..<count {
            sum += self[index]
        }
        return sum / Double(count)
    }
}
print([1260.0, 1200.0, 98.6, 37.0].average())
```

```swift
extension Container {
  	// where 分句写明了容器中新方法需要满足什么要求才能可用。
    func average() -> Double where Item == Int {
        var sum = 0.0
        for index in 0..<count {
            sum += Double(self[index])
        }
        return sum / Double(count)
    }
  	// where 分句写明了容器中新方法需要满足什么要求才能可用。
    func endsWith(_ item: Item) -> Bool where Item: Equatable {
        return count >= 1 && self[count-1] == item
    }
}
let numbers = [1260, 1200, 98, 37]
print(numbers.average())
// Prints "648.75"
print(numbers.endsWith(37))
// Prints "true"
```

#### 关联类型的泛型Where分句

```swift
protocol Container {
    associatedtype Item
    mutating func append(_ item: Item)
    var count: Int { get }
    subscript(i: Int) -> Item { get }
    
  	// Iterator 中的泛型 where 分句要求遍历器以相同的类型遍历容器内的所有元素，无论遍历器是什么类型。
    associatedtype Iterator: IteratorProtocol where Iterator.Element == Item
    func makeIterator() -> Iterator
}
```

#### 泛型下标

- 在尖括号中的泛型形式参数 Indices 必须是遵循标准库中 Sequence 协议的某类型；
- 下标接收单个形式参数， indices ，它是一个 Indices 类型的实例；
- 泛型 where 分句要求序列的遍历器必须遍历 Int 类型的元素。这就保证了序列中的索引都是作为容器索引的相同类型。

合在一起，这些限定意味着传入的 indices 形式参数是一个整数的序列。

```swift
extension Container {
    subscript<Indices: Sequence>(indices: Indices) -> [Item]
        where Indices.Iterator.Element == Int {
            var result = [Item]()
            for index in indices {
                result.append(self[index])
            }
            return result
    }
}
```

### 返回一个不透明类型

```swift
// some Shape
// 结果就是，函数返回一个遵循 Shape 协议的类型，而不需要标明具体类型。
func flip<T: Shape>(_ shape: T) -> some Shape {
    return FlippedShape(shape: shape)
}
```

#### 不透明类型和协议类型的区别

> 通常来讲，协议类型能提供更多存储值的弹性，不透明类型则能给你更多关于具体类型的保证。

```swift
protocol Container {
    associatedtype Item
    var count: Int { get }
    subscript(i: Int) -> Item { get }
}
extension Array: Container { }

// 你不能使用 Container 作为函数的返回类型，因为这个协议有一个关联类型。你也不能使用它作为范型返回类型的约束因为它在函数体外没有足够的信息来推断它到底需要成为什么范型类型。

// Error: Protocol with associated types can't be used as a return type.
func makeProtocolContainer<T>(item: T) -> Container {
    return [item]
}
 
// Error: Not enough information to infer C.
func makeProtocolContainer<T, C: Container>(item: T) -> C {
    return [item]
}

// 使用不透明类型 some Container 作为返回类型则能够表达期望的 API 约束——函数返回一个容器，但不指定特定的容器类型：

func makeOpaqueContainer<T>(item: T) -> some Container {
    return [item]
}
let opaqueContainer = makeOpaqueContainer(item: 12)
let twelve = opaqueContainer[0]
print(type(of: twelve))
// Prints "Int"
```

## 引用计数

+ 弱引用 `weak`

    ARC 会在被引用的实例被释放是自动地设置弱引用为 nil 。由于弱引用需要允许它们的值为 nil ，<u>它们一定得是可选类型。</u>

    <u>在 ARC 给弱引用设置 nil 时不会调用属性观察者。</u>

+ 无主引用 `unowned`

    和弱引用类似，无主引用不会牢牢保持住引用的实例。但是不像弱引用，总之，<u>无主引用假定是永远有值的。因此，无主引用总是被定义为非可选类型。</u>你可以在声明属性或者变量时，在前面加上关键字  `unowned`  表示这是一个无主引用。

+ 可选无主引用

    <u>使用无主可选引用时，你需要负责保证它总引用到一个合法对象或 nil 。</u>

### 闭包捕获列表

> Swift 要求你在闭包中引用self成员时使用 self.someProperty 或者 self.someMethod （而不只是 someProperty 或 someMethod ）。这有助于提醒你可能会一不小心就捕获了 self 。

```swift
// 一个延迟计算属性，其值是一个闭包
// -- 在闭包内部体中 -- 使用了捕获列表
lazy var someClosure: (Int, String) -> String = {
    [unowned self, weak delegate = self.delegate!] (index: Int, stringToProcess: String) -> String in
    // closure body goes here
}

// 无参数闭包
lazy var someClosure: () -> String = {
    [unowned self, weak delegate = self.delegate!] in
    // closure body goes here
}
```

## 内存安全

```swift
var stepSize = 1                          // 全局对象，可以直接在函数中访问
func increment(_ number: inout Int) {     // inout 写访问
    number += stepSize                    // stepSize 读访问， number 写访问 ， 读写重叠
}
 
increment(&stepSize)
// Error: conflicting accesses to stepSize
```

## 访问级别

Swift 为代码的实体提供个五个不同的访问级别。这些访问级别和定义实体的源文件相关，并且也和源文件所属的模块相关。

+ **Open** 访问 和 **public** 访问 允许实体被定义模块中的任意源文件访问，同样可以被另一模块的源文件通过导入该定义模块来访问。在指定框架的公共接口时，通常使用 **open** 或 **public** 访问。 **open** 和 **public** 访问 之间的区别将在之后给出；
+ **Internal** 访问 允许实体被定义模块中的任意源文件访问，但不能被该模块之外的任何源文件访问。通常在定义应用程序或是框架的内部结构时使用。
+ **File-private**  访问 将实体的使用限制于当前定义源文件中。当一些细节在整个文件中使用时，使用 file-private 访问隐藏特定功能的实现细节。
+ **private** 访问 将实体的使用限制于封闭声明中。当一些细节仅在单独的声明中使用时，使用 **private** 访问隐藏特定功能的实现细节。
+  **open** 访问是最高的（限制最少）访问级别，**private** 是最低的（限制最多）访问级别。

open 访问仅适用于类和类成员，它与 public 访问区别如下：

+ public 访问，或任何更严格的访问级别的类，只能在其定义模块中被继承。
+ public 访问，或任何更严格访问级别的类成员，只能被其定义模块的子类重写。
+ open 类可以在其定义模块中被继承，也可在任何导入定义模块的其他模块中被继承。
+ open 类成员可以被其定义模块的子类重写，也可以被导入其定义模块的任何模块重写。
+ 显式地标记类为 open 意味着你考虑过其他模块使用该类作为父类对代码的影响，并且相应地设计了类的代码。

#### 默认访问级别

> internal

## 高级运算符

####  取反

> *位取反运算符*（ ~ ）是对所有位的数字进行取反操作：

```swift
let initialBits: UInt8 = 0b00001111
let invertedBits = ~initialBits  // equals 11110000
```

#### 位与

> *位与运算符*（ & ）可以对两个数的比特位进行合并。它会返回一个新的数，只有当这两个数*都*是 1 的时候才能返回

```swift
let firstSixBits: UInt8 = 0b11111100
let lastSixBits: UInt8 = 0b00111111
let middleFourBits = firstSixBits & lastSixBits // equals 00111100
```

#### 位或

> *位或运算符*（ | ）可以对两个比特位进行比较，然后返回一个新的数，只要两个操作位任意一个为 1 时，那么对应的位数就为 1

```swift
let someBits: UInt8 = 0b10110010
let moreBits: UInt8 = 0b01011110
let combinedbits = someBits | moreBits // equals 11111110
```

#### 位异或

> 位异或运算符，或者说“互斥或”（ ^ ）可以对两个数的比特位进行比较。它返回一个新的数，当两个操作数的对应位不相同时，该数的对应位就为 1

```swift
let firstBits: UInt8 = 0b00010100
let otherBits: UInt8 = 0b00000101
let outputBits = firstBits ^ otherBits // equals 00010001
```

#### 位移

```swift
let shiftBits: UInt8 = 4 // 00000100 in binary
shiftBits << 1           // 00001000
shiftBits << 2           // 00010000
shiftBits << 5           // 10000000
shiftBits << 6           // 00000000
shiftBits >> 2           // 00000001
```

