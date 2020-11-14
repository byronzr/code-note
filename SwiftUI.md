# SwiftUI

### View 基类

> 实际 UI

### PreviewProvider 基类

> 预览界面 ( Resume: Option + Command + Return )
>
> 刷新界面 ( Option + Command + P )

### Button

> Button.init(action: label: ) 实际上 `action` 与 `label` 都是闭包
>
> 在 `代码`中按住`Command` 选择 Button在弹出菜单中选择 `Extract Subview` 可将 Button 提取为一个新的 View

```swift
BUtton(action:{
  // closure 1
}){
  // closure 2
}
```

### ForEach

> ForEach(_: id:)
>
> 列举元素，接受一个数组，且数组中的元素需要满足 `Identifiable` 协议

```swift
// Student 因为不支持 `Identifiable` 协议，所以要扩展 `Hashalbe` 协议，
// 并在 ForEach 中配合 \.self 使用
struct Student: Hashable {
    let name: String
}

struct ContentView: View {
    let students = [Student(name: "Harry Potter"), Student(name: "Hermione Granger")]
    var body: some View {
        List(students, id: \.self) { student in
            Text(student.name)
        }
    }
}
```



## 基本布局

### HStack 水平堆叠 (Horizontal Stack)

> VStack( alignment: spacing: content:)

```swift
HStock{
  // elements
}
.frame(maxHeight: expanded ? .infinity : 0) // 根据 expanded 设置 frame 高度 PS： .infinity 剩余高度
// 增加一个手势（Tap）拖尾闭包实现 expanded 的 true/false 转换
//.animation(.default) // 隐式动画当前 View 及其 子View 属性发生变化时都会激活，如：初始化（低效，耗资源）
.onTapGesture {
   // 显示动画，只在手势触发时激活
   withAnimation {
      self.expanded.toggle()
   }
}
```

### VStack 垂直堆叠 ( Vertical Stack )

```swift
VStack(spacing:8) { // 每个元素间距为 8 
  // elements 
}
```



### ZStack 层

```swift
ZStack{
  // elements
}
```



### @State 关联数据

> @State 属性值，在SwiftUI内部会自动转换为一对setter和getter,这对属性，进行赋值操作将会触发 View 的刷新。
>
> @State 属性值，<u>无法在多个View之间被共享，只能在当前View.body中被操作。属性之间彼此有关联情况过于复杂也不适合。</u>

```swift
...
@State private var brain: CalculatorBrain = .left("0")

```



### @Binding

> @Binding 修饰属性，将值语义的属性“转换”为引用语义。对 @Binding 属性进行赋值，改变的将不是属性本身，而是它的引用

```swift
...
@Binding var brain:CalculatorBrain
```

> 当将@Binding的属性值传入时，实参（投参）需要为加`$` 符，投递的是`映射属性`

```swift
@State private var brain: CalculatorBrain = .left("0")
CalculatorButtonRow(row:row,brain:self.$brain)
```



### @Published

> 替代 @State 解决，多View之间共享与复杂关系的问题，注意： <u>class 必须扩展 ObservableObject 协议</u>
>
> @Published必须被存储和持有，所以只能声明在 class 中。

```swift
class CalculatorModel: ObservableObject {
  @Published var brain: CalculatorBrain = .left("0")
}
```



### @EnvironmentObject

> 对于“跳跃式”多个View层级的状态间的共享，有很大的便捷性

```swift
// 通过 SceneDelegate.swift 中赋值，所有 @EnvironmentObject 申明的变量将可用
// 与 全局单例 类似，仅类似而已
window.rootViewController = UIHostingController(
  rootView:ContentView().environmentObject(CalculatorModel()) 
)
```

### Image ( 读取资源 )

```swift
Image("Pokemon-\(model.id)")
  .resizable() // 默认 Image 与 frame 无关，resizable 变成可修改
  .frame(width: 50, height: 50, alignment: /*@START_MENU_TOKEN@*/.center/*@END_MENU_TOKEN@*/) // 修改宽高
  .aspectRatio(contentMode: .fit) // 保持原比例
```

### Image （ 读取 SF Symbol ）

```swift
// 同样可以用资源的方式控制大小
// 但对于 SF Symbol 有更简单的大小控制方式
Image(systemName: "star")
  .font(.system(size: 25)) // 仅控制图片大小
  .foregroundColor(.white)
  .frame(width: 30, height: 30, alignment:.center) // frame 控制可点区域
```

