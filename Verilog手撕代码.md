#### HDLBits重点复习



#### 同步与异步复位 

- 同步复位的优点大概有3条：
      a、有利于仿真器的仿真。
      b、有利于时序分析，而且综合出来的fmax一般较高。
      c、只有在时钟有效电平到来时才有效，所以可以滤除高于时钟频率的毛刺。
      缺点主要有以下几条：
      a、复位信号的有效时长必须大于时钟周期，才能真正被系统识别并完成复位任务。同时还要考虑，诸如：clk skew,组合逻辑路径延时,复位延时等因素。
      b、FPGA内寄存器中支持异步复位专用的端口CLR，所以，倘若采用同步复位的话，综合器就会在寄存器的数据输入端口插入组合逻辑，这样就会耗费较多的逻辑资源。
  
- 对于异步复位来说，他的优点也有三条：
      a、大多数目标器件库的dff都有异步复位端口，因此采用异步复位可以节省资源。
      b、设计相对简单。
      c、异步复位信号识别方便，而且可以很方便的使用FPGA的全局复位端口GSR。
      缺点： 
      a、在复位信号释放(release)的时候容易出现问题。具体就是说：倘若复位释放时恰恰在时钟有效沿附近，就很容易使寄存器输出出现亚稳态，从而导致亚稳态。
      b、复位信号容易受到毛刺的影响。
  
- 异步复位同步释放电路

  同步复位和异步复位都不可靠，将两者结合，取长补短,既解决了同步复位的资源消耗问题，也解决了异步复位的亚稳态问题。其根本思想，也是将异步信号同步化。

  ```verilog
  always @ (posedge clk)
      rst_n <= a_rst_n;                 //关键：异步复位信号用同步时钟打一拍(也可以多拍)
  
  always @ (posedge clk or negedge rst_n)
           if(!rst_n) b <= 1'b0;
           else b <= a;
  always @ (posedge clk or negedge rst_n)
           if(!rst_n) c <= 1'b0;
           else c <= b;     
  ```

  

#### 跨时钟域CDC

https://www.cnblogs.com/icparadigm/p/12794483.html 
https://www.cnblogs.com/icparadigm/p/12794422.html

- 亚稳态

  - 是什么

    时序逻辑在跳变时，由于异步信号、跨时钟域等原因，不满足setup或hold条件，输出在0和1之间产生振荡。

  - 原因

    D触发器的内部是一个主从锁存器(master-slave latch)，依靠背靠背的反相器锁存数据。

    ![image-20201115220814212](pics/image-20201115220814212.PNG)

    时钟为低电平时，主锁存器更新输入值，从锁存器保持上一个输出值不变。

    ![image-20201115221127230](pics/image-20201115221127230.PNG)

    时钟为高电平时，主锁存器保持上一个输出值不变，从锁存器更新输入。

    ![image-20201115221511788](pics/image-20201115221511788.PNG)

    由于反相器需要一定时间才能锁定，若时钟跳变前后，未完成锁存时钟就改变，最后输出的电平高低会不稳定，这就是亚稳态。

  - 危害

    错误的逻辑会一直传递下去导致系统错误。

  - 指标

    MTBF-- mean time between failure. 两次失效之间的平均时间。
    $$
    MTBF(T_{MET})=\frac{e^{T_{MET}}}{C_1} * \frac{1}{C_2*f_{clk}*f_{data}}
    $$
    C1 和C2 是常数，依赖于器件工艺和操作环境。

    fCLK 和fDATA 参数取决于设计规格：fCLK 是接收异步信号的时钟域的时钟频率，fDATA 是异步数据的翻转频率（toggling frequency）。

    TMET 参数是亚稳态转稳定的时间（Metastability setting time）,或者说时序裕量大于寄存器Tco可以让潜在的亚稳态信号达到稳定的值的时间。**TMET 对同步链来说就是链中每个寄存器输出时序裕量的和。**

  - 减少亚稳态的方法

    1. 改善工艺
    2. 降低时钟速率和数据翻转。
    3. 增大时序裕量（使用多级同步器打拍）

