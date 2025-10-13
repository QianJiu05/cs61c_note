# intro
处理器结构：从上到下

CPU - Registers - CPU Cache - L1 Cache - L2 Cache - L3 Cache - Physical Memory - Random Access Memory (RAM) - Virtual Memory - Solid-State Memory (Flash,like SSD/HDD) - Magnetic Disks

每级内存都增加一个量级，价格也便宜一个量级

冗余思想：计算三次，取出现最多次数的答案作为结果
# number representation

真实的世界是模拟的（连续），要导入模拟信号，需要：1.采样，2.量化

N bits 可以表示 2^N

偏置编码：利用偏置，把一个范围的数字映射到另一个范围，比如int32->uint32都表示65535个数字，但是一个是只有正数的。
# C basics
C编译器将C映射为机器语言（0和1）

将 .c文件编译成 .o文件，然后连接这些 .o文件成为可执行文件executables

.o就是机器码

可执行文件是与架构有关的，比如MIPS、X86、RISC-V，也与系统有关，比如Win、Linux

## C Pre-Processor 编译前的预处理
foo.c->[CPP]->foo.i->[Compiler]

1. CPP把注释全用space替代
2. CPP把#include的内容导入到文件中
3. CPP把#define的内容替换
4. CPP控制#if/#endif的内容

通过int main(int argc, char *argv[])来接收参数

malloc作用于heap，但是对malloc的调用需要增长stack
# float
只要发送有效位和所处位置就可以表示float

对于浮点数：1.01（two） x 2 ^ -1 
| 表头1 | 表头2 |
|:------|-------|
|. | binary point二进制小数点 |
|1|mantissa位数|
|2|base|
|-1|exponent|

float32：

31-30（1bit）：S

30-23（8bit）：指数

22-0（23bit）：significand

float = -1 ^S x (1 + Significand) x 2 ^[指数-127]

为了支持包含∞的计算，/0应该产生∞而不是overflow。（比如X/0 > Y是可能的表达式)


当进行加法时，如果有一个很大的数和1，float无法同时存储1和这个大数，那么1将被舍弃。所以浮点数的加法是不具有结合律的

pricision：精度；accuracy：与实际值的差距

四种舍入方法：
1. 向上
2. 向下
3. 直接舍弃最后一位
4. 奇数时0.5进1，偶数时0.5舍弃。3.5->4;2.5->2;

半精度fp16常用于机器学习

