---
title: HDB3 Decode And Endecode System
date: 2021-09-30
tags: 
    - Quartus Ⅱ
    - Modelsim
    - Verilog HDL
categories: Principle of Communication
copyright: true
---

:pushpin:

- HDB3编解码系统，由Verilog HDL实现QuartusⅡ和Modelsim联合进行功能仿真，在FPGA芯片实测。
- 整体功能模块时序逻辑都比较简单，没有进行时序仿真，直接上FPGA实测，基本没有大问题。
- FPGA主芯片采用ALTREA EP2C8T144C8N，故需要使用QuartusⅡ13.0及以下版本，否则无法使用芯片库，可用采用多版本共存方式，Altera允许多个版本同时使用，并且有清除的版本区分。
- 数模转换 D/A ：实现数字模拟转换功能，将数字信号采集变成模拟信号输出，使用National Semiconductor DAC 0832
- 模数转换 A/D ：实现模拟数字转换功能，将模拟信号采集后变成数字信号输出，使用Maxim Integrated ADC MAX7820
- 整体包括分频器设计、M 序列设计、映射表设计、编码电路设计、D/A 转换设计、A/D 采样设计、 位同步设计、解码电路设计、编解码系统总体电路设计。
:dash::clock930::sleeping:
<!--more-->

## Principle ##

- HDB3码为三阶高密度双极性码，是AMI码的一种改进型，保持了AMI的优点而克服了其的缺点，使连“0”个数不超过三个；
- 编码规则如下：
  - i.先检查消息码的连“0”个数。当连“0”数目小于等于3时，则于AMI码的编码规则一样。
  - ii.当连“0”数目超过3个时，则将每4个连“0”化作一小节，用“000V”替代。（V取值+1或-1）应与其前一个相邻的非“0”脉冲的极性相同（因为这破坏了极性交替的规则，所以V称为破坏脉冲）。
  - iii.相邻的V码极性必须交替。当V码取值能满足之前的条件时但不能满足此要求时，则将“0000”用“B00V”替代。B的取值与后面的V脉冲一致，用于解决此问题。因此，B称为调节脉冲。
  - iv.V码后面的传号码极性也要交替。
- 编码比较复杂，但解码却比较简单，从编码规则上看，每一个破坏脉冲V总是与前一非“0”脉冲同极性（包括B在内），也就是说从收到的符号序列中可以容易地找到破坏点V，于是也判定V符号以及前面的三个符号必是连“0”符号，从而恢复4个连“0”码，再将所有-1变成+1后便得到原始消息码。
- m序列是最长线性反馈移位寄存器序列的简称。它是由线性反馈的移存器产生的周期最长的序列。在这种反馈移存器中避免出现全“0”状态，否则移存器的状态将不会改变，n级移存器共有 种可能的状态，除去全“0”状态外，还有 状态可用。产生m序列的充要条件就是移位寄存器的特征多项式为本原多项式，可根据n阶的m序列本原多项式的反复移位来产生我们需要的m序列。

## Design ##

- 由原始的的32.768MHz时钟，经过第一次256分频用于后续的码元定时恢复，继续经过16分频产生8KHz的码元时钟，由M序列产生模块生成随机的M序列来模拟传输过程中的随机信号，根据AMI码和HDB3的编码规则对M序列进行调整，产生单极性的HDB3码，但由于存在0、+1、-1三种情况，至少需要两位数据才能存储，所以进行一次双极性变换，对应转化成八位数据，至此完成编码过程。

## Frame Diagram ##

![Fig.1 Frame Diagram][1]
<center><font size=2>Fig.1 Frame Diagram</font></center>

## Code ##

### 分频设计 ###

- even256_div模块
  - 模块even256_div的作用是将最开始的32.768MHz系统时钟进行偶数256分频，产生新的时钟信号，供给后续16分频模块使用，同时用于解码部分的码元定时恢复模块判决时钟使用。

``` Verilog HDL
module even256_div(clk,rst,clk_out);

input clk,rst;
output clk_out;
reg clk_out;
reg[7:0] count;
// 偶数分频
parameter N=256;

always @(posedge clk or negedge rst)
    if(!rst) begin
        // 复位初始化
        count<=1'b0;
        clk_out<=1'b0;
    end
    else if(N%2==0)begin
        if(count<N/2-1) count <=count+1'b1;
        else begin
            // 计数到一半，时钟翻转实现分频
            count<=1'b0;
            clk_out<=~clk_out;
        end
    end

endmodule
```

