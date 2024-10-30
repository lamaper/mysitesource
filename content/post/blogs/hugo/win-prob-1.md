+++
date = '2024-09-27T22:43:00+08:00'
draft = true
title = '电脑疑难杂症解决办法（1）'
tags = ['blogs', 'hugo']
categories = ['blogs']
+++

lamaper

我的电脑经常出现一些莫名其妙的问题，有些解决方法值得复刻，可能什么时候就会用到，特意记录一下

### Q：Windows11下运行Photoshop2022大概率闪退

疑似是兼容性的问题，解决办法是在`%Appdata%\Adobe\Adobe Photoshop 2020\Adobe Photoshop 2020 Settings`下创建或修改`PSUserConfig.txt`为

``````
EnableDocumentGroup 0
UXPLearnAndSearch 0
``````

然后右键快捷方式图标，选择兼容性Windows8，再以管理员模式启动，即可解决问题。[2024-8-22]

参考文献：[ps2022一用魔棒就闪退_360问答 (so.com)](https://wenda.so.com/q/1677315181211439)

[解决Ps点击魔法棒功能闪退，AdobePhotoshop2022魔法棒功能无法使用，Photoshop魔法棒无法使用_ps2022使用魔棒闪退资源-CSDN文库](https://download.csdn.net/download/lanbingwang/89080951?utm_source=bbsseo)

### Ｑ：如何删除“设备与驱动器”中的百度网盘／迅雷网盘的图标

注册表：

```
"计算机\HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\MyComputer\NameSpace\"
```

删除相应字段

### Q：如何将windows11的默认输入语言改为英文

在`Win+I`进入设置窗口后，进入`时间和语言\语言和区域`，点击`添加语言`，选择`English（美国）`，安装。

然后进入`时间和语言\输入\高级键盘设置`，替换默认输入法即可。

### Q：如何将windows11切换输入法的快捷键改为Shift+Ctrl

进入`时间和语言\输入\高级键盘设置`，点击蓝字`输入语言热键`，选择`在输入语言之间`，选择快捷键即可。

### Q：windows11下通过格式工厂转换图片闪退

同理photoshop闪退，使用管理员模式运行即可。