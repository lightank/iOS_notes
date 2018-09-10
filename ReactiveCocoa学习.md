# ReactiveObjC学习
* [ReactiveObjC][ReactiveObjC] 
* [ReactiveCocoa][ReactiveCocoa]

本文主要是学习[ReactiveObjC][ReactiveObjC]

为了对`ReactiveObjC`有全方位了解，请查看其项目的 [README](https://github.com/ReactiveCocoa/ReactiveObjC/blob/master/README.md)、[Framework Overview][FrameworkOverview] 、[Memory Management][MemoryManagement] 和 [Design Guidelines][Design Guidelines]。

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

```objc
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

```objc
[[self.usernameTextField.rac_textSignal
  filter:^BOOL(id value) {
    NSString *text = value; // implicit cast
    return text.length > 3;
  }]
  subscribeNext:^(id x) {
    NSLog(@"%@", x);
  }];
```

## 内存管理

匿名构造管道是`ReactiveObjc`的其中一个设计理念。回顾至今为止你写的所有响应式代码，这应该是显而易见的。

为了支持这种特性，`ReactiveObjc`维系保持了它自己的全局信号集（`global set of signals`）。如果信号有一个或多个订阅者的话，信号就会被激活。如果所有的订阅者都给移除了，该信号就可以被回收。想知道更多关于`ReactiveObjc`内管理这个过程的内容，你可以浏览 [Memory Management][MemoryManagement]文档。

这就剩下最后一个问题了：怎样取消信号的订阅呢？订阅在接收到`completed`或`error`事件后，就会自动移除（你很快就会学到更多关于这部分的内容）。而要手动移除的话可以借助`RACDisposable`.

`RACSignal`的订阅方法都返回了一个`RACDisposable`实例用以在处理方法中手动移除订阅。举一个基于现有管道的简单例子：

```
RACSignal *backgroundColorSignal =
  [self.searchText.rac_textSignal
    map:^id(NSString *text) {
      return [self isValidSearchText:text] ?
        [UIColor whiteColor] : [UIColor yellowColor];
    }];
 
RACDisposable *subscription =
  [backgroundColorSignal
    subscribeNext:^(UIColor *color) {
      self.searchText.backgroundColor = color;
    }];
 
// at some point in the future ...
[subscription dispose];
```

可能在实际中你很少会这样做，但知道这么一个可行操作还是很有价值的。

> 注意：相对应的，如果你创建了一个管道但不曾对其订阅，这管道里的代码，包括像`doNext:`这样的副作用都永远不会执行。


# 基本函数

## - ignore: (id)

忽略给定的值，注意，这里忽略的既可以是地址相同的对象，也可以是- isEqual:结果相同的值，也就是说自己写的Model对象可以通过重写- isEqual:方法来使- ignore:生效。常用的值的判断没有问题，如下：

```objc
[[self.inputTextField.rac_textSignal ignore:@"sunny"] subscribeNext:^(NSString *value) {
    NSLog(@"`sunny` could never appear : %@", value);
}];
```

## -ignoreValues

这个比较极端，忽略所有值，只关心Signal结束，也就是只取Comletion和Error两个消息，中间所有值都丢弃。
注意，这个操作应该出现在Signal有终止条件的的情况下，如rac_textSignal这样除dealloc外没有终止条件的Signal上就不太可能用到。

## filter

过滤信号，使用它可以获取满足条件的信号.

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

```objc
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

```objc
RACSignal *signal1 = [@[ @1, @2 ] rac_sequence].signal;
RACSignal *signal2 = [@[ @4, @5 ] rac_sequence].signal;

[[signal1 merge:signal2] subscribeNext:^(NSNumber *value) {
    NSLog(@"%@", value);
}];
```

![rac_merge][rac_merge]

## ignore

忽略调某些值的信号.

```
// 内部调用filter过滤，忽略掉ignore的值
[[_textField.rac_textSignal ignore:@"1"] subscribeNext:^(id x) {
    
    NSLog(@"%@",x);
}];
```

## distinctUntilChanged

当上一次的值和当前的值有明显的变化就会发出信号，否则会被忽略掉。

```objc
// 过滤，当上一次和当前的值不一样，就会发出内容。
// 在开发中，刷新UI经常使用，只有两次数据不一样才需要刷新
[[_textField.rac_textSignal distinctUntilChanged] subscribeNext:^(id x) {
  
    NSLog(@"%@",x);
}];
```

## take

从开始一共取N次的信号

```objc
// 1、创建信号
RACSubject *signal = [RACSubject subject];

// 2、处理信号，订阅信号
[[signal take:1] subscribeNext:^(id x) {
    
    NSLog(@"%@",x);
}];

// 3.发送信号
[signal sendNext:@1];
[signal sendNext:@2];
```

## takeLast

取最后N次的信号,前提条件，订阅者必须调用完成，因为只有完成，就知道总共有多少信号.

```objc
// 1、创建信号
RACSubject *signal = [RACSubject subject];

// 2、处理信号，订阅信号
[[signal takeLast:1] subscribeNext:^(id x) {
    NSLog(@"%@",x);
}];

// 3.发送信号
[signal sendNext:@1];

[signal sendNext:@2];

[signal sendCompleted];
```

## takeUntil:(RACSignal *)

当给定的signal完成前一直取值。最简单的栗子就是UITextField的rac_textSignal的实现（删减版本）:

```objc
- (RACSignal *)rac_textSignal {
	@weakify(self);
	return [[[[[RACSignal
		concat:[self rac_signalForControlEvents:UIControlEventEditingChanged]]
		map:^(UITextField *x) {
			return x.text;
		}]
		takeUntil:self.rac_willDeallocSignal] // bingo!
}
```

## - takeUntilBlock:(BOOL (^)(id x))

对于每个next值，运行block，当block返回YES时停止取值，如：

```objc
[[self.inputTextField.rac_textSignal takeUntilBlock:^BOOL(NSString *value) {
    return [value isEqualToString:@"stop"];
}] subscribeNext:^(NSString *value) {
    NSLog(@"current value is not `stop`: %@", value);
}];
```

## - takeWhileBlock:(BOOL (^)(id x))

上面的反向逻辑，对于每个next值，block返回 YES时才取值


## skip:(NSUInteger)

跳过几个信号,不接受。

```objc
// 表示输入第一次，不会被监听到，跳过第一次发出的信号
[[_textField.rac_textSignal skip:1] subscribeNext:^(id x) {
    NSLog(@"%@",x);
}];
```

## - skipUntilBlock:(BOOL (^)(id x))

和`- takeUntilBlock:`同理，一直跳，直到block为YES

## - skipWhileBlock:(BOOL (^)(id x))

和`- takeWhileBlock:`同理，一直跳，直到block为NO

## switchToLatest

用于signalOfSignals（信号的信号），有时候信号也会发出信号，会在signalOfSignals中，获取signalOfSignals发送的最新信号。

```objc
RACSubject *signalOfSignals = [RACSubject subject];
RACSubject *signal = [RACSubject subject];

// 获取信号中信号最近发出信号，订阅最近发出的信号。
// 注意switchToLatest：只能用于信号中的信号
[signalOfSignals.switchToLatest subscribeNext:^(id x) {
    NSLog(@"%@",x);
}];
[signalOfSignals sendNext:signal];
[signal sendNext:@1];
```

## combineLatest

```objc
RACSignal *signal1 = [@[ @1, @2 ] rac_sequence].signal;
  RACSignal *signal2 = [@[ @3, @4 ] rac_sequence].signal;

  [[signal1 combineLatestWith:signal2] subscribeNext:^(RACTuple *value) {
    NSLog(@"%@", value);
  }];
```

![rac_combineLatest][rac_combineLatest]


## combineLatestWith

将多个信号合并起来，并且拿到各个信号的最新的值,必须每个合并的signal至少都有过一次sendNext，才会触发合并的信号。

```objc
RACSignal *signalA = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
       
       [subscriber sendNext:@1];
       
       return nil;
   }];
   
   RACSignal *signalB = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
       
       [subscriber sendNext:@2];
       
       return nil;
   }];

   // 把两个信号组合成一个信号,跟zip一样，没什么区别
   RACSignal *combineSignal = [signalA combineLatestWith:signalB];
 
   [combineSignal subscribeNext:^(id x) {
      
       NSLog(@"%@",x);
   }];
   
   // 底层实现：
   // 1.当组合信号被订阅，内部会自动订阅signalA，signalB,必须两个信号都发出内容，才会被触发。
   // 2.并且把两个信号组合成元组发出。
```

## reduce

分解，将元组里的值拆分出来。

```
RACSignal *signalA = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
     
     [subscriber sendNext:@1];
     
     return nil;
 }];
 
 RACSignal *signalB = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
     
     [subscriber sendNext:@2];
     
     return nil;
 }];
 
 // 聚合
 // 常见的用法，（先组合在聚合）。combineLatest:(id<NSFastEnumeration>)signals reduce:(id (^)())reduceBlock
 // reduce中的block简介:
 // reduceblcok中的参数，有多少信号组合，reduceblcok就有多少参数，每个参数就是之前信号发出的内容
 // reduceblcok的返回值：聚合信号之后的内容。
RACSignal *reduceSignal = [RACSignal combineLatest:@[signalA,signalB] reduce:^id(NSNumber *num1 ,NSNumber *num2){
 
    return [NSString stringWithFormat:@"%@ %@",num1,num2];
    
}];
 
 [reduceSignal subscribeNext:^(id x) {
    
     NSLog(@"%@",x);
 }];
 
 // 底层实现:
 // 1.订阅聚合信号，每次有内容发出，就会执行reduceblcok，把信号内容转换成reduceblcok返回的值。
```

## combineLatest & reduce

```objc
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

解包信号里的信号

```objc
RACSignal *signal1 = [@[ @1, @2 ] rac_sequence].signal;
RACSignal *signal2 = [RACSignal return:signal1];

[[signal2 flatten] subscribeNext:^(NSNumber *value) {
    NSLog(@"%@", value);
}];
```

![rac_flatten][rac_flatten]

## flattenMap

解包信号里的信号

```objc
RACSignal *signal = [@[ @1, @2 ] rac_sequence].signal;

[[signal flattenMap:^RACStream *(NSNumber *value) {
    return [RACSignal return:@(value.integerValue * 2)];
}] subscribeNext:^(NSNumber *value) {
    NSLog(@"%@", value);
}];
```

![rac_flattenMap][rac_flattenMap]

## replay

重放：当一个信号被多次订阅,反复播放内容

```objc
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

```objc
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

```objc
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

```objc
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

```objc
RACSubject *letters = [RACSubject subject];
RACSubject *numbers = [RACSubject subject];

RACSignal *combined =
    [RACSignal zip:@[ letters, numbers ]
            reduce:^(NSString *letter, NSString *number) {
              return [letter stringByAppendingString:number];
            }];

// Outputs: A1 B2 C3
[combined subscribeNext:^(id x) {
  NSLog(@"%@", x);
}];

[letters sendNext:@"A"];
[letters sendNext:@"B"];
[letters sendNext:@"C"];
[numbers sendNext:@"1"];
[numbers sendNext:@"2"];
[numbers sendNext:@"3"];
```

![rac_zip][rac_zip]

## zipWith

把两个信号压缩成一个信号，只有当两个信号同时发出信号内容时，并且把两个信号的内容合并成一个元组，才会触发压缩流的next事件。

```objc
RACSignal *signalA = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        
        [subscriber sendNext:@1];
        
        
        return nil;
    }];
    
    RACSignal *signalB = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        
        [subscriber sendNext:@2];
        
        return nil;
    }];

    
    
    // 压缩信号A，信号B
    RACSignal *zipSignal = [signalA zipWith:signalB];
    
    [zipSignal subscribeNext:^(id x) {
       
        NSLog(@"%@",x);
    }];
    
    // 底层实现:
    // 1.定义压缩信号，内部就会自动订阅signalA，signalB
    // 2.每当signalA或者signalB发出信号，就会判断signalA，signalB有没有发出个信号，有就会把最近发出的信号都包装成元组发出。