- even16_div模块
  - 模块even16_div的作用是将256分频输出的时钟再次进行偶数分频，得到的时钟就是用于m序列产生的码元时钟。

``` Verilog HDL
module even16_div(clk,rst,clk_out);
    
input clk,rst;
output clk_out;
reg clk_out;
reg[3:0] count;
// 偶数分频
parameter N=16;

always @(posedge clk or negedge rst)
    if(!rst) begin
        // 复位初始化
        count<=1'b0;
        clk_out<=1'b0;
    end
    else if(N%2==0)begin
        if(count<N/2-1) count <=count+1'b1;
        else begin
            // 计数到一半，时钟翻转实现分频
            count<=1'b0;
            clk_out<=~clk_out;
        end
    end

endmodule
```

### M序列设计 ###

- m_sequence模块
  - 模块m_sequence的作用是根据7阶m序列的本原多项式，循环位移，使用16分频的码元时钟产生伪随机序列模拟基带传输信号。

``` Verilog HDL
module m_sequence(clk,rst,ena,data_out,load);

input clk;                        //时钟信号
input rst;                        //复位信号，低电平有效
input ena;                        //控制信号，高电平时序列发生器开始工作
output data_out;
output load;                      //控制信号，为高电平时表示伪随机序列开始
reg data_out,load;
reg [6:0]temp;                    //7级移位寄存器产生周期为127的m序列

always @(posedge clk or negedge rst) begin
    if(!rst) begin
            data_out <= 0;
            load <= 1'b0;            //控制信号设为无效
            temp <= 7'b1111_111;     //移位寄存器初始状态设为全1
    end
    else begin                       //开始产生序列信号
        if(ena) begin                //判断序列发生器的控制信号是否有效
                //若控制信号有效
                load <= 1'b1;        //将控制伪随机序列产生的信号设为有效        
                temp <= {temp[5:0],temp[2]^temp[6]};        //对应f(x)=x^7+x^3+1
                //当控制伪随机数列产生的信号使能时将移位寄存器的最高位做为m序列进行输出
            if(load)    data_out <= temp[6];
        end
        else load <= 1'b0;    //若控制信号无效，则不开始产生伪随机序列
    end
end

endmodule
```

### 编码电路设计 ###

- 模块add_v的作用是找到m序列中4个连续的0，并将第四个连0变成v。
  - 输入信号为“0”码和“1”码，输出信号用两位二进制数“00”“01”，“11”分别表示信号为“0”、“1”、“V”；
  - 设置了一个寄存器用于存储当前码元位置处连“0”个的个数；
  - 对输入信号进行判断，若输入高电平，则计数器复位，输出为“01”；若输入低电平，则将计数器加一，并判断此时计数器的值是否为“4”，若计数器的值为“4”，则表示出现四个连“0”，将该“0”信号变为“V”且计数器复位，输出为“11”，否则，输出为“00”。

``` Verilog HDL
module add_v(rst,data_in,data_out,clk);

input data_in,rst,clk;
output [1:0]data_out;
reg [1:0]data_out;            //用00、01、11分别表示输入信号为“0、1、V”
reg [1:0]counter;               //设置连0计数器

always@(posedge clk or negedge rst) begin
    if(!rst) begin
        counter <= 0;
        data_out <= 0;
    end
    else if(data_in == 1'b1) begin  //判断输入信号是否为1
        counter <= 0;                //若为1则计数器复位，输出“01”
        data_out <= 2'b01;
    end
    else if(data_in == 1'b0) begin                        //若输入信号为0，计数器+1
        counter <= counter + 1; 
        if(counter == 2'b11) begin    //判断连0个数是否达到4个，因为非阻塞赋值，此时计数器值应为3时，表示出现4个连0
            data_out <= 2'b11;        //将0的输出变为V
            counter <= 0;            //计数器复位
        end
        else data_out <= 2'b00;       //若连0数不为4，输出“00”
    end
end

endmodule
```

