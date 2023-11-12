---
title: fastapi_amis_admin 的主页
description: 
published: true
date: 2022-11-24T06:15:33.687Z
tags: fastapi_amis_admin
editor: markdown
dateCreated: 2022-11-01T02:52:42.691Z
---



# fastapi_amis_admin

## 简介

本文主要写fastapi_amis_admin 实现configmap 的配置项管理，结合reloader 实现对deployment 环境变量的修改和重启。
官方文档链接:

```bash
http://docs.gh.amis.work/
```
## 实现方式

1、fastapi_amis_admin 初级使用 

[Fastapi_amis_admin]: https://github.com/amisadmin/fastapi_amis_admin

### 特点

- **高性能**：基于[FastAPI](https://fastapi.tiangolo.com/)。享受所有好处。
- **高效率**：完善的代码类型提示。更高的代码可重用性。
- **支持异步和同步混合写入**：`ORM` 基于`SQLModel`& `Sqlalchemy`。自由定制数据库类型。支持同步和异步模式。可扩展性强。
- **前端分离**：前端由 渲染`Amis`，后端界面由 自动生成`fastapi-amis-admin`。接口是可重用的。
- **扩展性强**：后台页面支持`Amis`页面和普通`html`页面。轻松自由地自定义界面。
- **自动api文档**：自动生成接口文档`FastAPI`。轻松调试和共享接口。

### 依赖项

- [FastAPI](https://fastapi.tiangolo.com/)：完成 Web 部件。
- [SQLModel](https://sqlmodel.tiangolo.com/)：完成`ORM`模型映射。[将SQLAlchemy](https://www.sqlalchemy.org/)与[Pydantic](https://pydantic-docs.helpmanual.io/)完美结合 ，并拥有它们的所有功能。
- [Amis](https://baidu.gitee.io/amis)：完成管理页面演示。

### 安装

```
pip install fastapi_amis_admin
```

### 基础实例用法

```python
from fastapi import FastAPI
from fastapi_amis_admin.admin.settings import Settings
from fastapi_amis_admin.admin.site import AdminSite

# create FastAPI application
app = FastAPI()

# create AdminSite instance
site = AdminSite(settings=Settings(database_url_async='sqlite+aiosqlite:///amisadmin.db'))

# mount AdminSite instance
site.mount_app(app)

if __name__ == '__main__':
    import uvicorn

    uvicorn.run(app, debug=True)
```



FormAdmin

```python
from typing import Any
from fastapi import FastAPI
from pydantic import BaseModel
from starlette.requests import Request
from fastapi_amis_admin.amis.components import Form
from fastapi_amis_admin.admin import admin
from fastapi_amis_admin.admin.settings import Settings
from fastapi_amis_admin.admin.site import AdminSite
from fastapi_amis_admin.crud.schema import BaseApiOut
from fastapi_amis_admin.models.fields import Field

# create FastAPI application
app = FastAPI()

# create AdminSite instance
site = AdminSite(settings=Settings(database_url_async='sqlite+aiosqlite:///amisadmin.db'))


# register FormAdmin
@site.register_admin
class UserLoginFormAdmin(admin.FormAdmin):
    page_schema = 'UserLoginForm'
    # set form information, optional
    form = Form(title='This is a test login form', submitText='login')

    # create form schema
    class schema(BaseModel):
        username: str = Field(..., title='username', min_length=3, max_length=30)
        password: str = Field(..., title='password')

    # handle form submission data
    async def handle(self, request: Request, data: BaseModel, **kwargs) -> BaseApiOut[Any]:
        if data.username == 'amisadmin' and data.password == 'amisadmin':
            return BaseApiOut(msg='Login successfully!', data={'token': 'xxxxxx'})
        return BaseApiOut(status=-1, msg='Incorrect username or password!')


# mount AdminSite instance
site.mount_app(app)

if __name__ == '__main__':
    import uvicorn

    uvicorn.run(app, debug=True)

```

### 相关项目

- [`Amis-Admin-Theme-Editor`](https://github.com/swelcker/amis-admin-theme-editor):fastapi-amis-admin 的主题编辑器。允许添加自定义 css 样式并动态应用主题 --vars 更改。
- [`FastAPI-User-Auth`](https://github.com/amisadmin/fastapi_user_auth)：一个简单而强大的`FastAPI`用户`RBAC` 认证和授权库。
- [`FastAPI-Scheduler`](https://github.com/amisadmin/fastapi_scheduler): 一个简单的计划任务管理`FastAPI`扩展，基于`APScheduler`.
- [`FastAPI-Amis-Admin-Demo`](https://github.com/amisadmin/fastapi_amis_admin_demo): 一个示例`FastAPI-Amis-Admin`应用程序。
- [`FastAPI-User-Auth-Demo`](https://github.com/amisadmin/fastapi_user_auth_demo): 一个示例`FastAPI-User-Auth`应用程序。













