---

layout: post
title: 'Python JSON 读写中文处理'
date: '2016-2-17'
header-img: "img/post-bg-web.jpg"
tags:
     - python
     - json
     - 中文编码
author: 'Bill Quan'

---

## Python JSON  中文编码问题

在python文件最上面可以修改文件的编码格式，改为utf-8（如下）可以解决大部分在文件中的中文处理问题。

	# coding = utf-8

然而今天遇到一个问题，用python往json文件中写中文时，写下来的文件中，中文全变成了类似ascii码（如下）

	{"author": "\u718a\u4ed4\u997c", "desc": ""}

	Python code：
	line = json.dumps(dict(item) + "\n"
	with open('items.json', 'a') as f:
    	f.write(line)
    f.close()

经网上查找资料后，解决，特此记录。
dump函数有个缺省参数ensure_ascii默认为true，此参数为true时，所有在输出中的非ASCII字符都将转为 ascii字符。调用dump或dumps处理带中文的json文件时，须将此参数设为False，方可解决问题。正确代码如下：

	line = json.dumps(dict(item), ensure_ascii = False) + "\n"
    with open('items.json', 'a') as f:
        f.write(line)
    f.close()

####附录：dump函数相关解释
dump (obj, fp, skipkeys=False, **ensure_ascii=True,** check_circular=True, allow_nan=True, cls=None, indent=None, separators=None, encoding='utf-8', default=None, sort_keys=False, **kw)

 If ``ensure_ascii`` is true (the default), **all non-ASCII characters in the output are escaped with ``\uXXXX`` sequences, and the result is a ``str`` instance consisting of ASCII characters only**.  If ``ensure_ascii`` is ``False``, some chunks written to ``fp`` may be ``unicode`` instances. This usually happens because the input contains unicode strings or the ``encoding`` parameter is used. Unless ``fp.write()`` explicitly understands ``unicode`` (as in ``codecs.getwriter``) this is likely to cause an error.
 
 ensure_ascii为真（默认）时：所有在输出中的非ASCII字符都将转为 ascii字符 （也就是说结果是1个只包含ASCII字符的str实例） ，如果不能正确转码（中文之类的）则 抛出异常！
 ensure_ascii为假时：一些写入fp的块可能为Unicode实例。这通常是因为输入包含Unicode字符串或编码参数设置所致。除非“fp.write()清楚地知道这是Unicode字符串（如同codecs.getwriter），否则可能会导致错误。



<br>

> 如有任何知识产权、版权问题或理论错误，还请指正。
>
> 转载请注明原作者及以上信息。
