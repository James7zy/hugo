+++
date = '2025-11-18T16:31:01+08:00'
draft = false 
title = 'vscode gunGlobal配置'
categories = ['ide']
+++

# vscode 支持gtags快速跳转
在vscode中找到并比编辑配置json文件
本地配置记录
```
{
        "editor.fontSize": 20,
        "editor.accessibilityPageSize": 20,
        "window.zoomLeve": 5,
        "workbench.editor.enablePreview": false,
        "gnuGlobal.globalExecutable": "C:\\GUNGlobal\\bin\\global.exe",
        "gnuGlobal.gtagsExecutable": "C:\\GUNGlobal\\bin\\gtags.exe",
        "graphviz-preview.dotPath": "D:\\Software\\Graphviz\\bin\\dot.exe",
        "workbench.colorTheme": "Solarized Light",
        "workbench.settings.useSplitJSON": true,
        "security.workspace.trust.untrustedFiles": "open",
        "git.enableSmartCommit": true,
        "gnuGlobal.encoding": ""
    
}
```
搜索指针的情况：
```
grep -r "\->"中间要加一个转义。双引号后面有一个\转义字符。
```

远程配置：

```shell
{
    "gnuGlobal.encoding": "utf-8",
    "gnuGlobal.globalExecutable": "/usr/bin/global",
    "gnuGlobal.gtagsExecutable": "/usr/bin/gtags",
}
```

