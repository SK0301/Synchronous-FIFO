# Synchronous-FIFO
# Design and Implementation of a Synchronous FIFO in IcarusVerilog <br>

//> A First in First out (FIFO) is a Hardware Module, used in Digital Circuits which serves as a synchronizer between the modules. <br>

//> When Data is transferring between the modules which has different clock frequencies( this phenomenon is called Clock Domain Crossing(CDC)), FIFO acts as a bridge between them in order to synchronize the data transfer correctly. <br>

//> FIFO Usage matters when Data transfer is happening between the modules where Write clock frequency >> Read clock frequency. <br>
<p align="center">
  <img src="https://github.com/user-attachments/assets/378e5ce7-64b0-47df-81a5-f908b6853d46" width="350"/>
</p>
<p align="center">
Fig.1:  Block diagram of FIFO
</p>

//> The Implemented FIFO is a 16x8 FIFO ie. it contains 16 data bytes.( 16 is the depth and 8 is the width of FIFO ) <br>
    The data in and data out is controlled by status flags "Full" and "Empty". <br>
    These status flags are controlled by read and write pointers. <br>

 //> Here's the detailed explanation of the verilog code of the synchronous FIFO that I've provided:<br>
module fifo_16x8 ( <br>
    input wire clk,           <br>
    input wire rst,           <br>
    input wire wr_en,          <br>
    input wire rd_en,           <br>
    input wire [7:0] data_in,   <br>
    output reg [7:0] data_out,   <br>
    output wire full,         <br>
    output wire empty    <br>     
); <br>

In the above section of code, we have declared all ports to the FIFO. (Refer the image for better clarity) <br>

Now, We proceed to declare the Depth of FIFO. Here we're declaring depth in parameters so that at any point of time, we can change the dimensions of FIFO <br>

    // Parameters
    parameter DATA_WIDTH = 8;
    parameter FIFO_DEPTH = 16;
    parameter ADDR_WIDTH = $clog2(FIFO_DEPTH); // 4 bits for addressing 16 locations
  //>  Here,  the last line ADDR_WIDTH will adjust according to the Depth of FIFO <br>
  //> Now we shall declare the registers, according to the parameters provided above <br>
  
    // Internal registers
    reg [DATA_WIDTH-1:0] mem [0:FIFO_DEPTH-1]; // Memory array
    reg [ADDR_WIDTH-1:0] wr_ptr;              // Write pointer
    reg [ADDR_WIDTH-1:0] rd_ptr;              // Read pointer

Now, we will declare the status flags and Read & Write Operation: <br>

    // Status flags
    assign full  = ((wr_ptr + 1) == rd_ptr);
    assign empty = (wr_ptr == rd_ptr);

    // Write and read operations
    always @(posedge clk) begin
        if (rst) begin
            wr_ptr <= 0;
            rd_ptr <= 0;
            data_out <= 0;
        end else begin
            // Write to FIFO
            if (wr_en && !full) begin
                mem[wr_ptr] <= data_in;
                wr_ptr <= wr_ptr + 1;
            end
            // Read from FIFO
            if (rd_en && !empty) begin
                data_out <= mem[rd_ptr];
                rd_ptr <= rd_ptr + 1;
            end
        end
    end

endmodule  <br>

# Working Principle:
>> When the Rd_ptr and wr_ptr are equal, Empty is high i.e no data in FIFO, so data can not be read from FIFO <br>
>> When wr_en is high and full is 0, write operation starts  <br>
>> After data has written, when rd_en is high and empty is zero, read operation is done  <br>

# Waveform Results: 
<p align="center">
  <img src="https://github.com/user-attachments/assets/8c5e8b93-83aa-4eb1-8f43-6d77f914be53" width="350"/>
</p>

We can observe that when empty is high and rd_en is high, data_out is capturing, here the catch is when rd_en is high and empty is low, then only data is captured, but it will be captured after the clock signal arrives, so empty being in high doesn't make any error   <br>