- 模块add_b的作用是为了保证在添加“V”码后的序列中，相邻的“V”码极性必须交替，当“V”码取值不能满足此要求时，则将“000V”用“B00V”替代。
  - 输入信号为add_v模块的输出，输出信号用两位二进制数“00”、“01”、“10”、“11”分别表示信号为“0”、“1”、“B”、“V”；
  - 设置两个寄存器分别用于存储“1”码和“V”码的个数；
  - 设置四位移位寄存器方便实现用“B00V”替代“000V”的功能；
  - 对输入信号进行判断，若输入为“00”，将“1”码计数器加上“V”码计数器的值，并复位“V”码计数器；若输入为“01”，先将“1”码计数器加一，然后加上“V”码计数器的值，最后复位“V”码计数器；若输入为“11”，则“V”码计数器加一；
  - 用“1”码计数器和“V”码计数器的值判断最终的输出信号，若输入不为“11”，则输出不变；若输入为“11”，判断此时“1”码计数器的值，当此前“1”码的个数为奇数时，输出不变，当此前“1”码的个数为偶数时，输出“10”。

``` Verilog HDL
module add_b(rst,data_in,data_out,clk);

input clk,rst;
input [1:0]data_in;
output [1:0]data_out;        //用00、01、10、11分别表示输入信号为“0、1、B、V”
reg counter1;                    //设置“01”的计数器
reg counterv;                    //设置“11”的计数器
reg [1:0]buffer[3:0];        //取代节选择

always@(posedge clk or negedge rst) begin
//设置四位移位寄存器方便插“B”的实现
    if(!rst) begin
        buffer[3]<=0;
        buffer[2]<=0;
        buffer[1]<=0;
        buffer[0]<=0;
    end
    else begin
        buffer[3]<=buffer[2];
        buffer[2]<=buffer[1];
        buffer[1]<=buffer[0];
        buffer[0]<=data_in;
    end
end

always@(posedge clk or negedge rst) begin
//对输入进行判断
    if(!rst) begin
        counter1 = 0;
        counterv = 0;
    end
else if(data_in == 2'b11) //如果输入为“11”，则counterv加1
        counterv <= counterv + 1'b1;
    else if(data_in == 2'b01) begin //如果输入为“01”，则counter1加1
        counter1 <= counter1 + 1'b1 + counterv;
        counterv <= 1'b0;
    end
    else begin
        counter1 <= counter1 + counterv;
        counterv <= 1'b0;
    end
end

//若输入为“11”,如果此前1的个数为奇数，则输出不变，若为偶数，则输出“10”
//若输入不为“11”,则输出不变
assign data_out = (counter1%2 == 0) && (counterv == 1)? 2'b10 : buffer[3];

endmodule
```

- 模块polar的作用是单极性信号变为双极性信号。
  - 输入信号为add_b模块的输出，输出信号用“00”表示“0”码，用“10”表示“+1、+B、+V”码，用“01”表示“-1、-B、-V”；
  - 设置极性判断标志位，当其为“1”时，表示“+1”和“-V”，当其为“0”时，表示“-1”和“+V”；
  - 对输入信号进行判断，若输入为“11”（V码），利用极性判断标志位判断该码正负；若输入为“01”（1码）或者“10”（B码），利用极性判断标志位判断该码正负，同时将极性判断标志位翻转；若输入为“00”（0码），则输出为“00”。

