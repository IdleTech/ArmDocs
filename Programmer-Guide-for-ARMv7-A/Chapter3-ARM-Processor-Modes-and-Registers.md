# ARM处理器模式和寄存器

ARM体系结构是一种模型结构。在引入安全扩展之前，它具有七个处理器模式，如表3-1所示。六个特权模式和一个非特权的用户模式，特权模式可以执行一些不能在用户（非特权）模式下执行的任务。在用户模式下，限制执行影响整体系统配置的操作，比如MMU配置和缓存操作。模式与异常事件相关联，关于异常的内容会在第十一章异常处理中介绍。

<div align="center">表3-1 ARMv6以前的ARM处理器模式</div>

|模式 | 功能 | 特权 |
|----|------|-----|
|User (USR)| 大部分程序和应用运行的模式 | 非特权|
|FIQ | 进入快速中断异常 | 特权 |
|IRQ | 进入中断异常 | 特权 |
|Supervisor (SVC) | 进入复位或执行一条访管指令（SVC） | 特权 |
|Abort (ABT) | 进入内存访问异常 | 特权 |
|Undef (UND) | 进入未定义指令执行异常 | 特权 |
|System (SYS)| OS执行的模式，和用户模式共用寄存器 | 特权 |

TrustZone安全扩展（见图2-2）的引入为处理器创建了两个安全状态，独立于特权和处理器模式。使用一个新的监管者模式当作安全和非安全状态的切换途径，每一个安全状态中处理器模式都独立存在。

![图3-1 安全和非安全空间](\image\Figure3-1-Secure-and-Non-secure-worlds.jpg)

TrustZone安全扩展在第二十一章中进行描述，但是当前实现了TrustZone扩展的处理器中，系统安全通过将设备的所有硬件和软件资源分割开来实现，这样软件与硬件或者在安全子系统的安全空间，或者其他系统运行的普通空间（非安全）。当处理器处于非安全状态时，它不能访问分配给安全状态的内存块。

这样安全监视器就充当了两个空间之间切换的门卫。如果实现了安全扩展，在监控模式执行的软件控制安全空间和非安全空间的转换。

ARMv7-A架构虚拟扩展除了当前已存的模式之外，增加了超级管理模式（Hyp）。虚拟化可以让多个操作系统共存，在同一个硬件平台上同时运行。ARM虚拟化扩展因此使得在同一个平台上操作多个操作系统变得可能。

![图3-2 超级管理者模式](\image\Figure3-2-The-hypervisor.jpg)

如果处理器实现了虚拟化扩展，它就有了与之前架构不同的特权模型。在非安全状态下有三个特权级别，PL0，PL1和PL2。

**PL0**: 运行于用户模式的应用软件的特权级别。执行在用户模式的软件是非特权软件，它们无法访问体系结构中的一些特权功能。尤其明显的一点，这些软件不能修改处理器的配置设置。执行在PL0级别的软件仅仅可以进行非特权内存访问。

**PL1**: 非用户模式和非超级管理模式的软件运行在PL1级别。通常操作系统运行在PL1级别，PL1模式指除了用户模式和超级管理模式外其他的所有模式。操作系统可以运行在PL1下的所有模式中，OS的软件执行在PL0级别（即用户模式）。

**PL2**: 超级管理模式通常是超级管理员使用的模式，它控制着运行在PL1上的操作系统，可以实现在操作系统之间切换。如果实现了虚拟化扩展，超级管理员会执行在PL2级别（超级管理模式）。超级管理员可以控制和开启多操作系统共存，并且在同一个处理器系统上运行。

这些特权级别在TrustZone安全空间和普通（非安全的）空间之间是相互隔离的。

> 特权级别定义了在当前的安全状态下可以访问的资源权限，但是并没有实现任何访问其他安全状态下资源的权限。

图3-3给出了在不同的处理器状态下可用的处理器模式。

![图3-3 特权级别](\image\Figure3-3-Privilege-levels.jpg)

特定处理器和模式是否存在依赖于处理器是否实现相应的架构扩展，如表3-2所示。

