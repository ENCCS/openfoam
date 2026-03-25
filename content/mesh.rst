.. _mesh:

Meshing
=======

.. questions::

   - What is a mesher?
   - What are the choices we have for native OpenFOAM meshing?
   - How do we visualise the generated mesh(es)?
   - Can we import meshes from other CFD packages into OpenFOAM?
   - What are the tools we can use to evaluate mesh metrics and quality?
   

.. objectives::

   - Understand the basics of blockMesh and snappyHexMesh
   - Know how to use blockMesh and snappyHexMesh
   - Learn to inspect a mesh in Paraview
   - Learn to use checkMesh to summarise statistics and metrics of mesh quality
  

.. instructor-note::

   - 15 min teaching
   - 0 min exercises



Mesh generation
---------------

A few meshers that generate OF-native meshes are available:

- blockMesh – Block-structured hexahedral mesher
- snappyHexMesh – Unstructured hexa-dominated mesher
- cfMesh – Unstructured mesher with different available meshing strategies. Works on recent OF versions with some tweeking
- Pointwise - commercial (and paid), has native OF export capabilities 

Moreover, a plethora of conversion tools exist to import meshes from both free and commercial meshers, as outlined :ref:`here<Mesh conversion>`.



blockMesh
+++++++++

blockMesh is a structured hexahedral mesh generator, meaning that it can generate several blocks with a structured topology:

- Key features:

   - structured hex mesh
   - built using blocks
   - supports cell size grading
   - supports curved block edges

- Constraints:

   - requires consistent block-to-block connectivity
   - ordering of points is important


It is well suited to simple geometries that can be described by a few blocks, but challenging to apply to cases with a large number of blocks due to book-keeping requirements, i.e. 
the need to manage point connectivity and ordering.

The utility is controlled using a ``blockMeshDict`` dictionary, located in the case **system** directory. 
The user has to manually define all topological entities (points, edges, blocks, etc.) and is split into the following sections:

   - points
   - edges
   - blocks
   - patches

An example of ``blockMeshDict`` dictionary for a 2D cavity is provided below:

.. code-block:: cpp

   /*--------------------------------*- C++ -*----------------------------------*\
    =========                 |
    \\      /  F ield         | OpenFOAM: The Open Source CFD Toolbox
    \\    /   O peration     | Website:  https://openfoam.org
        \\  /    A nd           | Version:  12
        \\/     M anipulation  |
    \*---------------------------------------------------------------------------*/
    FoamFile
    {
        format      ascii;
        class       dictionary;
        object      blockMeshDict;
    }
    // * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //

    convertToMeters 0.1;

    vertices
    (
        (0 0 0)
        (1 0 0)
        (1 1 0)
        (0 1 0)
        (0 0 0.1)
        (1 0 0.1)
        (1 1 0.1)
        (0 1 0.1)
    );

    blocks
    (
        hex (0 1 2 3 4 5 6 7) (20 20 1) simpleGrading (1 1 1)
    );

    boundary
    (
        movingWall
        {
            type wall;
            faces
            (
                (3 7 6 2)
            );
        }
        fixedWalls
        {
            type wall;
            faces
            (
                (0 4 7 3)
                (2 6 5 1)
                (1 5 4 0)
            );
        }
        frontAndBack
        {
            type empty;
            faces
            (
                (0 3 2 1)
                (4 5 6 7)
            );
        }
    );


    // ************************************************************************* //



The types a boundary can have are listed here:

 - `patch`: generic type used for most boundary boundaries;
 - `wall`: for walls;
 - `empty`: for a 2D case, defining the side boundaries parallel to the 2D plane
   in which the solution is obtained;
 - `cyclic`, "cyclicAMI": for periodic boundary conditions. Come in pairs;
 - `symmetry`: symmetry boundary;

 A distinction has to be made between boundary types and boundary conditions:

 - Boundary conditions for fields are set in the **0** folder!
 - But some boundary types essentially define the condition, e.g. cyclic.
 - In this case, the conditions in the **0** folder must match the boundary type, 
   e.g. cyclic boundary condition for the cyclic boundary type. Same with "empty"


snappyHexMesh
+++++++++++++

