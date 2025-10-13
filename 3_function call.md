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
## 堆栈
在函数调用前，需要存储旧的寄存器值，当从函数返回后要恢复这些值

旧寄存器值存储在堆栈上，stack在内存中，需要一个reg指向它：sp（x2，stack pointer）

堆栈从顶部向下增长

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
 