| 模式 | 编码 | 功能 | 安全状态 | 特权级别 |
|------|-----|----------------|-----------|
|用户模式(USR)| 10000 | 大部分应用程序运行非特权模式 | Both | PL0 |
|FIQ | 10001 | 发生快速中断异常时进入的模式 | Both | PL1 |
|IRQ | 10010 | 发生中断异常时进入的模式 |  Both | PL1 |
|管理模式(SVC) | 10011 | 当处理器复位或执行访管调用时进入 | Both | PL1 |
|超级管理模式(MON) | 10110 | 实现安全扩展，参考第二十一章 | Secure only| PL1 |
|中止(ABT) | 10111 | 内存访问异常时进入 | Both | PL1 |
|Hyp (HYP) | 11010 | 实现虚拟化扩展，参考第二十二章 | Non-secure | PL2 |
|未定义(UND) | 11011 | 当执行未定义指令时进入 | Both | PL1 |
|系统(SYS) | 11111 | 特权模式，和用户模式共享寄存器 | Both | PL1 |

通用操作系统，比如Linux，和它的应用程序通常运行在非安全状态。安全状态通常是硬件供应商指定的固件运行的状态，或安全软件。在有些情况下，执行在安全状态的软件甚至比运行在非安全状态下的软件权限更高。

当前的处理器模式和执行状态包含在当前程序状态寄存器（CPSR）中。修改处理器状态和模式可以通过特权软件实现，或者发生异常时跳转到特定的部分执行。

###3.1 寄存器###

ARM架构提供了十六个32位通用寄存器（R0-R15）给软件执行使用。前十五个寄存器（R0-R14）可以用于通用目的的数据存储，R15作为程序计数器，当内核执行指令时会自动修改它的值。软件显示修改R15寄存器会改变程序执行流程。软件也可以访问CPSR，保存先前执行模式的CPSR值的寄存器被称为保存程序状态寄存器（SPSR）。

![图3-4 用户模式程序员可见寄存器](\image\Figure3-4-Programmer-visible-registers-for-user-code.jpg)

尽管软件可以访问这些寄存器，但是这也要依赖于软件执行所处的模式和访问的寄存器，同一个寄存器在不同的模式下可能对应不同的物理存储位置。这被称为备用寄存器，在图3-5中的阴影部分的寄存器是备用寄存器。他们使用不同的物理存储空间，对它们的访问只能在对应的模式下才可以实现。

![图3-5 ARM寄存器集](\image\Figure3-5-The-ARM-register-set.jpg)

在所有模式中，"低位寄存器"和R15使用相同的物理存储位置。图3-5给出了一些“高位寄存器”在不同模式备用。例如R8-R12在FIQ模式中具有备用版本，也就是说FIQ模式下访问R8-R12将访问不同于用户模式下R8-R12的物理存储位置。除了用户模式和系统模式，其他模式下R13和SPSR寄存器都是备用寄存器。

尽管有备用寄存器，软件通常并不需要指定要访问那个寄存器实例，这是由软件从那个模式下发起的访问来确定。例如执行在用户模式的程序指定访问R13寄存器会访问`R13_usr`，而执行在访管模式的程序指定访问R13会访问`R13_svc`寄存器。R13（在所有模式下）是操作系统栈指针，但是当不需要栈操作时它也可以用做通用寄存器。

当使用带有链接的转移指令（BL）调用子过程时，R14（链接寄存器）保存了返回地址。当它不需要支持从子过程中返回时，R14也可以用作通用寄存器。`R14_svc`，`R14_irq`，`R14_abt`，`R14_fiq`和`R14_und`用法类似，在中断或异常发生时用于保存R15中的返回地址，或在中断或异常过程中当转移链接指令执行时保存返回地址。

R15寄存器是程序计数器，保存了当前程序执行地址（实际上，它通常指向ARM状态下当前执行指令前面的八个字节的位置，Thum状态下的当前指令前的四个字节处，之所以是当前指令后第二条指令地址，其实是最初ARM1处理器三段流水线遗留下来的问题）。当在ARM状态下执行读取R15，位0-1总是0，位2-31包含了当前的PC值；在Thumb状态下，位0在读取时总是0。

R0-R14寄存器的值在系统复位后是未知值，SP是栈指针，每一种模式下，在使用栈之前必须让引导代码初始化。ARM架构过程调用标准（AAPCS）或ARM嵌入式ABI（AEABI）（参考第十五章应用程序二进制接口）详细描述了为了完成不同工具链或编程语言之间的互操作，软件应该如何使用通用寄存器。

#####3.1.1 超级管理模式#####

支持虚拟扩展的处理器具有一些额外在超管模式使用的寄存器。超管模式运行在特权级别PL2。它有自己版本的R13（SP）和SPSR寄存器。超管模式使用用户模式的链接寄存器保存函数返回地址，有一个专用的寄存器ELR_hyp用于存储异常返回地址。Hyp模式仅在普通空间可用，为虚拟化提供功能，并且虚拟化功能也只能在当前超管模式下可访问。参考第二十二章虚拟化中更多详细内容。

