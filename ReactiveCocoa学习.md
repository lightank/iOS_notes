# ReactiveCocoa学习
* [ReactiveObjC][ReactiveObjC] 
* [ReactiveCocoa][ReactiveCocoa]

本文主要是学习[ReactiveObjC][ReactiveObjC]

为了对`ReactiveObjC`有全方位了解，请查看其项目的 [README](https://github.com/ReactiveCocoa/ReactiveObjC/blob/master/README.md)、[Framework Overview][FrameworkOverview] 和 [Design Guidelines][Design Guidelines]。

实例：

所有对于判断表单输入是否合法的逻辑都被整合为一串逻辑了。每次不论哪个输入框被修改了，用户的输入都会被 reduce 成一个布尔值，然后就可以自动来控制注册按钮的可用状态了。

```objc
RACSignal *formValid = [RACSignal
  combineLatest:@[
    self.username.rac_textSignal,
    self.emailField.rac_textSignal,
    self.passwordField.rac_textSignal,
    self.passwordVerificationField.rac_textSignal
  ]
  reduce:^(NSString *username, NSString *email, NSString *password, NSString *passwordVerification) {
    return @([username length] > 0 && [email length] > 0 && [password length] > 8 && [password isEqual:passwordVerification]);
  }];

RAC(self.createButton.enabled) = formValid;
```

ReactiveCocoa由两大主要部分组成：[signals][signals] (RACSignal) 和 [sequences][sequences] (RACSequence)。

signal 和 sequence 都是streams，他们共享很多相同的方法。ReactiveCocoa在功能上做了语义丰富、一致性强的一致性设计：signal是*push*驱动的stream，sequence是*pull*驱动的stream。

ReactiveCocoa中的两个基本概念

>* 信号（Signal） a signal is a steam of values，signals can be transformed, combined,etc.
>* 订阅者（Subscriber） a subscriber subscribes to a signal. RAC lets blocks,objects, and properties subscribe to signals

# 概述

ReactiveCocoa由两大主要部分组成：signals (RACSignal) 和 sequences (RACSequence)。

signal 和 sequence 都是 [streams][streams]，他们共享很多相同的方法。ReactiveCocoa在功能上做了语义丰富、一致性强的一致性设计：signal是_push_驱动的stream，sequence是_pull_驱动的stream。

## RACSignal
>* 异步控制或事件驱动的数据源：Cocoa编程中大多数时候会关注用户事件或应用状态改变产生的响应。
>* 链式以来操作：网络请求是最常见的依赖性样例，前一个对server的请求完成后，下一个请求才能构建。
>* 并行独立动作：独立的数据集要并行处理，随后还要把他们合并成一个最终结果。这在Cocoa中很常见，特别是涉及到同步动作时。

---

>Signal会触发它们的subscriber三种不同类型的事件：
>* 下一个事件从stream中提供一个新值。不像Cocoa集合，它是完全可用的，甚至一个signal可以包含 nil。
>* 错误事件会在一个signal结束之前被标示出来这里有一个错误。这种事件可能包含一个 NSError 对象来标示什么发生了错误。错误必须被特殊处理——错误不会被包含在stream的值里面。
>* 完成事件标示signal成功结束，不会再有新的值会被加入到stream当中。完成事件也必须被单独控制——它不会出现在stream的值里面。
>
>一个signal的生命由很多下一个(next)事件和一个错误(error)或完成(completed)事件组成（后两者不同时出现）。


## RACSequence

>* 简化集合转换：你会痛苦地发现 Foundation 库中没有类似 map 和 filter、fold/reduce 等高级函数。
<br/>
> Sequence是一种集合，很像 NSArray。但和数组不同的是，一个sequence里的值默认是_延迟_加载的（只有需要的时候才加载），这样的话如果sequence只有一部分被用到，那么这种机制就会提高性能。像Cocoa的集合类型一样，sequence不接受 nil 值。
> 
> RACSequence 允许任意Cocoa集合在统一且显式地进行操作。

```
RACSequence *normalizedLongWords = [[words.rac_sequence
    filter:^ BOOL (NSString *word) {
        return [word length] >= 10;
    }]
    map:^(NSString *word) {
        return [word lowercaseString];
    }];
```

## 格式

将方法名左边对齐，更加清晰

```
[[self.usernameTextField.rac_textSignal
  filter:^BOOL(id value) {
    NSString *text = value; // implicit cast
    return text.length > 3;
  }]
  subscribeNext:^(id x) {
    NSLog(@"%@", x);
  }];
```

# 基本函数

## filter

```objc
RACSignal *signal = [@[ @1, @2, @3 ] rac_sequence].signal; 
signal = [signal filter:^BOOL(NSNumber *value) {
    return value.integerValue % 2;
}];
[signal subscribeNext:^(NSNumber *value) {
    NSLog(@"%@", value);
}];
```

![rac_filter][rac_filter]

## map

```
RACSignal *signal = [@[ @1, @2, @3 ] rac_sequence].signal;
  signal = [signal map:^id(NSNumber *value) {
    return @(value.integerValue * 2);
  }];
  [signal subscribeNext:^(NSNumber *value) {
    NSLog(@"%@", value);
  }];
```

![rac_map][rac_map]

## merge

```
RACSignal *signal1 = [@[ @1, @2 ] rac_sequence].signal;
RACSignal *signal2 = [@[ @4, @5 ] rac_sequence].signal;

[[signal1 merge:signal2] subscribeNext:^(NSNumber *value) {
    NSLog(@"%@", value);
}];
```

![rac_merge][rac_merge]

## combineLatest

```
RACSignal *signal1 = [@[ @1, @2 ] rac_sequence].signal;
  RACSignal *signal2 = [@[ @3, @4 ] rac_sequence].signal;

  [[signal1 combineLatestWith:signal2] subscribeNext:^(RACTuple *value) {
    NSLog(@"%@", value);
  }];
```

![rac_combineLatest][rac_combineLatest]

## combineLatest & reduce

```
RACSignal *signal1 = [@[ @1, @2 ] rac_sequence].signal;
  RACSignal *signal2 = [@[ @3, @4 ] rac_sequence].signal;

  [[[signal1 combineLatestWith:signal2]
      reduceEach:^id(NSNumber *v1, NSNumber *v2) {
        return @(v1.integerValue * v2.integerValue);
      }] subscribeNext:^(RACTuple *value) {
    NSLog(@"%@", value);
  }];
```

![rac_combineLatest%26reduce][rac_combineLatest%26reduce]

## flatten

```
RACSignal *signal1 = [@[ @1, @2 ] rac_sequence].signal;
RACSignal *signal2 = [RACSignal return:signal1];

[[signal2 flatten] subscribeNext:^(NSNumber *value) {
    NSLog(@"%@", value);
}];
```

![rac_flatten][rac_flatten]

## flattenMap

```
RACSignal *signal = [@[ @1, @2 ] rac_sequence].signal;

[[signal flattenMap:^RACStream *(NSNumber *value) {
    return [RACSignal return:@(value.integerValue * 2)];
}] subscribeNext:^(NSNumber *value) {
    NSLog(@"%@", value);
}];
```

![rac_flattenMap][rac_flattenMap]

## replay

```
RACSubject *letters = [RACReplaySubject subject];
RACSignal *signal = letters;
[signal subscribeNext:^(id x) {
  NSLog(@"S1: %@", x);
}];
[letters sendNext:@"A"];
[signal subscribeNext:^(id x) {
  NSLog(@"S2: %@", x);
}];
[letters sendNext:@"B"];
[signal subscribeNext:^(id x) {
  NSLog(@"S3: %@", x);
}];
[letters sendNext:@"C"];
```

![rac_replay][rac_replay]

## not replay

```
RACSubject *letters = [RACSubject subject];
RACSignal *signal = letters;
[signal subscribeNext:^(id x) {
    NSLog(@"S1: %@", x);
}];
[letters sendNext:@"A"];
[signal subscribeNext:^(id x) {
    NSLog(@"S2: %@", x);
}];
[letters sendNext:@"B"];
[letters sendNext:@"C"];
```
![rac_not_replay][rac_not_replay]

## replayLast

```
RACSubject *letters = [RACSubject subject];
RACSignal *signal = [letters replayLast];
[signal subscribeNext:^(id x) {
  NSLog(@"S1: %@", x);
}];
[letters sendNext:@"A"];
[signal subscribeNext:^(id x) {
  NSLog(@"S2: %@", x);
}];
[letters sendNext:@"B"];
[signal subscribeNext:^(id x) {
  NSLog(@"S3: %@", x);
}];
[letters sendNext:@"C"];
```

![rac_replayLast][rac_replayLast]

## replayLazily

```
RACSubject *letters = [RACSubject subject];
RACSignal *signal = [letters replayLazily];
[letters sendNext:@"A"];
[signal subscribeNext:^(id x) {
  NSLog(@"S1: %@", x);
}];
[letters sendNext:@"B"];
[signal subscribeNext:^(id x) {
  NSLog(@"S2: %@", x);
}];
[letters sendNext:@"C"];
[signal subscribeNext:^(id x) {
  NSLog(@"S3: %@", x);
}];
[letters sendNext:@"D"];
```

![rac_replayLazily][rac_replayLazily]

## zip

```
RACSubject *letters = [RACSubject subject];
RACSignal *signal = [letters replayLazily];
[letters sendNext:@"A"];
[signal subscribeNext:^(id x) {
  NSLog(@"S1: %@", x);
}];
[letters sendNext:@"B"];
[signal subscribeNext:^(id x) {
  NSLog(@"S2: %@", x);
}];
[letters sendNext:@"C"];
[signal subscribeNext:^(id x) {
  NSLog(@"S3: %@", x);
}];
[letters sendNext:@"D"];
```

![rac_zip][rac_zip]

## subscribeNext
RACSignal提供了若干的方法用以订阅这些不同的事件类型。每个方法都会接受一个或多个block作为入参，当新的事件出现时，block中的逻辑代码就会执行。在这个例子中，你可以看到subscribeNext:方法就提供了一个block，在每个next事件到达时执行里面的代码。

## map

对信号进行映射

## RAC宏

RAC宏将一个信号的输出和一个对象的属性绑定起来。宏接受两个参数，第一个是包含改变属性的对象，第二个为属性的名字。每次当信号发出一个新的事件，事件的值就会传递给绑定的属性。

```
RAC(self.passwordTextField, backgroundColor) =
  [validPasswordSignal
    map:^id(NSNumber *passwordValid) {
      return [passwordValid boolValue] ? [UIColor clearColor] : [UIColor yellowColor];
    }];
```

## RACTuplePack RACTupleUnpack

```
RACTuple *tuple = RACTuplePack(address, amount);
RACTupleUnpack(NSString *address, NSString *amount) = pair;
```

# 应用
## KVO 监听

```
@weakify(self)
    [RACObserve(self.person, name)
        subscribeNext:^(id x) {
             @strongify(self)
            self.nameLabel.text = x;
    }];
```

## 信号绑定
RAC是通过Category对UITextField进行绑定的，我们来看一下代码：
``` Objective-C
- (RACSignal *)rac_textSignal {
 @weakify(self);
 return [[[[[RACSignal
  defer:^{
   @strongify(self);
   return [RACSignal return:self];
  }]
  concat:[self rac_signalForControlEvents:UIControlEventAllEditingEvents]]
  map:^(UITextField *x) {
   return x.text;
  }]
  takeUntil:self.rac_willDeallocSignal]
  setNameWithFormat:@"%@ -rac_textSignal", self.rac_description];
}
```
逐步分析如上操作
### 通过defer进行信号的创建

该创建方式和createSignal的区别在于前者是延迟创建,只有在信号被订阅时才会创建

### concat关联信号

关联信号的实质就是订阅另一个信号.即点击事件所触发的信号.
`[self rac_signalForControlEvents:UIControlEventAllEditingEvents]`.
所有继承UIControl的类都可以对操作事件进行订阅,事件订阅是如何实现的呢?
我们来看一下代码:

```Objective-C
- (RACSignal *)rac_signalForControlEvents:(UIControlEvents)controlEvents {
 @weakify(self);

 return [[RACSignal
  createSignal:^(id<RACSubscriber> subscriber) {
   @strongify(self);

   [self addTarget:subscriber action:@selector(sendNext:) forControlEvents:controlEvents];
   [self.rac_deallocDisposable addDisposable:[RACDisposable disposableWithBlock:^{
    [subscriber sendCompleted];
   }]];

   return [RACDisposable disposableWithBlock:^{
    @strongify(self);
    [self removeTarget:subscriber action:@selector(sendNext:) forControlEvents:controlEvents];
   }];
  }]
  setNameWithFormat:@"%@ -rac_signalForControlEvents: %lx", self.rac_description, (unsigned long)controlEvents];
}
```

关键代码: `[self addTarget:subscriber action:@selector(sendNext:) forControlEvents:controlEvents];`

当前信号订阅了信号B,信号B在所有`UITextField`的编辑状`UIControlEventAllEditingEvents`事件被触发时,信号B发送`next`事件,进而通知当前信号

### `map`修改返回值

map方法的作用,即修改信号内的返回值,由于外部并不关心`UITextField`本身,而是它的文本内容,所以只返回text属性.

map是怎么实现的呢?

```Objective-C
- (instancetype)map:(id (^)(id value))block {
    NSCParameterAssert(block != nil);

    Class class = self.class;
    
    return [[self flattenMap:^(id value) {
        return [class return:block(value)];
    }] setNameWithFormat:@"[%@] -map:", self.name];
}
```

在这里我们讨论一下map和flattenMap的使用场景

* `map`用于处理信号的返回值
* `flattenMap`用于处理信号中的信号

### takeUntil确定信号的生命周期

订阅UITextField的生命周期信号,在组件dealloc的时候,处理掉该信号
看代码

```
- (RACSignal *)rac_willDeallocSignal {
 RACSignal *signal = objc_getAssociatedObject(self, _cmd);
 if (signal != nil) return signal;

 RACReplaySubject *subject = [RACReplaySubject subject];

 [self.rac_deallocDisposable addDisposable:[RACDisposable disposableWithBlock:^{
  [subject sendCompleted];
 }]];

 objc_setAssociatedObject(self, _cmd, subject, OBJC_ASSOCIATION_RETAIN);

 return subject;
}
```

RAC通过`runtime`对`dealloc`信号进行管理,`addDisposable`的官方解释是`Adds the given disposable. If the receiving disposable has already been disposed of, the given disposable is disposed immediately`,即可以理解为是两个信号关联释放.在这里的作用是,当外部订阅生命周期的信号被释放时,监听生命周期的信号本身已经没有存在的意义,随之释放.

### setNameWithFormat设置信号名

为信号设置一个别名,用于后续的调试,这里就不做赘述了.


# 输入框的双向绑定
* 无论是通过键盘或者code赋值来改变textField.text的值的时候，我们都要让绑定的string对应的改变，这才是我们要的双向绑定
    
| RACChannelTo | rac_newTextChannel |
| --- | --- |
| 当通过code改变self.textField.text的值的时候才会把这个值发送出去 | 当通过键盘改变self.textField. text的值的时候才会把这个值发送出去 |

```
RACChannelTo(self, string) = RACChannelTo(self.textField, text);
    @weakify(self);
    [self.textField.rac_textSignal subscribeNext:^(NSString * _Nullable x) {
        @strongify(self);
        self.string = x;
    }];

简化代码

    RACChannelTo(self, string) = RACChannelTo(self.textField, text);
    [self.textField.rac_textSignal subscribe:RACChannelTo(self, string)];
```

* UITextView的情况及处理跟UITextField一样
对于整篇文章双向绑定的分析，主要围绕三个点

```
string -> textField.text
键盘改变textField.text -> string
code赋值textField.text -> string
满足上面三点就算完全的双向绑定
```


# 参考链接
* [干货集中营-ReactiveCocoa+RXSwift+MVVM](https://www.jianshu.com/p/f32a4824797e)
* [图解ReactiveCocoa基本函数](https://www.jianshu.com/p/38d39923ee81)
* [Reactive​Cocoa](https://nshipster.cn/reactivecocoa/) [Mattt](https://nshipster.cn/authors/mattt/)撰写、 [Croath Liu](https://nshipster.cn/translators/croath-liu/)翻译
* [ReactiveCocoa代码分析之UITextField](https://www.jianshu.com/p/a027898612e6)
* [ReactiveCocoa教程：上半部【译】][ReactiveCocoa教程：上半部【译】]
* [ReactiveCocoa教程：下半部【译】][ReactiveCocoa教程：下半部【译】]
* [Blocks Programming Topics](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Blocks/Articles/00_Introduction.html#//apple_ref/doc/uid/TP40007502-CH1-SW1)
* [How Do I Declare A Block in Objective-C?](http://goshdarnblocksyntax.com)
* [图解ReactiveCocoa基本函数](https://www.jianshu.com/p/38d39923ee81)

[ReactiveObjC]:https://github.com/ReactiveCocoa/ReactiveObjC
[ReactiveCocoa]:https://github.com/ReactiveCocoa/ReactiveCocoa
[FrameworkOverview]:https://github.com/ReactiveCocoa/ReactiveObjC/blob/master/Documentation/FrameworkOverview.md
[Design Guidelines]:https://github.com/ReactiveCocoa/ReactiveObjC/blob/master/Documentation/DesignGuidelines.md
[signals]:https://github.com/ReactiveCocoa/ReactiveObjC/blob/master/Documentation/FrameworkOverview.md#signals
[sequences]:https://github.com/ReactiveCocoa/ReactiveObjC/blob/master/Documentation/FrameworkOverview.md#sequences
[streams]:https://github.com/ReactiveCocoa/ReactiveObjC/blob/master/Documentation/FrameworkOverview.md#streams
[ReactiveCocoa教程：上半部【译】]:https://www.jianshu.com/p/247df483a3cf
[ReactiveCocoa教程：下半部【译】]:https://www.jianshu.com/p/1fc69a1d9471
[rac_combineLatest%26reduce]:https://github.com/lightank/iOS_notes/blob/master/Resource/rac/rac_combineLatest%26reduce.png
[rac_combineLatest]:https://github.com/lightank/iOS_notes/blob/master/Resource/rac/rac_combineLatest.png
[rac_filter]:https://github.com/lightank/iOS_notes/blob/master/Resource/rac/rac_filter.png
[rac_flatten]:https://github.com/lightank/iOS_notes/blob/master/Resource/rac/rac_flatten.png
[rac_flattenMap]:https://github.com/lightank/iOS_notes/blob/master/Resource/rac/rac_flattenMap.png
[rac_map]:https://github.com/lightank/iOS_notes/blob/master/Resource/rac/rac_map.png
[rac_merge]:https://github.com/lightank/iOS_notes/blob/master/Resource/rac/rac_merge.png
[rac_not_replay]:https://github.com/lightank/iOS_notes/blob/master/Resource/rac/rac_not_replay.png
[rac_replay]:https://github.com/lightank/iOS_notes/blob/master/Resource/rac/rac_replay.png
[rac_replayLast]:https://github.com/lightank/iOS_notes/blob/master/Resource/rac/rac_replayLast.png
[rac_replayLazily]:https://github.com/lightank/iOS_notes/blob/master/Resource/rac/rac_replayLazily.png
[rac_zip]:https://github.com/lightank/iOS_notes/blob/master/Resource/rac/rac_zip.png

