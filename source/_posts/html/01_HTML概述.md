---
title: HTML基本概念
date: 2023-04-08 08:00:00
categories: 
- web前端
- html
---

# 网页和网站

打开浏览器看到的一个页面即为网页（web page），网页可以包括文字、链接、图片、音乐、视频等。网站是由多个网页组成的。

网页的显示过程：

1. 开发项目并打包部署到服务器
2. 用户在浏览器输入网站
3. 浏览器找到对应的服务器地址，请求静态资源（可以存放在世界任何地方）
4. 服务器返回静态资源给浏览器
5. 浏览器对静态资源进行解析和展示





# 认识HTML

HTML(HyperText Markup Language)：超文本标记语言，是一种用于创建网页的标准标记语言。

标记语言由无数个标记（标签、tag）组成，是对某些内容进行特殊标记，以供其他解释器识别处理的语言。

超文本表示不仅可以插入普通的文本，还可以插入图片、音频、视频、超链接等内容。

HTML文件扩展名为.htm/.html，因为Win95/Win98系统的文件扩展名不能超过3字符，所以使用.htm，现在统一使用.html。





# HTML 元素

HTML元素可在 https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element 中查找。

![](元素结构.png)



元素的属性：

* 属性包含元素的额外信息，这些信息不会出现在实际的内容中
* 一个属性必须包含：空格、属性名称、属性值
* 有些属性是每个元素都可以设置的，称为全局属性，比如class、id、title属性
* 有些属性是元素特有的，比如meta元素的charset属性，img元素的alt属性



常见全局属性：

* id：定义唯一标识符
* class：一个以空格分隔的元素的类名列表
* style：给元素添加内联样式
* title：包含表示与其所属元素相关信息的文本，这些信息通常可以作为提示呈现给用户



有些元素只有一个标签，称为单标签元素，例如br、img、hr、meta、input

```html
<input type="text">
```



**注意：HTML元素不区分大小写，但是推荐小写**





# HTML结构分析

## 完整的HTML结构

一个完整的HTML结构包括：

* 文档声明
* html元素
  * head元素
  * body元素

```html
<!DOCTYPE html>
<html>
    <head>
        <title>HTML文档结构</title>
    </head>
    <body>
        <h1>我是标题</h1>
    </body>
</html>
```



## 文档声明

HTML最上方的一段文本称为文档类型声明，用于声明文档类型。

```html
<!DOCTYPE html>
```

以上语句告诉浏览器当前页面是HTML5页面，让浏览器用HTML5的标准去解析识别内容。

必须放在HTML文档的最前面，不能省略，否则可能出现兼容性问题。



## html元素

html元素表示一个HTML文档的根，也被称为根元素，所有其他元素必须是此元素的后代。

W3C标准建议为html元素增加lang属性，作用是：

* 帮助语音合成工具确定要使用的发音
* 帮助翻译工具确定要使用的翻译规则



常用规则：

* lang="en"：英文
* lang="zh-CN"：中文



## body元素

body元素里面的内容是你在浏览器窗口中看到的东西，也就是网页的具体内容和结构。