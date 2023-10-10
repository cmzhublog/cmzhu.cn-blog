---
title: 静态页面处理
description: 
published: true
date: 2022-11-02T03:29:22.784Z
tags: fastapi_amis_admin
editor: markdown
dateCreated: 2022-11-02T03:29:22.784Z
---

# CDN 静态文件处理



## 研究目的

为了 该项目能够离线运行

## 示例

![image-page-admin.png](/image/image-page-admin.png)

## 源码分析

### 模板位置

```shell
./fastapi_amis_admin/amis/templates 文件夹中，后续如果需要调整模板，可调整html 文件
```

从模板文件中，不难发现，所有的静态css 文件都从线上文件下载下来，一但出现离线环境，页面就会使用不了。

```html
<link href="${site_icon}" rel="shortcut icon" type="image/x-icon"/>
    <link href="${cdn}/${pkg}/sdk/sdk.css" rel="stylesheet" title="default"/>
    <link href="${cdn}/${pkg}/sdk/helper.css" rel="stylesheet"/>
    <link href="${cdn}/${pkg}/sdk/iconfont.css" rel="stylesheet"/>
    <script src="${cdn}/${pkg}/sdk/sdk.js"></script>
    <script src="${cdn}/vue@2.6.14/dist/vue.js"></script>
    <script src="${cdn}/history@4.10.1/umd/history.js"></script>
```

又通过分析页面发现，cdn地址，通过变量传输，在fastapi_amis_admin 肯定有地赋值。最终发现，settings 文件中有对外提供amis_cdn 和amis_pkg 变量，可修改其配置完成（通过环境变量注入，通过初始化settings 时注入，这里介绍初始化注入）。

```python
class Settings(BaseSettings):
    """项目配置"""
    host: str = '127.0.0.1'
    port: int = 8000
    debug: bool = False
    version: str = '0.0.0'
    site_title: str = 'FastAPI Amis Admin'
    site_icon: str = 'https://baidu.gitee.io/amis/static/favicon_b3b0647.png'
    site_url: str = ''
    root_path: str = '/admin'
    database_url_async: str = Field('', env='DATABASE_URL_ASYNC')
    database_url: str = Field('', env='DATABASE_URL')
    language: str = ''  # 'zh_CN','en_US'
    amis_cdn: str = 'https://unpkg.com'
    amis_pkg: str = 'amis@1.10.2'

    @validator('amis_cdn', 'root_path', 'site_url', pre=True)
    def valid_url(url: str):
        return url[:-1] if url.endswith('/') else url

    @root_validator(pre=True)
    def valid_database_url(cls, values):
        if not values.get('database_url') and not values.get(
                'database_url_async'):
            values.setdefault(
                'database_url',
                'sqlite+aiosqlite:///amisadmin.db?check_same_thread=False')
        return values
```



```python
sitesettings = Settings(database_url_async='sqlite+aiosqlite:///amisadmin.db', language='zh_CN', site_title="BIGQUANT", amis_cdn="./static")
```



此时地址是启动在当前路由下查询，所以需要通过fastapi提供静态路由，来解决本地静态文件访问问题。

```python
app.mount(f"{sitesettings.root_path}/static", StaticFiles(directory="userbox-config-manager/static"), name="static")
```

