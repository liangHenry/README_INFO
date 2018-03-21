---
title: MarkDown-语法说明
date: 2017-07-16
tags: MarkDown
categories: tool 
---

# 简单用法介绍

## 标题  
标题`#`后面最好加一个空格，当不加空格可能会不识别。  
```  
This is h1
==这里两个就有效果，为了划分代码可以加长  
This is h2
--这里两个就有效果，为了划分代码可以加长
# This is h1
## This is h2 
###### This is h6 ##################################
```

<!--more-->

## 字体：  
### 文本：  
hello world  
### 加粗倾斜：  
`***hello world***  ___hello world2___`  
效果：***hello world***   ___hello world2___  

### 加粗：  
`**hello world**  __hello world2__`  
效果：**hello world**  __hello world2__  
### 倾斜：  
`*hello world* _hello world2_`  
效果：*hello world* _hello world2_  
### 删除线：  
`~~hello world~~`  
效果：~~hello world~~  

## 列表   
### 无序列表  
用`*或-或+`+space
```  
* 选项1  
 * 选项1.1  
     * 选项1.1.1  
         * 选项1.1.1.1  
* 选项2  
* 选项3  
```
效果：  

* 选项1  
 * 选项1.1  
     * 选项1.1.1  
         * 选项1.1.1.1  
* 选项2  
* 选项3  

### 有序列表  
`num.`+space
```  
  1. 选项1  
  2. 选项2  
  3. 选项3  
```
效果：    
  1. 选项1  
  2. 选项2  
  3. 选项3  

## 换行
在需要换行的地方输入两个空格，然后回车即可。  
已经换行啦。

## 引用
通过用一个右尖括号`>`。也可以用两个或多个又尖括号表示二级或多级引用。在引用下面输入文字，是属于这个引用的。再加一个空行则退出引用。
```
>这是一级引用。&gt;送一个右尖括号。
>>这是二级引用。
>>>>>这是五级引用。  
测试五级引用内容。
```
效果如下：  
>这是一级引用。&gt;送一个右尖括号。  
>>这是二级引用。  
>>>>>这是五级引用。  
测试五级引用内容。  

