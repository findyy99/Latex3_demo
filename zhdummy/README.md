# zhdummy
zhdummy是一个例子来自[曾祥东老师的博客](https://stone-zeng.github.io/2019-11-20-l3tutorial-basic/)
## 主要架构
zhdummy/
 ├─zhdummy.sty
 └─test.tex

- `zhdummy.sty`是包`zhdummy`的样式文件。
- `test.tex`是用于测试`zhdummy.sty`的tex文件。
## 具体实现
### 假文的定义
假文的定义是用到了`\tl_const:Nn`函数，这个函数表示创建一个常量的`tl`。后面的`\c_zhdummy_text_i`中的`c`表示该`tl`是一个常量，由于我们的包名是`zhdummy`，故在此将模块名也设为`zhdummy`。接着我们手动创建了一系列的假文，假文来自于《易经》。
### 假文的`直接`使用
假文可以直接使用，下面是直接使用的列子：
```
\documentclass{ctexart}
\usepackage{zhdummy}

\begin{document}
\ExplSyntaxOn
\c_zhdummy_text_i
%或者使用\tl_use
\tl_use:N \c_zhdummy_text_ii
\end{document}
```
但是这样的使用存在一个很大的问题，就是使用的方式过于低效，而且这样的定义在latex的环境中直接使用时会报错：
```
\documentclass{ctexart}
\usepackage{zhdummy}

\begin{document}
\tl_use:N \c_zhdummy_text_i
\end{document}
```
`_` 在常规的类别码设置下代表下标，必须用在数学环境中，所以用 `_` 和 `:` 所定义的命令不能被 LATEX 接受。这会给用户造成了极大的麻烦，显然有悖于我们编写宏包的初衷。因此，接下来我们要创建一些**用户层（或文档层）**命令，以区分于**编程层**。
## 定义用户接口
个人理解，就是将函数的具体实现封装起来，暴露最基本的使用命令给用户，因为上例中的`\c_zhdummy_text_i`使用起来真的太麻烦了。
通过分析可以知道假文的定义有很多的共同点：
- 它们的前面都是统一的`\c_zhdummy_text_`；
- 它们的后面都是罗马数字如`i`、`iv`；
因此，我们可以只选择暴露一个函数可以选择具体假文的位置来输出假文，这样我们可以创建一个命令`zhdummy`:
```
\zhdummy
\zhdummy[1]
```
在不指定假文位置时，默认输出前4段假文，在指定假文顺序时，输出指定的假文。
为了实现这个用户的命令，我们需要解决以下的问题：
- 怎么将阿拉伯数字转化为罗马数字
- 怎么将转化后的数字与命令前段`\c_zhdummy_text_`进行拼接
- 以可靠的方式定义用户层命令

### 数字转换
`expl3`提供了将整数转换为罗马数字的函数：`\int_to_roman:n`，即接收一个整型参数，将其转换为小写罗马数字，相应的有`\int_to_Roman:n`。
可以先试试水（~在latex3中表示空格）:
```
\int_to_roman:n {1} ~
\int_to_roman:n {12} ~
```
结果应当为i xii。
### 拼合命令
类似Python中的`eval`函数，`expl3`也提供了将**字符串**转化为命令的手段。
为此，再回顾一下**参数指定**，它（参数指定）位于一个函数的`:`后面，描述了该函数的参数结构。基本的参数指定包括`n`、`N`、`p`等。例如`\tl_use:N`，表示接受一个token（如一个控制序列）作为参数。
现在，介绍一个新的参数`c`，可以将参数处理成一个控制序列（control sequence）的名称。以下几个写法事等价的：
```
\tl_use:N \c_zhdummy_text_i
\tl_use:c { c_zhdummy_text_i }
\tl_use:c { c _ zhdummy _ text _ i} % 空格在latex3中是自动忽略的
```
## `xparse`宏包简介
正所谓“临门一脚“，我们所有上述的工作都需要面向用户，latex3提供的解决方案是`xparse`宏包，这个宏包可以很方便地声明**用户层**的命令。
因此在底层代码，开发人员应该控制一定的粒度，使得绝大多数函数都只完成单一的工作。因而，底层函数的**参数**应该是确定的。但是从用户使用的角度来说，需求可能千变万化，但是接口应该尽可能保持统一，这就要求**参数形式**具有一顶的多样性。
> 这与C++中依靠函数重载实现的所谓*ad hoc多态*有异曲同工之妙。
`xparse`宏包提供了`\NewDocumentCommand`函数，其语法如下：
```
\NewDocumentCommand <func> {argc-spec} {code}
```
- `<func>`是提供给用户的使用命令，一般来说应该只包含字母，而不包含`_`、`:`、`@`等特殊符号
- `<arg-spec>`是参数指定（个人理解是指定参数的可选或者非可选的属性），可以是：
	- `m`：表示标准必选(*m*andatory)参数，可以是单个的token，或者花括号`{}`包围的一组`tokens`。
	- `o`：表示可选(*o*ptional)参数，需要用**方括号**`[]`包围；若未给出，则返回一个特殊的`-NoValue-`标记。
	- `O{<default>}`：同样为可选参数，但在未给出参数时，默认返回`default`。
	- 示例如下：
##### to do
- `<code>`是具体的实现代码，可以使用`#1`、`#2`这样的参数，这和传统的latex编程是一致的。

上面提到的，输入为*空*时，`o`型参数会返回一个特殊的`-NoValue-`标记。这个标记不是简单的`token list`，它必须通过`\IfNoValue(TF)`函数进行判断：
```
\IfNoValueTF {<arg>} {<true code>} {<false code>}
\IfNoValueT {<arg>} {<true code>}
\IfNoValueF {<arg>} {<false code>}
```
由上代码可知`\IfNoValue`函数会根据返回值是否是`-NoValue-`，选择执行的分支。
## 代码实现
最后，我们可以把代码组合起来了:
```
\NewDocumentCommand \zhdummy { o }
{
	\IfNoValueTF {#1}
	  {
	    \tl_use:N \c_zhdummy_text_i
            \tl_use:N \c_zhdummy_text_ii
            \tl_use:N \c_zhdummy_text_iii
            \tl_use:N \c_zhdummy_text_iv
            \tl_use:N \c_zhdummy_text_v
	  }
	  {
            \tl_use:c { c_zhdummy_text_ \int_to_roman:n {#1}
	  }
}
```
此时，在 test.tex 中即可按照比较常规的方式来使用假文了：

```
% test.tex
\documentclass{ctexart}
\usepackage{zhdummy}

\begin{document}
\zhdummy

\zhdummy[1]
\zhdummy[2]
\zhdummy[18]
\end{document}
```
编译后得到：
> 天地玄黄，宇宙洪荒。日月盈昃，辰宿列张。寒来暑往，秋收冬藏。闰馀成岁，律吕调阳。云腾致雨，露结为霜。
> 天地玄黄，宇宙洪荒。日月盈昃，辰宿列张。化被草木，赖及万方。

### 参考
https://stone-zeng.github.io/2019-11-20-l3tutorial-basic/
