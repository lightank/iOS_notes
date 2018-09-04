# ReactiveCocoa学习
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

为了支持这种特性，`ReactiveObjc`维系保持了它自己的全局信号集（`global set of signals`）。如果信号有一个或多个订阅者的话，信号就会被激活。如果所有的订阅者都给移除了，该信号就可以被回收。想知道更多关于`ReactiveObjc`内管理这个过程的内容，你可以浏览 [Memory Management][MemoryManagement]文档（译注：文档已失效）。

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

## combineLatest

```objc
RACSignal *signal1 = [@[ @1, @2 ] rac_sequence].signal;
  RACSignal *signal2 = [@[ @3, @4 ] rac_sequence].signal;

  [[signal1 combineLatestWith:signal2] subscribeNext:^(RACTuple *value) {
    NSLog(@"%@", value);
  }];
```

![rac_combineLatest][rac_combineLatest]

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

## doNext

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

对信号进行映射

## RAC宏

RAC宏将一个信号的输出和一个对象的属性绑定起来。宏接受两个参数，第一个是包含改变属性的对象，第二个为属性的名字。每次当信号发出一个新的事件，事件的值就会传递给绑定的属性。

```objc
RAC(self.passwordTextField, backgroundColor) =
  [validPasswordSignal
    map:^id(NSNumber *passwordValid) {
      return [passwordValid boolValue] ? [UIColor clearColor] : [UIColor yellowColor];
    }];
```

## RACTuplePack RACTupleUnpack

```objc
RACTuple *tuple = RACTuplePack(address, amount);
RACTupleUnpack(NSString *address, NSString *amount) = pair;
```

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


# 常见宏

## RAC(TARGET, [KEYPATH, [NIL_VALUE]])

用于给某个对象的某个属性绑定

```
// 只要文本框文字改变，就会修改label的文字
RAC(self.labelView,text) = _textField.rac_textSignal;
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


# Tips

## 避免引用循环

`@weakify`和`@strongify`是定义在`Extended Objective-C`库的宏，这也已经包含在`ReactiveCocoa`框架中。`@weakify`宏创建了弱应用的影子变量（`shadow variables`）（如果你需要多个弱引用，你可以传入多个变量），`@strongify`宏则使用先前传到@weakify的变量创建强引用。

>注意：如果你对@weakify和@strongify的具体操作感到好奇，你可以在Xcode中选择Product -> Perform Action -> Preprocess “RWSearchForViewController”。这会对视图控制器进行预处理，展开所有的宏让你看到最终的输出。



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
