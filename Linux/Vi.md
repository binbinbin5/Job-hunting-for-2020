[toc]



## 命令行模式

任何时候，不管用户处于何种模式，只要按一下ESC键，即可使Vi进入命令模式；我们在shell环境(提示符为$)下输入启动Vi命令，进入编辑器时，也是处于该模式下。在该模式下，用户可以输入各种合法的Vi命令，用于管理自己的文档。此时从键盘上输入的任何字符都被当做编辑命令来解释，若输入的字符是合法的Vi命令，则Vi在接受用户命令之后完成相应的动作。但需注意的是，所输入的命令并不在屏幕上显示出来。若输入的字符不是Vi的合法命令，Vi会响铃报警。

1）. 插入模式

```
i: 插入光标前一个字符 
 I: 插入行首 
 a: 插入光标后一个字符 
 A: 插入行未 
 o: 向下新开一行,插入行首 
 O: 向上新开一行,插入行首
```


2）. 从插入模式切换为命令行模式

```
按「ESC」键。
```


3）. 移动
光标：

```
vi可以直接用键盘上的光标来上下左右移动，但正规的vi是用小写英文字母「h」、「j」、「k」、「l」，分别控制光标左、下、上、右移一格。
 按「ctrl」+「b」：屏幕往“后”移动一页。
 按「ctrl」+「f」：屏幕往“前”移动一页。
 按「ctrl」+「u」：屏幕往“后”移动半页。
 按「ctrl」+「d」：屏幕往“前”移动半页。
 按数字「0」：移到文章的开头。
 按「G」：移动到文章的最后。
 按「$」：移动到光标所在行的“行尾”。
 按「^」：移动到光标所在行的“行首”
 按「w」：光标跳到下个字的开头
 按「e」：光标跳到下个字的字尾
 按「b」：光标回到上个字的开头
 按「#l」：光标移到该行的第#个位置，如：5l,56l。
```


文本:

```
{: 按段移动,上移 
 }: 按段移动,下移
 >>: 文本行右移 
  <<: 文本行左移
```


4）. 删除文字

```
x：每按一次，删除光标所在位置的“后面”一个字符。
 #x：例如，「6x」表示删除光标所在位置的“后面”6个字符。
 X：大写的X，每按一次，删除光标所在位置的“前面”一个字符。
 #X：例如，「20X」表示删除光标所在位置的“前面”20个字符。
 #dd：从光标所在行开始删除#行
 dd: 删除光标所在行,n dd 删除指定的行数 D: 删除光标后本行所有内容,包含光标所在字符 
 d0: 删除光标前本行所有内容,不包含光标所在字符
 dw: 删除光标开始位置的字,包含光标所在字符
```


5）. 复制

```
「yw」：将光标所在之处到字尾的字符复制到缓冲区中。
「#yw」：复制#个字到缓冲区
「yy」：复制光标所在行到缓冲区。
「#yy」：例如，「6yy」表示拷贝从光标所在的该行“往下数”6行文字。
「p」：将缓冲区内的字符贴到光标所在位置。注意：所有与“y”有关的复制命令都必须与“p”配合才能完成复制与粘贴功能。
```


6）. 替换

```
「r」：替换光标所在处的字符。
「R」：替换光标所到之处的字符，直到按下「ESC」键为止。
```

7）. 恢复/撤消/还原上一次操作

```
「u」：如果误执行一个命令，可以马上按下「u」，撤消上一个操作。按多次“u”可以执行多次撤消。
 Ctr-r: 反撤销
```



8）. 更改

```
「cw」：更改光标所在处的字到字尾处
「c#w」：例如，「c3w」表示更改3个字
```


9）. 跳至指定的行

```
ctrl+g列: 出光标所在行的行号。
 #G: 例如，「15G」，表示移动光标至文章的第15行行首。
 gg: 光标移动文件开头 
 G: 光标移动到文件末尾
```


10) .其他

```
.: 重复上一次操作的命令
 v: 按字符移动,选中文本 
 V: 按行移动,选中文本可视模式可以配合 d, y, >>, << 实现对文本块的删除,复制,左右移动
替换:
 r: 替换当前字符 
 R: 替换当前行光标后的字符
查找：
 /: str查找
 n: 下一个
 N：上一个
```


## 文本输入模式

在命令模式下输入插入命令i、附加命令a 、打开命令o、修改命令c、取代命令r或替换命令s都可以进入文本输入模式。在该模式下，用户输入的任何字符都被Vi当做文件内容保存起来，并将其显示在屏幕上。在文本输入过程中，若想回到命令模式下，按键ESC即可。

末行模式
末行模式也称ex转义模式。在命令模式下，用户按“:”键即可进入末行模式下，此时Vi会在显示窗口的最后一行(通常也是屏幕的最后一行)显示一个“:”作为末行模式的提示符，等待用户输入命令。多数文件管理命令都是在此模式下执行的(如把编辑缓冲区的内容写到文件中等)。末行命令执行完后，Vi自动回到命令模式。例如：:sp newfile
则分出一个窗口编辑newfile文件。如果要从命令模式转换到编辑模式，可以键入命令a或者i；如果需要从文本模式返回，则按Esc键即可。在命令模式下输入“:”即可切换到末行模式，然后输入命令。


```
: w filename （输入 「w filename」将文章以指定的文件名filename保存）
: wq (输入「wq」，存盘并退出vi)
: q! (输入q!， 不存盘强制退出vi)
```

1 末行模式下，将光标所在行的abc替换成123

```
:%s/abc/123/g
```


2 末行模式下，将第一行至第10行之间的abc替换成123

```
:1, 10s/abc/123/g
```


3 vim里执行 shell 下命令:

```
末行模式里输入!,后面跟命
```
令

4 列出行号

```
「set nu」：输入「set nu」后，会在文件中的每一行前面列出行号。
```


5 跳到文件中的某一行

```
「#」：「#」号表示一个数字，在冒号后输入一个数字，再按回车键就会跳到该行了，如输入数字15，再回车，就会跳到文章的第15行。
```


6 查找字符

```
「/关键字」：先按「/」键，再输入您想寻找的字符，如果第一次找的关键字不是您想要的，可以一直按「n」会往后寻找到您要的关键字为止。
「?关键字」：先按「?」键，再输入您想寻找的字符，如果第一次找的关键字不是您想要的，可以一直按「n」会往前寻找到您要的关键字为止。
```


7 保存文件

```
「w」：在冒号输入字母「w」就可以将文件保存起来。
```


8 离开vi

```
「q」：按「q」就是退出，如果无法离开vi，可以在「q」后跟一个「!」强制离开vi。
「qw」：一般建议离开时，搭配「w」一起使用，这样在退出的时候还可以保存文件。
```


