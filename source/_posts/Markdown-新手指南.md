---
title: Markdown 新手指南
date: 2016-01-02 21:10:32
tags: [Markdown]
---
[Markdown语法](http://wowubuntu.com/markdown/) 看起来挺简单的，它更像是一门动态的脚本语言一样（相对于我使用Java进行开发而言，习惯于称这种为脚本语言），好吧下面我们来学习这种脚本语言。
<!-- more -->
# 标题
在 Markdown 中，只需要在文本前面加上 # 即可产生一个一级标题，同理、你还可以增加二级标题、三级标题、四级标题、五级标题和六级标题，总共六级，只需要增加  # 即可，标题字号相应降低。例如：

    # 一级标题
    ## 二级标题
    ### 三级标题
    #### 四级标题
    ##### 五级标题
    ###### 六级标题
	这里是一个代码区块，后面会讲怎么实现
	

注：# 和「标题」之间有一个字符的空格，这是最标准的 Markdown 写法.

# 链接
在 Markdown 中，插入链接只需要使用 `[显示文本](链接地址)` 这样的语法即可，例如：

    [大道自然的博客](http://iamlibo.github.io/thinker/)

会显示为
[大道自然的博客](http://iamlibo.github.io/thinker/)

# 图片
在 Markdown 中，插入图片只需要使用 `![](图片链接地址) `这样的语法即可，例如：

	![](http://iamlibo.github.io/thinker/images/bluewater13.jpg)

图片的地址可以完整的外部链接地址，也可以站内的相对地址，例如站内地址要写相对于根的地址

	![](/thinker/images/bluewater13.jpg)

# 引用
在 Markdown 中，只需要在你希望引用的文字前面加上` > `就可以，例如：

	> 往事随风去，人生无回头，万事莫强求，苦乐由我心。
	
会显示为：
> 往事随风去，人生无回头，万事莫强求，苦乐由我心。

注：`>` 和「内容」之间有一个字符的空格.

# 粗体字和斜体字
在 Markdown 中，用 `**` 包含一段文本就是粗体，用 `*` 包含一段文本就是斜体。例如：

	*往事随风去*，人生无回头，万事莫强求，**苦乐由我心**

会显示为

*往事随风去*，人生无回头，万事莫强求，**苦乐由我心**

# 表格
在 Markdown 中，显示表格会感觉很麻烦，找一段网上的例子：

	| Tables        | Are           | Cool  |
	| ------------- |:-------------:| -----:|
	| col 3 is      | right-aligned | $1600 |
	| col 2 is      | centered      |   $12 |
	| zebra stripes | are neat      |    $1 |

显示为表格

| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |

注意中间列是居中显示，右面列表是居右显示。

---



还有其他的网站比我写得更详细，相关链接

[Markdown 语法说明（简体中文）](http://wowubuntu.com/markdown/)

[简书](http://www.jianshu.com/p/q81RER)
