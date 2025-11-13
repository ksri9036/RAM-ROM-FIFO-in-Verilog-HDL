# **EXP5: Design and Simulation of RAM, ROM, and FIFO Memory with Read and Write Operations Using Verilog HDL**

---

## **Aim**
To design, simulate, and verify the functionality of **RAM**, **ROM**, and **FIFO memory** modules with **read and write operations** using Verilog HDL.

---

## **Apparatus Required**
- System with **Vivado Design Suite**
---

## **Theory**

### **1. Random Access Memory (RAM)**
RAM is a volatile memory used to store data temporarily during program execution. It supports both **read** and **write** operations. Data can be accessed randomly using the address lines.  
In Verilog, RAM can be modeled using a `reg` array with write operation controlled by a **write enable (WE)** signal.

### **2. Read Only Memory (ROM)**
ROM is a non-volatile memory where data is permanently stored and can only be **read**, not written. The contents are initialized during declaration using an `initial` block or `$readmemb`/`$readmemh` commands.

### **3. First In First Out (FIFO) Memory**
FIFO is a sequential buffer that stores data such that the **first data written is the first data read**. It uses two pointers — **write pointer** and **read pointer** — to manage data flow. FIFO finds application in data buffering and inter-module communication.

---

## **Program**

### **1. RAM Module**
```
module ram (clk,we,addr,din,dout);
input clk;
input we;                 
input [11:0] addr;         
input [7:0] din;          
output reg [7:0] dout;     
reg [7:0] mem [0:4095];        

always @(posedge clk) 
begin
    if (we)
        mem[addr] <= din;      
        dout <= mem[addr];        
end
endmodule
```
### Testbench for RAM
```
module ram_tb;
reg clk;
reg we;
reg [11:0] addr;
reg [7:0] din;
wire [7:0] dout;
integer i;
ram dut (clk,we,addr,din,dout);

initial 
begin
    clk = 0;
    forever #5 clk = ~clk;
end

initial 
   begin
    we = 0;
    addr = 0;
    din = 0;
    #10;

       for (i = 0; i < 20; i = i + 1) 
        begin
        @(posedge clk);
        addr = $random % 4096;   // Random address (0-4095)
        din  = $random % 256;    // Random 8-bit data (0-255)
        we   = 1;
        @(posedge clk);
        we   = 0;
        end
$finish;
end
endmodule
```
### Simulation Output for RAM

<img width="1920" height="1200" alt="Screenshot 2025-11-05 082317" src="https://github.com/user-attachments/assets/4539a958-2d93-440b-b3ee-51113bcc33a3" />

### 2. ROM Module
```
module rom(
input clk,rst,
input[9:0]address,
output reg [9:0]dout );
reg[7:0] rom[1023:0];

initial 
begin
rom[10'd1000]=8'd100;
rom[10'd1021]=8'd125;
rom[10'd1023]=8'd105;
end
always@(posedge clk)
begin
if(rst)
dout<=8'b0;
else
dout<=rom[address];
end
endmodule
```
### Testbench for ROM
```
module rom_tb;
reg clk_t,rst_t;
reg [9:0] address_t;
wire [7:0] dout_t;
rom dut(.clk(clk_t),.rst(rst_t),.address(address_t),.dout(dout_t));
initial
begin
clk_t=1'b0;
rst_t=1'b1;
address_t=10'd0;
#50 rst_t=1'b0;
address_t=10'd1000;
#100;
address_t=10'd1021;
#100
address_t=10'd1023;
#100;
$finish;
end
always #10 clk_t=~clk_t;
endmodule
```
### Simulation Output for ROM

<img width="1920" height="1200" alt="Screenshot 2025-11-05 083249" src="https://github.com/user-attachments/assets/eee6cce3-9937-41d4-9b12-534b3bcd1b59" />


### 3. FIFO Memory Module
```
module fifo(clk,rst,wr_en,rd_en,data_in,full,data_out,empty,count);
input clk;
input rst;
input wr_en;
input [7:0] data_in;
input rd_en;
output reg full;
output reg [7:0] data_out;
output reg empty;
output reg [4:0] count;

reg [7:0] mem [0:15];
reg [3:0] wr_ptr;
reg [3:0] rd_ptr;

always @(posedge clk) 
  begin
       if (rst) 
         begin
           wr_ptr   <= 0;
           rd_ptr   <= 0;
           count    <= 0;
           data_out <= 0;
           full     <= 0;
           empty    <= 1;
         end 
     else 
       begin
           full  <= (count == 16);
           empty <= (count == 0);
           if (wr_en && !full) 
           begin
               mem[wr_ptr] <= data_in;
               wr_ptr <= wr_ptr + 1'b1;
           end
           if (rd_en && !empty) 
           begin
                   data_out <= mem[rd_ptr];
                   rd_ptr <= rd_ptr + 1'b1;
           end 

case ({wr_en && !full, rd_en && !empty})
   2'b10: count <= count + 1'b1;
   2'b01: count <= count - 1'b1;
   default: count <= count;
endcase
full  <= (count == 16);
           empty <= (count == 0);
       end
   end
endmodule
```
### Testbench for FIFO
```
`timescale 1ns/1ps

module fifo_tb;
reg clk;
reg rst;
reg wr_en;
reg rd_en;
reg [7:0] data_in;
wire [7:0] data_out;
wire full;
wire empty;
wire [4:0] count;

fifo uut (clk,rst,wr_en,rd_en,data_in,full,data_out,empty,count );

always #5 clk = ~clk;

initial 
begin
clk = 0;
rst = 1;
wr_en = 0;
rd_en = 0;
data_in = 8'h00;            
rst = 0;                          
rst = 1;                          
rst = 0;                         

repeat (5) 
begin
    @(posedge clk);
    wr_en = 1;
    data_in = data_in + 1;
end
@(posedge clk);
wr_en = 0;
repeat (3) 
  begin
    @(posedge clk);
    rd_en = 1;
end
@(posedge clk);
rd_en = 0;
#20;
$finish;
end

endmodule
```
### Simulation Output for FIFO

<img width="1920" height="1200" alt="Screenshot 2025-11-05 080046" src="https://github.com/user-attachments/assets/197d96af-3bb1-45b3-bd8e-a0880d40f882" />

### Result


The RAM, ROM, FIFO memory with read and write operations was designed and successfully simulated using Verilog HDL. The testbench verified both the write and read functionalities by simulating the memory operations and observing the output waveforms. The experiment demonstrates how to implement memory operations in Verilog, effectively modeling both the reading and writing processes.

The RAM, ROM, and FIFO memory modules were successfully designed, simulated, and verified using Verilog HDL in Vivado Design Suite.
All read and write operations performed as expected during simulation.
