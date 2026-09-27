---
title: "Verilog基本语法"
date: 2026-01-03
tags:
  - verilog
publish: true
---
## 1. 标识符命名规则

- 变量**第一个字符必须是字母或者下划线**，不能以数字或者美元符开始
- 变量命名区分字母大小写
- 命名不可与关键字一致 (verilog 中关键字全为小写)

## 2. 模块结构
模块的结构：端口定义、I/O说明、内部信号声明、功能定义

一个带异步清除端和使能端的D触发器
```verilog
module DFF(q, clk, d, clr, en);        // 端口定义
input clk, d, clr, en;                  // I/O说明
output q;                        
reg q;                                  // 内部变量定义

always @(posedge clk or posedge clr);   // 功能定义
begin
    if (clr) 
        q <= 0;
    else if(en)
        q <= d;
end
endmodule
```
### 2.1. 端口定义
`module name(port1, port2, port3);`
### 2.2. I/O说明
`input [<范围>] 变量名;`
`output [<范围>] 变量名;`
`inout [<范围>] 变量名;`
### 2.3. 内部信号声明
`reg [<范围>] 变量1, 变量2, ...;`
### 2.4. 功能定义
- assign方式 `assign c = a & b;`
    - 常用于组合逻辑
- 调用模块方式 `and u1(q, a, b)`
- always块
    - 可以描述时序/组合逻辑
    
> 在verilog中所有过程块、赋值语句、模块引用都是并行的
### 2.5. 模块调用
方式一：按顺序填充.。方便，但是不利于后期改动
```verilog
tri u1(sout, sin, ena);
```
方式二：使用原定义端口名标明
```verilog
tri u1(.out(sout), .in(sin), .enable(ena));
```
## 3. 数据类型
逻辑值|状态
---|---
0|低电平
1|高电平
z|高阻态
x|不定态
### 3.1. 常量
> 在程序运行过程中，其值不能被改变的量称为常量。默认位宽为32位
- 数字
    - 整数
        - 数字表达方式 `<位宽>' <进制><数字>`
        - 位宽缺省时，采用默认位宽32（与仿真器和计算机有关）
    - x/z
        - 都只能表示当前进制下某一位的状态
        - x表示不定态
        - z表示高阻态，写为问号符?更常见
    - 负数
        - 只需在位宽表达式最前加一个减号
    - 下划线
        - 只能用在数字之间
        - 可以提高可读性
- 参数型
    - 赋值语句右边必须是一个常数表达式
    - 常用于定义延迟时间和变量位宽
    - 可通过参数传递改变模块引用的参量
``` verilog
4'b1110    // 4位二进制数
10'h3ac    // 10位十六进制数
'd210      // 32位十进制数
8'b101_001_01;   // 合法
8'b_1001_1101;   // 非法
parameter r = 3;    // 声明r为实型参数3
```

### 3.2. 变量
- 线网wire
    - 默认值为z
    - I/O类型缺省时，默认为wire
- 寄存器reg `reg [n-1:0] a;`
    - 默认值为x
    - 在always块中被赋值的信号必须是reg型
- 存储器memory
    - memory是通过拓展reg型得来的
    - 通过一条赋值语句不能完成对存储器的赋值！要对其赋值，必须指明是哪个单元
```verilog
wire [n-1:0] 数据名1, 数据名2, ..., 数据名i；    // 共有i条总线，每条总线内有n条线路
wire [n:1] a;        // 注意此时第一条线路为a[1]
reg [n-1:0] 存储器名[m-1:0];    // 定义了m个存储器单元，每个单元是一个n位存储器
reg 存储器名[m-1:0];            // 定义了m个存储单元，每个单元是1位寄存器
```
## 4. 运算符和关键词
### 4.1. 运算符
- 基本算术运算符
    - 除法四舍五入
    - 求模结果符号与被操作数一致
    - 若有不定态x参与，则结果为x
![](/attachment/537825021249570.png)
- 位运算符
    - 长度不等时，位数少的操作数将自动在高位填充0
![|500](/attachment/309405021256901.png)
- 逻辑运算符
![|263](/attachment/191535321246125.png)
- 关系运算符
    - 若出现一个操作数不定，则返回值为x
![|375](/attachment/373965321241879.png)
- 等式运算符
    - 逻辑等式运算符，若出现x/z，结果为x
![|275](/attachment/127355421259759.png)
![|325](/attachment/312405521257363.png)
- 移位运算符
        - 在位数足够的情况下，左移可以实现乘2
        - 右移一定可以实现除2
![|475](/attachment/22545721254865.png)
- 位拼接运算符
    - 格式：{a, b[3:2], w}
    - 不允许存在未指明位宽的信号
    ![|350](/attachment/68145921236106.png)
- 缩减运算符
![|275](/attachment/350000022258546.png)
### 4.2. 关键词
- 关键词都是用小写字母定义
- 在变量定义时，不可引起冲突


## 5. 赋值语句和块语句
### 5.1. 赋值语句
- 非阻塞赋值(<=)
    - 进程开始，右端表达式同时计算；时钟到来，同时赋给左端变量
    - 常用于时序逻辑
![Non_Blocking|425](/attachment/517503321248868.png)
- 阻塞赋值(=)
    - 适用于组合逻辑电路
    - 上一条赋值语句可以在下一条语句立刻生效
    - 此时级联的寄存器失效，等效为同一级寄存器