```

## doNext

执行Next之前，会先执行这个Block

```objc
[[[[self.signInButton
   rac_signalForControlEvents:UIControlEventTouchUpInside]
   doNext:^(id x) {
     self.signInButton.enabled = NO;
     self.signInFailureText.hidden = YES;
   }]
   flattenMap:^id(id x) {
     return [self signInSignal];
   }]
   subscribeNext:^(NSNumber *signedIn) {
     self.signInButton.enabled = YES;
     BOOL success = [signedIn boolValue];
     self.signInFailureText.hidden = success;
     if (success) {
       [self performSegueWithIdentifier:@"signInSuccess" sender:self];
     }
   }];
```

到上面在按钮点击事件创建后添加了一个`doNext:`环节。注意`doNext:`是一个副作用，所以`block`没有返回任何值；它并不影响事件的内容。

`doNext:`的`block`中把按钮的的可用属性设为`NO`，同时隐藏了失败文本。直到`subscribeNext:`的`block`中按钮才再次变为可用，并根据登录的结果决定显示或隐藏失败文本。

![rac_doNext][rac_doNext]

## doCompleted

执行sendCompleted之前，会先执行这个Block

```objc
[[[[RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
[subscriber sendNext:@1];
[subscriber sendCompleted];
return nil;
}] doNext:^(id x) {
// 执行[subscriber sendNext:@1];之前会调用这个Block
NSLog(@"doNext");;
}] doCompleted:^{
// 执行[subscriber sendCompleted];之前会调用这个Block
NSLog(@"doCompleted");;
}] subscribeNext:^(id x) {
    NSLog(@"%@",x);
}]; 
```

## then

```objc
[[[self requestAccessToTwitterSignal]
  then:^RACSignal *{
    @strongify(self)
    return self.searchText.rac_textSignal;
  }]
  subscribeNext:^(id x) {
    NSLog(@"%@", x);
  } error:^(NSError *error) {
    NSLog(@"An error occurred: %@", error);
  }];
```

`then`方法会一直等待直到信号的`completed`事件被发送，然后转订阅参数代码块中返回的信号。这有效的将控制权从一个信号转递给下一个信号。

>注意：你已经在上一个管道弱引用过self，所以不再需要在这个管道前添加@weakify(self)了。


##  deliverOn:

内容传递切换到制定线程中，副作用在原来线程中,把在创建信号时block中的代码称之为副作用。

```objc
[[[[[[self requestAccessToTwitterSignal]
  then:^RACSignal *{
    @strongify(self)
    return self.searchText.rac_textSignal;
  }]
  filter:^BOOL(NSString *text) {
    @strongify(self)
    return [self isValidSearchText:text];
  }]
  flattenMap:^RACStream *(NSString *text) {
    @strongify(self)
    return [self signalForSearchWithText:text];
  }]
  deliverOn:[RACScheduler mainThreadScheduler]]
  subscribeNext:^(id x) {
    NSLog(@"%@", x);
  } error:^(NSError *error) {
    NSLog(@"An error occurred: %@", error);
  }];
```

上面的操作是在信号开始发送信号的线程中执行的。在管道的其他环节添加断点，你可能会惊讶地发现他们并不在同一个线程中执行！

>注意：如果你关注一下RACScheduler类你就能看到其提供了相当多的选择来实现不同的线程优先级和管道延迟处理。

## subscribeOn

内容传递和副作用都会切换到制定线程中。

## timeout

超时，可以让一个信号在一定的时间后，自动报错

```objc
RACSignal *signal = [[RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
    return nil;
}] timeout:1 onScheduler:[RACScheduler currentScheduler]];

