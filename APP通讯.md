# URL Scheme

## 创建URL Scheme
### 方法一
1. 首先在`Info.plist`中添加一行,选择`URL types`
2. 在展开的`Item 0`中填写`URL identifier`,这个用来唯一标识用户自定义的`URL Schemes`，推荐使用域名的反转形式，如:`com.huanyu.li.universalApp.urlScheme`
3. 在`Item 0`中添加新的一行，选择`URL Schemes`
4. 展开`URL Schemes`，在`Item 0`中输入自定义的`Scheme`的名称。在这里只需要输入自定义的`Scheme`的名称即可，不需要加上:`//`，例如这里输入的是`universalApp`,那么对应的自定义的URL就是`universalApp://`，这里可以输入多个。

### 方法二
1. `Open source`打开`Info.plist`
2. 在尾部添加下列内容

```xml
	<key>CFBundleURLTypes</key>
	<array>
		<dict>
			<key>CFBundleURLName</key>
			<string>com.huanyu.li.universalApp.urlScheme</string>
			<key>CFBundleURLSchemes</key>
			<array>
				<string>universalApp</string>
			</array>
		</dict>
	</array>
```

## 使用URL Scheme
### 1. 在Safari中使用

在Safari中直接在浏览器的地址栏中输入`universalApp://`,即可启动刚才的应用

### 2. 在其他的应用程序中使用

在需要调用的地方使用下面的代码即可实现调用

```objc
NSString *customURL = @"universalApp://";
[[UIApplication sharedApplication] openURL:[NSURL URLWithString:customURL]];
```
### 3. 参数的传递

```
- (void)openOtherApp
{
    NSString *customURL = @"universalApp://?token=123&order=1";
    [[UIApplication sharedApplication] openURL:[NSURL URLWithString:customURL]];
}
```
   
在`AppDelegate`中实现下面的两个方法

```
- (BOOL)application:(UIApplication *)application handleOpenURL:(NSURL *)url
    
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation
```

示例：

```
AppDelegate:
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation
{
    if ([sourceApplication isEqualToString:@"com.huanyu.li.universalApp.urlScheme"])
    {
        NSLog(@"调用的应用程序的Bundle ID是: %@", sourceApplication);
        NSLog(@"URL scheme:%@", [url scheme]);
        NSLog(@"URL query: %@", [url query]);
        return YES;
    }
    else
    {
        return NO;
    }
}
```

说明:

(1)上面的两个函数作用是一致的只是参数不同而已，函数的返回值是BOOL,如果为YES表示可以打开，NO表示不可以打开应用程序

(2)参数可以通过`[url query]`来获取，比如使用的是`universalApp://?token=123&order=1`那么通过`[url query]`获取到的值是`token=123&order=1`,然后可以通过这些数据再作相应的处理.

(3)调用的应用程序的`Bundle ID`可以通过`sourceApplication`参数获取

(4)通过`[url scheme]`可以获取到请求的`URL Scheme`，比如是通过`universalApp://`打开的那么`[url scheme]`的值就是`universalApp`。可以通过不同的参数来判断来源的合法性






# 参考链接
* [iOS中的URL Scheme](https://blog.devzeng.com/blog/ios-url-scheme.html)
