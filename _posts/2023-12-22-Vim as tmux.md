---
layout: post
tags: 工具
---
说到底，我之所以选择使用 Vim ，主要还是为了能在所有平台用上统一的快捷键。不光是在 VSCode 和 CLion 之间，甚至是在 MacOS 和 Windows 上统一。毕竟整天在 Command 键和 Control 键来回倒腾还是比较让人苦恼的。

所以问题就是我该如何统一不同平台上的终端工具呢？对于 Shell 的快捷键，我的选择是统一使用 PowerShell 搭配默认预装的 PSReadLine 里的 Vi 模式，一方面是你无法在 Windows 上使用原生的 zsh，bash 等在 Unix 下常见的 Shell，另一方面则是 pwsh 的语法与绝大部分面向对象的语言类似，没那么多与字符串的挣扎。

然后便要引出今天的主题了，如何统一 Terminal Emulator 的键位呢？第一种方案想必是用一个统一的 Terminal Emulator，比如说全用 Alacritty。事实上，我一开始也是这么干的。但是后来还是遇到了一些不太满意的地方，比如说字体渲染有一些 Glitch 、Windows 端上窗口外观很丑、不能跟随系统切换 Color Scheme 等等。所以我决定就在特定平台使用最好用的那个终端，然后统一快捷键。在 MacOS 上用 iTerm 2 ，在 Windows 上用 Windows Terminal。

仔细一想，我平时会用到的 Terminal Emulator 的快捷键也无非就是新建个 Tab，分割窗口，在窗口之间切换，关闭窗口一类的，那我岂不是是装个 tmux，完全不用 Terminal Emulator 的附加功能就可以统一快捷键了吗。但是很可惜，tmux 在 Windows 上是不支持的。所以我们就该请出能一定程度上替代 tmux 的 Vim 8.0 了。

自从 Vim v8.1 版本后，Vim 便内置了 terminal 功能，结合上 Vim 自带的窗口功能，我们可以在一个 Vim 进程里运行多个 Shell 。

在终端里打开 Vim ，输入 `:ter` ，默认的 Shell 就会以分屏的方式启动。想要想平时启动终端一样用全屏也很简单，输入 `:ter++curwin` 。
![](/assets/images/20231222135803.png)

如果我们想要开一个新的 tab 然后编辑文件（别忘了我们在Vim里），我们只需要进入到命令模式（在终端模式下按 `ctrl-w ：` ），然后 `:tabe <filename>` ，我们便得到了拥有两个 tab 的终端。
![](/assets/images/20231222135628.png)

在 tab 里分屏也是轻而易举，`:vert ter` 即可。

![](/assets/images/20231222140540.png)
为了在默认情况下打开 terminal emulator 就能直接启动 vim 中的 terminal ，我们在这里以 MacOS 加 iTerm 2 为例配置一下。我们以 Ps 和 PsFull 作为启动 PowerShell 和 全屏启动 PowerShell 的命令。

``` vim
command! Ps terminal pwsh
command! PsFull terminal++curwin pwsh
```

然后我们打开 iTerm 2 ，找到设置当中的 Profiles 。
![](/assets/images/20231222141323.png)
可以看到我这里的命令是先启动的 PowerShell ，然后启动的 Vim ，最后再在 Vim 里又启动了 PowerShell 。虽然看着比较套娃，但是注意到最外面的 PowerShell 带了 -Login 的参数。

这个选项只在 unix 平台上有效，根据[文档](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_pwsh?view=powershell-7.4#-login---l)，是为了读取你的 login profiles 比如 /etc/profile 或者 ~/.profile 然后初始化用的，比如你的环境变量。如果直接执行 Vim，Vim 有可能会找不到一些 Homebrew 或者你自己安装的软件。

至于在 Windows 平台，由于环境变量不是由 Shell 的配置文件来决定，所以大家可以直接启动 Vim -c "PsFull"，不用担心初始化的问题。

这样一来在 MacOS 和 Windows 上我就都可以使用 Vim 快捷键来管理终端窗口了。不过我又发现 Vim 在 Windows 下好像会把 Ansi Color Codes 中的红蓝、黄青搞反，最近我也是在研究如何能够修复，这可能会是下一篇的主题。
