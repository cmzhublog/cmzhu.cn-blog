## 项目设计方案

在完成了初步的示例演示后，接下来就是进入具体的预备开发阶段，一般在正式进入业务开发前，我们会针对本次需求的迭代内容进行多类的设计和评审，无设计不开发。但是问题在于，我们目前还缺很多初始化的东西没有做，因此在本章节中，我们主要针对项目目录结构、接口方案、路由注册、数据库等设计进行思考和设计开发。



### 目录结构设计

先将项目的标准目录结构创建出来， 便于后续的开发， 最终确定的目录结构如下



```bash
$ tree
├── configs  # 配置文件
├── docs  # 文档集合
├── global  # 全局变量
├── go.mod 
├── go.sum
├── internal  # 内部模块
│   ├── dao # 数据访问层 （Database Access Object）， 所有的数据库操作都会在dao层运行， mysql es 等
│   ├── middleware # HTTP 中间件
│   ├── model # 模型层， 用于存放模型对象
│   ├── routers # 路由相关逻辑处理
│   └── service  # 核心业务逻辑
├── main.go 
├── pkg  # 项目相关的模块包
├── scripts  # 项目相关的各种安装， 分析， 构建的脚本
├── storage # 项目生成的临时文件
└── third_party  # 第三方资源工具
```

本次开发中， 主要针对两大块基础业务功能， 功能点如下

- 标签管理： 文章所属的分类（标签）
- 文章管理： 整个文章内容的管理， 并且需要文章标签进行关联

开发第一步， 优先设计数据， 数据库设计如下。

![image](./02_golang%20%E9%A1%B9%E7%9B%AE%E5%BC%80%E5%8F%91%E9%A1%B9%E7%9B%AE%E8%AE%BE%E8%AE%A1%E6%96%B9%E6%A1%88.assets/db-design.jpg)

#### 数据库准备

1、 准备一个mysql 数据库， 使用5.7 即可， 并且在mysql 中执行以下sql 语句

```mysql
CREATE DATABASE
IF
	NOT EXISTS blog_service DEFAULT CHARACTER 
	SET utf8mb4 DEFAULT COLLATE utf8mb4_general_ci;
```

通过上述 SQL 语句，数据库将会创建本项目的数据库 blog_service，并设置它的默认编码为 utf8mb4。另外在每个数据表中，都包含同样的公共字段，如下：

```mysql
  `created_on` int(10) unsigned DEFAULT '0' COMMENT '创建时间',
  `created_by` varchar(100) DEFAULT '' COMMENT '创建人',
  `modified_on` int(10) unsigned DEFAULT '0' COMMENT '修改时间',
  `modified_by` varchar(100) DEFAULT '' COMMENT '修改人',
  `deleted_on` int(10) unsigned DEFAULT '0' COMMENT '删除时间',
  `is_del` tinyint(3) unsigned DEFAULT '0' COMMENT '是否删除 0 为未删除、1 为已删除',
```

大家在创建数据表时，注意将其同时包含写入就可以了。

#### 创建标签表

```mysql
CREATE TABLE `blog_tag` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(100) DEFAULT '' COMMENT '标签名称',
  
  ##======公共字段=======
  `created_on` int(10) unsigned DEFAULT '0' COMMENT '创建时间',
  `created_by` varchar(100) DEFAULT '' COMMENT '创建人',
  `modified_on` int(10) unsigned DEFAULT '0' COMMENT '修改时间',
  `modified_by` varchar(100) DEFAULT '' COMMENT '修改人',
  `deleted_on` int(10) unsigned DEFAULT '0' COMMENT '删除时间',
  `is_del` tinyint(3) unsigned DEFAULT '0' COMMENT '是否删除 0 为未删除、1 为已删除',
  # 此处请写入公共字段
  `state` tinyint(3) unsigned DEFAULT '1' COMMENT '状态 0 为禁用、1 为启用',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='标签管理';
```

创建标签表，表字段主要为标签的名称、状态以及公共字段。

### 创建文章表

```shell
CREATE TABLE `blog_article` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `title` varchar(100) DEFAULT '' COMMENT '文章标题',
  `desc` varchar(255) DEFAULT '' COMMENT '文章简述',
  `cover_image_url` varchar(255) DEFAULT '' COMMENT '封面图片地址',
  `content` longtext COMMENT '文章内容',
  ##======公共字段=======
  `created_on` int(10) unsigned DEFAULT '0' COMMENT '创建时间',
  `created_by` varchar(100) DEFAULT '' COMMENT '创建人',
  `modified_on` int(10) unsigned DEFAULT '0' COMMENT '修改时间',
  `modified_by` varchar(100) DEFAULT '' COMMENT '修改人',
  `deleted_on` int(10) unsigned DEFAULT '0' COMMENT '删除时间',
  `is_del` tinyint(3) unsigned DEFAULT '0' COMMENT '是否删除 0 为未删除、1 为已删除',
  # 此处请写入公共字段
  `state` tinyint(3) unsigned DEFAULT '1' COMMENT '状态 0 为禁用、1 为启用',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='文章管理';
```

创建文章表，表字段主要为文章的标题、封面图、内容概述以及公共字段。

#### 创建文章标签关联表

