Instructor's guide
------------------

```{contents} Table of contents
:depth: 2
```
### Learning outcomes

- Basic theory: touching upon numerical methods, CFD aspects
- Understand architecture of OpenFOAM as an engineering software
- Create and visualize meshes
- Set boundary conditions and forcing terms (such as turbulence models)
- Visualize results
- Compute statistics. Serial v/s parallel post-processing.
- Advanced: Define custom solvers

### Learner personas

#### T. Stark: CFD specialist 

- Runs turbulent **RANS simulations** of cars in virtual wind tunnels
- Responsible for getting a **good mesh** for simulating the flow
- They also have to choose appropriate **boundary conditions** and **turbulence models** such that the simulation is feasible.
- Once simulation is done, they have to distill simple statistics such as **skin friction drag** and **wake drag**

#### K. Khan: Visualization engineer

- Inspects **meshes** in Paraview
- Makes **3D visualization** of final results
- Builds **scripts** to automate visualization

#### Trinity: Researcher

- Expert in modelling new flow physics
- Builds **custom modules** to modify existing solvers

