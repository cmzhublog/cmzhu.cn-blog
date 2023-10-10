---
title: PageAdmin
description: 
published: true
date: 2022-11-24T07:29:44.368Z
tags: fastapi_amis_admin
editor: markdown
dateCreated: 2022-11-02T01:51:25.251Z
---

# PageAdmin

## 代码示例

```python
@site.register_admin
class UserBoxManager(admin.PageAdmin):
    group_schema = PageSchema(label='ConfigMap', icon='fa fa-wordpress')
    page_schema = PageSchema(label='配置管理', icon='fa fa-folder')
    columes = {"user":"用户名", "gpu_limit_by_user":"GPU 数量","cpu_limit_by_user":"CPU","mem_limit_by_user":"内存","node_selector_by_user":"节点限制" }

    async def get_table_columes(self) -> List[TableColumn]:
        if self.columes is None:
            return None
        result = []
        for name, label in self.columes.items():
            tablecolume = TableColumn()
            tablecolume.name = name
            tablecolume.label = label
            result.append(tablecolume)

        return result
    

    async def get_colume_dialog(self) ->Dialog:
        result_form = Form(title="用户信息")
        # result_form.title = "用户信息"
        # result_dialog = Dialog()
        body = []
        # 创建 Form Item
        for name, label in self.columes.items():

            formitem =  InputText()
            formitem.name, formitem.label = name, label
            formitem.placeholder = f"请输入{label}"
            
            if name == "user":
                formitem.required = True
            elif  name == "node_selector_by_user":
                formitem.required = True
                formitem.placeholder = f"请输入{label}, 格式必须为"+'{"key":"value"}'

            body.append(formitem)
        result_form.body = body
        result_form.api = f"{sitesettings.root_path}/usersettingsschema/update"
        data = {
            "&":"$$",
            "gpu_limit_by_user":"${IF(${gpu_limit_by_user} === '-' , 0, ${gpu_limit_by_user} )}",
            "cpu_limit_by_user":"${IF(${cpu_limit_by_user} === '-' , 0, ${cpu_limit_by_user} )}",
            "mem_limit_by_user": "${IF(${mem_limit_by_user} === '-' , '0Gi', ${mem_limit_by_user} )}",
            }

        return Dialog(title="信息修改", body=[result_form], data=data)


    async def get_page(self, request: Request) -> Page:
        page = await super().get_page(request)
        
        button = Button(type="button", label="新增配置", actionType="dialog", level="info", dialog=await self.get_colume_dialog())
        page.initApi = sitesettings.root_path+"/usersettingsschema/list?search=$search"
        table = Table(title="用户配置表", source="$rows")
        table.columns = await self.get_table_columes()

        action_edit = ActionType.Dialog(label='编辑', dialog=await self.get_colume_dialog())
        action_delete = ActionType.Ajax(
            label='删除', 
            confirmText="是否删除${user}用户吗?\n 注意：删除之后将为用户恢复默认配置。",
            api={
                "method": "delete",
                "url" :sitesettings.root_path+"/usersettingsschema/delete/$user",
                }
            )
        
        table.itemActions = [action_edit, action_delete]
        page.body = [SearchBox(name="search"),button,table]

        return page


```

## 代码介绍

### 继承PageAdmin

group_schema: PageSchema       指定schema 显示的内容
page_schema: PageSchema.       指定schema 显示的内容 

### 页面修改

async def get_page(self, request: Request) -> Page: 可继承原有的页面配置，如果修改相应的配置直接赋值即可。

```python
async def get_page(self, request: Request) -> Page:
  page = await super().get_page(request)
  ...
  return page
```

这里非常适合fastapiamis 的配置,功能是在page 上显示出对应的组件;这些配置都是在fastapi_amis_admin 的源码中可以找到(fastapi_amis_admin/amis/components.py)
 
 用法例如下方代码.
```
        button = Button(type="button", label="新增配置", actionType="dialog", level="info", dialog=await self.get_colume_dialog())
        page.initApi = sitesettings.root_path+"/usersettingsschema/list?search=$search"
        table = Table(title="用户配置表", source="$rows")
        table.columns = await self.get_table_columes()
        action_edit = ActionType.Dialog(label='编辑', dialog=await self.get_colume_dialog())
        action_delete = ActionType.Ajax(
            label='删除', 
            confirmText="是否删除${user}用户吗?\n 注意：删除之后将为用户恢复默认配置。",
            api={
                "method": "delete",
                "url" :sitesettings.root_path+"/usersettingsschema/delete/$user",
                }
            )
        
        table.itemActions = [action_edit, action_delete]
        page.body = [SearchBox(name="search"),button,table]
```

例: button 组件, 在源码(fastapi_amis_admin/amis/components.py) 定义了其参数,初始化时可以参照使用即可; 

另外 通过追述button 的继承关系,最终继承于pydantic库的BaseModel;可以读出最终是将配置转换成json 格式