[signal subscribeNext:^(id x) {
    
    NSLog(@"%@",x);
} error:^(NSError *error) {
    // 1秒后会自动调用
    NSLog(@"%@",error);
}];
```

## interval

定时：每隔一段时间发出信号

```objc
[[RACSignal interval:1 onScheduler:[RACScheduler currentScheduler]] subscribeNext:^(id x) {
   
    NSLog(@"%@",x);
}];
```

## startWith

给一个初始值

```objc
RAC(self.outputLabel, text) = [[self.inputTextField.rac_textSignal
    startWith:@"key is >3"] /* startWith 一开始返回的初始值 */
    filter:^BOOL(NSString *value) {
        return value.length > 3; /* filter使满足条件的值才能传出 */
}];
```

## retry

重试 ：只要失败，就会重新执行创建信号中的block,直到成功.

```objc
__block int i = 0;
    [[[RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
    
            if (i == 10) {
                [subscriber sendNext:@1];
            }else{
                NSLog(@"接收到错误");
                [subscriber sendError:nil];
            }
            i++;
        
        return nil;
        
    }] retry] subscribeNext:^(id x) {
        
        NSLog(@"%@",x);
        
    } error:^(NSError *error) {
      
        
    }];
```


## delay

延迟发送next

```objc
RACSignal *signal = [[[RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
   
    [subscriber sendNext:@1];
    return nil;
}] delay:2] subscribeNext:^(id x) {
  
    NSLog(@"%@",x);
}];
```

## throttle

```objc
[[[[[[[self requestAccessToTwitterSignal]
  then:^RACSignal *{
    @strongify(self)
    return self.searchText.rac_textSignal;
  }]
  filter:^BOOL(NSString *text) {
    @strongify(self)
    return [self isValidSearchText:text];
  }]
  throttle:0.5]
  flattenMap:^RACStream *(NSString *text) {
    @strongify(self)
    return [self signalForSearchWithText:text];
  }]
  deliverOn:[RACScheduler mainThreadScheduler]]
  subscribeNext:^(NSDictionary *jsonSearchResult) {
    NSArray *statuses = jsonSearchResult[@"statuses"];
    NSArray *tweets = [statuses linq_select:^id(id tweet) {
      return [RWTweet tweetWithStatus:tweet];
    }];
    [self.resultsViewController displayTweets:tweets];
  } error:^(NSError *error) {
    NSLog(@"An error occurred: %@", error);
  }];
```

`throttle`节流操作只有在时间间隔内没有接收到新的`next`事件时才会发送`next`事件给下一环节。这是不是相当简单！

编译运行，这时搜索结果只在停止输入超过500毫秒时才会更新。这感觉好多了对吗？你的用户也会这么想的。


## subscribeNext
RACSignal提供了若干的方法用以订阅这些不同的事件类型。每个方法都会接受一个或多个block作为入参，当新的事件出现时，block中的逻辑代码就会执行。在这个例子中，你可以看到subscribeNext:方法就提供了一个block，在每个next事件到达时执行里面的代码。

## map

用于把源信号内容映射成新的内容

```objc
// 监听文本框的内容改变，把结构重新映射成一个新值.

    // Map作用:把源信号的值映射成一个新的值

    // Map使用步骤:
    // 1.传入一个block,类型是返回对象，参数是value
    // 2.value就是源信号的内容，直接拿到源信号的内容做处理
    // 3.把处理好的内容，直接返回就好了，不用包装成信号，返回的值，就是映射的值。

    // Map底层实现:
    // 0.Map底层其实是调用flatternMap,Map中block中的返回的值会作为flatternMap中block中的值。
    // 1.当订阅绑定信号，就会生成bindBlock。
    // 3.当源信号发送内容，就会调用bindBlock(value, *stop)
    // 4.调用bindBlock，内部就会调用flattenMap的block
    // 5.flattenMap的block内部会调用Map中的block，把Map中的block返回的内容包装成返回的信号。
    // 5.返回的信号最终会作为bindBlock中的返回信号，当做bindBlock的返回信号。
    // 6.订阅bindBlock的返回信号，就会拿到绑定信号的订阅者，把处理完成的信号内容发送出来。

       [[_textField.rac_textSignal map:^id(id value) {
        // 当源信号发出，就会调用这个block，修改源信号的内容
        // 返回值：就是处理完源信号的内容。
        return [NSString stringWithFormat:@"输出:%@",value];
    }] subscribeNext:^(id x) {

        NSLog(@"%@",x);
    }];
```

## flattenMap

用于把源信号内容映射成新的内容

```objc
// 监听文本框的内容改变，把结构重新映射成一个新值.

  // flattenMap作用:把源信号的内容映射成一个新的信号，信号可以是任意类型。

    // flattenMap使用步骤:
    // 1.传入一个block，block类型是返回值RACStream，参数value
    // 2.参数value就是源信号的内容，拿到源信号的内容做处理
    // 3.包装成RACReturnSignal信号，返回出去。

    // flattenMap底层实现:
    // 0.flattenMap内部调用bind方法实现的,flattenMap中block的返回值，会作为bind中bindBlock的返回值。
    // 1.当订阅绑定信号，就会生成bindBlock。
    // 2.当源信号发送内容，就会调用bindBlock(value, *stop)
    // 3.调用bindBlock，内部就会调用flattenMap的block，flattenMap的block作用：就是把处理好的数据包装成信号。
    // 4.返回的信号最终会作为bindBlock中的返回信号，当做bindBlock的返回信号。
    // 5.订阅bindBlock的返回信号，就会拿到绑定信号的订阅者，把处理完成的信号内容发送出来。

    [[_textField.rac_textSignal flattenMap:^RACStream *(id value) {
        // block什么时候 : 源信号发出的时候，就会调用这个block。
        // block作用 : 改变源信号的内容。
        // 返回值：绑定信号的内容.
        return [RACReturnSignal return:[NSString stringWithFormat:@"输出:%@",value]];
    }] subscribeNext:^(id x) {
        // 订阅绑定信号，每当源信号发送内容，做完处理，就会调用这个block。
        NSLog(@"%@",x);
    }];
```

## FlatternMap和Map的区别

1. FlatternMap中的Block返回信号。
2. Map中的Block返回对象。
3. 开发中，如果信号发出的值不是信号，映射一般使用Map
4. 开发中，如果信号发出的值是信号，映射一般使用FlatternMap。

总结：signalOfsignals用FlatternMap。

```objc
// 创建信号中的信号
    RACSubject *signalOfsignals = [RACSubject subject];
    RACSubject *signal = [RACSubject subject];

    [[signalOfsignals flattenMap:^RACStream *(id value) {

     // 当signalOfsignals的signals发出信号才会调用

        return value;

    }] subscribeNext:^(id x) {

        // 只有signalOfsignals的signal发出信号才会调用，因为内部订阅了bindBlock中返回的信号，也就是flattenMap返回的信号。
        // 也就是flattenMap返回的信号发出内容，才会调用。

        NSLog(@"%@aaa",x);
    }];

    // 信号的信号发送信号
    [signalOfsignals sendNext:signal];

    // 信号发送内容
    [signal sendNext:@1];
```

## concat

按一定顺序拼接信号，当多个信号发出的时候，有顺序的接收信号。

```objc
RACSignal *signalA = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        
        [subscriber sendNext:@1];
        
        [subscriber sendCompleted];
        
        return nil;
    }];
    RACSignal *signalB = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        
        [subscriber sendNext:@2];
        
        return nil;
    }];
    
    // 把signalA拼接到signalB后，signalA发送完成，signalB才会被激活。
    RACSignal *concatSignal = [signalA concat:signalB];
    
    // 以后只需要面对拼接信号开发。
    // 订阅拼接的信号，不需要单独订阅signalA，signalB
    // 内部会自动订阅。
    // 注意：第一个信号必须发送完成，第二个信号才会被激活
    [concatSignal subscribeNext:^(id x) {
        
        NSLog(@"%@",x);
        
    }];
    
    // concat底层实现:
    // 1.当拼接信号被订阅，就会调用拼接信号的didSubscribe
    // 2.didSubscribe中，会先订阅第一个源信号（signalA）
    // 3.会执行第一个源信号（signalA）的didSubscribe
    // 4.第一个源信号（signalA）didSubscribe中发送值，就会调用第一个源信号（signalA）订阅者的nextBlock,通过拼接信号的订阅者把值发送出来.
    // 5.第一个源信号（signalA）didSubscribe中发送完成，就会调用第一个源信号（signalA）订阅者的completedBlock,订阅第二个源信号（signalB）这时候才激活（signalB）。
    // 6.订阅第二个源信号（signalB）,执行第二个源信号（signalB）的didSubscribe
    // 7.第二个源信号（signalA）didSubscribe中发送值,就会通过拼接信号的订阅者把值发送出来.
