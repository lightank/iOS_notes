# Swift
* 可选值 ?? "?" 如果为空则返回"?"
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