### Image ( modifier )

> 适用全局统一式样，跨 View,减少维护与重复

```swift
// 使用
Image(systemName: "chart.bar")
  .modifier(ToolButtonModifier())

// modifier define
struct ToolButtonModifier: ViewModifier {
    func body(content: Content) -> some View {
        content
            .font(.system(size: 25))
            .foregroundColor(.white)
            .frame(width: 30, height: 30, alignment: /*@START_MENU_TOKEN@*/.center/*@END_MENU_TOKEN@*/)

    }
}
```

### RoundedRectangle

```swift
ZStack{
    // 描边
    RoundedRectangle(cornerRadius: 20)
    .stroke(model.color, style: StrokeStyle(lineWidth:4))
    // 背景圆角
    RoundedRectangle(cornerRadius: 20)
    // 填充渐变
    .fill(
        // 渐变
        LinearGradient(
            gradient: Gradient(colors: [.white,model.color]), // 渐变色数组
            startPoint: .leading,
            endPoint: .trailing
        )
    )
}
```

### List 

> 默认带分隔线，动画会抖动，但默认有Cell重用机制

```swift
// 要求 pokemon 元素遵守 Identifiable 协议
List(PokemonViewModel.all) { pokemon in
    PokemonInfoRow(model: pokemon, expanded: false)
}

// Identifiable 协议
// Hashable 支持 Int / String / UUID 等
protocal Identifiable {
  associatedtype ID: Hashable // 该协议最重要，也是唯一的要求就是 关联值 ID 必须满足 Hashable 协议
  var id: Self.ID {get} // id 为 ID 关联值约束，所以 id 必须满足 Hashable 协议
}
```

### ScrollView

> ScrollView 默认没有 Cell 重用机制

```swift
struct PokemonList: View {
    var body: some View {
        ScrollView{
          	// pokemon 支持 Identifiable 协议，所以不需要 id: \.self了
            ForEach(PokemonViewModel.all) { pokemon in
                PokemonInfoRow(model: pokemon, expanded: false)
            }
        }
    }
}
```

> 一个常用的唯一展开案例

```swift
// list
struct PokemonList: View {
    @State var expandingIndex: Int?
    var body: some View {
        ScrollView{
            ForEach(PokemonViewModel.all) { pokemon in
                PokemonInfoRow(model: pokemon, expanded: self.expandingIndex == pokemon.id) // expanded 维护当前展开的 id ，对应 Row.expanded 要从 @State 改为 let
                    .onTapGesture{
												// ...
                        ) {
                            if self.expandingIndex == pokemon.id {
                                self.expandingIndex = nil
                            }else{
                                self.expandingIndex = pokemon.id
                            }
                        }
                    }
            }
        }
    }
}

// row 
struct PokemonInfoRow: View {
  	// @State var expanded: Bool // 原 row 独立控制状态
    let expanded: Bool // 被 ScrollView 传递
    // ...
}

```

### Section / Picker / TextField / SecureField / Toggle

```swift
// ObservableObject 与 ObservedObject
class Settings: ObservableObject {
    // 注册与登录切换
    enum AccountBehavior:CaseIterable{
        case register,login
    }
    
    // 排序方式
    enum Sorting: CaseIterable {
        case id,name,color,favorite
    }
    
    @Published var accountBehavior = AccountBehavior.login
    @Published var email = ""
    @Published var password = ""
    @Published var verifyPassword = ""
    
    @Published var showEnglishName = true
    @Published var sorting = Sorting.id
    @Published var showFavoriteOnly = false
}

extension Settings.Sorting {
    var text:String {
        switch self {
        case .id: return "ID"
        case .name: return "Name"
        case .color: return "Color"
        case .favorite: return "Favorite"
        }
    }
}

extension Settings.AccountBehavior {
    var text:String {
        switch self {
        case .register: return "Register"
        case .login: return "Login"
        }
    }
}

struct SettingView: View {
    @ObservedObject var settings = Settings()
    var body: some View {
        Form{
            accountSection
            optionSection
        }
    }
    
    var accountSection: some View {
        Section(header:Text("Account")){
            // account behavior
            Picker(
                selection: $settings.accountBehavior,
                label: Text("")
            ){
                ForEach(Settings.AccountBehavior.allCases,id:\.self) {
                    Text($0.text)
                }
            }
            // 改变 pickstyle 的式样
            .pickerStyle(SegmentedPickerStyle())
            
            // email
            TextField("Email",text:$settings.email)

            // password
            SecureField("Password",text:$settings.password)

            // confirm
            // 动态增加字段
            if settings.accountBehavior == .register {
                SecureField("confirm",text:$settings.verifyPassword)
            }

            // button
            Button(settings.accountBehavior.text){
                print("login/register")
            }
        }
    }
    
    var optionSection: some View {
        Section(header:Text("options")){
            Toggle("English Name", isOn: $settings.showEnglishName)
            Picker(
                selection:$settings.sorting,
                label:Text("")
            ){
                ForEach(Settings.Sorting.allCases,id:\.self){
                    Text($0.text)
                }
            }
            Toggle("Favorite Only", isOn: $settings.showFavoriteOnly)
            
            Button("Clear"){
                print("clear")
            }
        }
    }
}
```

