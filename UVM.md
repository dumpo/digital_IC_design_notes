### UVM

#### UVM验证平台

- driver 负责向 DUT 发送数据
- sequencer用于产生数据，一个 sequencer 通过启动一个 sequence，从 sequence 获取
  数据，并把这些数据转交给 driver。这种功能划分让 driver 不再关注数据的产生，而
  只负责数据的发送，职能更加清晰，更容易使用。
- monitor 主要用于监测 DUT 的输出
- In_agent和Out_agent是UVM中的agent，是简单的把driver，monitor 和 sequencer 封装在一起。通常来说， agent 对应的是物理接口协议，不同的接口协议对应不同的 agent，接口协议规定了数据的交换格式和方式， agent 通过 driver和 monitor 来实现接口协议的这些内容。在一个验证平台通常会有多个 agent，如上
  面所示的 In_agent，里面有 driver 和 monitor，用于向 DUT 发送数据，而 Out_agent
  中只有 monitor，用于监测 DUT 的输出。 
- env 则相当于是一个特大的容器，把所有的 uvm_component 都包含在其内部作为其成员变量。

<img src="C:\Users\APU\OneDrive\notes\pics\image-20210714081234837.png" alt="image-20210714081234837" style="zoom: 50%;" />

<img src="C:\Users\APU\OneDrive\notes\pics\image-20210714081339843.png" alt="image-20210714081339843" style="zoom:50%;" />

<img src="C:\Users\APU\OneDrive\notes\pics\image-20210715081353214.png" alt="image-20210715081353214" style="zoom: 50%;" />

#### 常用的 uvm_component

- uvm_driver：所有的 driver 都要派生自 uvm_driver。 driver 的功能主要就是向
  sequencer索要sequence_item(transaction)，并且把sequence_item里的信息驱动到DUT
  的接口上，这相当于完成了从 transaction 级别到 DUT 能够接受的 pin 级别的信息的
  转变。
- uvm_monitor： 所有的monitor都要派生自uvm_monitor。monitor做的事情与driver
  相反， driver 向 DUT 的 pin 上发送数据，而 monitor 则是从 DUT 的 pin 上接收数据，
  并且把接收到的数据转换成 transaction 级别的 sequence_item，并且把转换后的数据
  发送给 scoreboard，供 scoreboard 比较。
- uvm_sequencer：所有的 sequencer 都要派生自 uvm_sequencer。 sequencer 的功能
  就是组织管理 sequence，当 driver 要求数据时，它就把 sequence 生成的 sequence_item
  转发给 driver。
- uvm_scoreboard： 一般的 scoreboard 都要派生自 uvm_scoreboard。 scoreboard 的
  功能就是比较 reference model 和 monitor 分别发送来的数据，根据比较结果判断 DUT
  是否正确工作。
- reference model： reference model 直接派生自 uvm_component。 reference model
  的作用就是模仿 DUT，完成与 DUT 相同的功能。 DUT 是用 verilog 写成的时序电路，
  而 reference model 则可以直接使用 systemverilog 高级语言的特性，同时还可以通过
  DPI 等接口调用其它语言来完成与 DUT 相同的功能。
- uvm_agent：所有的 agent 要派生自 uvm_agent。与前面几个比起来， uvm_agent
  的作用并不是那么明显。它只是把 driver 和 monitor 封装在一起，根据参数值来决定
  是只实例化 monitor 还是要实例化 driver 和 monitor 呢？这个主要是从可重用性的角
  度来考虑的。如果在做验证平台的时候不考虑可重用性，那么 agent 其实是可有可无
  的。
- uvm_env： 所有的 env 要派生自 uvm_env。 env 是 environment 的缩写。 env 把验
  证平台上用到的固定不变的 component 都封装在一起。这样，当要跑不同的 case 时，
  只要在 case 中实例化一个 env 就可以了。
- uvm_test：所有的 case 要派生自 uvm_test。 case 与 case 之间差异很大，所以从
  uvm_test 派生出来的类各不相同。任何一个派生出的 case 中，都要实例化 env，只
  有这样，当 case 在运行的时候，才能把数据正常的发给 DUT，并正常的接收 DUT
  的数据。  



#### 常用的 uvm_object

- uvm_sequence_item：所有 的我们自己定义的 transaction 要从 uvm_sequence_item
  派生。 transaction 就是封装了一定信息的一个类，如一个 mac transaction 就是把一个
  mac 帧封装在了一起，包括目的地址，源地址，帧类型，帧的数据， FCS 校验和等。
  driver 从 sequencer 中得到 transaction，并且把其转换成 pin 级别的信号。 需要注意的
  是， UVM 中有一个 uvm_transaction 类，在以前的实现（OVM 的较老的版本）中，
  是可以直接从 uvm_transaction 来派生一个自己的 transaction，但是 在 UVM 中，是强
  烈不推荐从 uvm_transaction 派生一个 transaction，而要从 uvm_sequence_item 派生。
  事实上， uvm_sequence_itme 是从 uvm_transaction 派生而来的，因此， uvm_sequence_
  item 相比 uvm_transaction 添加了很多实用的成员变量和函数，从 uvm_sequence_item
  直接派生，就可以使用这些新增加的成员变量和函数。