```

## then

用于连接两个信号，当第一个信号完成，才会连接then返回的信号

```objc
// then:用于连接两个信号，当第一个信号完成，才会连接then返回的信号
// 注意使用then，之前信号的值会被忽略掉.
// 底层实现：1、先过滤掉之前的信号发出的值。2.使用concat连接then返回的信号
[[[RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
 
    [subscriber sendNext:@1];
    [subscriber sendCompleted];
    return nil;
}] then:^RACSignal *{
    return [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        [subscriber sendNext:@2];
        return nil;
    }];
}] subscribeNext:^(id x) {
  
    // 只能接收到第二个信号的值，也就是then返回信号的值
    NSLog(@"%@",x);
}];
```
## merge

把多个信号合并为一个信号，任何一个信号有新值的时候就会调用

```
// merge:把多个信号合并成一个信号
    //创建多个信号
    RACSignal *signalA = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        
        [subscriber sendNext:@1];
        
        
        return nil;
    }];
    
    RACSignal *signalB = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        
        [subscriber sendNext:@2];
        
        return nil;
    }];

    // 合并信号,任何一个信号发送数据，都能监听到.
    RACSignal *mergeSignal = [signalA merge:signalB];
    
    [mergeSignal subscribeNext:^(id x) {
       
        NSLog(@"%@",x);
        
    }];
    
    // 底层实现：
    // 1.合并信号被订阅的时候，就会遍历所有信号，并且发出这些信号。
    // 2.每发出一个信号，这个信号就会被订阅
    // 3.也就是合并信号一被订阅，就会订阅里面所有的信号。
    // 4.只要有一个信号被发出就会被监听。
```

## bind

`ReactiveObjc`操作的核心方法是`bind`（绑定）,而且RAC中核心开发方式，也是绑定，之前的开发方式是赋值，而用RAC开发，应该把重心放在绑定，也就是可以在创建一个对象的时候，就绑定好以后想要做的事情，而不是等赋值之后在去做事情。

在开发中很少使用bind方法，bind属于RAC中的底层方法，RAC已经封装了很多好用的其他方法，底层都是调用bind，用法比bind简单.

```
// 假设想监听文本框的内容，并且在每次输出结果的时候，都在文本框的内容拼接一段文字“输出：”

    // 方式一:在返回结果后，拼接。
        [_textField.rac_textSignal subscribeNext:^(id x) {

            NSLog(@"输出:%@",x);

        }];

    // 方式二:在返回结果前，拼接，使用RAC中bind方法做处理。
    // bind方法参数:需要传入一个返回值是RACStreamBindBlock的block参数
    // RACStreamBindBlock是一个block的类型，返回值是信号，参数（value,stop），因此参数的block返回值也是一个block。

    // RACStreamBindBlock:
    // 参数一(value):表示接收到信号的原始值，还没做处理
    // 参数二(*stop):用来控制绑定Block，如果*stop = yes,那么就会结束绑定。
    // 返回值：信号，做好处理，在通过这个信号返回出去，一般使用RACReturnSignal,需要手动导入头文件RACReturnSignal.h。

    // bind方法使用步骤:
    // 1.传入一个返回值RACStreamBindBlock的block。
    // 2.描述一个RACStreamBindBlock类型的bindBlock作为block的返回值。
    // 3.描述一个返回结果的信号，作为bindBlock的返回值。
    // 注意：在bindBlock中做信号结果的处理。

    // 底层实现:
    // 1.源信号调用bind,会重新创建一个绑定信号。
    // 2.当绑定信号被订阅，就会调用绑定信号中的didSubscribe，生成一个bindingBlock。
    // 3.当源信号有内容发出，就会把内容传递到bindingBlock处理，调用bindingBlock(value,stop)
    // 4.调用bindingBlock(value,stop)，会返回一个内容处理完成的信号（RACReturnSignal）。
    // 5.订阅RACReturnSignal，就会拿到绑定信号的订阅者，把处理完成的信号内容发送出来。

    // 注意:不同订阅者，保存不同的nextBlock，看源码的时候，一定要看清楚订阅者是哪个。
    // 这里需要手动导入#import <ReactiveCocoa/RACReturnSignal.h>，才能使用RACReturnSignal。
    [[_textField.rac_textSignal bind:^RACStreamBindBlock{

        // 什么时候调用:
        // block作用:表示绑定了一个信号.

        return ^RACStream *(id value, BOOL *stop){

            // 什么时候调用block:当信号有新的值发出，就会来到这个block。

            // block作用:做返回值的处理

            // 做好处理，通过信号返回出去.
            return [RACReturnSignal return:[NSString stringWithFormat:@"输出:%@",value]];
        };

    }] subscribeNext:^(id x) {

        NSLog(@"%@",x);

    }];
```

是一个非常重要的函数，在Rac Doc中被描述为‘basic primitives, particularly’，它是RACStream监测“值”和控制“运行状态”的基本方法，个人认为看注释文档不能理解它是干嘛的，而且bind英语“捆绑，绑定，强迫，约束”这几个意思也感觉对不上，我觉得叫“绑架”倒是更贴切一点。在-bind：之后，之前的RACStream就处于被“绑架”的状态，被绑架的RACStream每产生一个值，都要经过“绑架者”来决定：
1. 是否使这个RACStream结束（被绑架者是否还能继续活着）
2. 用什么新的RACStream来替换被绑架的RACStream，传出的结果也成了新RACStream产生的值（绑匪可以选择再抓一个人质放之前那个前面）

举个具体栗子，RACStream的 - take：方法，这个方法使一个RACStream只取前N次的值（有缩减）：

```objc
- (instancetype)take:(NSUInteger)count {
    Class class = self.class;

    return [[self bind:^{ // self被绑架
        __block NSUInteger taken = 0;

        return ^ id (id value, BOOL *stop) { // 这个block在被绑架的self每输出一个值得时候触发
            RACStream *result = class.empty;

            if (taken < count) result = [class return:value]; // 未达到N次时将原值原原本本的传递出去
            if (++taken >= count) *stop = YES; // 达到第N次值后干掉了被绑架的self

            return result; // 将被绑架的self替换为result
        };
    }]];
}
```

-concat: 和 -zipWith: 就是将两个RACStream连接起来的基本方法了：

`[A concat:B]`中A和B像`皇上`和`太子`的关系，A是皇上，B是太子。皇上健在的时候统治天下发号施令（value），太子就候着，不发号施令（value），当皇上挂了（completed），太子登基当皇上，此时发出的号令（value）是太子的。

`[C zipWith:D]`可以比喻成一对`平等恩爱的夫妻`，两个人是“绑在一起“的关系来组成一个家庭，决定一件事（value）时必须两个人都提出意见（当且仅当C和D同时都产生了值的时候，一个value才被输出，CD只有其中一个有值时会挂起等待另一个的值，所以输出都是一对值（RACTuple）），当夫妻只要一个人先挂了（completed）这个家庭（组合起来的RACStream）就宣布解散（也就是无法凑成一对输出时就终止）

## +empty

一个不返回值，立刻结束(Completed)的函数，意思是执行它之后除了立刻结束啥都不会发生，可以理解为RAC里面的nil。

## +return:

是一个直接返回给定值，然后立刻结束的函数，比如 f(x) = 213


# 类
## RACSignal
RACSiganl:信号类,一般表示将来有数据传递，只要有数据改变，信号内部接收到数据，就会马上发出数据。

注意：
* 信号类(`RACSiganl`)，只是表示当数据改变时，信号内部会发出数据，它本身不具备发送信号的能力，而是交给内部一个订阅者去发出。
* 默认一个信号都是冷信号，也就是值改变了，也不会触发，只有订阅了这个信号，这个信号才会变为热信号，值改变了才会触发。
* 如何订阅信号：调用信号RACSignal的subscribeNext就能订阅。

RACSiganl简单使用:

```objc
// RACSignal使用步骤：
    // 1.创建信号 + (RACSignal *)createSignal:(RACDisposable * (^)(id<RACSubscriber> subscriber))didSubscribe
    // 2.订阅信号,才会激活信号. - (RACDisposable *)subscribeNext:(void (^)(id x))nextBlock
    // 3.发送信号 - (void)sendNext:(id)value
    
    
    // RACSignal底层实现：
    // 1.创建信号，首先把didSubscribe保存到信号中，还不会触发。
    // 2.当信号被订阅，也就是调用signal的subscribeNext:nextBlock
    // 2.2 subscribeNext内部会创建订阅者subscriber，并且把nextBlock保存到subscriber中。
    // 2.1 subscribeNext内部会调用siganl的didSubscribe
    // 3.siganl的didSubscribe中调用[subscriber sendNext:@1];
    // 3.1 sendNext底层其实就是执行subscriber的nextBlock
    
    // 1.创建信号
    RACSignal *siganl = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        // block调用时刻：每当有订阅者订阅信号，就会调用block。
        // 2.发送信号
        [subscriber sendNext:@1];
        
        // 如果不再发送数据，最好发送信号完成，内部会自动调用[RACDisposable disposable]取消订阅信号。
        [subscriber sendCompleted];
 
        return [RACDisposable disposableWithBlock:^{
            // block调用时刻：当信号发送完成或者发送错误，就会自动执行这个block,取消订阅信号。
            // 执行完Block后，当前信号就不在被订阅了。
            NSLog(@"信号被销毁");
        }];
    }];
    
    // 3.订阅信号,才会激活信号.
    [siganl subscribeNext:^(id x) {
        // block调用时刻：每当有信号发出数据，就会调用block.
        NSLog(@"接收到数据:%@",x);
    }];
