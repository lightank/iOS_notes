# iOS 国际化
## 主流语言参考

| 本国语言名 | 英文名 | 中文名 | 英文缩写
| ----- | ----- | ----- | -----
| English | English | 英语 | en
| Français | French | 法语 | fr
| Deutsch | German | 德语 | de
| 简体中文 | Chinese (Simplified) | 简体中文 | zh-Hans
| 繁體中文 | Chinese (Traditional) | 繁体中文 | zh-Hant
| 日本語 | Japanese | 日语 | ja
| Español | Spanish | 西班牙语 | es
|  | Spanish (Mexico) | 墨西哥西班牙语 |  es-MX
Italiano | Italian | 意大利语 | it
| | Dutch | 荷兰语 | nl 
한국어 | Korean | 韩语 | ko
| | Portuguese (Brazil) | 巴西葡萄牙语 | pt-BR
| Português | Portuguese (Portugal) | 葡萄牙语 | pt-PT
| Dansk | Danish | 丹麦语 | da
|  | Finnish | 芬兰语 | fi
|  | Norwegian (Bokmål) |  挪威语 | nb
|  | Swedish | 瑞典 | sv
|  | Russian | 俄罗斯语 | ru
|  | Polish | 波兰语 | pl
|  | Turkish | 土耳其语 | tr
|  | Arabic | 阿拉伯语 | ar
|  | Thai | 泰国语 | th
|  | Czech | 捷克语 | cs
|  | Hungarian | 匈牙利语 | hu
|  | Catalan | 加泰罗尼亚语  | ca
|  | Croatian | 克罗地亚语 | hr
|  | Greek | 希腊语 | el
|  | Hebrew | 希伯来语 | he
|  | Romanian | 罗马尼亚语 | ro
|  | Slovak | 斯洛伐克语 | sk
|  | Ukrainian | 乌克兰语 | uk
|  | Indonesian | 印度尼西亚 | id
|  | Malay | 马来语 | ms
|  | Vietnamese | 越南 | vi


## 应用名称本地化/国际化
* 选中`Info.plist`, 按下键盘上的`command + N`，选择`Strings File`（`iOS` -> `Resource`-> `Strings File`）
* 文件名字命名为`InfoPlist`, 且必须是这个名字,点击create后, Xcode左侧导航列表就会出现名为`InfoPlist.strings`的文件
* Tips：即使有多个`Target`、`*.plist`文件，系统也只认`InfoPlist.strings`的文件
    
## 代码中字符串的本地化
* 选中`Info.plist`, 按下键盘上的`command + N`，选择`Strings File`（`iOS` -> `Resource`-> `Strings File`）
* 文件名字命名为`Localizable`, 且必须是这个名字,点击create后, Xcode左侧导航列表就会出现名为`Localizable.strings`的文件


## 配置需要国际化的语言
* 选中`project` -> `Info` -> `Localizations`，然后点击`+`，添加需要国际化/本地化的语言（默认需要勾选`Use Base Internationalization`）
* 应该会弹出对话框，默认勾选了上面建的`*.strings`文件，直接点击finish

## 本地化
### 配置语言
* 选中`*.strings`,如果没有展开文件，则在`Xcode`的`File inspection`中点击`Localize`,勾选上对应的语言，如已经展开，则忽略

### 应用名称
* 在`InfoPlist.strings(english)`中加入如下代码：`CFBundleDisplayName = "Localizable App Name";`
* 在`InfoPlist.strings(French)`中加入如下代码：`CFBundleDisplayName = "Le nom de la localisation de l'App";`
* 在`InfoPlist.strings(Chinese(Simplified))`中加入如下代码：`CFBundleDisplayName = "国际化App名称";`
* 备注：CFBundleDisplayName可以使用双引号，也可以不使用双引号！

### 代码本地化
* 假设有`NSString *title = NSLocalizedString(@"click", nil);`
* 在`Localizable.strings(english)`中加入如下代码：`"click" = "hit";`
* 在`Localizable.strings(Chinese(Simplified))`中加入如下代码：`"click" = "点击";`

### 图片本地化

* 新版Xcode中`*.xcassets`不支持国际化（属性页面中没有`Localization`），Xcode5以前是支持的
* 需要国际化的图片放在自定义`Group`里，不需要的图片放在`*.xcassets`里

