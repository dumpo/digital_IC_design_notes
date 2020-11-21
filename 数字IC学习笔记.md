# 数字IC学习笔记

数字IC前端秋招常见问题整理，整理自CSDN博客，《Verilog HDL数字系统设计教程》，《Verilog HDL高级数字设计》，《Verilog编程艺术》，《综合与时序分析的设计约束》，《数字电子技术基础》，《SOC设计方法与实现》，《计算机体系结构》等

公众号 IC加油站、摸鱼范式





## 流程flow 

### ASIC

```mermaid
graph TB
A[需求]-->B[芯片架构]
B-->C[rtl编写]
C-->D[功能仿真验证]
D-->E[综合与DFT]
E-->F[形式验证]
F-->G[STA静态时序分析]
G-->H[布局布线]
H-->I[时钟树综合与后端优化]
I-->J[设计规则检查 布线图原理图检查]
J-->K[生产GDSII Tapeout]

```

#### DFT

​	可测性设计，插入用于测试的硬件逻辑，提高芯片的可测性。主要包括内部扫描、内建测试和边界扫描三种。

#### 形式验证

​	保证综合后的网表功能与RTL一致。静态验证的一种。

#### 时序约束和分析

​	通过计算各通路的延迟，确保setup、hold、skew、clockfrequency等指标符合要求，并优化面积、功耗性能。包括STA静态时序分析DTA动态时序分析。

### FPGA

```mermaid
graph TD
A[系统规划]-->B[rtl设计]
B-->C[功能仿真<br>前仿 RTLLevel]
C-->D[综合 布局布线]
D-->E[时序仿真<br>后仿 GateLevel]
E-->F[板级验证]

```

### 工具使用

- 时序分析：PrimeTime
- Verilog/SystemVerilog编译仿真：VCS+Verdi/
- 形式验证 formality
- 综合工具：DC
- 各种工具的自动化：TCL脚本 
- Linux自动化：SHELL脚本
- 编译过程自动化：Makefile 
- 文本处理等：Python/Perl



---

## 数字电子技术

### 组合逻辑

- #### 最大最小项

- #### 转换为与非或非形式

- #### MUX实现异或

  ![mux实习异或](pics/20201101131622950.png)

- #### MUX实现and

  ![在这里插入图片描述](pics/20201101131646829.png)


### 时序逻辑

- ####   同步时序逻辑的门电路实现

  1.状态转移图、状态转换表
  2.状态化简
  3.状态分配
  4.求状态方程、驱动方程
  5.逻辑图

- #### 竞争和冒险的区别，成因，危害，处理方法

- #### 卡诺图化简与消除竞争冒险

- #### 推挽，OC门与OD门

  OC门或OD门只能输出低电平，高电平通过加上拉电阻完成，可以互相连接完成“线与”。
  推挽输出结构的低电平输出能力与OC门或OD门是一样的，但是高电平输出能力比OC门或OD门强很多，因为是直接上拉到了电源！因此推挽输出可以输出很高的电流。
  需要注意的是，配置为推挽输出的两个管脚，不能连一起，一个配置为输出高，另一个配置为输出低，会产生很大的电流，导致IO烧毁。
  ​

- #### 门电路实现latch，DFF

- #### 斯密特触发器

### CMOS

#### CMOS实现门电路

- ##### 反相器

![img](pics/反相器.png)

- ##### 与非<img src="pics/与非门.png" alt="img" style="zoom: 80%;" />

上并下串

- ##### 或非

##### <img src="pics/或非门.png" alt="img" style="zoom:80%;" />

上串下并

#### CMOS与TTL对比

- ##### 逻辑电平

#### <img src="pics/CMOS电平.png" alt="img" style="zoom: 67%;" /><img src="pics/TTL电平.png" alt="img" style="zoom: 67%;" />

 TTL的噪声容限更大

- ##### 功耗

