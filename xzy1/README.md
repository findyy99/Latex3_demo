# 这是一个来自项子越老师B站课程的例子
## 例子分析
1、首先是使用了`\cs_set:Npn`，创建了一个`my_factorial`函数，这个函数接收一个参数，因为定义的是`my_factorial:n`。
2、接着在函数内部干了两个事情：a. 定义了一个整数变量`l_tmpa_int`，b. 清除`l_tmp_seq`序列的内容。
3、接着使用了一个`int_step_inline`的循环，循环以`my_factorial`函数接收的参数为终点，在循环内部也干了两个事情：a. 将循环变量（类比Python循环中的i）加入到`l_tmpa_seq`，类似Python中列表的`append`方法。b. 将2中定义的整数变量重新赋值给循环变量与`l_tmpa_int`的乘积。
4、最后，使用了`\seq_use:Nn`将`\l_tmpa_seq`中的变量用数学符号（乘法）连接起来，等于连乘的结果——`l_tmpa_int`。
## To do
做一个`cls`把这个求阶乘的函数封装起来。
