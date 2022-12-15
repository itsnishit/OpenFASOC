# OpenFASOC
FASOC in OpenFASOC stands for Fully-Autonomous SoC. It is used for automating the Synthesis process using Customizable Cell-Based Synthesizable Analog Circuits. OpenFASOC is focused on developing a complete system-on-chip (SoC) synthesis tool from user specification to GDSII using fully open-sourced tools. This project is led by a team of researchers at the University of Michigan is inspired from FASoC whcih sits on proprietary tools. 

# 1.1 OpenFASOC Design Flow
![image](https://user-images.githubusercontent.com/86912339/207783838-34f8b847-93f0-454c-b0be-bc1e8429e39c.png)

The OpenFASOC Design flow starts by taking the design specifications in the form of `.json` format. The OpenFASOC uses an approach involving auxiliary cells. Auxiliary cells are manually designed standard cells which can act as reusable components in different analog circuits. Then the OpenFASOC generator determines the number of auxillary cell to be added to optimize the design. The generator uses the model file model file to automatically determine the number of Aux-Cell to be added. Here, the model file is saved in the form of a `.csv` file.

In other words, the generator iteratively searches the the model file to find the optimum number of auxillary cells to be added. After finding the optimum structure, the behavioral verilog description is created. Then the synthesis tool `yosys` maps the behavioral verilog to the structural verilog. This step is followed by place and route step. Place and Route is performed by OpenROAD tool. This step is followed by Design Rule Checks (DRC), Layout Vs Schematic (LVS) and Parasitic Extraction (PEX). Unlike traditional Analog-Design flow, in OpenFASOC flow, the complete process of aux-cell generation to layout generation is automated which significantly reduced the design time. 


# 1.2 Working & Analysis of Temperature Sensor Generator circuit
An all-digital temperature sensor, that relies on a new subthreshold oscillator (achieved using the auxiliary cell “Header Cell“) for realizing synthesizable thermal sensors.
The way that works is we have a subthreshold current that has an exponential dependency on the temperature, the frequency generated from the subthreshold ring oscillator is also dependent on temperature. So we can sense the temperature by comparing the difference between the clock frequency generated from a reference oscillator and the clock frequency from the proposed frequency generator.

## Circuit Schematic
![image](https://user-images.githubusercontent.com/86912339/207783952-497b5423-a336-4c30-890e-0aba24745613.png)


## Auxiliary Cells
The physical implementation of the analog blocks in the circuit is done using two manually designed standard cells(also called auxiliary cells):
1.  HEADER cell, containing the transistors in subthreshold operation;
2.  SLC cell, containing the Split-Control Level Converter.

The gds and lef files of HEADER and SLC cells are pre-created before the start of the Generator flow. 

The layout of the HEADER cell is shown below:
![image](https://user-images.githubusercontent.com/86912339/207790293-73e76139-1f87-48de-aa11-94d2ef3c8bb2.png)

The layout of the SLC cell is shown below:
![image](https://user-images.githubusercontent.com/86912339/207790397-06b138c4-3092-46ce-aa5e-5f74d86fa342.png)

## OpenFASOC Flow
The Block diagram of the OpenFASOC flow is given as follows:
![image](https://user-images.githubusercontent.com/86912339/207791563-9c94c32b-6724-4b7d-ba67-f274570c04f2.png)

### Verilog Generation
Running make sky130hd_temp (temp for “temperature sensor”) executes the temp-sense-gen.py script from temp-sense-gen/tools/. This file takes the input specifications from test.json and outputs(generates) Verilog files containing the description of the circuit. The test.json file allows the user to customize operating range of temperature (takes minimum and maximum temperature), power, area, error and otimization (can select priority).

The below attached is the test.json file corresponding to temperature sensor:
![image](https://user-images.githubusercontent.com/86912339/207794031-37cf6adb-a9d4-45f2-a8f3-7db2feb25b42.png)

Now, to generate the verilog files as per input specs, run the command:
```
make sky130hd_temp_verilog
```
![verilog_gen_1](https://user-images.githubusercontent.com/86912339/207805576-47f64b79-fd06-4bbe-8ead-0c8e3b4f20d2.png)
![image](https://user-images.githubusercontent.com/86912339/207805660-0b0fc010-8404-40b9-9f3c-e582989e73e4.png)
As it is visible, the Error value, number of Inverters, number of Headers etc. are obtained. Once the Verilog file is generated successfully, the message is displayed on the screen.

### Automation for remaining flow
The remaining steps are automated by running the command:
```
make sky130hd_temp
```

