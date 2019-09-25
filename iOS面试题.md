# runtime 
1. 应用
2. 能用来干嘛

# runloop
1. 应用
2. 能用来干嘛

# 性能优化：启动优化、网络优化、UI优化等
## 启动优化
>
>1. App启动过程
    * 解析Info.plist
        * 加载相关信息，例如如闪屏
        * 沙箱建立、权限检查
    * Mach-O加载
        * 如果是胖二进制文件，寻找合适当前CPU类别的部分
        * 加载所有依赖的Mach-O文件（递归调用Mach-O加载的方法）
        * 定位内部、外部指针引用，例如字符串、函数等
        * 执行声明为__attribute__((constructor))的C函数
        * 加载类扩展（Category）中的方法
        * C++静态对象加载、调用ObjC的 +load 函数
    * 程序执行
        * 调用main()
        * 调用UIApplicationMain()
        * 调用applicationWillFinishLaunching
>t(App 总启动时间) = t1( main()之前的加载时间 ) + t2( main()之后的加载时间 )。
t1 = 系统的 dylib (动态链接库)和 App 可执行文件的加载时间；
t2 = main()函数执行之后到AppDelegate类中的applicationDidFinishLaunching:withOptions:方法执行结束前这段时间。

main()函数之前耗时的影响因素

* 动态库加载越多，启动越慢。
* ObjC类越多，启动越慢
* C的constructor函数越多，启动越慢
* C++静态对象越多，启动越慢
* ObjC的+load越多，启动越慢

>main()函数之前
* 重新梳理架构，减少动态库、ObjC类的数目，通过合并功能类似的类和扩展（Category）来减少Category的数目
* 定期扫描不再使用的动态库、类、函数，例如每两个迭代一次，关于清理项目中没用到的类，可以借助AppCode代码检查工具
* 尽量不要用C++虚函数(创建虚函数表有开销)
* 用dispatchonce()代替所有的__attribute__((constructor))函数、C++静态对象初始化、ObjC的+load
* 压缩资源图片

>main()函数之后
* 尽量使用纯代码编写，减少xib的使用
* 启动阶段的网络请求，是否都放到异步请求
* 一些耗时的操作放到后面去执行，或异步执行等
* 优化rootViewController加载，减少或延后加载不需要的视图及逻辑
* 网络请求的优化。。。
* 数据本地缓存，先布局视图，加载本地缓存，再加载网络资源

>显式初始化
* 使用 +initialize 来替代 +load
* 不要使用 __atribute__((constructor)) 将方法显式标记为初始化器，而是让初始化方法调用时才执行。比如使用 dispatch_once(),pthread_once() 或 std::once()。也就是在第一次使用时才初始化，推迟了一部分工作耗时。

>二进制重排

