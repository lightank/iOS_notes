# iOS 10.3+ 动态修改AppIcon
当前该功能只支持iOS 10.3以上的系统，且只能更换项目中提前配置好的Icon图，所以可以作为一项附加功能使用。应用场景如下：

API方法

```
@interface UIApplication (UIAlternateApplicationIcons)
// 如果为NO，当前进程不支持替换图标
@property (readonly, nonatomic) BOOL supportsAlternateIcons NS_EXTENSION_UNAVAILABLE("Extensions may not have alternate icons") API_AVAILABLE(ios(10.3), tvos(10.2));

// 传入nil代表使用主图标，完成后的操作将会在任意的后台队列中异步执行；如果需要更改UI，请确保在主队列中执行。
- (void)setAlternateIconName:(nullable NSString *)alternateIconName completionHandler:(nullable void (^)(NSError *_Nullable error))completionHandler NS_EXTENSION_UNAVAILABLE("Extensions may not have alternate icons") API_AVAILABLE(ios(10.3), tvos(10.2));

// 如果alternateIconName为nil，则代表当前使用的是主图标
@property (nullable, readonly, nonatomic) NSString *alternateIconName NS_EXTENSION_UNAVAILABLE("Extensions may not have alternate icons") API_AVAILABLE(ios(10.3), tvos(10.2));
@end
```

配置（Info.plist）

```
<key>CFBundleIcons</key>
<dict>
	<key>CFBundleAlternateIcons</key>
	<dict>
		<key>YourImageOneName</key>
		<dict>
			<key>CFBundleIconFiles</key>
			<array>
				<string>YourImageOneName</string>
			</array>
			<key>UIPrerenderedIcon</key>
			<false/>
		</dict>
		<key>YourImageTwoName</key>
		<dict>
			<key>CFBundleIconFiles</key>
			<array>
				<string>YourImageTwoName</string>
			</array>
			<key>UIPrerenderedIcon</key>
			<false/>
		</dict>
	</dict>
	<key>CFBundlePrimaryIcon</key>
	<dict>
		<key>CFBundleIconFiles</key>
		<array>
			<string>AppIcon</string>
		</array>
		<key>UIPrerenderedIcon</key>
		<false/>
	</dict>
</dict>
```

其中 `YourImageOneName`、`YourImageTwoName`是你要切换的appicon图片名称，注意，这个图片不能放到Assets中，直接跟代码放一块，那个 `CFBundleIconFiles` 是个数组可以指定多张图片以适配不同机型

切换代码

```
- (void)changeAppIcon:(NSString *)imageName
{
    if ([[UIApplication sharedApplication] supportsAlternateIcons])
    {
        NSLog(@"支持动态替换");
        [[UIApplication sharedApplication] setAlternateIconName:imageName completionHandler:^(NSError * _Nullable error) {
            if (error) {
                NSLog(@"修改AppIcon出错了: %@", error.description);
            }
        }];
    }
    else
    {
        NSLog(@"不支持动态替换");
    }
}
```

如果想还原以前的，imageName传nil就行


# 参考链接
* [iOS 10.3+ 动态修改AppIcon](http://zxy.science/2018/05/15/oc-change-appicon/)