# HW 3

## Problem 1

My testbench:

```verilog
`timescale 1ns / 1ps

module tb1;
    reg a,b,clk;
    wire [1:0] u,v;
       
    DUT testDUT(
        .u(u),
        .v(v),
        .a(a),
        .b(b),
        .clk(clk)  
    );
    
    always begin
        #5 clk = ~clk;
    end
    
    initial begin
        clk = 0;
        
        a = 0;
        b = 0;
        #10;
        
        a = 0;
        b = 1; 
        #10
        
        a = 1;
        b = 0;
        #10;
        
        a = 1;
        b = 1;
        #10;
          
        $finish;
    end

always @ (posedge clk)begin
    $strobe("at time=%0t: a=%b b=%b  _u = %b, _v = %b, u = %b, v = %b", $time, a, b, testDUT._u, testDUT._v, u, v);
end
    
endmodule
```

output:
```
at time=5000: a=0 b=0  _u = 00, _v = xx, u = 00, v = xx
at time=15000: a=0 b=1  _u = 1x, _v = 01, u = 1x, v = 01
at time=25000: a=1 b=0  _u = 00, _v = 10, u = 00, v = 10
at time=35000: a=1 b=1  _u = 00, _v = 10, u = 00, v = 10
```

## Problem 2

My testbench:

```verilog
`timescale 1ns / 1ps

module tb1;
    reg a,b,clk;
    wire [1:0] u,v;
       
    DUT testDUT(
        .u(u),
        .v(v),
        .a(a),
        .b(b),
        .clk(clk)  
    );
    
    always begin
        #5 clk = ~clk;
    end
    
    initial begin
        clk = 0;
        
        a = 0;
        b = 0;
        #10;
        $display("display at time=%0t: a=%b b=%b  _u = %b, _v = %b, u = %b, v = %b", $time, a, b, testDUT._u, testDUT._v, u, v);
        
        
        
        a = 0;
        b = 1; 
        #10
        $display("display at time=%0t: a=%b b=%b  _u = %b, _v = %b, u = %b, v = %b", $time, a, b, testDUT._u, testDUT._v, u, v);
        
        a = 1;
        b = 0;
        #10;
        $display("display at time=%0t: a=%b b=%b  _u = %b, _v = %b, u = %b, v = %b", $time, a, b, testDUT._u, testDUT._v, u, v);
        
        a = 1;
        b = 1;
        #10;
        $display("display at time=%0t: a=%b b=%b  _u = %b, _v = %b, u = %b, v = %b", $time, a, b, testDUT._u, testDUT._v, u, v);
          
        $finish;
    end

always @ (posedge clk)begin
    $display("display at time=%0t: a=%b b=%b  _u = %b, _v = %b, u = %b, v = %b", $time, a, b, testDUT._u, testDUT._v, u, v);
    $strobe("strobe   at time=%0t: a=%b b=%b  _u = %b, _v = %b, u = %b, v = %b \n", $time, a, b, testDUT._u, testDUT._v, u, v);
end
    
endmodule

```

output:
```
display at time=5000: a=0 b=0  _u = xx, _v = xx, u = xx, v = xx
strobe   at time=5000: a=0 b=0  _u = 00, _v = xx, u = 00, v = xx 

display at time=10000: a=0 b=0  _u = 00, _v = xx, u = 00, v = xx
display at time=15000: a=0 b=1  _u = 00, _v = xx, u = 00, v = xx
strobe   at time=15000: a=0 b=1  _u = 1x, _v = 01, u = 1x, v = 01 

display at time=20000: a=0 b=1  _u = 1x, _v = 01, u = 1x, v = 01
display at time=25000: a=1 b=0  _u = 1x, _v = 01, u = 1x, v = 01
strobe   at time=25000: a=1 b=0  _u = 00, _v = 10, u = 00, v = 10 

display at time=30000: a=1 b=0  _u = 00, _v = 10, u = 00, v = 10
display at time=35000: a=1 b=1  _u = 00, _v = 10, u = 00, v = 10
strobe   at time=35000: a=1 b=1  _u = 00, _v = 10, u = 00, v = 10 
```

## Problem 3

My testbench:

```verilog
`timescale 1ns / 1ps

module tb1;
    reg a,b,clk;
    wire [1:0] u,v;
       
    DUT testDUT(
        .u(u),
        .v(v),
        .a(a),
        .b(b),
        .clk(clk)  
    );
    
    always begin
        #5 clk = ~clk;
    end
    
    initial begin
        clk = 0;
        
        a = 0;
        b = 0;
        #10;
        $display("display at time=%0t: a=%b b=%b u = %b, v = %b", $time, a, b, u, v);
        
        
        
        a = 0;
        b = 1; 
        #10
        $display("display at time=%0t: a=%b b=%b u = %b, v = %b", $time, a, b, u, v);
        
        a = 1;
        b = 0;
        #10;
        $display("display at time=%0t: a=%b b=%b u = %b, v = %b", $time, a, b, u, v);
        
        a = 1;
        b = 1;
        #10;
        $display("display at time=%0t: a=%b b=%b u = %b, v = %b", $time, a, b, u, v);
          
        $finish;
    end

always @ (posedge clk)begin
    $display("display at time=%0t: a=%b b=%b  u = %b, v = %b", $time, a, b, u, v);
    $monitor("monitor at time=%0t: a=%b b=%b  u = %b, v = %b \n", $time, a, b, u, v);
end
    
endmodule

```

output:
```
display at time=5000: a=0 b=0  u = 00, v = 10
monitor at time=5000: a=0 b=0  u = 00, v = 10 

display at time=10000: a=0 b=0 u = 00, v = 10
monitor at time=10000: a=0 b=1  u = 00, v = 10 

display at time=15000: a=0 b=1  u = 00, v = 10
display at time=20000: a=0 b=1 u = 00, v = 10
monitor at time=20000: a=1 b=0  u = 00, v = 10 

display at time=25000: a=1 b=0  u = 00, v = 10
display at time=30000: a=1 b=0 u = 00, v = 10
monitor at time=30000: a=1 b=1  u = 00, v = 10 

display at time=35000: a=1 b=1  u = 00, v = 10
display at time=40000: a=1 b=1 u = 00, v = 10
```
