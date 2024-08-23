.. _mesh:

Mesh
====

.. questions::

   - What is a mesher?

.. objectives::

   - Understand the basics of blockMesh and snappyHexMesh
   - Know how to use blockMesh and snappyHexMesh

.. instructor-note::

   - 15 min teaching
   - 0 min exercises



Mesh generation
---------------

There are a couple of mesher available:

- blockMesh – Block-structured hexahedral mesher
- snappyHexMesh – Unstructured hexa-dominated mesher
- cfMesh – Unstructured mesher with different available meshing strategies
- makeFaMesh - Create finite-area meshes from volume-mesh patches
- Other commercial mesh generation

blockMesh
+++++++++

blockMesh is a structured hexahedral mesh generator:

- Key features:

   - structured hex mesh
   - built using blocks
   - supports cell size grading
   - supports curved block edges

- Constraints:

   - requires consistent block-to-block connectivity
   - ordering of points is important


It is well suited to simple geometries that can be described by a few blocks, but challenging to apply to cases with a large number of blocks due to book-keeping requirements, i.e. the need to manage point connectivity and ordering.

The utility is controlled using a ``blockMeshDict`` dictionary, located in the case **system** directory. 
It manually define everything: vertices, blocks, curved edges, boundaries and is split into the following sections:

   - points
   - edges
   - blocks
   - patches

An example of the ``blockMeshDict`` dictionary is provided below:


Boundary types are listed here:

 - "patch": generic type used for most boundary boundaries
 - "wall": for walls
 - "empty": for a 2D case, defining the side boundaries parallel to the 2D plane in which the solution is obtained
 - "cyclic", "cyclicAMI": for periodic boundary conditions. Come in pairs
 - "symmetry": symmetry boundary

Boundary types vs boundary conditions:

 - Boundary conditions for fields are set in the **0** folder!
 - But some boundary types essentially define the condition, e.g. cyclic.
 - In this case, the conditions in the **0** folder must match the boundary type, e.g. cyclic boundary condition for the cyclic boundary type. Same with "empty"


snappyHexMesh
+++++++++++++

snappyHexMesh is a fully parallel, split hex, mesh generator that guarantees a minimum mesh quality. Controlled using OpenFOAM dictionaries, it is particularly well suited to batch driven operation.

Some of the key features are listed here::

   - starts from any pure hex mesh (structured or unstructured)
   - reads geometry in triangulated formats, e.g. in stl, obj, vtk
   - no limit on the number of input surfaces
   - can use simple analytically-defined geometry, e.g. box, sphere, cone
   - generates prismatic layers
   - scales well when meshing in parallel
   - can work with dirty surfaces, i.e. non-watertight surfaces

Meshing controls are set in the ``snappyHexMeshDict`` located in the case **system** directory, which contains the following sections:

    - Geometry: specification of the input surfaces
    - Castellation: starting from any pure hex mesh, refine and optionally load balance when running in parallel. The refinement is specified both according to surfaces, volumes and gaps
    - Snapping: guaranteed mesh quality whilst morphing to geometric surfaces and features
    - Layers: prismatic layers are inserted by shrinking an existing mesh and creating an infill, subject to the same mesh quality constraints
    - Mesh quality: mesh quality settings enforced during the snapping and layer addition phases
    - Global setting

The overall meshing process is summarised by the figure below:

.. figure:: img/snappyHexMesh-overview-small.png
   :align: center

   Figure source: OpenFOAM documentation `Meshing process <https://doc.openfoam.com/2312/tools/pre-processing/mesh/generation/snappyhexmesh/#meshing-process/>`__.




This includes:

   - Put the stl of the geometry to **constant/triSruface**
   - Create a ``blockMeshDict`` with one block of cubic cells. This will define the largest cell size.
   - Create the background mesh using the ``blockMesh`` utility (or any other hexahedral mesh generator)
   - Create a ``surfaceFeatureExtractDict`` in **system** and extract the features on the surfaces with ``surfaceFeatureExtract`` utility
   - Create the background mesh using the ``blockMesh`` utility (or any other hexahedral mesh generator)
   - Setting up the ``snappyHexMeshDict`` input dictionary
   - Running ``snappyHexMesh`` in serial or parallel


Note:

- Running ``snappyHexMesh`` will produce a separate directory for each step of the meshing process. The mesh in **constant** folder will be intact.
- Running ``snappyHexMesh –overwrite`` to write only the final mesh directly to **constant** folder


Mesh manipulation
-----------------

The following tools are useful when manipulating the mesh, e.g. scaling the geometry, identifying patches and creating sets and zones for physical models and post-processing::

   - surfaceTransformPoints
   - topoSet


Mesh conversion
---------------

Quite a few tools exist for mesh conversion::

    - ccmToFoam
    - fireToFoam
    - fluentMeshToFoam, fluent3DMeshToFoam
    - gmshToFoam
    - ansysfoam
  

Summary
-------

 - OpenFOAM has several meshing tools, suitable for both simple and complex geometries
 - It’s possible to do a lot with snappyHexMesh, including industrial flows
 - It requires a lot of parameter tweeking and one has to know the tool well
 - Generally, speciallized commercial meshers are still a bit better