```

## RACSubscriber

`RACSubscriber`表示订阅者的意思，用于发送信号，这是一个协议，不是一个类，只要遵守这个协议，并且实现方法才能成为订阅者。通过create创建的信号，都有一个订阅者，帮助他发送数据。

## RACDisposable

`RACDisposable`用于取消订阅或者清理资源，当信号发送完成或者发送错误的时候，就会自动触发它。

  * 使用场景:不想监听某个信号时，可以通过它主动取消订阅信号(调用`disposable`方法)。


## RACSubject

`RACSubject`:信号提供者，自己可以充当信号，又能发送信号。

  * 使用场景:通常用来代替代理，有了它，就不必要定义代理了。

`RACSubject`替换代理

```objc
// 需求:
    // 1.给当前控制器添加一个按钮，modal到另一个控制器界面
    // 2.另一个控制器view中有个按钮，点击按钮，通知当前控制器
    
步骤一：在第二个控制器.h，添加一个RACSubject代替代理。
@interface TwoViewController : UIViewController

@property (nonatomic, strong) RACSubject *delegateSignal;

@end

步骤二：监听第二个控制器按钮点击
@implementation TwoViewController
- (IBAction)notice:(id)sender {
    // 通知第一个控制器，告诉它，按钮被点了
    
     // 通知代理
     // 判断代理信号是否有值
    if (self.delegateSignal) {
        // 有值，才需要通知
        [self.delegateSignal sendNext:nil];
    }
}
@end

步骤三：在第一个控制器中，监听跳转按钮，给第二个控制器的代理信号赋值，并且监听.
@implementation OneViewController 
- (IBAction)btnClick:(id)sender {
    
    // 创建第二个控制器
    TwoViewController *twoVc = [[TwoViewController alloc] init];
    
    // 设置代理信号
    twoVc.delegateSignal = [RACSubject subject];
    
    // 订阅代理信号
    [twoVc.delegateSignal subscribeNext:^(id x) {
        NSLog(@"点击了通知按钮");
    }];
    
    // 跳转到第二个控制器
    [self presentViewController:twoVc animated:YES completion:nil];
    
}
@end
```


## RACReplaySubject

`RACReplaySubject`:重复提供信号类，`RACSubject`的子类。

`RACReplaySubject`与`RACSubject`区别:

* RACReplaySubject可以先发送信号，再订阅信号，RACSubject就不可以。
    * 使用场景一:如果一个信号每被订阅一次，就需要把之前的值重复发送一遍，使用重复提供信号类。
    * 使用场景二:可以设置capacity数量来限制缓存的value的数量,即只缓充最新的几个值。

## RACSubject和RACReplaySubject简单使用:

```objc
// RACSubject使用步骤
    // 1.创建信号 [RACSubject subject]，跟RACSiganl不一样，创建信号时没有block。
    // 2.订阅信号 - (RACDisposable *)subscribeNext:(void (^)(id x))nextBlock
    // 3.发送信号 sendNext:(id)value

    // RACSubject:底层实现和RACSignal不一样。
    // 1.调用subscribeNext订阅信号，只是把订阅者保存起来，并且订阅者的nextBlock已经赋值了。
    // 2.调用sendNext发送信号，遍历刚刚保存的所有订阅者，一个一个调用订阅者的nextBlock。

    // 1.创建信号
    RACSubject *subject = [RACSubject subject];

    // 2.订阅信号
    [subject subscribeNext:^(id x) {
        // block调用时刻：当信号发出新值，就会调用.
        NSLog(@"第一个订阅者%@",x);
    }];
    [subject subscribeNext:^(id x) {
        // block调用时刻：当信号发出新值，就会调用.
        NSLog(@"第二个订阅者%@",x);
    }];

    // 3.发送信号
    [subject sendNext:@"1"];


    // RACReplaySubject使用步骤:
    // 1.创建信号 [RACSubject subject]，跟RACSiganl不一样，创建信号时没有block。
    // 2.可以先订阅信号，也可以先发送信号。
    // 2.1 订阅信号 - (RACDisposable *)subscribeNext:(void (^)(id x))nextBlock
    // 2.2 发送信号 sendNext:(id)value

    // RACReplaySubject:底层实现和RACSubject不一样。
    // 1.调用sendNext发送信号，把值保存起来，然后遍历刚刚保存的所有订阅者，一个一个调用订阅者的nextBlock。
    // 2.调用subscribeNext订阅信号，遍历保存的所有值，一个一个调用订阅者的nextBlock

    // 如果想当一个信号被订阅，就重复播放之前所有值，需要先发送信号，在订阅信号。
    // 也就是先保存值，再订阅值。

    // 1.创建信号
    RACReplaySubject *replaySubject = [RACReplaySubject subject];

    // 2.发送信号
    [replaySubject sendNext:@1];
    [replaySubject sendNext:@2];

    // 3.订阅信号
    [replaySubject subscribeNext:^(id x) {
        NSLog(@"第一个订阅者接收到的数据%@",x);
    }];

    // 订阅信号
    [replaySubject subscribeNext:^(id x) {
        NSLog(@"第二个订阅者接收到的数据%@",x);
    }];
