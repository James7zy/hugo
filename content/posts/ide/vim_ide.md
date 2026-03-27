
+++
date = '2025-11-18T15:43:25+08:00'
draft = false 
title = '高效使用vim'
categories = ['ide']
+++

# 高效使用VIM 
**VIM这么难用**为什么这么多人热衷

- https://www.zhihu.com/question/437735833/answer/1716873219

# VIM 配置
​    针对搜索代码关键字延迟大，<font color=red>**提高代码搜索速度。**</font>
**拷贝到自己**Home目录解压即可完成配置。之后你就可以开始折腾你的开发环境了。需要了解如何配置的参考一下。
<font color=red>如果对配置不敢兴趣，直接跳到**快乐出发**</font>

# 进阶学习use_vim_as_IDE
需要了解如何配置的全参考：https://github.com/yangyangwithgnu/use_vim_as_ide
<font color=red>**喜欢折腾系列研究：**</font>

- https://www.zhihu.com/question/47691414
- https://zhuanlan.zhihu.com/p/36279445

可以针对自己的喜好开始定制vim，自己定制了我的习惯参考：

https://github.com/James7zy/vim

# 快乐出发

​    根据自己的喜好配置.vimrc在自己的HOME目录下,所有命令的前导键是分号 ----->  ;输入分号后就可以执行自己的快捷操作！
- 调出文件树,直接打开VIM输入分号fl.
```
;fl
```
- 调出文件符号分号ilt.
```
;ilt
```
# Vim快捷键
1、vim 拷贝如何使用正确拷贝粘贴
```
删除 dd 这个会将内容
p 粘贴
高亮功能：shift +8
将所有的高亮去除，:noh
```
# 高效搜索快捷键
在你的VIM种基础搜索功能。
```
! grep -rn xxx
```
分号sp，这里的分号是在vimrc中映射的快捷键。可以根据自己的需求映射。
- 在自己的$Home .vimrc中处理做处理。
- 输入分号时不需要进入命令行模式。
```
;sp
```
## gtags搜索

- 以Linux 源码为例
Linux 源码对GTAGS支持比较好可以使用编译,提高代码匹配度。
```
make gtags
```
- 普通的工程中使用，生成GTAGS等三个文件。
```
gtags -v 
```
```
在vim 命令行中输入冒号:Gtag 之后Tab可以补全对应命令。
示例搜索：Gtags  XXXX
```
完整配置GTAGS以及跳转功能：
https://zhuanlan.zhihu.com/p/36279445
集成gtags-cscope
```shell
cs find s xxxx
```
查找更多cs命令
```
cs help
```
## 指定范围替换
```
1. 整个文件批量替换
:%s/旧文本/新文本/g

2. 指定范围内替换，替换21行-52行中的文本
:21,52s/旧文本/新文本/g
```
# 全选文件：
```
ggVG在VIM复制后就可以使用

程序之间拷贝粘贴呢？
在vim中使⽤"*y使⽤进⾏复制，然后在应⽤程序中⽤ctrl+v粘贴。

从应⽤程序到vim则在应⽤。
程序中使⽤ctrl+c复制，在vim中使⽤shift+insert粘贴。
```
# 高级复制功能：

寄存器复制问题：
```
全部删除：按esc后，然后dG
全部复制：按esc后，然后ggyG
全选⾼亮显⽰：按esc后，然后ggvG(这个好像有点问题）或者ggVG正确
vim如何与剪贴板交互（将vim的内容复制出来）

习惯了在windows环境各个应⽤程序之间如UltraEdit，记事本，eclipse之间ctrl+c,ctrl+v进⾏复制粘贴的你，如何在vim与别的windows应⽤
程序之间拷贝粘贴呢？
当然你可以在vim⾥选择⽤⿏标，选中⼀块⽂字然后右键复制，再到应⽤程序⾥ctrl+v粘贴，只不过这样效率就差多了。

更好的做法是，在vim中使⽤"*y使⽤进⾏复制，然后在应⽤程序中⽤ctrl+v粘贴。
从应⽤程序到vim则在应⽤程序中使⽤ctrl+c复制，在vim中使⽤shift+insert粘贴。
如：
"*yy复制⼀⾏
"*y2w复制⼆个词
……
实现的原理是：
"   表⽰使⽤寄存器
"*   表⽰使⽤当前选择区

我个⼈推荐使⽤ctrl+insert复制，shift+insert粘贴。
vim有多个剪贴板，其中就包括了系统剪贴板。使⽤命令:reg可以看到各个剪贴板的内容。其中“”表⽰当前使⽤的剪贴板，“0-9是历史剪贴
板，“#就是系统剪贴板了（你可以在系统⾥拷贝⼀些东西，看是不是会出现在“#剪贴板⾥）。在vim中使⽤y可以把内容拷贝到“”号剪贴板，
继续y会把新的东西放⼊“”，⽽原来“”的东西就会被压⼊“0-9的各个历史剪贴板中。X11系统下还有⼀个“*的剪贴板对应中键拷贝粘
贴，windows不知道有没有。
解决第⼀个问题：
“+y 把选中内容拷贝到”+号剪贴板，即系统剪贴板
“+p 把系统剪贴板的内容粘贴到vim，这⼀个⽤shift+insert也可完成
解决第⼆个问题：
“0p 可以把已经被挤到”0剪贴板的内容A重新粘贴出来
嫌长的做⼀个map，映射到某个功能键或组合就⽅便了。 
```

# 关闭单个标签
```
:bd
```
# VIM高亮插件vim-highlighter

使用高亮插件：  [GitHub - azabiong/vim-highlighter: Highlight words and expressions](https://github.com/azabiong/vim-highlighter)

```shell
Key Map
The plugin uses the following default key mappings which you can assign in the configuration file.
  let HiSet   = 'f<CR>'
  let HiErase = 'f<BS>'
  let HiClear = 'f<C-L>'
  let HiFind  = 'f<Tab>'
Default key mappings: f Enter, f Backspace, f Ctrl+L and f Tab

In normal mode, HiSet and HiErase keys set or erase high
lights of the word under the cursor. HiClear key clears all highlights.

The plugin supports jumping to highlights using two sets of commands.
The Hi < and Hi > commands move the cursor back and forth to recently
 highlighted words or matching highlights at the cursor position.
The Hi { and Hi } commands, on the other hand, move the cursor to 
the nearest highlighted word, and can be used to jump to adjacent
 highlights in different patterns.
```

# Happying
 痛苦使用VIM，到最后解放你的鼠标！！
VIM熟练需要一个持续学习的过程，可以参考推荐书籍。

推荐书籍：[Vim实用技巧-Drew Neil-微信读书](https://weread.qq.com/web/bookDetail/ce132a905b207dce166506f)