### NavigationVIew

```swift
struct PokemonRootView: View {
    var body: some View {
        NavigationView{
            PokemonList().navigationBarTitle("Pokemon List")
        }
    }
}
```

----

# View

1. Store（状态机） 会在 SceneDelegate 是创建，类似全局变量
2. 可读写/交互变量用 Binding<T>申明 $store的映射值。
3. 只读属性可直接返回 store 的引用
4. 按钮事件调用 store.dispatch(.login( _: _:)) 
5. 主线程结束操作，等待 store.state 变更后 update UI

```sequence
SceneDelegate->View: MainTab().environmentObject(Store())
View-->dispatch: @EnvironmentObject var store:Store
View-->dispatch: var settingsBinding:Binding<AppState.Settings>{ $store.appState.settings }
View-->dispatch: var settings: AppState.Settings { store.appState.settings }
View->dispatch: .login(email: self.settings.email, password: self.settings.password)
```



# Store & AppState

1. Store（状态机） 在 SceneDelegate 中被创建，全局共用
2. dispatch 方法调用 reduce 方法对已定义 action 行为进行简单过滤
3. Reduce 方法返回 `未来的`  state 与可用的 command(action/publisher)
4. 如果 command 有效，则执行。当处理完成后更新`未来的 ` state

```sequence
AppState->Store: define struct
Store->dispatch:(_ action: AppAction)
dispatch->reduce:(state: AppState,action: AppAction)
reduce->LoginAppCommand: (email:email,password: password)
reduce->dispatch:(AppState,AppCommand?)


```

# AppCommand / Subscription

1. AppCommand 协议要求实现的一个接受 Store（状态机）的 execute 方法
2. `execute` 被调用时对 `publisher` 的结果进行逻辑判断
3. 按结果二次调用 store 状态机，对 state 数据进行更新
4. token 是为了在完成数据下载前，持有 publisher,防止 AnyCancellable 无持有者自行释放
5. 完成一次读取则释放

```sequence
AppCommand-->LoginAppCommand: protocol
LoginAppCommand->execute: (in store: Store)
execute-->token: SubscriptionToken
execute->LoginRequest.publisher: (email:email,password: password)
LoginRequest.publisher-->store: dispatch(.accountBehaviorDone(result: .failure(error)))
LoginRequest.publisher-->store: dispatch(.accountBehaviorDone(result: .success(user)))
store->State: update
store-->token: success: seal(in: token)
store-->token: failure: token.unseal()
```

# LoginRequest & Error 

1. 构建一个支持 Publisher
2. 结构属性在初始化时，以形参传入。
3. 理论上要在此处对结果进行`预判`以发送 `.success` / `.failure` 
4. 设计中，接收到数据后，切换至主线程（因为该接收的值会用于更新UI）

```sequence
AppError-->LoginRequest: define
LoginRequest-->Future: AnyPublisher<User,AppError>
Future--> promise in: proccess code
promise in->receive: promise(.success(user))
promise in->receive: promise(.failure(.passwordWrong))
receive->eraseToAnyPublisher: erase
receive-->LoginAppCommand: promise
```



# Check Email on start

```sequence
Store->init(): app start
init()->setupObservers(): setup
setupObservers()->checker.isEmailValid:
checker.isEmailValid->sink: subscription
sink->dispatch: .emailValid
checker.isEmailValid->store:(in: &disposeBag)
```

