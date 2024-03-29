# 执行模块
![执——行！](item:tis3d:execution_module)

执行模块是用来给TIS-3D计算机编程的主要手段。当安 装于[外壳](../block/casing.md)时，对着它使用里面写着代码的（MC原版）书就可以导入程序。
为使开发体验更加便捷，可以考虑使用[编码书](code_book.md)，它是 每位专业人士真正的、经过实践检验的工具。

## 架构
执行模块让用户能以极大的灵活性控制TIS-3D计算机执 行的操作。执行模块采用一套优化指令集的底层汇编语言进行编程。
在模块活动时，执行模块从程序中的第一条指令开始，向下逐条执行。每条指令（除非是跳转指令）执行完成后都会让程序计数器前进到下一条指令。如果程序计数器离开指令的有效范围（最后一条执行完成），程序会自动从第一条指令继续执行，除非明确地停止程序。*直接来说，程序会循环执行*。

指令可以对不同种类的目标执行操作。目标可以是能读取值的有效来源，或者是能写入值的有效目的地。有效目标包括执行模块的四个端口，执行模块的寄存器和少量的虚拟寄存器与端口。

TIS-3D计算机以及扩展的执行模块支持16位的值，这16 位看作单个有符号数值，范围为-32768至 32767。*由于技术上的限制，一些模块可能会选择把数值以无符号十六进制的形式显示，包括执行模块*。

## 操作目标
`ACC`
为寄存器。它是一个执行模块的主要寄存器。算术操作的结果均会存储到此寄存器。

`BAK`
为不可寻址的寄存器。它是一个特殊的寄存器，可以用来存储来自`ACC`的值。它不能被直接寻址（访问），只 能用`SAV`和`SWP`指令访问。

`NIL`
为虚拟寄存器。它是一个伪目标，用来写入以丢弃数值，或者读取以产生零值。

`LEFT`, `RIGHT`, `UP`, `DOWN`
为端口。它们代表了执行模块的上下左右四端口。一般来说对它们进行的操作慢于对内部寄存器的操作。

`ANY`
为虚拟端口。它是一个伪目标，对它进行的操作（读写）会同时在上下左右四个端口执行，不过实际上只会在第一个完成操作的端口上执行。
例如，`MOV 10 ANY`将会向所有端口写入数值10。但从某个端口读取后，其他端口便不能再读取该值。在所有端口上的写入操作会在其中一个完成后迅速取消。
*原译者注：经本人实验，从ANY读入时，如果同时有多个接口输入，优先级是：左>右>上>下；向ANY输出时，如果同时有多个接口可以输出，优先级是上>左>右>下*

`LAST`
为虚拟端口。它是一个伪目标，会存储上一次对`ANY`伪 目标操作时*实际*执行了操作的端口。

## 编程语言规范
除了指令列表之外，提供给一个执行模块的汇编代码可以包含元数据。
注释是对代码的文本标注，在程序执行时会被完全忽略。标签在代码中标注了可以被跳转指令定位的位置。
注释、标签和空行对编译后的程序没有影响。这有关于`JRO`指令的使用。

### 注释
开头为`#`（#）井号的字符串是注释。它们可以作为单独的一行出现，也可与指令和标签位于同一行。
例如：
`#`单行注释
`LOOP: #`循环的开始
`MOV 0, ACC #`重置ACC

### 定义（define）
有一种特殊的注释可以创建或移除“定义”，它是数值的别名。这样可以把配置值移至同一位置，方便处理和重复使用，从而能轻松编写可重复使用的代码。
*译注：类似C语言的#define*
一个“定义”可以用`#define A B`来创建，用`#undef A`来移除。
例如：
`#`让A=RIGHT
`#define A RIGHT`
`#`将值从右端口移动到ACC
`MOV A ACC`
`#undef A`
`MOV A ACC #`错误：A已经被取消定义

### 标签
结尾是`:`（:）冒号的字符串为标签。标签指向它后面跟 着的指令。当它被用作跳转指令的目标时，程序就会直接跳转至其后的指令。这也意味着在程序执行方面，一个标签被放在单独的一行还是它所指向的指令那一行开头没有区别。
例如：
`START: MOV 8, ACC`  
`LOOP:`  
`SUB 1`  
`JGZ LOOP`  
`JMP START`  
`#`这一行永远不会执行到（上一行跳转回开头）

### 指令
`NOP`
它是一条伪指令，对执行模块的端口和寄存器没有任何影响，也即什么都不做。
`NOP`会被自动编译为`ADD NIL`。

#### 数据传输
`MOV <SRC> <DST>`
把数据从`SRC`传到`DST`。合法的目标请参见上文。要注意，对寄存器或内部状态量的操作一般来说比对端口的操 作要快，也因供应方而异。
例如：
`MOV 8, ACC` 将数值8写入到`ACC`寄存器 
`MOV LEFT, RIGHT`从左端口读取一个值，然后把它写 入到右端口。
`MOV DOWN, NIL`从上端口读取一个值，然后把它写入`NIL`，效果等于丢弃它。

`SWP`
交换`ACC`和`BAK`寄存器中的当前值。

`SAV`
把`ACC`寄存器中当前的值复制到`BAK`寄存器。

