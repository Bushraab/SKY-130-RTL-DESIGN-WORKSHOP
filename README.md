# SKY-130-RTL-DESIGN-WORKSHOP
![image](https://user-images.githubusercontent.com/86364063/123573872-6adc2c80-d7ec-11eb-9bc5-900743f4ea6c.png)
# TABLE OF CONTENTS
# Day 1 - Introduction to Verilog RTL Design and Synthesis

* Introduction to iverilog design test bench

* Labs using Iverilog and GTKWave

* Introduction to Yosys

* Libraries and their Significance

* Exploring Yosys and SKY130PDKs

# Day 2 - Timing libs, Hierarchical vs. Flat Synthesis and Efficient Flop Coding Styles

* Further Information on .lib

* Hierarchical vs. Flat Synthesis

* Flop Coding Styles

* Some Interesting Optimisations for Special Cases

# Day 3 - Combinational and Seqeuntial Optimisations

* Introduction to Logic Optimisations

* Combinational Logic Optimisations

* Sequential Logic Optimisations

# Day 4 - Gate Level Simulation, Blocking vs. Non-Blocking Assignments and Synthesis Simulation Mismatch

* Introduction to Gate Level Simulations

* Understanding Synthesis Simulation Mismatches

* Exploring GLS and Mismatches with Iverilog

# Day 5 If, case,for loop and for generate

* If case constructs

* Labs on Incomplete If case

* Incomplete Overlapping case

* FOR loop vs for generate statements

# DAY 1: Introduction to Verilog RTL Design and Synthesis

The first day covers the basics of RTL Design, Testbench, Simulation and Synthesis. Open-Source softwares like iverilog (simulator) and YOSYS (Synthesis) are provided through remote access in the portal to practice labs

RTL Design - It consists of an actual verilog code / a set of verilog codes that have the functionality to meet the required design specifications of the circuit TestBench - Testbench is a setup that one uses to apply a set of stimuli (test-case vector) to check the functional working of the design file

We do the above processes using a simulator software. The simulator is loaded with the design and its respective testbench file after which it looks for changes in the input signals and depending on the change, the output is evaluated. These changes in input and corresponding output values are dumped in a special format file called "value change dump" (.vcd) file. This file can be pictorially represented in waveforms using a waveform tool like gtkwave.

 # Setup the lab instance with libraries and verilog files
 
 Firstly, we have to clone 2  repository namely sky130RTLDesignAndSynthesisWorkshop which contain the required library files and verilog design files to perform the simulations and logic synthesis parts of the workshop. It can be done using basic linux command gitclone ex: git clone https://github.com/kunalg123/vsdflow.git . We are given a default set of files and libraries shown below to work on using the practical lab instance
 
 ![Screenshot (172)](https://user-images.githubusercontent.com/86364063/123578184-e50faf80-d7f2-11eb-80d6-d9b653b75234.png)
 
  # Simulation using iverilog simulator - 2:1 multiplexer rtl design
  
After cloning the respective repositories in our lab instance, we perform a simulation run of 2:1 multiplexer rtl file namely good_mux.v and its corresponding testbench file tb_good_mux.v to obtain .vcd files and analyze the waveform in gtkwave to see the change in output instances with respect to change in input values.

# Verilog file of a simple 2:1 multiplexer

![image](https://user-images.githubusercontent.com/86364063/123579091-cf02ee80-d7f4-11eb-8c01-8c4592163bd5.png)

# Verilog testbench file of the corresponding 2:1 multiplexer with stimuli

![image](https://user-images.githubusercontent.com/86364063/123579114-dc1fdd80-d7f4-11eb-9615-7760512063c6.png)

To simulate these files in Iverilog, the following command can be used.

          iverilog good_mux.v tb_good_mux.v
          
After that, Iverilog create an output file by the name of a.out which will be added in the file directory. To generate this VCD file, we must execute the a.out file as follows.

          ./a.out
          
After execution, a VCD file with the file extension .vcd will be generated. In our case, it is called tb_good_mux.vcd as that is the name specified in our test bench file. To view this VCD file in GTKWave, the following command is issued.

          gtkwave tb_good_mux.vcd
          
now,in the GTKWave interface panel, we can add the unit under test and select the inputs and outputs whose waveforms we want to view. Now we can confirm if the waveform of our 2:1 Multiplexer matches the design specifications or not. As visible from the below waveform, it does athere to the design specifications.

![image](https://user-images.githubusercontent.com/86364063/123579670-f8704a00-d7f5-11eb-9f03-df6afea0d67b.png)

# Introduction to Yosys

Yosys is a framework for Verilog RTL synthesis. A synthesizer is a tool used for converting RTL based verilog code to netlist. RTL is the behavioural representation of the required specification in Verilog HDL. Netlist is the representation of the design in the form of standard cells present in the library. The Yosys synthesizer flow is as follows.

![image](https://user-images.githubusercontent.com/86364063/123580056-a5e35d80-d7f6-11eb-85f0-4d5dd1740dc4.png)

Therefore,Yosys makes use of the commands read_verilog to read the verilog design, read_liberty to read the .lib, and write_verilog to write the netlist file.

To verify the synthesis output, we can follow the same procedure as we did when verifying verilog design as the netlist must obey the same specifications as the original RTL design. In order to do this, we can pass the netlist file along with the original RTL testbench to our simulator and generate the VCD file. This VCD file can be viewed in the waveform viewer to confirm the behaviour of the synthesized netlist. This is shown below.

![image](https://user-images.githubusercontent.com/86364063/123580146-d75c2900-d7f6-11eb-9061-81e6d1979a0c.png)

# Libraries and their Significance

A synthesizer conducts RTL to Gate level translation, wherein the behavioural design is converted to basic gates using the standard cell libraries provided, and connections are made between these gates. Libraries (.lib) are a collection of basic logic modules to implement any boolean logical functionalities, and may contain different flavours of the same gate such as 2-input/3-input or fast/slow.

# Need for Fast Cells:

For a digital logic circuit, the combinational delay in the logic path determines its maximum speed of operation. Lets take an example of a basic combinational circuit shown below with two D Flip-flops and some combinational circuit betwen them, where CLK is the clock signal and DFF B holds the output of the circuit.

combi ckt

In this cicruit, the minimum size of 1 clock cycle is determined with the following relation

# TCLK > TCQ_A + TCOMBI + TSETUP_B
where,
            TCQ_A is the propogation delay of DFF A
            TCOMBI is the propogation delay of the combination circuit
            TSETUP_B is the setup time for DFF B (min. time before the clock edge that input data must be supplied)

Hence, for maximum performance we need a smaller value of TCLK, which can be achieved by using faster cells to reduce the value of TCOMBI as much as possible.

# Need for Slow Cells:

In order to prevent any hold violations, we need cells that work slower. If we consider the above example, there must be a minimum amount of time during which the output of the combinational circuit must be stable after the active edge of the clock, for DFF B to reliably capture the data at its input. This minimum delay is known as the Hold Time of the circuit. Hence, the following condition must be sastisfied to prevent hold violations.

# THOLD_B < TCQ_A + TCOMBI

Thus, we need fast cells to meet performance requirements as well as slow cells to meet hold times in the .lib collection. To pick appropriate cells, the user must offer "constraints" to the synthesizer.

Note: As the primary load in a digital circuit is capacitance, the charge/discharge times of the capacitor decides cell delay. To discharge capacitors fast we need transistors capable of sourcing more current, thus needing wider transistors with more area and power requirements. While slower cells need narrow transitors with less area and power requirements.

# Exploring Yosys and SKY130PDKs

Let us explore Yosys and the SKY130 libraries using the same example of the simple 2:1 Multiplexer from the previous sections. To start yosys, we must use the command yosys in the terminal. Once invoked, the yosys prompt should appear as follows.

![Screenshot (179)](https://user-images.githubusercontent.com/86364063/123580375-58b3bb80-d7f7-11eb-9d40-02ea096f1930.png)

First, we must read the SKY130 libraries using the command read_liberty -lib filepath. Next, we must read the design using the command read_verilog filename.v. We must now specify the module name of the design we are synthesizing using the command synth -top modulename.

![Screenshot (174)](https://user-images.githubusercontent.com/86364063/123580527-a6c8bf00-d7f7-11eb-93bd-9d0ec889810c.png)

Once this is done, we can generate the netlist using the command abc -liberty your_library_filepath. For our case, this would be abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib. Succesfully executing this command should return the following report in the yosys prompt.

![image](https://user-images.githubusercontent.com/86364063/123580789-23f43400-d7f8-11eb-8a3c-477e0972eec1.png)

As visible in the report, yosys has found 3 inputs, 1 ouptut and 0 internal connections. This holds true for our example of the 2:1 Multiplexer. Further, yosys also mentions the cells used in the logic realisation. To observe a graphical view of the realisation, the command show can be used. This should generate the following graphic.

![image](https://user-images.githubusercontent.com/86364063/123580873-50a84b80-d7f8-11eb-9313-899db94bf7d6.png)

Finally, we can write the netlist using the command write_verilog -noattr filename.v. Here, the property "-noattr" is used to prevent yosys from dumping extra information in the final netlist file

![image](https://user-images.githubusercontent.com/86364063/123580927-6ae22980-d7f8-11eb-9808-b15d1baf88fd.png)


# Day 2 - Timing libs, Hierarchical vs. Flat Synthesis and Efficient Flop Coding Styles


* Further Information on .lib

The SKY130 library file used for this workshop is sky130_fd_sc_hd_tt_025C_1v80.lib

Here,
         fd is SkyWater Foundry
         sc is Standard Cell
         hd is High Density
         tt is Typical Process
         025C is 25°C operating Temperature
         1v80 is 1.8V operating Voltage

Below is some of the contents of this .lib file.

* lib file

Here, you can see that the library provides details like technology (CMOS), power parameters, voltage parameters, current draw, area, timings and delays, etc. This file hold information on every standard cell provided in the library, along with all its flavours. Let's compare the flavours of a basic OR gate.

or gate

From the above image it is evident that each iteration of the OR gate has different power and area consumptions. Larger are and power values are due to wider transistors which are required in faster designs. Hence, in the above case, or2_4 is the faster cell and or2_0 is the slower cell.

* Hierarchical vs. Flat Synthesis

When synthesizing a design containing multiple modules, the notion of heirarchical vs. flat design comes up. To understand their differrences, let's compare the two using an example. Below is a design that instantiates two low level modules, namely an OR gate (sub module u2) and an AND gate (sub module u1). This file is available under the verilog_files directory as multiple_modules.v

* multiple modules

Now, let us synthesize this in Yosys with the following commands.

     yosys
     read_liberty -lib .../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
     read_verilog multiple_modules.v
     synth -top multiple modules
     abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib


Finally, we can write the netlist using the command write_verilog -noattr filename.v. Here, the property "-noattr" is used to prevent yosys from dumping extra information in the final netlist file. Let's name our file as good_mux_netlist.v and execute the command. The final netlist represention is shown below.

If we see the graphical view of the realisation using the command show multiple_modules, we can notice that the actual OR and AND gates are not visible but only the sub modules u1 and u2 are shown. This is known as Heirarchical Synthesis as the heirarchies are preserved. This is shown below.

![image](https://user-images.githubusercontent.com/86364063/123581353-55213400-d7f9-11eb-9a7a-47cbd54a6995.png)

If we generate the netlist using write_verilog -noattr multiple_modules_h_netlist.v, we see a similar story. The heirarchies are preserved, as there is one instantiation of each sub module under multiple_modules. The netlist is shown below.

![image](https://user-images.githubusercontent.com/86364063/123581383-6d914e80-d7f9-11eb-82e2-8143ddb7c5fd.png)

To avoid heirarchical synthesis, we can use the command flatten in the yosys prompt after the synth command. Now, we can generate the netlist again using the command write_verilog -noattr multiple_modules_f_netlist.v.

The netlist generated here does not contain instantiations of sub modules or any heirarchical structure. It directly contains one module with mutiple standard cells. This is known as Flat Synthesis. Its netlist is shown below.

![image](https://user-images.githubusercontent.com/86364063/123581448-944f8500-d7f9-11eb-8549-6cb05ff0d1ae.png)

If we view it graphically using the show command, we can observe only standard cell implementations and no heirarchy present.

![image](https://user-images.githubusercontent.com/86364063/123581502-acbf9f80-d7f9-11eb-9492-23d8a2697a13.png)

From a multiple module design file, it is possible to just synthesize a single sub module. This is done using the command synth -top sub_module_name, and is known as Sub Module level Synthesis. This is used when,

we have multiple instances of the same module and just want to synthesize it once, then replicate it however many times.
we are using a divide and conquer approach (used in massive designs when the tool does not do an appropriate or well optmised job).

# Flop Coding Styles

In combinational circuits, each circuit element or cell experiences a time delay for the output to change based on a change in the input. This delay is known as Propogation Delay. Due to these propogation delays, the circuit might experience unwanted transitions in the output, espescially as the propagation delay stacks additively as the number of combinational circuits increase. These unwanted transitions are known as Glitches in the output.

To avoid glitches, we make use of D Flip-flops as storage elements or buffers in between the different combinational circuits. D flip-flops store the value present on their input, and its output changes only at clock edges. This brings stability between combinational circuits as the D flip-flops shield the combinational circuit they are feeding against glitches in their input, allowing the output of that combinational circuit to settle down.

Flip-flops come in various types. These are mainly:

* Synchronous vs. Asynchronous set
* Synchronous vs. Asynchronous reset
* Rising (positive) edge triggered vs. Falling (negative) edge triggered

To further understand understand the different flop styles, let's look at 3 D flip-flops available to us in the directory verilog_files.

For synthesizing designs involving D flip-flops in Yosys, we must use the command dfflibmap -liberty dff_library_filepath which in our case is dfflibmap -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib. This command is used to read the dff standard cells as some libraries may have seperate .lib files for these. The command must be issues after synth and before abc.

# 1. Rising edge D Flip-flop with asynchronous reset

* Verilog code:

module dff_asyncres ( input clk ,  input async_reset , input d , output reg q );
always @ (posedge clk , posedge async_reset)
begin
	if(async_reset)
		q <= 1'b0;
	else	
		q <= d;
end
endmodule

* Waveform for dff_asyncres

![image](https://user-images.githubusercontent.com/86364063/123582111-ddec9f80-d7fa-11eb-905e-8620eb05e6e3.png)

* Graphical representation of synthesized netlist for dff_asyncres

![image](https://user-images.githubusercontent.com/86364063/123582158-f65cba00-d7fa-11eb-9ea3-e6c620e39344.png)

# 2. Rising edge D Flip-flop with asynchronous set

* Verilog code:

module dff_async_set ( input clk ,  input async_set , input d , output reg q );
always @ (posedge clk , posedge async_set)
begin
	if(async_set)
		q <= 1'b1;
	else	
		q <= d;
end
endmodule

* Waveform for dff_asyncres

![image](https://user-images.githubusercontent.com/86364063/123582432-6d924e00-d7fb-11eb-97ed-0b08328e9020.png)


* Graphical representation of synthesized netlist for dff_asyncres

![image](https://user-images.githubusercontent.com/86364063/123582466-7edb5a80-d7fb-11eb-85a8-b4e3169dc266.png)

# 3. Rising edge D Flip-flop with synchronous reset

* Verilog code:

module dff_syncres ( input clk , input async_reset , input sync_reset , input d , output reg q );
always @ (posedge clk )
begin
	if (sync_reset)
		q <= 1'b0;
	else	
		q <= d;
end
endmodule

* Waveform for dff_asyncres

![image](https://user-images.githubusercontent.com/86364063/123582620-bfd36f00-d7fb-11eb-9b93-2d2d820c6271.png)


* Graphical representation of synthesized netlist for dff_asyncres

![image](https://user-images.githubusercontent.com/86364063/123582643-c8c44080-d7fb-11eb-91f1-e72bbab0dd10.png)


# Special Cases for Optimisations:

# Case 1:

Let's consider the following design where the 3 bit input is multiplied by 2 and the output is a 4 bit value.

module mul2 (input [2:0] a, output [3:0] y);
	assign y = a * 2;
endmodule

The ouput y[3:0] is nothing but the input a[2:0] appended with a 0 at the LSB. Or, we can say that y = {a, 0}. If we synthesize the netlist and look at its graphical realisation, we will see the same optimisation occuring in the netlist.

![image](https://user-images.githubusercontent.com/86364063/123582795-16d94400-d7fc-11eb-8b28-149f2b7ad871.png)

# Case 2:

Let's consider the following design where the 3 bit input is multiplied by 9 and the output is a 6 bit value.

module mult8 (input [2:0] a , output [5:0] y);
	assign y = a * 9;
endmodule


 The ouput y[5:0] is nothing but the input a[2:0] appended with itself. Or, we can say that y = {a, a}. If we synthesize the netlist and look at its graphical realisation, we will see the same optimisation occuring in the netlist.
 
 ![image](https://user-images.githubusercontent.com/86364063/123582999-7b949e80-d7fc-11eb-96ce-3d251b1294e9.png)


# Day 3 - Combinational and Seqeuntial Optimisations
Introduction to Logic Optimisations
There are broadly two types of logic available, combinational and sequential. In order to save cost by reducing power and area consumptions of a design, we must optimise the design as best as possible. For each type of logic, there exist different methods of opimisation, as follows.

# 1. Combinational optimisation methods:

* Squeezing the logic for Area and Power savings
* Constant Propogation
* Direct Optimisation
* Boolean Logic Optimisation
* K-Map
* Quine-McKluskey Algorithm

# 2. Sequential optimisation methods:

* Basic
* Sequential Constant Propogation
* Advanced
* State Optimisation
* Retiming
* Sequential Logic Cloning (Floor Plan Aware Synthesis)

* Combinational Logic Optimisations

Let's take a look at some examples of combinational optimisations using the files opt_check.v, opt_check2.v, opt_check3.v, opt_check4.v, and multiple_modules_opt.v. All of these files are under the verilog_files directory.

* Example 1: opt_check.v

module opt_check (input a , input b , output y);
	assign y = a?b:0;
endmodule
Using boolean logic simplification, we can tell that y = ab. Let us synthesize this in yosys using the following commands.

    read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
    read_verilog opt_check.v
    synth -top opt_check
    
Before realising the netlist, we must issue a command to yosys to perform constant propogation and optimisations. this can be done using the opt_clean -purge command

After this step, we can continue as usual with abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib and write_verilog -noattr opt_check_netlist.v commands. If we view the graphical realisation witht the show command, we can see that Yosys has synthesized an AND gate as expected.

![image](https://user-images.githubusercontent.com/86364063/123583564-7421c500-d7fd-11eb-899a-d17339cdb5dc.png)

# Example 2: opt_check2.v

module opt_check2 (input a , input b , output y);
	assign y = a?1:b;
endmodule

Similar to the example 1, lets continue with optimisations for this design. Here we expect the output to be an OR gate based on boolean optimisation, since the output can be simplified to y = a + b. If we generate the netlist and look at its graphical realisation, we get the following.

![image](https://user-images.githubusercontent.com/86364063/123583638-974c7480-d7fd-11eb-8be2-0fb2c715f960.png)

# Example 3: opt_check3.v

module opt_check3 (input a , input b, input c , output y);
	assign y = a?(c?b:0):0;
endmodule

For this design, we expect the output to be a 3 input AND gate based on boolean optimisation, as the output can be simplified to y = abc. If we generate the netlist and look at its graphical realisation, we get the following.

![image](https://user-images.githubusercontent.com/86364063/123583706-b5b27000-d7fd-11eb-828e-054892f49240.png)

# Example 4: opt_check4.v

module opt_check4 (input a , input b , input c , output y);
	assign y = a?(b?(a & c):c):(!c);
endmodule
For this design, after boolean logic optimisation we can conclude that the output can be simplified to a single xnor gate with the output equation y = a⊙c. If we generate the netlist and look at its graphical realisation, we get the following.

![image](https://user-images.githubusercontent.com/86364063/123583764-cebb2100-d7fd-11eb-884e-41b12c100f1c.png)

# Example 5: multiple_module_opt.v

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

![image](https://user-images.githubusercontent.com/86364063/123641306-c1268b00-d83f-11eb-8807-7defc52ced86.png)

# Sequential Logic Optimisations
Let's take a look at some examples of sequential optimisations using sequential constant propogation. We shall be using the files dff_const1.v, dff_const2.v, dff_const3.v, dff_const4.v, and dff_const5.v. All of these files are under the verilog_files directory.

* Example 1: dff_const1.v

module dff_const1(input clk, input reset, output reg q);
always @(posedge clk, posedge reset)
begin
	if(reset)
		q <= 1'b0;
	else
		q <= 1'b1;
end
endmodule
At first glance, it may seem that the output bit q should be equal to an inverted reset or !reset. However, as the reset is synchronous, so the output depends on both the reset and clk edge. This can be confirmed by simulating the design in Iverilog, and viewing the VCD with GTKWave as follows.

![image](https://user-images.githubusercontent.com/86364063/123641452-e87d5800-d83f-11eb-95be-0fba1d90ac1d.png)

If we observe the waveform above, when reset becomes 0, q only becomes 1 at the next clock edge. Hence, we do not get a sequential constant, and no optimisations should be possible here. Let's confirm the same using Yosys synthesis and optimisation as follows.

We must use the command dfflibmap -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib as our design includes D flip-flops. We can then generate the netlist using abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib and write_verilog -noattr dff_const1_netlist.v. To view the graphical realisation, we use the show command.

![image](https://user-images.githubusercontent.com/86364063/123641598-0c409e00-d840-11eb-9d6e-697e3e896c6b.png)

* Example 2: dff_const2.v

module dff_const2(input clk, input reset, output reg q);
always @(posedge clk, posedge reset)
begin
	if(reset)
		q <= 1'b1;
	else
		q <= 1'b1;
end
endmodule
Here, we can see that regardless of the inputs, the ouput q always remains constant at 1. This can be observed in the waveform viewer as well.

![image](https://user-images.githubusercontent.com/86364063/123641730-3003e400-d840-11eb-8d61-dd2fb268ac70.png)

![image](https://user-images.githubusercontent.com/86364063/123641777-40b45a00-d840-11eb-9116-bfc8ab49f54b.png)

* Example 4: dff_const4.v

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
Here, we can see that regardless of the reset input, q1 is always going to be constant at 1. As q can only be 1 or q1 depending on the reset input, but q1 = 1. Thus q is also constant at the value 1. We can confirm this with the simulated waveforms as shown below.

![image](https://user-images.githubusercontent.com/86364063/123642017-76f1d980-d840-11eb-85a3-a805467a4d53.png)

![image](https://user-images.githubusercontent.com/86364063/123642051-7ce7ba80-d840-11eb-99bf-ae5395293001.png)


# Day 4 - Gate Level Simulation, Blocking vs. Non-Blocking Assignments and Synthesis Simulation Mismatch

* Introduction to Gate Level Simulations

Gate Level Simulation (GLS) is the process of executing the netlist as the unit under test, instead of the source design file as we conducted up until now. We can make use of the original testbench itself to simulate the behaviour of the netlist, as the netlist is logically the same as the source RTL code.

GLS is an important procedure as we must ensure that the netlist meets the design specifications. Unlike the RTL code which uses logic, the netlist utilizes physical gates to realise the same logic. These gates introduce timing concerns like propogation delays and hold times. Hence, using gate level simulations, we can ensure that the timings of the design are met as per the specifications.

The flow for gate level simulations using Iverilog is shown below.

![image](https://user-images.githubusercontent.com/86364063/123642217-abfe2c00-d840-11eb-9511-3e1fd49e4734.png)

Understanding Synthesis Simulation Mismatches

If the netlist is a true representation of the RTL code, then why must we verify the functionality of the netlist? This is because sometimes there are mismatches between the pre-synthesis simulations and the post-synthesis simulations. This is known as synthesis simulation mismatch and can occur due to, but not limited to, the following reasons:

* Missing sensitivity list
* Blocking and non-blocking assignments
* Non-standard verilog coding

Let us look into the above issues using some example cases.

# Missing Sensitivity List:

As we have learnt earlier, a simulator only updates or changes its output when it finds a change in the input. The verilog code below describes a 2:1 Multiplexer. There are 3 inputs i0, i1, and sel and 1 output y.

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
In this example the simulator checks for changes in the input sel to update the value of the output y, as the always block is evaluated only for sel. If sel were constant at 0 and i0 were to change its value, we would observe no change in the output y. This means the design would function more like a latch instead of a multiplexer. If we were to synthesize this design however, we would observe a multiplexer in our netlist instead of latch like behaviour. This would cause synthesis simulation mismatch.

This can be fixed by replacing the always @(sel) line with always @(*) instead which looks for changes in any input.

# Blocking and Non-Blocking Statements:

Within the always block in a verliog code, assignments can be of two types:

* Blocking statements:

These are assignments that make use of the = operator.
Here, statements are always executed in the order that they are written.

* Non-blocking statements:

These are assignments that use the <= operator.
Here, all RHS blocks are evaluated parallelly when the always block is entered, and then assigned to the LHS
.
Let's look at some examples of blocking statements and how they can cause synthesis simulation mismatches.

Below, we have verilog code for a serial shift register with input d and output q1. As the code using blocking statements, the line q0 = d; will execute before q1 = q0;. This means that the value at d will direectly be updated at q1 at every clock edge, and the design will function as a single D flip-flop instead of a 2 bit shift register.

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
This problem will cause a mismatch in the pre-synthesis and post-synthesis simulations as the netlist will still function as a 2 bit shift register. This can be fixed by either writing the line q1 = q0; before the line q0 = d; or by using non-blocking statements instead as now both q0 and q1 would be evaluated parallelly regardless of their position. Hence, the non-blocking operator <= is preferred when using sequential designs.

Let's look at another example. Here we have the verilog code for some combinational logic.

module code (input a, input b, input c, output reg y);
reg q0;

always @(*)
begin
	y = q0 & c;
	q0 = a|b;
        
end 
endmodule
Similar to the earlier example, the AND assignment would occur before the OR assignment. This means the output y would get a delayed value of a|b, as the value of q0 would get updated after the output y is updated. This too would cause synthesis and simulation mismatches, and can be fixed by writing the line q0 = a|b; before y = q0 & c;.

Exploring GLS and Mismatches with Iverilog
To further understand gate level simulations and mismatches between pre-synthesis and post-synthesis simulation, let us try some examples in Iverilog.

* Example 1:

Below is the verilog code for a 2:1 multiplexer using a ternary operator. Let's attempt gate level simulations on this design.

module ternary_operator_mux (input i0 , input i1 , input sel , output y);
	assign y = sel?i1:i0;
endmodule
First, we must synthesize this file using Yosys and generate its netlist. We can view the graphical realisation of the same below.

!(ternarymux show)[images/Day4/4-1.png]

Now that we have the netlist, we can run the GLS using Iverilog by specifying the gate level models with the following command.

iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v ternary_operator_mux_net.v tb_ternary_operator_mux.v

We can view the simulated waveforms using GTKWave and verify that the generated netlist does behave like a 2:1 multiplexer.

![image](https://user-images.githubusercontent.com/86364063/123642723-3e063480-d841-11eb-8796-0531c43efeb8.png)

* Example 2:

Here, we have the verilog code for the file bad_mux.v which is available in the directory verilog_files.

module bad_mux (input i0 , input i1 , input sel , output reg y);
always @(sel)
begin
	if(sel)
		y <= i1;
	else 
		y <= i0;
end
endmodule
This code is similar to one of the earlier examples discussed. It should behave more like a latch than a 2:1 multiplexer in the pre-synthesis simulation due to the missing sensitivity list. This can be confirmed by simulating the design in Iverilog and viewing its waveform.

![image](https://user-images.githubusercontent.com/86364063/123642845-5f672080-d841-11eb-9790-4d65375e1af8.png)

As we can see, the design does not function as a multiplexer, and the output only gets updated when there is a change in the value of the input sel. However, if we try to generate its netlist using yosys, we can see below that the synthesized netlist contains a multiplexer and not a latch like circut.

![image](https://user-images.githubusercontent.com/86364063/123642890-6d1ca600-d841-11eb-8c0c-6bbb63c6e2c4.png)

If we now run gate level simulation on the synthesized netlist and view the waveform, we should see that it functions exactly like a 2:1 multiplexer.

![image](https://user-images.githubusercontent.com/86364063/123642948-81f93980-d841-11eb-9bcd-423df472cbd6.png)

* Example 3:

Let's take a look at an example of mismatch due to blocking statements. We shall use the verilog design file blocking_caveat.v which is available in the directory verilog_files. The code for the same is as follows.

module blocking_caveat (input a , input b, input c, output reg d); 
reg x;

always @(*)
begin
	d = x & c;
	x = a | b;
end
endmodule
Similar to an earlier example, we should get a synthesis simulation mismatch. This is because of the blocking statements used, as the output d will always be evaluated before x. If we simulate this file in Iverilog, the simulated waveform should tell the same story.

![image](https://user-images.githubusercontent.com/86364063/123643071-a523e900-d841-11eb-9d1e-e331b3c475c5.png)

If we observe the instance of time highlighted by the cursor, we can see that the output y holds the value 1. However it should hold the value 0, as both the inputs a and b are 0. This means a | b should output 0, which when AND with c, should give an output of 0. But due to the blocking statements used, x actually holds a 1 tick delayed value of a | b, hence giving us an incorrect output.

Now, let's try generating the netlist for this design and viewing its graphical realisation using Yosys.

![image](https://user-images.githubusercontent.com/86364063/123643164-bd940380-d841-11eb-93d9-bc3afbae6c24.png)

As we can see, the netlist does not include any latches to hold delayed values. It only includes an OR 2 AND gate. If we run gate lavel simulations on this netlist in Iverilog, we should see the following waveform.

![image](https://user-images.githubusercontent.com/86364063/123643252-d2709700-d841-11eb-8a2e-fb07673c532a.png)


# Day 5 - If.. Case Statements and for loop & for generate statements

* If.. Statements

if statements are used to write priority logic. It can infer a multiplexer HW is written properly. The top most / beginning if condition has the highest priority. If not written properly including all conditions and outputs, they can result in INFERRED LATCHES.

INFERRED LATCHES due to improper if constructs
Consider the example of the code using if statement given below

![image](https://user-images.githubusercontent.com/86364063/123643706-4c088500-d842-11eb-9c78-92b1a79711c4.png)

It is evident that the if.. conditions are not properly met. When i0 is 1, output y = i1, and the else condition of what happens when i0 is not 1 is left out without any mention. Hence, during synthesis, the yosys will consider retaining the previous value for y when i0 is not 1. This is the cause for inferred D-Latch that we see in the output netlist below.

![Screenshot (217)](https://user-images.githubusercontent.com/86364063/123645175-b968e580-d843-11eb-96e4-8169a56388f3.png)

![Screenshot (218)](https://user-images.githubusercontent.com/86364063/123645649-22505d80-d844-11eb-8d28-703d7321ddbd.png)

The example of an incomplete case mentioned above can easily be avoided by using a default statement while specifying case options. Let us consider the example of a complete case verilog code given below and verify the output waveforms and netlists.

![image](https://user-images.githubusercontent.com/86364063/123645770-3dbb6880-d844-11eb-8f60-92d3ca17366c.png)

![Screenshot (220)](https://user-images.githubusercontent.com/86364063/123646124-8d9a2f80-d844-11eb-8b04-e0dea45c5180.png)

An example of a badly written case statement is given below. Here, the simulator gets confused when the value of sel is 2'b1? as it can either have 1 or 0 as value. This might result in a confusion as 2'b10 will then have two outputs i2 and i3. This mismatch is clearly explained in the waveforms of simulation and GLS listed below.

![image](https://user-images.githubusercontent.com/86364063/123646243-a86ca400-d844-11eb-80eb-f1061e3301c8.png)

![Screenshot (221)](https://user-images.githubusercontent.com/86364063/123646415-cf2ada80-d844-11eb-8010-ba088b53242a.png)

# for loop vs for generate statements

* for loop statements

for loops are used in looping constructs to evaluate an expression for a repeated number of iterations. They are used inside an always block. They are not used in repeated instantiation of modules or HW blocks.

Consider the following example of a 4:1 MUX code using for loop statements. Here, for loops are an easy way of repeatedly running an evaluation expression thereby saving coding space and time. The resulting rtl vs testbench and GLS waveforms match.

![image](https://user-images.githubusercontent.com/86364063/123646985-54ae8a80-d845-11eb-9fe7-2b9282f43056.png)

![Screenshot (222)](https://user-images.githubusercontent.com/86364063/123647157-790a6700-d845-11eb-90e8-56c91af64ac1.png)

![Screenshot (223)](https://user-images.githubusercontent.com/86364063/123647316-9c351680-d845-11eb-8b2b-fa1753c95fdb.png)

# For generate statements

* for generate statements are used to instantiate a hardware module for a large number of instantiations. Ex: to instantiate an AND gate 100 times. They should never be used inside an always block. Syntax:

genvar i;
generate
     for(.....) begin
      .........
      .........
     end
An example of using for generate statements is given below. We use generate statements and for loop to implement an 8-bit Ripple Carry Adder which uses multiple instantiations of Full Adder block.

![image](https://user-images.githubusercontent.com/86364063/123647556-d1416900-d845-11eb-9d13-2afc8a1e988f.png)

![image](https://user-images.githubusercontent.com/86364063/123647593-d7374a00-d845-11eb-88f8-f6d75dbd74d5.png)

![Screenshot (224)](https://user-images.githubusercontent.com/86364063/123648281-73615100-d846-11eb-9afe-a56655d56e82.png)


























































