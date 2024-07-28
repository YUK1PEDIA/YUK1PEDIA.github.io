---
layout: post
title: JavaWeb-前端
description: 记录
tag: 笔记
---


# javaweb

## web前端

### 1.Web标准

- Web标准也称为网页标准，由一系列的标准组成
- 三个组成部分：
  - **HTML：负责网页的结构（页面元素和内容）**
  - **CSS：负责网页的表现（页面元素的外观、位置等页面样式，如颜色、大小等）**
  - **JavaScript：负责网页的行为（交互效果）**



### 2.HTML、CSS

什么是HTML、CSS？

- **HTML（HyperText Markup Language）：超文本标记语言**
  - 超文本：超越了文本的限制，比普通文本更强大。除了文字信息，**还可以定义图片、音频、视频等内容**
  - 标记语言：由标签构成的语言
    - HTML标签都是预定义好的。例如：使用展示<a>超链接，使用<img>展示图片，<video>展示视频
    - HTML代码直接在浏览器中运行，HTML标签由浏览器解析
- **CSS（Cascading Style Sheet）：层叠样式表，用于控制页面的样式**



**小结**

- HTML结构标签

```html
<html>
	<head>
		<title>HTML</title>
	</head>
		
	<body>
			
	</body>
</html>
```



- 特点
  - HTML标签**不区分大小写**
  - HTML标签属性值单双引号都可以
  - HTML语法松散





**基础标签&样式**

**标题排版**

1.标题标签

- 标签：`<h1>...</h1>`（h1->h6重要程度依次降低）
- 注意：HTML标签都是预定义好的，不能自己随意定义



2.水平线标签：`<hr>`



3.图片标签：

```
<img src="..." width="..." height="...">
```



- 绝对路径
  - 绝对磁盘路径（本地文件路径）
  - 绝对网络路径（从互联网上加载并展示相应文件，访问必须联网）
- 相对路径：从当前文件开始查找
  - **./：代表当前目录，./可以省略**
  - **../：代表上一级目录，不可省略**

**示例代码**

```html
<!-- 声明文档类型为HTML -->
<!DOCTYPE html>
<html lang="en">
<head>
    <!-- 字符集为UTF-8 -->
    <meta charset="UTF-8">
    <!-- 浏览器兼容性 -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <!-- 标题 -->
    <title>焦点访谈：中国底气</title>
</head>
<body>
    <!-- img标签：
        src：图片资源路径
        width：宽度（px：像素；%，相对于父元素的百分比）
        height：高度 （px：像素；%，相对于父元素的百分比）
        一般只指定一种长度，另外一个长度会等比例缩放
        
        路径书写方式：
            绝对路径：
                1.绝对磁盘路径（本地文件路径）
                2.绝对网络路径（从互联网上加载并展示相应文件，访问必须联网）
            相对路径：
                ./：代表当前目录，./可以省略
                ../：代表上一级目录，不可省略
    -->
    <img src="img/news_logo.png"> 新浪政务 > 正文

    <h1>焦点访谈：中国底气</h1>

    <hr>
    2023年03月02日 21:50 央视网
    <hr>

</body>
</html>
```





**标题样式**

- CSS引入方式
  - 行内样式：`<h1 style="...">`
  - 内嵌样式：`<style>...</style>`
  - 外联样式：xxx.css        `<link herf="...">`
- 颜色标识
  - 关键字：red、green...
  - rgb表示法：rgb(255, 0, 0)、rgb(134, 100, 89)
  - 十六进制：#ff0000、#cccccc、#ccc
- 颜色属性
  - color：设置文本内容的颜色

- `<span>`标签
  - `<span>`是一个在开发网页时大量会用到的没有语义的布局标签
  - 特点：一行可以显示多个（组合行内元素），宽度和高度默认由内容撑开
- CSS选择器
  - 元素选择器：标签名 {...}
  - id选择器：#id属性值 {...}
  - 类选择器：.class属性值 {...}
  - **优先级：id选择器 > 类选择器 > 元素选择器**
- CSS属性
  - color：设置文本的颜色
  - font-size：字体大小

**示例代码**

```html
<!-- 声明文档类型为HTML -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>焦点访谈：中国底气</title>
    <!-- 方式二：内嵌样式，对于当前页面有效，只要是当前页面的h1标题，都应用这个样式 -->
    <style>
        h1 {
            /* color: red; */
            color: #4D4F53;
        }

        /* 元素选择器 */
        /* span {
            color: #968d92;
        } */

        /* 类选择器 */
        /* .cls {
            color: #968d92;
        } */

        /* ID选择器 */
        #time {
            color: #968d92;
            /* 设置字体大小 */
            font-size: 13px; 
        }
    </style>

    <!-- 方式三：外联样式，将样式单独定义在一个css文件中 -->
    <!-- <link rel="stylesheet" href="css/news.css"> -->
</head>
<body>
    <img src="img/news_logo.png"> 新浪政务 > 正文

    <!-- 方式一：行内样式（不推荐，仅仅针对当前标签有效） -->
    <!-- <h1 style="color: red;">焦点访谈：中国底气</h1> -->
    
    
    <h1>焦点访谈：中国底气</h1>

    <hr>
    <span class="cls" id="time">2023年03月02日 21:50</span>   <span class="cls">央视网</span>
    <hr>

</body>
</html>
```