## 连接
可以尖括号括起来，也可以`[]( "")`  
An email <cczhaoliang@126.com> link.  
代码如下：
```
An email <cczhaoliang@126.com> link. 
[liang_henry](http://lianghenry.github.io "lianghenry blog")  
See my [About](/about/) page for details.  
```
效果如下：  
An email <cczhaoliang@126.com> link.   
[liang_henry1](http://lianghenry.github.io "lianghenry blog")
See my [About](/about/) page for details.  

## 图片导入
```
![背景图片](https://www.baidu.com/img/bd_logo1.png)
```
效果：  
`![背景图片](https://www.baidu.com/img/bd_logo1.png)`  
![背景图片](https://www.baidu.com/img/bd_logo1.png)  
## 分割线
`*** 或 空行--- 或 - - - -` 三个或三个以上  
***  
- - - -  

---  
# 复杂一点写法  
## 兼容HTML
```
<p>
这是一个普通段落。
</p>
<table>
    <tr>
        <td>Foo</td>
    </tr>
</table>
<p>
这是另一个普通段落。
</p>
```
效果:  
<p>
这是一个普通段落。
</p>
<table>
    <tr>
        <td>Foo</td>
    </tr>
</table>
<p>
这是另一个普通段落。
</p>

## 代码块
可以用一个空行在以4个空格开通,或者用反引号一个或者三个反引号换行 并且可以在反引号后面加入代码语言：  
一个空行在以4个空格开通例子：     

    print("hello world")  
单个反引号例子:    
`print("hello world")` 

三个反引号换行，php语言：  
```  php 
print("hello world");
```

三个反引号换行，c语言: 
```  cpp 
print("hello world");
```


如果要在代码区段内插入反引号，你可以用多个反引号来开启和结束代码区段：   

```
``There is a literal backtick (`) here.``
```

## 特殊字符转换  
例如：`<>和&和©`需要转换成`&lt;&gt;和&amp;和&copy;`  
例如超链接中常用到`&`需要手动转换一下：
`http://images.google.com/image?num=30&q=bird'  
转换成：  
`http://images.google.com/images?num=30&amp;q=larry+bird`

## 隐藏超链接设置
参考式的链接是在链接文字的括号后面再接上另一个方括号，而在第二个方括号里面要填入用以辨识链接的标记：  
`This is [an example][id] reference-style link.`  
你也可以选择性地在两个方括号中间加上一个空格：  
`This is [an example] [id] reference-style link.`  
接着，在文件的任意处，你可以把这个标记的链接内容定义出来：  
`[id]: http://example.com/  "Optional Title Here"`  
备注：`"Optional Title Here"、'Optional Title Here'、(Optional Title Here)`  效果相同.并且title 不区分大小写。如果内容和连接相同可以省略连接`[Google][]`  
效果：
This is [link][id].
[id]: http://lianghenry.github.io "my blog"  
超链接：  

```
[超链接文字](url)
```
锚点链接: 
首先是建立一个跳转的连接：  

```
[说明文字](#jump)
```
然后标记要跳转到的位置：  

```
<span id = "jump">跳转到的位置</span>
```
这样就可以了


## Latex 公式  

可以创建行内公式，例如 `$\Gamma(n) = (n-1)!\quad\forall n\in\mathbb N$`。  
或者块级公式：`$$	x = \dfrac{-b \pm \sqrt{b^2 - 4ac}}{2a} $$`
书写一个质能守恒公式[^LaTeX]`$$E=mc^2$$`  
效果如下：
$\Gamma(n) = (n-1)!\quad\forall n\in\mathbb N$  
$$x = \dfrac{-b \pm \sqrt{b^2 - 4ac}}{2a} $$  
$$E=mc^2$$  

##表格
```
| Item      |    Value | Qty  |
| :-------- | --------:| :--: |
| Computer  | 1600 USD |  5   |
| Phone     |   12 USD |  12  |
| Pipe      |    1 USD | 234  |
```
| Item      |    Value | Qty  |
| :-------- | --------:| :--: |
| Computer  | 1600 USD |  5   |
| Phone     |   12 USD |  12  |
| Pipe      |    1 USD | 234  |

## 流程图[^Flow]  

``` flow
st=>start: Start
e=>end
op=>operation: My Operation
cond=>condition: Yes or No?
st->op->cond
cond(yes)->e
cond(no)->op
```

## 时序图[^Flow]:    

```sequence
Alice->Bob: Hello Bob, how are you?
Note right of Bob: Bob thinks
Bob-->Alice: I am good thanks!
```
## 高效绘制 [甘特图](https://www.zybuluo.com/mdeditor?url=https://www.zybuluo.com/static/editor/md-help.markdown#9-甘特图)

```gantt
    title 项目开发流程
    section 项目确定
        需求分析       :a1, 2016-06-22, 3d
        可行性报告     :after a1, 5d
        概念验证       : 5d
    section 项目实施
        概要设计      :2016-07-05  , 5d
        详细设计      :2016-07-08, 10d
        编码          :2016-07-15, 10d
        测试          :2016-07-22, 5d
    section 发布验收
        发布: 2d
        验收: 3d
```

## 复选框

使用 `- [ ]` 和 `- [x]` 语法可以创建复选框，实现 todo-list 等功能。例如：
```
- [x] 已完成事项
- [ ] 待办事项1
- [ ] 待办事项2
```
效果如下：  
- [x] 已完成事项
- [ ] 待办事项1
- [ ] 待办事项2

[^LaTeX]: 支持 **LaTeX** 编辑显示支持，例如：$\sum_{i=1}^n a_i=0$， 访问 [MathJax][5] 参考更多使用方法。


[^Flow]: **想了解更多，请查看**流程图**[语法][3]以及**时序图**[语法][4]。

  [3]: http://adrai.github.io/flowchart.js/
  [4]: http://bramp.github.io/js-sequence-diagrams/
  [5]: http://meta.math.stackexchange.com/questions/5020/mathjax-basic-tutorial-and-quick-reference
