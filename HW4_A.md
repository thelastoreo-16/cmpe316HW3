 
# HW 4 - COMPLETE

## Problem 1
Source code:
```verilog
// Filename: extinguisher.sv
module extinguisher (
    input logic sys_clk,        // System clock
    input logic clr_n,          // Active-low clear signal
    input logic enable,         // Enable signal
    output logic extinguish,    // 1-bit output for extinguish signal
    output logic [2:0] position // 3-bit position from the lower 3 bits of the counter
);

    // Internal counter 0-15 (4-bit wide)
    logic [3:0] counter;


    // Asynchronous clear resets the counter to 0
    always_ff @(posedge sys_clk, negedge clr_n) begin
        if (~clr_n) begin // active-low reset 
            counter <= 4'b0000;
        end else begin
            counter <= counter + 1;
        end
    end

    // Combinational logic for the extinguish signal
    always_comb begin
        // Position set the lower 3 bits
        position = counter[2:0];
        // Extinguish is high if enable is high and counter is in range [0, 7]
        if (enable && counter < 8) begin
            extinguish = 1'b1;
        end else begin
            extinguish = 1'b0;
        end
    end

endmodule

```

TestBench:
Signals
clk: Clock signal.
clr_n: Reset signal (low active).
enable: Turns the extinguishing on/off.
extinguish: Shows if the extinguishing is on.
position: The position of the process.

Test Cases:
Reset:
    We toggle clr_n to reset. After resetting, check if extinguish is off and position is 0.
Test 1:
    Turn off enable. After 20 ns, extinguish should be 0 and position should not change.
Test 2:
    Turn on enable. After 20 ns, extinguish should be 1 and position should start moving.
Test 3:
    Toggle enable on and off for a few cycles. Make sure extinguish changes accordingly, and position keeps updating.

```verilog
// Filename tb_extinguisher.sv
`timescale 1ns / 1ps

module tb_extinguisher;

    logic clk, clr_n, enable;
    logic extinguish;
    logic [2:0] position;

    // Instantiate the extinguisher module
    extinguisher uut (
        .sys_clk(clk),
        .clr_n(clr_n),
        .enable(enable),
        .extinguish(extinguish),
        .position(position)
    );

    // Generate 100 MHz clock (10 ns period)
    always begin 
        #5 clk = ~clk;
    end

    // Test sequence
    initial begin
        clk = 0;
        clr_n = 1;
        enable = 0;

        $display("\n========================");
        $display(">>> TESTBENCH START <<<");
        $display("========================\n");

        // Apply Reset
        clr_n = 0; #10;
        clr_n = 1; #10;

        // TEST 1: Ensure disable stops extinguishing
        $display(">>> TEST 1: ENABLE OFF <<<");
        enable = 0; #20;
        $display("[RESULT] Extinguish: %b, Position: %d", extinguish, position);

        // TEST 2: Enable the system
        $display(">>> TEST 2: ENABLE ON <<<");
        enable = 1; #20;
        $display("[RESULT] Extinguish: %b, Position: %d", extinguish, position);


        // TEST 4: Continue Progression, Checking for Consistency
        $display(">>> TEST 3: COUNTER CONTINUATION <<<");
        repeat (4) begin
            #20;
            $display("[STEP] Extinguish: %b, Position: %d", extinguish, position);
            enable = 0; #10;
            enable = 1; #10;
            #100;
        end

        $display("\n===========================");
        $display(">>> TESTBENCH COMPLETE <<<");
        $display("===========================\n");

        $finish;
    end

endmodule

```

Results:
```
========================
>>> TESTBENCH START <<<
========================

>>> TEST 1: ENABLE OFF <<<
[RESULT] Extinguish: 0, Position: 3
>>> TEST 2: ENABLE ON <<<
[RESULT] Extinguish: 1, Position: 5
>>> TEST 3: COUNTER CONTINUATION <<<
[STEP] Extinguish: 1, Position: 7
[STEP] Extinguish: 1, Position: 5
[STEP] Extinguish: 1, Position: 3
[STEP] Extinguish: 1, Position: 1

===========================
>>> TESTBENCH COMPLETE <<<
===========================
```

## Problem 2
Source code:
```verilog
// Filename igniter.sv
module igniter(
    input logic sys_clk,
    input logic clr_n,
    input logic enable_move,
    input logic [3:0] delta,
    output logic [2:0] position_q,
    output logic [2:0] position_d
    );
    
    // Combinational logic to calculate the next position
    always_comb begin
        if (enable_move) begin
            // modular arithmetic to calculate the new position
            position_d = (position_q + delta) % 8;
        end else begin
            // Position_d remains the same
            position_d = position_q;
        end
    end
     
    
    // Sequential logic for updating the position
    always_ff @(posedge sys_clk, negedge clr_n) begin
        if (~clr_n) begin
            // Asynchronous reset to zero
            position_q <= 3'b000;
        end else begin
            // Update position_q with position_d on the positive edge of sys_clk
            position_q <= position_d;
        end
    end

    
