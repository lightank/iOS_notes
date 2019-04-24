# iPhone 获取 .ips 文件
设置 → 隐私 → 分析 → 分析数据 → 选择 *.ips 文件 → 右上角分享

# iOS .ips(crash)崩溃报告文件分析
1. 在桌面新建一个文件夹，名字叫`crash`
2. 将`.ips`文件更名为`.crash`文件并放到`crash`文件夹中
3. 在路径`/Applications/Xcode.app/Contents/SharedFrameworks/DVTFoundation.framework/Versions/A/Resources/symbolicatecrash `中复制`symbolicatecrash`,并拷到桌面的`crash`文件夹里
4. 复制当前崩溃版本的`xxx.app.dSYM`文件粘贴到`crash`文件夹中。建议使用[dSYMTools](https://github.com/answer-huang/dSYMTools)来查看`dSYM UUID`是否一致。

    `dSYM文件`
     1. 可在`~/Library/Developer/Xcode/Archives`中找到
     2. `Xcode`->`Window`->`Organizer`->`Archives`->选择对应的`iOS Apps`->单击选择对应版本->右键选择`Show in Finder`
1. 打开终端, cd到桌面crash文件夹
    1. 输入`./symbolicatecrash`, 再输入空格
    2. 拖动`.crash`文件到终端, 会加入此文件路径, 再输入空格
    3. 拖动`xxx.app.dSYM`文件到终端, 再输入空格
    4. 输入`> log.crash`, 其中`log`为你想要的符号化的崩溃日志文件名, 可自行修改
    5. 以上也可以通过通配符`*`来输入命令：`./symbolicatecrash ./*.crash ./*.app.dSYM > log.crash`
    6. 在crash文件中就会新增log.crash文件，然后分析bug就可以了。
    
    如果终端报错：
    
    ```
    Error: "DEVELOPER_DIR" is not defined at ./symbolicatecrash line 69.
    ```
    
    输入：
    
    ```
    export DEVELOPER_DIR="/Applications/XCode.app/Contents/Developer"
    ```
    重复上述操作步骤
    
# 查看UUID
每一个 xx.app 和 xx.app.dSYM 文件都有对应的 UUID，crash 文件也有自己的 UUID，只要这三个文件的 UUID 一致，我们就可以通过他们解析出正确的错误函数信息了
## DSYM

终端执行

```
$ dwarfdump --uuid AppName.app.dSYM(这里是dsym文件的路径)
```

结果如下:

```
UUID: F73FXXXX-XXXX-XXXX-XXXX-0F7BB758XXXX (armv7) XXXXX
UUID: 371FXXXX-XXXX-XXXX-XXXX-61AB3F0CXXXX (arm64) XXXXX
```

## .app

终端执行

```
$ dwarfdump --uuid AppName.app/AppName (这里是dsym文件的路径 后面需要拼接AppName)
```

结果如下:

```
UUID: F73FXXXX-XXXX-XXXX-XXXX-0F7BB758XXXX (armv7) XXXXX
UUID: 371FXXXX-XXXX-XXXX-XXXX-61AB3F0CXXXX (arm64) XXXXX
```

## .crash

终端执行

```
$ grep "AppName arm64" *.crash(这里是crash文件路径)
```

结果如下:

```
0x1048dc000 - 0x10534bfff AppName arm64  <371fxxxxxxxxxxxxxxxx61ab3f0cxxxx>
```

同时也可以打开crash文件，第一行的中的`slice_uuid`内容`"slice_uuid":"371fxxxxxxxxxxxxxxxx61ab3f0cxxxx",`