![功耗对比](pics/功耗对比.png)

  TTL电路的功耗在工作频率的范围以内基本上是恒定 的。但是，CMOS电路的功耗则和频率有关。在静态（直流）条件下它的功耗是非常小的，并随着频率的增加而增加。这些特性如图12 . 7 中总的曲线所示。例如，一个低功耗肖特基（LS）TTL门电路的功耗是常量2.2 mW。一个 CMOS n电路的功耗在静态条件下是2.75uW,在频率为100kHz 时是170uW

   

- ##### 负载和扇出

  扇出是指在不会对门的工作特性造成不利影响的情况下，门所能够连接的负载门输人的最 大数目。例如，低功耗肖特基( L S ) T T L 电路的扇出是2 0 个单位负载。作为驱动电路的相同逻 辑系列的一个输入称为一个单位负载。

CMOS电路的负载不同于TTL电路 的负载，因为CMOS逻辑电路所使用的晶体管为驱动门提供了 一种占主导地位的电容性负载，限制条件是与驱动门输出电阻和负载门输人电容相关的充电和放电时间。当驱动门的输出 是高电平时，负载门的输人电容通过驱动门的输出电阻充电。当驱动门电路的输出是低电平 时，电容放电。如果给驱动门的输出添加更多的负载门输人，那么总的电容将会增加，因为输人电容实 际上是并行出现的。电容的增加会延长充电和放电时间，因此就会降低这个门电路的最大工 作频率。因此，C M O S  电路的扇出是和工作频率有关的。负载门输入越少，最大频率就 越高。

  TTL驱动门向处于高电平状态的负载门提供灌电流,从处于低电平状态的负载门吸收拉电流，当更多的负载门连接到驱动门时，驱动门电路上的负载就会增加。总的灌电流随着每增加 一个门输入而增加，随着电流的增加，驱动门内部的电压降也会增加，从而 引起输出电压V0 H 的降低。如果连接过多的负载门输人，V0 H 就会降到低于V0 H ( r a i n ) ,高电平噪声 容限也会降低，从而影响电路的正常工作。同样，随着灌电流的增加，驱动门的功耗也会增加。 
  总的拉电流也会随着每增加一个负载门输人而增加，当拉电流增加时， 驱动门电路内部的电压降也会增加，从而导致V0L
增加。如果添加了过多数的负载，那么将V0L会超过V0L(max),而低电平噪声容限会减小。在TTL电路中，拉电流的能力(低电平输出状态)是决定扇出数量的限制因素。

  

#### PMOS与NMOS，耗尽型与增强型的结构，区别

<img src="pics/MOS结构1.png" alt="img" style="zoom: 67%;" />
<img src="pics/MOS结构2.png" alt="img" style="zoom: 67%;" />

耗尽型可以工作在增强模式（VGS为正）或耗尽模式（VGS为负），增强型没物理沟道，衬底延伸到绝缘栅，只能工作在增强模式，低于阈值电压都是关断。

耗尽型与增强型的主要区别在于耗尽型MOS管在G端（Gate）不加电压时有导电沟道存在，而增强型MOS管只有在开启后，才会出现导电沟道；两者的控制方式也不一样，耗尽型MOS管的VGS（栅极电压）可以用正、零、负电压控制导通，而增强型MOS管必须使得VGS>VGS（th）（栅极阈值电压）才行。

这些特性使得耗尽型MOS管在实际应用中，当设备开机时可能会误触发MOS管，导致整机失效；不易被控制，使得其应用极少。因此，日常我们看到的NMOS、PMOS多为增强型MOS管；耗尽型用于功率器件。n型是电子导电，p型是空穴导电。电子迁移率高于空穴，因此PMOS由于存在导通电阻大、价格贵、替换种类少等问题，n型更常用。

### <span id="jump1">编码</span>

- ##### 格雷码与独热码

  格雷码：相邻之间只变1bit，编码密度高。功耗低；可用于CDC。状态机中可节省状态寄存器，适合写适合写条件不复杂但是状态多的状态机。；
  独热码：任何状态只有1bit为1，其余皆为0，编码密度低。但译码方便，节省组合逻辑；稳定性强，任意1bit错误都不会产生毛刺，适合写条件复杂但是状态少的状态机；

  （对于FPGA，可用资源数固定，资源足够就用独热码）


