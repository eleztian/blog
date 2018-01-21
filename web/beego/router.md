---
title: Beego Router
short: ""
date: 19.Jan.2018
tags:
    - c
    - server
    - web
    - beego
    - golang
    - notes
---
# beego-router

beego 默认采用RESTful路由，即什么样的请求执行什么样的方法，post请求执行post方法。

## 固定路由

```go
beego.Router("/", &controllers.MainController{})
// GET 请求 / 默认执行MainController.Get()方法
// POST 请求 / 默认执行MainController.Post()方法
```

## 正则路由

```go
//带有前缀的自定义正则 //匹配 :id 为正则类型。匹配 cms_123.html 这样的 url :id = 123。
beego.Router(“/cms_:id([0-9]+).html”, &controllers.CmsController{})

// 在对应的方法里采用如下获取请求参数。
this.Ctx.Input.Param(":id")
```

## 自定义方法

使用了自定义方法后，RESTful方式的路由失效，即不会去默认执行对应的control.Get()等方法。
eg:

```go
// 接受任何方式请求 /api/list
beego.Router("/api/list",&RestController{},"*:ListFood")
// 只有POST方式请求/api/create 才会响应RestController.CreateFood()
beego.Router("/api/create",&RestController{},"post:CreateFood")
```

## 自动匹配


```go
func init() {
  beego.AutoRouter(&controllers.ObjectController{})
}

func (c *ObjectController)func1() {
//...
}
func (c *ObjectController)func2() {
//...
}
```
这时，可以通过访问 http:/.../object/func1 执行func1方法，访问 http:/.../object/func2 执行func2 方法

如果将 ObjectController 改为 object 同样可行。

## 注解路由

//TODO: 还没有搞懂

```go
// @router /add [get]
func (c *MainController) AddPeople() {}
```