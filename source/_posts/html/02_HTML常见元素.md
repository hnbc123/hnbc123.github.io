---
title: HTML常见元素
date: 2023-04-08 17:05:00
categories:
- web前端
- html
---

# 常见文本元素

## h元素

h元素用于将重要的文字作为标题展示，h1-h6呈现了留个不同级别的标题，h1级别最高。

**h元素通常和SEO优化有关**

```html
<h1>h1标题标签</h1>
<h2>h2标题标签</h2>
<h3>h3标题标签</h3>
<h4>h4标题标签</h4>
<h5>h5标题标签</h5>
<h6>h6标题标签</h6>
```



## p元素

p元素用于表示文本的一个段落，多个段落之间会有一定的间距。

```html
<p>段落内容</p>
```



## img元素

img元素将一份图像嵌入文档，常见属性：

* src：必须，表示图片的文件路径
* alt：当图片加载不成功时显示，也用于屏幕阅读器描述图片

```html
<!-- 绝对路径 -->
<img src="D:/image.jpg" alt="">
<!-- 相对路径 -->
<!-- 
	.表示当前文件夹，可省略
	..表示上级文件夹
-->
<img src="./image.jpg" alt="">
```



## a元素

### 超链接

a元素定义超链接，用于跳转到另外一个链接。

常见属性：

* href：Hypertext Reference，指定要打开的URL地址
* target：指定在何处显示链接的资源
  * _self：默认值，在当前窗口打开
  * _blank：在新窗口打开
  * _parent：见iframe元素
  * _top：见iframe元素

### 锚点链接

a元素可以定义锚点链接，跳转到网页中的具体位置，步骤如下：

* 在要跳转到的元素上定义id属性
* 定义a元素，并且href指向对应的id

```html
<a href="#one">跳转段落1</a>
<a href="#two">跳转段落2</a>
<a href="#three">跳转段落3</a>

<p id="one">段落1</p>
<p id="two">段落2</p>
<p id="three">段落3</p>
```

### 图片链接

img元素和a元素一起使用，可以实现点击图片跳转。

```html
<a href="http://www.study.com">
	<img src="./images.jpg" alt="">
</a>
```

### 其他URL地址

a元素还可以用于下载资料或发送邮件等

```html
<a href="https://study/note.zip">资源下载</a>
<a href="mailto:12345@qq.com">发送邮件</a>
```



## iframe元素

iframe元素可以在一个HTML文档中嵌入其他HTML文档：

frameBorder属性：是否显示边框，1显示；2不显示

```html
<iframe src="./other/iframe元素所在的页面.html" frameborder="2"></iframe>
```



a元素target的其他值：

* _parent：在父窗口打开URL
* _top：在顶层窗口打开URL



## div元素和span元素

css出现后，可用div、span来编写HTML所有结构，样式都交由css处理。

div和span都是容器，用来包裹内容

* div元素(division)：多个div元素包裹的内容会在不同行显示。一般作为其他元素的父容器，用于把网页分割为多个独立的部分
* spanyuans：多个span包裹的内容会在同一行显示。一般用于区分特殊文本与普通文本，比如显示关键字





# 不常见文本元素

* strong元素：内容加粗、强调
* i元素：内容倾斜
* code元素：用于显示代码，偶尔会用来显示等宽字体
* br元素：换行元素，已不使用



更多元素可查看 https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element