- 单bit情况

  - 慢时钟域到快时钟域（**目标时钟频率必须是源时钟频率1.5倍或者以上**）

    电平同步，直接打拍。

    ```verilog
    /*
               +---------+        +---------+       +---------+
    asynch_in  |         | meta1  |         | meta2 |         | synch_out
    +----------+ D     Q +--------+ D     Q +-------+ D     Q +-------+
               |         |        |         |       |         |
     clk_b     |         | clk_b  |         |  clk_b|         |
    +----------+ CLK     | +------+ CLK     | +-----+ CLK     |
               |    R    | |      |     R   | |     |         |
               +----+----+ |      +-----+---+ |     +----+----+
                    |      +            |     +          |
                    |                   |                |
                    |                   |                |
     +--------------+-------------------+----------------+
    
    */
    
    always @(posedge clk_b or posedge rst) begin
        if(rst) begin
            meta1<=0;
            meta2<=0;
            synch_out<=0;
        end
        else begin
            meta1<=asynch_in;
            meta2<=meta1;
            synch_out<=meta2;
        end
    end
    assign pos_out_b=synch_out&~meta2;//高电平跳变沿
    assign neg_out_b=~synch_out&meta2;//低电平跳变沿
    ```

    

  - 快时钟域到慢时钟域

    脉冲同步器，即加握手信号，通过组合逻辑把脉冲展宽为电平信号，再向clkb传递，当确认clkb已经“看见”信号同步过去之后，再清掉clka下的电平信号。在应答信号到来之前，不允许源信号改变,可能漏采。

    ```verilog
    module pluse_sync
    (
        input rst_n,
        input clk_a,
        input clk_b,
        input pulse_a_in,
        output pulse_b_out,
        output level_b_out
    );
        reg q;//展宽脉冲信号
        reg q1_a2b,q2_a2b,sync_out;//a向b同步信号
        reg q1_b2a,q2_b2a;//b向a同步信号
        
        //q的置位与清零
        always @(posedge clk_a or negedge rst_n) begin
            if(~rst_n)
                q<=0;
            else if(pulse_a_in)
                q<=1;
            else if(q2_b2a)
                q<=0;        
        end
        
        //
        always@ (posedge clk_b or negedge rst_n) begin
            if(~rst_n) begin
                q1_a2b<=0;
                q2_a2b<=0;
                sync_out<=0;
            end
            else begin
                q1_a2b<=q;
                q2_a2b<=q1_a2b;
                sync_out<=q2_a2b;
            end
        end
        
        //
        always@(posedge clk_a or negedge rst_n) begin
            if(~rst_n) begin
                q1_b2a<=0;
                q2_b2a<=0;
            end
            else begin
                q1_b2a<=sync_out;
                q2_b2a<=q1_b2a;
            end
            assign pulse_b_out=sync_out&(~q2_a2b);
            assign level_b_out=sync_out;
    endmodule
    ```

    

  - **在使用同步器同步信号时，要求输入信号必须是源时钟域的寄存输出**。即Asynch_in必须是clk_a的DFF信号，中间不能经过组合逻辑。原因：根据FF的特性，输出在一个时钟周期内是不会改变的，数据的变化频率不会超过时钟频率，这样就能降低跨时钟信号变化的频率，减小亚稳态发生的概率

  - 应用

    - 输入去抖debounce

      ```verilog
    //可以滤掉的宽度是两个clk的cycle，对于大于两个cycle而小于三个cycle的信号，有些可以滤掉，有些不能滤掉，这与signal_i相对clk的相位有关。
      parameter BIT_NUM  = 4 ;
    reg [BIT_NUM-1 : 0] signal_deb ; 
      always @ (posedge clk or negedge rst_n)
      begin
          if (rst_n == 1'b0)
              signal_deb <= {BIT_NUM{1'b0}} ;
          else
              signal_deb <= # DLY {signal_deb[BIT_NUM-2:0],signal_i} ;
      end
      
      always @ (posedge clk or negedge rst_n)
      begin
          if (rst_n == 1'b0)
              signal_o <= 1'b1 ;
          else if (signal_deb[3:1]==3'b111) 
              signal_o <= # DLY 1'b1 ;
          else if (signal_deb[3:1]==3'b000)
              signal_o <= # DLY 1'b0 ;
          else ;
      end
      ```
    
      根据希望滤除的宽度，换算到clk下是多少个cycle数，从而决定使用多少级DFF。
    
      如果希望滤除的宽度相对cycle数而言较大，可以先在clk下做一个计数器，产生固定间隔的脉冲，再在脉冲信号有效时使用多级DFF去抓signal_i；或者直接将clk分频后再使用。
    
      也不一定全为1或0才判断有效/无效，见project/uart_tx 输入去抖。
    
    - 无毛刺时钟切换 
    
      基本思路：根据clock gating，使用负沿触发的flip-flop，确保sel信号在低电平时候更新，同时用and gate确保在时钟高电平时切换。
      
      ![img](pics/641-1605522451350.jpg)
      
      问题：sel信号异步可能带来亚稳态，所有要加同步器。
      
      ![img](pics/640.jpg)
      
      新的问题：
      
      1.引入同步器，当SEL变化到en0发生变化需要2个CLK0周期，才能把CLK0停下来；
      
      2.同上的理由，clk停下后目标clock不是马上开始反转，存在gap；
      
      3.在负沿触发的flop之前只加了一级正沿触发的flop，这样留给flop输出稳定下来的时间只有**半个周期**，可能会使得MTBF达不到我们需要的值。
      
      4.综合时候后面的两个AND门和OR门要设为don‘t touch
    
    ```verilog
    //glitch free clock switch 
    module clk_switch (
                    rst_n          , //
                    clka            , //
                    clkb            , //
                    sel_clkb      , //
                    clk_o            //
                    );
    
    //assign clka_n = ~clka;
    //assign clkb_n = ~clkb;
    
    // part1
    //always @ (posedge clka_n or negedge rst_n)
    always @ (posedge clka or negedge rst_n)
    begin
        if (!rst_n) begin
            sel_clka_d0 <= 1'b0;
            sel_clka_d1 <= 1'b0;
        end
        else begin
            sel_clka_d0 <= (~sel_clkb) & (~sel_clkb_dly3) ;
            sel_clka_d1 <= sel_clka_d0 ;
        end
    end
    
    // part2
    //always @ (posedge clka_n or negedge rst_n)
    always @ (posedge clka or negedge rst_n)
    begin
        if (!rst_n) begin
            sel_clka_dly1 <= 1'b0;
            sel_clka_dly2 <= 1'b0;
            sel_clka_dly3 <= 1'b0;
        end
        else begin
            sel_clka_dly1 <= sel_clka_d1;
            sel_clka_dly2 <= sel_clka_dly1 ;
            sel_clka_dly3 <= sel_clka_dly2 ;
        end
    end
    
    // part3
    //always @ (posedge clkb_n or negedge rst_n)
    always @ (posedge clkb or negedge rst_n)
    begin
        if (!rst_n) begin
            sel_clkb_d0 <= 1'b0;
            sel_clkb_d1 <= 1'b0;
        end
        else begin
            sel_clkb_d0 <= sel_clkb & (~sel_clka_dly3) ;
            sel_clkb_d1 <= sel_clkb_d0 ;
        end
    end
    
    // part4
    //always @ (posedge clkb_n or negedge rst_n)
    always @ (posedge clkb or negedge rst_n)
    begin
        if (!rst_n) begin
            sel_clkb_dly1 <= 1'b0;
            sel_clkb_dly2 <= 1'b0;
            sel_clkb_dly3 <= 1'b0;
        end
        else begin
            sel_clkb_dly1 <= sel_clkb_d1   ;
            sel_clkb_dly2 <= sel_clkb_dly1 ;
            sel_clkb_dly3 <= sel_clkb_dly2 ;
        end
    end
    
    // part5
    clk_gate_xxx clk_gate_a ( .CP(clka), .EN(sel_clka_dly1), .Q(clka_g)  .TE(1'b0) );
    clk_gate_xxx clk_gate_b ( .CP(clkb), .EN(sel_clkb_dly1), .Q(clkb_g)  .TE(1'b0) );
    //若不使用clock gating cell，part2 part4可不要
    //assign clka_g = clka & sel_clka_dly1 ;
    //assign clkb_g = clkb & sel_clkb_dly1 ;
    assign clk_o = clka_g | clkb_g ;
    
    endmodule
    
    ```
    
    ​	如果不用负沿触发的flop，可将最后的and门替换为门控时钟cell，同时en0之后再加一级flop，将延一拍之后的en0b_dly反馈给另外一路，这样才能保证在当前路**完全关断**的情况下切换.(也可以额外定义反向时钟)
    
    ![img](pics/642.jpg)

