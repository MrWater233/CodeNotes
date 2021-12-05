# 安装

```go
go get -u github.com/go-playground/validator/v10
```

# 校验标签

| 标签     | 含义                      |
| -------- | ------------------------- |
| required | 必填                      |
| gt       | 大于                      |
| gte      | 大于等于                  |
| lt       | 小于                      |
| lte      | 小于等于                  |
| min      | 最小值                    |
| max      | 最大值                    |
| oneof    | 参数集内的其中之一        |
| len      | 长度要求与 len 给定的一致 |

例子：

```go
type CountTagRequest struct {
	Name  string `form:"name" binding:"max=100"`
	State uint8 `form:"state,default=1" binding:"oneof=0 1"`
}
```

结构体中使用了`from`和`binding`两个标签，它们分别代表着表单的**映射字段名**和**入参校验的规则内容**，作用是**参数绑定**和**参数校验**



开源地址：https://github.com/go-playground/validator