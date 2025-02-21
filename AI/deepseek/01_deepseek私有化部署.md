### deepseek 私有化部署

> 使用https://ollama.com/ 对deepseek 进行私有化部署

### 部署步骤

1、 准备私有化ollama 环境,使用docker 进行运行环境拉取和镜像部署

```bash
$ docker pull ollama/ollama:0.5.12-rc1

$ docker run -d -v ${PWD}/ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama:0.5.12-rc1
```

2、运行对应版本的deepseek ，具体版本可以参[https://ollama.com/library/deepseek-r1:8b](https://ollama.com/library/deepseek-r1:8b)



