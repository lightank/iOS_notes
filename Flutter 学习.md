# Flutter 学习日志

## 开发环境
* Fultter 环境 [官网下载](https://flutter.dev/docs/get-started/install)、[GitHub 下载](https://github.com/flutter/flutter/releases)
    * [更新 PATH 环境变量](https://flutter.cn/docs/get-started/install/macos#update-your-path)
    * `.bash_profile`,
        * 一般在Mac上配置环境变量时经常要创建、编辑 `.bash_profile` 文件。创建该文件时一般都会选择在当前用户目录下，即Mac下的 ·.bash_profile· 文件的路径是 `/Users/YourMacUserName/.bash_profile` (如果该文件已经创建过的话)
        * 创建 .bash_profile: 
            * 启动终端 
            * `cd ~   或 cd /Users/YourMacUserName` 
            * 输入`touch .bash_profile` 
        * 查看 、编辑 `.bash_profile` 文件
            * 终端输入 `open -e .bash_profile`（如果只是查看，直接使用 `open .bash_profile`）
            * 编辑，添加 `export PATH="$PATH:[PATH_TO_FLUTTER_GIT_DIRECTORY]/flutter/bin`，请将其中`[PATH_TO_FLUTTER_GIT_DIRECTORY]` 更改为下载的 Fultter SDK 路径，添加前请终端验证此路径是否能够打开 `open [PATH_TO_FLUTTER_GIT_DIRECTORY]`
            * 关闭即可保存修改
        * 更新刚配置的环境变量
            * 输入 `source .bash_profile`
        *  如果你使用的是 [zsh](https://ohmyz.sh/)，终端启动时 `~/.bash_profile` 将不会被加载，解决办法就是修改 `~/.zshrc` ，在其中添加：`source ~/.bash_profile`
    * [设置 iOS 开发环境](https://flutter.cn/docs/get-started/install/macos#ios-setup)
* IDE:
    * Xcode for iOS [App Store 下载](https://apps.apple.com/cn/app/xcode/id497799835)、[开发者中心下载](http://developer.apple.com/download/more) 
    * [Android Studio](https://developer.android.com/studio) for Android
    * [Visual Studio Code](https://code.visualstudio.com/) 推荐

## 相关链接
* [Flutter cn](https://flutter.cn/)