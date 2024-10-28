<img src="resources/beckhoff_nat.png" width="400"/>

# Beckhoff TwinCAT TE1421 Simulation Runtime for FMI

This repository provides sample FMUs exported from the [Beckhoff TwinCAT TE1421 Simulation Runtime for FMI](https://www.beckhoff.com/en-us/products/automation/twincat/texxxx-twincat-3-engineering/te1421.html).
These FMUs (PneumaticCylinderController2.fmu for FMI 2 and PneumaticCylinderController3.fmu for FMI 3) contain a preconfigured TwinCAT UserMode Runtime for controlling a pneumatic cylinder model (PneumaticCylinderModel2.fmu for FMI 2 and PneumaticCylinderModel3.fmu for FMI 3) in a SiL-Simulation.
In addition to the FMUs, this repository also provides the TwinCAT project with the control algorithm.

Further documentation on the required TwinCAT installation, the control algorithm and the penumatic cylinder physics are available at [Beckhoff InfoSys](https://infosys.beckhoff.com/index.php?content=../content/1031/te1421_tc3_simulation_runtime_for_fmi/index.html&id=6985113427065552234).

## FMU Export Compatibility information

All provided FMUs support FMI 2/3 and implement both interface types (Co-Simulation and Model-Exchange). The FMUs are distributed as binary FMUs for 64-Bit Windows only.

### Validation Tools

- fmpy

### Importing Tools

- Dymola
- MapleSim
- OpenModelica
- Simulink&reg;

# Beckhoff TwinCAT TE1420 Target for FMI

Further documentation on the [Beckhoff TwinCAT TE1420 Target for FMI](https://www.beckhoff.com/en-us/products/automation/twincat/texxxx-twincat-3-engineering/te1420.html) is available at [Beckhoff InfoSys](https://infosys.beckhoff.com/index.php?content=../content/1031/te1420_tc3_target_fmi/index.html&id=).

## FMU Import Compatibility information

The TwinCAT FMU import requires source code FMUs, it supports FMI 2/3 and both interface types (Co-Simulation and Model-Exchange).

### Exporting Tools

- Dymola
- MapleSim
- [Reference FMUs](https://github.com/modelica/Reference-FMUs)
- Simulink&reg;

## License

Copyright &copy; Beckhoff 2024.
All rights reserved.
The models and accompanying materials may only be used for testing and validation of FMI implementations.