---
title: CSS常见属性
date: 2023-04-15 10:44:00
categories: 
- web前端
- css
---



# CSS概述

CSS表示层叠样式表（Cascading Style Sheet），又称串样式列表、级联样式表、串接样式表、阶层式样式表，是为网页添加样式的代码。

声明一个单独的CSS规则，属性名: 属性值。

* 属性名：要添加的CSS规则名称
* 属性值：要添加的CSS规则的值



# CSS引用方式

## 内联样式

内联样式(inline style)，也成行内样式，存在于HTML元素的style属性中。

```html
<div style="color: red; font-size: 20px;">div元素</div>
```

CSS样式之间用分号隔开，建议每条CSS样式后面都加分号。

在原生HTML编写过程中不推荐使用，在Vue的template某些动态样式中会使用。



## 内部样式表

内部样式表(internal style sheet)，将CSS放在HTML文件**head**元素里的**style**元素中。

```html
<head>
    <meta charset="UTF-8">
    <title>Document</title>
    <style>
        .title {
            margin: 20px 0;
            font-size: 24px;
            font-weight: 700;
        }
    </style>
</head>
```





## 外部样式表

外部样式表(external style sheet)是将CSS编写在一个独立的文件中，通过**link**元素引入。

```html
<head>
    <meta charset="UTF-8">
    <title>Document</title>
    <link rel="stylesheet" href="./css/styles.css">
</head>
```





## @import

可以在style元素或CSS文件中使用@import导入其他的CSS文件

```css
@import url(./other.css);

.title {
    color: red;
    font-size: 30px;
    background-color: skyblue;
}
```





# 官方文档

* CSS官方文档地址：https://www.w3.org/TR/?tag=css
* CSS推荐文档地址：https://developer.mozilla.org/zh-CN/docs/Web/CSS/Reference
* 查询某版本下CSS是否可用：https://caniuse.com/





# 常用属性

## 基础属性

* color：前景色（文字颜色）
* background-color：背景色
* width：宽度
* height：高度



## 文本属性

* text-decoration：设置文字的装饰线
  * none：无装饰线，可用于去除a元素默认的下划线
  * underline：下划线
  * overline：上划线
  * line-through：中划线（删除线）
* text-transform：设置文字的大小写转换
  * capitalize：每个单词首字母大写
  * uppercase：每个单词所有字符变为大写
  * lowercase：每个单词所有字符变为小写
  * none：没有任何影响
* text-indent：设置第一行内容的缩进。text-indent: 2em;表示缩进2个文字
* text-align：定义行内内容（例如文字）如何相对它的块父元素对齐
  * left：左对齐
  * right：右对齐
  * center：中间对齐
  * justify：两端对齐
* letter-spacing、word-spacing：分别设置字母、单词直接的间距，默认为0，可以为负





## 字体属性

* font-size：设置文字的大小

  * 数值+单位，单位通常为px，也可以用em，1em表示100%
  * 百分比，基于父元素的font-size计算，如50%表示等于父元素font-size的一半

* font-family：设置文字的字体名称，可设置多个，浏览器会选择列表中第一个该计算机上安装的字体，或通过@font-face指定的可以直接下载的字体

* font-weight：设置文字的粗细，常见取值为100-900，normal等于400，bold等于700

* font-style：设置文字的常规、斜体显示

  * normal：常规显示
  * italic：斜体显示，通常有专门的字体
  * oblique：文本倾斜显示（仅仅让文字倾斜）

* font-variant：影响小写字母的显示形式

  * normal：常规显示
  * small-caps：将小写字母替换为缩小过的大写字母

* line-height：设置文本的行高，行高是两行文字基线之间的间距，基线指与小写字母x最底部对齐的线。

  如果div中只有一行文字，设置line-height等于height可以让文字在div内部垂直居中

* font：缩写属性，font-style font-variant font-weight font-size/line-height font-family。

  * font-style、font-variant、font-weight可以随意调换顺序，也可以省略
  * /line-height可以省略，若不省略必须跟在font-size后面
  * font-size、font-family不可调换顺序，不可省略

  > /*  1.5的行高是相对于font-size的 */
  >
  > font: italic smaill-caps 700 30px/1.5 serif;