**超链接**

- 超链接
  - 标签：`<a>`
  - 属性：
    - herf：指定资源访问的url
    - target：指定在何处打开资源链接
      - _self：默认值，在当前页面打开
      - _blank：在空白页面打开
- CSS属性
  - text-decoration：规定添加到文本的修饰，**none表示定义标准的文本（如去除超链接的下划线等）**
  - color：定义文本颜色

**示例代码**

```html
<!-- 声明文档类型为HTML -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>焦点访谈：中国底气</title>
    <style>
        h1 {
            color: #4D4F53;
        }

        #time {
            color: #968d92;
            /* 设置字体大小 */
            font-size: 13px; 
        }

        a {
            color: black;
            /* 设置文本为标准文本，去除下划线 */
            text-decoration: none;
        }
    </style>

</head>
<body>
    <img src="img/news_logo.png"> <a href="http://gov.sina.com.cn/" target="_self">新浪政务</a>  > 正文
    
    <h1>焦点访谈：中国底气</h1>

    <hr>
    <span class="cls" id="time">2023年03月02日 21:50</span>   <span> <a href="https://news.cctv.com/2023/03/02/ARTIUCKFf9kE9eXgYE46ugx3230302.shtml" target="_blank">央视网</a> </span>
    <hr>

</body>
</html>
```





**正文排版**

- 音频、视频标签

```
<audio>、<video> 注意使用这两个标签时需要加上controls，将播放键显示出来
```

- 换行、段落标签

```
换行：<br> ；段落：<p>
```

- 文本加粗标签

```
<b>  <strong>两种标签
```

- CSS样式
  - line-height：设置行高
  - text-indent：定义第一个行内容的缩进
  - text-align：规定元素中的文本的水平对齐方式
- 注意

**在HTML中无论输入多少个空格，都只会显示一个。要显示多个空格，可以使用空格占位符：&nbsp**

**示例代码**

```html
<!-- 声明文档类型为HTML -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>焦点访谈：中国底气</title>
    <style>
        h1 {
            color: #4D4F53;
        }

        #time {
            color: #968d92;
            /* 设置字体大小 */
            font-size: 13px; 
        }

        a {
            color: black;
            /* 设置文本为标准文本，去除下划线 */
            text-decoration: none;
        }

        p {
            /* 设置首行缩进 */
            text-indent: 35px;
            /* 设置行高 */
            line-height: 40px;
        }

        #plast {
            /* 对齐方式 */
            text-align: right;
        }
    </style>

</head>
<body>
    <img src="img/news_logo.png"> <a href="http://gov.sina.com.cn/" target="_self">新浪政务</a>  > 正文
    
    <h1>焦点访谈：中国底气</h1>

    <hr>
    <span class="cls" id="time">2023年03月02日 21:50</span>   <span> <a href="https://news.cctv.com/2023/03/02/ARTIUCKFf9kE9eXgYE46ugx3230302.shtml" target="_blank">央视网</a> </span>
    <hr>

    <!-- 正文 -->
    <!-- 定义视频 -->
    <video src="video/1.mp4" controls width="950px"></video>
    <!-- 定义音频 -->
    <!-- <audio src="audio/1.mp3" controls></audio> -->

    <p>
        <strong>央视网消息</strong> （焦点访谈）：党的十八大以来，以习近平同志为核心的党中央始终把解决粮食安全问题作为治国理政的头等大事，重农抓粮一系列政策举措有力有效，我国粮食产量站稳1.3万亿斤台阶，实现谷物基本自给、口粮绝对安全。我们把饭碗牢牢端在自己手中，为保障经济社会发展提供了坚实支撑，为应对各种风险挑战赢得了主动。连续八年1.3万亿斤，这个沉甸甸的数据是如何取得的呢？
    </p>
    
    <p>
        人勤春来早，春耕农事忙。立春之后，由南到北，我国春耕春管工作陆续展开，春天的田野处处生机盎然。
    </p>

    <img src="img/1.jpg">

    <p>
        今年，我国启动了新一轮千亿斤粮食产能提升行动，这是一个新的起点。2015年以来，我国粮食产量连续8年稳定在1.3万亿斤以上，人均粮食占有量始终稳稳高于国际公认的400公斤粮食安全线。从十年前的约12200亿斤到2022年的约13700亿斤，粮食产量提高了1500亿斤。
    </p>

    <img src="img/2.jpg">

    <p>
        中国式现代化一个重要的中国特色是人口规模巨大的现代化。我们粮食生产的发展，意味着我们要立足国内，解决14亿多人吃饭的问题。仓廪实，天下安。保障粮食安全是一个永恒的课题，任何时候都不能放松。在以习近平同志为核心的党中央坚强领导下，亿万中国人民辛勤耕耘、不懈奋斗，我们就一定能够牢牢守住粮食安全这一“国之大者”，把中国人的饭碗牢牢端在自己手中，夯实中国式现代化基础。
    </p>


    <p id="plast">
        责任编辑：王树淼 SN242
    </p>


</body>
</html>
```