- 多bit情况

  - 多个信号合并

    如果可能，将多个信号合为一个传递。

  - 多周期路径法

    （常见于单bit同步，多bit一般用AFIFO）

  - 使用格雷码传递多个CDC位

    格雷码最常见的应用是在异步FIFO中,相邻的状态只变化一位，转化为单bit情况。

    **格雷码必须是计数到2^n才是每次改变一个bit。**如果计数器是从0~5计数，那么从5->0的计数，不止一个bit改变，就失去了只改变一个bit的初衷。所以就算浪费面积，也需要把FIFO深度设置为2^N。

  - 使用异步FIFO来传递多位信号

- Valid-Ready握手协议

  

#### 有限状态机FSM

- 分类

  - 一段式

    所有逻辑写在一个always模块，不建议

  - 二段式

    一个always描述组合逻辑（状态转移逻辑、状态输出），一个描述时序逻辑（各状态的行为）

  - 三段式

    二段式的基础上，把状态转移和状态输出分离开，易于修改，推荐

  - Moore型

  - Mealy型

- 状态编码的选择

  格雷码：可节省状态寄存器，适合写适合写条件不复杂但是状态多的状态机。
  独热码：节省组合逻辑，稳定性强，适合写条件复杂但是状态少的状态机。

  （对于FPGA，可用资源数固定，资源足够就用独热码）

