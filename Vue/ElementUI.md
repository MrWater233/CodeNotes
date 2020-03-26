# 一.主要标签

## 管理系统模板

- `el-container`：构建整个页面框架
- `el-aside`：构建页面左侧菜单
- `el-menu`：左侧菜单内容
  - `:default-openeds`：默认展开的菜单，通过菜单的index值来关联
  - `:default-active`：默认选中的菜单，通过菜单的index值来关联
- `el-submenu`：可展开的菜单
  - `index`：菜单下表，文本类型，不能是数值类型
- `template`：对应`el-submenu`菜单名
- `i`：写在`template`内，可以设置菜单图标
  - `clsss`：通过该属性设置
    - `el-icon-messgae`
    - `el-icon-menu`
    - `el-icon-setting`
- `el-menu-item`：不可再展开子节点
  - `index`：菜单下标