![Combine](/Users/byronzr/Desktop/Work Documents/Combine.png)



# 常规设计模式

### AppState

+ AppState.swift
+ 每个 TabView 定义一个 inline struct
+ AppState 中初始化每一个 struct

### Store

+ Store.swift
+ @Published var appState = AppState()
+ func dispatch(_ action: AppAction )
+ static func reduce(store: Store,action: AppAction) -> AppState

### View

+ @ObservedObject
+ @EnvironmentObject
+ 可缩写一个计算属性

```swift
@EnvironmentObject var store: Store
var settingsBinding: Binding<AppState.Settings>{
  $store.appState.settings
}
```

### SceneDelegate.swift

+ 初始化传 Store

```swift
let contentView = ContentView().environmentObject(Store())
```

### AppAction.swift

+ 存放所有可用 Action

```swift
enum AppAction {
    case notifyAddUser(name:String)
    case notifyUpdateUser(name:String)
    case notifyRefreshList
}
```

### AppCommand.swift

+ 执行“副作用”的集合

### AppError.swift

+ 错误定义集合

```swift
enum AppError: Error,Identifiable {
    var id: Int64 {errorCode}
    case ConnectError
}

extension AppError {
    var errorCode:Int64 {
        switch self {
        case .ConnectError:
            return 40001
        default:
            return 0
        }
    }
}
```



## Combine

+ 大部分定义在 Publishers 这个 enum 之中
+ Output 定义某个 Publisher 所发布值的类型
+ Failure 定义错误，使用 Subscribers.Completion 来描述，它含有两个 enum 成员，.failure(Failure) 和 .finished.
+ `scan` 让我们提供一个暂存值，每次事件发生时我们有机会执行一个闭包来更新这个暂存值，并准备好在下一次事件时使用它。<u>同时这个暂存值也将被作为新的 Publisher 事件被发送出去。</u>
+ `map` 和 `scan` 类似但可以修改值
+ `sink` 用于设置值。
+ `assign`  用于将事件值绑定到对象实例的属性上，<u>只能绑定在 class 实例上，而 ObservableObject 接口，只能由 class 修饰，正好是 assign 最常用的地方。</u>
+ `Subject` 主动发布一个事件 主要有 PassthroughSubject 和 CurrentValueSubject
+ `PassthroughSubject` 简单的转发 publisher ，并不会保留，如果在转发期，没有订阅，那么将会丢弃 publisher,所以主要是 `send` 方法
+ CurrentValueSubject 会包装并保留值，在订阅触发时发送出去。 所以主要是对 `value`属性的设置
+ Scheduler 协议中的 receive(on:options:) 可让下游在指定线程中接收事件，<u>一种切换线程的便利方式。</u>
+ Scheduler 协议中的 delay 用于延时发布
+ Scheduler 协议中的 debounce 用于防抖。
+ Empty 是一个最简单的发布者，不会有任何 `output` 值，只能用于表示某个事件已经发生 `Empty<Int, SampleError>()`
+ Just 是最简单的发送一个值，`Just(1)`
+ Publishers.Sequence<[Int],Never>(sequence:[1,2,3]) 可以简写成 [1,2,3].publisher



## publishers 中的常用方法



### reduce

> Publishers.reduce(initialResult:nextPartialResult:)，initialResult: 为初始值，nextPartialResult 为一个闭包处理函数，接收两个值，1、上次事件的计算值，2、本次事件值,
>
> Reduce 的语义是减少（汇总）成一个值

```swift
let one = [1,2,3,4,5].publisher

one.reduce(0) { (v, v2) -> Int in
    print(v,v2)
    return v+v2
}
//0 1
//1 2
//3 3
//6 4
//10 5
```

### scan

> Publishers.scan, 参数与 reduce 一致，不同的是将每一次计算值，以 publisher 事件发出。常见于下载

### compactMap

> 乎略（去除）nil 元素, 相当于简化的 `.map{Int($0)}.filter{$0!=nil}.map{$0!}`

### flatMap

> 将事件值重新生成为publisher发出，常见于 `二维数组` 的 `展平`

```swift
[[1,2,3],[4,5,6]]
.publisher
.flatMap{
  $0.publisher
}
```

## 错误处理

> 通常 publisher 定义完所有的错误处理行为，订阅方只负责 `sink`

### Fail

