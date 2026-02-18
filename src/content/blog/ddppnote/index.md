---
title: 小数电笔记
publishDate: 2025-08-18 08:00:00
description: '整理了小数电实验班后半学期lab部分的知识要点'
tags:
  - ddpp
  - 课程笔记
heroImage: { src: './ddppfig.png', color: '#B4C6DA' }
language: '中文'
---
写在前面：小数电实验班前半学期和后半学期分为两个部分，前半学期讲数电的基本知识，后半学期为8个写verilog的lab，这个笔记整理了后半学期的知识要点，是笔者复习用的所以写的比较简略，最好先
看完ppt和写完lab之后看这个能够快速回顾一下要点；2025年期末没有考察任何verilog代码书写：）

# ddpp note

## Verilog 语法注意事项

- reg & wire
    
    不声明[3:0]就默认1位二进制，不声明reg就默认wire; 常数写法4’b1001
    

![image.png](ddpp%20note%20209b386cab6380598fdec22cdee08699/image.png)

![image.png](ddpp%20note%20209b386cab6380598fdec22cdee08699/image%201.png)

```verilog
// 2-to-1 multiplexer with dual-polarity outputs
module weird_mux2( 
	input a,b, sel, 
	output z, 
	output reg zbar 
	// output reg [3:0] s //for 4-digital bin
	);
assign z = sel ? b: a;

// assign zbar = ~z; //if the zbar is wire
always @(posedge clk) begin
zbar = ~z;
end

endmodule
```

- 实例化

```verilog
~timescale 1ns/1ps module Testbench; 
wire [1:0] OUT; reg EN, CLK, RST;
initial CLK = 0;
always #2 CLK=~CLK;

initial begin
	#1
	EN = 0;
	RST = 1;
	#4
	EN = 1;
	RST = 0;
end
//already define module counter
counter u1(
.out (OUT)
.enable (EN)
.clk (CLK)
.reset (RST)
);
endmodule
```

- 同步reset & 异步reset DFF

```verilog
//sync
always @ ( posedge clk)
if (reset) begin
  q <= 1'b0;
end  else begin
  q <= data;
end

//async
always @ ( posedge clk or posedge reset)
if (reset) begin
  q <= 1'b0;
end  else begin
  q <= data;
end
```

reset应选取位posedge还是negedge？reset=1重置则从0变到1需要更新，选always @ posedge

- 逻辑运算符（lab2的gray code）

![image.png](ddpp%20note%20209b386cab6380598fdec22cdee08699/f75ccf7d-cb93-443e-8c4f-371daf68878c.png)

```verilog
//^按位异或
gray <= binary ^ (binary >> 1);
```

**移位 算数移位的区别**：算数移位符号位不变

- 阻塞赋值与非阻塞赋值
    
    1.阻塞赋值 a = b 赋予这一时刻的值；因为要赋予这一时刻值所以必须先运行前面的语句才知道这一时刻值是啥，称为“阻塞”
    
    2.非阻塞赋值 a <= b 赋予上一时刻的值
    
    3.verilog并行执行always块语句，因此禁止同一变量在多个always块赋值
    
- FSM

很多系统空闲状态都是rst为1，因为供电保证高电平FSM开始工作置为空闲

## 通信接口

- UART物理接口

**仅发射信息信号不发射时钟信号**（异步发射），提前约定了波特率说明发射接收端时钟怎么设置

原理：**移位寄存器串行发射**，发射时是并转串移位寄存器，接收是串转并移位寄存器

![image.png](ddpp%20note%20209b386cab6380598fdec22cdee08699/image%202.png)

![image.png](ddpp%20note%20209b386cab6380598fdec22cdee08699/image%203.png)

每次发送的数据包会包含：起始码、信息（可包含检纠错码）、结束码

波特率是说我发射接收不同的时钟周期要保证有一个大家公认的发射接收采样频率，一般为115200；

- SPI接口

![image.png](ddpp%20note%20209b386cab6380598fdec22cdee08699/image%204.png)

**发射信息信号也发射时钟信号**，

1. SCK：Clock from Controller
2. PICO：Peripheral-In Controller-Out
3. POCI：Peripheral-Out Controller-In
4. Cs：Chip Select

