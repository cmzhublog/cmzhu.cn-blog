## gitlab 添加子项目

参考文档： [https://git-scm.com/book/en/v2/Git-Tools-Submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules)



### 操作步骤

1、 git 克隆项目

2、 项目根目录下创建一个 `.gitmodules` 文件

3、 使用`git submodule` 添加子项目

```BASH
$ git submodule add https://github.com/chaconinc/DbConnector 
```

4、操作完成之后 ，会在项目根目录下看到一个`DbConnector` 的文件夹