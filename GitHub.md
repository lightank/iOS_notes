# GitHub
## 搜索
* [GitHub Trend](https://github.com/trending) 页面总结了每天/每周/每月周期的热门 Repositories 和 Developers，你可以看到在某个周期处于热门状态的开发项目和开发者。
* [GitHub Topic](https://github.com/topics) 展示了最新和最流行的讨论主题，在这里你不仅能够看到开发项目，还能看到更多非开发技术的讨论主题，比如 Job、Chrome 浏览器等。
* [高级搜索功能](https://github.com/search/advanced)
* [Searching on GitHub](https://help.github.com/articles/searching-on-github/) 官方给出的帮助指南

### 条件搜索
#### 搜开发者
| 搜索条件 | 备注 |
| :-: | :-: |
| location: | location:china ,匹配用户填写的地址在china |
| language: | language:jiavascript,匹配开发语言为javascript的开发者 |
| followers: | followers:> = 1000,匹配拥有超过1000名关注者的开发者 |
| in:fullname | jack in:fullname,匹配用户实名为jack的开发者 |

* 在 [这里](https://help.github.com/articles/about-searching-on-github/) 可以查看 GitHub 官方出品的搜索技巧
* 比如需要寻找国产软件，首先想到的应该是在 GituHub 上找国内开发者，搜索时设置 `location` 为 `China`，如果你要寻找使用 `javascript` 语言开发者，则再增加 `language` 为 `javascript`，整个搜索条件就是：`language:javascript location:china`，从搜索结果来看，我们找到了近 17000 名地区信息填写为 `china` 的 `javascript` 开发者，朋友们熟悉的阮一峰老师排在前列。根据官方指引，搜索 GitHub 用户时还支持使用 `followers`、`in:fullname` 组合条件进行搜索。

#### 搜项目
| 搜索条件 | 备注 |
| :-: | :-: |
| Awesome +关键字 | 神奇的关键字Awesome,帮助找到优秀的工具列表 |
| stars: | stars:> = 500，匹配收藏数量超过500的项目 |
| language: | language;javascript,匹配以javascript作为开发语言的项目 |
| forks: | forks:> = 500,匹配分支数量超过500的项目 |

* Awesome + 关键字: Awesome 似乎已经成为不少 GitHub 项目喜爱的命名之一，比如前面提及要找到优秀的 Windows 软件，可以尝试搜索 Awesome windows，排名前列的结果出现了 [Windows/Awesome](https://github.com/Awesome-Windows/Awesome) 项目
* 如果你明确需要寻找某类特定的项目，比如用某种语言开发、Stars 数量需要达到标准的项目，在搜索框中直接输入搜索条件即可。其中用于发现项目，我的用法是灵活运用下面几个搜索条件：stars:、language:、forks:，其实就是设置项目收藏、开发语言、派生的搜索条件，比如输入 stars:>=500 language:javascript，[得到的结果](https://github.com/search?q=stars%3A%3E%3D500+language%3Ajavascript) 就是收藏大于和等于 500 的 javascript 项目，排名前列是开源代码库和课程项目 freeCodeCamp、大热门的 Vue 和 React 项目。

## 优秀项目
* [free-programming-books](https://github.com/vhf/free-programming-books) 整理了所有和编程相关的免费书籍，同时也有 中文版项目。
* [github-cheat-sheet](https://github.com/tiimgreen/github-cheat-sheet/) 集合了使用 GitHub 的各种技巧。
* [android-open-project](https://github.com/Trinea/android-open-project) 涵盖 Android 开发的优秀开源项目。
* [chinese-independent-developer](https://github.com/1c7/chinese-independent-developer) 聚合所有中国独立开发者的项目。

## 开源软件
### Windows
* [NotePad++](https://github.com/donho/notepad-plus-plus) 轻量编辑器 [官网](https://notepad-plus-plus.org) [下载](https://notepad-plus-plus.org/download/v7.5.1.html)
* [Sumatra PDF](https://github.com/sumatrapdfreader/sumatrapdf) 轻量级 PDF 阅读器 [官网](https://www.sumatrapdfreader.org/free-pdf-reader.html) [下载](https://www.sumatrapdfreader.org/download-free-pdf-viewer.html)
* [Calibre](https://github.com/kovidgoyal/calibre) 电子书管理神器 [官网](https://calibre-ebook.com/) [下载](https://calibre-ebook.com/download) 
    * Calibre 支持 Windows、macOS、Linux。
* [Zogvm](https://github.com/zogvm/zogvm) 音视频管理系统
* [Wox](https://github.com/Wox-launcher/Wox) 一款快捷启动器，类似于 macOS 的 Alfred 和 Windows 的 Listary。它可以启动程序、执行命令行、搜索网页，还可以自定义各种功能。 [官网](http://www.getwox.com/) [下载](https://github.com/Wox-launcher/Wox/releases)
* [KeePassX](https://github.com/keepassx/keepassx) 一款本地密码管理软件，它是 KeePass 的一个移植版本，如今已经是全平台密码管理软件了。整体很简单，密码方面的功能够用，采用了 AES256 和 Twofish 算法来加密密码。支持自动排序、快速备份等功能。 [官网](http://www.keepassx.org/) [下载](https://www.keepassx.org/downloads)
    * KeePassX 支持 Windows、macOS、Linux。 
* [MacType](https://github.com/snowie2000/mactype) 字体渲染增强 [官网](http://www.mactype.net/) [下载](https://github.com/snowie2000/mactype/releases)
* [TranslucentTB](https://github.com/TranslucentTB/TranslucentTB) 一个可以让 Windows 底部的菜单栏变得模糊、完全透明的小插件。 [下载](https://github.com/TranslucentTB/TranslucentTB/releases)
* [Rainmeter](https://github.com/rainmeter/rainmeter) 一款功能极其强大的桌面动效软件，并在许多网友的帮助下逐渐建立起了强大的社区，可以高度自定义桌面。[官网](https://www.rainmeter.net/) [下载](https://github.com/rainmeter/rainmeter/releases/)


### Mac OS
* [鼠须管（Squirrel）](https://github.com/rime/squirrel) 一个 Mac 平台的输入法，它基于 RIME／中州韵输入法引擎，支持朙月拼音、仓颉五代、五笔86、粤拼、吴语、中古全拼／三拼、国际音标输入法及 emoji 表情等几乎所有音码和形码输入法。 [下载](http://rime.im/)
* [SketchI18N](https://github.com/cute/SketchI18N) Sketch 的多语言（简体中文、繁体中文、日语）显示插件，让 Sketch 显示中文界面。 [下载](https://github.com/cute/SketchI18N/archive/master.zip)
* [Spectacle](https://github.com/eczarny/spectacle) 窗口布局工具,通过快捷键完成窗口的迅速布局，支持上下左右中间全屏半屏四分之一屏等多种形式，帮你提高桌面空间利用率和窗口管理效率。 [下载](https://www.spectacleapp.com/)
* [LyricsX](https://github.com/ddddxxx/LyricsX) 一款功能完备的歌词工具,显示歌词的免费应用，支持 iTunes、Spotify 和 Vox，自动匹配和下载歌词，在桌面和菜单栏显示歌词。歌词源支持网易云音乐、QQ 音乐、虾米、酷狗等。 [下载](https://itunes.apple.com/us/app/lyricsx/id1254743014?mt=12)
* [IINA](https://github.com/lhc70000/iina) 颜值和功能兼顾的开源播放器，使用了 mpv 作为自己的播放内核，支持几乎所有常用媒体的播放（甚至是 GIF），支持 Touch Bar、 Force Touch 和字幕自动下载。 [下载](https://lhc70000.github.io/iina/zh-cn/)
* [Kawa](https://github.com/utatti/kawa) 给每个输入法定义一个快捷键,适合安装了多个输入法的用户，给每个输入法定义一个快捷键，一键切换输入法。
* [TinyPNG4Mac](https://github.com/kyleduo/TinyPNG4Mac) [TinyPNG](https://tinypng.com) 是一个专注 png 图片压缩的网站，平均压缩比例能达到70%，同时对于图片质量的损失几乎为 0 。TinyPNG4Mac 是 TinyPNG 服务的 Mac 客户端。首次使用需要到 TinyPNG 申请一个秘钥，需要填个邮箱和用户名即可，秘钥会自动发到邮箱。[下载](https://github.com/kyleduo/TinyPNG4Mac/releases)
* [uBlock Origin](https://github.com/el1t/uBlock-Safari) Safari 扩展，用来拦截广告净化网页内容，包含大量的可更新规则，支持规则自定义。 [下载](https://github.com/el1t/uBlock-Safari/releases)
* [Aria2GUI](https://github.com/yangshun1029/aria2gui) Aria2 是一个命令行下轻量级、多协议、多来源的下载工具（支持 HTTP/HTTPS、FTP、BitTorrent、Metalink），内建 XML-RPC 和 JSON-RPC 用户界面。Aria2GUI 是集成 aria2c 的可视化下载客户端，Safari 直接下载还需要配合 Safari2aria 使用。 [下载](https://github.com/yangshun1029/aria2gui/releases)


# 参考文章
* [GitHub 上那些免费好用的 Windows 软件](https://sspai.com/post/41563)
* [GitHub 中那些不错的免费软件](https://sspai.com/post/41325)
* [掌握 3 个搜索技巧，在 GitHub 上快速找到实用软件资源](https://sspai.com/post/46061)