## 多人开发情况下的字符串本地化
* 上面介绍的代码中字符串的本地化是使用的是默认的文件名`Localizable`,因为启动程序时，系统将根据语言加载相应的文件得到其对应的字符串文件，这个字符串可以通过系统将`NSLocalizedString`中的宏生成名为`Localizable.strings`的文件。那么如何让系统加载我们自己命名的本地化文件而非系统默认的`Localizable.strings`呢？这就是 `NSLocalizedStringFromTable(key, tbl, comment)`的用处。也就是说，如果你的`strings`文件名字不是`Localizable`而是自定义的话，如`VVS.strings`，那么你就得使用`NSLocalizedStringFromTable`这个宏来读取本地化字符串。

    ```
    // key：系统根据key取字符串
    // tbl：自定义strings文件的名字
    // comment：可以不传
    NSLocalizedStringFromTable(<#key#>, <#tbl#>, <#comment#>)
    ```
    
## 查看/切换本地语言
* 查看
  
  ```
  NSArray *languages = [[NSUserDefaults standardUserDefaults] valueForKey:@"AppleLanguages"];
    NSString *currentLanguage = languages.firstObject;
  ```
* 切换
  
  ```
  [[NSUserDefaults standardUserDefaults] setObject:@[@"zh-Hans"] forKey:@"AppleLanguages"]
  ```
  
* 重置
  
  ```
  [[NSUserDefaults standardUserDefaults] removeObjectForKey:@"AppleLanguages"];
  //[[NSUserDefaults standardUserDefaults] setObject:nil forKey:@"AppleLanguages"];
  ```

## Tips
* 切换语言无需在模拟器中设置，只需要在Xcode中进行如下设置： `Edit` -> `Scheme` -> `Run` -> `Arguments Passed On Launch` -> ` AppleLanguages` (语言代码)。其实本质上就是给`NSUserDefaults`中名为`AppleLanguages`的`key`赋值。
* 命名建议:对`Localizable.strings`使用命名空间：

  ```
  "Main.Home" = "主页";
  "My.Home" = "我的主页";
  ```
  
* 使用 `NSLocalizedStringFromTable(key, tbl, comment)` 或 `NSLocalizedString(key, comment)` 做国际化的话，手机系统语言更改后，需将App Kill后，重新进入才有改变，这样不友好，建议使用 `Bundle` 资源读取方法
    
  几个国际化的宏

  ```
    #define kCurrentLanguage (((NSArray *)([[NSUserDefaults standardUserDefaults] valueForKey:@"AppleLanguages"])).firstObject)
    #define LTLocalizedString(string) ([[NSBundle bundleWithPath:[[NSBundle mainBundle] pathForResource:kCurrentLanguage ofType:@"lproj"]] localizedStringForKey:(string) value:@"" table:nil])
    #define LTLocalizedStringWithTable(string, table) ([[NSBundle bundleWithPath:[[NSBundle mainBundle] pathForResource:kCurrentLanguage ofType:@"lproj"]] localizedStringForKey:(string) value:@"" table:table])
  ```

* 设置运行语言环境：`Edite Scheme` -> `Run` -> `Options` -> `Application Language`

* 对不同Target区分编程

    ```
    #if TargetA
    
    #elif TargetB   //可能有可以没有
    
    #endif
    ```
* 修改默认开发语言(Xcode默认开发语言是en英语)
    1. `Show in Finder` 工程文件 –> 显示包内容 –> 用文本打开`project.pbxproj` –> 搜索`developmentRegion` –> 将值改为`zh-Hans`


## 坑

* 简体中文环境下，模拟器取出当前语言为`zh-Hans`, 真机是`zh-Hans-CN`(前面是语言，后面是区域),而Xcode建的文件夹是`zh-Hans.lproj`,在真机运行下，直接根据当前语言去找`*.lproj`文件夹的话找不着，如果有富文本的话，会崩溃;英文同理；解决方案：
    
    ```
    // 取当前语言前缀
    + (NSString *)currentlprojName
    {
        NSString *prefix = [[self currentLanguage] componentsSeparatedByString:@"-"].firstObject;
        NSString *currentlprojName = [self lprojFileNameDictionary][prefix];
        return currentlprojName;
    }
    ```
* 多Target下的`Info.plist`国际化
    * 不同Target可能有不同的`*.plist`,但在国际化时，系统只认`InfoPlist.strings`的国际化

# 参考链接
* [Internationalization and Localization Guide](https://developer.apple.com/library/archive/documentation/MacOSX/Conceptual/BPInternational/LocalizingYourApp/LocalizingYourApp.html)
* [App Icons on iPhone, iPad and Apple Watch](https://developer.apple.com/library/archive/qa/qa1686/_index.html)
* [3分钟实现iOS语言本地化/国际化（图文详解）](https://www.jianshu.com/p/88c1b65e3ddb)
* [NSLocalizedString 多语言本地化](https://swiftcafe.io/post/ios-localize)