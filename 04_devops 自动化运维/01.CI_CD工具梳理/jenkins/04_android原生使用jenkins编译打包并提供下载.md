## Android 原生+gitlab+jenkins 实现流水线构建

### 1. 背景

1.1 为了保证安卓原生的自动化构建，实现apk包自动化构建、测试和发版一体化，提高工作效率，这里使用jenkins 流水线对安卓构建步骤进行自动化。



1.2 环境依赖

1. jenkins 
2. gitlab
3. ossfs 
4. nginx
5. gradle = 8.14.3
6. jdk21

1.3 实现方案架构



### 2. 具体实现步骤

#### 2.1 实现gitlab 指定分支提交代码触发流水线

1、 `jenkins` 安装 `Generic Webhook Trigger` 插件，用于支撑gitlab 触发jenkins pipeline 的自动化构建，另外插件也同样支持gitlab，github，云效等代码仓。

安装步骤 :

 登录jenkins > 系统管理 > 插件管理 > Installed plugins > 搜索Generic Webhook Trigger

![image-20251121102204802](./04_android%E5%8E%9F%E7%94%9F%E4%BD%BF%E7%94%A8jenkins%E7%BC%96%E8%AF%91%E6%89%93%E5%8C%85%E5%B9%B6%E6%8F%90%E4%BE%9B%E4%B8%8B%E8%BD%BD.assets/image-20251121102204802.png)

这里我的jenkins 已经安装完成，后续根据实际情况选择安装即可，安装完成后可在新建流水线中看到具体插件选项

![image-20251121102600930](./04_android%E5%8E%9F%E7%94%9F%E4%BD%BF%E7%94%A8jenkins%E7%BC%96%E8%AF%91%E6%89%93%E5%8C%85%E5%B9%B6%E6%8F%90%E4%BE%9B%E4%B8%8B%E8%BD%BD.assets/image-20251121102600930.png)



2、 配置 Generic Webhook Trigger 【后续以插件代替名称】，选中插件后，会显示gitlab 配置的地址等内容，

![image-20251121102845438](./04_android%E5%8E%9F%E7%94%9F%E4%BD%BF%E7%94%A8jenkins%E7%BC%96%E8%AF%91%E6%89%93%E5%8C%85%E5%B9%B6%E6%8F%90%E4%BE%9B%E4%B8%8B%E8%BD%BD.assets/image-20251121102845438.png)

![image-20251121102942990](./04_android%E5%8E%9F%E7%94%9F%E4%BD%BF%E7%94%A8jenkins%E7%BC%96%E8%AF%91%E6%89%93%E5%8C%85%E5%B9%B6%E6%8F%90%E4%BE%9B%E4%B8%8B%E8%BD%BD.assets/image-20251121102942990.png)

这里将Token 配置为 `devtest`，JENKINS_URL 为jenkins.cmzhu.cn;配置链接为：[https://jenkins.cmzhu.cn/generic-webhook-trigger/invoke?token=devtest](https://jenkins.cmzhu.cn/generic-webhook-trigger/invoke?token=devtest)

3、 在gitlab 项目中将上述提到的webhooks 连接配置在gitlab 仓库中，在此处添加新的webhooks 链接

![image-20251121111032673](./04_android%E5%8E%9F%E7%94%9F%E4%BD%BF%E7%94%A8jenkins%E7%BC%96%E8%AF%91%E6%89%93%E5%8C%85%E5%B9%B6%E6%8F%90%E4%BE%9B%E4%B8%8B%E8%BD%BD.assets/image-20251121111032673.png)

2、 webhooks 配置如下，配置完成后用于测试是否正常触发流水线

![image-20251121111332286](./04_android%E5%8E%9F%E7%94%9F%E4%BD%BF%E7%94%A8jenkins%E7%BC%96%E8%AF%91%E6%89%93%E5%8C%85%E5%B9%B6%E6%8F%90%E4%BE%9B%E4%B8%8B%E8%BD%BD.assets/image-20251121111332286.png)

3、 从请求中可以查看相应的post 信息，用于配置变量；

![image-20251121112145013](./04_android%E5%8E%9F%E7%94%9F%E4%BD%BF%E7%94%A8jenkins%E7%BC%96%E8%AF%91%E6%89%93%E5%8C%85%E5%B9%B6%E6%8F%90%E4%BE%9B%E4%B8%8B%E8%BD%BD.assets/image-20251121112145013.png)

并且这里发现代码分支的变量为： 

![image-20251121112308165](./04_android%E5%8E%9F%E7%94%9F%E4%BD%BF%E7%94%A8jenkins%E7%BC%96%E8%AF%91%E6%89%93%E5%8C%85%E5%B9%B6%E6%8F%90%E4%BE%9B%E4%B8%8B%E8%BD%BD.assets/image-20251121112308165.png)

#### 2.2 jenkins 自动拉取仓库代码并打包编译

 根据2.1 中最后的结果，可以按照实际情况获取gitlab 调用webhooks 时传给jenkins 的变量，来做对应自动化操作

1、 从上图中可以看到分支为ref 变量，这里在插件中添加变量 branch ，用于获取分支信息

![image-20251121114206077](./04_android%E5%8E%9F%E7%94%9F%E4%BD%BF%E7%94%A8jenkins%E7%BC%96%E8%AF%91%E6%89%93%E5%8C%85%E5%B9%B6%E6%8F%90%E4%BE%9B%E4%B8%8B%E8%BD%BD.assets/image-20251121114206077.png)

使用变量 alldata 用于获取所有json 变量

![image-20251121114231348](./04_android%E5%8E%9F%E7%94%9F%E4%BD%BF%E7%94%A8jenkins%E7%BC%96%E8%AF%91%E6%89%93%E5%8C%85%E5%B9%B6%E6%8F%90%E4%BE%9B%E4%B8%8B%E8%BD%BD.assets/image-20251121114231348.png)

2、流水线中勾选这两个参数，可在日志中打印对应的变量信息

![image-20251121114556094](./04_android%E5%8E%9F%E7%94%9F%E4%BD%BF%E7%94%A8jenkins%E7%BC%96%E8%AF%91%E6%89%93%E5%8C%85%E5%B9%B6%E6%8F%90%E4%BE%9B%E4%B8%8B%E8%BD%BD.assets/image-20251121114556094.png)

3、选择指定分支进行分支构建,并且相应的gitlab 也可以同样选择分支进行覆盖。

![image-20251121114654969](./04_android%E5%8E%9F%E7%94%9F%E4%BD%BF%E7%94%A8jenkins%E7%BC%96%E8%AF%91%E6%89%93%E5%8C%85%E5%B9%B6%E6%8F%90%E4%BE%9B%E4%B8%8B%E8%BD%BD.assets/image-20251121114654969.png)

4、 流水线pipeline 配置步骤

**环境变量**

    environment {
        GITLAB_CRED = 'JENKINS_CREDIT'  // Gitlab凭证ID
        GITLAB_URL = '${postdata_project_git_http_url}'
        GITLAB_BRANCH='${ref}'  
    }



#### 2.3 jenkins 编译完成后将产物推送至OSS

#### 2.4 消息通知开发人员下载

 





