## kubectl 命令自动补全

kubectl 默认安装之后 不带自动补全的, 如果要使用自动补全, 配置方案如下

### 配置步骤

1、 安装bash-completion(Rocky linux 9)

```bash
$ dnf install bash-completion -y
```

2、 设置配置

```bash
$ source <(kubectl completion bash)

$ echo "source <(kubectl completion bash)" >> ~/.bashrc
```