```

## RACTupl

元组类,类似NSArray,用来包装值.

## RACSequence

RAC中的集合类，用于代替`NSArray`、`NSDictionary`,可以使用它来快速遍历数组和字典。

## RACSequence和RACTuple简单使用

字典转模型

```
// 1.遍历数组
    NSArray *numbers = @[@1,@2,@3,@4];
    
    // 这里其实是三步
    // 第一步: 把数组转换成集合RACSequence numbers.rac_sequence
    // 第二步: 把集合RACSequence转换RACSignal信号类,numbers.rac_sequence.signal
    // 第三步: 订阅信号，激活信号，会自动把集合中的所有值，遍历出来。
    [numbers.rac_sequence.signal subscribeNext:^(id x) {
       
        NSLog(@"%@",x);
    }];
    
    
    // 2.遍历字典,遍历出来的键值对会包装成RACTuple(元组对象)
    NSDictionary *dict = @{@"name":@"xmg",@"age":@18};
    [dict.rac_sequence.signal subscribeNext:^(RACTuple *x) {
       
        // 解包元组，会把元组的值，按顺序给参数里面的变量赋值
        RACTupleUnpack(NSString *key,NSString *value) = x;
        
        // 相当于以下写法
//        NSString *key = x[0];
//        NSString *value = x[1];
        
        NSLog(@"%@ %@",key,value);
        
    }];
    
    
    // 3.字典转模型
    // 3.1 OC写法
    NSString *filePath = [[NSBundle mainBundle] pathForResource:@"flags.plist" ofType:nil];
    
    NSArray *dictArr = [NSArray arrayWithContentsOfFile:filePath];
    
    NSMutableArray *items = [NSMutableArray array];
    
    for (NSDictionary *dict in dictArr) {
        FlagItem *item = [FlagItem flagWithDict:dict];
        [items addObject:item];
    }
    
    // 3.2 RAC写法
    NSString *filePath = [[NSBundle mainBundle] pathForResource:@"flags.plist" ofType:nil];
    
    NSArray *dictArr = [NSArray arrayWithContentsOfFile:filePath];

    NSMutableArray *flags = [NSMutableArray array];
    
    _flags = flags;
    
    // rac_sequence注意点：调用subscribeNext，并不会马上执行nextBlock，而是会等一会。
    [dictArr.rac_sequence.signal subscribeNext:^(id x) {
        // 运用RAC遍历字典，x：字典
        
        FlagItem *item = [FlagItem flagWithDict:x];
        
        [flags addObject:item];
        
    }];
    
    NSLog(@"%@",  NSStringFromCGRect([UIScreen mainScreen].bounds));
    
    
    // 3.3 RAC高级写法:
    NSString *filePath = [[NSBundle mainBundle] pathForResource:@"flags.plist" ofType:nil];
    
    NSArray *dictArr = [NSArray arrayWithContentsOfFile:filePath];
    // map:映射的意思，目的：把原始值value映射成一个新值
    // array: 把集合转换成数组
    // 底层实现：当信号被订阅，会遍历集合中的原始值，映射成新值，并且保存到新的数组里。
    NSArray *flags = [[dictArr.rac_sequence map:^id(id value) {
     
        return [FlagItem flagWithDict:value];
        
    }] array];
```

## RACCommand

RAC中用于处理事件的类，可以把事件如何处理,事件中的数据如何传递，包装到这个类中，他可以很方便的监控事件的执行过程

监听按钮点击，网络请求

```
// 一、RACCommand使用步骤:
    // 1.创建命令 initWithSignalBlock:(RACSignal * (^)(id input))signalBlock
    // 2.在signalBlock中，创建RACSignal，并且作为signalBlock的返回值
    // 3.执行命令 - (RACSignal *)execute:(id)input
    
    // 二、RACCommand使用注意:
    // 1.signalBlock必须要返回一个信号，不能传nil.
    // 2.如果不想要传递信号，直接创建空的信号[RACSignal empty];
    // 3.RACCommand中信号如果数据传递完，必须调用[subscriber sendCompleted]，这时命令才会执行完毕，否则永远处于执行中。
    // 4.RACCommand需要被强引用，否则接收不到RACCommand中的信号，因此RACCommand中的信号是延迟发送的。
    
    // 三、RACCommand设计思想：内部signalBlock为什么要返回一个信号，这个信号有什么用。
    // 1.在RAC开发中，通常会把网络请求封装到RACCommand，直接执行某个RACCommand就能发送请求。
    // 2.当RACCommand内部请求到数据的时候，需要把请求的数据传递给外界，这时候就需要通过signalBlock返回的信号传递了。
    
    // 四、如何拿到RACCommand中返回信号发出的数据。
    // 1.RACCommand有个执行信号源executionSignals，这个是signal of signals(信号的信号),意思是信号发出的数据是信号，不是普通的类型。
    // 2.订阅executionSignals就能拿到RACCommand中返回的信号，然后订阅signalBlock返回的信号，就能获取发出的值。
    
    // 五、监听当前命令是否正在执行executing
    
    // 六、使用场景,监听按钮点击，网络请求


    // 1.创建命令
    RACCommand *command = [[RACCommand alloc] initWithSignalBlock:^RACSignal *(id input) {
        NSLog(@"执行命令");
        // 创建空信号,必须返回信号
        //        return [RACSignal empty];
        // 2.创建信号,用来传递数据
        return [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
            [subscriber sendNext:@"请求数据"];
            // 注意：数据传递完，最好调用sendCompleted，这时命令才执行完毕。
            [subscriber sendCompleted];
            return nil;
        }];
    }];
    
    // 强引用命令，不要被销毁，否则接收不到数据
    _conmmand = command;
    
    // 3.订阅RACCommand中的信号
    [command.executionSignals subscribeNext:^(id x) {
        [x subscribeNext:^(id x) {
            NSLog(@"%@",x);
        }];
    }];
    
    // RAC高级用法
    // switchToLatest:用于signal of signals，获取signal of signals发出的最新信号,也就是可以直接拿到RACCommand中的信号
    [command.executionSignals.switchToLatest subscribeNext:^(id x) {
        NSLog(@"%@",x);
    }];
    
    // 4.监听命令是否执行完毕,默认会来一次，可以直接跳过，skip表示跳过第一次信号。
    [[command.executing skip:1] subscribeNext:^(id x) {
        
        if ([x boolValue] == YES) {
            // 正在执行
            NSLog(@"正在执行");
            
        }else{
            // 执行完成
            NSLog(@"执行完成");
        }
        
    }];
   // 5.执行命令
    [self.conmmand execute:@1];
```

## RACMulticastConnection

用于当一个信号，被多次订阅时，为了保证创建信号时，避免多次调用创建信号中的block，造成副作用，可以使用这个类处理。

>使用注意:RACMulticastConnection通过RACSignal的-publish或者-muticast:方法创建.

```
// RACMulticastConnection使用步骤:
    // 1.创建信号 + (RACSignal *)createSignal:(RACDisposable * (^)(id<RACSubscriber> subscriber))didSubscribe
    // 2.创建连接 RACMulticastConnection *connect = [signal publish];
    // 3.订阅信号,注意：订阅的不在是之前的信号，而是连接的信号。 [connect.signal subscribeNext:nextBlock]
    // 4.连接 [connect connect]
    
    // RACMulticastConnection底层原理:
    // 1.创建connect，connect.sourceSignal -> RACSignal(原始信号)  connect.signal -> RACSubject
    // 2.订阅connect.signal，会调用RACSubject的subscribeNext，创建订阅者，而且把订阅者保存起来，不会执行block。
    // 3.[connect connect]内部会订阅RACSignal(原始信号)，并且订阅者是RACSubject
    // 3.1.订阅原始信号，就会调用原始信号中的didSubscribe
    // 3.2 didSubscribe，拿到订阅者调用sendNext，其实是调用RACSubject的sendNext
    // 4.RACSubject的sendNext,会遍历RACSubject所有订阅者发送信号。
    // 4.1 因为刚刚第二步，都是在订阅RACSubject，因此会拿到第二步所有的订阅者，调用他们的nextBlock

    
    // 需求：假设在一个信号中发送请求，每次订阅一次都会发送请求，这样就会导致多次请求。
    // 解决：使用RACMulticastConnection就能解决.
    
    // 1.创建请求信号
   RACSignal *signal = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        
        
        NSLog(@"发送请求");
     
        return nil;
    }];
    // 2.订阅信号
    [signal subscribeNext:^(id x) {
       
        NSLog(@"接收数据");
        
    }];
    // 2.订阅信号
    [signal subscribeNext:^(id x) {
        
        NSLog(@"接收数据");
        
    }];
    
    // 3.运行结果，会执行两遍发送请求，也就是每次订阅都会发送一次请求
    
    
    // RACMulticastConnection:解决重复请求问题
    // 1.创建信号
    RACSignal *signal = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        
        
        NSLog(@"发送请求");
        [subscriber sendNext:@1];
        
        return nil;
    }];
    
    // 2.创建连接
    RACMulticastConnection *connect = [signal publish];
    
    // 3.订阅信号，
    // 注意：订阅信号，也不能激活信号，只是保存订阅者到数组，必须通过连接,当调用连接，就会一次性调用所有订阅者的sendNext:
    [connect.signal subscribeNext:^(id x) {
       
        NSLog(@"订阅者一信号");
        
    }];
    
    [connect.signal subscribeNext:^(id x) {
        
        NSLog(@"订阅者二信号");
        
    }];
    
    // 4.连接,激活信号
    [connect connect];