- uvm_sequence：所有的 sequence 要从 uvm_sequence 派生一个。 sequence 就是
  sequence_item 的组合。 sequence 直接与 sequencer 打交道，当 driver 向 sequencer 索
  要数据的时候， sequencer 会转而向 sequence 要数据， sequence 发现有 sequence_item
  时，会把此 sequence_item 传递给 sequencer，并最终给 driver。

- config：所有 的 config 一般直接从 uvm_object 派生。 config 的主要功能就是规范
  验证平台的行为方式。如规定 CPU 的 driver 在读取总线时地址信号要持续几个时钟，
  片选信号从什么时候开始有效等等。

  除了上面几种类是派生自 uvm_object 外，还有下面几种：

- uvm_reg_item：它其实派生自 uvm_sequence_item。
  uvm_reg_map， uvm_mem， uvm_reg_field， uvm_reg， uvm_reg_file， uvm_reg_block
  等与寄存器相关的众多的类都是派生自 uvm_object。

- uvm_phase： 它派生自 uvm_object，其用处主要是控制 uvm_component 的行为
  方式，使得 uvm_component 平滑的在各个不同的 phase 之间依次运转。  

#### UVM机制

- factory 机制  

  实质上是重载了new函数。实例化时， UVM会通过factory机制在自己内部的一张表格中查看是否有相关的重载记录。 set_type_override_by_type语句相当于在factory机制的表格中加入了一条记录。 当查到有重载记录时， 会使用新的类型来替代旧的类型。  

- 根据一个字符串创建类实例

  定义一个类时，使用 uvm_component_utils 或 uvm_object_utils 宏来把它们注册 。

  提供一个称为 create_component_by_name 的函数，这个函数的输入是一个字符串，通过此函数，可以根据类名来创建一个类的实例。  

- 重载overrid

  1. 派生一个类，定义行为并注册到factory
  2. 使用set_type_override_by_type(my_driver::get_type(), new_driver::get_type())  

- field_automation 

  使用宏注册的字段是整数、 实数、 枚举等类型，具有打印、比较、pack、unpack，添加标志位等方法。

- transaction 

  transaction是一个抽象的概念。 一般来说， 物理协议中的数据交换都是以帧或者包为单位的， 通常在一帧或者一个包中要定义
  好各项参数， 每个包的大小不一样。 很少会有协议是以bit或者byte为单位来进行数据交换的。 以以太网为例， 每个包的大小至少
  是64byte。 这个包中要包括源地址、 目的地址、 包的类型、 整个包的CRC校验数据等。 transaction就是用于模拟这种实际情况， 一
  笔transaction就是一个包。 在不同的验证平台中， 会有不同的transaction。   

- sequence 机制  

- configdb 机制

  通过set、get参数在测试平台不同层次间传递参数。

  可使用set_config_int来代替uvm_config_db#(int):: set

  get_config_int来代替uvm_config_db#(int):: get

- register model  

- callback

- 平台运行

  - phase

    1. function phase

       如build_phase、 connect_phase等， 这些phase都不耗费仿真时间， 通过函数来实现.

       build_phase是自上而下执行的(一层层往下执行实例化  )

       除了build_phase之外， 所有不耗费仿真时间的phase（ 即function phase） 都是自下而上执行的  

    2. task phase  

       如run_phase等， 它们耗费仿真时间， 通过任务来实现。

       类似run_phase、 main_phase等task_phase也都是按照自下而上的顺序执行的。 但是与前面function phase自下而上执行不同的是， 这种task phase是耗费时间的， 所以它并不是等到“下面”的phase（ 如driver的run_phase） 执行完才执行“上面”的phase（ 如agent的run_phase） ， 而是将这些run_phase通过fork…join_none的形式全部启动。 所以， 更准确的说法是自下而上的启动， 同时在运行。  

       run_phase与12个小的动态运行（runtime——phase）并行

    对于同一层次的、 具有兄弟关系的component， 如driver与monitor， 执行顺序是按照字典序的。 这里的字典序的排序
    依据new时指定的名字 .

    

    <img src="C:\Users\APU\OneDrive\notes\pics\image-20210725012254268.png" alt="image-20210725012254268" style="zoom: 50%;" />

  - objection

    无论是run-time phase之间的同步， 还是run_phase与post_shutdown_phase之间的同步， 或者是run_phase与run_phase之间的同步， 它们都与objection机制密切相关  

    在进入到某一phase时， UVM会收集此phase提出的所有objection， 并且实时监测所有objection是否已经被撤销了， 当发现所有都已经撤销后， 那么就会关闭此phase， 开始进入下一个phase。 当所有的phase都执行完毕后， 就会调用$finish来将整个的验证平台关掉。  如果UVM发现此phase没有提起任何objection， 那么将会直接跳转到下一个phase中。  

    对于run_phase来说， 有两个选择可以使其中的代码运行： 第一是其他动态运行的phase中有objection被提起。 在这种情况下， 运行时间受其他动态运行phase中objection控制， run_phase只能被动地接受。 第二是在run_phase中raise_objection。 这种情况下运行时间完全受run_phase控制  

  - domain  

    同一domain所有对象的phase同步的。

    不同domain的phase可以不同步。