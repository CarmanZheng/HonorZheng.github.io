---
title: Bootstrap前端技术
layout: post
tags:Flask
categories: ''

---
### Bootstrap前端技术

#### 

## 1.布局容器

1. .container   用于固定宽度并支持响应式布局的容器

```html
<div class = "container">
    ...
</div>
```


2. .container-fluid类  用于100%宽度，占据全部视口

```html
<div class = "container-fluid">
    ...
</div>
```


随着屏幕或者视口的尺寸增加，系统会自动分成12列。

8/12   列数最大为12，可以通过设定col-md-x（x代表占比，x/12）参数，设定栅格占比，用来适应不同的设备（小屏幕，中屏幕，大屏幕等）

如果某一行的数字加起来大于12，多于的部分将另用于下一行

```html
<div class = 'container'>
    <div class ='row'>
          <div class = 'col-md-8' >
              8
        </div>
    </div>
</div>
```
通过xs参数控制在小屏幕时候的大小，md控制中等屏幕时候的大小

xs  -  超小屏幕（手机）；sm - 小屏幕（平板）； md - 中等屏幕（桌面显示器）； lg - 大屏幕（大桌面显示器）

```html
        <!-- 适应两种设备的布局 -->
        <div>
            <div class=" row">
                <div class="col-xs-4 col-md-4">
                    col-md-4 col-xs-6
                </div>
            </div>
        </div>
```



## 2.表格

在bootstrap里，给标签<table>添加.table 类赋予基本样式

--- 少量的内补（padding）和水平方向的分割线

```html
<!-- 渲染表格                           table 
     条纹表格（表格隔行有颜色）           table-striped
     添加边框                           table-bordered
     鼠标悬停有颜色                      table-hover
     -->
<body>
    <div class=" container">
        <table class ="table table-striped table-bordered table-hover" >
            <tr>
                <th>姓名</th>
                <th>性别</th>
                <th>年龄</th>

            </tr>
            <tr>
                <th>张三</th>
                <th>男</th>
                <th>26</th>
            </tr>
            <tr>
                <th>张三</th>
                <th>男</th>
                <th>26</th>
            </tr>
            <tr>
                <th>张三</th>
                <th>男</th>
                <th>26</th>
            </tr>
        </table>
    </div>
</body>
```

## 4.表单

单独的表单控件会被自动赋予一些全局样式。所有设置了 `.form-control` 类的 `<input>`、`<textarea>` 和 `<select>` 元素都将被默认设置宽度属性为 `width: 100%;`。 将 `label` 元素和前面提到的控件包裹在 `.form-group` 中可以获得最好的排列。

具体见 表单​ https://v3.bootcss.com/css/#forms 

## 5.组件

图标、下拉菜单、输入

```html
<span class="glyphicon glyphicon-search" aria-hidden="true"></span>
```

图标类不能和其它组件直接联合使用。它们不能在同一个元素上与其他类共同存在。应该创建一个嵌套的 `<span>` 标签，并将图标类应用到这个 `<span>` 标签上。



---

Echart数据可视化

一般来说，可以直接从 [CDN](https://www.jsdelivr.com/package/npm/echarts) 中获取构建后的 echarts，也可以从 [GitHub](https://github.com/apache/incubator-echarts/releases) 中的 `echarts/dist` 文件夹中获取构建好的 echarts，这都可以直接在浏览器端项目中使用。这些构建好的 echarts 提供了下面这几种定制：

- 完全版：`echarts/dist/echarts.js`，体积最大，包含所有的图表和组件，所包含内容参见：`echarts/echarts.all.js`。
- 常用版：`echarts/dist/echarts.common.js`，体积适中，包含常见的图表和组件，所包含内容参见：`echarts/echarts.common.js`。
- 精简版：`echarts/dist/echarts.simple.js`，体积较小，仅包含最常用的图表和组件，所包含内容参见：`echarts/echarts.simple.js`。

我们也可以自己构建 echarts，能够仅仅包括自己所需要的图表和组件。可以用这几种方式自定义构建：

- [在线自定义构建](https://echarts.apache.org/zh/builder.html)：比较方便。
- 使用 `echarts/build/build.js` 脚本自定义构建：比在线构建更灵活一点，并且支持多语言。
- 直接使用构建工具（如 [rollup](https://rollupjs.org/)、[webpack](https://webpack.js.org/)、[browserify](http://browserify.org/)）自己构建：也是一种选择。