# CocoaPods

`pod install` 如果报错
```
error: RPC failed; curl 18 transfer closed with outstanding read data remaining
fatal: the remote end hung up unexpectedly
fatal: early EOF
```
错误原因分析

git 有两种拉代码的方式，一个是 HTTP，另一个是 ssh。git 的 HTTP 底层是通过 curl 的。HTTP 底层基于 TCP，而 TCP 协议的实现是有缓冲区的。
所以这个报错大致意思就是说，连接已经关闭，但是此时有未处理完的数据。
解决方案

增大缓冲区大小。
切到 git 项目目录后，执行如下命令：

```
// 524288000 的单位代表 B，524288000B 也就是 500MB。
// 这个值的大小，可自行酌情设置
git config --global http.postBuffer 524288000
```
然后查看是否设置成功：

```
git config –list | grep postbuffer
```


# CocoaPods创建库

* [CocoaPods Guides](https://guides.cocoapods.org)
* [Podspec Syntax Reference](https://guides.cocoapods.org/syntax/podspec.html)

# 创建公开库

## 1. 注册Trunk

Trunk需要CocoaPods 0.33版本以上，用`pod --version`

```ruby
$ pod --version
1.5.3
```

如果版本低，需要升级：

```ruby
$ sudo gen install cocoapods
$ pod setup
```

查看自己是否注册过Trunk

```ruby
pod trunk me
```

如果没有注册则显示

```ruby
$ pod --version
[!] You need to register a session first.
```

如果已经注册则显示

```ruby
$ pod trunk me
  - Name:     lightank  <-------- 用户名/user name
  - Email:    1303908401@qq.com
  - Since:    September 6th, 20:19
  - Pods:
    - LTPrivacyPermission <-------- 已经有的公开库/ owned library
```

注册

```ruby
// 加上--verbose可以输出详细debug信息，方便出错时查看。
$ pod trunk register your-email@163.com "user_name" --verbose
 
"user_name" 里面代表你的用户名，最好起一个好的名字
your-email@163.com 代表你的邮箱
```

注册完成之后会给你的邮箱发个邮件,进入邮箱邮件里面有个链接,需要点击确认一下

注册成功后可以再查看一下个人信息pod trunk me

```ruby
$ pod trunk me
  - Name:     lightank  <-------- 用户名/user name
  - Email:    1303908401@qq.com
  - Since:    September 6th, 20:19
  - Pods:
    - LTPrivacyPermission <-------- 已经有的公开库/ owned library
```

## 2. 创建一个Git项目

1. GitHub上创建一个项目，比如：[LTPrivacyPermission](https://github.com/lightank/LTPrivacyPermission)
    
 ![cocoapods_create_repository](https://github.com/lightank/iOS_notes/blob/master/Resource/cocoapods/cocoapods_create_repository.png)
 
2. 将项目克隆下来,并添加公开库文件
 
 ![](https://github.com/lightank/iOS_notes/blob/master/Resource/cocoapods/cocoapods_create_folder.png)

## 3. 创建.podspec

创建.podspec: cd 到你的项目下，执行如下命令

```ruby
// 注 LTPrivacyPermission 这个是你框架的名称
$ pod spec create LTPrivacyPermission
```

## 4. 编辑.podspec文件

```ruby
Pod::Spec.new do |s|
  s.name         = "LTPrivacyPermission"
  s.version      = "0.0.1"
  s.summary      = "LTPrivacyPermission is a library for accessing various system privacy permissions"
  s.description  = <<-DESC
  LTPrivacyPermission is a library for accessing various system privacy permissions, you can use it for more friendly access.
                   DESC

  s.homepage     = "https://github.com/lightank/LTPrivacyPermission"
  s.license      = "MIT"
  s.author       = { "lightank" => "1303908401@qq.com" }
  s.platform     = :ios, "8.0"
  s.source       = { :git => "https://github.com/lightank/LTPrivacyPermission.git", :tag => "#{s.version}" }
  s.source_files = "LTPrivacyPermission/*.{h,m}"
  s.requires_arc = true
```

解释：

```ruby
s.name：名称，pod search 搜索的关键词,注意这里一定要和.podspec的名称一样,否则报错
s.version：版本号
s.summary：简介
s.homepage：项目主页地址
s.license：许可证
s.author：作者，如果多位作者，则用逗号分隔，如下：
    s.author       = {
                    "author1" => "email1@163.com",
                    "author2" => "email2@163.com",
                    "author1" => "email3@163.com",
    }
s.social_media_url：作者社交地址，yykit的示例如下：
    s.social_media_url = 'http://blog.ibireme.com'
s.platform：平台跟最低版本号，也可以如下写法：
    s.ios.deployment_target = "7.0"
    s.osx.deployment_target = "10.9"
    s.watchos.deployment_target = "2.0"
    s.tvos.deployment_target = "9.0"
s.source：代码来源，跟版本号，示例如下：
    // commit => "e65e8b2" 表示将这个Pod版本与Git仓库中某个commit绑定
    s.source = { :git => "https://github.com/lightank/LTPrivacyPermission.git", :commit => "e65e8b2" }
    // tag => 0.0.1 表示将这个Pod版本与Git仓库中某个版本的comit绑定
    s.source = { :git => "https://github.com/lightank/LTPrivacyPermission.git", :commit => "e65e8b2", :tag => 0.0.1 }
    // tag => s.version 表示将这个Pod版本与Git仓库中相同版本的comit绑定
    s.source = { :git => "https://github.com/lightank/LTPrivacyPermission.git", :tag => s.version }
s.source_files：需要包含的源文件，示例如下：
    "LTPrivacyPermission/*"
    "LTPrivacyPermission/*.{h,m}"
    "LTPrivacyPermission/**/*.{h,m}"
s.requires_arc：是否支持ARC
s.dependency：依赖库，不能依赖未发布的库，可以写多个依赖库
s.public_header_files：公开的头文件
s.private_header_files = 私有的头文件，示例如下：
    s.private_header_files = "YTKNetwork/YTKNetworkPrivate.h"
non_arc_files：不需要rac的文件，示例如下：
    non_arc_files = 'YYKit/Base/Foundation/NSObject+YYAddForARC.{h,m}', 'YYKit/Base/Foundation/NSThread+YYAdd.{h,m}'
s.libraries：需要导入的tbd库，示例如下：
    s.libraries = 'z', 'sqlite3'
s.frameworks：依赖的系统库，示例如下：
    s.frameworks = 'UIKit', 'CoreFoundation', 'CoreText'
s.dependency：依赖库，不能依赖未发布的库，可以写多个依赖库，示例如下：
    s.dependency = 'AFNetworking' , 'SDWebImage'
s.resources: 资源文件
s.ios.vendored_frameworks = 'YJKit/YJKit.framework' # 依赖的第三方/自己的framework
s.vendored_libraries = 'Library/Classes/libWeChatSDK.a' #表示依赖第三方/自己的静态库（比如libWeChatSDK.a）
#依赖的第三方的或者自己的静态库文件必须以lib为前缀进行命名，否则会出现找不到的情况，这一点非常重要
```

如果前面没有选择创建这个`LICENSE`文件， 创建`LICENSE`(许可证/授权)文件,此文件必须要有，内容如下，只需要把前面的版权改一下就行了,后面的都一样：

```
MIT License

Copyright (c) 2018 lightank

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

## 5. 上传到Git：

将包含配置好的 .podspec, LICENSE 的项目提交 Git

## 6. 版本打tag：

给指定提交版本打tag，如果版本是"0.0.1"，那么tag就为"0.0.1"，不用带"v"，不然会报错

个人推荐使用[sourcetree](https://www.sourcetreeapp.com)来打tag，管理git库等

## 7. 验证.podspec文件

```ruby
// --verbose 如果验证失败会报错误信息
$ pod spec lint LTPrivacyPermission.podspec --verbose
```

如果出现如下错误提示

```ruby
Analyzed 1 podspec.
[!] The spec did not pass validation, due to 1 warning (but you can use `--allow-warnings` to ignore it) and all results apply only to public specs, but you can use `--private` to ignore them if linting the specification for a private pod.
```

使用下列命令来消除警告

```ruby
$ pod lib lint --allow-warnings
```

出现以下错误

```ruby
xcrun: error: unable to find utility "simctl", not a developer tool or in PATH
```
去XCode设置里面，将Command line Tools设置一下，在Xcode>preferences>Locations里面，设置之后再运行终端即可

如果最后出现如下提示，则表示这个库通过了验证

```ruby
LTPrivacyPermission passed validation
```

## 8. 发布

发布时会验证 Pod 的有效性，如果你在手动验证 Pod 时使用了 `--use-libraries` 或 `--allow-warnings` 等修饰符，那么发布的时候也应该使用相同的字段修饰，否则出现相同的报错。

```ruby
// --use-libraries --allow-warnings
$ pod trunk push LTPrivacyPermission.podspec
```

出现这种情况就说明你发布成功了，等待人家审核就行了

```ruby
...
Congrats
LTPrivacyPermission (0.0.1) successfully published
...
```

## 9. 验证仓库

```ruby
$ pod search LTPrivacyPermission
```

如果出现下面的错误

```ruby
[!] Unable to find a pod with name, author summary, or description matching `LTPrivacyPermission`
```

删除缓存文件，重新生成

```ruby
$ rm ~/Library/Caches/CocoaPods/search_index.json
$ pod setup
```

如果还是不行，估计是在你的项目还在审核中，你可以通过其他辅助手段去验证；

1. 执行 pod trunk me 命令,看看有没有你的库
2. 在[CocoaPods](https://cocoapods.org)中所搜一下（这个也有延时，如果搜索到就出现这样的结果,展示的是上一个集成库的搜索结果）

共有库的创建这里就结束了


# 创建私有库

1. 创建一个私有的项目（可以是公司自己的git管理工具、也可以是码云上的）

    创建方法同公共库的第2步
    
2. 创建.podspec
  
    方法同公共库创建的第3步

3. 编辑.podspec文件

    方法同公共库创建的第4步

4. 上传到Git

    方法同公共库创建的第5步

5. 打tag

   方法同公共库创建的第6步

6. 验证.podspec文件

    方法同公共库创建的第7步

7. 添加一个私有库并和项目地址做绑定

```ruby
pod repo add ZYRunTimeCoT https://github.com/zhangyqyx/ZYRunTimeCoT.git
```

8. 向私有的库里添加podspec文件

```ruby
$ pod repo push ZYRunTimeCoT ZYRunTimeCoT.podspec
```

9. 新建一个项目进行验证 

xcode新建项目 在podfile中添加

```ruby
$ pod 'ZYRunTimeCoT', '~> 0.0.1'
```


如果提示 'unable to find a specification for'

需要在podfile文件中添加源地址

source 'https://github.com/zhangyqyx/ZYRunTimeCoT.git'

# 参考链接

* [CocoaPods创建自己的公开库、私有库](http://www.code4app.com/blog-847095-1887.html)
* [CocoaPods公有仓库的创建](http://qiubaiying.top/2017/03/08/CocoaPods公有仓库的创建/)
* [CocoaPods 私有仓库的创建（超详细）](https://www.jianshu.com/p/0c640821b36f)
* [使用cocoapods打包静态库(依赖私有库，开源库，私有库又包含静态库)](https://www.jianshu.com/p/9096a2eb2804)