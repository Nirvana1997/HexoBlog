---
title: vim使用技巧学习(一)
date: 2018-06-27 14:33:17
category: vim
tags:
- vim
- Linux
---
## 1.前言
我一直很疑惑vim为什么还有很多人使用,交互十分困难,上手难度很大.记得当时第一次用vim时,修改文件和保存文件都搞了半天.但后来听说很多公司实际开发时都会用vim.于是去网上找了找vim的优势,大概可以总结成以下几点:

### - vim对于一个远程shell是最友好的编辑器 -
alt键一般是难以通过远程登陆传播的,而ctrl键部分组合会被终端吃掉.而vim的交互只需要通过字母的组合键和冒号的指令.只有vim这种为终端shell设计的编辑器.他的快捷键设计使得自己能够正常的在shell中执行自己的绝大部分操作而不出故障.**这是我认为vim最重要的一点优势.**

### - 拥有较好的一致性 -
如果用IDE的话,不同的IDE功能不同,换一个新的IDE要习惯一个新的IDE的配置.但是vim只要你掌握最基本的用法就可以.

### - 完全依靠键盘,不使用鼠标 -
不能使用鼠标可以说是vim交互的缺点,但当你习惯以后,这反而会成为一种优势.一切操作都依靠键盘以后很多时候是可以提高工作效率的.

<!-- more -->

## 2.学习vim
### 常用快捷键
之前使用vim基本没怎么用过快捷键,只是通过上下左右到对应地方进行修改,这样对vim是有点委屈他的"才能"的.所以首先先整理下vim的常用快捷键.

<table>
<thead>
<tr>
<th>按键</th>
<th>效果</th>
</tr>
</thead>
<tbody>
<tr>
<td>a,i,r,o,A,I,R,O</td>
<td>进入编辑模式</td>
</tr>
<tr>
<td>h,backspace</td>
<td>左移动</td>
</tr>
<tr>
<td>l,space</td>
<td>右移动</td>
</tr>
<tr>
<td>j</td>
<td>下移动</td>
</tr>
<tr>
<td>k</td>
<td>上移动</td>
</tr>
<tr>
<td>0,</td>
<td>移动到行首</td>
</tr>
<tr>
<td>$</td>
<td>移动到行末，<code>1$</code>表示当前行的行尾，<code>2$</code>表示当前行的下一行的行尾</td>
</tr>
<tr>
<td>b</td>
<td>按照单词向前移动 字首</td>
</tr>
<tr>
<td>e</td>
<td>按照单词向后移动 字尾</td>
</tr>
<tr>
<td>w</td>
<td>按照单词向后移至次一个字首</td>
</tr>
<tr>
<td>H</td>
<td>移动到屏幕最上 非空白字</td>
</tr>
<tr>
<td>M</td>
<td>移动到屏幕中央 非空白字</td>
</tr>
<tr>
<td>L</td>
<td>移动到屏幕最下 非空白字</td>
</tr>
<tr>
<td>G</td>
<td>移动到文档最后一行</td>
</tr>
<tr>
<td>gg</td>
<td>移动到文档第一行</td>
</tr>
<tr>
<td>v</td>
<td>进入光标模式，配合移动键选中多行</td>
</tr>
<tr>
<td>Ctrl+f</td>
<td>向下翻页</td>
</tr>
<tr>
<td>Ctrl+b</td>
<td>向上翻页</td>
</tr>
<tr>
<td>u</td>
<td>撤销上一次操作</td>
</tr>
<tr>
<td>..</td>
<td>回到上次编辑的位置</td>
</tr>
<tr>
<td>dw</td>
<td>删除这个单词后面的内容</td>
</tr>
<tr>
<td>dd</td>
<td>删除光标当前行</td>
</tr>
<tr>
<td>dG</td>
<td>删除光标后的全部文字</td>
</tr>
<tr>
<td>d$</td>
<td>删除本行光标后面的内容</td>
</tr>
<tr>
<td>d0</td>
<td>删除本行光标前面的内容</td>
</tr>
<tr>
<td>y</td>
<td>复制当前行，会复制换行符</td>
</tr>
<tr>
<td>yy</td>
<td>复制当前行的内容</td>
</tr>
<tr>
<td>yyp</td>
<td>复制当前行到下一行，此复制不会放到剪切板中</td>
</tr>
<tr>
<td>nyy</td>
<td>复制当前开始的n行</td>
</tr>
<tr>
<td>p,P,.</td>
<td>粘贴</td>
</tr>
<tr>
<td>ddp</td>
<td>当前行和下一行互换位置</td>
</tr>
<tr>
<td>J</td>
<td>合并行</td>
</tr>
<tr>
<td>Ctrl+r</td>
<td>重复上一次动作</td>
</tr>
<tr>
<td>Ctrl+z</td>
<td>暂停并退出</td>
</tr>
<tr>
<td>ZZ</td>
<td>保存离开</td>
</tr>
<tr>
<td>xp</td>
<td>交换字符后面的交换到前面</td>
</tr>
<tr>
<td>~</td>
<td>更换当前光标位置的大小写，并光标移动到本行右一个位置，直到无法移动</td>
</tr>
</tbody>
</table>


