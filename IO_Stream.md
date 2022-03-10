# HTML简介

- 全称为：`Hyper Text Markup Language`。
- `HTML`称为超文本标记语言，是一种标识性的语言。它包括一系列标签．通过这些标签可以将网络上的文档格式统一，使分散的`Internet`资源连接为一个逻辑整体。`HTML`文本是由`HTML`命令组成的描述性文本，`HTML`命令可以说明文字，图形、动画、声音、表格、链接等。

# CSS简介

- 全称为：`Cascading Style Sheets`。
- `CSS`中属性对应值，某些属性对应的值可以作为单独的属性，可以为其单独地设置值。
- `CSS`选择器的某些标签在文档中会被归类为属性，具体需要看属性中的值如何设置，需要阅读范例。
- `CSS API`文档中属性对应的帮助文档中，会有`JavaScript`对应的设置语法，其中主要依赖于对象的`style`属性来设置样式。
- 了解到其中最为重要的是`CSS`选择器，可以通过标签名、类名、`ID`名等等对网页元素进行样式的设置，不仅仅包括颜色大小像素等等，甚至于元素的位置也可以使用`CSS`进行设置。

# JavaScript简介

- 全称为：`JavaScript`。
- 用来辅助设置`HTML`对象的属性，可以实现一些逻辑判断操作。
- `Chrome`插件`Tampermonkey`中的脚本就是需要使用`JavaScript`进行编写的，可以实现对网页动态的修改，查询你想要得到的数据等等。

# XML简介

- 全称为：`Extensible Markup Language`。
- 可扩展标记语言，标准通用标记语言的子集，简称`XML`。一种用于标记电子文件使其具有结构性的标记语言。
- `XML`由于语法十分严格，现多用于编写配置文件。

# IO流（输入输出流）

## 1. File类
- `File`
  
  ```java
  Constructor and Description
		File(String pathname)
  	通过将给定的路径名字符串转换为抽象路径名来创建新的 File实例。
  ```
## 2. OutputStream（字节流）

- `FileOutputStream`

  ```java
  Constructor and Description
  	FileOutputStream(File file)
      创建文件输出流以写入由指定的 File对象表示的文件。
  	FileOutputStream(String name)
      创建文件输出流以指定的名称写入文件。
  ```

- `FilterOutputStream <- BufferedOutputStream`

  ```java
  Constructor and Description
      BufferedOutputStream(OutputStream out)
      创建一个新的缓冲输出流，以将数据写入指定的底层输出流。
  ```

- `FilterOutputStream <- PrintStream`

  ```java
  Constructor and Description
  	PrintStream(File file)
  	使用指定的文件创建一个新的打印流，而不需要自动换行。
      PrintStream(OutputStream out)
  	创建一个新的打印流。
      PrintStream(String fileName)
  	使用指定的文件名创建新的打印流，无需自动换行。
  ```

## 3. InputStream（字节流）

- `FileInputStream`

  ```java
  Constructor and Description
      FileInputStream(File file)
  	通过打开与实际文件的连接创建一个 FileInputStream ，该文件由文件系统中的 File对象 file命名。
      FileInputStream(String name)
  	通过打开与实际文件的连接来创建一个 FileInputStream ，该文件由文件系统中的路径名 name命名。
  ```

- `FilterInputStream <- BufferedInputStream`

  ```java
  Constructor and Description
      BufferedInputStream(InputStream in)
  	创建一个 BufferedInputStream并保存其参数，输入流 in ，供以后使用。
  ```

## 4. Writer（字符流）
- `OutputStreamWriter <- FileWriter`

  ```java
  Constructor and Description
  	OutputStreamWriter(OutputStream out)
  	创建一个使用默认字符编码的OutputStreamWriter。
      OutputStreamWriter(OutputStream out, Charset cs)
  	创建一个使用给定字符集的OutputStreamWriter。
  ```

  ```java
  Constructor and Description
  	FileWriter(File file)
  	给一个File对象构造一个FileWriter对象。
      FileWriter(String fileName)
  	构造一个给定文件名的FileWriter对象。
  ```

- `BufferedWriter`

  ```java
  Constructor and Description
  	BufferedWriter(Writer out)
  	创建使用默认大小的输出缓冲区的缓冲字符输出流。
  ```

## 5. Reader（字符流）
- `InputStreamReader <- FileReader`

  ```java
  Constructor and Description
  	InputStreamReader(InputStream in)
  	创建一个使用默认字符集的InputStreamReader。
      InputStreamReader(InputStream in, Charset cs)
  	创建一个使用给定字符集的InputStreamReader。
  ```
  
  ```java
  Constructor and Description
  	FileReader(File file)
  	创建一个新的 FileReader ，给出 File读取。
      FileReader(String fileName)
  	创建一个新的 FileReader ，给定要读取的文件的名称。
  ```
- `BufferedReader`

  ```java
  Constructor and Description
  	BufferedReader(Reader in)
  	创建使用默认大小的输入缓冲区的缓冲字符输入流。
  ```

# Web相关概念整合

## 1. 软件架构

- `C/S`：客户端/服务器端，`Client/Server`
- `B/S`：游览器/服务器端，`Browser/Server`

## 2. 资源分类

- 静态资源：所有用户访问后，得到的结果都是一样的，成为静态资源。静态资源可以直接被游览器解析；
  - 如：`html`、`css`、`JavaScript`...
- 动态资源：每个用户访问相同资源后，得到的结果可能不一样，称为动态资源。动态资源被访问后，需要先转换为静态资源，再返回给浏览器。
  - 如：`servlet`、`jsp`、`php`、`asp`...

## 3. 网络通信三要素

- `IP`：电子设备（计算机）在网络中的唯一标识。
- 端口：应用程序在计算机中的唯一标识，端口号范围`0~65536`。
- 传输协议：规定了数据传输的规则，基础协议包括了`tcp`和`udp`：
  - `tcp`：安全协议，三次握手；速度稍慢。
  - `udp`：不安全协议；速度快。
