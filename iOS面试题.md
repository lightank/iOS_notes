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