## brew + oh my zsh 美化mac

1、 安装 `brew` 

```bash
$ /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

安装完成之后 如果找不到brew , 执行

```bash
$ eval "$(/opt/homebrew/bin/brew shellenv)"
```

如果需要一直使用, 需要将上述配置写入到~/.zshrc

```bash
$ echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' > ~/.zshrc
```



2、安装iter2

```bash
$ brew install iterm2
```

3、安装 zsh

```bash
$ brew install zsh
```

4、安装 oh-my-zsh

```bash
$ sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

如果忘记替换的玩家，可以在安装完成后输入以下指令进行替换。

```bash
$ sudo sh -c "echo $(which zsh) >> /etc/shells"
$ chsh -s $(which zsh)
```

如果想要替换回來也可以，記得要重新启动 iTerm 2 或是重开一个新窗口：

```bash
$ chsh -s /bin/bash# 之后换回 zsh
$ chsh -s /usr/loacl/bin/zsh
```