---

layout: post
title: '使用Py2exe打包PyQt4程序到exe'
date: '2016-3-27'
header-img: "img/post-bg-web.jpg"
tags:
     - Py2exe
     - PyQt4
     - python
author: 'Bill Quan'

---


最近用python写了个小程序，使用了PyQt库写了UI，完成之后使用py2exe打包成exe，遇到了一些坑，特此记录。

### PyQt4 和 py2exe

py2exe和Qt图像库之前都有分别接触用过，并不是什么陌生的东西。py2exe是一个很方便的可以把python程序打包成exe文件的库，安装时请注意安装和你的python版本和目标操作平台（32位还是64位的Windows）相对应的版本，**64位py2exe打包生成的exe无法在32位Windows上运行**。直接用pip安装py2exe会根据你的Python版本安装相应的py2exe。
Qt图形库之前本科时候搞嵌入式的时候用的多，很出色的一个跨平台图形库，用python写起来也还算顺手。

py2exe的setup.py文件

	# mysetup.py
	from distutils.core import setup
	import py2exe
	import sys
	
	#this allows to run it with a simple double click.
	sys.argv.append('py2exe')
	 
	py2exe_options = {
	        "includes": ["sip"],
	        "dll_excludes": ["MSVCP90.dll",],
	        "compressed": 1,
	        "optimize": 2,
	        "ascii": 0,
	        "bundle_files": 1,
	        }
	 
	setup(
	      name = 'ShanbayHelper',
	      version = '1.0',
	      windows = ['run.py',], 
	      zipfile = None,
	      options = {'py2exe': py2exe_options}
	      )
	
	
	#command-line exe
	#setup(console = ["Process_Management.py"])
	
	#nono command-line windows exe
	#setup(windows = ["run.py"], options = {'py2exe':{'includes':'sip'}})
	
	# execute cmd command : python setup.py py2exe

sys.argv.append('py2exe')一行，是允许程序通过双击的形式执行。

选项中“includes”是需要包含的文件，这里的”sip”是PyQt程序打包时需要添加的，如果不是PyQt程序不需要此项。

“dll_excludes”是需要排除的dll文件，这里的”MSVCP90.dll”文件，如果不排除的话会报error: MSVCP90.dll: No such file or directory错误。

“compressed”为1，则压缩文件。

“optimize”为优化级别，默认为0。

“ascii”指不自动包含encodings和codecs。

“bundle_files”是指将程序打包成单文件（此时除了exe文件外，还会生成一个zip文件。如果不需要zip文件，还需要设置zipfile = None）。1表示pyd和dll文件会被打包到单文件中，且不能从文件系统中加载python模块；值为2表示pyd和dll文件会被打包到单文件中，但是可以从文件系统中加载python模块。64位的Py2exe不要添加本句。

windows = ['pyqtdemo.py',]，这里使用的是windows，即没有命令行窗口出现，如果使用console则表示有命令行窗口出现。

执行该文件，会得到一个build文件夹和一个dist文件夹。其中，dist文件夹，就是你得到的打包程序。

### 遇到的几个问题

1. 编译时报缺失sip module错误

	需在setup.py中setup option中include sip，具体见上面的setup.py源码

2. 运行时报错Could not find or load the Qt platform plugin "windows"
	
	这是缺失了Qt的平台插件，缺少的这个插件在PyQt安装目录下**\plugins\platforms\qwindows.dll**文件夹中可以找到，将qwindows.dll拷贝到编译好的dist文件夹中，放在目录**platforms\qwindows.dll**,解决报错

3. 程序图标打包后不显示
	
	将“Python\Lib\site-packages\PyQt4\plugins\imageformats”文件夹复制到dist目录下，图标图像文件放到指定位置，解决，无须重新打包。



<br>

> 如有任何知识产权、版权问题或理论错误，还请指正。
>
> 转载请注明原作者及以上信息。
