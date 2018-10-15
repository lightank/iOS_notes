# 诉求

1. 构建不同的宏来方便切换相应的配置；
2. 配置三种数据环境根据宏的切换进行切换；
3. 三种图标/应用名称根据宏的切换进行切换显示；
4. 至少两个类型的包能同时安装在手机上；
5. 最好能使用脚本实现自动化打包放入bugly或者蒲公英等平台供内部测试人员下载；

# 创建多个Configuration
有两种方法可以用来创建我们需要新增的`Build Configuration`，这里新建一个名为`Sit`的配置项，是为了满足App的网络环境的切换。

![]()

![]()

# 环境的配置

1. 选择`Duplicate "Debug" Configuration`,得到`Debug copy`重命名为`Sit`
2. 然后`PROJECT` -> `Build Settings` -> `All` -> `Combined` -> search `Preprocessor Macros`，在`Sit`添加一个值为`SIT=1`
3. 重复上述操作分别添加`Dev`,`Uat`

![]()

![]()

**提示：**如果在`TARGETS`中搜索的`Preprocessor Macros`,你会发现`Multiple values`中有一个值是`$(inherited)`,`TARGETS`中的值会继承自`PROJECT`

在项目中新建一个`Header File`,命名为`ApplicationConfig`,添加一下代码

```
#ifdef DEBUG
// Debug模式

#else
// Release模式

#endif


#ifdef DEV
// DEV环境
#definede kBaseURL @"https://127.0.0.1/"

#endif

#ifdef SIT
// SIT环境
#definede kBaseURL @"https://127.0.0.2/"

#endif

#ifdef UAT
// UAT环境
#definede kBaseURL @"https://127.0.0.3/"

#endif

#ifdef PROD
// PROD环境
#definede kBaseURL @"https://127.0.0.4/"

#endif
```

在`Edit Scheme`中可以选择不同`Configuration`

![]()


# Tips

## 配置不同的AppIcon
配置AppIcon有两种必比较方便的方法。

第一种:

首先我们需要找UI设计师要三套不一样的图标，如下图这样取好对应的名称放入`*.xcassets`中：

![]()

然后再当前`Target`的`Build Setting`下搜索icon找到`Asset Catalog App Icon Set Name`，然后进行如下配置：

![]()

然后Edit Scheme选择相应的Configuration进行编译或者打包就能打出不同的图标了。

第二种

使用`User-Defined`配置三种`Configuration`下的变量，在`Info.plist`中进行配置，配置方法与下面的应用名称配置类似，这里不做过多描述。

## 配置不同的AppName

配置不同的应用名称，这里需要使用到User-Defined加上info.plist来进行配置；
首先，我们需要新增一个User-Defined，如下图：

![方法一]()
![方法二]()

然后在`info.plist`中加入`Bundle display name`，设置成我们刚刚新建的`User-Defined`值：`$(APP_DISPLAY_NAME)`

![方法二]()


## 配置不同Configuration下Target的`Bundle ldentifier`

`TARGETS` -> `Build Settings` -> `All` -> `Combined` -> search `Product Bundle Identifier`

我们可以根据自己的需要设置各种scheme下的配置不同的Bundle ID，如下图：

![]()

正常情况下，以上步骤完成之后，如上图选择`Edit Scheme`切换`Build Configuration`就能编译出相应环境下的App，但是如果你的App使用pods来管理第三方库，使用新建的配置项就会报错找不到第三方的库文件，错误信息类似如下：

```
library not found for -lAFNetworking

linker command failed with exit code 1 (use -V to see invocation)
```

原因是`pods`工程并未自动帮我们创建相应的pod配置项，发现这一点之后我手动创建了一个同样名为`Preform`的`pod`配置项，于是编译通过了，但是打ipa包的时候始终通不过，继续查找原因，原来`xcconfig`文件需要终端执行`pod install`进行全面配置，所以大家在新建完了之后记得要`pod install`一下，才能放心使用。


# 参考链接

* [Xcode Concepts](https://developer.apple.com/library/archive/featuredarticles/XcodeConcepts/Concept-Targets.html) by [Apple][Apple]
* [iOS - 开发一套代码多个app展示不同图标和名称](https://www.cnblogs.com/gongyuhonglou/p/7766291.html)




[Apple]:https://developer.apple.com/library/archive/navigation/