- 序列检测器0010

- 饮料售卖机

- 回文序列检测

#### FIFO

- 异步FIFO深度计算（背靠背）

- 同步

  - 深度为1

  - FF实现

  - SRAM实现

  - 乒乓buffer

    在读写直接需要等待时候使用，实质为流水线技术，用面积换速度，读写可以同时进行。
    
    典型应用：解决深度流水线中的时序反压问题，将长的握手链切割。

- 异步

#### 分频计数器

- 奇数

- 偶数

- 小数

#### 数字计算

- 超前进位加法器
- booth乘法器
- wallace乘法器
- Wallace树
- 除法器


#### 其他

- 边缘检测

  ```verilog
  module top_module (
      input clk,
      input [7:0] in,
      output [7:0] pedge
  );
      reg [7:0] temp;
      always@(posedge clk)begin
          temp<=in;
          pos_edge<=(~t&in);
          neg_edge<=(t&~in);
          both_edge<=pos_edge|neg_edge;
          //both_edge<=t^in;
      end
  endmodule
  ```

- 双边缘触发器

  ```verilog
  //法一：对结果进行寄存
  module top_module (
      input clk,
      input d,
      output q
  );
      reg q1,q2;
      assign q=clk?q1:q2;
      always @(posedge clk) q1<=d;
      always @(negedge clk) q2<=d;
  endmodule
  
  //法2：时钟倍频
  //法3；对时钟进行寄存
  ```

- 优先编码器

  - casez  ？

- 线性反馈移位寄存器（伪随机码发生器）

- 奇偶校验

  ```verilog
  function cal_parity_odd;
  input [31:0]address;
  begin
    cal_parity=^address;//缩减异或，1的个数为偶数结果为0，奇数为1
  end
  
      
  function cal_parity_even;
  input [31:0]address;
  begin
    cal_parity=^address;//缩减异或，1的个数为偶数结果为1，奇数为0
  end
  ```

  

