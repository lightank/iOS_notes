#Charles抓包

[Charles 官网](https://www.charlesproxy.com),本文使用的是[`Charles v 4.2.6`, 资源来自网络](https://www.waitsun.com/charles-4-2-6.html)

## `HTTPS`抓包设置
 1. `菜单栏` -> `Proxy` -> `SSL Proxying Settings` -> `SSL Proxying` -> 勾上`Enable SSL Proxying`, 添加`Host`:`*`, `Port`: `443`,443端口主要用于HTTPS服务
 2. `菜单栏` -> `Proxy` -> `Proxy Settings` -> `Proxies` -> 勾上`Enable transparent HTTP proxying`, `Port`填`8888`,或勾上 `Use a dynamic port`
 
## iPhone抓包

1. `Mac`必须与`iPhone`连接同一`WiFi`
2. `菜单栏` -> `Help` -> `SSL Proxying` -> `Install Charles Root Certificate on Mobile Device or Remote Browser` 会显示一个弹窗,内容如下:

    ```
    Configure your device to use Charles as its HTTP proxy on 192.168.1.118:8888, then browse to chls.pro/ssl to download and install the certificate.
    ```
3. 在手机上设置Wifi代理:
    打开`设置` -> `无线局域网` -> 点击已经连接的WIFI -> `HTTP 代理`模块中选择手动，设置服务器和端口号， 即上边弹窗提示的IP:192.168.1.118 及 端口：8888
4. 手机端证上安装证书:
    在手机上打开Safari浏览器，输入地址：chls.pro/ssl， 安装证书。证书会被安装在`设置` -> `通用` -> `描述文件`下面
5. 手机端证上信任证书:
    直接抓包可能报错:` Client SSL handshake failed: CA certificate could not be matched with a known, trusted CA (unknown_ca)`,是因为 `10.3` 以上系统需要你在`证书信任设置`中信任 `Charles Proxy CA`的证书, 具体错误信息如下:

    ```
    URL https://www.baidu.com
    Status  Failed
    Failure Client SSL handshake failed: CA certificate could not be matched with a known, trusted CA (unknown_ca)
    Notes   You may need to configure your browser or application to trust the Charles Root Certificate. See SSL Proxying in the Help menu.
    Response Code   200 Connection established
    Protocol    HTTP/1.1
    ```
    解决办法: 证书信任设置
    打开`设置` -> `通用` -> `关于本机` -> `证书信任设置`(这个在页面最下面），将`Charles`证书开关打开

## Mac抓包
1. `菜单栏` -> `Help` -> `SSL Proxying` -> `Install Charles Root Certificate`,此时会打开钥匙串访问安装`Charles Proxy CA`证书,双击证书,展开`信任`选项,在`使用证书时`选择`始终信任`, 如果证书安装不了请搜索`Charles Proxy CA`,删除就已失效证书再进行安装操作

##Tips
### 显示视图切换:

`Charles`抓包结果支持两种显示模式:

1. `Structure`:将网络请求按访问的域名分类,可以很清晰的看到请求的数据结构，而且是以域名划分请求信息的，可以很清晰的去分析和处理数据
2. `Sequence`:将网络请求按访问的时间排序,可以看到全部请求,这里的结果以数据请求的顺序来显示,最新的请求显示在最下面
    
### 添加忽略:
在抓包调试过程中有部分链接速度缓慢,而不抓包速度正常,那么可以忽略掉这个链接,比如这个链接是:`https://www.baidu.com`

1. `菜单栏` -> `Proxy` -> `Recoding Settings` -> `Exclude` -> `Add`
2. `Protocol` 中选择 `HTTPS`, `Host` 填写 `www.baidu.com`, 端口写`443`
    
###搜索功能:`command + F`

###修改服务器返回内容:

有些时候我们想让服务器返回一些指定的内容，方便我们调试一些特殊情况。例如列表页面为空的情况，数据异常的情况，部分耗时的网络请求超时的情况等。如果没有 `Charles`，要服务器配合构造相应的数据显得会比较麻烦。这个时候，使用 `Charles` 相关的功能就可以满足我们的需求。
根据具体的需求，`Charles` 提供了 `Map` 功能、 `Rewrite` 功能以及 `Breakpoints` 功能，都可以达到修改服务器返回内容的目的。这三者在功能上的差异是：

* Map 功能适合长期地将某一些请求重定向到另一个网络地址或本地文件。
* Rewrite 功能适合对网络请求进行一些正则替换。
* Breakpoints 功能适合做一些临时性的修改。

1. 修改请求的任何信息,包括 URL 地址、端口、参数等
1. 在`Structure`或`Sequence`中右键`链接`选择`Compose`
2. 修改之后点击 `Execute` 即可发送该修改后的网络请求. `Charles` 支持我们多次修改和发送该请求，这对于我们和服务器端调试接口非常方便
        
###断点调试
1. 确认在软件界面六边形图像中开启了`Enable Breakpoints`
2. 在`Structure`或`Sequence`中右键`链接`选择`Breakpoints`
3. 重新请求将会进入`Breakpoints`界面,选择`Edit Response`,再最下方选择需要修改的部位,如:`Headers`, `JSON Text`等,一般使用`JSON Text`查看返回数据较为方便
4. 修改完后点击`Execute`

###重复请求,给服务器做压力测试,
1. 在`Structure`或`Sequence`中右键`链接`选择`Repeat`或`Repeat Advanced`
2. 如果选择了`Repeat Advanced`,在弹出的对话框中，修改`迭代(打压)次数 Iteration`以及`并发线程数 Concurrency`次数,确定之后，即可开始打压
    
###只监听/查看特定请求
1. 监听,比如这个链接是:`https://www.baidu.com`
    1. `菜单栏` -> `Proxy` -> `Recoding Settings` -> `Include` -> `Add`
    2.  `Protocol` 中选择 `HTTPS`, `Host` 填写 `www.baidu.com`, 端口写`443`
2. 查看:以下方式任一均可实现
    1. 在`Structure`或`Sequence`中右键`https://www.baidu.com`选择`Focus`,那么请求将会变为`Other Hosts`,`https://www.baidu.com`两组
    2. 在主界面的中部的 `Filter` 栏中填入需要过滤出来的关键字。例如我们的服务器的地址是：`https://www.baidu.com` , 那么只需要在 Filter 栏中填入 `baidu.com` 即可
    
###模拟慢速网络
1. 开启慢速网络
    
    `菜单栏` -> `Proxy` -> `Throttle Setting` -> `Enable Throttling` ,可以设置 `Throttle Preset` 的类型
2. 如果只想模拟指定网站的慢速网络，可以再勾选上图中的 `Only for selected hosts` 项，然后在对话框的下半部分设置中增加指定的 hosts 项即可。


##文章借鉴:

1. [Charles 从入门到精通](http://blog.devtang.com/2015/11/14/charles-introduction/)