> Fail 最基本的错误 publisher 常用写法是: `Fail<Int,SampleError>(error: .samepleError)`

### mapErrror

> 对Error进行转换，保持错误最终订阅的一致性。

### tryMap

> 主动抛出错误，中断 publisher 的后续事件

### replaceError

> 将错误替换成给定值，同样会中断 publisher 的后续事件。

### catch

> 将错误替换成一个 publisher ,虽然同样会中断 后续事件，但同样可以利用新的 publisher 组织一个新的系列事件出来。

## 边界处理

### merge

>  合并 publisher

### zip

> 合并 publisher ,并将同一序列的事件结对

### combineLast

> 无论哪个 publisher 发生事件，都会与另一个 publisher 的最新值进行合并

### Future

> 绝大多数的现在的对象都提供了 publisher ，然后，如果对于没有提供的，可以使用 Future 自行实现。

```swift
Future<(Data, URLResponse), Error> { promise in
    loadPage(url: URL(string: "https://example.com")!) {
        data, response, error in
        if let data = data, let response = response {
        		promise(.success((data, response)))
        } else {
        		promise(.failure(error!))
        }
    }
}
```

### @Published

> Swift 中对 class 属性的一个封装，通过 get/set 的封，将 $属性包装成一个 publisher 

```swift
class Wrapper {
@Published var text: String = "hoho"
}
var wrapper = Wrapper()
check("Published") {
wrapper.$text
}
wrapper.text = "123"
wrapper.text = "abc"
// 输出：
// ----- Published -----
// receive subscription: (CurrentValueSubject)
// request unlimited
// receive value: (hoho)
// receive value: (123)
// receive value: (abc)”
```

### Cancellable , AnyCancellable 

> 满足一个 Cancellable 协议的对象，必须要用 connect() 来执行，得到一个 publisher, 返回



### coordinator 

> “makeCoordinator 是 UIViewControllerRepresentable 协议定义的一个方法。你可以在这里创建并返回一个任意类型的对象。这个对象在之后相关调用中都会被设置在 context.coordinator 属性里。使用这种方法，SwiftUI 让你可以在 View 的不同时期间传递任意数据。

# 绘图

## Path

### addArc (圆，赛道)

```swift
        Path { path in
            // 画一个缺口为180度的半圆
            // 起始角度 0 为右侧开始 # .degrees(0)
            // 结束角度为 180 结束  # .degrees(180)
            // 结束以顺时针开始画线  # clockwise: true
            path.addArc(center: .init(x: 100, y: 100), radius: 80, startAngle: .degrees(0), endAngle: .degrees(180), clockwise: true)
        }
        .stroke(Color.red,lineWidth:2)
```

### move

> 移动起始点

### addLine

> 画线，闭合后可在外层用 fill 方法

### addEllipse 椭圆只关注宽高

```swift 
Path{ path in
      path.addEllipse(in: CGRect(x: 0, y: 0, width: 100, height: 200))
}.stroke(Color.green, lineWidth: 6)
```



# 异常处理 Catch

```swift
import Foundation
import Combine

enum MyError:Error {
    case FetchError(content:String)
    case HostError
}

func check<P:Publisher>(publisher:()->P)->AnyCancellable{
    publisher()
        .sink { (completion) in
        switch completion {
        case .failure(let error):
            if let e = error as? MyError {
//                print("my error:",e)
                switch e {
                case .FetchError(content: let co):
                    print("fetch error:",co)
                default:
                    print("host error")
                }
                return
            }
            print("error: ",error)
        case .finished:
            print("finished: done")
        }
    } receiveValue: { (value) in
        print(value)
    }
}

check {
    Fail<Any, MyError>(error: MyError.HostError)
        // catch 的位子关系到上下文的推导，所以应当在 publisher 被 sink 之前进行捕获
        // 如果写到 check 方法中，则无法从上下文推导，无法通过编译
        .catch({ (err) -> AnyPublisher<Any, Never> in
            print(err)
            return Just(1).eraseToAnyPublisher()
        })
        .eraseToAnyPublisher()
}

check {
    Fail<Any, MyError>(error: MyError.FetchError(content: "test content")).eraseToAnyPublisher()
}
check {
    Fail<Any, Error>(error: URLError(URLError.cannotDecodeRawData)).eraseToAnyPublisher()
}
check {
    Just(1).eraseToAnyPublisher()
}
```