endmodule
```


My test bench:

Signals
position_q: Current position (input to the module).
position_d: Desired position (output from the module).
delta: Change in position (4-bit value, represents the movement).
enable_move: Controls whether movement is enabled.
sys_clk: The clock signal.
clr_n: Active-low reset signal.

Test Cases:
System Reset:
    The system is reset by asserting clr_n = 0 for 10 ns and then releasing it (clr_n = 1).

Test 1: No movement when enable_move is off.
    Set delta = 4'b0010 (move by 2).
    Ensure position_q and position_d remain unchanged when enable_move = 0.

Test 2: Movement within range.
    Set delta = 4'b0010 (move by 2).
    Enable the move with enable_move = 1.
    After disabling movement (enable_move = 0), check if position_q and position_d are updated correctly.

Test 3: Wrap around (modulo 8).
    Set delta = 4'b0011 (move by 3).
    Verify that the position wraps around correctly when position_q + delta exceeds 7 (mod 8).

Test 5: Large movement with multiple wraps.
    Set delta = 4'b0100 (move by 4).
    Ensure multiple wraps are handled correctly and the correct position is output.

Test 6: Reset.
    Apply a reset by asserting clr_n = 0 and then releasing it.

```verilog
module tb_igniter;
// Filename tb_igniter.sv
    logic [2:0] position_q, position_d;
    logic [3:0] delta;
    logic enable_move, sys_clk, clr_n;

    // Instantiate the igniter module
    igniter uut (
        .position_q(position_q),
        .position_d(position_d),
        .delta(delta),
        .enable_move(enable_move),
        .sys_clk(sys_clk),
        .clr_n(clr_n)
    );

    // Generate clock: 10ns period (100 MHz)
    always #5 sys_clk = ~sys_clk;

    // Test sequence
    initial begin
        sys_clk = 0;
        clr_n = 1;
        enable_move = 0;
        delta = 4'b0000;

        $display("\n========================");
        $display(">>> TESTBENCH START <<<");
        $display("========================\n");

        // Reset the system
        clr_n = 0; #10;
        clr_n = 1; #10;
            $display(">>> TEST FORMAT: mod8(pos_q + delta) = pos_d <<<");
        // TEST 1: No movement when enable_move = 0
        $display(">>> TEST 1: ENABLE OFF <<<");
        delta = 4'b0010; // Move by +2
        enable_move = 0;
        #20;
        $display("[RESULT] Position: %d (expected 2) Position_d: %d\n", position_q, position_d);


        // TEST 2: Moving forward within range
        $display(">> ENABLE ON FORMAT: mod8(pos_q + delta) = pos_d <<");
        $display(">>> TEST 2: MOVE FORWARD mod(0+2)=2 <<<");
        delta = 4'b0010; // Move by +2
        enable_move = 1; #10;
        enable_move = 0; #10; 
        $display("[RESULT] Position_q: %d  Position_d: %d\n", position_q, position_d);

        // TEST 3: Wrap around 
        $display(">>> TEST 3: WRAP AROUND mod(2+3)=5 <<<");
        delta = 4'b0011; // Move by +3
        enable_move = 1; #10;
        enable_move = 0; #10;
        $display("[RESULT] Position_q: %d  Position_d: %d\n", position_q, position_d);

       
        // TEST 5: Large movement to check multiple wraps 
        $display(">>> TEST 5: LARGE MOVE mod(5+4)=1 <<<");
        delta = 4'b0100; // Move by +4
        enable_move = 1; #10;
        enable_move = 0; #10;
        $display("[RESULT] Position_q: %d  Position_d: %d\n", position_q, position_d);

        // TEST 6: Reset 
        $display(">>> TEST 6: RESET <<<");
        clr_n = 0; #10;  // Apply reset
        clr_n = 1; #10;  
        $display("[RESULT] Position_q: %d  Position_d: %d\n", position_q, position_d);

        $display("===========================");
        $display(">>> TESTBENCH COMPLETE <<<");
        $display("===========================\n");

        $finish;
    end
endmodule

```

Results:
```
========================
>>> TESTBENCH START <<<
========================

>>> TEST FORMAT: mod8(pos_q + delta) = pos_d <<<
>>> TEST 1: ENABLE OFF <<<
[RESULT] Position: 0 (expected 2) Position_d: 0

>> ENABLE ON FORMAT: mod8(pos_q + delta) = pos_d <<
>>> TEST 2: MOVE FORWARD mod(0+2)=2 <<<
[RESULT] Position_q: 2  Position_d: 2

>>> TEST 3: WRAP AROUND mod(2+3)=5 <<<
[RESULT] Position_q: 5  Position_d: 5

>>> TEST 5: LARGE MOVE mod(5+4)=1 <<<
[RESULT] Position_q: 1  Position_d: 1

>>> TEST 6: RESET <<<
[RESULT] Position_q: 0  Position_d: 0

===========================
>>> TESTBENCH COMPLETE <<<
===========================
```


## Problem 3
Source code:
```verilog
// Filename candle_controller.sv
module candle_controller(
    input logic sys_clk,
    input logic clr_async,
    input logic clear_enable,
    input logic set_enable,
    input logic [2:0] pos_to_set,
    input logic [2:0] pos_to_clear,
    output logic [7:0] candle_state
    );
    
    always_ff @(posedge sys_clk, posedge clr_async) begin
    // Asynch active high clear
        if (clr_async) begin
        candle_state <= 8'b000000000;
        end else begin
        // Set enable high, use postion to set bit high in candle state
            if (set_enable) begin
                candle_state[pos_to_set] <= 1'b1;
            end
            // Clear enable high, check for seat and clear not overlapping
            if (clear_enable && (!set_enable && (pos_to_clear == pos_to_clear))) begin
                candle_state[pos_to_clear] <= 1'b0;
            end
       end
     end
