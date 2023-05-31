### 解决 Mac 每次打开终端都要执行source ~/.bash_profile

> 问题描述

​		在` ~/.bash_profile` 中配置环境变量可是每次重启终端后配置的不生效.需要重新执行 : `source ~/.bash_profile`
后来发现zsh加载的是 `~/.zshrc`文件，而 `.zshrc`文件中并没有定义任务环境变量（或者此文件不存在）。

>解决办法

在`~/.zshrc`（不存在则创建它）文件最后，增加一行：`source ~/.bash_profile `

创建命令：`vi  ~/.zshrc`