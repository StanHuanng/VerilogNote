# Verilog HDL笔记

### Chapter 1

#### 基本数值

##### 多行注释

/* 

注释内容

*/

##### 标识符

字母、数字、￥、_

不能以数字开头、标识符的字母区分大小写

##### 数值

1: VDD

0: GND

X: unknow state

Z:高阻态

##### 整数及其表示

**位宽 ‘ 进制符号 ’ 整数**

4'b'0101:四位二进制数0101

##### 

#### 数据类型

根据信号强度对数据进行分类

supply-strong-pull-large-weak-medium-small-highz

##### 连线型数据类型

wire:0,1,X

tri:0,1,X

supply1 电源线，高电平1

supply0 电源线，低电平0



##### 寄存器型数据类型

reg a;

reg[3:0] b;定义了一个4位的名为b的reg型变量

若将一个负数赋给reg型变量，将自动转换成其二进制补码形式



##### 声明连线型数据类型

数据类型 驱动强度 标/矢量 指定仿真延迟时间 [变量名]



##### 寄存器型数据类型

reg 位宽（可省略）变量名



##### 存储器型变量

reg[7:0]mem1[255:0]

// 定义了一个有256个8位寄存器的存储器mem1

//地址范围是0到255



#### 运算符和表达式

**算术表达式结果的长度由最长的操作数决定**

在数值比较中，操作值为不定态或高阻态，输出结果都为不定态 X

 可靠性设计要求不定态和高阻态进行比较

**若a=4'b1001,!a=1'b0** 注意逻辑非的概念

逻辑与&&



#### 按位运算符

对信号的每一位进行单独操作

按位取反~ 按位与& 按位或|

​	异或^

#### 移位运算符

<< >>左移位运算符和右移位运算符

用0填补移出的空位



#### 连接与复制运算符

 连接运算符”{ }"：将多个信号的某几位连接成为新的信号

复制运算符“{{}}”

```
a=3'b110;

c={2{a}};

c=6'b110110;
```



#### *模块

```
module name(port_list)
	端口定义
	数据类型说明
	逻辑功能描述
endmodule
```



EX 上升沿D触发器

```
module dff(din,clk,q)
	//端口定义
	input din,clk;
	output q;
	reg q;
	//功能描述，使用非阻塞赋值故用always
	always@(posedge clk)
		q<=din;
endmodule
```



### Chapter 2

#### 数据流建模

##### 连续赋值语句

连续赋值的目标类型为标量线网和向量线网（wire）

```
assign y=m|n;
assign #(3,2,4)c=a&b;
```

连续赋值语句是并行的，与位置顺序无关

连续赋值语句不能出现在过程块中

连续赋值语句中任何**小于其延时的信号变化脉冲都将被滤除掉**，不会体现在输出端口上

#### 行为级建模

过程语句：initial, always

语句块：串行语句块begin-end,并行语句块fork-join

赋值语句:过程连续赋值assign，过程赋值=、非阻塞式赋值<=

##### *非阻塞式赋值<= 

- **并行性**：在同一个 `always` 块中，所有非阻塞赋值语句会**同时执行**，顺序不影响结果。

- **时序逻辑建模**：专门用于描述时序逻辑（如寄存器、触发器），模拟硬件中多个寄存器同时更新的行为。

- **避免竞争条件**：确保在时钟边沿触发时，所有赋值操作基于原始值进行计算，再统一更新结果。

  ```
  always @(posedge clk) begin
      // 非阻塞赋值，同时更新所有寄存器的值
      reg1 <= a + b;  // 步骤1：计算 a+b，暂存结果
      reg2 <= reg1;   // 步骤2：使用 reg1 的旧值（非阻塞赋值的特性）
  end
  ```

  

##### initial过程语句

主要用于产生测试向量

```
initial
begin
	A=0;B=1;C=0
	#100 A=1;B=0;
	end

```

##### always语句块

always语句块的触发状态是**一直存在的**，只要满足always后面的敏感事件列表，就执行过程块

敏感事件：
@(a) 信号a的值发生变化时（**包括上升沿变化和下降沿变化**）

