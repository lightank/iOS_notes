# 解决Mac和iOS模拟器间拷贝粘贴的问题

[原文链接](https://www.jianshu.com/p/833b06b38692)

## 从Mac OS拷贝到iOS模拟器：

1. 将要Mac上的内容拷贝到Mac粘贴板上（可以使用`Command + C`，或者鼠标右键，点击复制）

2. 将窗口切换到iOS模拟器，然后使用Mac上键盘点击：`Command + V`（这一步操作是将Mac剪贴板上的内容拷贝到iOS模拟器的剪贴板上），然后在iOS模拟器上使用菜单选项（在输入框中，长按就会出来一个黑色的菜单条，如下图所示）中的粘贴（Paste）进行粘贴。

    ![](https://github.com/lightank/noteArticles/blob/master/Resource/Paste.png)
    
    完整操作如下：
    
    ![](https://github.com/lightank/noteArticles/blob/master/Resource/copy_Mac_to_Simulator.gif)

## 从iOS模拟器拷贝到Mac OS:

1. 在iOS模拟器中，使用菜单选项拷贝内容（不能使用`Command + C`），然后使用Mac上键盘点击：`Command + C`（这一步操作是将iOS模拟器剪贴板上的内容拷贝到Mac OS的剪贴板上）

2. 将窗口切换到非iOS模拟器窗口，就可以直接按照正常的粘贴方式（`Commond + V`，或者鼠标右键，点击粘贴），将iOS模拟器上拷贝的内容粘贴到Mac OS上了。

    完整操作如下：
    
    ![](https://github.com/lightank/noteArticles/blob/master/Resource/copy_Simulator_to_Mac.gif)