<table>
<thead>
<tr>
<th>按键</th>
<th>效果</th>
</tr>
</thead>
<tbody>
<tr>
<td>Ctrl+e</td>
<td>向下滚动</td>
</tr>
<tr>
<td>Ctrl+b</td>
<td>向上翻页</td>
</tr>
<tr>
<td>b</td>
<td>按照单词向前移动 字首</td>
</tr>
<tr>
<td>B</td>
<td>按照单词向前移动 字首 忽略一些标点符号</td>
</tr>
<tr>
<td>e</td>
<td>按照单词向后移动 字尾</td>
</tr>
<tr>
<td>E</td>
<td>按照单词向后移动 忽略一些标点符号</td>
</tr>
<tr>
<td>w</td>
<td>按照单词向后移至次一个字首</td>
</tr>
<tr>
<td>W</td>
<td>按照单词向后移至次一个字首 忽略一些标点符号</td>
</tr>
<tr>
<td>H</td>
<td>移动到屏幕最上 非空白字</td>
</tr>
<tr>
<td>M</td>
<td>移动到屏幕中央 非空白字</td>
</tr>
<tr>
<td>L</td>
<td>移动到屏幕最下 非空白字</td>
</tr>
<tr>
<td>G</td>
<td>移动到文档最后一行</td>
</tr>
<tr>
<td>gg</td>
<td>移动到文档第一行</td>
</tr>
<tr>
<td>(</td>
<td>光标到句尾</td>
</tr>
<tr>
<td>)</td>
<td>光标到局首</td>
</tr>
<tr>
<td>{</td>
<td>光标到段落开头</td>
</tr>
<tr>
<td>}</td>
<td>光标到段落结尾</td>
</tr>
<tr>
<td>nG</td>
<td>光标下移动到n行的首位</td>
</tr>
<tr>
<td>n$</td>
<td>光标移动到n行尾部</td>
</tr>
<tr>
<td>n+</td>
<td>光标下移动n行</td>
</tr>
<tr>
<td>n-</td>
<td>光标上移动n行</td>
</tr>
</tbody>
</table>

<table>
<thead>
<tr>
<th>指令</th>
<th>效果</th>
</tr>
</thead>
<tbody>
<tr>
<td>*</td>
<td>向下查找同样光标的字符</td>
</tr>
<tr>
<td>#</td>
<td>向上查找同样光标的字符</td>
</tr>
<tr>
<td>/code</td>
<td>查找 code 一样的内容，向后</td>
</tr>
<tr>
<td>?code</td>
<td>查找 code 一样的内容，向前</td>
</tr>
<tr>
<td>n</td>
<td>查找下一处</td>
</tr>
<tr>
<td>N</td>
<td>查找上一处</td>
</tr>
<tr>
<td>ma</td>
<td>在光标处做一个名叫a的标记 可用26个标记 (a~z)</td>
</tr>
<tr>
<td>`a</td>
<td>移动到一个标记a</td>
</tr>
<tr>
<td>d`a</td>
<td>删除当前位置到标记a之间的内容</td>
</tr>
<tr>
<td>:marks</td>
<td>查看所有标记</td>
</tr>
</tbody>
</table>

<table>
<thead>
<tr>
<th>指令</th>
<th>效果</th>
</tr>
</thead>
<tbody>
<tr>
<td>:q</td>
<td>一般退出</td>
</tr>
<tr>
<td>:q!</td>
<td>退出不保存</td>
</tr>
<tr>
<td>:wq</td>
<td>保存退出</td>
</tr>
<tr>
<td>:w filename</td>
<td>另存为 filename</td>
</tr>
<tr>
<td>:jumps</td>
<td>历史编辑文档记录</td>
</tr>
<tr>
<td>:set nu</td>
<td>设置行号显示</td>
</tr>
<tr>
<td>:set nonu</td>
<td>取消行号显示</td>
</tr>
<tr>
<td>:set</td>
<td>显示设置参数</td>
</tr>
<tr>
<td>:set autoindent</td>
<td>自动缩排，回车与第一个非空格符对齐</td>
</tr>
<tr>
<td>:syntax on/off</td>
<td>根据程序语法高亮显示</td>
</tr>
<tr>
<td>:set highlight</td>
<td>高亮设置查看</td>
</tr>
<tr>
<td>:set hlsearch</td>
<td>查找代码高亮显示</td>
</tr>
<tr>
<td>:nohlsearch</td>
<td>暂时关闭高亮显示</td>
</tr>
<tr>
<td>:set nohlsearch</td>
<td>永久关闭高亮显示</td>
</tr>
<tr>
<td>:set bg=dark</td>
<td>设置暗色调</td>
</tr>
<tr>
<td>:set bg=light</td>
<td>设置亮色调</td>
</tr>
</tbody>
</table>