#####3.1.2 程序状态寄存器#####

程序执行的任何时刻都可以访问16个通用寄存器（R0-R15）和当前状态寄存器（CRSR）。在用户模式下，程序可以访问限制格式的CPSR，被称为应用程序状态寄存器（APSR）。

当前程序状态寄存器（CPSR）用于存储如下的内容：

* APSR标志
* 处理器当前模式
* 关中断标志
* 当前处理器状态，即ARM，Thumb，ThumbEE或Jazelle
* 字节序
* IT快的异常状态位

程序状态寄存器（PSR）形成了备用寄存器的一个额外集合。每一个异常模式都有自己的保存的程序状态寄存器（SPSR），当异常发生时异常前的CPSR的副本被保存到SPSR中。

应用程序员必须使用APSR访问CPSR寄存器的一部分内容，这部分内容可以在非特权模式下修改。APSR只能用于访问N，Z，C，V，Q和GE[3:0]位。这些位并不直接访问，它们一般是被条件代码设置指令置位，并且被带条件执行的指令检测。例如，`CMP R0, R1`指令对比R0和R1寄存器中的值，如果R0和R1相等则设置零标志位（Z）。

图3-6给出了CPSR的位图。

![图3-6 CPSR位图](\image\Figure3-6-CPSR-bits.jpg)

每一位所表示的含义如下：

* N ALU执行结果为负数
* Z ALU执行结果为0
* C ALU运算中出现进位或借位
* V ALU运算出现溢出
* Q 累计饱和运算（也称为粘连计算）
* J 指示是否内核处于Jazelle状态
* GE[3:0] SIMD指令使用
* IT[7:2] Thumb-2指令集的If-Then条件执行
* E 控制加载或存储指令的字节序
* A 关闭异步中止
* I 关闭IRQ
* F 关闭FIQ
* T 内核是否在Thumb状态执行
* M[4:0] 指定处理器的模式（FIQ，IRQ，如表3-1描述的内容）

内核使用指令直接写入CPSR模式位实现模式间的切换。异常事件发生会自动修改处理器的模式。在用户模式，程序不可以修改PSR中控制处理器模式的位[4:0]，同时也不可以修改控制异常启用或禁用的A，I和F位。在第五章和第十一章中会详细介绍这些位。

#####3.1.3 协处理器CP15#####

系统控制协处理器CP15控制内核的很多功能，它具有16个32位的主寄存器。对CP15寄存器的访问是有权限限制的，不是所有的寄存器都可以在用户模式访问。CP15的寄存器访问指令中指定要访问的主寄存器，指令的另外一部分可以做更精确定义，它可以增加CP15中可以访问的32位物理寄存器数量。CP15中的在16个主寄存器中命名为C0到C15，但是通常使用名字引用。例如CP15的系统控制寄存器被称为CP15.SCTLR。

表3-3总结了在Cortex-A系列处理器中使用的相关协处理器寄存器的功能。在介绍MMU和Cache的操作时会更详细介绍这些寄存器。

|       寄存器         |                 描述                        |
|---------------------|---------------------------------------------|
|主ID寄存器（MIDR） | 处理器的标识信息（包括部分号和修订号） |
|多核相关性寄存器（MPIDR）| 在一个内核簇中唯一标识单个内核的方法 |
|**CP15 C1系统控制寄存器** | |
|系统控制寄存器（SCTLR） | 主处理器控制寄存器（参考第三章系统控制寄存器（SCTLR））|
|辅助控制寄存器（ACTLR） | 由处理器实现中定义，实现中特定的额外控制和配置选项 |
|协处理器访问控制寄存器（CPACR） | 对CP14/CP15以外协处理器访问的控制 |
|安全配置寄存器（SCR） | TrustZone使用（第二十一章） |
|**CP15 C2和C3，内存保护和控制寄存器**|  |
|转换表基址寄存器0（TTBR0）| 第一层转换表的基地址（参考第九章）|
|转换表基址寄存器1（TTBR1）| 第一层转换表的基地址（参考第九章）|
|转换表基址控制寄存器（TTBCR）| 控制TTB0和TTB1的使用（参考第九章）|
|**CP15 C5和C6，内存系统错误寄存器** |  |
|数据错误状态寄存器（DFSR）| 最近一次数据错误的状态信息（参考11章） |
|指令错误状态寄存器（IFSR）| 最近一次指令错误的状态信息（参考11章） |
|数据错误地址寄存器（IFSR）| 最近一次引起明确的数据中止异常的数据访问的指令地址（参考11章） |
|指令错误地址寄存器（IFSR）| 最近一次引起明确的预取中止异常的数据访问的指令地址（参考11章） |
|**CP15 C7，缓存维护和其他功能** |  |
|Cache维护和转移预测器维护功能| 参考第八章 |
|数据和指令栅栏操作| 参考第十章 |
|**CP15 C8 TLB维护操作** |  |
|**CP15 C9 性能监控** |  |
|**CP15 C12 安全扩展寄存器** |  |
|向量基地址寄存器（VBAR）| 在监管模式不处理的异常的异常向量基地址 |
|监管向量基地址寄存器（MVBAR）| 放入监管模式的所有异常的异常基地址 |
|**CP15 C13 处理器，上下文和线程ID寄存器** |  |
|上下文ID寄存器（CONTEXTIDR）| 参考第九章ASID描述|
|软件线程ID寄存器 | 参考第九章ASID描述|
|**CP15 C15 由具体实现定义的寄存器** |  |
|配置基址寄存器（CBAR）| 为GIC和本地定时器类型外围组件提供基地址 |

