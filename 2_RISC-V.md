# assembly language
CPU的工作就是执行指令。不同的CPU执行不同的指令集。特定实现的指令集是ISA（Instruction Set Architecture，指令集架构）

RISC：让指令短，可以在一周期内运行更多的指令。让软件通过组织简单指令来执行复杂的操作

CISC本质是RISC构建引擎，复杂指令在机器内部连接简单指令

指令集是一种特定的体系结构可以执行的指令集合

每行汇编代码代表一条计算器的指令。

一个4Ghz的CPU每0.25ns就可以访问一次寄存器

汇编没有变量，所有的操作都是对于寄存器的。非常快！

处理器通过发出地址，从内存读取数据或向内存写入数据来与内存通信

所有内存系统的目标都是使内存看起来无限快且无限大。由于只有很少的寄存器，希望让大部分内存看起来像寄存器一样快速
# RISC-V
RISC-V有32个寄存器，每个寄存器都是32bit的

32bit：1 word in RV32

在RV32中，每个字4字节，32位（bit） 字的宽度与架构变种相关联

汇编没有变量，操作它们的操作（指令中的动词）决定了实际上如何处理寄存器的内容。每行最多一条汇编指令

x0永远=0，因为总是需要用到0

## add assembly
add x1,x2,x3 --> a = b + c: x1=a,x2=b,x3=c -->x1 = x2 + x3

sub x3,x4,x5 --> x3 = x4 - x5

```
f=(g+h)-(i+j)
add x5,x20,x21 # a_temp = g+h
add x6,x22,x23
sub x19,x5,x6 # f = a_temp - b_temp
```
常数是立即数，`addi x3,x4,10 : x3 = x4 + 10`

减法：把立即数设置为负值，用补码表示，`addi x3,x4,-10`

把x4的内容移动到x3:`add x3,x4,x0`前文提到x0永远是0

优化编译器的使用通常是最小化寄存器的使用。对于特定内核使用的寄存器数量称为该内核的寄存器足迹

地址通常是基地址的偏移量。
数据存储到内存，从内存中加载

## memory address
数据一般小于32bits，大于8bits：如果所有东西都是8bits的倍数，处理器处理起来会比较方便

`8 bits = 1 byte`
`1 word = 4 byte = 4*8 bits`

每个32位的word可以容纳4个bytes

RISC-V是小端序的，最低有效字节获得最低的字节地址

如图所示内存片段：

| 31-24 | BYTE2 | BYTE1 | BYTE0 | 字的地址 |
|:------|-------|-------|-------|-------|
|15|14|13|12|[12]，第四个word|
|11|10|9|8|[8]，第三个word|
|7|6|5|4|[4]，第二个word|
|3|2|1|0|[0]，第一个word|

读取顺序：0-3，4-7，...

从下到上，word的地址分别是0，4，8，12。本质上字的地址与其最低的有效字节的地址相同

对于这么一个数字：
| BYTE3 | BYTE2 | BYTE1 | **BYTE0** | 
|:------|-------|-------|-------|
| 00000000 | 00000000 | 00000100 | **00000001** | 
大端表示：
| ADDR3 | ADDR2 | ADDR1 | ADDR0 | 
| **BYTE0** | BYTE1 | BYTE2 | BYTE3 | 
| **00000001** |  00000100 | 00000000 | 00000000 |
小端表示：
| ADDR3 | ADDR2 | ADDR1 | ADDR0 | 
| BYTE3 | BYTE2 | BYTE1 | **BYTE0** | 
| 00000000 | 00000000 | 00000100 | **00000001** | 

## 数据转移指令（lw、sw）
lw：load word，方向：从内存到寄存器

偏移是以字节（byte）为单位的

`g = h + A[3]` -->
```
lw x10,12(x15) # x10获得A[3],12是A偏移3word，3*4=12 bytes。x15是A[0]的指针

add x11,x12,x10 # g = h + A[3]
```
sw：store word，方向：从寄存器到内存

sw和lw的偏移量可以是任意长度。

riscv支持非内存对齐，但是效率很低，应该尽量写成内存对齐（4的倍数）

lb：load byte，以字节为单位:`lb x10,3(x11)`

符号扩展LBU：为了保留符号，把符号位x复制出来，写到所有bit里面。语法和LB一样

假设x10：xxxx xxxx xxxx xxxx xxxx xxxx [xzzz zzzz]

x10应该把第一个x进行拓展，前面所有bit都置为x

### example
```
addi x11,x0,0x3F5
sw x11,0(x5)
lb x12,1(x5)
```

res: 

x5[00 00 03 F5]

x11[00 00 03 F5]

x12[00 00 00 03] : x12加载x5的偏移1 byte数据（即忽略F5），由于有符号拓展，自动补0

---
addi的操作可以用lw+add来完成，lw把常数写进内存，再读取到reg里，然后add，但是会很慢

addi的立即数范围是不能超过32位的，所以会比较小，要更大的立即数需要用别的方法生成

## 分支预测
beq：branch if equal，相等时分支
```
beq reg1,reg2,L1
==>
if((value in reg1) == (value in reg2)){
    goto L1(lable);
}else{
    next;
}
```
bne: branch if not equal

branch语句改变了控制流
| 指令 | 全称 | 含义 |
|:------|-------|-------|
| beq | branch if equal | |
| bne | branch if not equal | |
| blt | branch if less than | |
| bge | branch if greater than/ equal | >= |
| bltu | 无符号版本blt | 比较，把reg的值视为无符号数 |
| bgeu | 无符号版本bge | |
一定跳转：jump(j),`j lable`

比较需要两个寄存器，不能与立即数进行比较

有小于和大于等于，所以没必要有另外两个指令，节省指令数量；但是有bltu用于比较无符号
### example
```
int a[20];
int sum = 0;
for(int i=0; i<20; i++){
    sum += a[i]
}
==>assembly
    add x9,x8,x0 # x9 = &a[0]
    add x10,x0,x0 # init sum = 0
    add x11,x0,x0 #init i = 0
    addi x13,x0,20 #init x13 = 20
Loop:
    bge x11,x13,Done #if x11 > x13(loop finish) goto Done
    lw x12,0(x9) # x12 = a[0]
    add x10,x10,x12 # sum += x12
    addi x9,x9,4 # &a[i+1],指针以四字节递增
    addi x11,x11,1 # i++
    j Loop
Done:
```
## 逻辑指令
| 逻辑 | 指令 | 逻辑操作 |
|:------|-------|-------|
| & | and | 按位和 |
| \| | or | 按位与 |
| ^ | xor | 按位异或 |
| << | sll | 左移，shift left logical |
| >> | srl | 右移， shift right logical |

