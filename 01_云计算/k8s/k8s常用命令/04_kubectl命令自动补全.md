## kubectl 命令自动补全

kubectl 默认安装之后 不带自动补全的, 如果要使用自动补全, 配置方案如下

### 配置步骤

1、 安装bash-completion(Rocky linux 9)

```bash
$ dnf install bash-completion -y
```

2、 设置配置

```bash
$ source /usr/share/bash-completion/bash_completion
$ source <(kubectl completion bash)
$ echo "source <(kubectl completion bash)" >> ~/.bashrc
```

3、 如果将kubectl 设置了别名, 可以使用下面命令来补全

```bash
## 将别名设置成 k
$ echo 'alias k=kubectl' >> ~/.bashrc
## 设置 k 也支持补全
$ echo 'complete -F __start_kubectl k' >>~/.bashrc

## 配置生效
$ source ~/.bashrc

```

