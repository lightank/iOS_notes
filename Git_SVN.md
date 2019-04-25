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
```
