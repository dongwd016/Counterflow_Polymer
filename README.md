# Counterflow Polymer

A Cantera implementation of the counterflow solid fuel regression rate calculation.  The algorithm is given in "G. Talamantes, Characterization of polyoxymethylene as a high-density fuel for use in hybrid rocket applications, Ph.D. thesis, The Pennsylvania State University (2019)."

## Schematic

<img src="img/counterflow_expt.png" width="400"/><img src="img/counterflow_schematic.png" width="300"/>

## Method

The regression rate, $r_b$, is calculated by considering energy and mass conservations. Energy conservation is given by:

```math
\dot{m}h_{\rm POM}^{T_i}-\dot{m}h_{\rm CH_2O}^{T_S}+\dot{Q}_{\rm cond}=0,
```

where $\dot{m}$ is the polymer fuel consumption rate, $T_i$ is the inlet solid POM temperature, $h_{\rm POM}^{T_i}$ is the POM enthalpy at temperature $T_i$, $T_S$ is the outlet gas phase CH<sub>2</sub>O temperature, $h_{\rm CH_2O}^{T_S}$ is the gas CH<sub>2</sub>O enthalpy at temperature $T_S$ and $\dot{Q}_{\rm cond}$ is the heat transfer rate. The regression rate can be obtained from the mass flow rate as

```math
\tag{1}
r_b=\frac{\dot{m}}{\rho_{\rm solid}}=\frac{\dot{Q}_{\rm cond}}{\Delta h\rho_{\rm solid}}=\frac{k_{\rm gas}\left.\frac{\partial T}{\partial x}\right|_{x=0}}{(h_{\rm POM}^{T_i}-h_{\rm CH_2O}^{T_S})\rho_{\rm solid}},
```

where $\rho_{\rm solid}$ is the solid POM density and $k_{\rm gas}$ is the gas thermal conductivity.

The regression rate can also be calculated from mass continuity.

```math
\tag{2}
\tilde{r_b}=\frac{\rho_{\rm gas}v_f}{\rho_{\rm solid}},
```

where $v_f$ is the gas phase fuel inlet velocity and $\rho_{\rm gas}$ is the gas density.

Required boundary conditions to solve the counterflow problem are
- Oxidizer inlet composition: provided. Pure oxygen for example.
- Oxidizer inlet temperature: provided. 298 K for example.
- Oxidizer inlet velocity: provided.
- Fuel inlet composition: provided. Pure POM.
- Fuel inlet temperature: provided. 700K for example. The regression rate is found to be insensitive to fuel inlet temperature under the heat transfer dominating assumption.
- Fuel inlet velocity: Unknown.

Fuel inlet velocity is the only unknown boundary condition. The algorithm first places a guess for fuel inlet velocity, for instance, equal to the oxidizer inlet velocity. Cantera CounterflowDiffusionFlame is then used to solve the counterflow problem. The solution provides gas phase density at the fuel side boundary, which allows the calculation of the regression rate from equation (2). The solution also provides the gas phase thermal conductivity and the temperature gradient at the fuel side boundary, which give the conductive heat flux. Subsequently, the regression rate can be calculated from equation (1). The goal is to find a fuel inlet velocity to match these two regression rates. Iterations are performed until the difference between two regression rates is less than a given tolerence, $10^{-9}$ m/s for example. At each iteration, the fuel inlet velocity is updated using the expression:

```math
v_{f,\rm new}=v_{f,\rm old}\frac{\tilde{r_b}+s(r_b-\tilde{r_b})}{\tilde{r_b}},
```

where $s$ is a scaling factor to assure convergence. $s=0.3$ for example.

## Example

Sample code is procided in the jupyter notebook file, "example/counterflow.ipynb". It reproduces Fig. 26 of the original thesis using Cantera CounterflowDiffusionFlame.

## Contributors

Wendi Dong, Hai Wang

Mechanical Engineering Department, Stanford University, Stanford, California 94305, United States

