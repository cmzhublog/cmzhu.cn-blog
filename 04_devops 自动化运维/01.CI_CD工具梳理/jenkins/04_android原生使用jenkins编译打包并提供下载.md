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



### 2. 具体实现步骤（gradle 原生）

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

```groovy
environment {
    GITLAB_CRED = 'JENKINS_CREDIT'  // Gitlab凭证ID
    GITLAB_URL = '${postdata_project_git_http_url}' //git 仓库地址：gitlab 调用jenkins webhooks 时会提供变量
    GITLAB_BRANCH='${ref}'  //git 仓库地址：gitlab 调用jenkins webhooks 时会提供变量
}
```

**代码拉取过程**

代码拉取仓库地址由gitlab 去请求流水线时，在调用时创建参数，具体参数其他可以在流水线日志中看见。

```groovy
        stage('拉取代码') {
            steps {
                checkout([
                    $class:"GitSCM",
                    branches:[[name:"${GITLAB_BRANCH}"]],
                    doGenerateSubmoduleConfigurations: false,
                    extensions: [[
                    $class:"RelativeTargetDirectory"
                    ]],
                    gitTool: "Default",
                    submoduleCfg: [],
                    userRemoteConfigs:[[
                        credentialsId:"${GITLAB_CRED}",
                        url:"${GITLAB_URL}"
                    ]]
                ])
            }
        }
```

**编译步骤**

gradle 的编译环境依赖相对简单，只需要对应版本的gradle 版本环境，就能正常编译.

```groovy
stage('Gradle构建') {
            agent { 
                docker {
                    image 'harbor.cmzhu.cn/devops/an/android_gradle_base:latest_20251120'
                    args '-v ${workspace}:/workspace'
                    reuseNode true
                }
            }
            steps {
                sh 'gradle clean '
                // sh "gradle assembleDebug"
                sh "gradle assembleRelease"
            }
        }
```

这里使用了一个`harbor.cmzhu.cn/devops/an/android_gradle_base:latest_20251120` 镜像，它的Dockerfile 如下；

```dockerfile
FROM  harbor.cmzhu.cn/3rdimages/docker.io/library/gradle:8.14.3-jdk21

ENV ANDROID_HOME=/opt/androidhome
ENV DEBIAN_FRONTEND=noninteractive

ARG ANDROID_HOME_TMP=/opt/androidhome

RUN mkdir -p /opt/androidhome
RUN wget http://sulphuric.cmzhu.cn/commandlinetools-linux-13114758_latest.zip -O /opt/androidhome/commandlinetools.zip \
  && unzip /opt/androidhome/commandlinetools.zip -d /opt/androidhome  \
  && rm -f /opt/androidhome/commandlinetools.zip

RUN wget http://sulphuric.cmzhu.cn/platform-tools-latest-linux.zip -O /opt/androidhome/platform-tools.zip \
  && unzip /opt/androidhome/platform-tools.zip -d /opt/androidhome \
  && rm -f /opt/androidhome/platform-tools.zip

RUN yes | ${ANDROID_HOME_TMP}/cmdline-tools/bin/sdkmanager --licenses --sdk_root=${ANDROID_HOME_TMP}
```

#### 2.3 jenkins 编译完成后将产物处理

android 包的产物被打包出来后，如何给对应开发人员提供安全可靠的下载地址？ 

这里是将打包后产物通过`ossutil` 根据时间戳的方式进行包名区分，这样能将所有通过流水线的包都保存，并且可以及时回溯到相应的历史版本包上。

```groovy 
def pushOss(){
    env.DateTimeNow = getDataStamp()
    env.TimeStampNow = getTimestamp()
    env.OSS_TARGET_DIR = "oss://android-pkg-bucket/android_apk/${postdata_project_path_with_namespace}/${env.DateTimeNow}"
    //     env.BUILD_PACKAGE_DIR = "app/build/outputs/apk"
    // 打包产物所在的目录
    env.BUILD_PACKAGE_DIR = private_get_target_dir()
    env.PROJECT_NAME = postdata_project_path_with_namespace.split("/")[1]
    env.TAR_NAME = private_get_online_pkg()

    sh """
        cd ${env.BUILD_PACKAGE_DIR}
        tar -czvf ${env.TAR_NAME} ./*/*.apk
        ossutil cp ${env.TAR_NAME} ${env.OSS_TARGET_DIR}/${env.TAR_NAME} -f
        
    """
}
```

从上述流水线代码中可以看出，使用ossutil 将打包后的产物根据时间戳压缩后上传至oss 中。

> 注意： 为了保证公司内部所有同事都可以正常下载，又不用每次安装ossutils 和每一个同事提供AK/SK，特使用ossfs 将该oss 桶挂载在主机上，使用nginx 提供https 服务，供办公室同事下载

```nginx
server {
    listen 443 ssl;
    server_name android.cmzhu.cn;
    root /data/oss-sgp-android-pkg-bucket;

    ssl_certificate /etc/nginx/ssl/cmzhu.cn.crt;
    ssl_certificate_key /etc/nginx/ssl/cmzhu.cn.key;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'HIGH:!aNULL:!MD5';
    ssl_prefer_server_ciphers on;
    autoindex on;

    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;


}

server {
    listen 80;
    server_name android.cmzhu.cn;

    return 308 https://$host$request_uri;
}
```

具体ossfs 使用步骤可参考阿里云官方提供：[在Linux系统中安装ossfs 2.0](https://help.aliyun.com/zh/oss/developer-reference/install-ossfs-2-0?spm=a2c4g.11186623.help-menu-31815.d_1_2_5_0.d4013b2f28rDBq)

#### 2.4 消息通知开发人员下载

通过钉钉群机器人和飞书群机器人，将消息通知到对应群组，供同事下载

 

### 3.0 flutter 编译步骤

flutter 的流水线触发步骤，jenkins 自动拉取，编译后产物处理和消息通知都和原生基本一致。不同的点在于编译的环境和命令不同，这里着重提flutter 的编译打包过程。

```groovy
        stage('flutter构建') {
            agent { 
                docker {
                    image 'docker.io/plugfox/flutter:3.32.5-android'
                    args '-v ${workspace}:/workspace'
                    reuseNode true
                }
            }
            steps {
                sh "flutter clean"
                sh "flutter pub get"
                sh "flutter build apk --release"
            }
        }
```

> 这里使用3.32.5 版本来进行编译打包，如果后续有更新其他版本，相应的去找其他版本对应的镜像。如果公共的镜像没办法正常使用，可自定义Dockerfile 来进行编译