#### 算术操作
`ADD <SRC>`
从指定的目标`SRC`读取一个数值，然后把它与`ACC`中的 当前值加和，最后把结果写入回`ACC`中。
注意，数据正负溢出不会发生。数值增减会在合法数据范围边界处停止。
例如：
`ADD 1`将`ACC`寄存器的当前值加1。
`ADD LEFT`从左端口读取一个数值，然后把它加到`ACC`中。

`SUB <SRC>`
从指定的目标`SRC`中读取一个数值，然后把它从`ACC`的 当前值中减去，最后把结果写入回`ACC`中。
注意，数据正负溢出不会发生。数值增减会在合法数据范围边界处停止。
例如：
`SUB 1`将`ACC`的值减去1。
`SUB LEFT`从左端口读取一个数值，然后把它从`ACC`中 减去。

`MUL <SRC>`
从指定的目标`SRC`中读取一个数值，然后把它与`ACC`的 当前值相乘，最后把结果写入回`ACC`中。
注意，数据正溢出不会发生。数值增减会在合法数据范围边界处停止。
例如：
`MUL 2`将`ACC`的值乘以2。
`MUL LEFT`从左端口读取一个数值，然后把`ACC`的值乘 以这个数。

`DIV <SRC>`
从指定的目标`SRC`中读取一个数值，然后把`ACC`的当前 值除以该数，最后把结果写入回`ACC`中。
注意，除以0会导致系统进入错误状态，并且自动重启。
例如：
`DIV 2`将`ACC`的值除以2。
`DIV LEFT`从左端口读取一个数值，然后把`ACC`的值除 以这个数。

`NEG`
将`ACC`中当前的值符号取反，结果存回`ACC`。

### 位操作
`AND <SRC>`
从指定的目标`SRC`中读取一个数值，然后和`ACC`的当前 值进行按位与运算。
例如：
`AND 0x00FF`将`ACC`中存储的值高8位变为0，低8位保 持不变。
`AND LEFT`从左端口读取一个值，然后把它用作`ACC`当 前值的位掩码。

`OR <SRC>`
从指定的目标`SRC`中读取一个数值，然后和`ACC`的当前 值进行按位或运算。
例如：
`OR 0x0001`将`ACC`中存储的值最低位设置为1，然后把 结果写入回`ACC`。
`OR LEFT`从左端口读取一个值，然后把它读取到的值中为1的位对应的`ACC`当前值的位设定为1，然后把结果写 入回`ACC`。

`XOR <SRC>`
从指定的目标`SRC`中读取一个数值，然后和`ACC`的当前 值进行按位异或运算。
例如：
`XOR 1`如果`ACC`中存储的值最低位为0，则设置为1，如果为1则设置为0。
`XOR LEFT`从左端口中读取一个数值，然后和`ACC`的 当前值进行按位异或运算。

`SHL <SRC>`
从指定的目标`SRC`中读取一个数值，然后以它为次数把`ACC`的当前值进行位左移运算，最后把结果写入回`ACC`中。
例如：
`SHL 4`将`ACC`中存储的值左移四位，比如0x0F会变成 0xF0。
`SHL LEFT`从左端口中读取一个数值，然后将`ACC`的当 前值左移“这个数值”位。

`SHR <SRC>`
从指定的目标`SRC`中读取一个数值，然后以它为次数把`ACC`的当前值进行位右移运算，最后把结果写入回`ACC`中。
例如：
`SHR 4`将`ACC`中存储的值右移四位，比如0xF0会变成0x0F。
`SHR LEFT`从左端口中读取一个数值，然后将`ACC`的当 前值右移“这个数值”位。

`NOT`
对`ACC`的当前值进行按位非运算，最后把结果写入回`ACC`中。
例如：
对存储数值`0xFF00`的`ACC`进行`NOT`，存储的值变为`0x00FF`。

### 操纵LAST
`RRLAST`
如果`LAST`的当前值不为`NIL`，将其代表的端口向右（顺时针）旋转一次。否则什么都不做。

`RLLAST`
如果`LAST`的当前值不为`NIL`，将其代表的端口向左（逆时针）旋转一次。否则什么都不做。

### 流程控制
`JMP <LABEL>`
无条件跳转到标签`LABEL`指向的指令继续执行。

`JEZ <LABEL>`
如果`ACC`的当前值**为0**，则跳转到跳转到标签`LABEL`指向的指令继续执行。否则继续执行程序的下一条指令。

`JNZ <LABEL>`
如果`ACC`的当前值**不为0**，则跳转到跳转到标签`LABEL`指向的指令继续执行。否则继续执行程序的下一条指令。

`JGZ <LABEL>`
如果`ACC`的当前值**大于0**，则跳转到跳转到标签`LABEL`指向的指令继续执行。否则继续执行程序的下一条指令。

`JLZ <LABEL>`
如果`ACC`的当前值**小于0**，则跳转到跳转到标签`LABEL`指向的指令继续执行。否则继续执行程序的下一条指令。

`JRO <SRC>`
无条件跳转到一条语句。此指令修改程序计数器的值，将`SRC`的值加到其中，程序会从新的地址继续执行。也就是跳转到`SRC`条语句之后。
`JRO 0`的效果相当于立即终止执行模块的运行。