snappyHexMesh is a fully parallel, split hex mesh generator that guarantees a
minimum (user-defined) mesh quality. Controlled using OpenFOAM dictionaries, it
is particularly well suited to batch driven operation.

Some of the key features are listed here:

- starts from any pure hex mesh (structured or unstructured), e.g. as generated
  by ``blockMesh``
- reads geometry in triangulated formats, e.g. in stl, obj, vtk
- no limit on the number of input surfaces
- can use simple analytically-defined geometry, e.g. box, sphere, cone
- generates prismatic layers
- scales well when meshing in parallel
- can work with dirty surfaces, i.e. non-watertight surfaces


The overall meshing process is summarised by the figure below:

.. figure:: img/snappyHexMesh-overview-small.png
   :align: center

   Figure source: OpenFOAM documentation `Meshing process <https://doc.openfoam.com/2312/tools/pre-processing/mesh/generation/snappyhexmesh/#meshing-process/>`__.

The process starts with a "background" mesh (usually generated with ``blockMesh``), which is then 
*castellated* (i.e. the cells are split close to the body). Then the *snapping* steps takes place,
during which the cells are "clipped" to fit the body. Further steps like growth of boundary 
layers and other refinements can follow.

Meshing controls are set in the ``snappyHexMeshDict`` located in the case
**system** directory, which contains the following sections:

- Geometry: specification of the input surfaces
- Castellation: starting from any pure hex mesh, refine and optionally load balance when 
  running in parallel. The refinement is specified both according to surfaces, volumes and gaps
- Snapping: guaranteed mesh quality whilst morphing to geometric surfaces and features
- Layers: prismatic layers are inserted by shrinking an existing mesh and creating an infill, 
  subject to the same mesh quality constraints
- Mesh quality: mesh quality settings enforced during the snapping and layer addition phases
- Global settings

This includes:

   - Put the stl of the geometry to **constant/triSruface**
   - Create a ``blockMeshDict`` with one block of cubic cells. This will define the largest cell size.
   - Create the background mesh using the ``blockMesh`` utility (or any other hexahedral mesh generator)
   - Create a ``surfaceFeatureExtractDict`` in **system** and extract the features on the surfaces with ``surfaceFeatureExtract`` utility
   - Create the background mesh using the ``blockMesh`` utility (or any other hexahedral mesh generator)
   - Setting up the ``snappyHexMeshDict`` input dictionary
   - Running ``snappyHexMesh`` in serial or parallel


Note:

- Running ``snappyHexMesh`` will produce a separate directory for each step of the meshing process. The 
  mesh in **constant** folder will be intact.
- Running ``snappyHexMesh –overwrite`` to write only the final mesh directly to **constant** folder


Mesh visualisation
------------------

The computational mesh can be inspected with Paraview. Starting from an OF case folder, a file 
(historically named ``r.foam``) has to be created (this will also be needed to visualise
simulation results). After that, Paraview can be opened and you can click on File->Open and
select the newly created ``r.foam``. Once the case is loaded, be sure to select 
"Surface with edges" as visualisation type. An example visualisation for a 2D cavity
is shown below.




.. figure:: img/cavity2D_mesh.png
   :align: center


.. _Mesh conversion:

Mesh conversion
---------------

  
Several meshers exist, both commercial and free/open source, which can be used indirectly
with OpenFOAM by importing the produced meshes with some conversion tool.

- ccm26ToFoam - can be used to import Star-CCM+ meshes. The original mesh cannot contain
  interfaces;
- fluentMeshToFoam - can be used to import meshes from ANSYS Fluent;
- gmshToFoam - can be used to import meshes from the open source tool Gmsh;
- ideasToFoam - can be used to convert meshes generated by I-DEAS and Salome;
- gambitToFoam - can be used to import meshes generated by GAMBIT;
- starToFoam - can be used to import meshes generated by STAR-CD meshers, such
  as PROSTAR.

Summary
-------

 - OpenFOAM has several meshing tools, suitable for both simple and complex geometries;
 - It’s possible to do a lot with snappyHexMesh, including industrial flows;
 - It requires a lot of parameter tweaking and one has to know the tool well;
 - Generally, specialised commercial meshers are still a bit better.