所有系统架构功能都是通过从/向位于协处理器15的寄存器组（CRn）读写通用寄存器（Rt）来实现。指令的操作数Op1，Op2和CRm域也可以用于选择操作的寄存器，如下例3-1所示的格式。

<div align="center">例3-1 CP15指令语法</div>

```
MRC p15, Op1, Rt, CRn, CRm, Op2 ; 将CP15寄存器读到ARM寄存器中
MCR p15, Op1, Rt, CRn, CRm, Op2 ; 将ARM寄存器写入到CP15寄存器中
```

本节内容不会将CP15所有的寄存器逐一详细介绍一遍，这些内容可以从ARM架构参考文档或产品文档中读到。作为如何从CP15寄存器中读取信息的例子，这里简单介绍一下只读的主ID寄存器（MIDR），格式如下图3-7所示。

![图3-7 主ID寄存器格式](\image\Figure3-7-Main-ID-Register-format.jpg)

在特权模式下，可以使用如下的指令读取CP15的寄存器：

```
MRC p15, 0, R1, c0, c0, 0
```

读取的结果放在寄存器R1中，用于显示处理器正在运行。以ARM Cortex处理器的实现来解析结果如下：

* 位[31:24] 处理器制作人，如果是ARM设计处理器该值为0x41
* 位[23:20] 变体，显示处理器修订号
* 位[19:16] 架构号，ARM架构v7会是值0xF
* 位[15:4] 零件号，例如Cortex-A8处理器值为0xC08
* 位[3:0] 修订号，处理器的补丁修订号

#####3.1.4 系统控制寄存器（SCTLR）#####

SCTLR是CP15使用的另外一组寄存器，控制标准内存，系统功能，为内核中实现功能提供状态信息。系统控制寄存器只能在PL1特权级别或更高时访问。

![图3-8 系统控制寄存器的简化视图](\image\Figure3-8-Simplified-view-of-the-system-control-register.jpg)

每一位代表的意义如下所述：

* TE - 开启Thumb异常，这个位控制是否异常发生在ARM或Thumb状态
* NMFI - 非屏蔽FIQ（NMFI）支持，参考第十二章的外部中断请求
* EE - 异常字节序，这个位定义了在进入到异常时CPSR.E的值
* U - 表示对齐模型的使用，参考第十四章对齐
* FI - 开启FIQ配置，参考12章外部中断请求
* V - 这一位选择异常向量表的基地址，参考十一章的向量表
* I - 开启指令缓存的位，参考例子3-2和8章中的清理Cache内存和置无效
* Z - 开启转移预测的位，参考例子3-2和8章中的清理Cache内存和置无效
* C - 缓存开启位，参考例子3-2和8章中的清理Cache内存和置无效
* A - 开启对其校验的位，参考十四章对齐
* M - 开启MMU，参考第九章配置和开启MMU

一部分引导代码会设置系统控制寄存器CP15:SCTLR的Z位，它会开启程序转移预测。设置Z位的代码序列如下例3-2所示。

```
MRC p15, 0, r0, c1, c0, 0  ; 读取控制寄存器配置数据
ORR r0, r0, #(1 << 2)      ; Set C bit
ORR r0, r0, #(1 << 12)     ; Set I bit
ORR r0, r0, #(1 << 11)     ; Set Z bit
MCR p15, 0, r0, c1, c0, 0  ; 写回系统控制寄存器配置数据
```

类似形式的代码可以在例13-3中看到。



























By Andy @2018-05-15 22:41:12