# OpenFASOC
FASOC in OpenFASOC stands for Fully-Autonomous SoC. It is used for automating the Synthesis process using Customizable Cell-Based Synthesizable Analog Circuits. OpenFASOC is focused on developing a complete system-on-chip (SoC) synthesis tool from user specification to GDSII using fully open-sourced tools. This project is led by a team of researchers at the University of Michigan is inspired from FASoC whcih sits on proprietary tools. 

# 1.1 Installation

## OpenFASOC
To install openfasoc run:
```
git clone https://github.com/idea-fasoc/openfasoc
cd openfasoc
pip install -r requirements.txt
```
## OpenROAD
OpenROAD is an integrated chip physical design tool that takes a design from synthesized Verilog to routed layout. 
```
git clone --recursive https://github.com/The-OpenROAD-Project/OpenROAD.git
cd OpenROAD
./etc/DependencyInstaller.sh
./etc/DependencyInstaller.sh -run
./etc/DependencyInstaller.sh -dev
mkdir build
cd build
cmake ..
make
```
If the ./ command fails to execute, run the same command using "sudo" whcich will give permission to files which are unable to open.

## Klayout
```
sudo apt install Klayout
```

## Netgen
```
sudo add-apt-repository ppa:ngsolve/ngsolve
sudo apt-get update
sudo apt-get install ngsolve
```

## Yosys
Yosys is a framework for Verilog RTL synthesis. It currently has extensive Verilog-2005 support and provides a basic set of synthesis algorithms for various application domains. 
Install the prerequisites as:

```
sudo apt-get install build-essential clang bison flex \
libreadline-dev gawk tcl-dev libffi-dev git \
graphviz xdot pkg-config python3 libboost-system-dev \
libboost-python-dev libboost-filesystem-dev zlib1g-dev
```
Install Yosys as:
```
git clone https://github.com/YosysHQ/yosys.git
make
sudo make install 
make test
```



# 1.2 OpenFASOC Design Flow
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
#### SYNTHESIS
The OpenROAD Flow starts with a flow configuration file (config.mk), the chosen platform (sky130hd, for example) and the Verilog files generated from the previous part. From them, synthesis is run using Yosys to find the appropriate circuit implementation from the available cells in the platform.The synthesis is run using Yosys to find the appropriate circuit implementation from the available cells in the platform. The synthesized verilog netlist is shown below in the following figure:

#### FLOORPLAN
Once Synthesis is successfully completed, floorplaaning is done. hen, the floorplan for the physical design is generated with OpenROAD, which requires a description of the power delivery network (in pdn.cfg).
This temperature sensor design implements two voltage domains: one for the VDD that powers most of the circuit, and another for the VIN that powers the ring oscillator and is an output of the HEADER cells. Such voltage domains are created within the floorplan.tcl script


#### PLACEMENT
Placement takes place after the floorplan is ready and has two phases: global placement and detailed placement. The output of this phase will have all instances placed in their corresponding voltage domain, ready for routing.


#### ROUTING
Routing is also divided into two phases: global routing and detailed routing. Right before global routing, OpenFASoC calls pre_global_route.tcl:
![image](https://user-images.githubusercontent.com/86912339/207863251-dcfb9863-99d4-4c71-93aa-b5937348c101.png)

This script sources two other files: add_ndr_rules.tcl, which adds an NDR rule to the VIN net to improve routes that connect both voltage domains, and create_custom_connections.tcl, which creates the connection between the VIN net and the HEADER instances.

The Global route power and area report is shown as:

The Finished power and area report is shown as:


The final design after Routing is shown as:
