.. _introduction:

OpenFOAM Introduction
=====================

.. questions::

   - What is OpenFOAM?
   - What problem does OpenFOAM solve? 

.. objectives::

   - Know the basics of OpenFOAM

.. instructor-note::

   - 15 min teaching
   - 0 min exercises


What is OpenFOAM ?
------------------

- OpenFOAM® (for “Open-source Field Operation And Manipulation”) is an open source Computation Fluid Dynamics (CFD) solve released and developed primarily by OpenCFD Ltd since 2004. 

- It has a large user base across most areas of engineering and science, from both commercial and academic organizations. 

- OpenFOAM has an extensive range of features to solve anything from complex fluid flows involving chemical reactions, turbulence and heat transfer, to acoustics, solid mechanics and electromagnetics. 


Different variants of OpenFOAM
------------------------------

There are two main variants of OpenFOAM:

    OpenCFD Ltd version (`www.openfoam.com <http://www.openfoam.com>`_):
        - New versions released twice a year (June-December)
        - Versions are named as (vYYMM) e.g. v1912
        - Website: https://www.openfoam.com

    OpenFOAM foundation version (`www.openfoam.org <http://www.openfoam.org>`_):
        - No defined date for new release 
        - Versions are named with with two digits like 4.1
        - Website: https://openfoam.org


One can read the history if interested:
https://www.openfoam.com/news/history


Which version to use?
---------------------

It depends on the features you want to use

    - Check the website and release notes to see which one fits better to your framework
    - If both include the features you need, do some performance and accuracy benchmarks to see which one is better
    - Otherwise it is just matter of taste!

We will be using the OpenCFD Ltd version with vXXXXXX for the training


OpenFOAM executables
--------------------

Unlike many other software, OpenFOAM does not have a unique executable. 
Once compiled, a large number of excutebles is generated and they fall into two categories: 

  - **solver**: that is designed to solve a specific continuum mechanics problem
  - **utility**: that is designed to perform tasks that involve data manipulation.

For every solver, mesh generation etc. there is a separate executable! 
You should run the right executable according to the solver you are using!
Check the documentation to see recommended solvers for different cases.

- ‘simpleFoam’: if you use SIMPLE algorithm
- ‘icoFoam’: if you use PISO algorithm for laminar flow
- ...


OpenFOAM vs commercial software
-------------------------------

Being an open source software, the advantages of OpenFOAM is quite obvious:

  - free to use
  - a strong focus on customization and flexibility
  - full control over the simulation

All these nice features come with a price: very deep learning curve. 
For example, contrary to the commercial CFD softwares, there are no default values in OpenFOAM. 
It is up to the user to set those values which means the user has to know what he/she is doing.