<table>
<thead>
<tr>
<th>按键</th>
<th>效果</th>
</tr>
</thead>
<tbody>
<tr>
<td>Ctrl+f</td>
<td>向文件尾翻一屏幕</td>
</tr>
<tr>
<td>Ctrl+b</td>
<td>向文件首翻一屏幕</td>
</tr>
<tr>
<td>Ctrl+d</td>
<td>向文件尾翻半屏幕</td>
</tr>
<tr>
<td>Ctrl+u</td>
<td>向文件首翻半屏幕</td>
</tr>
</tbody>
</table>

<table>
<thead>
<tr>
<th>按键</th>
<th>效果</th>
</tr>
</thead>
<tbody>
<tr>
<td>i</td>
<td>在光标前</td>
</tr>
<tr>
<td>I</td>
<td>在当前行首</td>
</tr>
<tr>
<td>a</td>
<td>在光标后</td>
</tr>
<tr>
<td>A</td>
<td>在当前行尾部</td>
</tr>
<tr>
<td>o</td>
<td>在当前行下新开一行</td>
</tr>
<tr>
<td>O</td>
<td>在当前行上新开一行</td>
</tr>
<tr>
<td>r</td>
<td>替换当前字符</td>
</tr>
<tr>
<td>R</td>
<td>替换当前行及后面的字符，直到按esc为止</td>
</tr>
<tr>
<td>s</td>
<td>从当前行开始，以输入的文本替代指定数目的字符</td>
</tr>
<tr>
<td>S</td>
<td>删除指定数目的行，并以输入的文本替代</td>
</tr>
<tr>
<td>ncw,nCW</td>
<td>修改指定数目n的字符</td>
</tr>
<tr>
<td>nCC</td>
<td>修改指定数目n的行</td>
</tr>
</tbody>
</table>

<table>
<thead>
<tr>
<th>按键</th>
<th>效果</th>
</tr>
</thead>
<tbody>
<tr>
<td>ndw,nDW</td>
<td>删除光标开始及其后 n-1 个字符</td>
</tr>
<tr>
<td>dw</td>
<td>删除这个单词后面的内容</td>
</tr>
<tr>
<td>dd</td>
<td>删除光标当前行</td>
</tr>
<tr>
<td>dG</td>
<td>删除光标后的全部文字</td>
</tr>
<tr>
<td>d$</td>
<td>删除本行光标后面的内容</td>
</tr>
<tr>
<td>d0</td>
<td>删除本行光标前面的内容</td>
</tr>
<tr>
<td>ndd</td>
<td>删除当前行，以及其后的n-1行</td>
</tr>
<tr>
<td>x</td>
<td>删除一个字符，光标后</td>
</tr>
<tr>
<td>X</td>
<td>删除一个字符，光标前</td>
</tr>
<tr>
<td>Ctrl+u</td>
<td>删除输入模式下的输入的文本</td>
</tr>
</tbody>
</table>

<table>
<thead>
<tr>
<th>指令</th>
<th>效果</th>
</tr>
</thead>
<tbody>
<tr>
<td>:split</td>
<td>创建新窗口</td>
</tr>
<tr>
<td>Ctrl+w</td>
<td>切换窗口</td>
</tr>
<tr>
<td>Ctrl-w =</td>
<td>所有窗口一样高</td>
</tr>
<tr>
<td>Ctrl-w+方向键</td>
<td>多窗口视图切换</td>
</tr>
</tbody>
</table>

<table>
<thead>
<tr>
<th>指令</th>
<th>效果</th>
</tr>
</thead>
<tbody>
<tr>
<td>:args</td>
<td>列出当前编辑的文件名</td>
</tr>
<tr>
<td>:next</td>
<td>打开多文件，使用 n(Next) p(revious)</td>
<td>N(ext) 切换</td>
</tr>
<tr>
<td>:file</td>
<td>列出当前打开的所有文件</td>
</tr>
</tbody>
</table>

以上快捷键主要来源于[vim 常用快捷键及使用技巧](https://www.jianshu.com/p/dde77e3b299f)

***
下篇博客会介绍vim的一些使用技巧~