- ##### 二进制与格雷码的转换
  
  自然二进制码转换成二进制格雷码，其法则是保留自然二进制码的最高位作为格雷码的最高位，而次高位格雷码为二进制码的高位与次高位相异或，而格雷码其余各位与次高位的求法相类似。实际操作时将只需将二进制码右移一位再与原值异或就行。只需一行代码：
```verilog
  assign  graydata = (bindata >> 1) ^ bindata;
```

  保留格雷码的最高位作为自然二进制码的最高位，二进制码的次高位为格雷码的次高位与二进制码的(次高位+1)进行异或，其余各位采用类似的方法。代码如下：

  ```verilog
  assign {bindata[7],bindata[6:0]}={graydata[7],bindata[7:1]^bindata[6:0]}
  ```



## Verilog设计

### 语言细节

#### 连续赋值assign与always过程赋值

  - 在连续赋值语句中，表达式右侧的计算结果可以立即更新表达式的左侧。在理解上，相当于一个组合逻辑之后直接连了一条线，这个逻辑对应于表达式的右侧，而这条线就对应于wire。 

  - 在过程赋值语句中，表达式右侧的计算结果在某种条件的触发下放到一个变量当中，而这个变量可以声明成reg类型。根据触发条件的不同，过程赋值语句可以建模不同的硬件结构：如果这个条件是时钟的上升沿或下降沿，那么这个硬件模型就是一个触发器；如果这个条件是某一信号的高电平或低电平，那么这个硬件模型就是一个锁存器；如果这个条件是赋值语句右侧任意操作数的变化，那么这个硬件模型就是一个组合逻辑。

#### wire和reg类型的区别

  - ##### 基本概念的差别
    
    wire型数据常用来表示以assign关键字指定的组合逻辑信号，模块的输入输出端口类型都默认为wire型，wire相当于物理连线，默认初始值是z。
reg型表示的寄存器类型，用于always模块内被赋值的信号，必须定义为reg型，代表触发器，常用于时序逻辑电路，reg相当于存储单元，默认初始值是x。
    
  - ##### 在赋值语句中的差别
    
    在连续赋值语句中，表达式右侧的计算结果可以立即更新表达式的左侧。在理解上，相当于一个逻辑之后直接连了一条线，这个逻辑对应于表达式的右侧，而这条线就对应于wire。
    在过程赋值语句中，表达式右侧的计算结果在某种条件的触发下放到一个变量当中，而这个变量可以声明成reg类型。根据触发条件的不同，过程赋值语句可以建模不同的硬件结构：如果这个条件是时钟的上升沿或下降沿，那么这个硬件模型就是一个触发器；如果这个条件是某一信号的高电平或低电平，那么这个硬件模型就是一个锁存器；如果这个条件是赋值语句右侧任意操作数的变化，那么这个硬件模型就是一个组合逻辑。
