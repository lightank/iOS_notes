# Git

## Tips
### `.gitignore`中增加了过滤规则但是不起作用
是由于在创建`.gitignore`文件或添加一些过滤规则之前就track了相应的内容，那么即使在`.gitignore`文件中写入新的过滤规则，这些规则也不会起作用，Git仍然会对这些文件进行版本管理。简单来说出现这种问题的原因就是Git已经开始管理这些文件了，所以你无法再通过过滤规则过滤它们。 

解决方法就是先把本地这些文件变成未track状态，具体来说就是在缓存里删除它们，然后提交：

```
git rm -r --cached .
git add .
git commit -m 'update .gitignore'
```

### 解决mac下的Sourcetree每次拉取提交都需要输入密码
sourceTree项目的GIT密码始终保存不到Mac的钥匙串中,明明在钥匙串中是存在的.但是在使用sourceTree pull/push代码的时候还是需要再输入密码,很是繁琐.

于是,网上搜索了一下,说的在https模式下,Mac需要使用osxkeychain凭据助手,并在Git中设置使用. 并且如果已经安装了 brew 的应该会自带了 osxkeychain .但是奇怪的是,我安装了brew的,使用brew安装应用也没有问题.那就只能手动的再设置一次了.

方法

1. 先使用命令下载 `git-credential-osxkeychain`：

    `curl http://github-media-downloads.s3.amazonaws.com/osx/git-credential-osxkeychain -o git-credential-osxkeychain`

1. 把 git-credential-osxkeychain 放入 bin目录：
    `mv git-credential-osxkeychain /usr/local/bin`

1. 给 `git-credential-osxkeychain` 赋权限：
    `chmod u+x /usr/local/bin/git-credential-osxkeychain`

4. 在Git全局配置中进行设置(也可以在某一个项目里面设置): 
    `git config --global credential.helper osxkeychain`

经过上面的设置，下次访问https的项目时只需要输入一次密码,就会存储到osx的钥匙串中了,以后再也不会在Git中询问了.

# SVN

针对Cornerstone 3的 log 报错

```
Could not contact repository to read the latest log entries.
```
解决方式：

```
应用程序 → Cornerstone.app → 右键显示包内容 → Contents → 打开Info.plist → 复制 Bundle identifier 的值，比如 com.zennaware.cornerstone3 → Quit Cornerstone(退出cornerStone) → Open Terminal(打开终端) → 输入以下文字
$: defaults delete com.zennaware.cornerstone3 HistoryCacheUsage
注意中间的值请改成对应的info.plist中的值

重启Cornerstone → 点击 log 弹出警告框

Do you want to download and cache the repository's history?

Never For This Repository     Not Now     Download

切记一定要选择Not Now,不要默认选择的Download
```