- $I^{2}C$接口

同步串行，

1. SCL作为时钟和start/stop condition；
2. SDA串行：1.是否开始 2.地址 3.读/写 4.数据 5.是否结束

![image.png](ddpp%20note%20209b386cab6380598fdec22cdee08699/image%205.png)

## 存储器

- ROM & SRAM & DRAM

![image.png](ddpp%20note%20209b386cab6380598fdec22cdee08699/image%206.png)

![image.png](ddpp%20note%20209b386cab6380598fdec22cdee08699/image%207.png)

![image.png](ddpp%20note%20209b386cab6380598fdec22cdee08699/image%208.png)

- regfile
    
    ```verilog
    module RegisterFile #(
      parameter DataWidth  = 32,
      parameter NumRegs    = 32,
      parameter IndexWidth = $clog2(NumRegs) //= 5
    ) (
      input  clk,
      input  writeEn,
      input  [IndexWidth-1:0] writeAddr,
      input  [ DataWidth-1:0] writeData,
      input  [IndexWidth-1:0] readAddr1,
      input  [IndexWidth-1:0] readAddr2,
      output [ DataWidth-1:0] readData1,
      output [ DataWidth-1:0] readData2
    );
    
      logic [DataWidth-1:0] regs[NumRegs];
    
      always_ff @(posedge clk) begin
        if (writeEn) begin
          regs[writeAddr] <= writeData;
        end
      end
    
      assign readData1 = regs[readAddr1];
      assign readData2 = regs[readAddr2];
    
    endmodule
    
    //FIFO based on reg
    module FIFO(//shenglue
    );
    
    RegisterFile dataReg(
      .clk (clk),
      .writeEn (wr),
      .writeAddr (opaddress),
      .writeData (data_in),
      .readAddr1 (opaddress),
      .readAddr2 (//no_use),
      .readData1 (data_out),
      .readData2 (//nouse)
    );
    
    assign opaddress = wr ? wr_ptr : rd_ptr;
    
    always @(posedge clk or negedge rstb) begin
        if (!rstb) begin
          wr_ptr   <= 0;
          rd_ptr   <= 0;
          fifo_cnt <= 0;
        end else begin
    	    if (wr && !full) begin
    		    wr_ptr <= wr_ptr+1;
    		    cnt <= cnt+1;
    		    end
    		  if (rd && !empty) begin
    			  rd_ptr <= rd_ptr+1;
    			  cnt <= cnt-1;
    			  end
    	  end
    end
    endmodule
    ```
    
- RAM FIFO LIFO的行为模型

```verilog
//RAM (sync,async)
module ram_sr # 
(parameter DATA_WIDTH = 8, ADDR_WIDTH = 8)
(
clk         , // Clock Input
address     , // Address Input
d           , // Data input
q           , // Data output
cs          , // Chip Select
web         , // Write Enable/Read Enable, low active
oe            // Output Enable
); 
localparam RAM_DEPTH = 1 << ADDR_WIDTH;
//...
//--------------Input Ports----------------------- 
input clk;
input [ADDR_WIDTH-1:0] address;
input cs;
input web;
input oe; 
//--------------Inout Ports----------------------- 
input [DATA_WIDTH-1:0]  d;
output [DATA_WIDTH-1:0]  q;
//--------------Internal variables---------------- 
reg [DATA_WIDTH-1:0] data_out;
reg [DATA_WIDTH-1:0] mem [0:RAM_DEPTH-1];

//--------------Core Function--------------------- 
// Tri-State Buffer control 
// output : When web = 1, oe = 1, cs = 1
assign q = (cs && oe && web) ? data_out : 'bz; 

// Memory Write Block 
// Write Operation : When web = 0, cs = 1
always @ (posedge clk)
begin : MEM_WRITE
   if ( cs && ~web ) begin
       mem[address] = d;
   end
end

// Memory Read Block 
// Read Operation : When web = 1, oe = 1, cs = 1
always @ (posedge clk)
//async
//always @ (address or cs or web or oe)
begin : MEM_READ
    if (cs && web && oe) begin
         data_out = mem[address];
    end
end
endmodule
```