``` Verilog HDL
module polar(rst,data_in,data_outP,data_outN,clk);

input [1:0]data_in;
input rst,clk;
output data_outP,data_outN;    //用P与N表示正负
reg [1:0] polar_out;                 //“00”表示0，“10”表示+1，“01”表示-1
reg data_outN,data_outP;
reg even;                             //设置极性判断标志,1表示+1和-V，0表示—1和+V

always@(posedge clk or negedge rst) begin
    if(!rst) even <= 1;
    else if(data_in == 2'b11) begin    //若输入为“11（V）”
        if(even ==     1)                          //判断极性标志，若even为1
            polar_out <= 2'b01;             //输出为“01（-1）”
        else                                     //若even为0
            polar_out <= 2'b10;             //输出为“10（+1）”
    end
    //若输入为“01（1）”或者“10（B）”
    else if(data_in == 2'b01 || data_in == 2'b10) begin
        if(even == 1) begin              //判断极性标志，若even为1
            even <= 0;                     //将even翻转
            polar_out <= 2'b10;         //输出为“10（+1）”
        end
        else begin                          //若even为0
            even <= 1;                      //将even翻转
            polar_out <= 2'b01;         //输出为“01（-1）”
        end
    end
    else if(data_in == 2'b00)        //若输入为“00（0）”
        polar_out <= 2'b00;            //输出为“00（0）”
end

always@(polar_out or rst) begin    
//将输出寄存器的两位数分别赋值给输出端口data_outP和data_outN
    if(!rst) begin
        data_outP <= 0;
        data_outN <= 0;
    end
    else if(polar_out == 2'b01) begin
        data_outP <= 0;                //PN=01表示-1
        data_outN <= 1;
    end
    else if(polar_out == 2'b10) begin
        data_outP <= 1;                //PN=10表示+1
        data_outN <= 0;
    end
    else begin
        data_outP <= 0;                //PN=00表示0
        data_outN <= 0;            
    end
end
endmodule
```

- 模块change的作用是将双极性信号变更为8位二进制数。
  - 输入信号为polar模块的输出，输出信号则为8位二进制数；
  - 对输入信号进行判断，若输入信号为“00”，则输出信号为“10000000”；若输入信号为“10”，则输出信号为“11111111”，若输入信号为“01”，则输出信号为“00000000”。

``` Verilog HDL
module change(rst,data_inP,data_inN,data_out,clk);

//二输入八输出模块，将PN信号转换为八位输出信号作为DAC的输入
input rst,clk,data_inP,data_inN;
output [7:0]data_out;
reg [7:0]data_out;

always@(posedge clk or negedge rst)    begin
    if(!rst)
        data_out <= 8'b10000000;
    else if(data_inP == 1 && data_inN == 0) begin        //PN=10对应输出HFF
        data_out <= 8'b11111111;
    end
    else if(data_inP == 0 && data_inN ==1) begin        //PN=01对应输出H00
        data_out <= 8'b00000000;
    end
    else if(data_inP == 0 && data_inN ==0) begin        //PN=00对应输出H80
        data_out <= 8'b10000000;
    end
end

endmodule
```

### D/A转换设计 ###

- 模拟数据传输过程在实验箱上用一条信号连接线来表示，用DA转换器将双极性的数字信号转换成模拟信号，经过信号连接线至AD转换器，转换成双极性的0和1输出到解码模块。由于使用Quartus Ⅱ时没有AD和DA转换器，所以直接连接了双极性输出和输入断，跳过了模拟传输过程，但实际上的AD转换器输出并不是完美的双极性数据，需要根据实验箱的调整而对代码进行多次调试。

### A/D采样设计 ###

- 由AD转换器输出的八位数据，由于模拟传输过程和转化过程存在时延差，需要设计一个码元定时恢复模块，用最初256分频产生的时钟信号不断检测二维数据，根据实际的数据特征来产生与16分频信号相同的8KHz信号，并且判断码元的起始时刻，此处码元定时恢复模块输出的码元时钟将用于整个解码模块。解码模块根据HDB3码的特征，找到相应的+1码和-1码，并且判断V码和B码并清零，实现解码。

### 位同步设计 ###

- 模块recover的作用是将八位二进制信号变更为一位的单极性信号，即码元定时恢复。
  - 对输入信号进行判断，八位输出信号有三种情况，8’h00、8’h80、8’hFF，分别代表-1，0和+1这三种情况，仿真时可以用三种情况的数字特征来进行边缘检测和时钟定时恢复；
  - 在实际调试过程中，AD转换输出的三种信号并不是确定的值，会存在毛刺现象，所以需要根据实际情况调整，我们采集到的数据是大于8’h58为+1，8’h30~8’h3f为0，小于8’h0f为-1，故根据三个区间，用256分频的时钟去采样，初始置flag为0，如果检测到区间发生跃变，则置1，同时进行定时恢复，连续八个时钟翻转一次，得到码元定时恢复的时钟提供后续模块使用。

### 解码电路设计 ###