- 串行-并行CRC

  - CRC由一称为生成多项式的常数去除该数据流的二进制数值而得，商数被放弃，余数作为冗余编码追加到数据流尾，产生新的数据流进行发送。在接收端，新的数据流被同一常数去除，检查余数是否为零。如果余数为零，就认为传输正确，否则就认为传输中己发生差错，该数据流重发。不同的生成多项式有不同的检错能力，为了得到优化的结果，必须根据需要选择合适 的生成多项式。

    | 名称         | 生成多项式                           | 简记式   | 应用                                       |
    | ------------ | ------------------------------------ | -------- | ------------------------------------------ |
    | CRC-4        | $X^4+X+1$                            | 3        |                                            |
    | CRC-8        | $X^8+X^5+X^4+1$                      | 0x31     |                                            |
    | CRC-8        | $X^8+X^2+X^1+1$                      | 0x07     |                                            |
    | CRC-8        | $X^8+X^6+X^4+X^3+X^2+X^1$            | 0x5E     |                                            |
    | CRC-12       | $X^{12}+X^{11}+X^3+X+1$              | 80F      |                                            |
    | CRC-16       | $X^{16}+X^{15}+X^2+1$                | 8005     | IBM SDLC                                   |
    | CRC-16-CCITT | $X^{16}+X^{12}+X^5+1$                | 1021     |                                            |
    | CRC-32       | $X^{32}+X^{26}+X^{23}+...+X^2+X+1$   | 04C1DB7  | ZIP，RAR，**IEEE 802 LAN/FDDI**，IEEE 1394 |
    | CRC-32C      | $X^{32}+X^{28}+X^{27}+...+X^8+X^6+1$ | 1EDC6F41 | SCTP                                       |

  - 计算过程：

    1. 将数据乘以$X^n$，n为生成多项式的最高次系数（CRC-n即为n），直接把数据左移n位。
    2. 左移后的数据，使用模2除法除以生成多项式，计算余数。
    3. 余数附加到原始数据后,发送。
    4. 接收端，传输信息的前一部分为原始数据流D;后一部分(最后n位数)为余数R。整个 数据流多项式被同一生成多项式G去除，商数被丢弃，余数应为0。如果余数不为0,说明传输数 据时发生错误，数据需要重传。

  - **串行CRC**:通常，CRC校验码的值可以通过线性移位寄存器和异或门求得，线性移位寄存器一次右移一 位，完成除法功能，异或门完成不带进位的减法功能。如果G(X)系数为’1’，则从被除数的高阶位减去除数，同时移位寄存器右移一位，准备为被除数的较低位进行运算。如果商数为’0’，则移位寄存器直接右移一位。

  - ![image-20201124231457190](pics/image-20201124231457190.png)

    ```verilog
    //CRC8 G(X)=X^16+X^12+X^5+1
    module CRC16_SER
        ( input Reset , //Reset signal 
          input Gclk ，//Clock signal 
          input Soc ， //Start of cell Data 
          input in，//input data of cell Crc 
          output reg [15:0] out //output CRC signal 
        )；
    	reg temp;
        integer i,j,k,l;
        always @(posedge Reset or posedge Gclk) 
            begin 
                if (Reset) 
                    Crc_out <= 16’b0;
                else if (Soc = l’bl) 
                    Crc_out <=16’b0 ;
                else begin 
                    Temp = Data_in ^ Crc_out[15]; 
                    
                    for (j=15;j>12;j=j-l) 
                        Crc_out[j] <= Crc_out[j-1]; 
                    Crc_out[12] <=Temp ^ Crc_out[ll]; 
                    
                    for (k=11;k>5;k=k-1) 
                        Crc_out[k] <=  Crc_out[k-1]; 
                    Crc_out[5] <=  Temp ^ Crc_out[4];
                    
                    for (l=4;l>0;l=l-1)
                        Crc_out[l] <= Crc_out[l-1];
                    Crc_out[0] <= Temp
                end
            end            
    endmodule
        
        
    ```

    

  - **并行CRC**:并行CRC校验码产生器 16位CRC同时输出，所以要求在一个时钟周期内，移位寄存器一次需要移16位。实际上，移位寄 存器不可能在一个时钟周期内移16位，所以这部分电路是用**组合逻辑来完成**。整个CRC校验码产生器由组合逻辑和16个输出寄存器组成。

    

- 独热码检测

  ```verilog
  function automatic logic is_onehot(input [WIDTH-1:0] sig);
    localparam SUM_WIDHT = $clog2(WIDTH) + 1;
    logic [SUM_WIDTH-1:0] sum;
    sum = '0;
    for(int i = 0; i < WIDHT; i++)
       sum = sum + sig[i];
    is_onehot = (sum == 1);
  endfunction
  ```
  

  
- 串并转换

  





- ### clock gating

  - 用AND门有一个显而易见的缺陷，由于当EN的变化会立刻反映到输出端，AND门输出会产生glitch。

    因此，**只需要控制EN到达AND门的时间，使得EN只在CLK为低的时候变化**

  ```verilog
  always @ (clk_in or EN)  begin
     if ( ! clk_in) 
        clkEn <= EN;
  end
  assign gated_clk = clkEn & clk_in;
  ```
  
  - 实际工程中，应尽量避免显式写上面的clock gating。厂家工艺库通常会提供integrated clock gating cell (ICG), 综合工具在综合时可以自动选择ICG插入在需要gating的地方。因为综合工具对上面的代码会综合出一个latch和一个AND门，而后端布局布线时可能会使得这两个门距离较远，timing上会受影响。而ICG是一个standard cell，则保证了latch和AND门距离很近。

  

  