![Blocking|325](/attachment/113603421236735.png)

实例：非阻塞赋值实现变量互换
```verilog
module exchange(x, y, clk, load, a, b);
output x, y;
input clk, load;
input a, b;
reg x, y;

always @(posedge clk or posedge load)
begin
    if(load) begin
        x <= a;
        y <= b;
    end
    else begin
        x <= y;
        y <= x;
    end
end
endmodule
```
### 5.2. begin_end块语句
- 这是一种顺序块，块内语句是从上到下执行的
- 只有最后一条语句执行完，才跳出当前块
- 每条语句的延迟实践是相对于前一条语句的仿真时间而言的
```verilog
// 方式1
begin
    语句...；
end
//方式2(可调用)
begin: 块名称
    语句...；
end
```
> fork_join语句是并行块
## 6. 条件语句
### 6.1. if_else 语句
> else总是与上一个 if 进行配对，为了保证逻辑关系不错误，可用begin_end块进行包裹
```verilog
// 语句示例
if(a > b)
    out <= a;
else
    out <= b;
// 一位十进制计数器：满十进一
module Counter10(q, clk, clr);
output [3:0] q;
input clk, clr;
reg [3:0] q;
always @(posedge clk or posedge clr) begin
    if (clr)
        q <= 4'b0000;
    else begin
        if (q == 4'b1001)
            q <= 4'b0000;
        else
            q <= q + 1'b1;
    end
end
endmodule
```
### 6.2. case语句
- case语句的表达式位宽必须相等，这样才能比较各位是否相等
- 找到第一个满足case表达式后，执行分支并跳出
![|500](/attachment/295563422253682.png)
- casez表示有z则该位相等
- casex表示x/z则该位相等
## 7. 循环语句
![|400](/attachment/461165522247228.png)
### 7.1. forever
- 执行无限循环，指导遇到系统任务$finish
- 常用于生成周期性波形，作为仿真测试信号
- 必须写在initial块中
```verilog
reg clk;
initial
begin
    clk = 1'b0;
    forever #10 clk = ~clk    //每十个时间单位翻转一次
end
```
### 7.2. repeat
```verilog
repeat(10)
begin
    $display("count = %d", count);
    count = count + 1;
end
```
### 7.3. while
对rega中1的个数进行计数
```verilog
begin
    reg [7:0] temp;
    count = 0;
    temp = rega;
    begin
        if(temp[0])
            count = count + 1;
        temp = temp >> 1;    
    end
end
```
### 7.4. for
初始化一个memory变量
```verilog
begin
    reg[7:0] temp;
    for(temp = 0; temp < size; temp = temp + 1)
        memory[temp] = 0;    // 将memory的各个单元赋为0
end
```
## 8. 结构说明语句、系统函数和任务
### 8.1. initial语句
- 一个语句只执行一次
- 多个initial语句是并行的
- -常用于产生测试信号和设置信号记录
### 8.2. always语句
- 每次触发都会执行
- always块是并行的
- `@*` 或 `@(*)` 表示对后面语句块中所有输入变量都是敏感的
- 边沿触发的always常用于描述时序逻辑，电平触发的always常用于描述组合逻辑
>在新版本语法中，可用逗号来替代关键字or
### 8.3. task
- 可定义自身仿真时间单位
- 可调用其它任务和函数，只有当这个任务完成后，控制才能返回
- 输入变量、输出变量任意
- 任务本身没有返回值
- 没有assign 、initial、always这类语句
![|500](/attachment/480722120248869.png)
```verilog
module task_myInstance (
    a, b
);
    input [1:0] a;
    output [1:0] b;
    reg [1:0] b;
    task NOT_new;
        input [1:0] in;     // 只能是输入输出，不能写其类型
        output [1:0] out;
        begin               // 注意没有assign\always等语句 
            out = ~in;
        end
    endtask
    always @(a, b)
        NOT_new(a, b);      // 真正的赋值控制开关在调用任务处
endmodule
```
### 8.4. function
- 与主模块共用一个仿真时间单位（不能包含任何的时间控制语句）
- 不能启动任务
- 至少有一个输入变量，不含输出端口
- 隐含一个以函数名为名的寄存器，用于保存返回值
![|500](/attachment/296182120230443.png)
```verilog
module func_myInstance();
input [15:0] old_byte;
output [15:0] new_byte;
reg [15:0] new_byte;
function [15:0] switch_byte;    // 隐含了一个和函数名一样的变量switch_byte
    input [15:0] old_byte;   
    begin
        switch_bytes = {old_byte[7:0], old_byte[15:8]};
    end
endfunction
endmodule
```
### 8.5. 常用的系统任务
![|475](/attachment/329533120236736.png)
- display自动换行
- write则只在一行里输出，需通过加换行符\n来实现换行
#### 8.5.1. 格式说明
![格式说明|425](/attachment/124753320256902.png)
#### 8.5.2. 普通字符
![普通字符|500](/attachment/16363420246126.png)
#### 8.5.3. 显示位宽问题
- 默认按照当前位宽的最大可能位宽进行输出
    - 例如用二进制进行输出，和用十进制进行输出，它们的位宽是不一样的
- 在%和进制字符前添加一个0，这样总是用最小的位宽来显示当前值
`$display("d=%0h a=%0h", data, addr);`

