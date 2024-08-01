## jenkins 对接gitlab 设置流水线

jenkins 插件市场比较丰富， 绝大部分市场都使用的jenkins进行流水线操作， 本文档解释如何将使用jenkins 项目来对接gitlab；

### 对接步骤

1、 安装jenkins 

2、安装jenkins 插件， 需要安装git 相关的插件来进行流水线的创建和对接

![image-20240801144707135](./02_jenkins%20%E5%AF%B9%E6%8E%A5gitlab%E9%85%8D%E7%BD%AE%E6%B5%81%E6%B0%B4%E7%BA%BF.assets/image-20240801144707135.png)

![image-20240801144747097](./02_jenkins%20%E5%AF%B9%E6%8E%A5gitlab%E9%85%8D%E7%BD%AE%E6%B5%81%E6%B0%B4%E7%BA%BF.assets/image-20240801144747097.png)

3、创建gitlab 的api凭证和代码拉取凭证，先进入凭证管理界面

![image-20240801144854920](./02_jenkins%20%E5%AF%B9%E6%8E%A5gitlab%E9%85%8D%E7%BD%AE%E6%B5%81%E6%B0%B4%E7%BA%BF.assets/image-20240801144854920.png)

然后创建gitlab-api-token 凭证， 里面存放gitlab 的api-token， 和gitlab-user-password 里面存放gitlab 拉取代码的用户和密码；

![image-20240801144945384](./02_jenkins%20%E5%AF%B9%E6%8E%A5gitlab%E9%85%8D%E7%BD%AE%E6%B5%81%E6%B0%B4%E7%BA%BF.assets/image-20240801144945384.png)

gitlab api-token 可在管理员权限创建， 只需要给api 权限即可

![image-20240801145233828](./02_jenkins%20%E5%AF%B9%E6%8E%A5gitlab%E9%85%8D%E7%BD%AE%E6%B5%81%E6%B0%B4%E7%BA%BF.assets/image-20240801145233828.png)

4、新建一个resume-cmzhu-cn 的流水线，其中配置如下面图集所示

![image-20240801145807390](./02_jenkins%20%E5%AF%B9%E6%8E%A5gitlab%E9%85%8D%E7%BD%AE%E6%B5%81%E6%B0%B4%E7%BA%BF.assets/image-20240801145807390.png)

![image-20240801145905699](./02_jenkins%20%E5%AF%B9%E6%8E%A5gitlab%E9%85%8D%E7%BD%AE%E6%B5%81%E6%B0%B4%E7%BA%BF.assets/image-20240801145905699.png)

![image-20240801145941577](./02_jenkins%20%E5%AF%B9%E6%8E%A5gitlab%E9%85%8D%E7%BD%AE%E6%B5%81%E6%B0%B4%E7%BA%BF.assets/image-20240801145941577.png)

pipeline script 的代码如下：

```bash
```