```verilog
module fifo (
  input             clk       , // Clock input
  input             rstb      , // Reset all function, low active
  input             rd        , // Pop one datum out
  input             wr        , // Push one datum in
  input   [15:0]     data_in  , // Data input port
  output  [15:0]     data_out , // Data output port
  output            empty     , // FIFO empty indicator, high active
  output            full        // FIFO full indicator, low active
);

  // === Parameter definitions ===
  parameter DATA_WIDTH = 16;
  parameter ADDR_WIDTH = 4; // FIFO depth = 2^4 = 16
  localparam DEPTH = 1 << ADDR_WIDTH;

  // === Internal signals ===
  reg [ADDR_WIDTH-1:0] wr_ptr;
  reg [ADDR_WIDTH-1:0] rd_ptr;
  reg [ADDR_WIDTH:0] fifo_cnt; // one more bit to detect full

  wire ram_cs = wr || rd;
  wire ram_web = ~wr;  // wr=1 => write (web=0), wr=0 => read (web=1)
  wire ram_oe  = rd;
  wire [15:0] ram_q;

  // === Instantiate RAM with Sync Read ===
  ram_sr #(
    .DATA_WIDTH(DATA_WIDTH),
    .ADDR_WIDTH(ADDR_WIDTH)
  ) u_ram (
    .clk(clk),
    .address(wr ? wr_ptr : rd_ptr),
    .d(data_in),
    .q(ram_q),
    .cs(ram_cs),
    .web(ram_web),
    .oe(ram_oe)
  );

  // === Output assignment ===
  assign data_out = ram_q;

  // === FIFO state indicators ===
  assign empty = (fifo_cnt == 0);
  assign full  = (fifo_cnt == DEPTH);

  // === FIFO control logic ===
  always @(posedge clk or negedge rstb) begin
    if (!rstb) begin
      wr_ptr   <= 0;
      rd_ptr   <= 0;
      fifo_cnt <= 0;
    end else begin
      // Write operation
      if (wr && !full) begin
        wr_ptr <= wr_ptr + 1;
        fifo_cnt <= fifo_cnt + 1;
      end

      // Read operation
      if (rd && !empty) begin
        rd_ptr <= rd_ptr + 1;
        fifo_cnt <= fifo_cnt - 1;
      end
    end
  end

endmodule
```

## 四种架构设计技术

能量效率：每消耗 1 焦耳（Joule）能量，处理器能执行多少次操作（Op），**单位**：**Op/J=OPS/W**
功率效率：单位时间内（即每秒）完成多少操作，除以功耗（W），**单位**：**OPS/W**

延迟：数据输入和被处理数据输出之间的时间差（时钟周期）
吞吐量（Throughput）：每次处理的数据量时钟周期（比特/秒）

![image.png](ddpp%20note%20209b386cab6380598fdec22cdee08699/image%209.png)

![image.png](ddpp%20note%20209b386cab6380598fdec22cdee08699/image%2010.png)

![image.png](ddpp%20note%20209b386cab6380598fdec22cdee08699/image%2011.png)

![image.png](ddpp%20note%20209b386cab6380598fdec22cdee08699/image%2012.png)

## 浮点数

| Exponent | Fraction | Object | Value |
| --- | --- | --- | --- |
| 0 | 0 | 0 | -- |
| 0 | Nonzero | Denormalized number | (-1)^S × (0.F) × 2^(1-B) |
| Nonzero | Anything | Floating-point number | (-1)^S × (1.F) × 2^(E-B) |
| All “1” | 0 | infinity | -- |
| All “1” | Nonzero | NaN (not a number) | -- |

| Type | Sign | Exponent | Exponent bias | significand | total |
| --- | --- | --- | --- | --- | --- |
| Half (IEEE 754-2008) | 1 | 5 | 15 | 10 | 16 |
| Single | 1 | 8 | 127 | 23 | 32 |
| Double | 1 | 11 | 1023 | 52 | 64 |
| Quad | 1 | 15 | 16383 | 112 | 128 |

sign exponent fraction

0 00000000 0000001 =  $(-1)^{0} * (0.0000001)_{2} * 2^{1-127}$