endmodule
```

Testbench:
Signals:
sys_clk: The system clock signal.
clr_async: Asynchronous reset signal that clears the entire candle state.
clear_enable: Controls whether a candle position is cleared.
set_enable: Controls whether a candle position is set.
pos_to_set: Specifies which candle position to set.
pos_to_clear: Specifies which candle position to clear.
candle_state: The output representing the state of all the candles.



Test Cases:
Test 1: Asynchronous Reset:
    The reset is asserted by setting clr_async = 1 for 10 ns and then released (clr_async = 0).
    The state of candle_state is displayed after the reset.

Test 2: Set Candle at Position 0:
    set_enable = 1 and pos_to_set = 3'b000 to set the candle at position 0.
    After setting, the candle_state is displayed.

Test 3: Set Candle at Position 3:
    set_enable = 1 and pos_to_set = 3'b011 to set the candle at position 3.
    After setting, the candle_state is displayed.

Test 4: Clear Candle at Position 0:
    clear_enable = 1 and pos_to_clear = 3'b000 to clear the candle at position 0.
    After clearing, the candle_state is displayed.

Test 5: Set Precedence (Set Position 2, Clear Position 3):
    This test checks the precedence when both set_enable and clear_enable are asserted in the same cycle.
    set_enable = 1 and pos_to_set = 3'b010 to set position 2.
    clear_enable = 1 and pos_to_clear = 3'b011 to clear position 3.
    After this operation, candle_state is displayed, expecting position 2 to be set and position 3 to be uncleared.

Test 6: Set Precedence (Set Position 1, Clear Position 2):
    set_enable = 1 and pos_to_set = 3'b001 to set position 1.
    clear_enable = 1 and pos_to_clear = 3'b010 to clear position 2.
    After this operation, candle_state is displayed, expecting position 1 to be set and position 2 to be uncleared.

Test 7: Set Candle at Position 7 (Last Candle):
    set_enable = 1 and pos_to_set = 3'b111 to set position 7 (the last candle).
    After setting, the candle_state is displayed, confirming the correct state.

Test 8: Asynchronous Reset Again:
    This test checks if the system resets after performing operations.
   
```verilog
// Filename  tb_candle_controller.sv
`timescale 1ns / 1ps
module tb_candle_controller;

    // Testbench signals
    logic sys_clk;             // System clock signal
    logic clr_async;           // Asynchronous clear signal
    logic clear_enable;        // Clear enable signal
    logic set_enable;          // Set enable signal
    logic [2:0] pos_to_set;    // Position to set the candle
    logic [2:0] pos_to_clear;  // Position to clear the candle
    logic [7:0] candle_state;  // Output candle statea

    // Instantiate the candle_controller module
    candle_controller uut (
        .sys_clk(sys_clk),
        .clr_async(clr_async),
        .clear_enable(clear_enable),
        .set_enable(set_enable),
        .pos_to_set(pos_to_set),
        .pos_to_clear(pos_to_clear),
        .candle_state(candle_state)
    );

    // Clock generation
    always begin
        #5 sys_clk = ~sys_clk;  // Clock period of 10 units
    end

    // Test sequence
    initial begin
        // Initialize signals
        sys_clk = 0;
        clr_async = 0;
        clear_enable = 0;
        set_enable = 0;
        pos_to_set = 3'b000;
        pos_to_clear = 3'b000;

        $display("\n========================");
        $display(">>> TESTBENCH START <<<");
        $display("========================\n");
        
        // Reset the system
        $display(">>> TEST 1: Asynchronous Reset <<<");
        clr_async = 1; #10;  
        clr_async = 0; #10;
        $display("[RESULT] Candle state after reset: %b", candle_state);

        // Test 2: Set the first candle at position 0
        $display(">>> TEST 2: Set Candle at Position 0 <<<");
        set_enable = 1;
        pos_to_set = 3'b000;
        #10;  
        set_enable = 0;
        $display("[RESULT] Candle state: %b", candle_state);  

        // Test 3: Set candle at position 3
        $display(">>> TEST 3: Set Candle at Position 3 <<<");
        set_enable = 1;
        pos_to_set = 3'b011;
        #10;
        set_enable = 0;
        $display("[RESULT] Candle state: %b", candle_state); 
        // Test 4: Clear candle at position 0
        $display(">>> TEST 4: Clear Candle at Position 0 <<<");
        clear_enable = 1;
        pos_to_clear = 3'b000;
        #10;
        clear_enable = 0;
        $display("[RESULT] Candle state: %b", candle_state);  
        
        
        // Test 5: Clear candle at position 3 (conflict with set, precedence example)
        $display(">>> TEST 5: SetPrecedence |  Set at Position 2 & Clear at Position 3 <<<");
        set_enable = 1;
        pos_to_set = 3'b010;  // Set candle at position 2
        clear_enable = 1;
        pos_to_clear = 3'b011;  // Clear candle at position 3
        #10;
        set_enable = 0;
        clear_enable = 0;
        $display("[RESULT] Candle state: %b", candle_state);  // Expected: 00000100 (Position 2 set, Position 3 cleared)

        // Test 6: Set and Clear in the same cycle, set should have precedence
        $display(">>> TEST 6: SetPrecedence | Set at Position 1 & Clear at Position 2 <<<");
        set_enable = 1;
        pos_to_set = 3'b001;  
        clear_enable = 1;
        pos_to_clear = 3'b010;  
        #10;
        set_enable = 0;
        clear_enable = 0;
        $display("[RESULT] Candle state: %b", candle_state);  
       
     
        // Test 7: Set Candle at position 7 (last candle)
        $display(">>> TEST 7: Set Candle at Position 7 <<<");
        set_enable = 1;
        pos_to_set = 3'b111;
        #10;
        set_enable = 0;
        $display("[RESULT] Candle state: %b", candle_state);  // Expected: 10000110 (Position 7 set)

        // Test 8: Reset the system again
        $display(">>> TEST 8: Asynchronous Reset <<<");
        clr_async = 1;
        #10;
        clr_async = 0;
        $display("[RESULT] Candle state after reset: %b", candle_state);  // Expected: 00000000 (All candles off)
        
        $display("\n===========================");
        $display(">>> TESTBENCH COMPLETE <<<");
        $display("===========================\n");
        $finish;
    end

endmodule
```


Results:
```
========================
>>> TESTBENCH START <<<
========================

>>> TEST 1: Asynchronous Reset <<<
[RESULT] Candle state after reset: 00000000
>>> TEST 2: Set Candle at Position 0 <<<
[RESULT] Candle state: 00000001
>>> TEST 3: Set Candle at Position 3 <<<
[RESULT] Candle state: 00001001
>>> TEST 4: Clear Candle at Position 0 <<<
[RESULT] Candle state: 00001000
>>> TEST 5: SetPrecedence |  Set at Position 2 & Clear at Position 3 <<<
[RESULT] Candle state: 00001100
>>> TEST 6: SetPrecedence | Set at Position 1 & Clear at Position 2 <<<
[RESULT] Candle state: 00001110
>>> TEST 7: Set Candle at Position 7 <<<
[RESULT] Candle state: 10001110
>>> TEST 8: Asynchronous Reset <<<
[RESULT] Candle state after reset: 00000000

===========================
>>> TESTBENCH COMPLETE <<<
===========================
```


