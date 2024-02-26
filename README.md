# Quaisi One-Dimensional Counterflow Flame of Polymer Combustion

A Cantera implementation of the counterflow solid fuel regression rate calculation.  The algorithm is given in "G. Talamantes, Characterization of polyoxymethylene as a high-density fuel for use in hybrid rocket applications, Ph.D. thesis, The Pennsylvania State University (2019)."

## Schematic

The schematic uses polyoxymethylene (POM) as example:

<img src="img/counterflow_expt.png" width="400"/><img src="img/counterflow_schematic.png" width="300"/>

## Method

The regression rate, $r_b$, is calculated by considering energy and mass conservations. Energy conservation is given by:

```math
\dot{m}h(T_i)-\dot{m}h(T_S)+\dot{Q}_{\rm cond}=0,
```

where $\dot{m}$ is the polymer fuel consumption rate, $T_i$ is the solid polymer temperature at a large distance removed from the burning surface, $h(T_i)$ is the total enthalpy of the polymer at temperature $T_i$, $T_S$ is the gas-condensed phase interface temperature, $h(T_S)$ is the total enthalpy of the gas-phase polymer decomposition product at temperature $T_S$ and $\dot{Q}_{\rm cond}$ is the heat transfer rate. The regression rate can be obtained from the mass flow rate as

```math
\tag{1}
r_b=\frac{\dot{m}}{\rho_{\rm solid}}=\frac{\dot{Q}_{\rm cond}}{\Delta h\rho_{\rm solid}}=\frac{k_{\rm gas}\left.\frac{\partial T}{\partial x}\right|_{x=0}}{[h(T_i)-h(T_S)]\rho_{\rm solid}},
```

where $\rho_{\rm solid}$ is the mass density of the polymer, and $k_{\rm gas}$ is the gas thermal conductivity.

The regression rate can also be calculated from mass conservation:

```math
\tag{2}
\tilde{r_b}=\frac{\rho_{\rm gas}v_f}{\rho_{\rm solid}},
```

where $v_f$ is the gas phase fuel inlet velocity and $\rho_{\rm gas}$ is the gas density.

Required boundary conditions to solve the counterflow problem are
- Oxidizer inlet composition: provided. pure oxygen as example.
- Oxidizer inlet temperature: provided. 298 K as example.
- Oxidizer inlet velocity: provided.
- "Fuel inlet" composition: provided. POM assumedf.
- "Fuel inlet" temperature: provided. 700K as example. The regression rate is found to be insensitive to fuel inlet temperature under the heat transfer dominating assumption.
- "Fuel inlet" velocity: a dependent variable to be solved.

Fuel inlet velocity is treated as a boundary condition, which is solved. The algorithm first makes a guess for the "fuel inlet" velocity (which corrrespond to a given polymer regression rate), for instance, equal to the oxidizer inlet velocity. Cantera CounterflowDiffusionFlame then solves the counterflow problem, and the solution provides density at the fuel side boundary, which allows the calculation of the regression rate from equation (2). The solution also provides the gas-phase thermal conductivity and the temperature gradient at the fuel side boundary, which enables an calculation for the conductive heat flux. Subsequently, the regression rate is determined from equation (1). The goal is to find a "fuel inlet" velocity iteratuvely to match the regression rates. Iterations are made until the difference between the two regression rates is < a given tolerence, e.g., $10^{-9}$ m/s. At each iteration, the fuel inlet velocity is updated using the expression:

```math
v_{f,\rm new}=v_{f,\rm old}\frac{\tilde{r_b}+s(r_b-\tilde{r_b})}{\tilde{r_b}},
```

where $s$ is a scaling factor to assure convergence. $s=0.3$ as example.

## Example

Sample code is provided in the jupyter notebook file, "example/counterflow.ipynb". It reproduces Fig. 26 of Talamantes' thesis using Cantera CounterflowDiffusionFlame.

## Contributors

Wendi Dong, Hai Wang

Mechanical Engineering Department, Stanford University, Stanford, California 94305, United States