```

## RACScheduler

RAC中的队列，用GCD封装的。

## RACUnit

表⽰stream不包含有意义的值,也就是看到这个，可以直接理解为nil.

## RACEvent

把数据包装成信号事件(signal event)。它主要通过RACSignal的-materialize来使用，然并卵。


## 创建信号

使用`RACSignal`的`createSignal:`方法创建信号。描述这个信号的block是这个方法唯一的入参。当这个信号有订阅者的时候，block中的代码就会执行

```objc
-(RACSignal *)signInSignal {
  return [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
    [self.signInService
     signInWithUsername:self.usernameTextField.text
     password:self.passwordTextField.text
     complete:^(BOOL success) {
       [subscriber sendNext:@(success)];
       [subscriber sendCompleted];
     }];
    return nil;
  }];
}
```

这个block传入了一个遵守`RACSubscriber`协议的`subscriber`实例，实例中包含发送事件的方法；你可以发送任意数量的事件到下一环节，也可以通过`error`或者`complete`事件终止信号。在这个例子中，`subscriber`实例发送了表示登录结果的`next`事件，紧跟一个`complete`事件。

这个block的返回类型是一个`RACDisposable`对象，这让你可以处理一些可能需要的清除工作，比如当一个订阅被取消或废弃的时候。由于这个信号并不需要作清除处理，所以直接结果返回nil。

```objc
- (RACSignal *)requestAccessToTwitterSignal {
 
  // 1 - define an error
  NSError *accessError = [NSError errorWithDomain:RWTwitterInstantDomain
                                             code:RWTwitterInstantErrorAccessDenied
                                         userInfo:nil];
 
  // 2 - create the signal
  @weakify(self)
  return [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
    // 3 - request access to twitter
    @strongify(self)
    [self.accountStore
       requestAccessToAccountsWithType:self.twitterAccountType
         options:nil
      completion:^(BOOL granted, NSError *error) {
          // 4 - handle the response
          if (!granted) {
            [subscriber sendError:accessError];
          } else {
            [subscriber sendNext:nil];
            [subscriber sendCompleted];
          }
        }];
    return nil;
  }];
}
```

该方法做了以下操作：

定义了一个错误，当用户连接遭拒时发送。
像第一篇文章所说，类方法createSignal返回了一个RACSignal的实例。
通过账户库链接到推特。此时，用户会看到是否允许app连接到他们推特账号的提示。
当用户同意或拒绝了连接，信号事件就会发送。如果用户同意连接，一个next事件和紧接一个completed事件就会被发送。如果用户拒绝了连接一个error事件就会被发送。


# 应用
## 代替KVO

`rac_valuesAndChangesForKeyPath：`用于监听某个对象的属性改变。

```objc
@weakify(self)
    [RACObserve(self.person, name)
        subscribeNext:^(id x) {
             @strongify(self)
            self.nameLabel.text = x;
    }];
```

## 代替代理

`rac_signalForSelector：`用于替代代理。

## 监听事件

`rac_signalForControlEvents：`用于监听某个事件。

## 代替通知

`rac_addObserverForName:`用于监听某个通知。

## 监听文本框文字改变:

`rac_textSignal:`只要文本框发出改变就会发出这个信号。

## 处理当界面有多次请求时，需要都获取到数据时，才能展示界面

`rac_liftSelector:withSignalsFromArray:Signals:`当传入的`Signals`(信号数组)，每一个signal都至少sendNext过一次，就会去触发第一个`selector`参数的方法。

使用注意：几个信号，参数一的方法就几个参数，每个参数对应信号发出的数据。

```objc
// 1.代替代理
    // 需求：自定义redView,监听红色view中按钮点击
    // 之前都是需要通过代理监听，给红色View添加一个代理属性，点击按钮的时候，通知代理做事情
    // rac_signalForSelector:把调用某个对象的方法的信息转换成信号，就要调用这个方法，就会发送信号。
    // 这里表示只要redV调用btnClick:,就会发出信号，订阅就好了。
    [[redV rac_signalForSelector:@selector(btnClick:)] subscribeNext:^(id x) {
        NSLog(@"点击红色按钮");
    }];

    // 2.KVO
    // 把监听redV的center属性改变转换成信号，只要值改变就会发送信号
    // observer:可以传入nil
    [[redV rac_valuesAndChangesForKeyPath:@"center" options:NSKeyValueObservingOptionNew observer:nil] subscribeNext:^(id x) {

        NSLog(@"%@",x);

    }];

    // 3.监听事件
    // 把按钮点击事件转换为信号，点击按钮，就会发送信号
    [[self.btn rac_signalForControlEvents:UIControlEventTouchUpInside] subscribeNext:^(id x) {

        NSLog(@"按钮被点击了");
    }];

    // 4.代替通知
    // 把监听到的通知转换信号
    [[[NSNotificationCenter defaultCenter] rac_addObserverForName:UIKeyboardWillShowNotification object:nil] subscribeNext:^(id x) {
        NSLog(@"键盘弹出");
    }];

    // 5.监听文本框的文字改变
   [_textField.rac_textSignal subscribeNext:^(id x) {

       NSLog(@"文字改变了%@",x);
   }];
   
   // 6.处理多个请求，都返回结果的时候，统一做处理.
    RACSignal *request1 = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        
        // 发送请求1
        [subscriber sendNext:@"发送请求1"];
        return nil;
    }];
    
    RACSignal *request2 = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        // 发送请求2
        [subscriber sendNext:@"发送请求2"];
        return nil;
    }];
    
    // 使用注意：几个信号，参数一的方法就几个参数，每个参数对应信号发出的数据。
    [self rac_liftSelector:@selector(updateUIWithR1:r2:) withSignalsFromArray:@[request1,request2]];
    
    
}
// 更新UI
- (void)updateUIWithR1:(id)data r2:(id)data1
{
    NSLog(@"更新UI%@  %@",data,data1);
}
```

## 信号绑定
RAC是通过Category对UITextField进行绑定的，我们来看一下代码：
```objc
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

```objc
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

```objc
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

```objc
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


## 输入框的双向绑定
* 无论是通过键盘或者code赋值来改变textField.text的值的时候，我们都要让绑定的string对应的改变，这才是我们要的双向绑定
    
| RACChannelTo | rac_newTextChannel |
| --- | --- |
| 当通过code改变self.textField.text的值的时候才会把这个值发送出去 | 当通过键盘改变self.textField. text的值的时候才会把这个值发送出去 |

```objc
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

```objc
string -> textField.text
键盘改变textField.text -> string
code赋值textField.text -> string
满足上面三点就算完全的双向绑定
```

## 异步加载图片

```objc
-(RACSignal *)signalForLoadingImage:(NSString *)imageUrl {
 
  RACScheduler *scheduler = [RACScheduler
                         schedulerWithPriority:RACSchedulerPriorityBackground];
 
  return [[RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
    NSData *data = [NSData dataWithContentsOfURL:[NSURL URLWithString:imageUrl]];
    UIImage *image = [UIImage imageWithData:data];
    [subscriber sendNext:image];
    [subscriber sendCompleted];
    return nil;
  }] subscribeOn:scheduler];
}
```

由于你希望这个信号不在主线程中执行，上面的方法先获取了一个后台调度器。然后创建一个信号，该信号在有订阅者时下载图像数据并生成UIImage。最后一步就是使用`subscribeOn:`，以保证信号在提供的调度器中执行。


# 常用宏

## RAC(TARGET, [KEYPATH, [NIL_VALUE]])

用于给某个对象的某个属性绑定

```
// 只要文本框文字改变，就会修改label的文字
RAC(self.outputLabel, text) = self.inputTextField.rac_textSignal;
RAC(self.outputLabel, text, @"收到nil时就显示我") = self.inputTextField.rac_textSignal;
```

## RACObserve(self, name)

监听某个对象的某个属性,返回的是信号。

