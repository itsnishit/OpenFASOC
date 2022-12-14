# OpenFASOC
FASOC in OpenFASOC stands for Fully-Autonomous SoC. It is used for automating the Synthesis process using Customizable Cell-Based Synthesizable Analog Circuits. OpenFASOC is focused on developing a complete system-on-chip (SoC) synthesis tool from user specification to GDSII using fully open-sourced tools. This project is led by a team of researchers at the University of Michigan is inspired from FASoC whcih sits on proprietary tools. 

# 1.1 OpenFASOC Design Flow
(https://user-images.githubusercontent.com/86912339/207599894-7d1f079a-c910-437c-9514-d036af778416.png)
The OpenFASOC Design flow starts by taking the design specifications in the form of `.json` format. Then the OpenFASOC generator determines the number of auxillary cell to be added to optimize the design. The generator uses the model file model file to automatically determine the number of Aux-Cell to be added. Here, the model file is saved in the form of a `.csv` file.

In other words, the generator iteratively searches the the model file to find the optimum number of auxillary cells to be added. After finding the optimum structure, the behavioral verilog description is created. Then the synthesis tool `yosys` maps the behavioral verilog to the structural verilog. This step is followed by place and route step. Place and Route is performed by OpenROAD tool. This step is followed by Design Rule Checks (DRC), Layout Vs Schematic (LVS) and Parasitic Extraction (PEX). Unlike traditional Analog-Design flow, in OpenFASOC flow, the complete process of aux-cell generation to layout generation is automated which significantly reduced the design time. 


# 1.2 Working & Analysis of Temperature Sensor Generator circuit
An all-digital temperature sensor, that relies on a new subthreshold oscillator (achieved using the auxiliary cell “Header Cell“) for realizing synthesizable thermal sensors.
The way that works is we have a subthreshold current that has an exponential dependency on the temperature, the frequency generated from the subthreshold ring oscillator is also dependent on temperature. So we can sense the temperature by comparing the difference between the clock frequency generated from a reference oscillator and the clock frequency from the proposed frequency generator.

## Circuit Schematic
