# sky130_RTL_DesignAndSynthesis_Workshop

![VSD workshop](https://user-images.githubusercontent.com/78468534/120083126-45241080-c0e4-11eb-9c1e-c970c2737ca1.jpeg)
VSD workshop - RTL design using Verilog with SKY130 Technology

This repo is the report for 5-day workshop on RTL Design and Synthesis with SKY130 Technology. It was organized by VLSI Sytem Design.

## *Contents*
------------
* [Introduction to Tools and Technology](#introduction-to-tools-and-technology)
* [Day 1-Introduction to Verilog RTL design and Synthesis](#day-1-introduction-to-verilog-rtl-design-and-synthesis)
  * [Simulation Flow](#simulation-flow)
  * [Synthesis Flow](#synthesis-flow)
* [Day 2-Timing libs, hierarchical vs flat synthesis and efficient flop coding styles](#day-2-timing-libs-hierarchical-vs-flat-synthesis-and-efficient-flop-coding-styles)
  * [Library files](#library-files)
  * [Hierarchical and Flat Synthesis](#hierarchical-and-flat-synthesis)
  * [Flipflop Coding and Synthesis](#flipflop-coding-and-synthesis)
* [Day 3-Combinational and sequential optimizations](#day-3-combinational-and-sequential-optimizations)
  * [Combinational Logic Optimisations](#combinational-logic-optimisations)
  * [Sequential Logic Optimisations](#sequential-logic-optimisations)
* [Day 4-GLS, blocking vs non-blocking and Synthesis-Simulation mismatch](#day-4-gls-blocking-vs-non-blocking-and-synthesis-simulation-mismatch)
  * [GLS and Synthesis Simulation Mismatch](#gls-and-synthesis-simulation-mismatch)
  * [Blocking and Non Blocking Statements](#blocking-and-non-blocking-statements) 
* [Day 5-If, case, for loop and for generate](#day-5-if-case-for-loop-and-for-generate)
  * [IF and CASE Statements](#if-and-case-statements)
  * [FOR loop and FOR GENERATE](#for-loop-and-for-generate)
* [Acknowledgement](#acknowledgement)


---------
## Introduction to Tools and Technology
* **_SKY130_** - Sky130 process node and pdk(process design kit) are an open-source packages provided by Google and skywater for 130nm technology.
* **_iverilog_** - Iverilog stands for Icarus verilog, is an open source verilog simulator.
* **_GTKWave_** - GTKWave is an open-source vcd(value change dump) waveform viewer.
* **_Yosys_** - Yosys is an open-source synthesis tool.
These are the _open-source_ tools used in the labs for the workshop.
---------
## Day 1-Introduction to Verilog RTL design and Synthesis
First step is to import all files including sky130 libraries into the system. We import the files with the help of git clone. Next, to work with the tools, the tool flow must be clear.
The design flow consists of various types of files. These include:
* RTL design - This is the HDL(hardware description language) code for the logic which is to be implemented. Verilog is the most popular HDL used in RTL design and ends with extension ".v".
* Testbench - The testbench is code also written in HDL. The testbench instantiated the RTL design and observes its outputs for different input values to check the logic functionality of code.
* Gate level netlist - The gate level netlist contains the design in terms of individual gate connections as opposed to RTL design which is the behavioural code for the logic implemented.
### Simulation Flow
The iverilog simulator takes the verilog(RTL or netlist) file and test bench as input and produces vcd (Value Change Dump) file. This vcd file can be viewed using the GTKWave.  
*Note: The iverilog simulation flow is similar for both RTL simulation and Gate-level simulation(GLS).*
##### Commands for simulation
Use the below commands for simulation and view the waveform with iverilog and GTKWave respectively.

        iverilog file_name.v testbench_name.v           //creates an executable file
        ./a.out                                         //generates vcd file
        gtkwave testbench_name.vcd                      //view vcd file in gtkwave viewer
Snippet below shows the terminal for simulation flow of a synchronous reset d-flipflop.
![Simulation Flow CLI](https://user-images.githubusercontent.com/100671647/218275208-5565dee2-1a62-4cc5-9968-de1c4f260c17.png)

  
Waveform shown by the GTKwave for the same example is:
![dff_syncres_waveform](https://user-images.githubusercontent.com/100671647/218275245-adc6bd0c-b21e-42f6-ae15-a2cc13e36635.png)

  
### Synthesis Flow
The synthesis tool being used is Yosys. The inputs for Yosys are the RTL design written in HDL and the libraries required. Here the libraries by sky130 is used. Libraries exist as ".lib" files. These libraries contain different flavours of the most common logical blocks(like AND, NOT gate, MUX, Flip-flops, etc). The circuit is synthesised with these logical blocks.  
##### Why do we need different flavours?
The different flavours of same logic blocks(standard cells) allows these to be used in different applications. These flavours may work on different speeds. The faster the cell the more area and power required. The cell used depends on which parameter(s) is to be optimised.  
Additionally different cells are required to meet the timing requirements. More about that is discussed in [Day 2](day-2).  

First of all, Yosys tool is invoked in the terminal.

                $ yosys
                
![Yosys](https://user-images.githubusercontent.com/100671647/218275177-a072f799-4194-47bc-8464-c43747ce9010.png)


Now inside the yosys, type the following commands for synthesis

                yosys> read_liberty -lib ../path_to_library             //reads library file
                yosys> read_verilog file_name.v                         //reads RTL file to be synthesized
                yosys> synth -top file_name
                yosys> abc -liberty ../path_to_library                  //actual synthesis
                yosys> write_verilog -noattr netlist_name.v             //write created netlist as verilog file
                yosys> show                                             //shows netlist as circuit diagram with cells

Finally use "exit" command when you want to exit from yosys.  
*Note: The present working directory should contain RTL design before invoking yosys.
---------

## Day 2-Timing libs, hierarchical vs flat synthesis and efficient flop coding styles
### Library files
The library file used is *_sky130_fd_sc_hd_tt_025C_1v80.lib_*. The nomenclature for library is not random and shows important parameters most importantly the *Process*,*Temperature* and *Voltage*. These parameters are show in the end of the name "tt_025C_1v80".  
* tt means the process is "typical"
* 025C shows the temperature - 25 degree celsius
* 1v80 shows the voltage - 1.8V

![sky130 lib](https://user-images.githubusercontent.com/100671647/218275336-72a9436f-3473-4e6e-9bb1-a5f2c34e33b5.png)
  
_SKY130 library file_
  
##### Flavours of standard cells
As mentioned before, the library contains different flavours of same logic gates. This is done mainly to meet timing constraints.  
* *Faster cells* increases the clock frequency.
  T(clk) > T(cq_launch_flop) + T(combi) +T(setup_capture_flop)
* *Slower cells* are required to prevent hold violations.
  T(hold_capture_flop) < T(cq_launch_flop) + T(combi)
  
These different flavours are named different within the library. For example, the different flavours of and gate present are:
- based on delay:      _sky130_fd_sc_hd_and2_0 > sky130_fd_sc_hd_and2_2 > sky130_fd_sc_hd_and2_4_
- based on power/area: _sky130_fd_sc_hd_and2_0 < sky130_fd_sc_hd_and2_2 < sky130_fd_sc_hd_and2_4_

### Hierarchical and Flat synthesis
Many times in complex systems, different parts of system are designed seperately as RTL blocks(sub-modules) which can later be combined to form the whole system(top module). In such cases the synthesis can be done in two ways:
* *Hierarchial synthesis* - Here the top module is synthesised in terms of sub-modules. Only the higher level abstraction is required.
* *Flat Synthesis* - Here all sub-modules are also expanded and the top module is sysnthesised in terms of standard cells. It has lower abstraction compared to hierarchial synthesis.
Consider the example
![multiple_modules](https://user-images.githubusercontent.com/100671647/218275497-fdeda5b5-bea9-4c9d-8a1a-61b5e471d7df.png)
  
If this logic is synthesised noramlly it will do hierarchial synthesis. The netlist made following hierarchial synthesis is. 

![multiplemodules_hier](https://user-images.githubusercontent.com/100671647/218275732-a6960350-b426-4393-8d82-8c95d2ed08d7.png)

  
For flat synthesis an additional command needs to be added to the normal synthesis flow.

              yosys> synth -top file_name
              yosys> flatten
              yosys> abc -liberty ../path_to_library
              yosys> write_verilog flat_netlist_name

This will give us a flattened netlist like shown below.  

![multiplemodules_flat](https://user-images.githubusercontent.com/100671647/218276202-040394d0-1551-4380-be11-9524410d4310.png)

  
### Flipflop Coding and Synthesis
When it comes to flipflops, the set/reset logic is very important. They can be either synchronous or asynchronous. For flops with synchronous set/reset, the set/reset is dependant on clock and therefore this logic gets realised on the input (D- input).  
In the HDL code the asynchronous and  synchronous reset can be distinguished by the sensistivity list. A flipflop with asynchronous reset should have both clock and reset on the sensitiivity list. A flipflop with synchronous reset on the other hand would have only clock on the sensitivity list as the reset is also dependant on the clock.  

Below is the D flipflop with synchronous reset:
![dff_syncres_waveform](https://user-images.githubusercontent.com/100671647/218275245-adc6bd0c-b21e-42f6-ae15-a2cc13e36635.png)
Below is the D flipflop with asynchronous reset:
![asyncres_wave](https://user-images.githubusercontent.com/100671647/218276488-1f992f65-f191-4bda-aaef-c07c366dd56f.png)

 

Now during synthesis of sequential circuits, an additional step is required to point the tools towards library containing the flipflops. This is done by the command:

              yosys> synth -top file_name
              yosys> dfflibmap -liberty ../path_to_library
              yosys> abc -liberty ../path_to_library
              yosys> show
              
The circuits obtained by designing flipflops with asynchronous and synchronous reset are given below

D flipflop with asynchronous reset: 
![asyncres ckt](https://user-images.githubusercontent.com/100671647/218276632-a452217f-1d64-4c75-a409-57420e181513.png)

 D flipflop with synchronous reset:
![syncres ckt](https://user-images.githubusercontent.com/100671647/218276753-f8f1f2f9-691d-4912-9bd1-d9efe7be4a03.png)



The netlist clearly shows that the synchronous reset is applied at D input as expected.  
---------
 ## Day 3-Combinational and sequential optimizations
 The synthesis tool comes with many features. One of such features which has a huge impact on design is _optimisation_. The tool does optimisation on the logic (RTL design). Usually these optimisation are done to obtain least hardware and omit unwanted components.  
 ### Combinational Logic Optimisations
 In designs containing complex combinational logic, most of the time certain components are not reflected in the output and hence are considered _useless_. Such parts are removed by the tool. Sometimes sub-modules within a top module which does not affect the output may also be removed.    
 Consider the example:  
 ![combi_logic](https://user-images.githubusercontent.com/100671647/218276957-edcc4dd7-9dac-4e49-83ca-83753e0d6da8.png)

  
From the boolean logic for this RTL, it is clear that "b" input is not reflected on the output. Hence the synthesised netlist should optimisations. This becomes clear from the netlist given below.  
![combi_ckt](https://user-images.githubusercontent.com/100671647/218277202-6f08a0d6-ef28-418f-85e0-cdd86eab10ea.png)

  
Now consider a combinational circuit written as sub-modules.  
![mulmodopt logic](https://user-images.githubusercontent.com/100671647/218277304-e7ff73cb-c121-4240-9716-c4748ffb6c37.png)

  
The optimised netlist obtained is:  
![mulmod opt ckt](https://user-images.githubusercontent.com/100671647/218277663-49448462-d187-4ea2-8f0b-f1d36e07a063.png)


### Sequential Logic Optimisations
The most common optimisations in sequential circuits are optimisations for constant and optimising unused outputs.
  
For optimisation of constants in sequential circuits, consider below example.  
![seq logic](https://user-images.githubusercontent.com/100671647/218278071-ab1f5e77-2408-4d36-9164-fe3716cefda1.png)

  
After simulating with the help of testbench the waveform obtained is:  
![seq wave](https://user-images.githubusercontent.com/100671647/218278020-4c4f20fe-52b7-4a87-b2ab-6c89c4a15a57.png)

  
From the waveform it is clear the output remains constant for all cases and hence can be optimised. The synthesis report obtained is given below.  
![seq synth report](https://user-images.githubusercontent.com/100671647/218277845-b2e40342-2249-41e4-aed2-a1662c51e821.png)

  
Finally the optimised netlist given by the tools is:  
![seq netlist](https://user-images.githubusercontent.com/100671647/218278280-3dcc61be-9a8b-4217-8140-05efbc8829ab.png)

  
Now consider the next example for optimisation of unused outputs in sequential circuits.  
![count_opt logic](https://user-images.githubusercontent.com/100671647/218278365-bc5aeff1-19ad-4851-84a8-0910e021102e.png)

  
![countopt wave](https://user-images.githubusercontent.com/100671647/218278488-7fba19e6-e487-4e52-be74-0903dcae24cc.png)

_Waveform_  
  
  
From the waveform it is clear that output is only dependant single bit of the counter. Hence the other flops can be optimised.  
![countopt rprt](https://user-images.githubusercontent.com/100671647/218278578-38093345-2382-47ef-83e6-33083376cffb.png)

![countopt ckt](https://user-images.githubusercontent.com/100671647/218278668-b0b85404-e3ce-4685-975b-2b63cb29b552.png)
The synthesised output contains only one flipflop as other unused flops are optimised off.

---------
## Day 4-GLS, blocking vs non-blocking and Synthesis-Simulation mismatch
### GLS and Synthesis Simulation Mismatch
While using an HDL to write the RTL logic, if not careful, synthesis simulation mismatch may occur.
Gate-level simulation(GLS) is the simulation of gate level netlist of a design unlike the normal functional simulation where the behavioural RTL is simulated. When the RTL simulation is different from GLS the circuit is said to exhibit synthesis simulation mismatch.  
This can occur due to a variety of reasons including:
* missing sensitivity list
* wrong use of blocking/non-blocking statements
* non-standard verilog coding  

Lets see an example of such sythesis simulation mismatch
![badmx logic](https://user-images.githubusercontent.com/100671647/218278847-cd1ef0f2-f31d-41ca-ad97-f1d0ca39278b.png)
This is the RTL code for a mux, but the sensitivity list shows only "sel". This means the output will only change when there is a change in "sel" signal. This is obviously not the desired logic and will cause synthesis simulation mismatch due to the missing elements in sensitivity list.  
The waveform obtained from RTL simulation is :
![bdmx wave](https://user-images.githubusercontent.com/100671647/218278942-f99a8a8f-68e2-4d6b-b870-8adab868acc3.png)
compared to the waveform from GLS
![badmx gls](https://user-images.githubusercontent.com/100671647/218279392-dcb6efe0-2ce5-4557-b784-372e6aa0722e.png)

  
_Note: The verilog models of standard cells must also be called on the iverilog for GLS._
### Blocking and Non Blocking Statements
In verilog HDL, there are two types of assignment operators:
* <= this assignment creates a non-blocking statement
* =  this assignment creates a blocking statement
  
For non-blocking statements all the RHS are first found out before it is assigned. Since all such statements are assigned at the same time the order of statements are irrelevant. In the case of blocking statements, values are assigned instantly and therefore the order of statements are extremely important.  
While using blocking statements if care is not taken it could result in synthesis simulation mismatch. For example consider:
![block logc](https://user-images.githubusercontent.com/100671647/218279506-08861ae5-cece-4b00-9347-aa3cc888f31b.png)  
_RTL simulation waveform_
![blckng wave](https://user-images.githubusercontent.com/100671647/218279596-dc1b3b23-6d34-46eb-8cd1-ac78345278e2.png)

_GLS waveform_
![blckng gls](https://user-images.githubusercontent.com/100671647/218279714-3890bfed-8739-4710-89fa-adb661a1dcc9.png)


## Day 5-If, case, for loop and for generate
### IF and CASE statements
The _IF_ and _CASE_ statements are the conditional statements present in verilog HDL.  
If statements-    
              
              if (condition1)
                  ......
              else if (condition2)
                  ......
              else
                  ......
Case statements-

             case(sel)
                condition1 : ......
                condition2 : ......
                default    : ......
                
The main difference between  case and if statements is the priority level. If statements has priority level and case statements do not. Although these statements are required in almost all high level design it also brings complexities. An incomplete if or case statements may create additional problems. For example consider:

![inco_if](https://user-images.githubusercontent.com/100671647/218279790-412bdcbe-d4e4-47d7-96d8-4b53e6837f85.png)  
The incomplete if statement in this example with infer unwanted latches. This is evident in simulation and synthesis results given below.  
_Simulation waveform_
![incomp_if wave](https://user-images.githubusercontent.com/100671647/218279871-0f83ec2f-9621-43d4-8c29-990a2b90cced.png)

Synthesis report and Synthesized Circuit:
![incomp_if rprt_ckt](https://user-images.githubusercontent.com/100671647/218280095-9c1f6a0b-547f-4c32-8cd9-1439bc6cb535.png)
  
 
Similarly unwanted latches are inferred for incomplete case statements as well.


### FOR loop and FOR GENERATE
For loops are always written inside "always" statements. The syntax for "for" loop is similar to that in C. For loops are used where multiple evaluating statements need to be run.  
For example:  
_Demux using for loop_
![demux_gen logic](https://user-images.githubusercontent.com/78468534/120116279-17eb6700-c1a5-11eb-9c1a-0b65321bd293.jpeg)

  
For generate statements are written outside "always" statement. It is used for instantiating a module multiple times within an RTL.  
For example:  
_Ripple carry adder using for generate_
![rca logic](https://user-images.githubusercontent.com/78468534/120116313-39e4e980-c1a5-11eb-945e-9cd88ea0a5ed.jpeg)

  
  
## Acknowledgement
* [Kunal Ghosh](https://github.com/kunalg123)
* [Shon Taware](https://github.com/ShonTaware)
* [Sumanto Kar](https://github.com/Eyantra698Sumanto)
