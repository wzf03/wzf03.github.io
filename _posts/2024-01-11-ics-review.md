---
title: "ICS 复习提纲"
date: 2024-01-11 19:14:00 08:00
categories: [PKU, CSAPP]
tags: [linux, csapp, lesson]
math: true
img_path: /assets/img/csapp_review
image: ../covers/csapp.jpg
---

## 信息的表示和处理

### 进制转换

- 十转十六：反复除 16，余数依次从低向高放置
- 十六转十：每一位乘 16 的对应次幂
- 十六进制与二进制互转

### C 语言类型

- long 一般在 64 位为 8 字节，在 32 位为 4 字节
- char 的符号 C 标准并不保证，大多数编译器视为**有符号**
- int 为 4 字节
- C99 引入 `int32_t`，`int64_t` 等 `<inttypes.h>`

### 字节顺序

- 大端法：最高有效字节在前
  - IBM 和 Oracle 大多数机器（不包括使用 Intel 兼容的处理器的个人计算机）
- 小端法：最低有效字节在前
  - x86_64（即 Intel 兼容机）
- 双端法：可以配置为大端或小端
  - ARM（但 Android 和 IOS 只能运行在小端模式）

### ASCII

- 终止字节：`0x00`
- 十进制数字 $x$: `0x3x`
- a-z: `0x61`-`0x7A`
- A-Z: `0x41`-`0x5A`

### 移位运算

- 算数右移：在左端补上原**最高有效位**的值
  - 几乎所有机器/编译器组合对**有符号数**使用算数右移
- 逻辑右移：补零
  - 无符号数
- 移位量过大是 UB
  - $w$ 位数据类型，移位 $k$，**许多**机器上位移量由 $k \mod w$ 得到
- 移位运算符在 C 语言中的优先极很低

### 整数表示

- 无符号： $UMax_w = 2^w -1$
- 有符号： $TMin_w = - 2^{w-1}\qquad TMax_w = 2^{w-1} -1$
- 补码中负数转二进制：对应正数取反（反码）加 1
- 补码中二进制转数：$B2T_w = - x_{w-1}2^{w-1} + \sum^{w-2}_{i = 0}x_i2^i$
- C 语言没有要求必须以补码来表示有符号数，但是要求了其取值范围（恰好为补码可以表示的取值范围）
  - `<limits.h>`: `INT_MAX`, `UINT_MAX`
- 反码：所有位取反表示负数；原码：符号位为 1 表示负数
- 有符号和无符号互转：位值不变（二进制表示不变，翻译方式改变）
- C 语言中的整数字面量：![image-20231102171739456](./image-20231102171739456.png)
- C 语言中的字面量不能直接表示负数，所以 `-1` 实际上是对 `1` 做取负操作。因此 `-2147483648` 是会出问题的，要用 `-2147483647-1`
  - 字面量 `2147483648` 会被解释成 `unsigned int` 类型
  - 如：在 C90 标准的 32 位程序中：(`-2147483648 < 2147483647`)为假（`2147483648u` 取负后仍不变）
- 拓展位表示
  - 有符号：符号扩展
  - 无符号：零扩展
  - 短类型提升时先拓展再考虑有无符号的转变，`short` 转 `unsigned` 时先转 `int` 再转 `unsigned`
- 减少位表示：截断
- 运算
  - 有无符号加法：溢出、截断
    - 无符号溢出检测：$s = x +_w^uy$，当$s < x$（$s < y$）时即溢出
    - 无符号加法逆元：
      $$\require{cases} -^u_wx = \begin{cases} x, & x = 0 \\ 2^w-x,&x > 0 \end{cases}$$
    - 有符号溢出检测：$s = x+^t_w y$，当$x > 0, y > 0, s\le0$或$x < 0, y < 0, s\ge0$时发生溢出，负数加正数不会发生溢出
    - 有符号加法逆元：
      $$-^t_wx =\begin{cases}TMin_w,&x = TMin_w \\ -x,&x > TMin_w \end{cases}$$
  - 减法：$x-y = x+(-y)$
  - 乘法：截断，**有无符号的乘法位级表示一致**
    - 优化：位移和加法组合（是否选用与运算和机器都相关）
    - $x*14 =((x << 3)+(x << 2)+(x << 1))=((x << 4)-(x << 1))$
  - 除法：很慢，**舍入到 0**
    - 右移：**向下舍入**。对于无符号舍入正确，对于有符号负数错误。
    - 向上舍入：$(x+(1 <<k)-1)> > k$

### 浮点数 IEEE754

$V = (-1)^s\times M\times 2^E$

- 32 位：k = 8, n = 23
- 64 位：k = 11, n = 52
- 规格化：$exp \ne 0$ 并且 $exp\ne1\dots1$
  - $E = e-Bias\quad Bias = 2^{k-1}-1$
  - $M = 1.f_{n-1}\dots f_0$
- 非规格化：$exp = 0$，表示 0 或接近 0 的数（逐渐下溢）
  - $E = 1-Bias$
  - $M = f$
- 特殊值：$exp = 1\dots 1$
  - $f = 0$时$$\begin{cases} +\infty,&s = 0\\ -\infty,&s = 1\end{cases}$$
  - $f\ne0$时 NaN（Not a Number）
- 将**正**浮点数按位解释成无符号整数时大小关系不变，**负**浮点数则关系反向
- 0 有两种表示
- `double` 可以精确表示 `int`；`float` 不行

#### 浮点数舍入规则

- 向偶数舍入（对于中间值）、向零舍入、向下舍入、向上舍入
- 向偶数舍入是默认行为，能够平衡处理一组数时带来的舍入误差
- 对$XX\cdots X.YY\cdots Y100\cdots0\cdots$的浮点数（表示中间值）才需要考虑偶数舍入

#### 浮点数运算规则

- $1/-0 = -\infty\quad 1/+0 =+\infty$，其他非法运算产生 NaN
- 加法：
  - 形成阿贝尔群（可交换）
  - **不可结合**（$(3.14+1e10)-1e10\ne3.14+(1e10-1e10)$）
  - 满足单调性（除 NaN）：$a\ge b \Rightarrow x+a\ge x+b$
  - 封闭的（可能产生无穷和 NaN）
- 乘法
  - 封闭
  - 可交换
  - 不可结合
  - 不具备分配性
  - 满足单调性（除 NaN）
- 无穷值可以用于比较
- NaN 与任何值都不相等
  - $NaN \neq NaN \to True$
  - $NaN \neq  x \to True$
  - $NaN [=,\ge ,>,\le ,<] x \to False$

## 程序的机器级表示

### ISA

指令集体系结构、指令集架构、Instruction Set Architecture