```shell
CREATE TABLE `blog_article_tag` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `article_id` int(11) NOT NULL COMMENT '文章 ID',
  `tag_id` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '标签 ID',
  ##======公共字段=======
  `created_on` int(10) unsigned DEFAULT '0' COMMENT '创建时间',
  `created_by` varchar(100) DEFAULT '' COMMENT '创建人',
  `modified_on` int(10) unsigned DEFAULT '0' COMMENT '修改时间',
  `modified_by` varchar(100) DEFAULT '' COMMENT '修改人',
  `deleted_on` int(10) unsigned DEFAULT '0' COMMENT '删除时间',
  `is_del` tinyint(3) unsigned DEFAULT '0' COMMENT '是否删除 0 为未删除、1 为已删除',
  # 此处请写入公共字段
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='文章标签关联';
```

mysql 创建完成之后查询表结构如下

```mysql
mysql> show tables;
+------------------------+
| Tables_in_blog_service |
+------------------------+
| blog_article           |
| blog_article_tag       |
| blog_tag               |
+------------------------+
3 rows in set (0.00 sec)
```

### Golang 中创建model 

数据库表的创建完成之后， 我们需要在目录 `internal/model` 中创建对应model 对象， 以便于后续程序的使用



创建公共model

在internel/model 目录下创建model.go 文件

```go 
type Model struct {
	ID	uint32 `gorm:"primary_key" json:"id"`
	CreatedBy string `json:"created_by"`
	ModifiedBy string `json:"modified_by"`
	CreatedOn uint32 `json:"created_on"`
	ModifiedOn uint32 `json:"modified_on"`
	DeleteOn uint32 `json:"deleted_on"`
	IsDel uint8 `json:"is_del"`
}
```

创建标签的model tag.go文件

```go
type Tag struct {
	*Model
	Name string `json:"name"`
	State uint8 `json:"state"`
}

func (t Tag) TableName() string {
	return "blog_tag"
}
```

创建文章model

在 `internal/model` 目录下创建 article.go 文件，写入如下代码

```go
type Article struct {
	*Model
	Title string `json:"title"`
	Desc  string `json:"desc"`
	Content string `json:"content"`
	CoverImageUrl string `json:"cover_image_url"`
	State uint8 `json:"state"`
}

func (a Article) TableName() string {
	return "blog_article"
}
```

创建文件标签model

在 `internal/model` 目录下创建 article_tag.go 文件，写入如下代码：

```go
type ArticleTag struct {
	*Model
	TagID uint32 `json:"tag_id"`
	ArticleID uint32 `json:"article_id"`

}

func (a ArticleTag) TableName() {
	return "blog_article_tag"
}
```

### 路由

在完成数据库设计后， 我们需要对业务模块管理接口进行设计， 这一块最核心的就是增删改查的RESTful API 的设计和编写， 在 RESTful API 中HTTP 方法动作如下

- GET：读取/检索动作。
- POST：新增/新建动作。
- PUT：更新动作，用于更新一个完整的资源，要求为幂等。
- PATCH：更新动作，用于更新某一个资源的一个组成部分，也就是只需要更新该资源的某一项，就应该使用 PATCH 而不是 PUT，可以不幂等。
- DELETE：删除动作。

标签管理

| 功能         | HTTP 方法 | 路径      |
| ------------ | --------- | --------- |
| 新增标签     | POST      | /tags     |
| 删除指定标签 | DELETE    | /tags/:id |
| 更新指定标签 | PUT       | /tags/:id |
| 获取标签列表 | GET       | /tags     |

 文章管理

| 功能         | HTTP 方法 | 路径          |
| ------------ | --------- | ------------- |
| 新增文章     | POST      | /articles     |
| 删除指定文章 | DELETE    | /articles/:id |
| 更新指定文章 | PUT       | /articles/:id |
| 获取指定文章 | GET       | /articles/:id |
| 获取文章列表 | GET       | /articles     |

#### 路由管理

在确定了业务接口设计后，需要对业务接口进行一个基础编码，确定其方法原型，把当前工作区切换到项目目录的 `internal/routers` 下，并新建 router.go 文件，写入代码：

```go
package main

import (
	"github.com/gin-gonic/gin"
)

func NewRouter() *gin.Engine{
	r := gin.New()
	r.Use(gin.Logger())
	r.Use(gin.Recovery())
	
	apiv1 := r.Group("/api/v1"){
		apiv1.POST("/tags")
		apiv1.DELETE("/tags/:id")
		apiv1.PUT("/tags/:id")
		apiv1.PATCH("/tags/:id/state")
		apiv1.get("/tags")

		apiv1.POST("/articles")
		apiv1.DELETE("/articles/:id")
		apiv1.PUT("/articles/:id")
		apiv1.PATCH("/articles/:id/state")
		apiv1.GET("/articles/:id")
		apiv1.GET("/articles")
	}
	return r
}	
```

#### 处理程序

接下来编写对应路由的处理方法，我们在项目目录下新建 `internal/routers/api/v1` 文件夹，并新建 tag.go（标签）和 article.go（文章）文件，写入代码下述代码。

```go
package 

import (
	"github.com/gin-gonic/gin"
)

type Tag struct {}

func NewTag() Tag {
	return Tag{}
}

func (t Tag) Get(c *gin.Context){}
func (t Tag) List(c *gin.Context){}
func (t Tag) Create(c *gin.Context){}
func (t Tag) Update(c *gin.Context){}
func (t Tag) Delete(c *gin.Context){}

```

article.go 文件

```go
import (
	"github.com/gin-gonic/gin"
)

type Article struct {}

func NewArticle() Article {
	return Article{}
}

func (a Article) Get(c *gin.Context) {}
func (a Article) List(c *gin.Context) {}
func (a Article) Create(c *gin.Context) {}
func (a Article) Update(c *gin.Context) {}
func (a Article) Delete(c *gin.Context) {}
```

