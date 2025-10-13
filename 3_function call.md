# function call
函数调用步骤
1. 指定参数位置，让函数能够访问到
2. 转移到函数
3. 获取函数需要的本地资源
4. 执行函数任务
5. 把 return value放置到调用者能访问的地方，存储使用过的寄存器，释放本地存储
6. 把控制流返回到起点
## 函数调用约定
| 名字 | x | 功能 |
|:------|-------|-------|
| a0-a7 | x10-x17 | argument,八个传参的regs，其中a0 a1存储返回值|
| ra | x1 | return address,存储返回地址，使程序能返回到起始点 |
| s0-s1 | x8-x9 | saved reg |
| s2-s11 | x18-x27| saved reg |

`jal : jump and link`一条指令，跳转并保存返回地址
```
1008 addi ra,zero,1016 # ra = 1016，保存返回地址
1012 j sum # goto sum
==>
1008 jal sum #保存返回地址并且跳转
```
jal不需要知道代码在内存中的哪里，自动将返回地址（调用者的下一个地址）保存在返回地址寄存器中

link意味着链接调用者地址，允许函数返回到正确地址。

从函数中返回: `jr ra`,jump register

伪指令`ret = jr ra`

>在RISC-V架构中，jal（Jump and Link）指令用于实现函数调用。它通过将程序计数器（PC）的值加4后的地址保存到指定的寄存器中，从而记录返回地址。具体操作：
>
>jal指令会将当前指令的下一条指令地址（即PC+4）保存到目标寄存器rd中。通常情况下，这个目标寄存器是x1，也称为ra（return address）。
>
>然后，jal指令会根据指令中的立即数（20位有符号偏移量）来更新程序计数器（PC），从而跳转到目标地址。
>
>例如，`jal x1, label`这条指令会将`PC+4`保存到x1寄存器中，然后跳转到label所指定的地址。当函数执行完毕后，可以通过jalr（Jump and Link Register）指令从x1寄存器中读取返回地址，从而跳回调用点

实际上，只有两条指令
```
jal rd, Lable #jump and link，把返回地址保存完跳转
jalr rd,rs,imm #jump and link reg，读取返回地址跳转回去
---------------------------------
j,jr,ret是伪指令
j: jal x0,Lable
```
### jalr 

rd：目标寄存器，用于保存返回地址。如果不需要保存返回地址，可以使用x0（零寄存器）。

rs：源寄存器，包含目标地址的基地址。

imm：立即数，是一个12位有符号偏移量，用于计算最终的目标地址。
1. 指令的执行过程
计算目标地址：

目标地址是通过将rs寄存器的值与imm立即数相加得到的。

公式为：target_address = rs + imm。

2. 保存返回地址：
   
返回地址是当前指令的下一条指令地址（PC + 4），这个地址会被保存到rd寄存器中。

3. 跳转到目标地址：
   
程序计数器（PC）被更新为目标地址，从而实现跳转。
#### 示例
假设当前PC值为0x1000，rs寄存器的值为0x2000，imm为0x10，rd为x1。

计算目标地址：
target_address = rs + imm = 0x2000 + 0x10 = 0x2010。

保存返回地址：
返回地址是PC + 4 = 0x1000 + 4 = 0x1004，这个值会被保存到x1(ra)寄存器中。

跳转到目标地址：
PC被更新为0x2010，程序跳转到这个地址继续执行。


jalr指令中的立即数imm允许在跳转时进行偏移，这提供了更多的灵活性。例如：

函数返回：在函数返回时，通常不需要偏移，可以直接使用jalr x0, x1, 0，这等同于jr x1（跳转到x1寄存器中的地址，不保存返回地址）。

间接跳转：在某些情况下，可能需要在基地址上加上一个偏移量来跳转到特定的地址。例如，跳转到一个数组中的某个元素对应的代码位置。


## 堆栈
在函数调用前，需要存储旧的寄存器值，当从函数返回后要恢复这些值

旧寄存器值存储在堆栈上，stack在内存中，需要一个reg指向它：sp（x2，stack pointer）

堆栈从顶部向下增长
|  | 地址 |
|:------|-------|
|frame | <-0xFFFFFFF0　|
|frame |　|
|frame |　|
|frame |　|
|frame | <-sp　|

stack frame包含：
1. 返回指令的地址
2. 参数
3. 其他变量
堆栈帧是连续的内存块，sp告诉stack frame底部是哪