```
[RACObserve(self.view, center) subscribeNext:^(id x) {  
    NSLog(@"%@",x);
}];
```

## RACTuplePack

把数据包装成RACTuple（元组类）

```
// 把参数中的数据包装成元组
RACTuple *tuple = RACTuplePack(@10,@20);
```

## RACTupleUnpack

把RACTuple（元组类）解包成对应的数据

```
// 把参数中的数据包装成元组
RACTuple *tuple = RACTuplePack(@"xmg",@20);
// 解包元组，会把元组的值，按顺序给参数里面的变量赋值
// name = @"xmg" age = @20
RACTupleUnpack(NSString *name,NSNumber *age) = tuple;
```

这个宏是最常用的，`RAC()`总是出现在等号左边，等号右边是一个`RACSignal`，表示的意义是将一个对象的一个`属性`和一个`signal`绑定，`signal`每产生一个value（id类型），都会自动执行：

```objc
[TARGET setValue:value ?: NIL_VALUE forKeyPath:KEYPATH];
```

数字值会升级为NSNumber *，当setValue:forKeyPath时会自动降级成基本类型（int, float ,BOOL等），所以RAC绑定一个基本类型的值是没有问题的

```objc
· RACObserve(TARGET, KEYPATH)
```

作用是观察TARGET的KEYPATH属性，相当于KVO，产生一个`RACSignal`

最常用的使用，和RAC宏绑定属性：

```objc
RAC(self.outputLabel, text) = RACObserve(self.model, name);
```



# Tips

## 避免引用循环

`@weakify`和`@strongify`是定义在`Extended Objective-C`库的宏，这也已经包含在`ReactiveCocoa`框架中。`@weakify`宏创建了弱应用的影子变量（`shadow variables`）（如果你需要多个弱引用，你可以传入多个变量），`@strongify`宏则使用先前传到@weakify的变量创建强引用。

>注意：如果你对@weakify和@strongify的具体操作感到好奇，你可以在Xcode中选择Product -> Perform Action -> Preprocess “RWSearchForViewController”。这会对视图控制器进行预处理，展开所有的宏让你看到最终的输出。

## Side Effects

RACSignal在被subscribe的时候可能会产生副作用，先举个官方的例子：

```objc
__block int aNumber = 0;

// Signal that will have the side effect of incrementing `aNumber` block
// variable for each subscription before sending it.
RACSignal *aSignal = [RACSignal createSignal:^ RACDisposable * (id<RACSubscriber> subscriber) {
    aNumber++;
    [subscriber sendNext:@(aNumber)];
    [subscriber sendCompleted];
    return nil;
}];
// This will print "subscriber one: 1"
[aSignal subscribeNext:^(id x) {
    NSLog(@"subscriber one: %@", x);
}];
// This will print "subscriber two: 2"
[aSignal subscribeNext:^(id x) {
    NSLog(@"subscriber two: %@", x);
}];
```

上面的signal在作用域外部引用了一个int变量，同时在signal的运算过程中作为next事件的值返回，这就造成了所谓的副作用，因为第二个订阅者的订阅而影响了输出值。

我的理解来看，这个事儿做的就不太地道，一个正经的函数式编程中的函数是不应该因为进行了运算而导致后面运算的值不统一的。但对于实际应用的情况来看也到无可厚非，比如用户点击了“登录”按钮，编程时把登录这个业务写为一个login的RACSignal，当然，第一次调用登录和再点一次第二次调用登录的结果肯定不一样了。所以说RAC式编程减少了大部分对临时状态值的定义，但不是全部哦。

怎么办呢？我觉得最好的办法就是“约定”，RAC design guide里面介绍了对于一个signal的命名法则：

>Hot signals without side effects 最好使用property，如“textChanged”，不太理解什么情况用到这个，权当做一个静态的属性来看就行。
Cold signals without side effects 使用名词类型的方法名，如“-currentText”，“currentModels”，同时表明了返回值是个啥（这个尤其得注意，RACSignal的next值是id类型，所以全得是靠约定才知道具体返回类型）
Signals with side effects 这种就是像login一样有副作用的了，推荐使用动词类型的方法名，用对动词基本就能知道是不是有副作用了，比如“-loginSignal”和“-saveToFile”大概就知道前面一个很可能有副作用，后面一个多存几次文件应该没副作用

当然，也可以multicast一个event，使得某些特殊的情况来共享一个副作用，后面再具体讲，先一个官方的简单的栗子：

```objc
// This signal starts a new request on each subscription.
RACSignal *networkRequest = [RACSignal createSignal:^(id<RACSubscriber> subscriber) {
    AFHTTPRequestOperation *operation = [client
        HTTPRequestOperationWithRequest:request
        success:^(AFHTTPRequestOperation *operation, id response) {
            [subscriber sendNext:response];
            [subscriber sendCompleted];
        }
        failure:^(AFHTTPRequestOperation *operation, NSError *error) {
            [subscriber sendError:error];
        }];

    [client enqueueHTTPRequestOperation:operation];
    return [RACDisposable disposableWithBlock:^{
        [operation cancel];
    }];
}];

// Starts a single request, no matter how many subscriptions `connection.signal`
// gets. This is equivalent to the -replay operator, or similar to
// +startEagerlyWithScheduler:block:.
RACMulticastConnection *connection = [networkRequest multicast:[RACReplaySubject subject]];
[connection connect];

[connection.signal subscribeNext:^(id response) {
    NSLog(@"subscriber one: %@", response);
}];

[connection.signal subscribeNext:^(id response) {
    NSLog(@"subscriber two: %@", response);
}];
```

当地一个订阅者subscribeNext的时候触发了AFNetworkingOperation的创建和执行，开始网络请求，此时又来了个订阅者订阅这个Signal，按理说这个网络请求会被“副作用”，重新发一遍，但做了上面的处理之后，这两个订阅者接收到了同样的一个请求的内容。


# MVVM

* 模型(M):保存视图数据。
* 视图+控制器(V):展示内容 + 如何展示
* 视图模型(VM):处理展示的业务逻辑，包括按钮的点击，数据的请求和解析等等。



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
* [最快让你上手ReactiveCocoa之基础篇](https://www.jianshu.com/p/87ef6720a096)
* [最快让你上手ReactiveCocoa之进阶篇](https://www.jianshu.com/p/e10e5ca413b7)
* [Reactive Cocoa Tutorial \[1\] = 神奇的Macros](http://blog.sunnyxx.com/2014/03/06/rac_1_macros/)
* [ReactiveCocoa v2.5 源码解析之架构总览](http://blog.leichunfeng.com/blog/2015/12/25/reactivecocoa-v2-dot-5-yuan-ma-jie-xi-zhi-jia-gou-zong-lan/)
* [ReactiveCocoa学习笔记](http://yulingtianxia.com/blog/2014/07/29/reactivecocoa/)
* [iOS开发下的函数响应式编程](http://williamzang.com/blog/2016/06/27/ios-kai-fa-xia-de-han-shu-xiang-ying-shi-bian-cheng/)

[ReactiveObjC]:https://github.com/ReactiveCocoa/ReactiveObjC
[ReactiveCocoa]:https://github.com/ReactiveCocoa/ReactiveCocoa
[FrameworkOverview]:https://github.com/ReactiveCocoa/ReactiveObjC/blob/master/Documentation/FrameworkOverview.md
[Design Guidelines]:https://github.com/ReactiveCocoa/ReactiveObjC/blob/master/Documentation/DesignGuidelines.md
[signals]:https://github.com/ReactiveCocoa/ReactiveObjC/blob/master/Documentation/FrameworkOverview.md#signals
[sequences]:https://github.com/ReactiveCocoa/ReactiveObjC/blob/master/Documentation/FrameworkOverview.md#sequences
[streams]:https://github.com/ReactiveCocoa/ReactiveObjC/blob/master/Documentation/FrameworkOverview.md#streams
[MemoryManagement]:https://github.com/ReactiveCocoa/ReactiveObjC/blob/master/Documentation/MemoryManagement.md
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
[rac_doNext]:https://github.com/lightank/iOS_notes/blob/master/Resource/rac/rac_doNext.png
