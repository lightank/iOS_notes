# Git

## ssh
1. 首先运行terminal检查是否已经有SSH Key

	```
	cd ~/.ssh		// 进入ssh目录,如果不能进入该目录，说明没生成过
	ls		// 查看ssh文件夹内容,如果ssh文件夹中有id_rsa、id_rsa.pub文件，说明之前生成过ssh秘钥，可以直接使用，跳过步骤2，直接进入步骤3

	git config --list    // 检查下是否配置过git账户
	```
	
2. 创建一个SSH key

	```
	ssh-keygen -t rsa -C "your_email@example.com"
	```
	接着又会提示你输入两次密码（该密码是你push文件的时候要输入的密码，而不是github管理者的密码），

	当然，你也可以不输入密码，直接按回车。那么push的时候就不需要输入密码，直接提交到github上了，如：
	
	```
	Enter passphrase (empty for no passphrase): 
	# Enter same passphrase again:
	```
	
	当你看到下面这段代码，那就说明，你的 SSH key 已经创建成功，你只需要添加到github的SSH key上就可以了：
	
	```
	our identification has been saved in /c/Users/you/.ssh/id_rsa.
	# Your public key has been saved in /c/Users/you/.ssh/id_rsa.pub.
	# The key fingerprint is:
	# 01:0f:f4:3b:ca:85:d6:17:a1:7d:f0:68:9d:f0:a2:db your_email@example.com
	```
	
3. 添加公钥到你的远程仓库（github）
	
	```
	// 查看你生成的公钥，或者执行 pbcopy < ~/.ssh/id_rsa.pub 直接复制内容到粘贴板
	cat ~/.ssh/id_rsa.pub
	
	// 把terminal上显示的内容copy出来
	ssh-rsa ******************省略一堆字串*********************************** your_email@example.com
	
	// 登陆你的github帐户。点击你的头像，然后 Settings -> 左栏点击 SSH and GPG keys -> 点击 New SSH key
	// 然后你复制上面的公钥内容，粘贴进“Key”文本域内。 title域，自己随便起个名字。
	// 点击 Add key
	
	//完成以后，验证下这个key是不是正常工作，输入：
	ssh -T git@github.com
	
	如果第二步输入了密码就会让输入密码：
	Enter passphrase for key '/Users/shutong/.ssh/id_rsa':
	
	如果，看到：
	Hi shu-tong! You've successfully authenticated, but GitHub does not provide shell access.

	成功.
	```
	
	
3. 重新配置
	
	```
	1.配置账户
	git config --global user.name "your account name"     ->用户名，建议拼音或英文
	git config --global user.email "your account email"     ->邮箱地址
	
	2.生成秘钥 
	ssh-keygen -t rsa -C "your_email@example.com"         ->上面的邮箱地址
	```



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

### git库迁移版本库(保留原版本库的所有内容)
如果你想从别的 Git 托管服务那里复制一份源代码到新的 Git 托管服务器上的话，可以通过以下步骤来操作。

1. 从原地址克隆一份裸版本库，比如原本托管于 GitHub
`git clone --bare git://github.com/username/project.git`

    如果是本地库，应该是`git clone --bare 本地库路径`
2. 然后到新的 Git 服务器上创建一个新项目，比如 Gitcafe。
3. 以镜像推送的方式上传代码到 GitCafe 服务器上。

    ```
    cd project.git
    git push --mirror git@gitcafe.com/username/newproject.git
    ```
4. 删除本地代码

    ```
    cd ..
    rm -rf project.git
    ```
5. 到新服务器 GitCafe 上找到 Clone 地址，直接 Clone 到本地就可以了
`git clone git@gitcafe.com/username/newproject.git`
这种方式可以保留原版本库中的所有内容。


## [oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh)

自动补全：[autojump](https://github.com/wting/autojump)

命令行：

```
// 查看当前分支情况
$:gst
效果为显示分支信息：
位于分支 huanyu/feature-xxx
您的分支与上游分支 'origin/huanyu/feature-xxx' 一致

// 合并分支到当前分支
$:gm origin/teamwork/feature-xxx
效果：合并 origin/teamwork/feature-xxx 分支到 huanyu/feature-xxx 分支

// git push 推送当前分支到远端
& gp

```

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
