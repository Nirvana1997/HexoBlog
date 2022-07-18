---
title: Godot的return和pass
category:
  - Godot
tags:
  - Godot
date: 2022-07-09 14:31:04
---

使用godot中经常遇到一些问题，官方文档有时也找不到，国内网站甚至也很难搜索，所以查到一些问题就记录一哈~
<!-- more -->

## return和pass

每次godot新建一个gd文件，会自动生成一些函数，比如：

```gdscript
func _onready():
	pass
```

像这样把函数列出来并用一个pass作为函数体。

起初我以为`pass`等于`return`，官方文档对语法也没有详细说明，但今天才发现他们不一样。

godot中函数体是不能为空的，否则会报错，`pass`这个关键词只是为了防止报错。因此，在有其他代码的情况下，pass是不生效的：

```gdscript
func my_func():
	pass
	var x = 1 # 这行代码是会正常执行的
```

### 参考资料

* [Difference between PASS and RETURN](https://godotengine.org/qa/19110/difference-between-pass-and-return)