## 性能优化
### 卡顿监控
卡顿监控的实现一般有两种方案：
1. 主线程卡顿监控。通过子线程监测主线程的 runLoop，判断两个状态区域之间的耗时是否达到一定阈值。具体原理和实现，[这篇文章](http://www.tanhao.me/code/151113.html/)介绍得比较详细。
2. FPS监控。要保持流畅的UI交互，App 刷新率应该当努力保持在 60fps。监控实现原理比较简单，通过记录两次刷新时间间隔，就可以计算出当前的 FPS。

在实际应用过程我们发现，无论是主线程监控，还是 FPS 监控，抖动都比较大。因此，微信团队提出了一套综合的判断方法，结合了主线程监控、FPS监控，以及CPU使用率等指标，作为判断卡顿的标准。

### 解决方案

1. 优化业务流程
性能优化看似高深，真正落到实处才会发现，最大的坑往往都隐藏在于业务不断累积和频繁变更之处。优化业务流程就是在满足需求的同时，提出更加高效优雅的解决方案，从根本上解决问题。从实践来看，这种方法解决问题是最彻底的，但通常也是难度最大的。

2. 合理的线程分配
由于 GCD 实在太方便了，如果不加控制，大部分需要抛到子线程操作都会被直接加到 global 队列，这样会导致两个问题，1.开的子线程越来越多，线程的开销逐渐明显，因为开启线程需要占用一定的内存空间（默认的情况下，主线程占1M,子线程占用512KB）。2.多线程情况下，网络回调的时序问题，导致数据处理错乱，而且不容易发现。为此，我们项目定了一些基本原则。
    * UI 操作和 DataSource 的操作一定在主线程。
    * DB 操作、日志记录、网络回调都在各自的固定线程。
    * 不同业务，可以通过创建队列保证数据一致性。例如，想法列表的数据加载、书籍章节下载、书架加载等。

    合理的线程分配，最终目的就是保证主线程尽量少的处理非UI操作，同时控制整个App的子线程数量在合理的范围内。

3. 预处理和延时加载

    * 预处理，是将初次显示需要耗费大量线程时间的操作，提前放到后台线程进行计算，再将结果数据拿来显示。
    * 延时加载，是指首先加载当前必须的可视内容，在稍后一段时间内或特定事件时，再触发其他内容的加载。这种方式可以很有效的提升界面绘制速度，使体验更加流畅。（UITableView 就是最典型的例子）

    这两种方法都是在资源比较紧张的情况下，优先处理马上要用到的数据，同时尽可能提前加载即将要用到的数据。在微信读书中阅读的排版是优先级最高的，所在在阅读过程中会预处理下一页、下一章的排版，同时可能会延时加载阅读相关的其它数据（如想法、划线、书签等）。

4. 缓存

    cache可能是所有性能优化中最常用的手段，但也是我们极不推荐的手段。cache建立的成本低，见效快，但是带来维护的成本却很高。如果一定要用，也请谨慎使用，并注意以下几点：

    * 并发访问 cache 时，数据一致性问题。
    * cache 线程安全问题，防止一边修改一边遍历的 crash。
    * cache 查找时性能问题。
    * cache 的释放与重建，避免占用空间无限扩大，同时释放的粒度也要依实际需求而定。

5. 使用正确的API

    使用正确的 API，是指在满足业务的同时，能够选择性能更优的API。

    选择合适的容器;
    * 了解 imageNamed: 与 imageWithContentsOfFile:的差异(imageNamed: 适用于会重复加载的小图片，因为系统会自动缓存加载的图片，imageWithContentsOfFile: 仅加载图片)
    * 缓存 NSDateFormatter 的结果。
    * 寻找 (NSDate *)dateFromString:(NSString )string 的替换品。
    * 不要随意使用 NSLog().
    * 当试图获取磁盘中一个文件的属性信息时，使用 [NSFileManager attributesOfItemAtPath:error:] 会浪费大量时间读取可能根本不需要的附加属性。这时可以使用 stat 代替 NSFileManager，直接获取文件属性：
    
### 如何预防性能问题
1. 内存泄露检测工具
2. FPS/SQL性能监测工具条
3. UI / DataSource主线程检测工具。保证所有的UI的操作和 DataSource 操作一定是在主线程进行，同样是由tower同学贡献。实现原理是通过 hook UIView 的 -setNeedsLayout，-setNeedsDisplay，-setNeedsDisplayInRect 三个方法，确保它们都是在主线程执行。

优化滑动性能主要涉及三个方面：
避免图层混合

* 确保控件的opaque属性设置为true，确保backgroundColor和父视图颜色一致且不透明。
* 如无特殊需要，不要设置低于1的alpha值。
* 确保UIImage没有alpha通道。

避免临时转换
* 确保图片大小和frame一致，不要在滑动时缩放图片。
* 确保图片颜色格式被GPU支持，避免劳烦CPU转换。

慎用离屏渲染
* 绝大多数时候离屏渲染会影响性能。
* 重写drawRect方法，设置圆角、阴影、模糊效果，光栅化都会导致离屏渲染。
* 设置阴影效果是加上阴影路径。eg：`imgView.layer.shadowPath = UIBezierPath(rect: imgView.bounds).CGPath`
* 滑动时若需要圆角效果，开启光栅化。
    
    ```
    // 设置圆角
    label.layer.masksToBounds = true
    label.layer.cornerRadius = 8
    label.layer.shouldRasterize = true
    label.layer.rasterizationScale = layer.contentsScale
    ```

优化小细节：
* 将耗时操作（如文件IO）放到子线程，处理完后再异步到主线程
    * 解析网络回包数据
    * 异步排版
    * 将耗时操作放到工作线程异步执行占了优化工作的大头，在这个过程中，要注意多线程问题，比如线程安全问题、死锁、野指针等，比如容器类的读写操作，最常用的NSMutableArray, NSMutableDictionary等, 这类集合在调用读写方法时并不是线程安全，简单地在里面进行加锁操作是可以保证线程安全，不过也可能会导致其他耗时问题。
* 耗时函数优化
    * 解决方法：优化调用耗时，或者将耗时操作放到别的地方去
* 减少`- (void)scrollViewDidScroll:(UIScrollView *)scrollView`这个函数里面的耗时操作：
    * 这个方法在任何方式触发 contentOffset 变化的时候都会被调用（包括用户拖动，减速过程，直接通过代码设置等），可以用于监控 contentOffset 的变化，并根据当前的 contentOffset 对其他 view 做出随动调整。但是这个方法在滚动的时候每秒调用上百次，如果在里面加入耗时操作就可能对掉帧率造成很大影响。
    * 解决方法：优化调用耗时，或者将耗时操作放到别的地方去
* 缓存
    * 在业务上，我们会读取一些设置项来展示或者进行不同的功能，这些选项的即时读取可能是非常耗时的（尤其是涉及非线程安全容器的读取，里面往往是利用了互斥锁或者信号量等机制保证线程安全，耗时就更加严重），我们可以使用静态变量和dispatch_once来保存起来，避免每次都去要读取一遍。
* 减少不必要的操作
    * 为了方便回溯用户的操作行为，我们会在App里面加上很多log，一般log都涉及IO操作，不是必要的log我们要减少，尽量只在关键点打log。
* 懒加载view
    * 不要在cell里面嵌套太多的view，这会很影响滑动的流畅感，而且更多的view也需要花费更多的CPU跟内存。假如由于view太多而导致了滑动不流畅，那就不要在一次就把所有的view都创建出来，把部分view放到需要显示cell的时候再去创建。
* 利用主线程不同的runloop
    * 我们可以利用不同的runloop来优化这个耗时问题。主线程的 RunLoop 里有两个预置的 Mode：kCFRunLoopDefaultMode 和 UITrackingRunLoopMode。这两个 Mode 都已经被标记为"Common"属性。DefaultMode 是 App 平时所处的状态，TrackingRunLoopMode 是追踪 ScrollView 滑动时的状态。当我们初始化挂件并加到 DefaultMode 时，这个事件 会得到重复回调，但此时滑动一个TableView时，RunLoop 会将 mode 切换为 TrackingRunLoopMode，这时 初始化挂件函数就不会被回调，因而也不会影响到滑动操作。
    ```
    - (void)performSelectorOnMainThread:(SEL)aSelector withObject:(nullable id)arg waitUntilDone:(BOOL)wait modes:(nullable NSArray<NSString *> *)array
    ```


## 网络优化
http 0.9、1.1、2.0

## UI优化



## 动画、pop、hero
## 多线程
## 刘海屏适配

# objc_msgSend:[深入解构objc_msgSend函数的实现](https://www.jianshu.com/p/df6629ec9a25)
熟悉OC语言的Runtime(运行时)机制以及对象方法调用机制的开发者都知道，所有OC方法调用在编译时都会转化为对C函数objc_msgSend的调用。

objc_msgSend函数是所有OC方法调用的核心引擎，它负责查找真实的类或者对象方法的实现，并去执行这些方法函数。因调用频率是如此之高，所以要求其内部实现近可能达到最高的性能。这个函数的内部代码实现是用汇编语言来编写的，并且其中并没有涉及任何需要线程同步和锁相关的代码。将函数的实现反汇编成C语言的伪代码：

```
//下面的结构体中只列出objc_msgSend函数内部访问用到的那些数据结构和成员。

/*
其实SEL类型就是一个字符串指针类型，所描述的就是方法字符串指针
*/
typedef char * SEL;

/*
IMP类型就是所有OC方法的函数原型类型。
*/
typedef id (*IMP)(id self, SEL _cmd, ...); 


/*
  方法名和方法实现桶结构体
*/
struct bucket_t  {
    SEL  key;       //方法名称
    IMP imp;       //方法的实现，imp是一个函数指针类型
};

/*
   用于加快方法执行的缓存结构体。这个结构体其实就是一个基于开地址冲突解决法的哈希桶。
*/
struct cache_t {
    struct bucket_t *buckets;    //缓存方法的哈希桶数组指针，桶的数量 = mask + 1
    int  mask;        //桶的数量 - 1
    int  occupied;   //桶中已经缓存的方法数量。
};

/*
    OC对象的类结构体描述表示，所有OC对象的第一个参数保存是的一个isa指针。
*/
struct objc_object {
  void *isa;
};

/*
   OC类信息结构体，这里只展示出了必要的数据成员。
*/
struct objc_class : objc_object {
    struct objc_class * superclass;   //基类信息结构体。
    cache_t cache;    //方法缓存哈希表
    //... 其他数据成员忽略。
};



/*
objc_msgSend的C语言版本伪代码实现.
receiver: 是调用方法的对象
op: 是要调用的方法名称字符串
*/
id  objc_msgSend(id receiver, SEL op, ...)
{

    //1............................ 对象空值判断。
    //如果传入的对象是nil则直接返回nil
    if (receiver == nil)
        return nil;
    
   //2............................ 获取或者构造对象的isa数据。
    void *isa = NULL;
    //如果对象的地址最高位为0则表明是普通的OC对象，否则就是Tagged Pointer类型的对象
    if ((receiver & 0x8000000000000000) == 0) {
        struct objc_object  *ocobj = (struct objc_object*) receiver;
        isa = ocobj->isa;
    }
    else { //Tagged Pointer类型的对象中没有直接保存isa数据，所以需要特殊处理来查找对应的isa数据。
        
        //如果对象地址的最高4位为0xF, 那么表示是一个用户自定义扩展的Tagged Pointer类型对象
        if (((NSUInteger) receiver) >= 0xf000000000000000) {
            
            //自定义扩展的Tagged Pointer类型对象中的52-59位保存的是一个全局扩展Tagged Pointer类数组的索引值。
            int  classidx = (receiver & 0xFF0000000000000) >> 52
            isa =  objc_debug_taggedpointer_ext_classes[classidx];
        }
        else {
            
            //系统自带的Tagged Pointer类型对象中的60-63位保存的是一个全局Tagged Pointer类数组的索引值。
            int classidx = ((NSUInteger) receiver) >> 60;
            isa  =  objc_debug_taggedpointer_classes[classidx];
        }
    }
    
   //因为内存地址对齐的原因和虚拟内存空间的约束原因，
   //以及isa定义的原因需要将isa与上0xffffffff8才能得到对象所属的Class对象。
    struct objc_class  *cls = (struct objc_class *)(isa & 0xffffffff8);
    
   //3............................ 遍历缓存哈希桶并查找缓存中的方法实现。
    IMP  imp = NULL;
    //cmd与cache中的mask进行与计算得到哈希桶中的索引，来查找方法是否已经放入缓存cache哈希桶中。
    int index =  cls->cache.mask & op;
    while (true) {
        
        //如果缓存哈希桶中命中了对应的方法实现，则保存到imp中并退出循环。
        if (cls->cache.buckets[index].key == op) {
              imp = cls->cache.buckets[index].imp;
              break;
        }
        
        //方法实现并没有被缓存，并且对应的桶的数据是空的就退出循环
        if (cls->cache.buckets[index].key == NULL) {
             break;
        }
        
        //如果哈希桶中对应的项已经被占用但是又不是要执行的方法，则通过开地址法来继续寻找缓存该方法的桶。
        if (index == 0) {
            index = cls->cache.mask;  //从尾部寻找
        }
        else {
            index--;   //索引减1继续寻找。
        }
    } /*end while*/

   //4............................ 执行方法实现或方法未命中缓存处理函数
    if (imp != NULL)
         return imp(receiver, op,  ...); //这里的... 是指传递给objc_msgSend的OC方法中的参数。
    else
         return objc_msgSend_uncached(receiver, op, cls, ...);
}

/*
  方法未命中缓存处理函数：objc_msgSend_uncached的C语言版本伪代码实现，这个函数也是用汇编语言编写。
*/
id objc_msgSend_uncached(id receiver, SEL op, struct objc_class *cls)
{
   //这个函数很简单就是直接调用了_class_lookupMethodAndLoadCache3 来查找方法并缓存到struct objc_class中的cache中，最后再返回IMP类型。
  IMP  imp =   _class_lookupMethodAndLoadCache3(receiver, op, cls);
  return imp(receiver, op, ....);
}
```

# OC中的锁

@synchronized. 关键字加锁,互斥锁,性能较差不推荐使用.加锁的对象需要是同一个对象,

```
@synchronized(OC对象，一般使用self){
          需要加锁的代码
}
注意：
   a.加锁的代码尽量少
   b.添加的OC对象必须在多个线程中都是同一对象
   c.优点是不需要显示的创建锁对象，便可以实现锁的机制
   d.@synchronized块会隐式的添加一个异常处理例程来保护代码，该处理例程会在异常抛出的时候自动的释放互斥锁。所以如果不想让隐式的异常处理例程带来额外的开销，你可以考虑使用锁对象。
```

NSLock 对象锁。 互斥锁 不能多次调用lock方法，会造成死锁

```
_mutexLock = [ [NSLock alloc] init];

在需要加锁的地方
// 加锁
[_mutexLock lock];
操作代码
。。。。。
// 解锁
[_mutexLock unlock]；
```

NSRecursiveLock 递归锁。 场景限制.在递归中使用NSLock 多次调用lock ，会阻塞线程,换成NSRecursiveLock 便可以轻松解决,不过使用场景比较局限。

```
_rsLock = [[NSRecursiveLock alloc] init];
同NSLock 加锁，解锁
```

NSConditionLock 条件锁。

```
//主线程中
    NSConditionLock *theLock = [[NSConditionLock alloc] init];

    //线程1
    dispatch_async(self.concurrentQueue, ^{
        for (int i=0;i<=3;i++)
        {
            [theLock lock];
            NSLog(@"thread1:%d",i);
            sleep(1);
            [theLock unlockWithCondition:i];
        }
    });

    //线程2
    dispatch_async(self.concurrentQueue, ^{
        [theLock lockWhenCondition:2];
        NSLog(@"thread2");
        [theLock unlock];
    });
    
在线程1中的加锁使用了lock，是不需要条件的，所以顺利的就锁住了。
unlockWithCondition:在开锁的同时设置了一个整型的条件 2 。
线程2则需要一把被标识为2的钥匙，所以当线程1循环到 i = 2 时，线程2的任务才执行。

NSConditionLock也跟其它的锁一样，是需要lock与unlock对应的，只是lock,lockWhenCondition:与unlock，unlockWithCondition:是可以随意组合的，当然这是与你的需求相关的。
```

pthread_mutex （C语言）互斥锁 linux 底层.互斥锁（引入头文件#import <pthread.h>）

```
__block pthread_mutex_t mutex;
  pthread_mutex_init(&mutex, NULL);
pthread_mutex_lock(&mutex);
      NSLog(@"任务1");
      sleep(2);
      pthread_mutex_unlock(&mutex);

pthread_mutex_destroy(&mutex);  //释放该锁的数据结构
```

dispatch_semaphore （GCD）。信号量实现加锁(GCD 中提供的一种信号机制，但是有信号量与互斥锁是有区别的)

```
// 创建信号量
dispatch_semaphore_t semaphore = dispatch_semaphore_create(1);
    //线程1
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
         NSLog(@"任务1");
        sleep(10);
        dispatch_semaphore_signal(semaphore);
    });
```

OSSpinLock （不建议使用）


总结

* @synchronized：适用线程不多，任务量不大的多线程加锁
* NSLock：其实NSLock并没有想象中的那么差，不知道大家为什么不推荐使用
* dispatch_semaphore_t：使用信号来做加锁，性能提升显著
* NSCondition：使用其做多线程之间的通信调用不是线程安全的
* NSConditionLock：单纯加锁性能非常低，比NSLock低很多，但是可以用来做多线程处理不同任务的通信调用
* NSRecursiveLock：递归锁的性能出奇的高，但是只能作为递归使用,所以限制了使用场景
* NSDistributedLock：因为是MAC开发的，就不讨论了
* POSIX(pthread_mutex)：底层的api，复杂的多线程处理建议使用，并且可以封装自己的多线程
OSSpinLock：性能也非常高，可惜出现了线程问题
* dispatch_barrier_async/dispatch_barrier_sync：测试中发现dispatch_barrier_sync比dispatch_barrier_async性能要高，真是大出意外

Q：自旋和互斥的对比
A：相同点：都可以解决线程同步问题
不同点：
互斥锁会在访问被加锁数据时，会休眠等待，当数据解锁，互斥锁会被唤醒。
自旋锁遇到被加锁数据时，会进入死循环等待，当数据解锁，自旋锁马上访问。
自旋锁比互斥锁效率高！