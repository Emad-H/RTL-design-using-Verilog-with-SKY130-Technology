# RTL design using Verilog with SKY130 Technology

![verilog-flyer](Verilog-flyer.png)

A 5 day cloud based virtual training workshop conducted by VSD-IAT from 23<sup>rd</sup> to 27<sup>th</sup> June. The link to the workshop webpage can be found [here](https://www.vlsisystemdesign.com/rtl-design-using-verilog-with-sky130-technology/). Below is a brief, day-wise documentation about the topics covered in the course, along with my implementations of the lab sessions.

## Table of Contents

- [Day 1 - Introduction to Verilog RTL Design and Synthesis](#day-1---introduction-to-verilog-rtl-design-and-synthesis)
  * [The Basics](#the-basics)
  * [Setting up the Lab Environment](#setting-up-the-lab-environment)
  * 

## Day 1 - Introduction to Verilog RTL Design and Synthesis

### The Basics
**Design:** <br>
The actual Verilog code or set of Verilog codes containing the intended functionalities of the hardware model to meet the required specs. Below is an example of a simple latch design written in Verilog HDL.
```verilog
module latch (input clk , input reset , input d , output reg q);
always @ (clk,reset,d)
begin
	if(reset)
		q <= 1'b0;
	else if(clk)
		q <= d;
end
endmodule
```

**Testbench:** <br>
A program used for the purposes of verifying the functional correctness of the design. In a test bench, stimuli (test vectors) are set up to check whether the design meets the required specifications. Below is an example test bench written written in Verilog to test the above latch design.
```verilog
`timescale 1ns / 1ps
module tb_latch;
	// Inputs
	reg clk, reset, d;
	// Outputs
	wire q;

        // Instantiate the Unit Under Test (UUT)
	latch uut (
		.clk(clk),
		.reset(reset),
		.d(d),
		.q(q)
	);

	initial begin
	$dumpfile("tb_latch.vcd");
	$dumpvars(0,tb_latch);
	// Initialize Inputs
	clk = 0;
	reset = 1;
	d = 0;
	#500 $finish;
	end

always #25 clk = ~clk;
always #30 d = ~d;
always #15 reset=0;
endmodule
```

**Testbench Setup** <br>

<img src="images/Day1/1-0.png" width="70%">
Note: The design may have 1 or more primary inputs and 1 or more primary outputs, however a testbench does not have any primary inputs or outputs. <br>

<br>**Simulator:** <br>
It is the tool used to simulate the design, and check for its adherence to the specifications. They can be used to apply the test bench to the design. The simulator tool used for this workshop is Icarus Verilog (Iverilog). A simulator works by looking for changes on the input signals, and evaluating the output signals only when a change in value is observed on the input. Below is the simulation flow for Iverilog.
<img src="images/Day1/1-01.png" width="70%">

### Setting up the Lab Environment

```
mkdir vsd
git clone https://github.com/kunalg123/vsdflow.git
git clone https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop.git
```

![files after cloning](images/Day1/1-1.png)
