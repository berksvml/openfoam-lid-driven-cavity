# Lid-Driven Cavity Flow — OpenFOAM 13

A two-dimensional incompressible lid-driven cavity CFD case built from scratch using OpenFOAM 13.

This project was created to understand the structure of an OpenFOAM case, finite-volume discretization settings, pressure-velocity coupling, and basic CFD verification workflows.

## Problem Description

The problem consists of a square cavity with a moving upper wall.

The top wall moves in the positive x-direction and transfers momentum to the fluid, generating a primary clockwise recirculation region inside the cavity.

### Physical Parameters

| Parameter | Value |
|---|---|
| Cavity size | 0.1 m × 0.1 m |
| Lid velocity | 1 m/s |
| Kinematic viscosity | 1e-5 m²/s |
| Reynolds number | 10,000 |
| Flow model | Laminar |

The Reynolds number is calculated as:

```text
Re = U L / nu
```

which gives:

```text
Re = 10,000
```

## Case Structure

```text
.
├── 0
│   ├── U
│   └── p
├── constant
│   ├── momentumTransport
│   └── physicalProperties
└── system
    ├── blockMeshDict
    ├── controlDict
    ├── fvSchemes
    └── fvSolution
```

### `0/`

Contains the initial and boundary conditions for the velocity and pressure fields.

### `constant/`

Contains the physical properties of the fluid and the momentum transport model.

### `system/`

Contains the mesh definition, run control, discretization schemes, linear solver settings, and pressure-velocity coupling parameters.

## Mesh

The computational domain is generated using `blockMesh`.

The mesh consists of:

```text
20 × 20 × 1 cells
```

The front and back boundaries are defined as `empty`, resulting in a two-dimensional simulation.

## Boundary Conditions

### Moving Wall

```text
U = (1 0 0) m/s
```

The upper wall moves in the positive x-direction.

### Fixed Walls

```text
noSlip
```

The remaining cavity walls are stationary.

### Front and Back

```text
empty
```

These boundaries define the case as two-dimensional.

## Numerical Setup

### Solver

```text
incompressibleFluid
```

### Time Discretization

```text
Euler
```

First-order implicit time integration is used.

### Momentum Convection

```text
Gauss limitedLinearV 1
```

### Pressure Solver

```text
GAMG
```

### Velocity Solver

```text
smoothSolver
```

with a Gauss-Seidel smoother.

### Pressure-Velocity Coupling

The PIMPLE algorithm is used with:

```text
nCorrectors = 2
nNonOrthogonalCorrectors = 0
```

## Running the Case

Load the OpenFOAM environment:

```bash
source /opt/openfoam13/etc/bashrc
```

Generate the mesh:

```bash
blockMesh
```

Check the mesh:

```bash
checkMesh
```

Run the simulation:

```bash
foamRun
```

Open the results in ParaView:

```bash
paraFoam
```

## Verification

The case was independently constructed from scratch and compared against a reference laminar OpenFOAM cavity case.

The x-component of velocity, `Ux`, was extracted along the vertical cavity centreline:

```text
x = 0.05 m
```

using ParaView's `Plot Over Line` filter.

The resulting `Ux(y)` profiles were found to be nearly identical.

This comparison was used as a reproduction and verification check of the case setup.

> This is a verification exercise rather than a validation study. No experimental dataset or external benchmark solution was used.

## Key Observations

The moving lid generates a primary clockwise recirculation region.

Velocity gradients are strongest near the moving upper wall.

The vertical centreline velocity profile contains both positive and negative `Ux` values because of the recirculating flow.

A previous mesh refinement exercise also demonstrated the relationship between mesh resolution, time-step size, and the Courant number.

## Software

- OpenFOAM 13
- Ubuntu 24.04 on WSL2
- ParaView

## Author

Berks VML