总而言之，wire只能被assign连续赋值，reg只能在initial和always中赋值
    
  - ##### 端口信号和内部信号的差别
    
    信号可以分为端口信号和内部信号。出现在端口列表中的信号是端口信号，其它的信号为内部信号。
    对于端口信号，一旦定义位input或者output端口，默认就定义成了wire类型，输入端口只能是net类型（wire/tri）。输出端口可以是net类型，也可以是reg类型。若输出端口在过程块中赋值则为register类型；若在过程块外赋值(包括实例化语句），则为net类型。
    内部信号类型与输出端口相同，可以是net或reg类型。判断方法也与输出端口相同。若在过程块中赋值，则为reg类型；若在过程块外如assign赋值，则为net类型。
    若信号既需要在过程块中赋值，又需要在过程块外赋值。这种情况是有可能出现的，如决断信号。这时需要一个中间信号转换。
    inout是一个双向端口, inout端口不能声明为reg类型，只能是wire类型。

#### 阻塞赋值与非阻塞赋值的区别

-   非阻塞赋值在触发调节满足是，两条语句是同时进行的，一般时序逻辑用非阻塞赋值
-   阻塞赋值是顺序执行的，一般组合逻辑用阻塞赋值

#### founction与task的区别

|      |                             task                             |                          founction                           |
| :--: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| 定义 | 任务不能出现always语句；可以包含延时控制语句（#）,事件控制@等，但只能面向仿真，不能综合（可综合的任务只能实现组合逻辑） |      函数定义不能包含任何的时间控制语句，即#、@或者wait      |
| 包含 |              task可以包含其它的task和function。              |                     function不能包含task                     |
| 输入 |             task可以没有或者有多个任意类型的变量             |                  function至少有一个输入变量                  |
| 返回 | task则不返回值，也可以通过输出端口或双向端口返回一个或多个值 | function返回一个值，在函数的定义中，必须有一条赋值语句给函数中的一个内部变量赋以函数的结果值，该内部变量与函数具有相同的名字 |
| 调用 |            任务调用语句可以作为一条完整的语句出现            | 函数调用语句不能单独作为一条语句出现，只能作为赋值语句的右端操作数 |



#### localparam、parameter和define的区别 

|          | localparam | parameter    | define    |
| -------- | ---------- | ------------ | --------- |
| 作用域   | 当前module | 当前module   | 整个工程  |
| 参数传递 | 不可以     | 可           | 可        |
| 重定义   | 不可以     | defparameter | undef失效 |

注释：parameter如果在模块内部定义时视为localparam，无法进行参数传递，在模块名后写可以传递可以defparameter.

#### case casex casez的区别

- 语法
  - case语句的表达式的值有4中情况：0、1、z、x。4种是不同的，故表达式要严格的相等才可以操作分支语句。
  - casez语句中的表达式情况有三种：0、1、x。不用关心z，z可以和任何数值相等，即z =0.z= 1,z=x;
  - casex语句的表达式情况有二种：0、1.不用关心x和z。即x=z=0,x=z=1
- 综合
  -    都是可综合的
  - ​	case（不是casez/casex的时候）的index列表里面的x和z，都被综合工具认为是不可达到的状态就被去掉了
  - ​	casez和casex里面的x/z都被认为是don't care，所以综合出的电路会是一致的，因此这两个在综合来看没有孰优孰劣 

- 注意：要明确的是在case/casez/casex中'?'代表的不是don't care，而是'z'

#### 高阻态的意义和用法

- ​	该点输入电阻（输出电阻）无穷大，相当于断路，管角悬空，既不是高电平也不是低电平，其电平随外部电平高低而定。
- **应用实例1：**在总线连接的结构上。总线上挂有多个设备，设备与总线以高阻的形式连接。这样在设备不占用总线时自动释放总线，以方便其他设备获得总线的使用权。
- **应用实例2：**大部分单片机I/O使用时都可以设置为高阻输入，如STM32，AVR等等。高阻输入可以认为输入电阻是无穷大的，认为I/O对前级影响极小，电平随外部电平高低而定，除了高电平/低电平还能读中间的值，可用于AD连接。

#### 线与/线或

​	wand，wor，一条线上有多个同强度的驱动时，执行与/或

#### 系统函数

- clog2函数

  `$clog2`返回2为底的向上取整的对数，用于计算位宽。

-  `$signed`和`$unsigned`。首先明确这两个语句是可综合的。`$signed(c)`是一个function，将无符号数c转化为有符号数返回，不改变c的类型和内容。接上述代码历程：$unsigned同理。

  

#### 顺序块和并行块

- 顺序块（也称过程块）

  关键字begin_end

  顺序块中的语句一条条按顺序执行，只有前面语句执行完才执行后面的语句（除了带有内嵌延时控制的非阻塞赋值语句）。

  如果语句包括延时或事件控制，那么延迟总是相对于前面那条语句执行完成的仿真时间的

- 并行块（**<u>不可综合</u>**）

  关键字fork_join

  并行块内的语句是并发执行的；

  语句的执行的顺序是由各自语句内延迟或事件控制决定的

  语句中的延迟或事件控制是相对于块语句开始执行的时刻而言的。

- 可以嵌套

#### 命名块与生成块

- ​	命名块的定义：begin：name     
- 通过关键字disable提供了一种中止命名块执行的方法。disable可以用来从循环中退出、处理错误条件以及根据控制信号来控制某些代码是否被执行。对块语句的禁用导致紧接在块后面的那条语句被执行。disable则可以禁用设计中的任何一个命名块。
- generate可以循环生成，条件生成，case生成，综合时候只是按条件展开代码，可以简化代码提高可读性。

### 综合与时序分析

#### 如何防止综合出不想要的latch

1.  case if等判断条件写全
2.  always 敏感列表（只对仿真有影响，但是综合工具会自动补全敏感向量列表，所以在综合之后的电路中是不会有latch）
3.  不要出现自己给自己赋值的情况
4.  不要出现组合逻辑环路

#### 如何确保可综合

- 时序逻辑用非阻塞赋值，组合逻辑用阻塞赋值，同一个always块中既有时序逻辑又有组合逻辑时用非阻塞赋值，不要在同一个always块中混合使用，不要在两个及以上always块中对同一个变量赋值
- 所有内部寄存器都能复位，通过复位使信号初始状态可预测
- 不混合使用上升下降沿（可以考虑使用倍频时钟来设计）
- 不使用initial，不要使用延时，不使用循环次数不确定的循环语句
- 防止出现非目的性的Latch
- 不使用用户自定义原语（UDP元件）
- 用always过程块描述组合逻辑，应在敏感信号列表中列出所有的输入信号。
- 尽量使用同步方式设计电路。

#### 可综合语句的电路

- mux：case、if条件互斥，会综合出带优先结构的mux；若条件若不互斥，综合工具可以综合出并行的mux。（x会被当成don't care 按逻辑最小化确定为0或1）

- 组合逻辑：always@（\*），其中\*为自动补全敏感列表确保不综合出latch；无反馈的assign

- 锁存器：带反馈的assign，未补全条件的if、case；未补全敏感列表的always@（），且描述的行为是电平敏感的；

- 三态元件与总线接口：具有高阻态分支的assign 

  ```verilog
  reg [31:0] clk_to_bus
  assign data_to_bus=bus_enable?clk_to_bus:32'bz;
  //连接到总线的单向接口，enable时两者连接，否则高阻态断开
  ```

- 触发器：不完整的if、case、always且描述的行为是边缘敏感的。

  当一个变量在被<u>**内部行为**</u>（非外部行为）引用前就被周期性赋值，综合过程将消除该变量。

  边缘敏感行为赋值并在行为外用到该变量，综合为触发器的输出。

#### 不可综合语句

1. initial：只能在test bench中使用，不能综合。（我用ISE9.1综合时，有的简单的initial也可以综合，不知道为什么） 
2. events：event在同步test bench时更有用，不能综合。 
3. real：不支持real数据类型的综合。 
4. time：不支持time数据类型的综合。 
5. force 和release：不支持force和release的综合。 
6. assign 和deassign：不支持对reg 数据类型的assign或deassign进行综合，支持对wire数据类型的assign或deassign进行综合。 
7. fork join：不可综合，可以使用非块语句达到同样的效果。 
8. primitives：支持门级原语的综合，不支持非门级原语的综合。 
9. table：不支持UDP 和table的综合。 
10. 敏感列表里同时带有posedge和negedge：如：always @(posedge clk or negedgeclk) begin...end    这个always块不可综合。 
11. 同一个reg变量被多个always块驱动 
12. 延时：以#开头的延时不可综合成硬件电路延时，综合工具会忽略所有延时代码，但不会报错。 如：a=#10 b;    这里的#10是用于仿真时的延时，在综合的时候综合工具会忽略它。也就是说，在综合的时候上式等同于a=b; 
13. 与X、Z的比较：可能会有人喜欢在条件表达式中把数据和X(或Z)进行比较，殊不知这是不可综合的，综合工具同样会忽略。所以要确保信号只有两个状态：0或1。

#### Verilog的延迟模型

- ​	惯性延迟

  可检测的最小脉冲宽度，相当于滤波，宽度低于惯性延迟的将被忽略。原因是器件和连线的电容效应，使具有“惯性”，电平不会突变。

  ```verilog
  assign #5 wireIn=~wireOut //在持续赋值语句前使用延时，可以描述惯性延时
      (#5) a = b   //延迟5单位后赋值，一般用于TB，非描述惯性延时！
  ```

  

- ​	传播延迟

  ```verilog
  a = (#5) b; //描述组合逻辑传播延迟
  a <= (#5) b//描述FF的传播延迟
  ```

#### 综合命令

- dont_use

  部分单元库的单元，由于工艺、性能功耗面积等原因，工艺厂商建议或者后端建议，不要使用的单元。在综合步骤设置dont_use属性，就可以避免综合使用这些单元。

- dont_touch

  对目标模块，设置dont_touch属性。会使得该目标模块，不进行任何优化。
  比如已经综合完成的IP，在系统综合的时候，对IP模块设置dont_touch属性。不再对该IP进行任何综合优化，大大节省系统综合时间，避免不必要的结果检查。
  比如时钟网络，不关心时钟的负载，所以前端综合不要求加buffer。后端加时钟树时，才会把这个dont_touch属性去掉。

#### Verilog的时序检查

- ​	specify块

  specify block用来描述从源点（source：input/inout port）到终点（destination：output/inout port）的路径延时（path delay），由specify开始，到endspecify结束，specify block可以用来执行以下三个任务：

  - 描述横穿整个模块的各种路径及其延时。（module path delay）

  - 脉冲过滤限制。（pulse filtering limit）
  
  - 时序检查。（timing check）

​	specify block有一个专用的关键字specparam用来进行参数声明，用法和parameter一样，不同点是两者的作用域不同：specparam只能在specify block内部声明及使用，而parameter只能在specify block外部声明及使用。

- 模块路径声明：

   先描述模块，再把延迟赋值给路径。路径有三种：simple path，edge-sensitive path，State-dependent path。

  - 模块路径的要求
    1. 源信号(src)应该是模块的i叩ut或inoutport。 
    2. 源信号(src)可以是scalar和vector的任意组合。
    3. 目的信号(dst)应该是模块的output或inoutport。
    4. 目的信号(dst)可以是scalar和vector的任意组合。
    5. 目的信号(dst)应该只能被一个驱动源(Driver)驱动。

- 简单路径

  简单路径(Simple path)可以使用下面的两种连接类型。 

  1. src *> dst：用于 src 和 dst 之间的全连接(Full connection)。 
  2. src => dst：用于 src 和 dst 之间的并行连接(Parallel connection)。

  ```verilog
  module mux8 (ini, in2, s, q); 
      output [7:0] q; 
      input [7:0] ini, in2; 
      input s; // Functional description omitted ... 
      specify 
          (ini => q) = (3, 4); //parallel connection 
          (in2 => q) = (2, 3); //parallel connection 
          (s *> q) = 1;		//full connection
          endspecify 
  endmodule
  ```

  ![image-20201116192159528](pics/image-20201116192159528.png)

- 模块路径极性(Module path polarity):

  - 1. Unknown polarity：使用 *> 和=>, src rise 导致 dst rise> fall or no change, src fall 导致 dst rise、fall or no_change。 *
    2. Positive polarity：使用 +*> 和 +=>, src rise 导致 dst rise or no change» src fall 导致 dst fall or no change。 *
    3. Negative polarity：使用-*> 和-=>, src rise 导致 dst fall or no change, src fall 导致 dst rise or no change。

- 边沿敏感路径

  沿敏感路径(Edge-sensitive path)就是在描述模块路径时对src使用了沿转换(Edge transition),用于描述在src指定沿上发生的input-to-output延迟。 

  沿敏感路径的使用原则如下： 

  1.    沿(Edge)可以是 posedge 或 negedge，与 src—起使用。 
  2.    如果src是vector,那么就只检查最低位(LSB)的沿转换。 
  3.    如果没有指定沿转换，那么src的任意沿转换都会导致路径有效。 
  4.    沿敏感路径可以使用=> 和*>o 对于 =>,dst应该是scalar； 对于 *>, dst可以是scalar或vector。 
  5.    沿敏感路径可以指出datapath的关系： +: (not invert)» -: (invert), : (not specify)。

  ```verilog
  //I. module path posedge clock--> out: rise delay=10, fall delay=8. 
  // data path in ——>out: not invert 
  (posedge clock => (out +: in)) = (10, 8); //2. module path negedeg clock——> out: rise delay=10, fall delay=8. 
  // data path in -->out: invert 
  (negedge clock => (out -: in)) = (10, 8); 
  //3. module path any_edge clock--> out: rise delay=10, fall delay=8. 
  // data path in -->out: not specify 
  (clock => (out : in)) = (10, 8); 
  //4. module path posedge clock--> out: rise delay=10, fall delay=8. 
  // no data path 
  (posedge clock => out) = (10, 8);
  
  ```

- 状态依赖路径

  对于状态依赖路径(State-dependent path),只有在指定的条件为true时，对应的延迟才能起作用。 状态依赖路径包含三部分：条件表达式(Conditional expression)、模块路径描述、模块路径上的延迟。 

  状态依赖路径语法如下：

  ```verilog
  if (module_path_expression ) simple_path_declaration 
  if (module_path_expression ) edge_sensitive_path_declaration 
  ifnone simple_path_declaration 
  ```

  条件表达式使用如下规则。 

  1. 条件表达式可以使用模块的input或inout port＞它们的bit-select或part-select、模块内定义 线网或变量、常数、specparams。 
  2. 如果条件表达式的值是x或z，那么也认为是true。
  3. 如果条件表达式的值是multi-bit,那么只检查最低位(LSB)。 
  4. ifnone用于当所有条件为false时默认的状态依赖路径。 
  5. ifnone只能用于简单路径。

- 模块路径延迟

  模块路径赋值的原则如下：

  1. 左侧是模块路径描述，右侧是一个或多个延迟值。
  2. 延迟值可以放在一个括号内。 
  3. 延迟值可以是常数，也可以是specparamso 
  4. 延迟值可以是一个数值，也可以是一个表示(maxlyp:min)的三元组。 
  5. 对于路径延迟与信号转换的关系，可以指定1、2、3、6或12个延迟值。 

- 时序检查

  - 使用$开头的命令，<u>非系统命令</u>，系统命令不能出现再specify块内，时序检查命令不可在specify块外
  - 稳定时间窗口描述，检查事件发生的时间窗口 ，有＄setup、＄hold、＄setuphold、＄recovery、＄removal、＄recrem
  - 时钟和控制信号检查，检查时间两个事件的差值，有＄skew、＄timeskew、＄fullskew、 ＄width、$width、$period、$nochange



#### 可综合与不可综合的语句总结

- 所有综合工具都支持的结构 always，assign，begin，end，case，wire，tri，aupply0，supply1，reg，integer，default，for，function，and，nand，or，nor，xor，xnor，buf，not，bufif0，bufif1，notif0，notif1，if，inout，input，instantitation，module，negedge，posedge，operators，output，parameter 
- 所有综合工具都不支持的结构 time，defparam，$finish，fork，join，initial，delays，UDP，wait 
- 有些工具支持有些工具不支持的结构 casex，casez，wand，triand，wor，trior，real，disable，forever，arrays，memories，repeat，task，while。

#### 时序约束的分类和任务

- STA静态时序分析

  分析门级网表的拓扑结构，计算所有通路的传输延迟，生成有向无环图。但有可能检查了**无效路径**，从而生成错误虚假的违例警告。

- DTA动态时序分析

  基于电路的行为级、门级、开关级模型进行动态仿真。依赖于激励源，有可能漏掉关键路径，漏报时序违约。

- 两者对比

  |                              | STA          | DTA          |
  | ---------------------------- | ------------ | ------------ |
  | 方法                         | 仿真         | 路径分析     |
  | 激励源                       | 不需要       | 需要         |
  | 覆盖率                       | 与激励源无关 | 与激励源有关 |
  | 风险                         | 警告错误     | 丢失警告     |
  | **最大最小分析**，与综合配合 | 可           | 不可         |
  | 内存占用                     | 小           | 大           |
  | 运行时间                     | 快           | 慢           |

- 任务
  - 建立时间约束
  - 保持时间约束
  - 脉冲宽度约束
  - 时钟偏移（clock skew）约束
  - 时钟周期约束
  

#### 时序异常的约束

- 虚假路径	

  不需要满足任何时序要求的路径，EDA忽略该路径的时序

- 多周期路径

  需要多个周期来传输数据的路径，EDA放宽该路径的时序

- 最小延迟和最大延迟

  当有对最小延迟和最大延迟有特殊要求（与setup、hold约束的推测值不同）时指定

#### PTV

​	 PVT也称为Operating condition，分别是Process工艺，Voltage电压，Temperature温度，有下列组合

- WC：worst case slow，低电压，高温度，慢工艺 -> 一般情况下delay最大，setup 差。
- WCL：worst case low-temperature，低电压，低温度，慢工艺 -> <u>温度反转效应</u>时delay最大，setup差。
- LT：即low-temperature，也叫bc（best case fast），高电压，低温度，快工艺 -> 一般情况下delay最小，hold差。
- ML：max-leakage，高电压，高温度，快工艺 -> 温度反转效应下delay最小，hold差。
- TC：typical，也叫tt，普通电压，普通温度，标准工艺 -> 各种typical。
- BC：Best case。高电压，快工艺，常温0℃ or 25℃。
- 注：温度反转效应（Temperature Inversion Effect）
  - 工艺在90nm以上的时候，随着温度的升高，delay增大，所以worst corner是PVTmax，
  - 是65nm以下，随着温度的降低，delay增大，worst corner可能是PVTmax,也可能是PVTmin，这就是温度反转效应。
  - 温度对Transistors的影响：低温时，迁移率增大(导致快switching趋势)，但是Vt增大(导致慢switching趋势)，最后结果取决于迁移率和Vt谁起更重要作用（65nm以下制程主要是Vt起作用，90nm以上主要是电子迁移率）。
  - 温度对寄生参数的影响：低温时，wire电阻更低。

#### HVT，SVT，LVT Cell的区别

根据不同的阈值电压，工艺库里的Cell大致分为HVT，SVT/RVT，LVT。H高，L低，S中等（standard/Regular）。阈值电压低，速度快，但由于关断漏电导致功耗高。

#### OCV（On-chip-Variation）

​	片上变化，指同一时钟，一段按最快路径（clk->B）计算，一段按最慢路径(clk->A)计算，导致约束过于悲观。可声明公用部分，修正补偿延迟差异，称为“时钟网络悲观效应降低”

```verilog
                      +-----+     *****    +-----+      
                      |     +---** C1  *---+     |      
                      | F1  |     *****    | F2  |      
                      |     |              |     |      
                    +-+>A   |         |----+>B   |      
                    | +-----+         |    +-----+      
                    |                 |                 
                    .                 +                 
                   /_\               /_\                
                    |                 |                 
          clk       |                 |                 
          ----------+-----------------+               
```



#### 时钟激励的约束

#### setup time与hold time和什么有关，setup和hold裕度计算

#### 时序违例的修复

| 方案 | 作用 |
| ---- | ---- |
| 延长时钟周期 | 在性能指标约束内消除时序违例 |
| 调制关键路径 | 减少线网延迟 |
| 更换器件，调制器件尺寸 | 减少器件延迟，改善建立和保持裕度 |
| 时钟树重新设计 | 改善时钟偏移 |
| 更好算法、系统结构 | 减少通路延迟 |
| 改变工艺 | 减少器件和通路延迟 |

#### 门电路的内部延迟

​	tf，tr

​	nand的平衡

​	![img](pics/641.jpg)









