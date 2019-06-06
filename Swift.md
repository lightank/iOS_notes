# Swift
* 颜色，输入 `color literal` 就会出现颜色快
* 在字符串使用 `\(字符串里的对象内容)` 
* Swift 要求所有实例变量（Swift里实例变量就是属性，属性就是实例变量）都要初始化，如果有一个属性没有实例化就会报错 `Class 'xxxx' has noinitializers`,属性不能定义完就完事，得有初始值，可以在定义的时候给初始值，Swift有强大的类型推测，一般不需要指定类型
* 属性观察器
    
    ```
    var flipCount: Int = 0 {
        didSet {
            
        }
    }
    ```
* storyboard 中相同类型的视图可以command 多选后一起编辑
* 可选值是一种类型，有且只有两种状态，有值和缺省值，这是个枚举值，Swift的每个枚举情况都可以有关联值，如果有值就作为关联值返回给你，nil作为可选值缺省值。nil总是代表可选值类型缺省值。当你使用 `!` 强制解包时如果返回一个可选类型缺省值，这个时候没有关联值，就会崩溃，崩溃信息 `Unexpectedly found nil while unwrapping an Optional value`, 关于可选类型最小语法 `if let number = carButtons.firstIndex(of: sender)`。 可选值 ?? "?" 如果为空则返回"?"
* nil 仅代表可选值not set 状态
* assert([].indices.contains(index), "out of")
* enum func 用mutating 修饰就可以 self = .case,当然这个枚举值得var
* 四大支柱：类、结构体、枚举、协议，这些都是type
* 引用类型：类、闭包
* 协议必须全部实现，加@objc代表这是个oc的协议，可以使用optional做可选，这是个向下兼容的方案
* 值类型方法用mutating 修饰才可以修改值
* protocol SomeProtocol：class， InheritedProtocols  代表这个协议只能被类继承，这样就不用mutating 修饰
* func中参数需继承多个协议，那么用 & 来连接 func method(x: protocol1 & protocol2)
* Self:类的type