# RTL design using Verilog with SKY130 Technology

![verilog-flyer](Verilog-flyer.png)

A 5 day cloud based virtual training workshop conducted by VSD-IAT from 23<sup>rd</sup> to 27<sup>th</sup> June. The link to the workshop webpage can be found [here](https://www.vlsisystemdesign.com/rtl-design-using-verilog-with-sky130-technology/). Below is a brief, day-wise documentation about the topics covered in the course, along with my implementations of the lab sessions.

## Table of Contents

- [Day 1 - Introduction to Verilog RTL Design and Synthesis](#day-1---introduction-to-verilog-rtl-design-and-synthesis)
  * [The Basics](#the-basics)
  * [Setting up the Lab Environment](#setting-up-the-lab-environment)
  * [Exploring Iverilog and GTKWave](#exploring-iverilog-and-gtkwave)
  * [Introduction to Yosys](#introduction-to-yosys)
  * [Libraries and their Significance](#libraries-and-their-significance)
  * [Exploring Yosys and SKY130PDKs](#exploring-yosys-and-sky130pdks)
- [Day 2 - Timing libs, Hierarchical vs. Flat Synthesis and Efficient Flop Coding Styles](#day-2---timing-libs-hierarchical-vs-flat-synthesis-and-efficient-flop-coding-styles)
  * [Further Information on .lib](#further-information-on-lib)
  * [Hierarchical vs. Flat Synthesis](#hierarchical-vs-flat-synthesis)
  * [Flop Coding Styles](#flop-coding-styles)
  * [Some Interesting Optimisations for Special Cases](#some-interesting-optimisations-for-special-cases)
- [Day 3 - Combinational and Seqeuntial Optimisations](#day-3---combinational-and-seqeuntial-optimisations)
  * [Introduction to Logic Optimisations](#introduction-to-logic-optimisations)
  * [Combinational Logic Optimisations](#combinational-logic-optimisations)
  * [Sequential Logic Optimisations](#sequential-logic-optimisations)
- [Day 4 - Gate Level Simulation, Blocking vs. Non-Blocking Assignments and Synthesis Simulation Mismatch](#day-4---gate-level-simulation-blocking-vs-non-blocking-assignments-and-synthesis-simulation-mismatch)
  * [Introduction to Gate Level Simulations](#introduction-to-gate-level-simulations)
  * [Understanding Synthesis Simulation Mismatches](#understanding-synthesis-simulation-mismatches)
  * [Exploring GLS and Mismatches with Iverilog](#exploring-gls-and-mismatches-with-iverilog)
- [](#)
  * [](#)

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

Icarus Verilog is an implementation of Verilog HDL and operates as a compiler for Verilog simulation. When a design and testbench file is fed to this simulator, it outputs a VCD or Value Change Dump file. This VCD file holds data about the changes in the inputs and outputs of the source design. To view the contents of the VCD file in a visually comprehensible manner, a Waveform Viewer tool is used. For our lab sessions, the tool used is GTKWave, which is a GTK+ based wave viewer tool.

### Setting up the Lab Environment

In order to set up the tool flow and files for running the lab sessions, the following commands are used.
```
mkdir vsd
git clone https://github.com/kunalg123/vsdflow.git
git clone https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop.git
```
These should add the necessary directories for the lab environment, including the sky130 standard cell libraries, its standard cell verilog models, as well as the source design and testbench verilog files for the lab sessions. Once the git cloning is succesful, the following base directories should be available on your file system.

![files after cloning](images/Day1/1-1.png)

For lab sessions, the following verilog design files and testbench models of some basic digital circuit components are available under the verilog_files directory.

![verilog files](images/Day1/1-2.png)

### Exploring Iverilog and GTKWave

To understand how to use these tools, lets explore Iverilog and GTKWave using an example of a simple 2:1 Multiplexer from the provided verilog_files directory. Let us take a look at the source verilog code for the design and testbench files, labeled as good_mux.v and tb_good_mux.v respectively.

![good_mux.v](images/Day1/1-7.png)<br>
*Fig.: Verilog code for 2:1 Multiplexer Design*

![tb_good_mux.v](images/Day1/1-6.png)<br>
*Fig.: Testbench for 2:1 Multiplexer*

To simulate these files in Iverilog, the following command can be used.

```
iverilog good_mux.v tb_good_mux.v
```

If done correctly, Iverilog should create an output file by the name of a.out which will be added in the file directory. To generate the VCD file, we must execute the a.out file as follows.

![iverilog execute](images/Day1/1-3.png)

Upon execution, a VCD file with the file extension .vcd will be generated. In our case, it is called tb_good_mux.vcd as that is the name specified in our test bench file. To view this VCD file in GTKWave, the following command is issued.

```
gtkwave tb_good_mux.vcd
```

This should generate the following response in the command terminal, as well as open up the GTKWave interface.

![gtkwave command](images/Day1/1-4.png)

Finally, in the GTKWave interface panel, we can add the unit under test and select the inputs and outputs whose waveforms we want to view. Now we can confirm if the waveform of our 2:1 Multiplexer matches the design specifications or not. As visible from the below waveform, it does athere to the design specifications.

![gtkwave waveform](images/Day1/1-5.png)

### Introduction to Yosys

Yosys is a framework for Verilog RTL synthesis. A synthesizer is a tool used for converting RTL based verilog code to netlist. RTL is the behavioural representation of the required specification in Verilog HDL. Netlist is the representation of the design in the form of standard cells present in the library. The Yosys synthesizer flow is as follows.

<img src="images/Day1/1-8.png" width="70%">

Yosys makes use of the commands ```read_verilog``` to read the verilog design, ```read_liberty``` to read the .lib, and ```write_verilog``` to write the netlist file. <br>

To verify the synthesis output, we can follow the same procedure as we did when verifying verilog design as the netlist must obey the same specifications as the original RTL design. In order to do this, we can pass the netlist file along with the original RTL testbench to our simulator and generate the VCD file. This VCD file can be viewed in the waveform viewer to confirm the behaviour of the synthesized netlist. This is shown below.

<img src="images/Day1/1-9.png" width="70%">

### Libraries and their Significance

A synthesizer conducts RTL to Gate level translation, wherein the behavioural design is converted to basic gates using the standard cell libraries provided, and connections are made between these gates. Libraries (.lib) are a collection of basic logic modules to implement any boolean logical functionalities, and may contain different flavours of the same gate such as 2-input/3-input or fast/slow.

**Need for Fast Cells:**

For a digital logic circuit, the combinational delay in the logic path determines its maximum speed of operation. Lets take an example of a basic combinational circuit shown below with two D Flip-flops and some combinational circuit betwen them, where CLK is the clock signal and DFF B holds the output of the circuit.

![combi ckt](images/Day1/1-10.png)

In this cicruit, the minimum size of 1 clock cycle is determined with the following relation

T<sub>CLK</sub> > T<sub>CQ_A</sub> + T<sub>COMBI</sub> + T<sub>SETUP_B</sub> <br>
where, <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;T<sub>CQ_A</sub> is the propogation delay of DFF A <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;T<sub>COMBI</sub> is the propogation delay of the combination circuit <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;T<sub>SETUP_B</sub> is the setup time for DFF B (min. time before the clock edge that input data must be supplied) <br>

Hence, for maximum performance we need a smaller value of T<sub>CLK</sub>, which can be achieved by using faster cells to reduce the value of T<sub>COMBI</sub> as much as possible.

**Need for Slow Cells:**

In order to prevent any hold violations, we need cells that work slower. If we consider the above example, there must be a minimum amount of time during which the output of the combinational circuit must be stable after the active edge of the clock, for DFF B to reliably capture the data at its input. This minimum delay is known as the Hold Time of the circuit. Hence, the following condition must be sastisfied to prevent hold violations.

T<sub>HOLD_B</sub> < T<sub>CQ_A</sub> + T<sub>COMBI</sub> <br>

Thus, we need fast cells to meet performance requirements as well as slow cells to meet hold times in the .lib collection. To pick appropriate cells, the user must offer "constraints" to the synthesizer.

Note: As the primary load in a digital circuit is capacitance, the charge/discharge times of the capacitor decides cell delay. To discharge capacitors fast we need transistors capable of sourcing more current, thus needing wider transistors with more area and power requirements. While slower cells need narrow transitors with less area and power requirements.

### Exploring Yosys and SKY130PDKs

Let us explore Yosys and the SKY130 libraries using the same example of the simple 2:1 Multiplexer from the previous sections. To start yosys, we must use the command ```yosys``` in the terminal. Once invoked, the yosys prompt should appear as follows.

![yosys prompt](images/Day1/1-11.png)

First, we must read the SKY130 libraries using the command ```read_liberty -lib filepath```. Next, we must read the design using the command ```read_verilog filename.v```. We must now specify the module name of the design we are synthesizing using the command ```synth -top modulename```. This can seen in the image below.

![yosys read](images/Day1/1-12.png)

Once this is done, we can generate the netlist using the command ```abc -liberty your_library_filepath```. For our case, this would be ```abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib```. Succesfully executing this command should return the following report in the yosys prompt.

![yosys report](images/Day1/1-13.png)

As visible in the report, yosys has found 3 inputs, 1 ouptut and 0 internal connections. This holds true for our example of the 2:1 Multiplexer. Further, yosys also mentions the cells used in the logic realisation. To observe a graphical view of the realisation, the command ```show``` can be used. This should generate the following graphic.

![yosys graphic](images/Day1/1-14.png)

Finally, we can write the netlist using the command ```write_verilog -noattr filename.v```. Here, the property "-noattr" is used to prevent yosys from dumping extra information in the final netlist file. Let's name our file as good_mux_netlist.v and execute the command. The final netlist represention is shown below.

![netlist](images/Day1/1-15.png)

## Day 2 - Timing libs, Hierarchical vs. Flat Synthesis and Efficient Flop Coding Styles

### Further Information on .lib

The SKY130 library file used for this workshop is sky130_fd_sc_hd_tt_025C_1v80.lib

Here, <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**fd** is SkyWater Foundry <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**sc** is Standard Cell <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**hd** is High Density <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**tt** is Typical Process <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**025C** is 25°C operating Temperature <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**1v80** is 1.8V operating Voltage <br>

Below is some of the contents of this .lib file.

![lib file](images/Day2/1-0.png)

Here, you can see that the library provides details like technology (CMOS), power parameters, voltage parameters, current draw, area, timings and delays, etc. This file hold information on every standard cell provided in the library, along with all its flavours. Let's compare the flavours of a basic OR gate.

![or gate](images/Day2/1-1.png)

From the above image it is evident that each iteration of the OR gate has different power and area consumptions. Larger are and power values are due to wider transistors which are required in faster designs. Hence, in the above case, or2_4 is the faster cell and or2_0 is the slower cell.

### Hierarchical vs. Flat Synthesis

When synthesizing a design containing multiple modules, the notion of heirarchical vs. flat design comes up. To understand their differrences, let's compare the two using an example. Below is a design that instantiates two low level modules, namely an OR gate (sub module u2) and an AND gate (sub module u1). This file is available under the verilog_files directory as multiple_modules.v

![multiple modules](images/Day2/1-2.png)

Now, let us synthesize this in Yosys with the following commands.
```
yosys
read_liberty -lib .../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog multiple_modules.v
synth -top multiple modules
abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```
![heir synth](images/Day2/1-3.png)

If we see the graphical view of the realisation using the command ```show multiple_modules```, we can notice that the actual OR and AND gates are not visible but only the sub modules u1 and u2 are shown. This is known as Heirarchical Synthesis as the heirarchies are preserved. This is shown below.

![heir image](images/Day2/1-4.png)

If we generate the netlist using ```write_verilog -noattr multiple_modules_h_netlist.v```, we see a similar story. The heirarchies are preserved, as there is one instantiation of each sub module under multiple_modules. The netlist is shown below.

![heir netlist](images/Day2/1-5.png)

To avoid heirarchical synthesis, we can use the command ```flatten``` in the yosys prompt after the ```synth``` command. Now, we can generate the netlist again using the command ```write_verilog -noattr multiple_modules_f_netlist.v```. This is shown in the image below.

![flatten yosys](images/Day2/1-6.png)

The netlist generated here does not contain instantiations of sub modules or any heirarchical structure. It directly contains one module with mutiple standard cells. This is known as Flat Synthesis. Its netlist is shown below.

![flat netlist](images/Day2/1-7.png)

If we view it graphically using the ```show``` command, we can observe only standard cell implementations and no heirarchy present.

![flat image](images/Day2/1-8.png)

Note: From a multiple module design file, it is possible to just synthesize a single sub module. This is done using the command ```synth -top sub_module_name```, and is known as Sub Module level Synthesis. This is used when, 
- we have multiple instances of the same module and just want to synthesize it once, then replicate it however many times.
- we are using a divide and conquer approach (used in massive designs when the tool does not do an appropriate or well optmised job).

### Flop Coding Styles

In combinational circuits, each circuit element or cell experiences a time delay for the output to change based on a change in the input. This delay is known as Propogation Delay. Due to these propogation delays, the circuit might experience unwanted transitions in the output, espescially as the propagation delay stacks additively as the number of combinational circuits increase. These unwanted transitions are known as Glitches in the output. <br>

To avoid glitches, we make use of D Flip-flops as storage elements or buffers in between the different combinational circuits. D flip-flops store the value present on their input, and its output changes only at clock edges. This brings stability between combinational circuits as the D flip-flops shield the combinational circuit they are feeding against glitches in their input, allowing the output of that combinational circuit to settle down. <br>

Flip-flops come in various types. These are mainly:
- Synchronous vs. Asynchronous set
- Synchronous vs. Asynchronous reset
- Rising (positive) edge triggered vs. Falling (negative) edge triggered

To further understand understand the different flop styles, let's look at 3 D flip-flops available to us in the directory verilog_files.

Note: For synthesizing designs involving D flip-flops in Yosys, we must use the command ```dfflibmap -liberty dff_library_filepath``` which in our case is ```dfflibmap -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib```. This command is used to read the dff standard cells as some libraries may have seperate .lib files for these. The command must be issues after ```synth``` and before ```abc```.

**1. Rising edge D Flip-flop with asynchronous reset**

```
module dff_asyncres ( input clk ,  input async_reset , input d , output reg q );
always @ (posedge clk , posedge async_reset)
begin
	if(async_reset)
		q <= 1'b0;
	else	
		q <= d;
end
endmodule
```
*Verilog code for dff_asyncres**

![asyncres wave](images/Day2/1-9.png) <br>
*Waveform for dff_asyncres*

![asyncres img](images/Day2/1-12.png) <br>
*Graphical representation of synthesized netlist for dff_asyncres*

**2. Rising edge D Flip-flop with asynchronous set**

```verilog
module dff_async_set ( input clk ,  input async_set , input d , output reg q );
always @ (posedge clk , posedge async_set)
begin
	if(async_set)
		q <= 1'b1;
	else	
		q <= d;
end
endmodule
```
*Verilog code for dff_sync_set**

![async_set wave](images/Day2/1-10.png) <br>
*Waveform for dff_async_set*

![async_set img](images/Day2/1-13.png) <br>
*Graphical representation of synthesized netlist for dff_async_set*

**3. Rising edge D Flip-flop with synchronous reset**

```verilog
module dff_syncres ( input clk , input async_reset , input sync_reset , input d , output reg q );
always @ (posedge clk )
begin
	if (sync_reset)
		q <= 1'b0;
	else	
		q <= d;
end
endmodule
```
*Verilog code for dff_syncres**

![syncres wave](images/Day2/1-11.png) <br>
*Waveform for dff_syncres*

![syncres img](images/Day2/1-14.png) <br>
*Graphical representation of synthesized netlist for dff_syncres*

### Some Interesting Optimisations for Special Cases

**Case 1:**

Let's consider the following design where the 3 bit input is multiplied by 2 and the output is a 4 bit value.

```verilog
module mul2 (input [2:0] a, output [3:0] y);
	assign y = a * 2;
endmodule
```

If we take a look at its truth table, we can see the following.

|a[2:0]|y[3:0]|
|---|---|
|000|0000|
|001|0010|
|010|0100|
|011|0110|
|100|1000|
|101|1010|
|110|1100|
|111|1110|

Here, the ouput y[3:0] is nothing but the input a[2:0] appended with a 0 at the LSB. Or, we can say that ```y = {a, 0}```. If we synthesize the netlist and look at its graphical realisation, we will see the same optimisation occuring in the netlist.

![2x mul](images/Day2/1-15.png)

**Case 2:**

Let's consider the following design where the 3 bit input is multiplied by 9 and the output is a 6 bit value.

```verilog
module mult8 (input [2:0] a , output [5:0] y);
	assign y = a * 9;
endmodule
```

If we take a look at its truth table, we can see the following.

|a[2:0]|y[5:0]|
|---|---|
|000|000000|
|001|001001|
|010|010010|
|011|011011|
|100|100100|
|101|101101|
|110|110110|
|111|111111|

Here, the ouput y[5:0] is nothing but the input a[2:0] appended with itself. Or, we can say that ```y = {a, a}```. If we synthesize the netlist and look at its graphical realisation, we will see the same optimisation occuring in the netlist.

![9x mul](images/Day2/2-16.png)

## Day 3 - Combinational and Seqeuntial Optimisations

### Introduction to Logic Optimisations

There are broadly two types of logic available, combinational and sequential. In order to save cost by reducing power and area consumptions of a design, we must optimise the design as best as possible. For each type of logic, there exist different methods of opimisation, as follows.

**1. Combinational optimisation methods:**

- Squeezing the logic for Area and Power savings
- Constant Propogation
  * Direct Optimisation
- Boolean Logic Optimisation
  * K-Map
  * Quine-McKluskey Algorithm


**2. Sequential optimisation methods:**

- Basic
  * Sequential Constant Propogation
- Advanced
  * State Optimisation
  * Retiming
  * Sequential Logic Cloning (Floor Plan Aware Synthesis)

### Combinational Logic Optimisations

Let's take a look at some examples of combinational optimisations using the files opt_check.v, opt_check2.v, opt_check3.v, opt_check4.v, and multiple_modules_opt.v. All of these files are under the verilog_files directory.

**Example 1: opt_check.v**

```verilog
module opt_check (input a , input b , output y);
	assign y = a?b:0;
endmodule
```
Using boolean logic simplification, we can tell that y = ab. Let us synthesize this in yosys using the following commands.

![yosys cmd](images/Day3/3-0.png)

Before realising the netlist, we must issue a command to yosys to perform constant propogation and optimisations. this can be done using the ```opt_clean -purge``` command as follows.

![yosys purge](images/Day3/3-1.png)

After this step, we can continue as usual with ```abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib``` and ```write_verilog -noattr opt_check_netlist.v``` commands. If we view the graphical realisation witht the ```show``` command, we can see that Yosys has synthesized an AND gate as expected.

![opt1 show](images/Day3/3-2.png)

**Example 2: opt_check2.v**

```verilog
module opt_check2 (input a , input b , output y);
	assign y = a?1:b;
endmodule
```

Similar to the example 1, lets continue with optimisations for this design. Here we expect the output to be an OR gate based on boolean optimisation, since the output can be simplified to y = a + b. If we generate the netlist and look at its graphical realisation, we get the following.

![opt2 show](images/Day3/3-3.png)

Here, we expected an OR gate but got an iso low power cell instead. This is due to certain Yosys optimisations, however it functions the same.

**Example 3: opt_check3.v**

```verilog
module opt_check3 (input a , input b, input c , output y);
	assign y = a?(c?b:0):0;
endmodule
```

For this design, we expect the output to be a 3 input AND gate based on boolean optimisation, as the output can be simplified to y = abc. If we generate the netlist and look at its graphical realisation, we get the following.

![opt3 show](images/Day3/3-4.png)

As we can see, Yosys synthesizes a 3 input AND after optimisations as expected.

**Example 4: opt_check4.v**

```verilog
module opt_check4 (input a , input b , input c , output y);
	assign y = a?(b?(a & c):c):(!c);
endmodule
```

For this design, after boolean logic optimisation we can conclude that the output can be simplified to a single xnor gate with the output equation y = a⊙c. If we generate the netlist and look at its graphical realisation, we get the following.

![opt4 show](images/Day3/3-5.png)

We get the same result after the Yosys synthesis as we expected from the optimisation.

**Example 5: multiple_module_opt.v**

```verilog
module sub_module1(input a , input b , output y);
	assign y = a & b;
endmodule

module sub_module2(input a , input b , output y);
	assign y = a^b;
endmodule

module multiple_module_opt(input a , input b , input c , input d , output y);
wire n1, n2, n3;

sub_module1 U1 (.a(a), .b(1'b1), .y(n1));
sub_module2 U2 (.a(n1), .b(1'b0), .y(n2));
sub_module2 U3 (.a(b), .b(d), .y(n3));

assign y = c | (b & n1); 

endmodule
```

To optimise this design, we must use Flat Synthesis as otherwise the optimisations will not be performed on the sub module level. Thus, we must use the ```flatten``` command.

![multopt show](images/Day3/3-6.png)

Once the optimisations are conducted by Yosys, we can observe that we only need a single standard cell to realise a design that originally contained multiple modules.

### Sequential Logic Optimisations

Let's take a look at some examples of sequential optimisations using sequential constant propogation. We shall be using the files dff_const1.v, dff_const2.v, dff_const3.v, dff_const4.v, and dff_const5.v. All of these files are under the verilog_files directory.

**Example 1: dff_const1.v**

```verilog
module dff_const1(input clk, input reset, output reg q);
always @(posedge clk, posedge reset)
begin
	if(reset)
		q <= 1'b0;
	else
		q <= 1'b1;
end
endmodule
```

At first glance, it may seem that the output bit q should be equal to an inverted reset or !reset. However, as the reset is synchronous, so the output depends on both the reset and clk edge. This can be confirmed by simulating the design in Iverilog, and viewing the VCD with GTKWave as follows.

![dff1 sim](images/Day3/3-7.png)

![dff1 wave](images/Day3/3-8.png)

If we observe the waveform above, when reset becomes 0, q only becomes 1 at the next clock edge. Hence, we do not get a sequential constant, and no optimisations should be possible here. Let's confirm the same using Yosys synthesis and optimisation as follows.

![dff1 cmd](images/Day3/3-9.png)

We must use the command ```dfflibmap -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib``` as our design includes D flip-flops. We can then generate the netlist using ```abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib``` and ```write_verilog -noattr dff_const1_netlist.v```. To view the graphical realisation, we use the ```show``` command.

![dff1 show](images/Day3/3-10.png)

As you can see, no optimisations can be conducted on this design.

**Example 2: dff_const2.v**

```verilog
module dff_const2(input clk, input reset, output reg q);
always @(posedge clk, posedge reset)
begin
	if(reset)
		q <= 1'b1;
	else
		q <= 1'b1;
end
endmodule
```

Here, we can see that regardless of the inputs, the ouput q always remains constant at 1. This can be observed in the waveform viewer as well.

![dff2 wave](images/Day3/3-11.png)

As the output is always constant, it can easily be opitmised using Yosys as below.

![dff2 show](images/Day3/3-15.png)

**Example 3: dff_const3.v**

```verilog
module dff_const3(input clk, input reset, output reg q);
reg q1;

always @(posedge clk, posedge reset)
begin
	if(reset)
	begin
		q <= 1'b1;
		q1 <= 1'b0;
	end
	else
	begin
		q1 <= 1'b1;
		q <= q1;
	end
end
endmodule
```

Here, we might think that the output q should always be constant at the value 1. For an ideal circuit, this may be true. When reset is 0 then q1 should be 1, making the output q to be 1 as well. But when we consider the propogation delay time of D flip-flop q1, the output of q1 = 1 will not be present exactly at th clock edge. Thus, q takes the value 0 until the next clock edge whne it read an input of 1 from q1. This is confirmed with the simulated waveform below.

![dff3 wave](images/Day3/3-12.png)

Hence, both the flip-flops are needed and no optimisation can be conducted on this particular design. We can confirm this using Yosys as shown below.

![dff3 show](images/Day3/3-16.png)

As you can see, both the D flip-flops are present in the synthesized netlist.

**Example 4: dff_const4.v**

```verilog
module dff_const4(input clk, input reset, output reg q);
reg q1;

always @(posedge clk, posedge reset)
begin
	if(reset)
	begin
		q <= 1'b1;
		q1 <= 1'b1;
	end
	else
	begin
		q1 <= 1'b1;
		q <= q1;
	end
end
endmodule
```

Here, we can see that regardless of the reset input, q1 is always going to be constant at 1. As q can only be 1 or q1 depending on the reset input, but q1 = 1. Thus q is also constant at the value 1. We can confirm this with the simulated waveforms as shown below.

![dff4 wave](images/Day3/3-13.png)

As the output is always constant, it can easily be opitmised using Yosys as shown in the graphical realisation below.

![dff4 show](images/Day3/3-17.png)

Using optimisation, Yosys has concurred that no D flip-flops are required in the netlist.

**Example 5: dff_const5.v**

```verilog
module dff_const5(input clk, input reset, output reg q);
reg q1;

always @(posedge clk, posedge reset)
begin
	if(reset)
	begin
		q <= 1'b0;
		q1 <= 1'b0;
	end
	else
	begin
		q1 <= 1'b1;
		q <= q1;
	end
end
endmodule
```

Again, we might expect this design to be easily optimised as q = !reset. However as the design uses a synchronous reset, as well as due to propogation delay time of D flip-flop q1 (similar to example 3), we cannot replace the flip-flops. Hence, we do not get sequential constants. This can be viewed in the simulated waveforms as well.

![dff5 wave](images/Day3/3-14.png)

Since both D flip-flops cannot be replaced, no optimisation should be possible in this design. Let's look at the Yosys graphical realisation for this file.

![dff5 show](images/Day3/3-18.png)

As expected, no optimisations could be conducted by Yosys.

## Day 4 - Gate Level Simulation, Blocking vs. Non-Blocking Assignments and Synthesis Simulation Mismatch

### Introduction to Gate Level Simulations

Gate Level Simulation (GLS) is the process of executing the netlist as the unit under test, instead of the source design file as we conducted up until now. We can make use of the original testbench itself to simulate the behaviour of the netlist, as the netlist is logically the same as the source RTL code.

GLS is an important procedure as we must ensure that the netlist meets the design specifications. Unlike the RTL code which uses logic, the netlist utilizes physical gates to realise the same logic. These gates introduce timing concerns like propogation delays and hold times. Hence, using gate level simulations, we can ensure that the timings of the design are met as per the specifications.

The flow for gate level simulations using Iverilog is shown below.

<img src="images/Day4/4-0.png" width="70%">

Note: For timing validation, we must run GLS with gate level models that are delay annotated (Timing Aware GLS).

### Understanding Synthesis Simulation Mismatches

If the netlist is a true representation of the RTL code, then why must we verify the functionality of the netlist? This is because sometimes there are mismatches between the pre-synthesis simulations and the post-synthesis simulations. This is known as synthesis simulation mismatch and can occur due to, but not limited to, the following reasons:

- Missing sensitivity list
- Blocking and non-blocking assignments
- Non-standard verilog coding

Let us look into the above issues using some example cases.

**Missing Sensitivity List:**

As we have learnt earlier, a simulator only updates or changes its output when it finds a change in the input. The verilog code below describes a 2:1 Multiplexer. There are 3 inputs i0, i1, and sel and 1 output y.

```verilog
module mux(input i0, input i1, input sel, output reg y);

always @(sel)
begin
	if (sel)
	begin
		y = i1;
	end
	else 
	begin
		y = i0;
            
	end
end
endmodule
```

In this example the simulator checks for changes in the input sel to update the value of the output y, as the always block is evaluated only for sel. If sel were constant at 0 and i0 were to change its value, we would observe no change in the output y. This means the design would function more like a latch instead of a multiplexer. If we were to synthesize this design however, we would observe a multiplexer in our netlist instead of latch like behaviour. This would cause synthesis simulation mismatch.

This can be fixed by replacing the ```always @(sel)``` line with ```always @(*)``` instead which looks for changes in any input.

**Blocking and Non-Blocking Statements:**

Within the ```always``` block in a verliog code, assignments can be of two types:

- Blocking statements:
  * These are assignments that make use of the ```=``` operator.
  * Here, statements are always executed in the order that they are written.
- Non-blocking statements:
  * These are assignments that use the ```<=``` operator.
  * Here, all RHS blocks are evaluated parallelly when the ```always``` block is entered, and then assigned to the LHS.
 
Let's look at some examples of blocking statements and how they can cause synthesis simulation mismatches.
 
Below, we have verilog code for a serial shift register with input d and output q1. As the code using blocking statements, the line ```q0 = d;``` will execute before ```q1 = q0;```. This means that the value at d will direectly be updated at q1 at every clock edge, and the design will function as a single D flip-flop instead of a 2 bit shift register.

 ```verilog
module code (input clk, input reset, input d, output reg q1);
reg q0;

always @(posedge clk,posedge reset)
begin
	if(reset)
	begin
		q0 = 1'b0;
        	q1 = 1'b0;
	end
	else
	begin
        	q0 = d;
		q1 = q0;
        
	end
end
endmodule
```
This problem will cause a mismatch in the pre-synthesis and post-synthesis simulations as the netlist will still function as a 2 bit shift register. This can be fixed by either writing the line ```q1 = q0;``` before the line ```q0 = d;``` or by using non-blocking statements instead as now both q0 and q1 would be evaluated parallelly regardless of their position. Hence, the non-blocking operator ```<=``` is preferred when using sequential designs.

Let's look at another example. Here we have the verilog code for some combinational logic.

```verilog
module code (input a, input b, input c, output reg y);
reg q0;

always @(*)
begin
	y = q0 & c;
	q0 = a|b;
        
end 
endmodule
```

Similar to the earlier example, the AND assignment would occur before the OR assignment. This means the output y would get a delayed value of a|b, as the value of q0 would get updated after the output y is updated. This too would cause synthesis and simulation mismatches, and can be fixed by writing the line ```q0 = a|b;``` before ```y = q0 & c;```.

### Exploring GLS and Mismatches with Iverilog

To further understand gate level simulations and mismatches between pre-synthesis and post-synthesis simulation, let us try some examples in Iverilog.

**Example 1:**

Below is the verilog code for a 2:1 multiplexer using a ternary operator. Let's attempt gate level simulations on this design.
 
```verilog
module ternary_operator_mux (input i0 , input i1 , input sel , output y);
	assign y = sel?i1:i0;
endmodule
```

First, we must synthesize this file using Yosys and generate its netlist. We can view the graphical realisation of the same below.

!(ternarymux show)[images/Day4/4-1.png]

Now that we have the netlist, we can run the GLS using Iverilog by specifying the gate level models with the following command.

```
iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v ternary_operator_mux_net.v tb_ternary_operator_mux.v
```

![gls cmd](images/Day4/4-2.png)

We can view the simulated waveforms using GTKWave and verify that the generated netlist does behave like a 2:1 multiplexer.

![ternarymux wave](images/Day4/4-3.png)

**Example 2:**

Here, we have the verilog code for the file bad_mux.v which is available in the directory verilog_files.

```verilog
module bad_mux (input i0 , input i1 , input sel , output reg y);
always @(sel)
begin
	if(sel)
		y <= i1;
	else 
		y <= i0;
end
endmodule
```

This code is similar to one of the earlier examples discussed. It should behave more like a latch than a 2:1 multiplexer in the pre-synthesis simulation due to the missing sensitivity list. This can be confirmed by simulating the design in Iverilog and viewing its waveform.

![badmux wave](images/Day4/4-4.png)

As we can see, the design does not function as a multiplexer, and the output only gets updated when there is a change in the value of the input sel. However, if we try to generate its netlist using yosys, we can see below that the synthesized netlist contains a multiplexer and not a latch like circut.

![badmux show](images/Day4/4-5.png)

If we now run gate level simulation on the synthesized netlist and view the waveform, we should see that it functions exactly like a 2:1 multiplexer.

![badmux gls show](images/Day4/4-6.png)

This is known as synthesis simulation mismatch since the simulation of the design and the simulation of the synthesized netlist are different.

**Example 3:**

Let's take a look at an example of mismatch due to blocking statements. We shall use the verilog design file blocking_caveat.v which is available in the directory verilog_files. The code for the same is as follows.

```verilog
module blocking_caveat (input a , input b, input c, output reg d); 
reg x;

always @(*)
begin
	d = x & c;
	x = a | b;
end
endmodule
```

Similar to an earlier example, we should get a synthesis simulation mismatch. This is because of the blocking statements used, as the output d will always be evaluated before x. If we simulate this file in Iverilog, the simulated waveform should tell the same story.

![blocking wave](images/Day4/4-7.png)

If we observe the instance of time highlighted by the cursor, we can see that the output y holds the value 1. However it should hold the value 0, as both the inputs a and b are 0. This means a | b should output 0, which when AND with c, should give an output of 0. But due to the blocking statements used, x actually holds a 1 tick delayed value of a | b, hence giving us an incorrect output.

Now, let's try generating the netlist for this design and viewing its graphical realisation using Yosys.

![blocking show](images/Day4/4-8.png)

As we can see, the netlist does not include any latches to hold delayed values. It only includes an OR 2 AND gate. If we run gate lavel simulations on this netlist in Iverilog, we should see the following waveform.

![blocking gls wave](images/Day4/4-9.png)

Here, we can observe that the output looks at the present value of inputs, and not the past values like in the pre-synthesis simulation. Thus, we get a synthesis simulation mismatch due to blocking statements.

## Day 5 - If, Case, For Loop and For Generate

### IF Constructs

Similar to regular programming languages, IF statements in verilog also follow certain priority logic. But, verilog is a hardware description language, so these conditions directly affect the physical gates synthesized. Incorrect usage of IF conditions can cause certain optimisation issues in the synthesis of a design. 

Let's look at a template for an IF block. Here we have an output y that gets assigned to certain statements bases on IF conditionals.

```verilog
if (cond_1)
begin
	y = statement_1;
end
else if (cond_2)
begin
	y = statement_2;
end
else if (cond_3)
begin
	y = statement_3;
end
else
begin
	y = statement_4;
end
```

In this code block, we would assume that the synthesized netlist might contain a single multiplexer. However, the first IF statement holds the highest priority in this block of code. This means that if cond_1 is satisfied, we do not enter the next IF statements. Thus we get a ladder like multiplexer structure in the final design instead of a single multiplexer, which is shown below.

![if block](images/Day5/5-0.png)

Another issue can arise from incorrect IF statements when we use incomplete IF statements. Incomplete IF statements occur when IF blocks do not end with an accompanying else block. This is seen in the verilog code block below.

```verilog
if (cond_1)
begin
	y = statement_1;
end
else if (cond_2)
begin
	y = statement_2;
end

...
```

Here, we have not specified what happens if both cond_1 and cond_2 are false. This is an incomplete IF statement. When the synthesizer comes accross this, it will add an Inferred Latch in the final synthesis. This can be seen in the diagram below.

![incomplete if](images/Day5/5-1.png)

Since the tool does not know what to do when both conditions are false, it will infer a latch to store the latest value of the output. When both conditions are false, the stored value in the latch will be driven to the output.

We must always take into consideration what hardware will our verilog code directly be translated to. Sometimes however, incomplete IF constructs are perfectly fine in cases such as counters where latches must store the previous output as the input when no enable condition is found.

### CASE Constructs

Let's look at the following verilog code block. Here, the inferred hardware would be a 4:1 multiplexer. Note that CASE statements do not have prioiry logic like IF statements.

```verilog
always @(*)
begin
	case(sel)
		2'b00: begin
		       y = statement_1;
		       end
		2'b01: begin
		       y = statement_2;
		       end
		2'b10: begin
		       y = statement_3;
		       end
		2'b11: begin
		       y = statement_4;
		       end
	endcase
end
```

Some caveats with using CASE statements:

**1. Incomplete CASE.**

```verilog
always @(*)
begin
	case(sel)
		2'b00: begin
		       y = statement_1;
		       end
		2'b01: begin
		       y = statement_2;
		       end
	endcase
end
```

This occurs when some cases are not specified inside the CASE block. For example, if the 2'b10 and 2'b11 cases were not mentioned, the tool would synthesize inferred latches at the 3rd and 4th inputs of the multiplexer. To avoid this, we can make use of the ```default:``` case inside the CASE block so that the tool knows what to do when a case that is not specified occurs.

**2. Partial assignments**

```verilog
always @(*)
begin
	case(sel)
		2'b00: begin
		       x = a;
		       y = b;
		       end
		2'b01: begin
		       x = c;
		       end
	      default: begin
	      	       x = d;
		       y = d;
		       end
	endcase
end
```

In the above example, we have 2 outputs x and y. This will create two 4:1 multiplexers, one for each output. If we look at case 2'b01, we have specified the value of x for this case, but not the value of y. We might assume that it is okay to do so, as a default case is specified for both the outputs, and if we don't directly specify the value of y, the tool will imply the default case. This, however, is incorrect. In partial assignments such as this, the tool will infer a latch at the 2nd input for multiplexer y as no value is specified.

**3. Overlapping cases**

```verilog
always @(*)
begin
	case(sel)
		2'b00: begin;
		       y = a;
		       end
		2'b01: begin
		       y = b;
		       end
		2'b10: begin
		       y = c;
		       end
		2'b1?: begin
		       y = d;
		       end
	endcase
end
```

In the above code block, 2'b1? specifies that the LSB can be either 0 or 1. This means when the sel input is holding a value 3, conditions 3 and 4 both hold true. If we used an IF condition here, due to priority logic, condition 4 would be ignored when condition 3 is met. However, in the CASE statement, both conditions would hold true as there is no priority logic, and we would get an unpreidctable output. This is known as an overlapping case.

### Synthesizing Incorrect IF and CASE constructs

**Example 1:**

Below is the file titled incomp_if.v, and can be found in the directory verilog_files.

```verilog
module incomp_if (input i0 , input i1 , input i2 , output reg y);
always @ (*)
begin
	if(i0)
		y <= i1;
end
endmodule
```

The code contains an incomplete IF statement as no else condition is mentioned. As we have learnt, we should see latch like behaviour in the simulation. Let's simulate this design using the following commands.

![incif cmd](images/Day5/5-3.png)

![incif wave](images/Day5/5-4.png)

From the above waveform, we can observe that when i0 vecomes low, the output y holds the previous value of input i1. This shows latch like behaviour, and can further be detailed by looking at the synthesis output using yosys.

![incif show](images/Day5/5-5.png)

As we can see, an inferred latch is created in the synthesized netlist.

**Example 2:**

Let's look at a similar example of incomp_if2.v below.

```verilog
module incomp_if2 (input i0 , input i1 , input i2 , input i3, output reg y);
always @ (*)
begin
	if(i0)
		y <= i1;
	else if (i2)
		y <= i3;

end
endmodule
```

The above code contains an incomplete IF statement as well. Here, we have 2 inputs i1 and i3, as well as 2 conditional inputs i0 and i2. As we do not specify what happens to the output y when both i0 and i2 go low, we will get an issues in the final synthesis. Let us look at its simulated waveform.

![incif2 wave](images/Day5/5-6.png)

As you can see, when both i0 and i2 are low, the output y depicts latch like behaviour. This can be verified by checking the graphical realisation of the Yosys sysnthesis below. Yosys synthesized a multiplexer as well as a latch with some combinational logic for its enable pin.

![incif2 show](images/Day5/5-7.png)

**Example 3:**

```verilog
module incomp_case (input i0 , input i1 , input i2 , input [1:0] sel, output reg y);
always @ (*)
begin
	case(sel)
		2'b00 : y = i0;
		2'b01 : y = i1;
	endcase
end
endmodule
```

**Example 4:**

```verilog
module comp_case (input i0 , input i1 , input i2 , input [1:0] sel, output reg y);
always @ (*)
begin
	case(sel)
		2'b00 : y = i0;
		2'b01 : y = i1;
		default : y = i2;
	endcase
end
endmodule
```
follows i2 at default case

**Example 5:**

```verilog

module partial_case_assign (input i0 , input i1 , input i2 , input [1:0] sel, output reg y , output reg x);
always @ (*)
begin
	case(sel)
		2'b00 : begin
			y = i0;
			x = i2;
			end
		2'b01 : y = i1;
		default : begin
		           x = i1;
			   y = i2;
			  end
	endcase
end
endmodule
```

**Example 6:**

```verilog
  
module bad_case (input i0 , input i1, input i2, input i3 , input [1:0] sel, output reg y);
always @(*)
begin
	case(sel)
		2'b00: y = i0;
		2'b01: y = i1;
		2'b10: y = i2;
		2'b1?: y = i3;
		//2'b11: y = i3;
	endcase
end

endmodule
```

synth sim mismatch.

### Introduction to Looping Constructs

There are two distinct uses of FOR loops in verilog design, as follows.

**1. FOR loop**
  * Used within the ```always``` block
  * Used to evaluate expressions


**2. generate FOR loop**
  * Only used outside the ```always``` block
  * Used for instantiating hardware

### FOR 
(evaluating multiple assignments)

### FOR Generate

examples

ex 1:

```verilog
module mux_generate (input i0 , input i1, input i2 , input i3 , input [1:0] sel  , output reg y);
wire [3:0] i_int;
assign i_int = {i3,i2,i1,i0};
integer k;

always @ (*)
begin
	for(k = 0; k < 4; k=k+1) begin
		if(k == sel)
			y = i_int[k];
	end
end
endmodule
```

instantiate fa in loop

Rules for addition

N and N bit number --> Sum will be N+1 bit
N and M bit number --> Sum will be Max(N,M)+1 bit

