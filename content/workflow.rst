.. _workflow:

Workflow
========

.. questions::

   - What is 
   - What problem do 

.. objectives::

   - Know the basic settings and structure of OpenFOAM cases
   - Be able to create a simple test case

.. instructor-note::

   - 15 min teaching
   - 0 min exercises


Case structure
--------------

To setup a case, you need to have at least 3 directories in your case directory, namely **system**, **constant** and **<initial time directory>** (normally **0**).

.. code:: bash

 $ ls
 0 constant system


OpenFOAM cases are configured using plain text input files located across the three directories, in each input file, OpenFOAM uses a plain text dictionary format with keywords and values

**system**: contains input files for grid generators and solvers

    - ``controlDict``: the main simulation control parameters. This includes, e.g. timing information, write format, and optional libraries that can be loaded at run time
    - ``fvSchemes``: the selection of the numerical schemes
    - ``fvSolution``: the iterative solver and pressure-velocity coupling parameters
    - ``fvOptions``: user-specified finite volume options. Many OpenFOAM applications contain equation systems that can be manipulated at run time. These provide, e.g. additional source/sink terms, or enforce constraints.
    - ``blockMeshDict``: to control the block-structrued mesher blockMesh
    - ``snappyHexMeshDict``: to set the parameters for snappyHexMesh, another mesher shipped with OpenFOAM
    - ``decomposeParDict`` : to set the parameters of the domain decomposition used for running OpenFOAM in parallel
    - ...


**constant**: Contains values that are constant during simulation like transport properties of the fluid (viscosity models) and mesh coordinates

    - ``polyMesh``: where the computational mesh is stored
    - ``transportProperties``: the material properties of the fluid
    - ``turbulenceProperties``: the turbulence modelling 
    - ...

**<initial time directory>**: contains initial fields of the flow e.g. velocity, pressure etc. and boundary conditions


Additional directories can be generated, depending on user cases, most common ones include e.g.:

   - <result time directories>: field predictions as a function of iteration count or time
   - postProcessing: data typically generated by function objects
   - data conversion, e.g. VTK


A typical workflow for an OpenFOAM case is schematically shown below

.. code:: bash

 Start
   |
   v
 blockMesh // Create a block mesh (set by system/blockMeshDict)
   |
   v
 decomposePar // Divide into submeshes (set by system/decomposeParDict)
   |
   v
 snappyHexMesh // create complex mesh (set by system/snappyHexMeshDict)
   |
   V
 simpleFoam // run application(OpenFOAM solver) (set by system/controlDict)
   |
   V
 reconstructPar // Stitch together the solutions from  the submeshes



Of course, this can vary depending on what mesher you use, wether you run in parallel, etc. There may also be additional pre- or post-processing steps.



A few examples of the dictionaries are shown below:

.. tabs::

   .. tab:: controlDict

      .. code-block:: cpp

            /*--------------------------------*- C++ -*----------------------------------*\
            | =========                 |                                                 |
            | \\      /  F ield         | OpenFOAM: The Open Source CFD Toolbox           |
            |  \\    /   O peration     | Version:  v2306                                 |
            |   \\  /    A nd           | Website:  www.openfoam.com                      |
            |    \\/     M anipulation  |                                                 |
            \*---------------------------------------------------------------------------*/
            FoamFile
            {
                version     2.0;
                format      ascii;
                class       dictionary;
                object      controlDict;
            }
            // * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //
            
            application     icoFoam;
            
            startFrom       startTime;
            
            startTime       0;
            
            stopAt          endTime;
            
            endTime         0.5;
            
            deltaT          0.005;
            
            writeControl    timeStep;
            
            writeInterval   20;
            
            purgeWrite      0;
            
            writeFormat     ascii;
            
            writePrecision  6;
            
            writeCompression off;
            
            timeFormat      general;
            
            timePrecision   6;
            
            runTimeModifiable true;
            
            
            // ************************************************************************* //



   .. tab:: fvSchemes

      .. code-block:: cpp

            /*--------------------------------*- C++ -*----------------------------------*\
            | =========                 |                                                 |
            | \\      /  F ield         | OpenFOAM: The Open Source CFD Toolbox           |
            |  \\    /   O peration     | Version:  v2306                                 |
            |   \\  /    A nd           | Website:  www.openfoam.com                      |
            |    \\/     M anipulation  |                                                 |
            \*---------------------------------------------------------------------------*/
            FoamFile
            {
                version     2.0;
                format      ascii;
                class       dictionary;
                object      fvSchemes;
            }
            // * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //
            
            ddtSchemes
            {
                default         Euler;
            }
            
            gradSchemes
            {
                default         Gauss linear;
                grad(p)         Gauss linear;
            }
            
            divSchemes
            {
                default         none;
                div(phi,U)      Gauss linear;
            }
            
            laplacianSchemes
            {
                default         Gauss linear orthogonal;
            }
            
            interpolationSchemes
            {
                default         linear;
            }
            
            snGradSchemes
            {
                default         orthogonal;
            }
            
            
            // ************************************************************************* //


   .. tab:: fvSolution

      .. code-block:: cpp

            /*--------------------------------*- C++ -*----------------------------------*\
            | =========                 |                                                 |
            | \\      /  F ield         | OpenFOAM: The Open Source CFD Toolbox           |
            |  \\    /   O peration     | Version:  v2306                                 |
            |   \\  /    A nd           | Website:  www.openfoam.com                      |
            |    \\/     M anipulation  |                                                 |
            \*---------------------------------------------------------------------------*/
            FoamFile
            {
                version     2.0;
                format      ascii;
                class       dictionary;
                object      fvSolution;
            }
            // * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //
            
            solvers
            {
                p
                {
                    solver          PCG;
                    preconditioner  DIC;
                    tolerance       1e-06;
                    relTol          0.05;
                }
            
                pFinal
                {
                    $p;
                    relTol          0;
                }
            
                U
                {
                    solver          smoothSolver;
                    smoother        symGaussSeidel;
                    tolerance       1e-05;
                    relTol          0;
                }
            }
            
            PISO
            {
                nCorrectors     2;
                nNonOrthogonalCorrectors 0;
                pRefCell        0;
                pRefValue       0;
            }
            
            
            // ************************************************************************* //



Input types
-----------

Dictionaries
~~~~~~~~~~~~

OpenFOAM input dictionaries are designed to be human-readable ASCII text files, consisting of collections of keyword-value entries bounded by curly braces {}, e.g.

.. tabs::

   .. tab:: dictionary

      .. code-block:: cpp

            dictionary_name
            {
                labelType       1;
                scalarType      1.0;
                vectorType      (0 0 0);
                wordType        word;
                stringType      "string";
                ...
            }


The main basic entry types include:

.. list-table:: 
      :widths: 25 25 25 
      :header-rows: 1

      * - Type
        - Description
        - Example
      * - boolean
        - state
        - `on`, off, true, false
      * - label
        - integer
        - 123
      * - scalar
        - float
        - `123.456`
      * - word
        - a single word
        - value `value`
      * - string
        - quoted text
        - "this is a string value"
      * - list
        - a list of entries bounded by () braces
        - (0 1 2 3 4 5) 
      * - vector
        - a list of 3 values, nominally (x y z) components 
        - (0 0 0)
      * - sphericalTensor
        - a spherical tensor 
        - (0)
      * - symmTensor
        - a symmetric tensor defined by (xx xy xz yy yz zz)
        - (0 0 0 0 0 0)
      * - tensor
        - a nine component tensor defined by (xx xy xz yx yy yz zx zy zz)
        - `(0 0 0 0 0 0 0 0 0)`


Expressions
~~~~~~~~~~~

The Expressions functionality is a re-implementation of swak4Foam(SWiss Army Knife for Foam) created by Bernhard Gschaider and it was introduced since version v1912.
The Expressions syntax enables users to define custom expressions for use in a variety of scenarios that don’t exist yet in OpenFOAM, without the need to rely on coding in
C++, including:

    - pre-processing utilities
    - input dictionaries
    - boundary conditions
    - function objects (co-processing)
    - utilities, e.g. setting field values


Summary
-------

- ``fvOptions`` and ``functionObject`` practically remove the need for modifying the solver, as long as it captures your physics.
- Lots of ``fvOptions`` and ``functionObjects`` out there. Try and play with them!
- There is a coded type of ``fvOption`` and ``functionObject``, which allows you to simply write you own C++ to be executed! Will be compiled when the case runs, with no involvment from your side.