```
from pydantic import BaseModel

class Button(FormItem):
    """按钮"""
    className: str = None  # 指定添加 button 类名
    href: str = None  # 点击跳转的地址，指定此属性 button 的行为和 a 链接一致
    size: str = None  # 设置按钮大小 'xs'|'sm'|'md'|'lg'
    actionType: str = None  # 设置按钮类型 'button'|'reset'|'submit'| 'clear'| 'url'
    level: LevelEnum = None
    # 设置按钮样式 'link'|'primary'|'enhance'|'secondary'|'info'|'success'|'warning'|'danger'|'light'| 'dark'|'default'
    tooltip: Union[str, dict] = None  # 气泡提示内容 TooltipObject
    tooltipPlacement: str = None  # 气泡框位置器 'top'|'right'|'bottom'|'left'
    tooltipTrigger: str = None  # 触发 tootip 'hover'|'focus'
    disabled: bool = None  # 按钮失效状态
    block: bool = None  # 将按钮宽度调整为其父宽度的选项
    loading: bool = None  # 显示按钮 loading 效果
    loadingOn: str = None  # 显示按钮 loading 表达式


class FormItem(AmisNode):
    """表单项通用"""
    type: str = 'input-text'  # 指定表单项类型
    className: str = None  # 表单最外层类名
    inputClassName: str = None  # 表单控制器类名
    labelClassName: str = None  # label 的类名
    name: str = None  # 字段名，指定该表单项提交时的 key
    label: Template = None  # 表单项标签  模板或false
    value: Union[int, str] = None  # 字段的值
    labelRemark: "Remark" = None  # 表单项标签描述
    description: Template = None  # 表单项描述
    placeholder: str = None  # 表单项描述
    inline: bool = None  # 是否为 内联 模式
    submitOnChange: bool = None  # 是否该表单项值发生变化时就提交当前表单。
    disabled: bool = None  # 当前表单项是否是禁用状态
    disabledOn: Expression = None  # 当前表单项是否禁用的条件
    visible: Expression = None  # 当前表单项是否禁用的条件
    visibleOn: Expression = None  # 当前表单项是否禁用的条件
    required: bool = None  # 是否为必填。
    requiredOn: Expression = None  # 过表达式来配置当前表单项是否为必填。
    validations: Union[Validation, Expression] = None  # 表单项值格式验证，支持设置多个，多个规则用英文逗号隔开。
    validateApi: Expression = None  # 表单校验接口
    copyable: Union[bool, dict] = None  # 是否可复制  boolean 或 {icon: string, content:string}


class AmisNode(BaseAmisModel):
    """组件配置"""
    type: str = None  # 组件类型
    visible: bool = None  # 显示
    hidden: bool = None  # 隐藏
    visibleOn: Expression = None  # 条件显示
    hiddenOn: Expression = None  # 条件显示
    id: str = None
    name: str = None
    onEvent: dict = None


class BaseAmisModel(BaseModel):
    class Config:
        extra = Extra.allow
        json_loads = json.loads
        json_dumps = json.dumps

    def amis_json(self):
        return self.json(exclude_none = True, by_alias = True)

    def amis_dict(self):
        return self.dict(exclude_none = True, by_alias = True)

    def update_from_dict(self, kwargs: Dict[str, Any]):
        for k, v in kwargs.items():
            setattr(self, k, v)
        return self

    def update_from_kwargs(self, **kwargs):
        return self.update_from_dict(kwargs)
```





赋值对象可参照fastapi_amis_admin源码中提供的schema,如下；

```python
class Page(AmisNode):
    """页面"""
    __default_template_path__: str = f'{BASE_DIR}/templates/page.html'

    type: str = "page"  # 指定为 Page 组件
    title: SchemaNode = None  # 页面标题
    subTitle: SchemaNode = None  # 页面副标题
    remark: "Remark" = None  # 标题附近会出现一个提示图标，鼠标放上去会提示该内容。
    aside: SchemaNode = None  # 往页面的边栏区域加内容
    asideResizor: bool = None  # 页面的边栏区域宽度是否可调整
    asideMinWidth: int = None  # 页面边栏区域的最小宽度
    asideMaxWidth: int = None  # 页面边栏区域的最大宽度
    toolbar: SchemaNode = None  # 往页面的右上角加内容，需要注意的是，当有 title 时，该区域在右上角，没有时该区域在顶部
    body: SchemaNode = None  # 往页面的内容区域加内容
    className: str = None  # 外层 dom 类名
    cssVars: dict = None  # 自定义 CSS 变量，请参考样式
    toolbarClassName: str = None  # "v-middle wrapper text-right bg-light b-b"  # Toolbar dom 类名
    bodyClassName: str = None  # "wrapper"  # Body dom 类名
    asideClassName: str = None  # "w page-aside-region bg-auto"  # Aside dom 类名
    headerClassName: str = None  # "bg-light b-b wrapper"  # Header 区域 dom 类名
    initApi: API = None  # Page 用来获取初始数据的 api。返回的数据可以整个 page 级别使用。
    initFetch: bool = None  # True  # 是否起始拉取 initApi
    initFetchOn: Expression = None  # 是否起始拉取 initApi, 通过表达式配置
    interval: int = None  # 刷新时间(最小 1000)
    silentPolling: bool = None  # False  # 配置刷新时是否显示加载动画
    stopAutoRefreshWhen: Expression = None  # 通过表达式来配置停止刷新的条件
    regions: List[str] = None

    def amis_html(
            self,
            template_path: str = '',
            locale: str = 'zh_CN',
            cdn: str = 'https://unpkg.com',
            pkg: str = 'amis@1.10.1',
            site_title: str = 'Amis',
            site_icon: str = '',
    ):
        """渲染html模板"""
        template_path = template_path or self.__default_template_path__
        return amis_templates(template_path).safe_substitute(
            {
                "AmisSchemaJson": self.amis_json(),
                "locale": locale,
                "cdn": cdn,
                "pkg": pkg,
                "site_title": site_title,
                "site_icon": site_icon,
            }
        )

```

在使用时可与amis 官网的,理论上;如果需要使用的amis 组件在fastapi_amis_admin 中找不到,可以通过新建继承于AmisNode 类方式进行自定义;

[page组件]: https://aisuda.bce.baidu.com/amis/zh-CN/components/page


<img src="/image/image-page-admin.png" alt="image-page-admin" style="zoom:50%;" />



链接汇总
  AMIS: https://aisuda.bce.baidu.com/amis/zh-CN/docs/index
