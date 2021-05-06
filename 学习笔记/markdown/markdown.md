# Markdown语法

<div align="right" style="font-size: 30px;"><strong>Riku</strong></div> 

[TOC]

## 1.标题

```markdown
# This is an H1

## This is an H2

###### This is an H6
```

## 2. 块引用

```markdown
> This is a blockquote with two paragraphs. This is first paragraph.
>
> This is second pragraph. Vestibulum enim wisi, viverra nec, fringilla in, laoreet vitae, risus.

> This is another blockquote with one paragraph. There is three empty line to seperate two blockquote.
```

## 3. 列表

```markdown
## un-ordered list
*   Red
*   Green
*   Blue

## ordered list
1.  Red
2. 	Green
3.	Blue
```

## 4. 任务列表

**注意：**

​	这里在里面打上×后，出现可选框，然后选择**完成**或**未完成**

```markdown
- [ ] a task list item
- [ ] list syntax required
- [ ] normal **formatting**, @mentions, #1234 refs
- [ ] incomplete
- [x] completed
```

## 5. 代码块

```markdown
Here's an example:

​```
function test() {
  console.log("notice the blank line before this function?");
}
​```

syntax highlighting:
​```ruby
require 'redcarpet'
markdown = Redcarpet.new("Hello World!")
puts markdown.to_html
​```
```

## 6. 数学块

**注意：**

使用MathJax呈现LaTeX数学表达式。

在markdown源文件中，math块是一个由一对' $$ '标记包装的LaTeX表达式:

```markdown
$$
\mathbf{V}_1 \times \mathbf{V}_2 =  \begin{vmatrix}
\mathbf{i} & \mathbf{j} & \mathbf{k} \\
\frac{\partial X}{\partial u} &  \frac{\partial Y}{\partial u} & 0 \\
\frac{\partial X}{\partial v} &  \frac{\partial Y}{\partial v} & 0 \\
\end{vmatrix}
$$
```

## 7. 表

```markdown
| First Header  | Second Header |
| ------------- | ------------- |
| Content Cell  | Content Cell  |
| Content Cell  | Content Cell  |

| Left-Aligned  | Center Aligned  | Right Aligned |
| :------------ |:---------------:| -----:|
| col 3 is      | some wordy text | $1600 |
| col 2 is      | centered        |   $12 |
| zebra stripes | are neat        |    $1 |
```

## 8. 脚注

**注意：**

这里先在正文中输入内容，在后面输入`[^example]`

在页面下方填写注明即可`[^example]: This is text`

比如这是关于脚注[^1]的说明。

```markdown
You can create footnotes like this[^fn1] and this[^fn2].

[^fn1]: Here is the *text* of the first **footnote**.
[^fn2]: Here is the *text* of the second **footnote**.
```

[^1]: 脚注是一种在文章中，对某些名称或内容的补充说明！

## 9. 水平分界线

```markdown
***
---
```

## 10. YAML风格

```markdown
在文章的一开始处输入---即可
```

> 关于YAML语法，之后会找时间进行说明

## 11. 文章目录

```markdown
输入[toc]即可自动生成文章的目录
```

## 12. 链接

**引用外部链接：**

```markdown
This is [an example](http://example.com/ "Title") inline link.

[This link](http://example.net/) has no title attribute.
```
**在文章内进行链接：**

```markdown
Hold down Cmd (on Windows: Ctrl) and click on [this link](#block-elements) to jump to header `Block Elements`. 
```

**引用链接：**

```markdown
This is [an example][id] reference-style link.

Then, anywhere in the document, you define your link label on a line by itself like this:

[id]: http://example.com/  "Optional Title Here"

[Google][]
And then define the link:

[Google]: http://google.com/
```

## 13. 图片：

```markdown
![Alt text](/path/to/img.jpg)

![Alt text](/path/to/img.jpg "Optional title")
```

**效果图：**

![img](https://raw.githubusercontent.com/mowanqing/img/main/img/v2-23c14209922e3f4d2d18bce89a256171_720w.jpg)

## 14. 强调

```markdown
*single asterisks*

_single underscores_
```

**如果想保留\*或\_应使用转义符 \\**

## 15. 加粗

```markdown
**double asterisks**

__double underscores__
```

## 16. 行内代码块

```markdown
Use the `printf()` function.
```

## 17. 删除线

```markdown
~~Mistaken text.~~ becomes Mistaken text.
```

## 18. Emoji :happy:

```markdown
使用 :simle: 来表示emoji
```

## 19. HTML

```html
<span style="color:red">this text is red</span>
<center>文字居中</center>
```

## 20. 底线

```markdown
<u>Underline</u>
```

## 21. 在Typora中播放视频

1. 通过解析bilibili后得到的视频地址

2. 使用video标签进行播放

**效果图：**

<iframe src="https://player.bilibili.com/player.html?aid=928689451&bvid=BV1TT4y1K72w&cid=282453456&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true" height="350px"> </iframe>

```markdown
<video src="https://upos-sz-mirrorcos.bilivideo.com/upgcxcode/56/34/282453456/282453456-1-208.mp4?e=ig8euxZM2rNcNbRj7zdVhwdlhWTahwdVhoNvNC8BqJIzNbfq9rVEuxTEnE8L5F6VnEsSTx0vkX8fqJeYTj_lta53NCM=&uipk=5&nbs=1&deadline=1620127025&gen=playurlv2&os=cosbv&oi=3662552617&trid=feb4ab2297404c9bad3ed63a9bdaa105T&platform=html5&upsig=f150f0cd8986bc55a0ac937bd006ae99&uparams=e,uipk,nbs,deadline,gen,os,oi,trid,platform&mid=580104086&orderid=0,1&logo=80000000">
```

