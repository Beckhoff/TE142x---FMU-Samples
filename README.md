<picture>
  <source media="(prefers-color-scheme: dark)" srcset="resources/beckhoff_nat_dark.png" width="400">
  <source media="(prefers-color-scheme: light)" srcset="resources/beckhoff_nat_light.png" width="400">
  <img alt="Beckhoff New Automation.">
</picture>

# Beckhoff TwinCAT TE1421 Simulation Runtime for FMI

This repository provides sample FMUs exported from the [Beckhoff TwinCAT TE1421 Simulation Runtime for FMI](https://www.beckhoff.com/en-us/products/automation/twincat/texxxx-twincat-3-engineering/te1421.html).
These FMUs (PneumaticCylinderController2.fmu for FMI 2 and PneumaticCylinderController3.fmu for FMI 3) contain a preconfigured TwinCAT Usermode Runtime for controlling a pneumatic cylinder model (PneumaticCylinderModel2.fmu for FMI 2 and PneumaticCylinderModel3.fmu for FMI 3) in a SiL-Simulation.
In addition to the FMUs, this repository also provides the TwinCAT project with the control algorithm.

The [pneumatic cylinder model](#model) and the [control algorithm](#control) is documented below.
Further documentation on the required TwinCAT installation and on how to use the TE1421 is available at [Beckhoff InfoSys](https://infosys.beckhoff.com/index.php?content=../content/1033/te1421_tc3_simulation_runtime_for_fmi/index.html&id=6985113427065552234).

## FMU Export Compatibility information

The TwinCAT Runtime and the pneumatic cylinder model are provided as FMI2 and FMI3 FMUs. They implement both interface types (Co-Simulation and Model-Exchange). The TwinCAT Runtime FMUs are distributed as binary FMUs for 64-Bit Windows only, the pneumatic cylinder FMU provides binaries for 64-Bit Windows and also the source code.

### Validation Tools

- [fmpy](https://github.com/CATIA-Systems/FMPy)

### Importing Tools

- Dymola
- MapleSim
- Simulink&reg;

# Beckhoff TwinCAT TE1420 Target for FMI

Further documentation on the [Beckhoff TwinCAT TE1420 Target for FMI](https://www.beckhoff.com/en-us/products/automation/twincat/texxxx-twincat-3-engineering/te1420.html) is available at [Beckhoff InfoSys](https://infosys.beckhoff.com/index.php?content=../content/1033/te1420_tc3_target_fmi/index.html&id=).

## FMU Import Compatibility information

The TwinCAT FMU import requires source code FMUs, it supports FMI 2/3 and both interface types (Co-Simulation and Model-Exchange).

### Exporting Tools

- Dymola
- MapleSim
- [Reference FMUs](https://github.com/modelica/Reference-FMUs)
- Simulink&reg;

# Pneumatic cylinder example

Using the example of a _single-acting_ pneumatic cylinder and the associated position controller, a real-world SiL simulation scenario is elaborated below. The cylinder is modeled using the momentum conservation and the ideal gas law. The controller is derived using the method of _exact state linearization_.

## Model
The pneumatic cylinder to be modeled is a so-called _single-acting_ cylinder, i.e. the cylinder is pressurized with air from one side only and returned to its home position by spring force. To simplify matters, the system is modeled without a valve. 
The physical model of the pneumatic cylinder is shown in the following figure.

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="resources/PneumaticCylinderSketch_dark.svg" width="300">
  <source media="(prefers-color-scheme: light)" srcset="resources/PneumaticCylinderSketch_light.svg" width="300">
  <img alt="Physical abstraction of a pneumatic cylinder.">
</picture>

In this image, $m$ is the piston mass, $c$ the spring stiffness and $A$ the piston area. 
The piston position is denoted by $x$ and the air pressure in the cylinder is $p$. 
The air volume flow $q$ is the input variable.

The momentum conservation theorem yields the following formula for the piston movement

```math
\begin{align*}
m \cdot \ddot{x} &= F_p - F_c \\\
m \cdot \ddot{x} &= p \cdot A - c \cdot x.
\end{align*}
```

Here, $F_p$ is the force on the piston resulting from the pressure $p$ and $F_c$ is the spring's force of restitution. Furthermore, the ideal gas law is denoted by the formula bellow.

```math
\begin{align*}
p \cdot V = n \cdot R \cdot T
\end{align*}
```

Assuming an isothermal change of state, this formula can be rewritten as follows.

```math
\begin{align*}
\frac{p \cdot V}{n} = R \cdot T = \mathrm{const.}
\end{align*}
```

This equation can be used to derive a differential equation for the air pressure in the pneumatic cylinder. Derivation of the ideal gas equation (for the case of isothermal change of state) with respect to time yields

```math
\begin{align*}
\frac{\dot{p} \cdot V}{n} + \frac{p \cdot \dot{V}}{n} - \frac{p \cdot V \cdot \dot{n}}{n^2} = 0.
\end{align*}
```

With $q = \frac{V \cdot \dot{n}}{n}$ and $V = A \cdot x$ the air pressure dynamic then becomes

```math
\begin{align*}
\dot{p} &= \frac{p}{x} \cdot \left( \frac{q}{A} - \dot{x} \right).
\end{align*}
```

The differential equation system describing the overall system is therefore denoted by the following formula.

```math
\boxed{
\begin{align*}
m \cdot \ddot{x} &= p \cdot A - c \cdot x \\\
\dot{p} &= \frac{p}{x} \cdot \left( \frac{q}{A} - \dot{x} \right)
\end{align*}
}
```

The piston position $x$, the piston speed $\dot{x}$ and the cylinder air pressure $p$ can be used as state variables, resulting in the state vector $`\underline{x} = \begin{bmatrix} x & \dot{x} & p \end{bmatrix}^\mathrm{T} = \begin{bmatrix} x_1 & x_2 & x_3 \end{bmatrix}^\mathrm{T} `$. Assuming the position $y = x_1$ is ideally measured and the input variable is given by $u = q$, the overall system can be rewritten in the following non-linear state space representation.

```math
\boxed{
\begin{align*}
\dot{\underline{x}} &= 
\begin{bmatrix} 
x_2 \\\
-\frac{c}{m} \cdot x_1 + \frac{A}{m} \cdot x_3 \\\
-\frac{x_2 \cdot x_3}{x_1}
\end{bmatrix}
+
\begin{bmatrix} 
0 \\\
0 \\\
\frac{1}{A} \cdot \frac{x_3}{x_1}
\end{bmatrix}
\cdot u \\\
\underline{y} &= \begin{bmatrix} 1 & 0 & 0 \end{bmatrix} \cdot \underline{x}
\end{align*}
}
```

The system is singular in $x_1 = 0$. $x_3 = 0$ decouples the input variable. Saturating these two states by a lower limit is therefore mathematically necessary. In addition, pneumatic cylinders are limited by design, thus $x_1$ and $x_3$ must also have an upper limit. The model parameters are summarized in the following table.

| **Parameter**  | **Value**       | **Unit**            | **Description**                   |
| :------------- | :-------------- | :-----------------  | :-------------------------------- | 
| $m$            | $1$             | $\mathrm{kg}$       | Piston mass                       |
| $c$            | $2\cdot10^3$    | $\mathrm{N m^{-1}}$ | Spring stiffness                  |
| $A$            | $5\cdot10^{-3}$ | $\mathrm{m^2}$      | Piston area                       |
| $x(t=0)$       | $0.02$          | $\mathrm{m}$        | Initial position of the piston    |
| $x_{min}$      | $0.01$          | $\mathrm{m}$        | Lower piston position limit       |
| $x_{max}$      | $0.2$           | $\mathrm{m}$        | Upper piston position limit       |
| $\dot{x}(t=0)$ | $0$             | $\mathrm{ms^{-1}}$  | Initial piston speed              |
| $p(t=0)$       | $1\cdot10^3$    | $\mathrm{Pa}$       | Initial cylinder air pressure     |
| $p_{min}$      | $100$           | $\mathrm{Pa}$       | Lower cylinder air pressure limit |
| $p_{max}$      | $1\cdot10^6$    | $\mathrm{Pa}$       | Upper cylinder air pressure limit |

It is recommended to use an Euler integrator with a step size of $`h = 1\cdot10^{-3} \ \mathrm{s} `$ for the simulation.

## Control

Position control is a common requirement for a linear actuator. A PI controller is commonly used for this task. However, due to the non-linear air pressure dynamic, it is reasonable to expect that such a linear controller design will not provide a particularly high control bandwidth. A better approach is to take the non-linearity into account when designing the controller. A controller design method considering these non-linearities is the so-called _exact state linearization_.

First, the differential order $\delta$ is determined to perform an _exact state linearization_. The output equation $y$ is differentiated with respect to time until the input $u$ has an effect on a derivative of $y$.

```math
\begin{align*}
y &= x_1 \\\
\dot{y} &= \dot{x}_1 = x_2 \\\
\ddot{y} &= \dot{x}_2 = -\frac{c}{m} \cdot x_1 + \frac{A}{m} \cdot x_3 \\\
y^{(3)} &= -\frac{c}{m} \cdot \dot{x}_1 + \frac{A}{m} \cdot \dot{x}_3 \\\
&= -\frac{c}{m} \cdot x_2 - \frac{A}{m} \cdot \frac{x_2 \cdot x_3}{x_1} + \frac{1}{m} \cdot \frac{x_3}{x_1} \cdot u
\end{align*}
```

Here, $u$ has an effect on $y^{(3)}$, i.e. the difference order is $\delta=3$. Since the number of states is equal to the difference order, the _exact state linearization_ can be carried out by transforming the system to so-called _nonlinear control canonical form_. For this purpose, the new states $`\underline{z} = \begin{bmatrix} y & \dot{y} & \ddot{y} \end{bmatrix}^\mathrm{T} = \begin{bmatrix} z_1 & z_2 & z_3 \end{bmatrix}^\mathrm{T} `$ are selected. With the new state vector

```math
\begin{align*}
\underline{z} &=
\begin{bmatrix} 
x_1 \\\
x_2 \\\
-\frac{c}{m} \cdot x_1 + \frac{A}{m} \cdot x_3
\end{bmatrix}.
\end{align*}
```

The system is then denoted by the equation system bellow.

```math
\boxed{
\begin{align*}
\underline{\dot{z}} &=
\begin{bmatrix} 
z_2 \\\
z_3 \\\
- \frac{c}{m} \cdot x_2 - \frac{A}{m} \cdot \frac{x_2 \cdot x_3}{x_1}
\end{bmatrix}
+
\begin{bmatrix} 
0 \\\
0 \\\
\frac{1}{m} \cdot \frac{x_3}{x_1}
\end{bmatrix}
\cdot u
\end{align*}
}
```

The following control law is used for mathematical compensation of the non-linearity.

```math
\boxed{
\begin{align*}
u = m \cdot \frac{x_1}{x_3} \cdot \left( \frac{c}{m} \cdot x_2 + \frac{A}{m} \cdot \frac{x_2 \cdot x_3}{x_1} + v \right)
\end{align*}
}
```

The state linearized system has a new virtual input $v$, triple integrator behaviour $z^{(3)}_1 = v$ and is therefore unstable. A state controller can be used to stabilize the linearized system.

```math
\boxed{
\begin{align*}
v = - \underline{r}^T \cdot \underline{z} + f \cdot w
\end{align*}
}
```

In the state controller equation, $\underline{r}$ is the controller matrix, $f$ the reference gain and $w$ the reference input. The closed control loop with this state controller has the following dynamics.

```math
\begin{align*}
z^{(3)}_1 &= - r_1 \cdot z_1 - r_2 \cdot z_2 - r_3 \cdot z_3 + f \cdot w \\\
y^{(3)} &= - r_1 \cdot y - r_2 \cdot \dot{y} - r_3 \cdot \ddot{y} + f \cdot w \\\
\mathcal{L} \left\{ y^{(3)} \right\} &= \mathcal{L} \left\{ - r_1 \cdot y - r_2 \cdot \dot{y} - r_3 \cdot \ddot{y} + f \cdot w \right\} \\\
s^3 \cdot Y(s) &= - r_1 \cdot Y(s) - r_2 \cdot s \cdot Y(s) - r_3 \cdot s^2 \cdot Y(s) + f \cdot W(s)
\end{align*}
```

The closed control loop dynamic is a low pass of order three and can be denoted by the transfer function below.

```math
\begin{align*}
G_w(s) = \frac{Y(s)}{W(s)} = \frac{f}{s^3 + r_3 \cdot s^2 + r_2 \cdot s + r_1}
\end{align*}
```

For calculating the controller gains and the reference gain, the closed control loop dynamic is required to have a triple real eigenvalue in $\lambda \in \mathbb{R}^-$ and an overall gain of $|G_w(s)| = 1$.

```math
\begin{align*}
G_w(s) = \frac{f}{s^3 + r_3 \cdot s^2 + r_2 \cdot s + r_1} &\stackrel{!}{=} \frac{-\lambda^3}{(s-\lambda)^3}
\end{align*}
```

Using this requirement then yields the controller parameters as follows.

```math
\boxed{
\begin{align*}
r_1 &= - \lambda^3 \\\
r_2 &= 3 \cdot \lambda^2 \\\
r_3 &= - 3 \cdot \lambda \\\
f &= - \lambda^3
\end{align*}
}
```

The resulting control loop is stable and steady-state accurate.

## Control Loop

The control loop schema is provided in the following figure. The controller output $q$ (air volume flow) is connected to the model's input. The model's output $\underline{x}$ (piston position $x$, piston velocity $\dot{x}$ and air pressure $p$) is connected to the controller's input. The controller's setpoint $w$ is provided by the user from within TwinCAT.

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="resources/ControlLoopDiagram_dark.svg" width="300">
  <source media="(prefers-color-scheme: light)" srcset="resources/ControlLoopDiagram_light.svg" width="300">
  <img alt="Control loop block diagram.">
</picture>

# License

Copyright &copy; Beckhoff 2025.
All rights reserved.
The models and accompanying materials may only be used for testing and validation of FMI implementations.
