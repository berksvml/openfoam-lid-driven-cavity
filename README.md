<div align="center">

# Lid-Driven Cavity Flow

### A Finite Volume CFD Case Built from Scratch with OpenFOAM 13

![OpenFOAM](https://img.shields.io/badge/OpenFOAM-13-blue?style=for-the-badge)
![CFD](https://img.shields.io/badge/CFD-Finite%20Volume-orange?style=for-the-badge)
![Flow](https://img.shields.io/badge/Flow-Incompressible-green?style=for-the-badge)
![Model](https://img.shields.io/badge/Model-Laminar-purple?style=for-the-badge)

![Ubuntu](https://img.shields.io/badge/Ubuntu-24.04-E95420?style=flat-square&logo=ubuntu&logoColor=white)
![WSL2](https://img.shields.io/badge/Platform-WSL2-0078D4?style=flat-square&logo=windows&logoColor=white)
![ParaView](https://img.shields.io/badge/Post--Processing-ParaView-3B4D98?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)

</div>

---

## Overview

This repository contains a two-dimensional incompressible lid-driven cavity CFD case independently constructed from scratch using **OpenFOAM 13**.

The main objective of this project was to understand the complete structure of an OpenFOAM case rather than simply running an existing tutorial.

The case was manually configured, including:

- mesh generation,
- initial and boundary conditions,
- physical properties,
- momentum transport model,
- numerical discretization schemes,
- linear solver configuration,
- pressure-velocity coupling,
- transient solution control,
- and CFD post-processing.

The final solution was compared against a reference laminar OpenFOAM cavity case using the vertical centreline velocity profile, `Ux(y)`, as a reproduction and verification exercise.

---

## Project Highlights

- Built the complete OpenFOAM case structure manually
- Generated a structured hexahedral mesh using `blockMesh`
- Defined velocity and pressure fields from scratch
- Configured all boundary conditions manually
- Used the `incompressibleFluid` solver framework
- Applied a laminar momentum transport model
- Configured finite-volume discretization schemes
- Used GAMG for pressure and `smoothSolver` for velocity
- Applied PIMPLE pressure-velocity coupling
- Investigated Courant number sensitivity during mesh refinement
- Visualized velocity magnitude, vectors, and streamlines in ParaView
- Extracted the vertical centreline velocity profile
- Compared the independently constructed case against a reference OpenFOAM case

---

## Problem Description

The lid-driven cavity is a classical CFD benchmark problem.

The computational domain consists of a square cavity with a moving upper wall.

The upper wall moves in the positive x-direction with a constant velocity and transfers momentum to the fluid.

The remaining walls are stationary and satisfy the no-slip condition.

The motion of the upper wall generates a primary clockwise recirculation region inside the cavity.

### Flow Configuration

```text
                    U = 1 m/s
             ───────────────────→
             ┌───────────────────┐
             │                   │
             │        ↻          │
             │                   │
             │                   │
             └───────────────────┘

               Fixed no-slip walls
```

---

## Physical Parameters

| Parameter | Value |
|---|---|
| Cavity width | 0.1 m |
| Cavity height | 0.1 m |
| Lid velocity | 1 m/s |
| Kinematic viscosity | 1 × 10⁻⁵ m²/s |
| Reynolds number | 10,000 |
| Flow model | Laminar |
| Flow type | Incompressible |
| Simulation type | Transient |

The Reynolds number is defined as:

```text
Re = U L / ν
```

For this case:

```text
U = 1 m/s
L = 0.1 m
ν = 1 × 10⁻⁵ m²/s
```

Therefore:

```text
Re = 10,000
```

---

## Repository Structure

```text
.
├── 0
│   ├── U
│   └── p
├── constant
│   ├── momentumTransport
│   └── physicalProperties
├── system
│   ├── blockMeshDict
│   ├── controlDict
│   ├── fvSchemes
│   └── fvSolution
├── .gitignore
├── LICENSE
└── README.md
```

### `0/`

Contains the initial field values and boundary conditions.

```text
0/U
```

Defines the velocity field.

```text
0/p
```

Defines the kinematic pressure field.

### `constant/`

Contains physical properties and physical model definitions.

```text
constant/physicalProperties
```

Defines the fluid kinematic viscosity.

```text
constant/momentumTransport
```

Defines the momentum transport model.

### `system/`

Contains the numerical and simulation control settings.

```text
system/blockMeshDict
```

Defines the computational geometry and mesh.

```text
system/controlDict
```

Defines the solver, time-step size, end time, and output settings.

```text
system/fvSchemes
```

Defines the finite-volume discretization schemes.

```text
system/fvSolution
```

Defines the linear solvers and pressure-velocity coupling parameters.

---

## Mesh

The computational mesh is generated using OpenFOAM's `blockMesh` utility.

The domain is represented using a single hexahedral block.

### Mesh Resolution

```text
20 × 20 × 1
```

This corresponds to:

```text
400 finite-volume cells
```

The front and back boundaries are defined using the `empty` boundary type.

As a result, the case is treated as a two-dimensional simulation.

### Mesh Definition

The main mesh configuration is:

```text
hex (0 1 2 3 4 5 6 7) (20 20 1) simpleGrading (1 1 1)
```

The mesh uses uniform grading in all directions.

```text
simpleGrading (1 1 1)
```

---

## Boundary Conditions

### Moving Wall

The upper wall moves in the positive x-direction.

```text
U = (1 0 0) m/s
```

OpenFOAM boundary condition:

```text
type fixedValue;
value uniform (1 0 0);
```

---

### Fixed Walls

The left, right, and bottom walls are stationary.

```text
type noSlip;
```

Therefore:

```text
U = (0 0 0)
```

at the stationary walls.

---

### Front and Back Boundaries

The front and back surfaces are defined as:

```text
type empty;
```

This allows the case to be treated as two-dimensional.

---

## Physical Model

The case uses a laminar momentum transport model.

The `momentumTransport` configuration is:

```text
simulationType laminar;
```

No RANS or LES turbulence model is used.

Therefore, no additional turbulence transport equations such as `k` or `epsilon` are solved.

---

## Numerical Setup

### Solver Framework

The OpenFOAM 13 solver framework used in this case is:

```text
incompressibleFluid
```

The solver is selected in `system/controlDict`.

---

### Time Integration

The transient time derivative is discretized using:

```text
Euler
```

This corresponds to a first-order implicit Euler time integration scheme.

The time-step size is:

```text
Δt = 0.005 s
```

The simulation runs until:

```text
t = 10 s
```

---

### Gradient Scheme

The default gradient scheme is:

```text
Gauss linear
```

---

### Momentum Convection

The momentum convection term is discretized using:

```text
Gauss limitedLinearV 1
```

The corresponding OpenFOAM entry is:

```text
div(phi,U) Gauss limitedLinearV 1;
```

The limiter provides additional numerical robustness compared with an unrestricted linear convection scheme.

---

### Laplacian Scheme

The Laplacian terms are discretized using:

```text
Gauss linear corrected
```

The `corrected` option applies a correction for mesh non-orthogonality.

---

### Pressure Solver

Pressure is solved using:

```text
GAMG
```

with:

```text
tolerance = 1e-06
relTol = 0.1
```

The final pressure correction uses:

```text
relTol = 0
```

---

### Velocity Solver

Velocity is solved using:

```text
smoothSolver
```

with:

```text
GaussSeidel
```

as the smoother.

The velocity solver tolerance is:

```text
1e-05
```

---

## Pressure-Velocity Coupling

The simulation uses the PIMPLE pressure-velocity coupling algorithm.

The configuration is:

```text
nCorrectors 2;
nNonOrthogonalCorrectors 0;
```

Two pressure-velocity correction loops are performed during each time step.

Because the mesh is structured and orthogonal, no additional non-orthogonal pressure corrections are used.

A pressure reference is defined using:

```text
pRefCell 0;
pRefValue 0;
```

This removes the pressure null-space present in a closed incompressible domain.

---

## Running the Simulation

### 1. Load the OpenFOAM Environment

```bash
source /opt/openfoam13/etc/bashrc
```

---

### 2. Navigate to the Case Directory

```bash
cd openfoam-lid-driven-cavity
```

---

### 3. Generate the Mesh

```bash
blockMesh
```

---

### 4. Check the Mesh

```bash
checkMesh
```

A successful mesh check should end with:

```text
Mesh OK.
```

---

### 5. Run the Simulation

```bash
foamRun
```

---

### 6. Check the Latest Simulation Time

```bash
foamListTimes -latestTime
```

---

### 7. Open the Case in ParaView

```bash
paraFoam
```

---

## Post-Processing

The results were analysed using ParaView.

The following post-processing techniques were used.

### Velocity Magnitude

The velocity magnitude field was visualized using:

```text
U
→ Magnitude
```

The highest velocity occurs near the moving upper wall.

---

### Velocity Vectors

The `Glyph` filter was used to display velocity vectors.

```text
Orientation Array = U
Scale Array = U
```

The vector field clearly shows a primary clockwise recirculation region.

---

### Streamlines

The `Stream Tracer` filter was used to visualize the flow topology.

The resulting streamlines show the main cavity vortex generated by the moving lid.

---

## Vertical Centreline Velocity Profile

A vertical line was defined at:

```text
x = 0.05 m
```

using ParaView's `Plot Over Line` filter.

The line coordinates were:

```text
Point1 = (0.05 0.00 0.005)
Point2 = (0.05 0.10 0.005)
```

The x-component of velocity:

```text
Ux
```

was extracted along the line.

The resulting velocity profile contains both positive and negative velocity values due to the recirculating flow.

The negative `Ux` region represents reverse flow in the lower part of the cavity.

---

## Verification

The OpenFOAM case in this repository was constructed independently from scratch.

The resulting solution was compared against a reference laminar OpenFOAM cavity case.

The vertical centreline velocity profile:

```text
Ux(y)
```

was extracted from both simulations using identical sampling coordinates.

The resulting velocity profiles were found to be nearly identical.

This indicates that the independently constructed case reproduces the same physical and numerical problem as the reference OpenFOAM case.

> **Note**
>
> This comparison represents a reproduction and numerical verification exercise.
>
> It is not a formal validation study because no experimental dataset or external benchmark solution was used.

---

## Courant Number and Mesh Refinement Observation

During a separate mesh refinement exercise, the mesh resolution was increased from:

```text
20 × 20 × 1
```

to:

```text
80 × 80 × 1
```

The original time-step size was initially retained.

This caused the maximum Courant number to increase significantly.

The solution became numerically unstable and eventually resulted in a floating-point exception.

The time-step size was reduced from:

```text
Δt = 0.005 s
```

to:

```text
Δt = 0.001 s
```

After reducing the time-step size, the simulation completed successfully.

This exercise demonstrated the relationship:

```text
Co ~ U Δt / Δx
```

and showed that spatial mesh refinement must also be considered together with temporal resolution in transient CFD simulations.

---

## Key Observations

- The moving lid transfers momentum into the cavity.
- A primary clockwise vortex develops inside the domain.
- The strongest velocity gradients occur near the moving upper wall.
- Reverse flow is present in the lower region of the cavity.
- The vertical centreline velocity profile contains positive and negative `Ux` values.
- Mesh refinement can significantly affect the Courant number.
- Time-step size must be reconsidered when spatial resolution is increased.
- OpenFOAM case verification can be performed using quantitative field comparisons rather than visual inspection alone.

---

## Tools and Software

| Tool | Usage |
|---|---|
| OpenFOAM 13 | CFD solver and finite-volume framework |
| Ubuntu 24.04 | Linux environment |
| WSL2 | Linux environment on Windows |
| ParaView | CFD post-processing |
| Git | Version control |
| GitHub | Project repository |

---

## Learning Outcomes

This project provided practical experience with:

- OpenFOAM case architecture
- finite-volume mesh generation
- velocity and pressure boundary conditions
- dimensional field definitions
- incompressible CFD modelling
- transient simulation setup
- finite-volume discretization schemes
- iterative linear solvers
- PIMPLE pressure-velocity coupling
- Courant number monitoring
- numerical instability diagnosis
- ParaView post-processing
- CFD verification workflows
- Git and GitHub project management

---

## Future Work

Possible extensions of this project include:

- mesh independence study,
- comparison with benchmark lid-driven cavity data,
- higher-order temporal discretization,
- alternative convection schemes,
- comparison between laminar and RANS models,
- `k-omega SST` simulations,
- automated centreline velocity extraction,
- Python-based CFD result comparison,
- residual monitoring,
- and quantitative vortex-centre tracking.

---

## Author

**Berksvml**

Aerospace Engineer

Interested in:

- Computational Fluid Dynamics
- Aerodynamics
- Finite Volume Methods
- OpenFOAM
- Numerical Methods
- Aerospace Engineering

---

## License

This project is licensed under the MIT License.

See the `LICENSE` file for details.