**页面布局**

**盒子模型**

- 盒子：页面中所有的元素（标签），都可以看作是一个**盒子**，由盒子将页面中的元素包含在一个矩形区域内，通过盒子的视角更方便的进行页面布局
- 盒子模型组成：**内容区域（content）、内边距区域（padding）、边框区域（border）、外边距区域（margin）**

**注意：如果只需要设置某一个方位的边框、内边距、外边距，可以在属性名后加上：“-位置”，如：padding-top、padding-left、padding-right、padding-bottom...**

![6e2fc6a81a4a3c9a693691eefa73d172_720.jpg](https://s2.loli.net/2024/07/28/QSWTp1ogNc5PF8X.jpg) 

**其中margin并不包含在盒子之内** 



- 布局标签：实际开发网页时，会大量地使用div和span这两个没有语义的布局标签
- 标签：`<div> <span>`
- 特点
  - div标签
    - 一行只显示一个（独占一行）
    - 宽度默认是父元素的宽度，高度默认由内容撑开
    - 可以设置宽高
  - span标签
    - 一行可以显示多个
    - 宽度和高度默认由内容撑开
    - 不可以设置宽高

**示例代码**

```html
<!-- 声明文档类型为HTML -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>焦点访谈：中国底气</title>
    <style>
        h1 {
            color: #4D4F53;
        }

        #time {
            color: #968d92;
            /* 设置字体大小 */
            font-size: 13px; 
        }

        a {
            color: black;
            /* 设置文本为标准文本，去除下划线 */
            text-decoration: none;
        }

        p {
            /* 设置首行缩进 */
            text-indent: 35px;
            /* 设置行高 */
            line-height: 40px;
        }

        #plast {
            /* 对齐方式 */
            text-align: right;
        }
        
        #center {
            width: 65%;
            /* 外边距：上、右、下、左 */
            /* margin: 0% 17.5% 0% 17.5%; */

            /* 若margin只指定两个值，就代表上下外边距为第一个值，左右外边距为第二个值 */
            /* auto代表自动计算外边距 */
            margin: 0 auto;  
        }
    </style>

</head>
<body>
    <div id="center">
        <img src="img/news_logo.png"> <a href="http://gov.sina.com.cn/" target="_self">新浪政务</a>  > 正文
        
        <h1>焦点访谈：中国底气</h1>

        <hr>
        <span class="cls" id="time">2023年03月02日 21:50</span>   <span> <a href="https://news.cctv.com/2023/03/02/ARTIUCKFf9kE9eXgYE46ugx3230302.shtml" target="_blank">央视网</a> </span>
        <hr>

        <!-- 正文 -->
        <!-- 定义视频 -->
        <video src="video/1.mp4" controls width="950px"></video>
        <!-- 定义音频 -->
        <!-- <audio src="audio/1.mp3" controls></audio> -->

        <p>
            <strong>央视网消息</strong> （焦点访谈）：党的十八大以来，以习近平同志为核心的党中央始终把解决粮食安全问题作为治国理政的头等大事，重农抓粮一系列政策举措有力有效，我国粮食产量站稳1.3万亿斤台阶，实现谷物基本自给、口粮绝对安全。我们把饭碗牢牢端在自己手中，为保障经济社会发展提供了坚实支撑，为应对各种风险挑战赢得了主动。连续八年1.3万亿斤，这个沉甸甸的数据是如何取得的呢？
        </p>
        
        <p>
            人勤春来早，春耕农事忙。立春之后，由南到北，我国春耕春管工作陆续展开，春天的田野处处生机盎然。
        </p>

        <img src="img/1.jpg">

        <p>
            今年，我国启动了新一轮千亿斤粮食产能提升行动，这是一个新的起点。2015年以来，我国粮食产量连续8年稳定在1.3万亿斤以上，人均粮食占有量始终稳稳高于国际公认的400公斤粮食安全线。从十年前的约12200亿斤到2022年的约13700亿斤，粮食产量提高了1500亿斤。
        </p>

        <img src="img/2.jpg">

        <p>
            中国式现代化一个重要的中国特色是人口规模巨大的现代化。我们粮食生产的发展，意味着我们要立足国内，解决14亿多人吃饭的问题。仓廪实，天下安。保障粮食安全是一个永恒的课题，任何时候都不能放松。在以习近平同志为核心的党中央坚强领导下，亿万中国人民辛勤耕耘、不懈奋斗，我们就一定能够牢牢守住粮食安全这一“国之大者”，把中国人的饭碗牢牢端在自己手中，夯实中国式现代化基础。
        </p>


        <p id="plast">
            责任编辑：王树淼 SN242
        </p>
    </div>

</body>
</html>
```





**表格、表单标签**

**表格标签**

- 场景：在网页中以表格形式整齐展示数据，如：班级表
- 标签

![1.jpg](https://s2.loli.net/2024/07/28/DQWx6qfNyV5eKGC.jpg) 

**注意：必须有border才能看见表格的边框**

**示例代码**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>HTML-表格</title>
    <style>
        td {
            text-align: center; /* 单元格内容居中展示 */
        }
    </style>
</head>
<body>
    
    <table border="1px" cellspacing="0"  width="600px">
        <tr>
            <th>序号</th>
            <th>品牌Logo</th>
            <th>品牌名称</th>
            <th>企业名称</th>
        </tr>
        <tr>
            <td>1</td>
            <td> <img src="img/huawei.jpg" width="100px"> </td>
            <td>华为</td>
            <td>华为技术有限公司</td>
        </tr>
        <tr>
            <td>2</td>
            <td> <img src="img/alibaba.jpg"  width="100px"> </td>
            <td>阿里</td>
            <td>阿里巴巴集团控股有限公司</td>
        </tr>
    </table>

</body>
</html>
```





**表单标签**

- 场景：在网页中主要负责数据采集功能，如登录、注册等数据收集
- 标签：`<form>`

- 表单项：不同类型的input元素、下拉列表、文本域等

```
<input>：定义表单项，通过type属性控制输入形式
<select>：定义下拉列表
<textarea>：定义文本域
```

- 属性
  - action：规定当提交表单时向何处发送表单数据，URL
  - method：规定用于发送表单数据的方式。GET、POST

**注意：表单项必须有name属性才可以提交**

**示例代码**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>HTML-表单</title>
</head>
<body>
    <!-- 
    form表单属性: 
        action: 表单提交的url, 往何处提交数据 . 如果不指定, 默认提交到当前页面
        method: 表单的提交方式 .
            get: 在url后面拼接表单数据, 比如: ?username=Tom&age=12 , url长度有限制 . 默认值
            post: 在消息体(请求体)中传递的, 参数大小无限制的.
    -->   
    <form action="" method="post">
        用户名：<input type="text" name="username">
        年龄：<input type="text" name="age">

        <input type="submit" value="提交">

    </form>


</body>
</html>
```





**表单项**

```
<input>：表单项，通过type属性控制输入形式
<select>：定义下拉列表，<option>定义列表项
<textarea>：文本域
```

![1.jpg](https://s2.loli.net/2024/07/28/IeC576Ynh9MRcQr.jpg) 

**示例代码**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>HTML-表单项标签</title>
</head>
<body>

<!-- value: 表单项提交的值 -->
<form action="" method="post">
     姓名: <input type="text" name="name"> <br><br>
     密码: <input type="password" name="password"> <br><br> 
     性别: <label><input type="radio" name="gender" value="1"> 男</label>
          <label><input type="radio" name="gender" value="2"> 女 </label> <br><br>
     爱好: <label><input type="checkbox" name="hobby" value="java"> java </label>
          <label><input type="checkbox" name="hobby" value="game"> game </label>
          <label><input type="checkbox" name="hobby" value="sing"> sing </label> <br><br>
     图像: <input type="file" name="image">  <br><br>
     生日: <input type="date" name="birthday"> <br><br>
     时间: <input type="time" name="time"> <br><br>
     日期时间: <input type="datetime-local" name="datetime"> <br><br>
     邮箱: <input type="email" name="email"> <br><br>
     年龄: <input type="number" name="age"> <br><br>
     学历: <select name="degree">
               <option value="">----------- 请选择 -----------</option>
               <option value="1">大专</option>
               <option value="2">本科</option>
               <option value="3">硕士</option>
               <option value="4">博士</option>
          </select>  <br><br>
     描述: <textarea name="description" cols="30" rows="10"></textarea>  <br><br>
     <input type="hidden" name="id" value="1">

     <!-- 表单常见按钮 -->
     <input type="button" value="按钮">
     <input type="reset" value="重置"> 
     <input type="submit" value="提交">   
     <br>
</form>

</body>
</html>
```





### 3.JavaScript

**引入方式**

- 内部脚本引入
- 外部脚本引入

**示例代码**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>JS-引入方式</title>
    <!-- 内部脚本 -->
    <!-- <script>
        alert('hello js');
    </script> -->

    <!-- 外部脚本，且script标签不能自闭和 -->
    <script src="js/demo.js"></script>
</head>
<body>

</body>
</html>
```



**JS基础语法**

- **书写语法**

  - 区分大小写

  - 每行结尾分号可有可无

  - 注释和java一样，单行、多行

  - 大括号表示代码块

  ```javascript
  // 判断
  if (cnt == 3) {
  	alert(cnt);
  }
  ```

- **输出语句**
  - **使用 window.alert() 写入警告框**
  - **使用 document.write() 写入HTML输出**
  - **使用 console.log() 写入浏览器控制台** 

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>JS-基本语法</title>
</head>
<body>
    
</body>
<script>
    /* alert("JS"); */

    // 方式一：弹出警告框
    window.alert("hello.js");

    // 方式二：写入html页面
    document.write("hello.js");

    // 方式三：浏览器控制台输出
    console.log("hello.js");
</script>
</html>
```



- **变量**
  - JavaScript中用 **var** 关键字（variable的缩写）来声明变量
  - JavaScript是一门弱类型语言，变量可以存放不同类型的值
  - 变量名需要遵循如下规则
    - 组成字符可以是任何字母、数字、下划线或美元符号
    - **数字不能开头**
    - 建议用驼峰命名

```javascript
var a = 20;
a = "张三";
```



**注意**：JavaScript中除了用 **var** 可以声明变量，还可以用 **let** 和 **const** 声明变量

**var** 声明的变量作用域大，是**全局变量，可以重复定义**

**let** 所声明的变量**是局部变量，不能重复定义**

**const** 声明的变量是**常量，不能改变**



**示例代码**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>JS-基础语法</title>
</head>
<body>
    
</body>
<script>
    // var定义变量
    /* var a = 10;
    a = "张三";
    alert(a); */

    // 特点1：var作用域大，属于全局变量
    // 特点2：var可以重复定义
    /* {
        var x = 1;
        var x = "A";
    }
    alert(x); */

    // let定义变量：局部变量，且不能重复定义
    /* {
        let x = 1;
    }
    alert(x); */

    // const：常量，不能改变
    const pi = 3.14;
    pi = 3.15
    alert(pi);

</script>
</html>
```



- **数据类型、运算符、流程控制语句**

  JavaScript中分为：**原始类型**和**引用类型（对象）**

  - **原始类型**

    - number：数字（整数、小数、NaN(Not a Number)）
    - string：字符串，单双引号皆可
    - boolean
    - null
    - undefined：当声明的变量未初始化，该变量的默认值为undefined

    **使用 typeof 运算符可以获取数据类型**

  - **运算符**

    和 java 运算符基本相同，但是多了一个比较运算符“===”

    - **“===”用于判断是否相等，且不会进行类型转换（即必须要两个变量类型和值都相等，才会判断相等）**

  - **类型转换**

    - 字符串转为数字：将字符串字面值转为数字。**如果字面值不是数字，则转为NaN**
    - 其他类型转为boolean
      - Number：0和 NaN 为false，其他均为true
      - String：空字符串为false，其余为true
      - Null 和 undefined：均为false

  - **流程控制语句**

    和 java 相同

**示例代码**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>JS-运算符</title>
</head>
<body>
    
</body>
<script>
    var age = 20;
    var _age = "20";
    var $age = 20;
    
    // alert(age == _age);
    // alert(age === _age);
    // alert(age === $age);


    // 类型转换 - 其他类型转为数字
    alert(parseInt("12")); // 12
    alert(parseInt("12A45")); // 12，从头往后匹配，如果遇到不是数字的内容，则停止匹配
    alert(parseInt("A45"));


    // 类型转换 - 其他类型转为boolean
    // if(0){
    //     alert("0 转换为false");
    // }
    // if(NaN){
    //     alert("NaN 转换为false");
    // }
    // if(1){
    //     alert("除0和NaN其他数字都转为 true");
    // }

    // if(""){
    //     alert("空字符串为 false, 其他都是true");
    // }
        
    // if(null){
    //     alert("null 转化为false");
    // }
    // if(undefined){
    //     alert("undefined 转化为false");
    // }

</script>
</html>
```



- **函数**

  - js 函数通过 `function` 关键字进行定义

  ```javascript
  function functionName(参数1, 参数2...) {
  	// 要执行的代码
  }
  ```

  - 参数不需要类型，**返回值也不需要定义类型**，直接在函数内部 return 返回即可
  - 调用：函数名(实际参数列表)
  - 另一种定义方式：

  ```javascript
  var functionName = function(参数1, 参数2) {
  	// 要执行的代码
  }
  ```

  - 函数可以接受任意个参数，但超出参数个数的参数并没有用



- **对象**

  -  基础对象：Array、String、JSON

    - Array对象的定义、属性和方法

      - 定义

      ```javascript
      var array = new Array(元素列表); // 方式一
      var arr = new Array(1,2,3,4);
      
      var array = [元素列表]; // 方式二
      var arr = [1,2,3,4];
      // 数组访问和 java 一样，arr[index] = value
      
      // 特点：数组的长度类型均可变
      var arr1 = [1,2,3,4];
      arr1[10] = 50; // 不会报错
      arr1[9] = true;
      arr1[8] = 'A';
      ```

      - 属性：length ：设置或返回数组中元素的数量

      ```javascript
      var arr = [1,2,3,4];
      for (let index = 0; index < arr.length; i++) {
      	console.log(arr[index]);
      }
      ```

      - 方法

      ```javascript
      var arr = [1,2,3,4];
      
      // forEach：遍历数组中有值的元素，如果数组中有的值未定义，forEach不会遍历
      arr.forEach(function(e) {
      	console.log(e);
      })
      
      // 以上代码用箭头函数可简化 -- 简化函数定义
      // 箭头函数：(...) => {...}
      // 如果给箭头函数起名：var xxx = (...) => {...}
      arr.forEach((e) => {
          console.log(e);
      })
      
      
      // push：添加元素到数组末尾
      arr.push(7,8,9);
      console.log(arr);
      
      
      // splice：删除元素
      arr.splice(start, delcnt); // 从start索引开始删除delcnt个元素
      arr.splice(2, 2); // 从索引为2的位置开始，删除2个元素
      console.log(arr); // 删掉了3和4
      ```

    - String对象的定义、属性和方法

      - 定义

      ```javascript
      var 变量名 = new String("..."); // 方式一
      var str = new String("hello string");
      
      var 变量名 = "..."; // 方式二
      // 单引号和双引号相同
      var str = "hello string";
      var str = 'hello string';
      ```

      - 属性：length：获取长度

      ```javascript
      var str = "hello string";
      console.log(str.length); // 输出为12
      ```

      - 方法

      ```javascript
      var str = "hello string";
      // charAt：获取指定位置的字符
      console.log(charAt(3)); // 输出为l
      
      // indexOf：检索子字符串在字符串的位置
      console.log("indexOf("lo")"); // 输出3
      
      // trim：去除字符串左右两侧的空格
      var str = "   hello string    ";
      var s = str.trim();
      console.log(s); // 输出为"hello string"
      
      // substring(start, end) 截取字符串，含头不含尾
      console.log(s.substring(0, 5)); // 输出为"hello"
      ```

    - JSON对象

      - 定义（**JSON本质是字符串**，用字符串的方式定义）

      ```javascript
      var 变量名 = '{"key1": value1, "key2": value2}';
      var userStr = '{"name":"Jerry", "age":18, "addr":["北京","上海","西安"]}';
      // 此时不能通过 userStr.name 的方式获取 name ，因为 userStr 此时是字符串，并不是对象
      ```

      - JSON 字符串和 JS 对象的相互转换

      ```javascript
      var 变量名 = '{"key1": value1, "key2": value2}';
      var userStr = '{"name":"Jerry", "age":18, "addr":["北京","上海","西安"]}';
      
      // json字符串--js对象
      var obj = JSON.parse(jsonstr);
      console.log(obj.name);
      
      // js对象--json字符串
      var jsonStr = JSON.stringify(obj);
      console.log(jsonStr);
      ```

  - BOM对象：浏览器对象模型，允许 JS 和浏览器对话

    - 组成：**Window浏览器窗口对象**、Navigator浏览器对象、Screen屏幕对象、History历史记录对象、**Location地址栏对象**

    - Window浏览器窗口对象

      - 获取：直接使用 window ，其中 window 可以省略

      ```javascript
      window.alert("hello window");
      alert("hello window");
      ```

      - 属性
        - history：对 History 对象的只读引用
        - location：用于窗口或框架的 Location 对象
        - navigator：对 Navigator 对象的只读引用
      - 方法
        - alert()：显示带有一段信息和一个确认按钮的警告框
        - confirm()：显示带有一段信息以及确认按钮和取消按钮的对话框
        - setInterval()：按照指定的周期（以毫秒记）来调用函数或计算表达式
        - setTimeout()：在指定的毫秒数后调用函数或计算表达式

    - Location地址栏对象

      - 获取：使用 window.location 获取，其中 window. 可省略

      ```javascript
      window.location.属性;
      location.属性;
      ```

      - 属性

        - href：设置或返回完整的 URL

        ```javascript
        location.href = "https://www.itcast.cn";
        ```
    
    - 示例代码
    
    ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>JS-对象-BOM</title>
    </head>
    <body>
        
    </body>
    <script>
        //获取
        window.alert("Hello BOM");
        alert("Hello BOM Window");
    
        //方法
        //confirm - 对话框 -- 确认: true , 取消: false
        var flag = confirm("您确认删除该记录吗?");
        alert(flag);
    
        //定时器 - setInterval -- 周期性的执行某一个函数
        var i = 0;
        setInterval(function(){
            i++;
            console.log("定时器执行了"+i+"次");
        },2000);
    
        //定时器 - setTimeout -- 延迟指定时间执行一次 
        setTimeout(function(){
            alert("JS");
        },3000);
    
        //location
        alert(location.href);
    
        location.href = "https://www.itcast.cn";
    
    </script>
    </html>
    ```
    
    
    
    

  - DOM对象：文档对象模型

    -  将标记语言的各个组成部分封装为对应的对象
      -  Document：整个文档对象
      -  Element：元素对象
      -  Attribute：属性对象
      -  Text：文本对象
      -  Comment：注释对象

    -  JS 通过 DOM ，就能够对 HTML 进行操作
      -  改变 HTML 元素的内容
      -  改变 HTML 元素的样式
      -  对 HTML DOM 事件作出反应
      -  添加和删除 HTML 元素


```html
<html>
	<head>
		<title>DOM</title>
	</head>
	<body>
		<h1>DOM对象标题</h1>
		<a href="https://itcast.cn">传智教育</a>
	</body>
</html>
```

![1.png](https://s2.loli.net/2024/07/26/hgaxJ3dC5H8eszt.png)  




- 事件监听

  - 事件绑定

  ```html
  // 方式1：通过 html 标签中的事件属性进行绑定
  <input type="button" onclick="on()" value="按钮1">
  
  <script>
  	function on() {
  		alert('我被点击了！'');
  	}
  </script>
  
  
  // 方式2：通过 DOM 元素属性绑定
  <input type="button" id="btn" value="按钮2">
  
  <script>
  	document.getElementById('btn').onclick=function() {
  	alert('我被点击了！');
  	}
  </script>
  ```

  - 常见事件

  ![1.png](https://s2.loli.net/2024/07/26/Y4dP9rAtLH76pfV.png) 





### 4.Vue

Vue 是一套前端框架，免除原生 JavaScript 中的 DOM 操作，简化书写。基于 MVVM(Model-View-ViewModel) 思想，实现数据的双向绑定，将编程的关注点放在数据上



- Vue 快速入门

  - 新建 html 页面，引入 Vue.js 文件

  ```html
  <script src="js/vue.js"></script>
  ```

  - 在 JS 代码区域，创建 Vue 核心对象，定义数据模型

  ```html
  <script>
  	new Vue({
  		el: "#app",
  		data: {
  			message: "hello vue!"
  		}
  	})
  </script>
  ```

  - 编写视图

  ```html
  <div id="app">
  	<input type="text" v-model="message">
  	{{ message }}
  </div>
  ```

- Vue 常用指令

  - html 标签上带有 **v- 前缀**的特殊属性，不同指令具有不同含义

   ![1.png](https://s2.loli.net/2024/07/26/biuyI1Yr4WFNpDS.png)

  - v-bind 和 v-on

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta http-equiv="X-UA-Compatible" content="IE=edge">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Vue-指令-v-bind</title>
      <script src="js/vue.js"></script>
  </head>
  <body>
      <div id="app">
  
          <a v-bind:href="url">链接1</a>
          <a :href="url">链接2</a>
  
          <input type="text" v-model="url">
  
      </div>
  </body>
  <script>
      //定义Vue对象
      new Vue({
          el: "#app", //vue接管区域
          data:{
             url: "https://www.baidu.com"
          }
      })
  </script>
  </html>
  ```

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta http-equiv="X-UA-Compatible" content="IE=edge">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Vue-指令-v-on</title>
      <script src="js/vue.js"></script>
  </head>
  <body>
      <div id="app">
  
          <input type="button" value="点我一下" v-on:click="handle()">
  
          <input type="button" value="点我一下" @click="handle()">
  
      </div>
  </body>
  <script>
      //定义Vue对象
      new Vue({
          el: "#app", //vue接管区域
          data:{
             
          },
          methods: {
              handle: function(){
                  alert("你点我了一下...");
              }
          }
      })
  </script>
  </html>
  ```

  - v-if 和 v-show

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta http-equiv="X-UA-Compatible" content="IE=edge">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Vue-指令-v-if与v-show</title>
      <script src="js/vue.js"></script>
  </head>
  <body>
      <div id="app">
          
          年龄<input type="text" v-model="age">经判定,为:
          <span v-if="age <= 35">年轻人(35及以下)</span>
          <span v-else-if="age > 35 && age < 60">中年人(35-60)</span>
          <span v-else>老年人(60及以上)</span>
  
          <br><br>
  
          年龄<input type="text" v-model="age">经判定,为:
          <span v-show="age <= 35">年轻人(35及以下)</span>
          <span v-show="age > 35 && age < 60">中年人(35-60)</span>
          <span v-show="age >= 60">老年人(60及以上)</span>
  
      </div>
  </body>
  <script>
      //定义Vue对象
      new Vue({
          el: "#app", //vue接管区域
          data:{
             age: 20
          },
          methods: {
              
          }
      })
  </script>
  </html>
  ```

  - v-for

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta http-equiv="X-UA-Compatible" content="IE=edge">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Vue-指令-v-for</title>
      <script src="js/vue.js"></script>
  </head>
  <body>
      <div id="app">
  
          <div v-for="addr in addrs">{{addr}}</div>
  
          <hr>
  
          <div v-for="(addr,index) in addrs">{{index + 1}} : {{addr}}</div>
  
      </div>
  </body>
  <script>
      //定义Vue对象
      new Vue({
          el: "#app", //vue接管区域
          data:{
             addrs:["北京", "上海", "西安", "成都", "深圳"]
          },
          methods: {
              
          }
      })
  </script>
  </html>
  ```



- Vue 生命周期

![1.png](https://s2.loli.net/2024/07/26/wiIagyclx4uTVLh.png) 

一般来说只用掌握 mounted 状态。

mounted：挂载完成，Vue 初始化成功，html 页面渲染成功。（**发送请求到服务端，加载数据**）



- 前端项目工程化

  - 默认首页： index.html、入口文件：main.js
  - Vue 的组件文件以 `.vue` 结尾，每个组件由三个部分组成：`<template>、<script>、<style>` 。

  ![1.png](https://s2.loli.net/2024/07/28/cDyauM6jvO8Ti4b.png) 



- Element-UI 组件库

  - 如何使用 Element-UI 组件库

    - 在当前工程的目录终端下执行

    ```
    npm install element-ui@2.15.3
    ```

    - 在 main.js 中引入 Element-UI 组件库

    ```js
    import ElementUI from 'element-ui';
    import 'element-ui/lib/theme-chalk/index.css';
    
    Vue.use(ElementUI);
    ```

  - 访问官网复制组件代码即可



- Vue 路由
  - 前端路由：URL 中的 hash 与组件之间的对应关系
  - Vue Router：Vue 的官方路由
    - 组成
      - `VueRouter` ：路由器类，根据路由请求在路由视图中动态渲染选中的组件
      - `<router-link>` ：请求链接组件，浏览器会解析成 `<a>`
      - `<router-view>` ：动态视图组件，用来渲染展示与路由路径对应的组件

![1.png](https://s2.loli.net/2024/07/28/TI81Gseov9QALHf.png) 







### 5.Ajax

- 概念：Asynchronous JavaScript And XML，异步的 JavaScript 和 XML

- 作用

  - 数据交换：通过 Ajax 可以给服务器发送请求，获取服务器响应数据
  - 异步交互，可以在**不重新加载整个页面**的情况下，与服务器交换数据并更新部分网页的技术，比如搜索联想、用户名是否可用的校验等等

- Axios：对原生的 Ajax 了封装，简化书写

  - 要使用 Axios ，首先需要引入 Axios 的 js 文件，再使用 Axios 发送请求获取响应结果
  - Axios 自带了请求方式别名
    - `axios.get(url [, config])`
    - `axios.delete(url [, config])`
    - `axios.post(url [, data[, config]])`
    - `axios.put(url [, data[, config]])`

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta http-equiv="X-UA-Compatible" content="IE=edge">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Ajax-Axios</title>
      <script src="js/axios-0.18.0.js"></script>
  </head>
  <body>
      
      <input type="button" value="获取数据GET" onclick="get()">
  
      <input type="button" value="删除数据POST" onclick="post()">
  
  </body>
  <script>
      function get(){
          // 通过axios发送异步请求-get
          axios({
              method: "get",
              url: "http://yapi.smart-xwork.cn/mock/169327/emp/list"
          }).then(result => {
              console.log(result.data);
          })
  
          // 使用axios自带的get别名
          axios.get("http://yapi.smart-xwork.cn/mock/169327/emp/list").then(result => {
              console.log(result.data);
          })
      }
  
      function post(){
          // 通过axios发送异步请求-post
          axios({
              method: "post",
              url: "http://yapi.smart-xwork.cn/mock/169327/emp/deleteById",
              data: "id=1"
          }).then(result => {
              console.log(result.data);
          })
  
          // 使用axios自带的post别名
          axios.post("http://yapi.smart-xwork.cn/mock/169327/emp/deleteById","id=1").then(result => {
              console.log(result.data);
          })
      }
  </script>
  </html>
  ```





### 关于打包部署

可以用 Nginx 进行本地部署测试

