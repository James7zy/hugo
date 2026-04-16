# Alacritty
Windows 环境搭建Alacritty.
## Theme
```shell
# We use Alacritty's default Linux config directory as our storage location here.
mkdir -p ~/.config/alacritty/themes
git clone https://github.com/alacritty/alacritty-theme ~/.config/alacritty/themes
```

Add an import to your `alacritty.toml` (Replace `{theme}` with your desired colorscheme):

```
如果没有就添加：
C:\Users\10360\AppData\Roaming\alacritty/alacritty.toml

[general]
import = [
    "~/.config/alacritty/themes/themes/{theme}.toml"
]
```

# alacritty 配置

在windows文件：C:\Users\10360\AppData\Roaming\alacritty

```
[terminal.shell]
program = "nu"

#改变窗口设置
[window]

#窗口大小
dimensions = { columns = 120, lines = 35 }

padding = { x = 4, y = 2 }
dynamic_padding = true
#透明度
opacity = 0.9
#窗口名字
title = "Alacritty"

[font]
normal = { family = "JetBrainsMono Nerd Font", style = "Regular" }

[general]
import = [
    "~/.config/alacritty/themes/themes/{alacritty_0_12}.toml"
]
```


# Starship Windows

## Prompt

 Windows增强个人用的是`Starship`，又是自带`Rust`光环，主打一个快，但奈何电脑配置不行，主要的锅还是`Windows`本身和`Powershell`

### 安装Starship

[https://starship.rs/guide/#%F0%9F%9A%80-installation](https://starship.rs/guide/#🚀-installation)

在目录下新建`C:\Users\User\Documents\PowerShell\Microsoft.PowerShell_profile.ps1`

添加以下内容

```
starship preset pure-preset
```

### 配置Prompt

选择`Pure Prompt`(个人喜好)

```
starship preset pure-preset
```

在目录下新建`C:\Users\User.config\starship.toml`

```
[line_break]
disabled = true
```

The `line_break` module separates the prompt into two lines.

I really hate two lines prompt, so I disabled it.

### 将 alacrity 注册为右键菜单启动

新建文本文件，填入如下内容，修改后缀为 .reg 运行，将配置写入注册表。

```
Windows Registry Editor Version 5.00

[HKEY_CLASSES_ROOT\Directory\Background\shell\alacritty]
@="Alacritty terminal here"
"Icon"="C:\\Tools\\Alacritty\\alacritty.ico"

[HKEY_CLASSES_ROOT\Directory\Background\shell\alacritty\command]
@="C:\\Tools\\Alacritty\\alacritty.exe"
```

当然，你也可以按照自己的喜好手动编辑注册表。
其中 Icon 这一行中使用的的图标文件可以在 GitHub 官方仓库 下载，或者也可以使用自己喜欢的内容。
你也可以将 @="Alacritty terminal here" 修改为自己喜欢的内容。

# 快速连接ssh
```
C:\Users\你的用户名\.ssh\config
```
示例配置：
```
Host myserver
    HostName 192.168.1.100
    User root
    Port 22
```
------

以后连接只需要：
```
ssh myserver
```
