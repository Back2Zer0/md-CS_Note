Vim 学习笔记

> b站用户：木羽寻清
>
>  vim普通模式的理念是：移动和操作
> 移动：就是光标移动，快速定位光标
> 操作:操作后接范围，这个范围①以当前光标为起点，以一次移动为终点。②可以以文本对象作为范围
> 文本对象：官方定义为双引号“”，单引号''，大括号{}小括号【】等里面的内容。
> 文本对象才是能极大提高编辑速度的关键，我们首先要搞清楚vim可以解决的文件编辑问题:一个文件对于操作系统是最小的单位，对于里面的字符却是最大的单位。
> 我们可以将整个文件逐级分割来更准确的编辑:屏幕、段落、行、单词、字符。我们的上下左右/hjkl对应的是字符级。e，w，b对应单词级。$，0，f对应行级。{}对应段落级。反正随你怎么分，只要能理解这些都是文本对象就好，文本对象可以确定操作范围 





三种模式：普通模式、命令模式、文本编辑模式、可视模式。

h j k l 普通模式下的文本移动键。



输入一个数字，再按移动键，可以跳行。

比如 4j 是向下跳4行。



w 键可以跳到下一个单词的开头，b 键跳到上一个单词的开头



双击 G ：回到最上方，对应home键

大写的G：回到最下方，对应 end 键



Ctrl+U

Ctrl+d  ：Page Down 向下翻页



单击 f ：寻找键。

f + 字母：寻找到最近的字母开头的单词位置。



单击 y：复制

> `yaw`: yank all word ，复制整个单词
>
> `y4j`: 复制下 4 行的内容
>
> `yfr`：复制到 r 为止的内容

单击 p：粘贴

单击 d：删除 delete

单击 u：撤销 undo



单击 i：光标之前输入

单击 a:  光标之后输入

单击 c：change

> `caw`:删除光标所在处的词并进入编辑模式
>
> `cc`删除这一行并进入编辑模式





按 v 进入可视模式