- 模块trans8to1的作用是将八位二进制数据转换为P和N双极性信号。
  - 输入信号为AD转换的8位输出，输出信号则为P和N各一位的二进制数代表双极性；
  - 对输入信号进行判断，若输入信号为“3’h00”，则输出信号为“P=0,N=1”；若输入信号为“3’h80”，则输出信号为“P=0,N=0”，若输入信号为“3’hFF”，则输出信号为“P=1,N=0”；
  - 但实际情况是，真实的AD转换输出不会是刚好的3’h00、3’h80和3’hFF，经过实际检测，输出的范围分别是3’h00~8'h0f、8'h30~8'h3f、大于8'h58，和理想的情况相差较大，如果实际操作时用理想的三种情况检测，将不能恢复出双极性信号。

``` Verilog HDL
module trans8to1(rst_n, indata_8, outdata_P, outdata_N,clk);

input rst_n,clk;
input [7:0] indata_8;
output outdata_P, outdata_N;
reg outdata_P, outdata_N;

always @(posedge clk or negedge rst_n) begin
    if(!rst_n) begin
        // 复位初始化
        outdata_P <= 0;
        outdata_N <= 0;
    end
    else begin
    // 三个区间分别对应三种双极性P和N的值来表示+1、0和-1
    if(indata_8 >= 8'h58) begin
            outdata_P<=1;
            outdata_N<=0;
        end
        else if(indata_8 <= 8'h3f && indata_8 >= 8'h30) begin
            outdata_P<=0;
            outdata_N<=0;
        end
        else if(indata_8 <= 8'h0f) begin
            outdata_P<=0;
            outdata_N<=1;
        end
    end
end

endmodule
```

- 模块findv的作用是根据P和N两位数据进行逻辑判定，找到V码，输出只包含0和1的序列。
  - 设置两个flag来表示+1和-1的个数状态，根据HDB3码中的连续同号的1来找到V码；
  - 如果PN是10，则flag2置0，flag1自加1，如果flag1自加+后为2则表示有两个连续的+1；
  - 如果PN是01，则flag1置0，flag2自加1，如果flag2自加+后为2则表示有两个连续的-1；
  - 无论是找到连续的+1还是-1都全部置11表示连续的1，正常不连续的+1和-1用01表示，其他状况都清00表示0基本还原原始序列。

``` Verilog HDL
module findv(rst_n, indata_P, indata_N, outdata,clk);

input rst_n,clk;
input indata_P, indata_N;
output reg [1:0] outdata;
reg [1:0]flag1;
reg [1:0]flag2;

initial begin
    flag1=2'b00;
    flag2=2'b00;
end

always @(posedge clk or negedge rst_n) begin
    if(!rst_n) begin
        // 复位初始化
        flag1=2'b00;
        flag2=2'b00;
        outdata <= 2'b00;
    end
    else begin
        if(indata_P==1'b1 && indata_N==1'b0) begin
        // 如果PN是10，则flag2置0，flag1自加1，如果flag1自加+后为2则表示有两个连续的+1
            flag2<=2'b00;
            flag1=flag1+2'b01;
            if(flag1==2'b10) begin
                outdata<=2'b11;
                flag1<=2'b00;
            end
            else outdata<=2'b01;
        end
        else if(indata_P==1'b0 && indata_N==1'b1) begin
        // 如果PN是01，则flag1置0，flag2自加1，如果flag2自加+后为2则表示有两个连续的-1
            flag1<=2'b00;
            flag2=flag2+2'b01;
            if(flag2==2'b10) begin
                outdata<=2'b11;
                flag2<=2'b00;
            end
            else outdata<=2'b01;
        end
        // 其他情况为00则正常置00
        else if(indata_P==1'b0 && indata_N==1'b0) outdata<=2'b00;
        //else outdata<=2'b01;
    end
end

endmodule
```

- 模块delvb的作用是将序列进行最后的处理，根据HDB3的连0规则将多余的1清零，恢复出原始的m序列完成最终的解码。
  - 设置一个buffer来暂存前四位数据，HDB3的编解码规则是对连续的四个0进行处理，之前的模块已经找到了V码，这里需要对V码以及B码清零；
  - 但实际上只要找到了V码，将V码本身和之前的三位一共四位全部清零，就可以不用考虑B码的影响；
  - 检测indata的输入若为11则表示这是一个V码，则使用buffer将四位数据清零，输出接到buffer的前三位，用counterv的0和1状况来判定清零还是直接buffer输出，完成最终的解码。

``` Verilog HDL
module delvb(rst_n, indata, outdata,clk);

input rst_n,clk;
input [1:0]indata;
output outdata;
reg [3:0]buffer;
reg bufferdata;
reg counterv;

always@(posedge clk or negedge rst_n)
begin// 设置四位buffer暂存前四位数据
if (!rst_n) begin
    buffer[3]<=0;
    buffer[2]<=0;
    buffer[1]<=0;
    buffer[0]<=0;
end
else begin
    buffer[3]<=buffer[2];
    buffer[2]<=buffer[1];
    buffer[1]<=buffer[0];
    buffer[0]<=bufferdata;
end
end


always @(posedge clk or negedge rst_n) begin
    // 复位初始化
    if(!rst_n) bufferdata <= 0;
    else begin
        if(indata==2'b01) begin
        // 如果是正常的01，则数据还是1
            counterv <= 0;
            bufferdata<=1;
        end
        else if(indata==2'b00) begin
        // 如果是正常的00，则数据还是0
            counterv <= 0;
            bufferdata<=0;
        end
        else if(indata==2'b11) begin
        // 如果是的11，则表示该位为V码，将counterv标记1
            counterv <= 1;
            bufferdata<=0;
        end
    end
end

assign outdata = (counterv == 1)  ? 0 : buffer[2];

endmodule
```

### 编解码系统总体电路设计 ###

- 顶层模块的作用是将所有的功能模块进行连接，并将部分功能模块的输出端口拉出，以方便最终的仿真及FPGA引脚的绑定。
  - 分频功能的连接：将系统时钟作为even256_div模块的输入，其输出为系统时钟的256分频时钟，将其作为even_div16模块的输入，其输出为256分频时钟的16分频时钟；
  - M序列生成功能的连接：用16分频时钟作为m_sequence模块的驱动时钟，输入使能信号，输出为m序列；
  - HDB3编码功能的连接：该功能下模块的驱动时钟都为16分频时钟，将m_sequence模块的输出作为add_v模块的输入，将add_v模块的输出作为add_b模块的输入，将add_b模块的输出作为polar模块的输入，将polar模块的输出作为change模块的输入，输出8位二进制数；
  - 码元定时恢复功能的连接：用256分频时钟作为recover模块的驱动时钟，ad转换器的输出作为recover模块的输入，输出为码元定时恢复时钟；
  - HDB3译码功能的连接:该功能下模块的驱动时钟都为码元定时恢复时钟，ad转换器的输出作为trans8to1模块的输入，将trans8to1模块的输出作为findv模块的输入，将findv模块的输出作为delvb模块的输入，delvb模块的输出即为译码恢复的m序列。

``` Verilog HDL
module HDB3(rst,
                clk,clk_256,clk_16,clk_recover,clk_256_ad,clk_256_da,
                ena,
                data_out,data_out_ad,outm,outv,outb,outP,outN,data_out_check,
                finallyout,outdata_P,outdata_N,outdata_v
                );


input  rst,clk,ena;
input [7:0]data_out_ad;
output clk_256,clk_16,clk_recover,clk_256_ad,clk_256_da;
output [7:0]data_out_check;
output [7:0]data_out;
output [1:0]outv;
output [1:0]outb;
output outm,outP,outN;
output outdata_P,outdata_N;
output [1:0]outdata_v;
output finallyout;
//分频
even256_div div256(clk,rst,clk_256);
even16_div div16(clk_256,rst,clk_16);
//m序列
m_sequence m(clk_16,rst,ena,outm,load);
//编码
add_v u1(rst,outm,outv,clk_16);
add_b u2(rst,outv,outb,clk_16);
polar u3(rst,outb,outP,outN,clk_16);
change u4(rst,outP,outN,data_out,clk_16);
//码元定时恢复
recover u8(clk_256, rst, data_out_ad, clk_recover);
//译码
trans8to1 u5(rst, data_out_ad, outdata_P, outdata_N,clk_recover);
findv u6(rst, outdata_P, outdata_N, outdata_v,clk_recover);
delvb u7(rst, outdata_v, finallyout,clk_recover);

assign clk_256_ad = clk_256;
assign clk_256_da = clk_256;
assign data_out_check = data_out_ad;

endmodule
```

