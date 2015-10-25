<!--
Title|Cmder简单使用小结
Id|cmder-simple-use
Date|2015-09-10 20:33:00
Status|Publish
Type|Post
Tags|tools,tech
Excerpt|Cmder是一款Windows环境下非常简洁美观易用的cmd替代者，它支持了大部分的Linux命令。
-->
[Cmder](http://bliker.github.io/cmder/)是一款Windows环境下非常简洁美观易用的cmd替代者，它支持了大部分的Linux命令。

从官网下载下来一个zip安装包，解压之后运行根目录的Cmder.exe即可。但是此时会有两个问题，一是`ls`命令不支持中文，二就是中文提示会有字体重叠现象。

###1、解决中文乱码问题
把一下几行代码添加到config/aliases文件末尾即可解决中文乱码问题：

    l=ls --show-control-chars 
	la=ls -aF --show-control-chars 
	ll=ls -alF --show-control-chars
	ls=ls --show-control-chars -F
###2、解决文字重叠问题
`Win + Ait + P` 唤出设置界面 > mian > font > monospce 的勾勾去掉(点两下).
###3、配置其在win+r中打开
把根目录加到系统环境的path变量中即可。
###4、添加右键
可以关注这个[gist](https://gist.github.com/unmric/8067104)。在Cmder根目录新建一个`init.bat`，输入以下代码：

    @echo off
	SET CMDER_ROOT=%~dp0
	reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Directory\Background\shell\Cmder" /ve /d "Cmder Here" /f
	reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Directory\Background\shell\Cmder" /v "Icon" /d "\"%CMDER_ROOT%cmder.exe\"" /f
	reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Directory\Background\shell\Cmder" /v "Extended" /f
	reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Directory\Background\shell\Cmder\command" /ve /d "\"%CMDER_ROOT%cmder.exe\" \"%%V\"" /f
	pause
以管理员身份运行init.bat即可。删除的话再在根目录新建一个`uninit.bat`，依然是以管理员身份运行。代码如下：

    @echo off
	Reg delete "HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Directory\Background\shell\Cmder" /f
	pause
###5、alias设置
在 Cmder 的 config 文件夹中有一个叫 aliases 的文件它是专门设置 alias 的。当然它不同于 alias 那么死板, 其中有一个参数 `$*` 它等同于命令参数的其他部分。
example1:
 ls --color $* 在执行 ls 的时候就等于在他前面添加了 --color.
 example2:
 假设你有一个vps，你可以设置一个快速链接你vps的命令，在config/aliases文件末尾加这个一行即可： 
 
    sshvps=ssh -p 22 username@x.x.x.x
###6、添加快捷键
右键 cmder.exe > 创建快捷方式 > 右键快捷方式 > 点击快捷键项 > 按 Ctrl + Alt + T.
以后按 `Ctrl + Alt + T` 的时候就会运行 Cmder 了.
###7、[Chocolatey](http://chocolatey.org/)软件包管理系统
安装chocolatey：

    @powershell -NoProfile -ExecutionPolicy unrestricted -Command "iex ((new-object net.webclient).DownloadString('https://chocolatey.org/install.ps1'))" && SET PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin
安装完之后，想使用再想安装ruby，只需在cmder里执行：

    choco install ruby