当程序结束，堆栈帧抛弃这块stack，free这块内存：
``` 
#序言proloal
addi sp,sp,-8 #stack增长2 byte，用于存储两个变量
sw s1,4(sp) # 入栈，把s1原有的值保存起来
sw s0,0(sp) # 入栈 s0
#程序执行
add s0,a0,a1 # s0存第一个结果
add s1,a2,a3 # s1存第二个结果
sub a0,s0,s1 # a0存储返回值
#结尾
lw s0,0(sp) # 恢复s0原值
lw s1,4(sp) # 恢复s1原值
addi sp,sp,8 # 调整堆栈删除这两个变量的空间
```
 ### 嵌套调用和reg 协议
 x1存返回地址，当main调用f1，f1调用f2，那么处理完f2后f1不知道回哪里了

 caller：调用者；callee：被调用的函数

 当从callee返回时，需要知道哪些寄存器是可能改变的、哪些是不会变的

 把寄存器分为两类：保存的寄存器和易丢失（临时）寄存器

 1. 保存的寄存器，可以相信值未改变
   
    saved register（s0-s11），s0（fp），sp，gp，tp

 2. 不保存的寄存器
    
    参数寄存器，返回寄存器：a0-a7，ra

    暂时寄存器：t0-t6

不保存的寄存器需要进入堆栈才能保证其不丢失

非x+序号的寄存器是ABI name，可以在汇编中调用（比如a0，ra，t0等等）

## 内存分配
C有两种存储类型：automatic 和 static

自动变量是函数的本地变量，当函数退出就会被丢弃

静态变量始终存在于程序的起始到结束

用堆栈来保存不能存在寄存器的变量

程序框架或激活记录：保存寄存器和局部变量的堆栈段

```
1.before call
---
| | <--sp
| |
| |
---

2.during call
---------------------------------------------
| saved return addr(if needed) 保存的返回地址  | 
| saved argument regs(if any) 保存的参数寄存器 |
| saved saved regs(if any) 保存的保存寄存器    |
| local variables(if any) 本地变量            | <--sp
---------------------------------------------

3.after call
---
| | <--sp
| |
| |
---
```
使用大块数据时，sp可能会很大
### example sumSquare
```
int sumSquare(int x, int y){
    return mult(x,x)+y;
}
a0存x，a1存y

sumSquare:
 #push
    addi sp,sp,-8
    sw ra,4(sp) # 把当前函数的返回地址存到内存中
    sw a1,0(sp) # save y
    mv a1,a0 # mult(x,x)
    jal mult # call mult，jal自动把ra存成当前指令的下一条了（lw），执行完mult自动跳回来
    lw a1,0(sp) # restore y
    add a0,a0,a1 # mult()+y，a0存储的是mult调用的返回值
    lw ra,4(sp) #get ret addr取回返回地址
 #pop
    addi sp,sp,8 #restore stack 删除栈
    jr ra # 退出函数调用

在本函数中 sumsquare：caller；mult：callee
```
### RV32约定（RV64、RV128有不同的内存布局）
stack从高内存地址向下增长：BFFF_FFF0（hex）

stack必须16字节对齐（之前的example没处理这个）

RV32的程序（text segment）在低地址：0001_0000(hex)
```
---------------------
|stack,goes down     |<---sp = BFFF_FFF0
|                    |
|                    |
|                    |
|dynamic data,goes up|
---------------------
|static data         |
--------------------- <---1000 0000
|text                |
--------------------- <---pc = 0001 0000
|reserved            |
----------------------
底部是系统调用内容，中断，IO等
```
# 总结
## 算数、逻辑
```
add rd,rs1,rs2
sub rd,rs1,rs2
and rd,rs1,rs2
or  rd,rs1,rs2
xor rd,rs1,rs2
sll rd,rs1,rs2
srl rd,rs1,rs2
sra rd,rs1,rs2
```
## 立即数
```
addi rd,rs1,imm
subi rd,rs1,imm
andi rd,rs1,imm
ori  rd,rs1,imm
xori rd,rs1,imm
slli rd,rs1,imm
srli rd,rs1,imm
srai rd,rs1,imm
```
## load、store
```
lw  rd,rs1,imm
lb  rd,rs1,imm
lbu rd,rs1,imm
sw  rd,rs1,imm
sb  rd,rs1,imm
```
## branching、jumps
```
beq rs1,rs2,Label
bne rs1,rs2,Label
bge rs1,rs2,Label
blt rs1,rs2,Label
bgeu rs1,rs2,Label
bltu rs1,rs2,Label
jal rd,Lable
jalr rd,rs,imm
```