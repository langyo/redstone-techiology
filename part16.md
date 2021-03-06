# Verilog HDL 语言快速入门

用手画 CPU 早已是过去的事情了。为了制作超大规模的集成电路，人们开发了一系列的硬件描述语言，以计算机辅助人们生成需要的 HDL 电路。Verilog HDL 便是其中之一，它早已流行于各个芯片大厂中，成为信息技术在硬件方面的得力助手。如果想了解现代计算机的运作原理，除了需要大量的理论知识外，还需要的便是能看懂以这种语言描绘的电路。

阅读本文，您需要一定的编程基础。您可以通过学习本索引的姊妹篇[这是 C++ 的世界！](https://langyo-v2.gitbook.io/cppworld/)来快速入门，然后再回到本篇文章继续学习，会轻松很多。

本文实际上是一则笔记，不能完整地教授您有关 Verilog HDL 的全部语法知识，仅仅能起到快速入门、能基本看懂他人以 Verilog HDL 写成的芯片电路逻辑的作用。

### 目录

* [基本概念](part16#ji-ben-gai-nian)
* [模块](part16#mo-kuai)
* [模块实例化](part16#mo-kuai-shi-li-hua)
* [常数值与常量](part16#chang-shu-zhi-yu-chang-liang)
* [变量](part16#bian-liang)
* [标识符](part16#biao-shi-fu)
* [注释](part16#zhu-shi)
* [赋值](part16#fu-zhi)
* [默认网络类型](part16#mo-ren-wang-luo-lei-xing)
* [运算符](part16#yun-suan-fu)
* [条件判断](part16#tiao-jian-pan-duan)
* [always过程块](part16#always-guo-cheng-kuai)
* [分支选择](part16#fen-zhi-xuan-ze)
* [预处理](part16#yu-chu-li)
* [正逻辑与负逻辑](part16#zheng-luo-ji-yu-fu-luo-ji)
* [电路仿真](part16#dian-lu-fang-zhen)
* [模拟时钟](part16#mo-ni-shi-zhong)
* [系统任务](part16#xi-tong-ren-wu)
* [IcarusVerilog仿真程序的使用](part16#IcrausVerilog-fang-zhen-cheng-xu-de-shi-yong)

## 基本概念

Verilog HDL 是一种 HDL 语言（Hardware Description Language，硬件描述语言）。使用此类语言可以进行抽象度较高的 RTL（Register Transfer Level，寄存器传输级）电路的设计，是在当代设计 CPU 这类超大规模逻辑电路最好的选择。RTL 是根据寄存器间的信号流动和电路逻辑来记述电路动作的一种设计模型。

逻辑综合，是将 RTL 级别记述的抽象电路转换到门电路级别的电路网表的过程。逻辑综合时，针对 ASIC（Application Specific Integrated Circult）、FPGA（Field Programmable Gate Array）等不同的电路实现技术，需要使用这些技术厂商提供的相应的目标元件库。

## 模块

定义一个模块的格式：
```verilog
module <模块名>(
  <输入以及输出的信号定义>,
  ...
);
  <具体的电路描述>
endmodule
```

其中，输入以及输出信号的定义是若干行开头为 input、output 或 inout 的声明，和计算机程序设计语言中的形参颇有相似之处。

具体示例：
```verilog
module adder(
  input wire[31:0] in_0,  // 输入 0 号线
  input wire[31:0] in_1,  // 输入 1 号线
  output wire[31:0] out   // 输出线
);  // 别忘了末尾的分号
  assign out = in_0 + in_1;  // 让输出的信号为两个输入信号的和，加法器的电路会由电路仿真器自动生成
endmodule
```

## 模块实例化

模块实例化，实质上就是建立一个新的模块本体，并指定这个新模块与哪些线路连接。

实例化一个模块的格式：
```verilog
<模块类型名> <模块实例名>(
  .<模块定义的接口名> (<要连入这个接口的线路/寄存器名>),
  ...
);
```

具体示例：
```verilog
// 沿用之前写好的 adder，我们可以将其连接到一些线上
adder adder1(
  .in_0 (register_in_1),
  .in_1 (register_in_2),
  .out (register_out_6)
);
```

## 常数值与常量

这个语言没有定义诸如 true、false、nil、null 这类关键字，而是直接用数字表达逻辑值，简单又直接。数字 0 和 1 分别代表低电平与高电平，也分别代表逻辑假和逻辑真。一定要记住这两个数字在硬件设计语言中的逻辑意义，因为它们与我们日常使用的绝大部分计算机编程语言的含义是正好相反的！

为了表达电路中的特殊状态信号，此语言还定义了两个字母 x 和 z 作为特殊关键字，分别有不定值与高阻值（电气绝缘状态）的意义。

对于具有位宽的信号线与寄存器，可以为其赋予不同进制的常数，可以使用二进制、八进制、十进制与十六进制。一个常数的格式：

```<位宽> '<底数(进制)> <数值>```

这三个部分之间不留空格，互相紧挨着写。位宽必须为一个十进制数，而底数则为一个字母：h 为十六进制，d 为十进制，o 为八进制，b 则为二进制。

具体示例：

> 6'b111100
>
> 6'o74
>
> 6'd60
>
> 6'h3c

另外，这个语言也能定义常量。强烈建议不要往程序中使用魔数（Magic Number，即直接写意义不明的数字），因为那样会让电路描述变得晦涩难懂、难以维护。常量本质上也是常数值数字，但它以一个有意义的名字代替了这个数字，从而提升电路描述的可读性，并且也易于维护。

它的定义格式如下：

```verilog
`define XXX XXX
```

前一个为这个常量的名字，后一个为你要定义的值，可以为任意的常数值。

定义完成后，就可以使用了：

```verilog
`XXX
```

直接将这种格式的文本写进表达式就可以了。

例如：

```verilog
`define A 1
reg n = `A;
```

## 变量

变量的声明格式：

```<数据类型> [符号] [位宽] <变量名> [元素数];```

------

数据类型分为两大类：寄存器型与网络型。当出现了一个寄存器型的变量时，电路生成器会往电路里塞个寄存器单元。而网络型，其实就是线路，这种类型的变量只能起到连接寄存器及 I/O 单元的用途，自身无法储存任何信号。

寄存器型有以下几种：

| 类型 | 默认位宽 | 默认是否有符号 | 类型说明 |
| :--  | :--   | :-- | :-- |
| reg  | 1  | 无 | 比特数据 |
| integer | 32 | 有 | 整数 |
| real | 64 | 有 | 实数 |

网络型有以下几种：

| 类型 | 类型说明 |
| :--  | :--   |
| wire/tri | 线连接 |
| wor/trior | 线或连接 |
| wand/triand | 线与连接 |
| tri1/tri0 | 有上拉或下拉的连接 |
| supply0/supply1 | 接地或接电源的连接 |

> 网络型变量的位宽均默认为 1，并且均无符号。

------

符号分为 signed 与 unsigned，分别是有符号和无符号。符号可以省略。

要将一个变量在有符号与无符号间转换，可以使用系统任务（system task）$signed() 与 $unsigned()。

------

位宽类似于区间，格式如下：

```[a:b]``` 或 ```[a]```

其中，a > b 且 a, b ∈ N（a 和 b 都是正整数或 0）。

原则上，b 应当从 0 数起。例如，我要表达 32 位位宽的线路，在声明变量时应当这么写：

```verilog
[31:0]
```

而要将其分为 4 份 8 位的位宽，则可以这么写：

```verilog
[31:24] [23:16] [15:8] [7:0]
```

请注意，位数是由 0 而不是由 1 算起的！

------

元素数的表达格式与位宽一直，但它用于指示此变量是一个阵列（可理解为数组），并由此确定了这个阵列的元素个数。

> P.S. 为什么元素数也用类似区间的格式呢？似乎是为了选定此变量阵列中特定范围的元素？

------

如果要声明多个类型一致的变量，可以将所有要声明的变量名同写在一条声明的变量名的位置，相互间以逗号隔开。

例如：

```verilog
reg a, b, c;
```

## 标识符

标识符是为变量、模块等取名时使用的名字。它是一连串的字符，而不是单个字符。

标识符仅可由大小写字母(a-z, A-Z)、数字(0-9)、下划线(_)与美元符号($)组成。

## 注释

单行注释：

```verilog
// ...
```

多行（块状）注释：

```verilog
/*
  ...
 */
```

## 赋值

赋值依据成分的不同，可分为声明时赋值与过程赋值。例如：

```verilog
wire [15:0] dbyte;
wire [7:0] byte0 = dbyte[15:8];
wire [7:0] byte1 = dbyte[7:0];  // 声明时赋值

assign byte0 = dbyte[7:0];
assign byte1 = dbyte[15:8];     // 过程赋值
```

过程赋值（Procedural Assignment）又可分为阻塞式赋值与非阻塞式赋值。阻塞式赋值严格保证赋值的顺序，必须在前一个的赋值任务完成后，接下来的赋值才会继续；非阻塞式赋值则不会强行规定同在一片上下文中赋值操作的顺序，可以同时并行执行。例如：

```verilog
a = a + 1;
b = a + 1;  // 阻塞式赋值，执行后 a = 1，b = 2。
a2 <= a2 + 1;
b2 <= a2 + 1; // 非阻塞式赋值，本行和上一行是同时执行的，所以执行时使用的 a2 的值均为最初的 0。状态为 a = 1, b = 1。
```

## 默认网络类型

有时候，设计者需要去写大规模的网表，在这种情况下去声明大量的网表变量是十分费事的。为了简化代码，你可以在源代码中加入编译器指令 ````default_nettype``` 来指定默认声明的变量类型。例如：

```verilog
`default_nettype wire  // 设置默认网络类型为 wire
```

但在一般情况下，这种机制反而会带来麻烦，因为编译器将检测不到拼写错误的变量名、自动将这种误用的变量名当成一个新的变量。所以，除非必要，否则少用此功能为妙。

要想关闭此特性，可以设置默认网络类型为 ```none```：

```verilog
`default_nettype none  // 不启用默认网络类型
```

## 运算符

**算术运算符**

| 符号 | 说明 |
| :-: | :-: |
| + | 加 |
| - | 减 |
| * | 乘 |
| / | 除 |
| % | 取模（求两个数之间相除后的余数） |

**位运算符**

| 符号 | 说明 |
| :-: | :-: |
| ~ | 位取反(NOT) |
| & | 位与(AND) |
| \| | 位或(OR) |
| ^ | 位异或(XOR) |
| ~^ | 位同或(NXOR) |

**缩减运算符**

| 符号 | 说明 |
| :-: | :-: |
| & | 与(AND) |
| ~& | 与非(NAND) |
| \| | 或(OR) |
| ~\| | 或非(NOR) |
| ^ | 异或(XOR) |
| ~^ | 同或(NXOR) |

**移位运算符**

| 符号 | 说明 |
| :-: | :-: |
| << | 左移 |
| >> | 右移 |

**等式运算符**

| 符号 | 说明 |
| :-: | :-: |
| == | 等于 |
| != | 不等于 |
| === | 全等于（x/z 这两个常数值也会参与比较） |
| !== | 全不等于（x/z 这两个常数值也会参与比较） |

**关系运算符**

| 符号 | 说明 |
| :-: | :-: |
| > | 大于 |
| < | 小于 |
| >= | 大于等于/不小于 |
| <= | 小于等于/不大于 |

**逻辑运算符**

| 符号 | 说明 |
| :-: | :-: |
| ! | 逻辑取反 |
| \|\| | 逻辑或 |
| && | 逻辑与 |

**三目运算符**

| 符号 | 说明 |
| :-: | :-: |
| a ? x : y | 对于条件表达式 a ? x : y，先计算条件 a，然后进行判断。如果 a 的值为真，则这个表达式的运算结果为 x 的值；否则计算 y 的值，这个表达式的运算结果为 y 的值。 |

**拼接运算符**

| 符号 | 说明 |
| :-: | :-: |
| { 信道 1 的某几位, 信道 2 的某几位, ..., 信道 n 的某几位} | 用于拼接两条表达式的运算结果，拼接方式为位拼接。 |

优先级由高到低排序：
> ~ & ~& | ~| ^ ~^ !
>
> * / %
> 
> + -
>
> << >>
>
> < <= > >=
>
> == != === !==
>
> & ^ ~^
> 
> |
>
> &&
>
> ||
> 
> ..?..:..

缩减运算符是用于对信道的所有位进行位运算，最终只输出一位结果的运算符，这与输入和输出有着一样长度的位运算符有很大不同！

圆括号不能视作真正的运算符，因为它对于电路设计有着特殊的作用：它能够改变表达式的执行顺序和运算符的优先级。

拼接运算符是带有位宽概念的语言所特有的运算符，用于拼接几个零散的位宽数据到一块。例如：

```verilog
wire[7:0] byte0, byte1, byte2, byte3;
wire[31:0] word = {byte0, byte1, byte2, byte3};
```
## 条件判断

格式：

```verilog
if(<表达式>) <语句序列>
if(<表达式>) <语句序列> else <语句序列>
```

具体示例：

```verilog
if(a > b) begin    // 注意这里的语句块用的不是花括号
    ...
end else if (a == b) begin    // 可以使用 else if 结构
    ...
end else begin
    ...
end
```

使用逻辑运算符来判断是最直观、易懂的方法，但在实际设计中也不可避免地直接以数值作为条件。这里的数字与逻辑值对应的关系与软件编程语言不通，0 为逻辑假，1 为逻辑真，请多加注意。

## 循环

格式：

```verilog
for(<初始化语句>; <判别式>; <递推语句>) ...
while(<表达式>) ...
```

具体示例：

```verilog
for(i = 0; i < 10; i = i + 1) begin  // 不能使用 i++ 或 ++i
    ...
end
while (i < 10) begin
    ...
end
```

实际上这与许多流行的编程语言对循环的定义没多大区别。另外注意，没有 ```do ... while``` 循环。

循环语句也可以在 ```initial``` 或 ```always``` 语句块中使用。

## always 过程块

为了让电路能接入外部信号、随外部的信号变化作出反应，verilog 引入了 always 过程块。

格式：

```verilog
always @(<事件表达式>) <语句序列>
always #<常数表达式> <语句序列>
```

最基本也最简单的事件表达式只有一个通配符“*”，意思是任何输入信号变化时都会执行过程块中的代码。

例如：

```verilog
module adder(
    input wire[31:0] in_0,
    input wire[31:0] in_1,
    output reg[31:0] out
);
    always @(*) begin
        out = in_0 + in_1;    // 阻塞式加法
    end
endmodule
```

事件多种多样。时钟信号上升时的动作记为 posedge，下降时记为 negedge。在事件名称的后面，应当紧跟要监测的信号名。事件表达式还可以使用 or 列举多个条件。为存储元件设置异步复位（reset）信号时，除了时钟信号外还要写上复位信号的边沿和信号名。

例如，下面演示一个异步存储器，它根据时钟信号来选择是否同步输入信号：

```verilog
module ff(
    input wire clk,    // 时钟
    input wire reset_, // 复位（负逻辑）
    input wire d_in,   // 输入的数据
    output reg d_out   // 输出的数据
);
    always @(posedge clk or negedge reset_) begin
        if(reset_ == 1'b0) begin    // 异步复位
            d_out <= 1'b0;
        end else begin              // 凭时钟同步输入数据
            d_out <= d_in;
        end
    end
endmodule
```

## 分支选择

对于大规模判断线路信号的情况，传统的条件判断语句一个一个写十分费时费力。为了应付这种情况，我们可以使用可批量进行条件判断的 case 语句块。

格式：

```verilog
case (<表达式>)
    <表达式>: <语句序列>
    <表达式>, <表达式>, ..., <表达式>: <语句序列>
    default: <语句序列>
endcase
```

具体示例：

```verilog
always @(*) begin
    case (in)
        2'b00: out = 4'b0001;
        2'b01: out = 4'b0010;
        2'b10: out = 4'b0100;
        default: begin out = 4'bxxxx; end
    endcase
end
// 明眼人应该能看得出来，其实上面的这个玩意就是个译码器~
```

使用 always 语句描述组合电路时，如果信号未被复制，很可能会使编译器引入不必要的锁存器，使得原本应当为组合电路的模块搞成了时序电路。（？）

出现上述这种情况的原因是因为不完整的 case 语句或没有 else 的 if 语句。为了规避这个问题，我们在设计时一定要补全条件判断，default 与 else 一定得补全，或将变量在条件判断前先赋予默认值。

当然，有些时候确实不用考虑默认情况，或在考虑到的情况之外随便输出什么都可以。这种情况被称为 ```Don't care```（忽略），输出为逻辑综合时优化的数值。在 Verilog HDL 中，声明 ```Don't care``` 的方法是为默认情况下的输出赋予不定值，以让编译器可以毫无顾忌地对电路大胆优化。

例如：
```verilog
default: out = 4'bxxxx;
```

## 预处理

预处理是在代码编译前对其进行预先的一些处理，例如设置宏与条件编译等。这种机制和 C/C++ 语言原理上相差不大。预处理使用编译指示符对编译器进行控制。

编译指示符以后引号（`）开头。与 C 语言的预编译不同的是，不仅开头需要加后引号，后续的所有编译指示符全都需要加后引号。（？）

格式：
```verilog
// 包含其它的 Verilog 源代码文件
`include "<相对路径>"
// 声明宏
`define <宏名> <值>
// 条件编译，当满足什么条件时编译哪些语句块，否则编译哪些语句块
`ifdef <宏名> ... `else ... `endif
// 条件编译，当不满足什么条件时编译哪些语句块，否则编译哪些语句块
`ifndef <宏名> ... `else ... `endif
// 设置仿真执行的时间单位
`timescale <单位时间>/<精度>
```

不同于 C 语言的预处理，这里的预处理是作为语法的一部分存在的，因此你可以在写了预处理指令的行上正常写注释。（？）

例如：

```verilog
`include "abc.h"	// 这里的注释可以和刚刚的 include 写在同一行
```

## 正逻辑与负逻辑

控制信号的有效、无效于信号高低电平相对应时，高电平有效、低电平无效的分配方式称为正逻辑。反之，高电平无效、低电平有效的分配称为负逻辑。

不论信号电平的高低，控制信号转为有效状态的动作称为断言（assert），转为无效状态的动作称为无效（negate）。并且，信号有效时称为使能（enable），信号无效时称为非使能（disable）。

## 电路仿真

使用 Verilog HDL 不仅可以设计电路，还可以对所设计的电路进行仿真。通过仿真可以实现逻辑验证，从而测试设计好的电路是否可以正常工作。记述仿真程序的文件称为 Testbench。类似的，Testbench 在 Verilog HDL 中本身便被定义为一种模块，与普通模块的区别在于不定义输出，并且使用 initial 关键字。例如：

```verilog
`timescale 1ns/1ps        // 设置 timescale
                          // （单位时间：1ns，时间精度：1ps）
module test_bench;        // 定义一个 Testbench 类型的模块 test_bench，无需任何输入输出
    reg adder01_in_0;     // 接入到被测模块的输入输出
    reg adder01_in_1;
    wire adder01_out;
    adder adder01(        // 实例化一个被测模块
        .in_0(adder01_in_0),
        .in_1(adder01_in_1),
        .out(adder01_out)
    );
    initial begin ... end // 测试代码
endmodule
```

在正式测试前，必须先设置仿真环境的时钟单位周期与时间度量精度。在``` `timescale```中设定的时间由数字和单位组成（单位可以选择 ```fs```, ```ps```, ```ns```, ```us```, ```ms``` 和 ```s```）。单位时间用来指定仿真的一个单位时间相当于多少秒，时间精度表示的是仿真处理时的单位时间精度。您没有必要取过小的时间精度，因为这会加剧您的电路仿真时间。单位时间必须不小于时间精度。

initial 语句是在仿真时执行且只执行一次的语句，与延迟描述配合使用，即可生成测试用例。例如：

```verilog
initial begin
    #0 begin    // 在时刻 0 时执行
        ...
    end
    #10 begin   // 在时刻 10 时执行
        ...
    end
    #10 begin   // 在时刻 20 时执行，注意左侧数字仍然为 10！
        ...     // 井号右侧的数字是相对于上一次指定的时刻所延迟的时间！
    end
end
```

“#”字符用于表述延迟语句。延迟语句由一个“#”和一个紧随其后的正整数组成，这个数字乘以由``` `timescale```设置的单位时间得到的时间长度，即为这次延迟的时长。延迟语句仅在仿真时生效，用于在特定的延迟时长后才向特定电路施加信号。例如：

```verilog
initial begin // initial 语句只在仿真开始时的时刻 0 执行一遍
    #0 c = 1'b1;
    #10 c = 1'b0;
    #10 c = 1'b1;
    #10 c = 1'b0;
end
```

但是，在仿真程序中信号的变化是无延迟的，也就是你无法解释在某个确切的时刻那一瞬间，被改动的线路的信号是高还是低，只能确定在这一瞬间之前或之后是高还是低。

而在现实中，诸如信号的高低电平间的跃迁是需要一定时间的（虽然时间特别短），从一种信号转换到另一种信号是有一定延迟的。Verilog HDL 的仿真器遵循这个规律，统一规定在某个确切瞬间的时刻的信号值与此瞬间前一时间单位刻的信号值相同，而在此瞬间时刻的后一刻才开始使用新的信号值。

![method-draw-imagebbdd5.png](https://miao.su/images/2019/05/29/method-draw-imagebbdd5.png)

## 模拟时钟

一般来讲，在实际的时序电路中，时钟信号依赖于外部的专用硬件。但在仿真时，我们无法去关联或构建一个专用的时钟硬件，只凭 Verilog HDL 写不出来。所幸，仿真时还是有手段去模拟时钟的：

```verilog
always #10 begin
    clk <= ~clk; // 每 10 单位时间反转一遍时钟信号
end
initial begin
    #0 begin
        clk <= 1'b0; // 在时刻 0 初始化
    end
    ...
end
```

## 系统任务

系统任务是 Verilog HDL 内置的一些底层函数，可用于控制仿真、逻辑综合等的流程，或进行日志的输出、波形的生成等。

- ```$signed(var)```

将一个变量名所对应的线路或寄存器上的数据当做有符号数处理。

- ```$unsigned(var)```

将一个变量名所对应的线路或寄存器上的数据当做无符号数处理。

- ```$display(string, ...)```

类似于 C 语言的 printf 函数，主要用于调试输出。它会将第一个参数以后的所有参数分别与第一个字符串参数中的通配符意义对应，并以这些参数所指向的线路或寄存器的值替代那些通配符，最后向控制台输出一行合成出的字符串。

- ```$write(string, ...)```

与 ```$display``` 功能一致，唯一不同的是它输出的内容末尾不换行。

- ```$time```

返回当前仿真已经进行了多长时间的值。

- ```$finish```

结束仿真。

- ```$readmemh(file, var)```

读取由第一个参数制定路径的文件，将其中的数据读入第二个参数所对应的寄存器阵列中。这可以很方便地快速读入、初始化存储器，例如以这种方式读入操作系统的内核引导程序。

由 ```$readmemh``` 读取的镜像文件为一连串记述二进制数据的十六进制数字文本文件。对于寄存器阵列，镜像文件中的每一行数据依次赋予阵列中的每个元素。

例如，有一个镜像文件 ```core.dat```：

> 00112233
>
> 1123ABCD
>
> FEDCBA98
>
> 11111111
> 
> ...

然后在源代码中引用：

```verilog
reg [31:0] mem [1023:0];
initial begin
    $readmemh("core.dat", mem);
end
```

此时寄存器阵列中的内容是这样的：

```
mem[0]: 00112233
mem[1]: 1123ABCD
mem[2]: FEDCBA98
mem[3]: 11111111
...
```

- ```dumpfile(file)```

将仿真得到的波形输出以 VCD 格式写入指定路径的文件中。与 ```$dumpvars``` 配合使用才能输出有意义的内容。

- ```dumpvars(begin_tick, module_or_wire)```

指定某个模块中的所有线路或某根具体的线路/寄存器的信号波形输出到由 ```$dumpfile``` 指定的文件中。

## parameter 类型

这个类型有点像 C 语言的常量类型，但它仅适用于仿真模块。例如：

```verilog
parameter STEP = 100.0000;
#(STEP / 2) begin
   ...
end
```

## Icarus Verilog 仿真程序的使用

Icarus Verilog 是一个 CLI，你可以通过对其输入命令行参数控制其具体的仿真行为细节。

Icarus Verilog 的安装程序会自动将已安装程序的路径加入到环境变量中。如果安装后 iverilog 命令无法正常执行（提示没有此程序），那么您应当检查并手动设置此部分的内容。

此 CLI 有以下选项参数：

- -D *macro* *\[=defn\]*

定义一个名为```macro```的宏，值为```defn```。如果未指定```defn```，将自动使用默认值 1。

- -l *includedir*

将```includedir```指向的路径加入进 include 语句包含文件时的搜索路径列表中。

- -o *filename*

指定仿真输出的文件名。

- -s *topmodule*

指定最顶层（相当于```main```）模块的名称。

- -y *libdir*

将```libdir```指向的路径加入进库文件的搜索路径列表中。


示例指令：

```
iverilog -s regfile test -o regfile test.out regfile test.v regfile.v
vvp regfile test.out
```

其中，```iverilog```用于生成编译后的电路文件（test.out），后续的```vvp```指令则用于执行仿真部分，并生成最终可用的波形文件。

要查看波形文件，可以使用软件 GTKWave。

