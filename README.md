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
- [Day 2 - Timing libs, Hierarchical vs. Flat Synthesis and Efficient Flop Coding Styles](#day-2---timing-libs,-hierarchical-vs.-flat-synthesis-and-efficient-flop-coding-styles)

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
