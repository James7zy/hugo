+++
date = '2025-11-18T16:31:01+08:00'
draft = false 
title = 'NVIM 增加gtags'
categories = ['ide']
+++

**NVIM 增加：**

编译：nvim

```
make CMAKE_BUILD_TYPE=RelWithDebInfo
sudo make install
```



编译global!! TO DO ..

```
TO DO ...
```

​		符号索引是个重要功能，不论阅读新项目，还是开发复杂点的大项目，符号索引都能帮你迅速掌握项目脉络，加快开发进度。传统 ctags 系统虽和 vim 结合紧密，但只能查定义无法查引用，cscope 能查引用，但只支持 C 语言，C++都不支持，况且常年不更新。ctags 由于使用文本格式存储数据，虽用了二分查找，但打开 Linux Kernel 这样的大项目时，查询会有卡顿的感觉。

[GTags](https://link.zhihu.com/?target=https%3A//www.gnu.org/software/global/) （或者叫做 GNU GLOBAL）比起 ctags 来说，有几个主要的优点：

1. 不但能查定义，还能查引用
2. 原生支持 6 种语言（C，C++，Java，PHP4，Yacc，汇编）
3. 扩展支持 50+ 种语言（包括 go/rust/scala 等，基本覆盖所有主流语言）
4. 使用性能更好的本地数据库存储符号，而不是 ctags 那种普通文本文件
5. 支持增量更新，每次只索引改变过的文件
6. 多种输出格式，能更好的同编辑器相集成

曾经用过 gtags 的人或许会问，gtags 我都用过好几年了，也没见用出朵花来啊？ 现在不管是 vscode 还是 sublime text 或者 emacs ，不都有 gtags 插件了么，要用简单得很，还有比较好的用户体验，你在 vim 下配置半天图啥呢？

答案是，如果还停留在这些传统体验上，那我也没必要写这篇文章了。现实中，大部分人都没有用对 gtags，如果你能够在 Vim 下正确使用 gtags，不但能极大的方便你开发复杂项目或者阅读新项目代码，还能获得比上面所有编辑器更好的体验。

Vim中的符号索引，他真能玩出花来，接下来本文将一步步教你 DIY 一套超越市面上任何编辑器（vscode，emacs，vscode）体验的最强静态符号索引系统。



**正确安装 GTags**

请首先安装最新版本 gtags，目前版本是 6.6.2，Windows 下可到 [[这里\]](https://link.zhihu.com/?target=http%3A//adoxa.altervista.org/global/) 下载可执行，Linux 下请自行编译最新版（Debian / Ubuntu 自带的都太老了），Mac 下检查下 brew 安装的版本至少不要低于 6.6.0 ，否则请自己编译。

只写 C/C++/Java 的话，那么到这里就够了，gtags 原生支持。如想要更多语言，那么 gtags 是支持使用 ctags/universal-ctags 或者 pygments 来作为分析前端支持 50+ 种语言。使用 ctags/universal-ctags 作为前端只能生成定义索引不能生成引用索引，因此我们要安装 pygments ，保证你的 $PATH 里面有 python，接着：

```bash
pip install pygments
```

保证 Vim 里要设置过两个环境变量才能正常工作：

```vim
let $GTAGSLABEL = 'native-pygments'
let $GTAGSCONF = '/path/to/share/gtags/gtags.conf'
```

第一个 GTAGSLABEL 告诉 gtags 默认 C/C++/Java 等六种原生支持的代码直接使用 gtags 本地分析器，而其他语言使用 pygments 模块。

第二个环境变量必须设置，否则会找不到 native-pygments 和 language map 的定义， Windows 下面在 gtags/share/gtags/gtags.conf，Linux 下要到 /usr/local/share/gtags 里找，也可以把它拷贝成 ~/.globalrc ，Vim 配置的时候方便点。

实际使用 pygments 时，gtags 会启动 python 运行名为 pygments_parser.py 的脚本，通过管道和它通信，完成源代码分析，故需保证 gtags 能在 $PATH 里调用 python，且这个 python 安装了 pygments 模块。

正确安装后，可以通过命令行 gtags 命令和 global 进行测试，注意shell 下设置环境变量。



**自动生成 Gtags**

VSCode 中的 C++ Intellisense 插件就是使用 Gtags 来提供 intellisense 的，但是它有两个非常不好用的地方：

- 代码修改了需要自己手动去运行 gtags ，更新符号索引
- 会在代码目录下生成：GTAGS，GRTAGS，GPATH 三个文件，污染我的项目目录

第一条是我过去几次使用 gtags 最头疼的一个问题；第二条也蛋疼，碍眼不说，有时不小心就把三个文件提交到代码仓库里了，极端讨厌。

那么 Vim 8 下有无更优雅的方式，自动打点好 gtags 三个文件，放到一个统一的地方，并且文件更新了自动帮我更新数据，让我根本体验不倒 gtags 的这些负担，完全流畅的使用 gtags 的各种功能呢？

当然有，使用我在《[韦易笑：如何在 Linux 下利用 Vim 搭建 C/C++ 开发环境?](https://www.zhihu.com/question/47691414/answer/373700711)》中介绍过的 [gutentags](https://link.zhihu.com/?target=https%3A//github.com/ludovicchabant/vim-gutentags) 插件来打理，它不但能根据文件改动自动生成 ctags 数据，还能帮我们自动更新 gtags 数据，稍微扩充一下上文的配置，让 gutentags 同时支持 ctags/gtags：

```vim
" gutentags 搜索工程目录的标志，当前文件路径向上递归直到碰到这些文件/目录名
let g:gutentags_project_root = ['.root', '.svn', '.git', '.hg', '.project']

" 所生成的数据文件的名称
let g:gutentags_ctags_tagfile = '.tags'

" 同时开启 ctags 和 gtags 支持：
let g:gutentags_modules = []
if executable('ctags')
	let g:gutentags_modules += ['ctags']
endif
if executable('gtags-cscope') && executable('gtags')
	let g:gutentags_modules += ['gtags_cscope']
endif

" 将自动生成的 ctags/gtags 文件全部放入 ~/.cache/tags 目录中，避免污染工程目录
let g:gutentags_cache_dir = expand('~/.cache/tags')

" 配置 ctags 的参数，老的 Exuberant-ctags 不能有 --extra=+q，注意
let g:gutentags_ctags_extra_args = ['--fields=+niazS', '--extra=+q']
let g:gutentags_ctags_extra_args += ['--c++-kinds=+px']
let g:gutentags_ctags_extra_args += ['--c-kinds=+px']

" 如果使用 universal ctags 需要增加下面一行，老的 Exuberant-ctags 不能加下一行
let g:gutentags_ctags_extra_args += ['--output-format=e-ctags']

" 禁用 gutentags 自动加载 gtags 数据库的行为
let g:gutentags_auto_add_gtags_cscope = 0
```

通过上面的配置，可以在后台自动打理 ctags 和 gtags 数据库，检测文件改动，并更新到 ~/.cache/tags 目录中，避免污染你的项目目录。

上面定义了项目标志文件（.git, .svn, .root, .project, .hg），gutentags 需要确定当前文件所属的项目目录，会从当前文件所在目录开始向父目录递归，直到找到这些标志文件。如果没有，则 gutentags 认为该文件是个野文件，不会帮它生成 ctags/gtags 数据，这也很合理，所以如果你的项目不在 svn/git/hg 仓库中的话，可以在项目根目录 touch 一个空的名为 .root 的文件即可。

现在我们在 Vim 中随便编辑文件，gtags 数据库就会默默的帮我们生成好了，如果你使用 airline ，还能再 airline 上看到生成索引的状态。gtags 程序包里有个 gtags-cscope 的程序，可用 cscope 的接口来为 Vim 提供 cscope 的所有操作，只需要再 vim 中修改一下 cscopeprg 指向这个 gtags-cscope 程序，就可以 cs add 添加 gtags 数据库，然后像 cscope一样的使用 gtags 了。

Vim 里原有的 cscope 机制可以设定好数据文件后，启动一个 cscope 进程并用管道和其链接，通过管道命令实现定义和引用的查询，你修改了 cscopeprg 指向 gtags-cscope 后，就可以在 Vim 中用 :cs add path 命令启动 gtags-cscope 这个子进程，链接 gtags 的数据库，然后提供全套 cscope 类似的操作。

gtags-cscope 还有一个优点就是我后台更新了 gtags 数据库，不需要像 cscope 一样调用 cs reset 重启 cscope 子进程，gtags-cscope 一旦连上永远不用重启，不管你啥时候更新数据库，gtags-cscope 进程都能随时查找最新的符号。

那么最后临门一脚，我们将要想办法避免这个手工 cs add 的过程。



**数据库自动切换**

gutentags 可以为我们自动 cs add 命令添加当前更新好的 gtags 数据库到 vim ，但是你编辑一个项目还好，如果你同时编辑两个以上的项目，gutentags 会把两个数据库都连接到 vim 里，于是你搜索一个符号，两个项目的结果都会同时出现，基本没法用了。

所以上面的配置中禁用了 gutentags 自动加载，我们可以每次查询单独执行一遍外部的 gtags-cscope 工具，将结果放到 quickfix。这样做可以避免项目之间结果混在一起，启动前配好项目目录和数据库目录，查询完就退出，稍微封装下即可，唯一问题就是用起来有点慢。

更好的方法是继续使用 vim 自带 cscope 系统，并解决好数据库链接断开问题：首先要能找到当前文件所属项目的 gtags 数据库被 gutentags 放到哪里了，其次一开始用不着 cs add 加载任何 gtags 数据库，只有在真正查询前增加个检测：

1. 如果当前项目数据库已经添加过，就继续开始查询工作。
2. 没有添加的话就断开其他所有项目的 gtags 数据库，再添加本项目数据库。

过程说起来很复杂，用起来却很简单，我写了个 [gutentags_plus.vim](https://link.zhihu.com/?target=https%3A//github.com/skywind3000/gutentags_plus) 的小脚本做这个事，直接用里面的 GscopeFind 命令，像 cs find 一样用就行了。

搭配 gutentags，这个脚本在你每次 GscopeFind 前帮你处理数据库加载问题，已经加载过的数据库不会重复加载，非本项目的数据库会得到即时清理，所以你根本感觉不到 gtags 的存在，只管始用 GscopeFind g 命令查找定义，GscopeFind s 命令查找引用，既不用 care gtags 数据库加载问题更不用关心何时更新，你只管写你的代码，打开你要阅读的项目，随时都能通过 GscopeFind 查询最新结果，并放入 quickfix 窗口中：

![img](https://pic3.zhimg.com/v2-f0cbbf0efe6bfb33d5a0544973757d5e_1440w.jpg)

这个小脚本末尾还还定义了一系列快捷键：

- <leader>cg - 查看光标下符号的定义
- <leader>cs - 查看光标下符号的引用
- <leader>cc - 查看有哪些函数调用了该函数
- <leader>cf - 查找光标下的文件
- <leader>ci - 查找哪些文件 include 了本文件

比如打开 Linux 代码树，memory.c 光标停留在 core_initcall 函数名上面，然后 <leader>cc，下面 quickfix 窗口立马就列出了调用过该函数的位置。

得益于 gtags 的数据存储格式，再大的项目，也能给你瞬间完成查询，得益于 gtags-cscope 的接口，vim中可以对同一个项目持续服用相同的 gtags-cscope 子进程，采用管道通信，避免同项目多次查询不断的启动新进程，查询毫无卡顿。

到此为止，我们在 vim 中 DIY 了一个比 vscode 流畅得多的符号索引体验，无缝结合 gtags 的程度超过以往任何编辑器，让你象在 IDE 里一样毫无负担的查找定义和引用，而IDE 只支持一两种语言，咱们起步就覆盖 50+ 种语言。



**快速预览**

我们从新项目仓库里查询了一个符号的引用，gtags噼里啪啦的给了你二十多个结果，那么多结果顺着一个个打开，查看，关闭，再打开很蛋疼，可使用 [vim-preview](https://link.zhihu.com/?target=https%3A//github.com/skywind3000/vim-preview) 插件高效的在 quickfix 中先快速预览所有结果，再有针对性的打开必要文件：

![img](https://pic4.zhimg.com/v2-9a90d56b84804fe3c5e53aa884215c51_1440w.jpg)

按照插件文档配置 keymap，就可以在quickfix中对应结果那一行，按 p键在右边打开预览窗口查看文件，多次按 p预览多个文件都会固定在右侧的预览窗口显示，不会打开新窗口或tab，更不会切走当前文件，也不用你因为预览新结果而要在文件窗口和 quickfix 窗口中来回切换，即便你想上下滚动预览窗口里的代码，也可以不用离开quickfix窗口，直接 alt+U/D 就可以在 quickfix 中遥控 preview 窗口上下滚屏了。

当你阅读完预览内容可以用大写 P 关闭预览窗口，然后正常用回车在新窗口或者tab中打开想要具体操作的文件了，这依赖 switchbuf 设置可以看vim帮助文档，不想看了 F10 关闭 quickfix 窗口就是。

搭配前文介绍过的 [vim-unimpaired](https://zhida.zhihu.com/search?content_id=6702468&content_type=Article&match_order=1&q=vim-unimpaired&zhida_source=entity) 插件，你还可以在不操作 quickfix窗口的情况下，使用快捷键进行上下结果跳转，Vim的好处在于有比较多的标准基础组件，比如 quickfix，emacs 就没有这样的基础设施，虽然 elisp 都可以实现，每个插件各自实现了一个差不多的 quickfix 窗口，碎片化严重，无法像vim那样一些插件往 quickfix里填充数据，一些插件提供 quickfix 方便的预览和跳转，还有一些插件可以根据quickfix里的结果内容做进一步的处理和响应，他们搭配在一起能够形成合力，这在碎片化严重的 emacs 里是看不到的。

通过上面的一系列 DIY，我们陆续解决了：按需自动索引，数据库自动连接以及结果快速预览等以往使用 gtags 的痛点问题，反观其他编辑器，符号索引功能或多或少都有这样那样不如意的地方。

所以如果你想得到这样一个其他编辑器从没达到过的IDE级别的符号索引系统，又能支持比IDE更多语言，那么花点时间DIY 一下也是值得的。



接下来我们谈 Language Server：

[韦易笑：Vim 8 中 C/C++ 符号索引：LSP 篇](https://zhuanlan.zhihu.com/p/37290578)



\----

**错误排查**：gutentags: gutentags: gtags-cscope job failed, returned: 1

这说明 gtags 在生成数据时出错了

第一步：判断 gtags 为何失败，需进一步打开日志，查看 gtags 的错误输出：

```text
let g:gutentags_define_advanced_commands = 1
```

先在 vimrc 中添加上面这一句话，允许 gutentags 打开一些高级命令和选项。然后打开你出错的源文件，运行 “:GutentagsToggleTrace”命令打开日志，它会将 ctags/gtags 命令的输出记录在 Vim 的 message 记录里。接着保存一下当前文件，触发 gtags 数据库更新，稍等片刻你应该能看到一些讨厌的日志输出，然后当你碰到问题时在 vim 里调用 ":messages" 命令列出所有消息记录，即可看到 gtags 的错误输出，方便你定位。

第二步：禁用 pygments，将环境变量改为：

```vim
let $GTAGSLABEL='native'
```

然后调试纯 C/C++ 项目看是否工作。

第三步：恢复 pygments 设置，并在项目根目录命令行运行：

```bash
$ export GTAGSLABEL=native-pygments
$ gtags
```

看是否正常工作，如果 pygments_parser.py 报错，则修正一下，6.6.2 的 pygments_parser.py 在 Windows 下面有个文件名大小写的小 bug，需要手工更改一下：

```python
    def get_lexer_by_langmap(self, path):
        ext = os.path.splitext(path)[1]
        lang = self.langmap[ext]
        if lang:
            name = lang.lower()
            if name in LANGUAGE_ALIASES:
                name = LANGUAGE_ALIASES[name]
            lexer = pygments.lexers.get_lexer_by_name(name)
            return lexer
        return None
```

第三行有问题，项目目录中存在类似一个大写的 .Bat 文件名就会出错，前面做了大小写判断，觉得它支持，后面又没转换大小写检测，不用 pygments 就没问题，我正向官方反馈，官方修正前，需要改为：

```python
        lang = self.langmap.get(ext)
```
