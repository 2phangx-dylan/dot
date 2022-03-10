### HTML 简介

HTML 称为超文本标记语言（Hyper Text Markup Language），是一种标识性的语言。它包括一系列标签．通过这些标签可以将网络上的文档格式统一，使分散的 Internet 资源连接为一个逻辑整体。HTML 文本是由 HTML 命令组成的描述性文本，HTML 命令可以说明文字，图形、动画、声音、表格、链接等。

### CSS 简介

CSS 称为层叠样式表（Cascading Style Sheets），CSS 中属性对应值，某些属性对应的值可以作为单独的属性，可以为其单独地设置值。

CSS 选择器的某些标签在文档中会被归类为属性，具体需要看属性中的值如何设置，需要阅读范例。

CSS API 文档中属性对应的帮助文档中，会有 JavaScript 对应的设置语法，其中主要依赖于对象的 style 属性来设置样式。

了解到其中最为重要的是 CSS 选择器，可以通过标签名、类名、ID 名等等对网页元素进行样式的设置，不仅仅包括颜色大小像素等等，甚至于元素的位置也可以使用 CSS 进行设置。

### JavaScript 简介

用来辅助设置 HTML 对象的属性，可以实现一些逻辑判断操作。Chrome 插件 Tampermonkey 中的脚本就是需要使用 JavaScript 进行编写的，可以实现对网页动态的修改，查询你想要得到的数据等等。

### XML 简介

XML称为可扩展标记语言（Extensible Markup Language），标准通用标记语言的子集，简称 XML。一种用于标记电子文件使其具有结构性的标记语言。

XML 由于语法十分严格，现多用于编写配置文件。

### Web 相关概念整合

#### 1. 软件架构

C/S：客户端/服务器端，Client/Server

B/S：游览器/服务器端，Browser/Server

#### 2. 资源分类

静态资源：所有用户访问后，得到的结果都是一样的，成为静态资源。静态资源可以直接被游览器解析；
- 如：HTML、CSS、JavaScript

动态资源：每个用户访问相同资源后，得到的结果可能不一样，称为动态资源。动态资源被访问后，需要先转换为静态资源，再返回给浏览器。
- 如：Servlet、Jsp、PHP、Asp

#### 3. 网络通信三要素

IP：电子设备（计算机）在网络中的唯一标识。

端口：应用程序在计算机中的唯一标识，端口号范围 0~65536。

传输协议：规定了数据传输的规则，基础协议包括了 tcp 和 udp。
- tcp：安全协议，三次握手；速度稍慢。
- udp：不安全协议；速度快。

### Test

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, world.");
    }
}
```