@(a or b) / @(a, b）信号a或信号b的值发生改变时

@(posedge clock) 上升沿触发

@(negedge clock) 下降沿触发

@(posedge clk or negedge reset) 当clk的上升沿没到来或reset信号的下降沿到来时



**采用过程对组合电路进行描述时，作为全部的输入信号需要列入敏感信号列表**

**采用过程对时序电路进行描述时，需要把时间信号和部分输入信号列入敏感信号列表**

EX 4in1 MUX

```
module mux4_1(out,in0,in1,in2,in3);
	output out;
	input in0,in1,in2,in3;
	input[1:0]sel;
	reg out;
	always @(in0,in1,in2,in3,sel)
		case(sel)
		2'b00: out=in0;
		2'b01: out=in1;
		2'b10: out=in2;
		2'b11: out=in3;
		default: out=2'bx;
		endcase
	endmodule
```



EX同步置数、同步清零计数器

```
module counter1(out,data,load,reset,clk)
	output[7:0]out;
	input[7:0]data;
	input load,clk,reset;
	reg[7:0]out;
	
	always@(posedge clk)
	begin
		if(reset) out=8'h00;
		else if(load) out=data;
		else out=out+1
		end
endmodule
```

如果是**异步**清零计数器，在敏感事件表加入clear信号

```
always @(posedge clk or negedge clear)
begin
	if(!clear)
	out=0;
	else out=out+1
end
```

不直接写clear是因为实际硬件清零是电平敏感，但Verilog用边沿触发敏感列表+条件判断模拟该行为。



**在assign语句中左边的变量一定要赋值为wire类型**

**在initial和always语句中左边的变量一定要赋值为reg类型**



#### 语句块

串行：begin-end

并行：fork-join

串行执行时语句依次执行，存在延迟

并行执行时语句同时执行，语句间不存在延迟



#### 过程赋值语句

阻塞赋值语句 "="

非阻塞赋值语句"<="

**阻塞赋值语句先计算等号右端的值，，然后立刻将计算的值赋给左边的变量，与仿真时间无关**

**非阻塞赋值语句先计算等号右端的值，等到延时时间结束时，将计算的值赋给左边的值**

只在begin-end中考虑，fork-join中不考虑

EX：如何用阻塞语句写出非阻塞语句的电路特点

```verilog
module example(clk,din,out1,out2);
	input clk,din;
	output out1,out2;
	always@(posedge)
		begin
//实质为调换两个非阻塞性赋值的顺序，先赋值后者，生成的电路便会产生两个gate
		out2=out1;
		out1=din;
	end
	endmodule
		
```

流水线设计：将组合电路拆分为多个小的组合电路，每一个电路的时间常数小于大的，从而在整体上达到提高电路速度的作用



只有在行为级描述中的串行语句块中使用阻塞性赋值语句，才是串行结构，**其他在Verilog中都是并行结构**



##### 过程连续赋值语句

assign deassign force release(后两者优先级更高)

当临时改变变量的值时，用deassign将赋予临时值的变量初始化



##### case语句

```
case(控制表达式)
	值1：语句块
	值2：语句块
	default:语句块
endcase
```

若在case语句中未包含全所有状态，将会产生锁存器



##### repeat循环语句

用repeat所引导的循环语句表示执行固定次数的循环

```verilog
module repeat_tb;
reg clock;
initial
begin
	clock=0;
	repeat(8)#50 clock=~clock
//产生8个时钟周期
end
endmodule
```

 

##### for循环语句

当采用循环语句进行计算和赋值的描述时，可以综合等到逻辑电路

EX用for实现8位移位寄存器

```verilog
module shift_regist(Q,D,rst,clk)
output[7:0]Q;
input D,rst,clk;
integer i;
    always@(posedge clk)
        if(!rst)Q<=8'b00000000;
    else
        begin
            for(i=7;i>0;i=i-1)
                Q[i]<=Q[i-1];
        end
endmodule
```

当for中的变量没有具体的物理意义，仅仅用作表征关系