## 调试与验证 ##

![Fig.2 FPGA][2]
<center><font size=2>Fig.2 FPGA</font></center>

![Fig.3 引脚分配][3]
<center><font size=2>Fig.3 引脚分配</font></center>

![Fig.4 FPGA连接][4]
<center><font size=2>Fig.4 FPGA连接</font></center>

![Fig.5 256分频和16分频][5]
<center><font size=2>Fig.5 256分频和16分频</font></center>

![Fig.6 16分频和m序列][6]
<center><font size=2>Fig.6 16分频和m序列</font></center>

![Fig.7 16分频和码元定时恢复时钟][7]
<center><font size=2>Fig.7 16分频和码元定时恢复时钟</font></center>

![Fig.8 m序列和模拟传输信号][8]
<center><font size=2>Fig.8 m序列和模拟传输信号</font></center>

![Fig.9 恢复的m序列和码元定时恢复时钟][9]
<center><font size=2>Fig.9 恢复的m序列和码元定时恢复时钟</font></center>

![Fig.10 m序列前后][10]
<center><font size=2>Fig.10 m序列前后</font></center>

## 总结 ##

> 问题1：在FPGA仿真时，ad转换器输入的八位二进制数在一开始无法在示波器上检测到。

- 解决方法：一开始我们认为是因为ad转换器自身的问题，导致输出的8位二进制数只在较低位发生变化，但经过多次检测后，我们认为实际8位二进制数并没有输入进我们的顶层模块，于是我们通过对顶层模块代码进行检查，发现产生该问题的原因是输入8位二进制数的端口本应设置为输入端口吗，但我们设置为了输出端口，所以导致信号并没有传输进来，所以无法检测到。

> 问题2：码元定时恢复不理解是什么意思，导致并不能准确的恢复出时钟。

- 解决办法：请教老师后发现原来是用256分频所得的时钟来对八位信号进行采样，根据采集到的信号变化确定码元的起始位置，再根据数字特征定时翻转，起到恢复出16分频时钟的作用。

> 问题3：实际的AD转换器和理想状态不同，用理想的条件去检测无法正常将8位AD输出转成双极性数据。

- 解决办法：这个问题困扰的时间最久，由于AD转换输出的8位数字信号并不是非常完美的，存在跳变以及根据实验箱的实际情况并不会根据0~5V输出00~FF，于是将AD输出的八位信号分别绑定到FPGA的其他空余引脚，对每个引脚上的数据进行分析，判定-1，0和+1分别对应该位的0状态还是1状态，最终得到我们需要的三种状态区间，高四位可以比较准确的判定，而第四位的0和1状态反复跳变，最终确定的区间为：3’h00——8'h0f代表-1，8'h30——8'h3f代表0，大于8'h58代表+1

## Download Source File ##

### [Project File][11] ###

### [Wave File][12] ###

<!-- markdownlint-disable-file MD025 MD033 -->
[1]: https://www.lingzhicheng.cn/usr/file/picture/HDB3/1.FrameDiagram.png
[2]: https://www.lingzhicheng.cn/usr/file/picture/HDB3/2.FPGA.png
[3]: https://www.lingzhicheng.cn/usr/file/picture/HDB3/3.引脚分配.jpg
[4]: https://www.lingzhicheng.cn/usr/file/picture/HDB3/4.jpg
[5]: https://www.lingzhicheng.cn/usr/file/picture/HDB3/5.256分频和16分频.jpg
[6]: https://www.lingzhicheng.cn/usr/file/picture/HDB3/6.16分频和m序列.jpg
[7]: https://www.lingzhicheng.cn/usr/file/picture/HDB3/7.16分频和码元定时恢复时钟.jpg
[8]: https://www.lingzhicheng.cn/usr/file/picture/HDB3/8.m序列和模拟传输信号.jpg
[9]: https://www.lingzhicheng.cn/usr/file/picture/HDB3/9.恢复的m序列和码元定时恢复时钟.jpg
[10]: https://www.lingzhicheng.cn/usr/file/picture/HDB3/10.m序列前后.jpg
[11]: https://www.lingzhicheng.cn/usr/file/HDB3_quartus+modelism.zip
[12]: https://www.lingzhicheng.cn/usr/file/HDB3wavefile.zip
