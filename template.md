# HW 2

## Problem 1

My source code:

```verilog
//Filename: hw2_1a
module hw2_1a(
    input wire a,
    input wire b,
    input wire c,
    output reg u,
    output reg v
    );
    
    //behavioral
    always@(*)begin
    
        //111 = 01
        if (a==1 && b==1 && c==1)begin
            u = 0;
            v = 1;
        end
        
        //110 = 11
        else if (a==1 && b==1 && c==0)begin
            u = 1;
            v = 1;
        end 
        
        //100 = 00
        else if (a==1 && b==0 && c==0)begin
            u = 0;
            v = 0;
        end
        
        else begin
            u = 0;
        end
    end
    
endmodule
```

## Problem 2

My source code:

```verilog
//Filename: hw2_2
module hw2_2(
    //ports list
    input clk,
    input rst,
    input load_shift_n,
    input [2:0] loadValue,
    output [2:0] q,
    output zeroFlag
    );
    
    //3 bit register
    reg [2:0] reg_q ;
    
    //zero flag
    assign zeroFlag = (reg_q == 3'b000);
    
    //triggered by the positive edge of the clock
    always@(posedge clk) begin
    
        //if the restart = 1
        if (rst) begin
            reg_q <= 3'b000; //load the register with 0s
        end
        else begin
            if(load_shift_n)begin
                reg_q <= loadValue; //load the value into the register
            end
            else begin
                reg_q <= {1'b0, reg_q[2:1]}; //shift to the right & add a 0 
            end
        end
    end
    
    //assigns the register valye to q
    assign q = reg_q;

endmodule
```
## Problem 3

My source code:

```verilog
//Filename: hw2_3
module hw2_3(
    input [3:0] dataInA,
    input [3:0] dataInB,
    input s,
    output reg [3:0] result
    );
    
    //pick a combinational logic based on the selected signal
    always @(*) begin
        case (s)
            //use the bitwise logical operations?
            3'b000: result = dataInA & dataInB; // AND operation
            3'b001: result = dataInA | dataInB; // OR operation
            3'b010: result = ~(dataInA & dataInB); // NAND operation
            3'b011: result = ~(dataInA | dataInB); // NOR operation
            3'b100: result = dataInA ^ dataInB; // XOR operation
            default: result = 4'b0000; // Default case to handle unexpected values of s
        endcase
    end
    
endmodule
```

## Problem 4

My source code:

```verilog
//Filename: hw2_4
module hw2_4(
    //port list
    input clk,
    input [3:0] dataInA,
    input [3:0] dataInB,
    input [2:0] s,
    output reg [3:0] q //output of the register
    );
    
    reg [3:0] result;
    
    //pick a combinational logic based on the select signal
    always @(*) begin
        case (s)
            //use the bitwise logical operations?
            3'b000: result = dataInA & dataInB; // AND operation
            3'b001: result = dataInA | dataInB; // OR operation
            3'b010: result = ~(dataInA & dataInB); // NAND operation
            3'b011: result = ~(dataInA | dataInB); // NOR operation
            3'b100: result = dataInA ^ dataInB; // XOR operation
            default: result = 4'b0000; // Default case to handle unexpected values of s
        endcase
    end
    
    // Registered output logic
    always @(posedge clk) begin
        q <= result; // Register the result output at each clock edge
    end
    
endmodule
```



## Appendix 


Note that on many linux systems a document converter, pandoc, can be used to render this file as an html file, which can with further effort be converted to other formats including PDF.

```bash
pandoc template.md -o template.html --standalone
```