- 处理器状态
- 指令格式
- 每条指令对状态的影响

### 处理器状态

- 程序计数器，PC，`%rip`，下一条指令在内存中的地址
- 整数寄存器文件：16 个64位寄存器
- 条件码寄存器
- 向量寄存器：整数或浮点数

### 程序内存

- 可执行机器代码
- 操作系统所需信息
- 运行时栈
- 用户分配内存块
- 高 16 位为 0，能指定 $2^{48}$ 字节地址

### 数据格式

- 整数
  - b (byte): 1
  - w (word): 2
  - l (long): 4
  - q (quad): 8
  - o (oct): 16
- 浮点数
  - s (float): 4
  - l (double): 8

### 寄存器

> 未注明的为 caller-save

- rax, eax, ax, al：返回值
- rbx, ebx, bx, bl：callee-save
- rcx, ecx, cx, cl：第四个参数
- rdx, edx, dx, dl：第三个参数
- rsi, esi, si, sil：第二个参数
- rdi, edi, di, dil：第一个参数
- rbp, ebp, bp, bpl：callee-save
- rsp, esp, sp, spl：栈指针，特殊形式的 callee-save
- r8, r8d, r8w, r8b：第五个参数
- r9, r9d, r9w, r9b：第六个参数
- r10, r10d, r10w, r10b：
- r11, r11d, r11w, r11b：
- r12, r12d, r12w, r12b：callee-save
- r13, r13d, r13w, r13b：callee-save
- r14, r14d, r14w, r14b：callee-save
- r15, r15d, r15w, r15b：callee-save

### 条件码

- CF：进位标志(carry)，最高位进位，可检查有符号操作溢出
- ZF：零标志(zero)，操作结果为 0
- SF：符号标志(sign)，最近操作结果为负
- OF：溢出标志(overflow)，最近操作导致补码溢出
- leaq 不设置
- XOR: CF、OF 置零
- 移位：CF 为最后一个移出的位，OF 为零
- INC 和 DEC：会设置 OF、ZF，但不设置 CF

### 寻址

$Imm(r_b,r_i,s)$ 被计算为 $Imm + R[r_b]+R[r_i]\cdot s$

- Imm 为立即数
- s 只能为 1, 2, 4, 8
- 为数组访问提供了便利
- $r_i$不能为 rsp

### x86_64 指令

- 指令长度从 1 到 15 个字节，越常用越短
- 从某个位置开始可以唯一确定是哪条指令（即指令一定不是另外一个指令的前缀）
- 以 `.` 开头的行是指导汇编器和链接器工作的**伪指令**
- 1 字节或 2 字节指令会保持寄存器中其他字节不变，**4 字节指令则清零**（寄存器一共8字节）
- 立即数以 `$` 开头，汇编器会自动选择最紧凑方式编码
- nop (0x90)：对程序无影响，使函数代码对齐到 16 字节以优化性能
- rep：用来实现重复的字符串操作。rep ret 结合时 rep 视为空操作，避免 ret 成为跳转指令的目标，使得代码在 AMD 上运行更快（处理器设计缺陷）

#### 数据传输指令

- movb, movw, movl, movq S, D：S 和 D 都可为立即数、寄存器或者内存地址，但不能都为内存地址
- movabsq I, R：能以 64 位立即数为源操作数（movq 不能）
- `mov[s,z][b,w,l][w,l,q]`：拓展传送，没有 `movzlq`（被 movl 替代）
- cltq：对 rax 系列寄存器做符号拓展

#### 栈相关指令

- pushq
- popq

#### 算术和逻辑操作

除 leaq 都有 b, w, l, q 四种变种

