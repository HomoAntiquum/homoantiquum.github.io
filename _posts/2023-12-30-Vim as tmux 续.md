---
layout: post
tags:
  - 工具
---
![效果预览](/assets/images/20231230133746.png)
这篇文章主要是关于在 Windows 平台上使用 Vim 作为终端的一些设置。

上篇文章提到：Vim 在 Windows 下会把 Ansi Color Codes 中的红蓝、黄青搞反。我们可以用以下这个脚本确认终端的 Ansi Color Palette 。

``` bash
#!/bin/bash
#
#   This file echoes a bunch of color codes to the
#   terminal to demonstrate what's available.  Each
#   line is the color code of one forground color,
#   out of 17 (default + 16 escapes), followed by a
#   test use of that color on all nine background
#   colors (default + 8 escapes).
#
#   Copied from http://tldp.org/HOWTO/Bash-Prompt-HOWTO/x329.html
T='gYw'   # The test text

printf "\n         def     40m     41m     42m     43m     44m     45m     46m     47m\n";

for FGs in '    m' '   1m' '  30m' '1;30m' '  31m' '1;31m' '  32m' \
           '1;32m' '  33m' '1;33m' '  34m' '1;34m' '  35m' '1;35m' \
           '  36m' '1;36m' '  37m' '1;37m';

  do FG=${FGs// /}
  printf " $FGs \033[$FG  $T  "

  for BG in 40m 41m 42m 43m 44m 45m 46m 47m;
    do printf "$EINS \033[$FG\033[$BG  $T  \033[0m";
  done
  echo;
done
echo
```

这是在正常终端中的效果：
![](/assets/images/20231230134400.png)

而这是在 Vim 中的效果：
![](/assets/images/20231230135018.png)

可以看到颜色确实没有设置对，我猜测的是 Vim 在 Windows 下读取终端的颜色时把顺序搞错了，可能跟平台的定义也有关系。

考虑到 Vim 的代码我读起来确实比较费力（C 代码加上很多跟标准相关的知识要求），我决定用别的方法来修复它。

第一种方法是手动设置 `g:terminal_ansi_colors` ，比如添加这样的设置在你的 .vimrc 里

``` vim
let g:terminal_ansi_colors = [
    \ '#1F2428', '#EA4A5A', '#7FF9A4', '#FFA333',
    \ '#489DFF', '#A175F1', '#0EDEFA', '#F7FAFF',
    \ '#454647', '#F97583', '#B1FFBC', '#FFC073',
    \ '#79B8FF', '#B392F0', '#79EFFF', '#DEE1E6'
    \]
```

另一种方法则是使用一个设置了 terminal colors 的 color scheme ，这也是我选择的方案。我自己用的是 [iceberg.vim](https://github.com/cocopon/iceberg.vim) 。

同时为了让 Vim 在 Windows 下也能自动切换 background ，使 color scheme 也能跟着切换亮暗色主题，我们再安装上 [vim-lumen](https://github.com/vimpostor/vim-lumen) 。

最后别忘了在 .vimrc 里打开终端里对彩色的支持

``` vim
set termguicolors
set t_Co=256
```

![](/assets/images/20231230145532.png)
可以看到这时终端里的色彩终于是正确顺序了，但是可以发现我平时使用的 [Starship](https://starship.rs/) Prompt 的颜色不太对。很明显这里它在使用终端的 Ansi 颜色，但 Starship 本身是支持真彩色的。

那么可以猜到 Vim 的终端在 Windows 上不能告诉程序终端支持真彩色，所以我又去读了读文档，发现 Vim 在 Windows 上有 WinPTY 和 ConPTY 两种类型，其中 ConPTY 是新的实验性的终端类型，只有在比较新的 Windows 10 上才能使用。

既然是新的功能，我们就来试验一下。只需要稍微更改一下启动终端时的参数：

``` vim
if has('win32')
    command! PS terminal ++close ++type=conpty pwsh
    command! PSFull terminal ++close ++type=conpty ++curwin pwsh
else
    command! PS terminal ++close pwsh
    command! PSFull terminal ++close ++curwin pwsh
endif
```

![](/assets/images/20231230150453.png)
不错！现在 Starship 在 Vim 的终端里也能显示正确的彩色了。

最后我们稍微调整一下 Windows Terminal 的 color scheme 的背景（使边框色差不可见），就可以得到一个相当漂亮且好用的 pseudo tmux 了。

``` json
"list": [
    {
        "colorScheme": {
            "dark": "Gudai Iceberg Dark", // 背景使用 iceberg.vim 颜色
            "light": "Gudai Iceberg Light" // 背景使用 iceberg.vim 颜色
        },
        "commandline": "vim -c \"PSFull\"",
        "guid": "{2eb59471-bd2f-4963-a8ac-1c5533e57797}",
        "hidden": false,
        "icon": "C:\\Program Files\\Vim\\vim90\\bitmaps\\vim.ico",
        "name": "Vim",
        "scrollbarState": "hidden",
        "startingDirectory": "%USERPROFILE%"
    }
]
```

![亮色效果](/assets/images/20231230133746.png)
![暗色效果](/assets/images/20231230153318.png)