- leaq S, D：D <- &S，用来加载地址或者做一些简单的运算，**不改变条件码**
- INC（加一），DEC（减一），NEG（取负），NOT（取反）
- ADD，SUB，IMUL(不区分有无符号，高位舍弃)，XOR(可用于清零），OR，AND：**没有除**
- SAL（左移），SHL（左移），SAR（算术右移），SHR（逻辑右移）k, D：位移量 k 为 **立即数** 或者`%cl`

#### 特殊算术操作

- imulq S: 有符号全乘法，$R [\%rax]\times S$, 结果的高 64 位放 rdx，低 64 位放 rax
- mulq S: 无符号全乘法（不舍弃高位时有无符号乘法不同）
- cqto: 对 rax 符号拓展，高位放 rbx
- idivq S: 有符号除法$R [\%rax]\div S$，商放 rax，余数放 rbx
- divq S: 无符号除法

#### 比较和测试操作

- cmp [bwlq]：相当于 SUB 但不改变寄存器
- test [bwlq]：相当于 AND 但不改变寄存器

#### 访问条件码

SET D：其中 D 为内存地址（改变一字节）或**低位单字节寄存器**

- set [ez]：ZF
- setn [ez]：~ZF
- sets：SF，负数
- setns: ~SF
- 有符号
  - setg, setnle：~(SF^OF)&~ZF，大于
  - setge, setnl：~(SF^OF)
  - setl, setnge：SF^OF
  - setle, setng：(SF^OF)|ZF
- 无符号
  - seta, setnbe：~CF&~ZF 大于
  - setae, setnb：~CF
  - setb, setnae：CF
  - setbe, setna：CF|ZF

#### 跳转指令

- jmp 以及 jxx（条件跳转）
- jmp
  - 直接跳转 jmp label：跳转目标作为指令的一部分编码
  - 间接跳转 `jmp * R` 或 `jmp * ADDRESS`：跳转目标从寄存器或内存中读出
- jxx（条件格式如 SET），只能直接跳转
- 直接跳转位置编码：PC 相对的，地址偏移量由目标地址减跳转指令**下一条指令地址**得出，可为 1, 2, 4 字节

#### 条件传送指令

CMOV S, R 系列，同 SET

- 源 S 可为**寄存器**或**内存**
- 目的 R 只能为**寄存器**
- 源和目的可以为 2, 4, 8 字，不支持单字节；不需显示指定长度编码，由汇编器推断
- 读源值，检查条件码，然后选择是否更新寄存器，比条件跳转效率高

#### 过程调用指令

- call Label：直接调用，将返回地址**压入栈帧**并跳转到目标地址
- call \*Operand：间接调用
- ret：将返回地址弹出栈帧并跳转

#### 条件分支

- 条件控制：条件跳转
- 条件传送：计算条件操作的两种结果，根据条件选择一个，**不能产生副作用**，效率更高

在条件传送可用时，编译器根据效率来选择使用哪种方式

### 循环

使用条件跳转实现

- do while：直接翻译
- while:
  - 跳转到中间：直接翻译，开始时直接跳转到测试的位置
  - guarded-do：初始条件不成立就退出循环，然后翻译为 do while
- for: init+while

### Switch

- 跳转表：分支情况多，值跨度小
  - 储存在.rodate（Read-Only Data）中
  - fall through
  - 重复情况则使用相同标号
- 否则翻译为条件分支

### 过程

#### 参数传递

- 六个及以内的参数通过寄存器传递
- 超过六个的部分通过栈传递，从后向前压入栈中（**后面的参数先访问**）
- 使用的寄存器大小根据参数大小决定
- 栈传参时，**所有数据大小向 8 倍数对齐**

#### 运行时栈

- 从高地址向低地址增长
- **栈帧**(stack frame)：过程需要的空间超过寄存器大小时在栈上分配的空间
  - 过程 P 调用过程 Q 时会把返回地址压入过程 P 的栈帧中
  - 当前执行的过程的栈帧总是在栈顶
  - 不需要栈帧情况：所有变量和参数可保存在寄存器且不调用其他函数
- 栈传参
- 栈上局部存储：通过减、加%rsp 来分配、释放

### 结构 Structure

**数据对齐**：

- 提高性能；某些处理器要求必须对齐
- 规则：任何 K 字节对象的地址必须是 K 的倍数
- 末尾也需要填充以保证连续的结构也满足对齐要求

### 联合 Union

Union 的总大小等于它最大字段的大小

### 攻击与防御

**攻击**：

- 缓冲区溢出：栈内存访问越界
- 空操作雪橇(nop sled)：插入很多 nop 来对抗栈随机化

**防御**：

- 栈随机化：栈的位置在程序每次运行都发生变化
  - Linux: 地址空间布局随机化 Address-Space Layout Randomization，程序的不同部分加载到内存的不同区域
- 栈破坏检测：栈保护者(stack protecter) / 金丝雀(canary) / 哨兵值(guard value)
- 限制可执行代码区域：处理器将读和执行模式分离（与操作系统配合）

### 变长栈帧

- GCC 扩展支持定义栈上的变长数组
- alloca 可以在栈上分配内存
- 使用 rbp(base pointer)来指向栈帧底来管理变长栈帧

### 浮点

- 所有能执行 x86_64 的处理器都能执行 SSE2 或更高
- 寄存器：MMX: MM(64), SSE: XMM(128), AVX: YMM(256)
- AVX: %ymm0~%ymm15，低 128 位为%xmm0~%xmm15，都是调用者保存
- %xmm0~%xmm7 传参，%xmm0 返回值
- AVX 浮点操作不能以立即数为操作数，必须从内存中读入

### Intel 与 ATT 格式区别

- Intel: Intel、Microsoft 文档
  - 省略指示大小的后缀
  - 省略寄存器前面%
  - 操作数顺序相反
  - `QWORD PTR [rbx]`
- ATT: GCC

## 处理器体系结构

### Y86_64

1. **寄存器**
   - callee-save 和 caller-save 规则与 x86_64 相同
   - ![image-20231103195132441](./image-20231103195132441.png)
2. **条件码**
   - ZF
   - SF
   - OF
3. **程序计数器**
4. **程序状态** ![image-20231103200644347](./image-20231103200644347.png)
5. **内存**

#### 指令集

- 只包括 8 字节整数操作，寻址方式简单
- 采用小端法编码
- 使用绝对寻址方式
- 算数指令只能操作寄存器
- **无二义性**：根据指令的第一个字节就能确定其他附加指令的含义和长度
- ![image-20231103195726159](./image-20231103195726159.png)
- ![image-20231103200033295](./image-20231103200033295.png)
- ![image-20231103200054449](./image-20231103200054449.png)
- 伪指令(assembler directives)：
  - .pos：指定从内存的何位置开始产生代码
  - .align：内存对齐
- pushq %rsp：压入%rsp 的原始值
- popq %rsp：将%rsp 置为从内存中读出的值

### RISV vs. CISV

| CISC                              | RISV                                     |
| --------------------------------- | ---------------------------------------- |
| 指令多                            | 指令少                                   |
| 有些指令延迟较长                  | 没有长延迟指令                           |
| 编码不定长                        | 编码定长                                 |
| 寻址方式多样                      | 寻址方式单一                             |
| 可对内存进行基本逻辑运算          | 只能操作寄存器                           |
| 机器级实现细节隐藏（ISA提供抽象） | 机器级实现可见（如禁止特殊的指令序列等） |
| 有条件码                          | 无条件码                                 |
| 栈过程密集                        | 寄存器过程密集（最多的有32个）           |

> Y86_64 兼有 RISV 和 CISV 特点

### HCL

- AND（&&），OR（||），NOT（!）![image-20231104232226748](./image-20231104232226748.png)
- 逻辑门输入：系统输入、某存储器单元输出、某逻辑门输出。多个逻辑门输出不能相连，必须无环。
- 逻辑门总是活动(active)，意味着 HCL 不是顺序求值
- 组合逻辑没有部分求值概念
- 字级信号都声明为 int，电路图中用中等粗度的线表示携带字的线路，虚线表示布尔信号的结果
- 情况表达式(case expression)：

  - ```hcl
    [
        select1: expr1;
        ...
        selectk: exprk;
    ]
    ```

  - 顺序求值，不要求互斥（实际的硬件多路复用器的信号必须互斥）

- 算术/逻辑单元（ALU）
  - 数据输入A，B；一个控制输入
  - B OP A
- 集合关系

  ```hcl
   iexpr in {iexpr1, iexpr2, ..., iexprk}
  ```

### 储存器和时钟

- 时钟寄存器（硬件寄存器，寄存器）
  - 直接将输入线和输出线连接到其他位置
  - 与程序寄存器相区分
  - 时钟上升沿加载输入信号
- 寄存器文件
  - 两读一写，随机访问
- 内存
  - 一地址输入，一写数据输入，一读数据输出
  - write控制信号0为读，1为写
  - 地址不合法error信号置1
- 只读存储器：用于读指令

### 顺序实现

- cmov Writeback中问题：if(Cnd)只需要指定内存的write端口的值就可以控制是否写入，不必多此一举
- OPq中valA和valB的顺序与ALU的端口顺序一样，都是为了与subq指令的顺序相对应（B=B-A）
- 很多操作有多种实现，但是最终选择的某种实现的原因大多是为了**简化电路**
- 写回阶段寄存器文件只接受valE或valM
- 访存阶段Addr只接受**valA**或**valE**
- 访存节点Data只接受**valP**或**valA**
- 硬件图中线的粗细
  - 中等粗度：宽度为字长
  - 细线：宽度为字节或更窄
  - 虚线：单个位

![image-20231103223242905](./image-20231103223242905.png)

![image-20231103223258952](./image-20231103223258952.png)

![image-20231104121522946](./image-20231104121522946.png)

### 流水线

- 提高系统吞吐量(throughput)，轻微增加延迟(latency)
- 运行时钟的速率由最慢阶段的延迟限制
- 流水线越深，边际收益率越低：因为流水线寄存器的延迟
- 需要处理数据相关和控制相关

#### SEQ+

使用状态寄存器保存指令执行过程中计算出的信号值，新时钟周期开始时再通过同样的逻辑计算出PC
![image-20231104125518258](./image-20231104125518258.png)

#### PIPE-

- 每个阶段前都有对应的流水线寄存器，这些寄存器中的信号名以**大写阶段名首字母**作为前缀
- 阶段中计算出来的信号以**小写阶段名首字母**作为前缀
- 合并valA和valP信号
- 分支预测策略：总是跳转

![image-20231104125955713](./image-20231104125955713.png)

#### PIPE

**处理冒险**(hazard)

- 暂停(stalling)：禁止流水线寄存器更新状态，即保持以前的状态
- 气泡(bubble)、取消：将流水线寄存器状态设置为某个固定的复位配置(reset configuration)，等效nop
- 触发条件 ![hazard_trigger](hazard_trigger.png)
- 同一个阶段不允许同时stall和bubble，且大多数控制条件互斥，除了以下两个
- 异常处理逻辑 ![image-20231104141323640](./image-20231104141323640.png)
- 组合A：分支预测错误和ret；不发生冲突 ![hazard_combination_A](2024-01-04-16-25-15.png)
- 组合B：加载使用冒险和ret；发生冲突 ![image-20231104141349640](./image-20231104141349640.png)
- 数据转发(data forwarding, by passing)，加载/转发冒险需要特殊考虑
- 转发优先级
  - 阶段越前优先级越高
  - 考虑`popq %rsp`即可得出同一阶段内转发的优先级顺序
    ![image-20231104141652853](./image-20231104141652853.png)

##### 异常处理

- 异常：halt指令、非法指令和功能码组合、访问非法地址
- 最深的指令引起的异常优先级最高
- 访存或写回阶段指令异常必须**禁止更新条件码或者数据内存**
- 如果某条指令被取消，那么关于这条指令的异常状态信息也都会被取消

![image-20231104140701037](./image-20231104140701037.png)

### 性能分析

#### 未使用的流水线周期

1. ret产生三个气泡
2. 加载/使用冒险产生一个
3. 分支预测错误产生两个

$$
CPI = \frac{C_i+C_b}{C_i}=1.0+\frac{C_b}{C_i}
$$

其中$C_i$为指令数，$C_b$为气泡数

## 优化程序性能

### 妨碍优化因素

- 内存别名使用
- 浮点运算局限
- 函数潜在副作用

### 衡量指标

每元素周期数CPE Cycle Per Element

### 优化方法

- 代码移动 code motion：提取不会变化的值，避免多次求值
- 减少过程调用
- 消除不必要的内存引用
- $n\times m$循环展开
- 分析关键路径
- 重新结合变换
- SIMD

### 理解现代处理器

- 指令级并行
- 超标量(superscalar)：每个时钟周期执行多个操作
- 乱序(out-of-order)：指令执行顺序不一定要与机器级程序中相同，只需要保证结果相同即可
- 指令控制单元(Instruction Control Unit ICU)和执行单元(Execution Unit EU)，执行单元功能互有重叠，能够并行执行
- 指令高速缓存(instruction cache)
- 分支预测(branch prediction)，投机执行(speculative execution)，退役(retire)，清空(flush)
- 指令译码
- 寄存器重命名
- 现代处理器架构 ![image-20231104194315140](./image-20231104194315140.png)

#### 功能单元性能

- 延迟 latency：完成运算所需总时间
- 发射时间 issue time：两个连续的同类型运算之间需要的最小时间周期数
- 容量 capacity：能够执行该运算的功能单元的数量
- 延迟界限：严格按顺序完成运算运行所需CPE
- 吞吐量界限：功能单元产生结果最大速率，CPE最小界限

## 存储器层次结构

DRAM和磁盘的性能滞后于CPU的性能

### 随机访问存储器(Random-Access Memory)

![image-20231104142755761](./image-20231104142755761.png)

DRAM存储器单元对干扰非常敏感，系统必须周期性读出重写来刷新每一位

#### 传统DRAM

![image-20231104143646151](./image-20231104143646151.png)

- 内存控制器(memory controller)依次发送RAS(Row Access Strobe)和CAS(Column Access Strobe)来访问第(i,j)个超单元![image-20231104143956900](./image-20231104143956900.png)
- 二维阵列：降低芯片上引脚数量，但增加了访问时间

#### 内存模块 memory module

![image-20231104144014177](./image-20231104144014177.png)多个DRAM芯片组织成内存模块同时读写

#### 增强DRAM

- 快页模式DRAM(Fast Page Mode, FPM DRAM)：行缓冲区可连续使用
- 扩展数据输出DRAM(Extended Data Out DRAM, EDO DRAM)：对FPM DRAM增强，允许CAS信号在时间上更紧密
- 同步DRAM(Synchronous DRAM, SDRAM)：比异步快
- 双倍数据速率同步DRAM(Double Data-Rate Synchronous DRAM, DDR SDRAM)：使用两个时钟信号沿使速率翻倍，DDR(2位)、DDR2(4位)、DDR3(8位)
- 视频RAM(Video RAM, VRAM)：允许并行读写，输出由内部缓冲区整个内容移位得到

#### 非易失性存储器 nonvolatile memory

只读存储器ROM Read-Only Memory

- PROM(Programmable ROM)：可编程ROM，用高电流熔断的方法编程一次
- EPROM(Erasable Programmable ROM)：可擦写可编程ROM，紫外线擦除，可擦1000次
- EEPROM(Electrically Erasable PROM, EEPROM)：电子可擦除ROM，可擦10^5次

闪存(flash memory)基于EEPROM，可用作固态硬盘(Solid State Disk, SSD)

固件(firmware)：存储在ROM中的程序，BIOS等

#### 总线 bus

先传地址再传数据

![image-20231104150244627](./image-20231104150244627.png)

### 磁盘

![image-20231104154349275](./image-20231104154349275.png)

- 盘片platter
- 表面surface：一盘片两表面
- 主轴spindle
- 旋转速率rotational rate，单位RPM，转每分钟(Revolution Per Minute)
- 磁道track：表面由一组称为磁道的同心圆组成
- 扇区sector：磁道划分为一组扇区，每个扇区包含相等数量的位
- 间隙gap：扇区间的分隔间隙，不存储数据位，存储用来标识扇区的格式化位
- 磁盘驱动器disk drive，磁盘disk，旋转磁盘rotating disk：一个或多个盘片叠加并封装
- 柱面cylinder：所有盘片上到主轴中心**距离相等**的磁道的集合

#### 磁盘容量

- 记录密度recording density 位/英寸：磁道一**英寸**的段中可以放入的位数
- 磁道密度track density 道/英寸：盘片中心出发半径上一英寸段上磁道数
- 面密度 areal density 位/平方英寸：$\text{记录密度}\times \text{磁道密度}$

#### 磁道组织

- 面密度很低时
  - 磁道分为数目相同的扇区
  - 扇区数由**最内扇区**决定
- 面密度高时
  - 多区记录 multiple zone recording
  - 每个区包含一组连续柱面
  - 区中每个柱面的每条磁道扇区数相同
  - 扇区数由**区中**最里面磁道能包含的扇区数决定

#### 读写操作

- 读/写头 read/write head， 在磁盘表面0.1微米处飞行，怕灰尘
- 传动臂 actuator arm
- 寻道 seek ：定位磁道

#### 访问时间

- 寻道时间 seek time
- 旋转时间 rotational latency：等待目标扇区转到读写头下
  - $T_{\text{max rotation}} = \frac{1}{\text{RPM}}\times\frac{60s}{1\text{min}}$
  - $T_{\text{avg rotation}} = \frac{1}{2} T_{\text{max rotation}}$
- 传送时间 transfer time
  $$
  T_{\text{avg transfer}} = \frac1{\text{RPM}}
  \times \frac1{\text{平均扇区数每磁道}}
  \times \frac{60s}{1min}
  $$

### 连接I/O设备

PCI peripheral Component Interconnect 外围设备连接

- USB Universal Serial Bus 通用串行总线
- 图形卡
- 主机总线适配器：连接一个或多个磁盘
  - SCSI：更快更贵
  - SATA
- 总线结构 ![image-20231104161249370](./image-20231104161249370.png)

#### CPU访问磁盘

- 内存映射I/O memory-mapped I/O：地址空间中一块地址为I/O通信保留
- 直接内存访问 Direct Memory Access DMA：设备不需CPU完成读写总线事务，直接将数据传到内存

### 固态硬盘

![image-20231104163541090](./image-20231104163541090.png)

- 页为读写单位
- 一页所属的块被整个擦除后才能写这一页，读不需要
- 擦除前要把块中有数据页复制到新块
- 闪存翻译层中平均磨损(wear leveling)通过平均擦除分布来最大化每个块寿命

### 单位

- 与DRAM和SRAM相关时：$K=2^{10}, M=2^{20}\dots$
- 与磁盘这样的I/O设备时：$K=10^{3}, M=10^{6}\dots$

### 高速缓存

#### 局部性

- 时间局部性 temporal locality
- 空间局部性 spatial locality

#### 存储器层次

![image-20231104164733016](./image-20231104164733016.png)

- 位于k层的存储作为k+1层存储的缓存
- 一般而言，k层块数比k+1层少，数据在相邻的层间复制时块大小一致
- 不同的层间块大小有差异，且一般越低层的块越大（补偿较长的访问时间）

#### 缓存命中

- 缓存命中 cache hit
- 缓存不命中 cache miss
  - 替换replacing，驱逐evicting，牺牲块victim block
  - 替换策略 replacement policy
  - 冷不命中 cold miss 强制性不命中 compulsory miss 冷缓存 cold cache 缓存暖身 warmed up
  - 放置策略 placement policy 任意 vs. 取模
  - 冲突不命中 conflict miss：多个对象映射到同一个缓存
  - 容量不命中 capacity miss：缓存太小不足以处理工作集 working set

#### 通用高速缓存器组织结构

![image-20231104170937796](./image-20231104170937796.png)

$t=m-(b+s)$

- 直接映射高速缓存 direct-mapped cache: E=1
  - 抖动thrash：反复访问两组映射到相同高速缓存组的数据造成的miss
- 组相连高速缓存 set associative cache: 1 < E < C/B
  - 最不常使用策略 Least-Frequently-Used LFU
  - 最近最少使用 Least-Recently-Used LRU
- 全相连高速缓存 fully associative cache: E = C/B
  - 昂贵，适合用于TLB，它缓存页表项

##### 写

- 写命中
  - 直写 write-through 简单但每次写都引起总线流量
  - 写回 write-back 替代时才写到低一层，维护一个额外的修改位
- 写不命中
  - 写分配 write-allocate 加载低一层块到高速缓存中然后更新，利用空间局部性
  - 非写分配 not-write-allocate 直接写到低一层
- 直写配非写分配，写回配写分配

#### 真实架构举例

![image-20231104172220959](./image-20231104172220959.png)

#### 评估性能

- 不命中率 miss rate
- 命中率 hit rate
- 命中时间 hit time
- 不命中处罚 miss penalty

## 链接

### 编译器驱动程序 compiler driver

- C预处理器(cpp)：将main.c翻译为ASCII码中间文件main.i，包括宏展开等过程
- C编译器(cc1)：将main.i翻译为ASCII码**汇编**语言文件main.s
- 汇编器(as)：将main.s翻译为**可重定位目标文件**main.o
- 链接器(ld)：将多个\*.o文件以及必要的系统目标文件组合起来创建一个**可执行目标文件**

### 静态链接

- 符号解析：将每个引用正好和一个符号**定义**关联起来
- 重定位：链接器使用汇编器产生的重定位条目将符号定义和内存位置相关联

### 目标文件

- 可重定位目标文件：包含二进制代码和数据，用于创建可执行目标文件
- 可执行目标文件：包含二进制代码和数据，可以直接复制到内存并执行
- 共享目标文件：可在加载或运行时被动态地加载进内存并链接

### 可重定位目标文件

我们关注现代Linux和Unix操作系统下的**可执行可链接格式**(Executable and Linkable Format, ELF)

按照顺序，一个ELF可重定位目标文件的格式如下

1. ELF头
   - 开头的16字节序列描述了系统的字大小和字节顺序
   - 剩下部分包含ELF头大小、目标文件类型、机器类型、节头部表的偏移等信息
2. .text：已编译的程序代码
3. .rodata：只读数据，如字符串常量和跳转表等
4. .data：已初始化的全局和静态C变量（不为0）
5. .bss：未初始化或初始化为0的全局和静态C变量
6. .symtab：符号表，存放在程序中定义和引用的函数和全局变量信息
7. .rel.text：.text节中位置的列表，链接时需要修改这些位置
8. .rel.data：引用或定义的全局变量的重定位信息
9. .debug：调试符号表，只有加`-g`时才生成
10. .line：行号表，只有加`-g`时才生成
11. .strtab：以null结尾的字符串序列，包括.symtab和.debug中的符号表，及节头部表中的节名字
12. 节头部表：包括不同节的大小和位置

### 符号和符号表

#### 链接中符号类型

- 由**模块m定义**并能被其他模块引用的全局符号：非静态的C函数和全局变量
- 由**其他模块定义**并能被模块m引用的全局符号：外部符号
- 只被模块m定义和引用的**局部**符号：静态C函数和全局变量(static关键字)

#### 符号表条目格式

```c
typedef struct {
  int name;      /* String table offset */
  char type:4,   /* Function or data (4 bits) */
  binding:4;     /* Local or global (4 bits) */
  char reserved; /* Unused */
  short section; /* Section header index */
  long value;    /* Section offset(relocatable) or absolute address(executable) */
  long size;     /* Object size in bytes */
} Elf64_Symbol;
```

其中，section可以是目标文件的某个节，也可以是伪节：

- ABS：不该被重定位的符号
- UNDEF：未定义的符号
- COMMON：还未被分配位置的未初始化的数据目标，value给出对齐要求，size给出最小的大小

COMMON vs. .bss vs. .data

- COMMON：未初始化的全局变量
- .bss：未初始化的静态变量，初始化为0的全局和静态变量
- .data：已初始化为非0值的全局和静态变量

### 符号解析

符号名：

- 全局变量：符号名等于变量名
- 静态局部变量：gcc中为`name.123`之类的名字
- 非静态局部变量：不需要符号名

#### 多重定义符号名规则

- 强符号：声明且定义
- 弱符号：声明且未定义

1. 不允许有多个同名的强符号
2. 如果有一个强符号和多个弱符号同名，选择强符号
3. 如果有多个弱符号同名，任意选择一个

> !!! 规则只限制了强弱符号，未限制符号所代表的变量的类型

#### 静态链接库

`*.a`文件成为存档(archieve)文件，是一组连接起来的可重定位目标文件的集合

链接时，必须根据依赖关系的拓扑排序，将文件在命令行中排序（一个文件可出现多次）

#### 重定位

- 重定位节和符号定义：链接器将所有相同类型的节合并为聚合节
- 重定位节中的符号引用：链接器修改代码节和数据节中对每个符号的引用

#### 重定位条目

存放在.rel.text和.rel.data中

```c
typedef struct {
  long offset;      /* Offset of the reference to relocate */
  long type:32,     /* Relocation type */
       symbol:32;   /* Symbol table index */
  long addend;      /* Constant part of relocation expression */
} Elf64_Rela;
```

基本的重定位类型

- `R_X86_64_PC32`：32位PC相对地址引用
- `R_X86_64_32`：32位绝对地址引用

### 可执行目标文件

![ELF_executable](2024-01-04-18-16-37.png)

- ELF头还包括了程序的入口点：执行的第一条指令位置
- .init节定义了\_init函数，程序的初始化代码会调用
- 段头部表包含权限信息、对齐信息、段的偏移、长度等

### 动态链接共享库

- 同一个共享库的.text节可以被不同的正在运行的程序共享
- 使用动态链接库的程序包含.interp节，这节中包括了动态链接器的路径名(`ld-linux.so`)
- 加载器加载动态链接程序时先加载和运行动态链接器
- 动态链接器重定位动态链接库的文本和数据段到某个内存段，然后重定位原程序中所有对动态链接库中符号的引用

### 位置无关代码 Position-Independent Code, PIC

每个目标模块有自己的GOT和PLT

#### PIC 数据引用

- 内存中加载的目标模块的数据段和代码段的距离总是保持不变
- 使用动态链接库的程序并不能直接引用动态链接库中的全局变量
- 编译器在数据段开始创建了全局偏移量表**（Global Offset Table, GOT）**
- 二进制代码中使用对GOT的PC相对引用
- 动态链接器在加载时会重定位GOT中的条目使其指向正确的绝对位置

#### PIC 函数调用

- GNU使用延迟绑定技术将过程地址的绑定推迟到第一次调用
- PIC 函数调用需要GOT和过程连接表**(Procedure Linkage Table, PLT)**共同发挥作用
- 程序代码在调用动态链接库中的函数时会跳转到PLT中对应的条目
- GOT
  - `GOT[0]`和`GOT[1]`是动态链接器用到的信息，`GOT[2]`是动态连接器入口点
  - 开始时每个条目指向对应PLT条目中的第二条指令
- PLT
  - PLT是一个数组，条目为16字节的代码
  - `PLT[0]`是特殊条目，跳转到连接器中；`PLT[1]`调用系统启动函数来初始化环境
  - `PLT[2]`开始的条目调用用户代码调用的函数
  - 条目的第一条指令跳转到GOT表条目中记录的地址
  - 条目的第二、三条指令调用动态链接器，动态链接器会修改GOT条目使其指向函数真正地址

### 库打桩

#### 编译时打桩

使用`-I.`参数添加比系统搜索库优先级更高的库

#### 链接时打桩

使用`--wrap f`标志告诉编译器将对符号`f`的引用解析成`__wrap_f`，并且把对符号`__real_f`的引用解析成`f`

#### 运行时打桩

使用`LD_PRELOAD`环境变量使动态链接器优先搜索指定库

## 异常控制流

### 异常

![exception_handler](exception_handler.png)

#### 异常表

- 异常处理需要软硬件高度结合
- 系统启动时会分配和初始化异常表的跳转表，使得条目$k$包含异常$k$的处理程序
- 异常表的起始地址放在**异常表基址寄存器**中

#### 异常类别

| 类别 | 原因             | 异步/同步 | 返回行为             |
| ---- | ---------------- | --------- | -------------------- |
| 中断 | 来自IO设备的信号 | 异步      | 总是返回到下一条指令 |
| 陷阱 | 有意的异常       | 同步      | 总是返回到下一条指令 |
| 故障 | 潜在的可恢复错误 | 同步      | 可能返回到当前指令   |
| 终止 | 不可恢复的错误   | 同步      | 不会返回             |

#### Linux/x86_64中的异常

1. 0~31 号是x86_64架构定义的异常
   - 0号除法错误（故障），不会恢复
   - 13号一般保护故障：引用未定义虚拟内存区域、写一个只读内存；一般报告为段错误
   - 14号缺页故障，可能恢复
2. 32~255 号是Linux系统定义的异常，是中断或者陷阱
   - Linux使用`syscall`指令来进行系统调用

#### 系统调用

- Linux中使用`syscall`指令进行系统调用
- 参数都是栈传递的，`%rax`是系统调用号
- `%rdi`, `%rsi`, `%rdx`, `%r10`, `%r8`, `%r9`包含最多6个参数
- 从系统调用返回时，`%rcx`和`%r11`会被破坏，`%rax`包含返回值
- 系统调用通过陷阱实现

### 进程

![process_address_space](process_address_space.png)

- 从下到上依次为：只读代码段、读/写段、运行时堆、共享库的内存映射区域、用户栈、内核虚拟内存
- 用户模式：不允许执行特权指令，不允许直接引用地址空间中内核区内代码和数据
- 从用户模式进入内核模式必须通过中断、故障或者陷阱这样的异常
- `/proc` 文件系统将许多内核数据结构的内容输出为一个用户程序可以读的文件，允许用户进程访问内核数据结构
- 内核中的调度器处理上下文切换；中断也可能引发上下文切换

### 系统调用错误处理

- 系统级调用函数遇到错误时通常会返回`-1`，并设置全局整数变量`errno`
- `stderror(errno)`返回和某个`errno`值有关的文本串

### 进程控制

#### 进程相关系统调用

- `pid_t getpid(void)` 返回当前进程的pid
- `pid_t getppid(void)` 返回父进程的pid
- `pid_t fork(void)` 创建子进程，调用一次，返回两次
- `pid_t waitpid(pid_t pid, int *statusp, int options)` 等待子进程返回
  - 返回值：-1若错误；0如WNOHANG且子进程未退出；子进程PID若成功
  - `pid=-1`则等待所有子进程；`pid>0`则等待特定子进程
  - options
    - `WNOHANG` 立即返回
    - `WUNTRACED` 同时返回被停止的子进程pid
    - `WCONTINUED` 同时返回被SICONT重新启动执行的子进程pid
  - 检查status
    - `WIFEXITED(status)` 通过return或exit正常终止
    - `WEXISTSTATUS(stauts)` 返回正常终止的退出状态
    - `WIFSIGNALED(status)` 子进程被未被捕获的信号终止
    - `WTERMSIG(status)` 信号编号
    - `WIFSTOPPED(status)` 引起返回的子进程当前是否是停止的
    - `WSTOPSIG(stauts)` 引起停止信号编号
    - `WIFCONTINUED(status)` 子进程受到SIGCONT重新启动
- `unsigned int sleep(unsigned int secs)` 挂起指定时间，返回剩下还要休眠的秒数
- `int execve(const char * filename, const char * argv[], const char *envp())`
  - 加载并运行可执行文件filename
  - 找不到filename时才返回到调用程序；若成功则调用一次从不返回
  - 按照惯例，`argv[0]` 是可执行目标文件的名字
  - envp为环境变量，指向以null结尾的指针数组，每个指针指向一个`name=value`的环境变量字符串
- `pid_t getpgrp(void)` 获取进程组
- `int setpgid(pid_t pid, pid_t pgid)` 设置进程组。pid是0则代表当前进程，gpid是0就用pid指定进程的PID作为进程组ID
- `int kill(pid_t pid, int sig)` 发送信号。0表示当前进程组，负数表示绝对值所在进程组
- `unsigned int alarm(unsigned int secs)` secs秒后发送信号。secs为0则不安排新的闹钟。
  - 对alarm的调用都会取消任何待处理的闹钟，并且返回任何待处理的闹钟返回前还剩下的秒数；没有则返回0

#### 进程状态

- 运行
- 停止：进程的执行被挂起(suspended)，如收到 SIGSTOP、SIGTSTP、SIGTTIN、SIGTTOU时，直到收到SIGCONT
- 终止：从主程序返回；调用`exit`；收到一个信号，其默认行为是终止

### 信号

![list_of_signals](list_of_signals.png)

- 信号不会排队，一种信号只有是否被接受到的状态，而不会记录接受次数
- 信号被阻塞时仍可以被发送，但不会被处理，直到进程取消阻塞
- 阻塞信号
  - 隐式阻塞：内核默认阻塞当前处理程序正在处理的信号类型
  - 显式阻塞 `sigprocmask`

#### 信号相关系统调用

- `sighandler_t signal(int signum, sighandler_t handler)` 修改和信号关联的信号处理程序
  - handler
    - SIG_IGN 忽略信号
    - SIG_DEF 恢复默认行为
    - `void handler(int)` 用户定义函数
  - 返回：成为则为旧的信号处理程序；失败则为`SIG_ERR`(不设置errno)
- `int sigprocmask(int how, const sigset_t *set, sigset_t *oldset)` 改变当前阻塞信号集合
  - how
    - SIG_BLOCK $\text{blocked} = \text{blocked} | \text{set}$
    - SIG_UNBLOCK
    - SIG_SETMASK
- `int sigemptyset(sigset_t *set)`
- `int sigfillset(sigset_t *set)`
- `int sigaddset(sigset_t *set, int signum)`
- `int sigdelset(sigset_t *set, int signum)`
- `int sigisnumber(const sigset_t *set, int signum)` 返回1若signum是set的成员

#### 编写信号处理程序注意事项

- 处理程序尽量简单
- 只调用异步信号安全的函数
  - 要么是可重入的
  - 要么不能被信号处理程序中断
  - printf, sprintf, malloc, exit, rand 都不是异步信号安全的
- 保护和恢复errno
- 访问共享的全局数据结构时阻塞所有信号
- 用volatile声明全局变量
- 用sig_atomic_t声明标志

#### 可移植的信号处理

```c
int sigaction(int sigsum, struct sigaction *act, struct sigaction *oldact);
```

Posix标准中定义的函数，被`Signal`函数包装

## 虚拟内存

### 物理和虚拟寻址

- 物理地址 Physical Address, PA
- 虚拟地址 Virtual Address, VA
- 地址翻译：将虚拟地址转换为物理地址
- 内存管理单元 Memory Management Unit, MMU：CPU上翻译虚拟地址的专用硬件

### 地址空间

- 主存中每字节都有一个选自虚拟地址空间的虚拟地址和一个选自物理地址空间的物理地址

### 虚拟内存作为缓存的工具

- **从概念上**，虚拟内存被组织为一个存放在磁盘上的N个连续的字节大小的单元组成的数组
- 这个磁盘上的数组的内存被缓存在主存中
- 磁盘上数据被分成块，作为数据的传输单元
- 虚拟内存的分割称为虚拟页Virtual Page, VP
- 物理内存的分割称为物理页Physical Page, PP；大小与VP一致；也称为页帧page frame

任意时刻，虚拟页面的集合被分为三个不相交的子集：未分配的、缓存的、未缓存的

#### DRAM 缓存的组织结构

- DRAM缓存在主存中缓存虚拟页
- 因为大的不命中处罚和访问首字节开销，所以虚拟页往往被设计得很大，为4kb到2gb
- DRAM缓存是全相连的
- 总是使用写回策略
- 替换算法更加复杂和精密

#### 页表 page table

- 存放在物理内存中
- 是 _页表条目 Page Table Entry, PTE_ 的数组
- 包含有效位和地址
  - 有效位为1：已缓存
  - 有效位为0，地址不为空：未缓存
  - 有效位为0，地址为空：未分配

### 虚拟内存作为内存管理的工具

- 操作系统为每个进程提供了一个独立的页表
- 多个虚拟页面可以映射到同一个物理页面上
- 连续的虚拟页面可以随机地分散在物理内存中

### 虚拟内存作为内存保护的工具

页表可以包含权限信息，如read, write, sup，如果指令违反权限，CPU触发一个一般保护故障（段错误）

### 地址翻译

- 基本参数
  - $N=2^n$ 虚拟地址空间中的地址数量
  - $M=2^m$ 物理地址空间中的地址数量
  - $P= 2^p$ 页的大小（字节）
- 虚拟地址 VA
  - VPO 虚拟页面的偏移量（字节）
  - VPN 虚拟页号
  - TLBI TLB索引
  - TLBT TLB标记
- 物理地址 PA
  - PPO 物理页面的偏移量（字节）
  - PPN 物理页号
  - CO 缓冲块内的字节偏移量
  - CI 高速缓存索引
  - CT 高速缓存标记

#### 地址映射

![address_translation](address_translation.png)

- 页表基址寄存器 Page Table Base Register, PTBR：指向当前页表

#### 快表 Translation Lookaside Buffer, TLB

- 小的，虚拟寻址的缓存，通常具有高度的相连度
- 每一行都保存一个由单个PTE组成的块
- 索引和标记：![tlb_addressing](tlb_addressing.png)

#### 多级页表

- 一级页表总是需要在主存中
- 虚拟内存系统在需要时在创建、页面调入、调出后面级的页表
- 多级页表映射结构：![klevel-page-table](klevel-page-table.png)

### Intel Core i7 Haswell 内存系统

- 支持48位(256TB)虚拟地址空间和52位(4PB)物理地址空间；兼容模式支持32位物理和虚拟地址空间
- 处理器封装 ![corei7_haswell_architecture](corei7_haswell_architecture.png)
- CR3控制寄存器指向第一级页表的起始位置，是进程上下文的一部分
- 地址翻译过程 ![corei7_address_translation](corei7_address_translation.png)
- 第一、二、三级页表条目格式 ![pte_format_level123](pte_format_level123.png)
- 第四级页表格式 ![pte_format_level4](pte_format_level4.png)
  - P=1时地址字段包含40位的PPN，这要求了物理页是4KB对齐的
- XD（禁止执行）位是64位系统中引入的用来禁止从某些内存页中取指令
- **每访问一个页时，MMU都会设置A位**，称为引用位，内核可以用这个位来实现页替换算法
- **每次对一个页进行了写之后，MMU都会设置D位**，称为修改位或脏位

### Linux 内存系统

- Linux中的虚拟内存
  - Linux将虚拟内存组织成一些区域（段）的集合
  - 区域就是已分配的虚拟内存的**连续片**（chunk）
  - 允许地址空间有间隙（不浪费）
  - **用于缺页处理程序**，在缺页处理程序中已经鉴权
- `vm_area_struct`数据结构
  - vm_end 区域结束
  - vm_start 区域开始
  - vm_prot 区域内所有页的读写许可权限
  - vm_flags 共享或私有
  - vm_next 指向下一结构（链表）
- 内存映射
  - 普通文件：如可执行文件
  - 匿名文件：由内核创建的二进制0
  - 使用匿名文件能够将内存访问模型统一
- Swap file
  - 匿名页只能交换到swapfile中
  - 物理内存紧缺的时候回收
  - kswapd负责在没法在低水位（物理内存剩余较多）的情况下分配内存时异步回收内存
  - 直接内存回收：alloc_page没有办法满足分配需求时尝试回收
  - swap-out：内存回收到磁盘
  - swap-in：磁盘读入到内存

### 内存分配

- 内部碎片：已分配块比有效载荷大
- 外部碎片：空闲内存合计起来满足一个分配请求，但是没有一个单独的空闲块足够大可以来处理这个请求

## 网络编程

![overview_of_network_app](overview_of_network_app.png)

### socket

```c
int socket(int domain, int type, int protocol);
```

- 创建一个套接字描述符
- domain
  - AF_INT 表示正在使用32位IP地址
- type
  - SOCKET_STREAM 表示套接字是连接的一个端点
- 最好使用`getaddrinfo`自动生成这些参数

### connect

```c
int connect(int clientfd, const struct sockaddr *addr, socklen_t addrlen);
```

- 尝试与套接字地址为addr的服务器建立互联网连接
- 会阻塞直到成功或错误

### bind

```c
int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```

- 告诉内核将`addr`中的服务器套接字地址和套接字描述符`sockfd`联系起来

### listen

```c
int listen(int sockfd, int backlog);
```

- 将`sockfd`从一个主动套接字转化为一个监听套接字
- `backlog`暗示了内存在开始拒绝连接请求前队列中要排队的未完成的连接请求的数量。可设为1024

### accept

```c
int accept(int listenfd, struct sockaddr *addr, int *addrlen);
```

- 等待来自客户端的连接请求并在`addr`中填写客户端的套接字地址，返回一个已连接描述符
- 监听描述符在服务器的生命周期中创建1次；已连接描述符每次接受请求创建一次

### getaddrinfo

```c
int getaddrinfo(const char *host, const char *service,
                const struct addrinfo *hints,
                struct addrinfo **result);
void freeaddrinfo(struct addrinfo *result);
```

- 将主机名、主机地址、服务名和端口号的字符串表示转化成套接字地址结构
- 返回后`result`指向一个`addrinfo`结构的链表。其中每个结构指向一个对应于host和service的套接字地址结构
- 客户端一般会遍历这个链表直到连接成功
- host可以是域名，可以是数字地址
- service可以是服务名（如http），也可以是端口号
- host和service至少指定一个

### getnameinfo

```c
int getnameinfo(const struct sockaddr *sa, socklen_t salen,
                char *host, size_t hostlen,
                char *service, size_t servlen, int flags);
```

- 将一个套接字地址结构转换成对应的主机和服务名字符串

## 并发编程

- 主线程 -- 对等线程
- 顺序程序、并发程序、并行程序
  - 并行程序 $\subset$ 并序

### 并发问题

#### 线程不安全函数

1. 不保护共享变量的函数
2. 保持跨越多个调用的状态的函数，如rand
3. 返回指向静态变量的指针的函数
4. 调用线程不安全函数的函数
