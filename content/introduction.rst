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

- OpenFOAM® (for “Open-source Field Operation And Manipulation”) is an open source general solver of Partial differential equations (PDEs) via the finite volume method. It is particularly popular in the computational fluid dynamics (CFD) community. 

- It has a large user base across most areas of engineering and science, from both commercial and academic organizations. 

- OpenFOAM has an extensive range of features to solve anything from complex fluid flows involving chemical reactions, turbulence and heat transfer, to acoustics, solid mechanics and electromagnetics. 


Different variants of OpenFOAM
------------------------------

There are two main variants of OpenFOAM:

    OpenCFD Ltd aka ESI version (`www.openfoam.com <http://www.openfoam.com>`_):
        - New versions released twice a year (June-December)
        - Versions are named as (vYYMM) e.g. v1912
        - Website: https://www.openfoam.com

    OpenFOAM foundation version (`www.openfoam.org <http://www.openfoam.org>`_):
        - No defined date for new release 
        - Semantic versioning is used. The most recent version is 12.
        - Website: https://openfoam.org


One can read the history if interested:
https://www.openfoam.com/news/history.

Moreover, another fork exists called `foam-extend`, a community-driven effort currently at version 5.0. This project, while being on a generally slower development
cycle, is more open to contributions from other actors and provides some experimental features not found in the other flavours, such as
turbomachinery toolboxes, immersed boundary method and tooling for reduced order models such as POD.


Which version to use?
---------------------

It depends on the features you want to use:

    - Check the website and release notes to see which one fits better to your framework;
    - If both include the features you need, do some performance and accuracy benchmarks to see which one is better;
    - Otherwise it is just matter of taste!

Some features are only available in one flavour (e.g. overset meshes only implemented in ESI), while others have been introduced in one and
then ported to the other. The foundation version is more prone to breaking changes (see e.g. the modular solvers below), whereas the ESI version
tends to require minor modifications across subsequent versions. foam-extend offers some features that neither of the other two have; while it
generally strives to have some degree of compatibility with the foundation version, some porting effort is probably required.


OpenFOAM executables
--------------------

Historically, unlike many other software, OpenFOAM does not have a unique executable. 
Once compiled, a large number of executables is generated, which fall into two categories: 

  - **solver**: that is designed to solve a specific continuum mechanics problem
  - **utility**: that is designed to perform tasks that involve data manipulation, mesh generation/conversion, etc.

This is the case for the ESI version of OpenFOAM and used to be for OpenFOAM foundation until version 10. 
From OpenFOAM 11 onwards, a radical shift of paradigm has taken place: so-called `modular <https://cfd.direct/openfoam/free-software/modular-solvers/>`__
solvers have been introduced. This approach consists in each solver being a *class* (in the OOP sense) and having one single executable (``foamRun``)
which launches the solver. This makes it easier to include multiple physics in the solution strategy, as well as adding additional equations to be solved. 
While this makes for much cleaner and maintainable code, it also means that a wealth of older tutorial cases/learning materials (such as blog and forum posts)
cannot immediately be reused on OpenFOAM :math:`\geq` 11. 

OpenFOAM vs commercial software
-------------------------------

OpenFOAM is open-source and it has a degree of focus on customisability and flexibility. The possibility of having full control on the simulation
and the fact that it is free make it very appealing for research use, but it is also widely used in industry (e.g. some automakers and F1 teams, pharma, chemical engineering...).
The costs lie in its steeper learning curve, as users need to have some solid theoretical understanding of CFD, and the absence of bells and whistles that come with
commercial software like friendly GUIs (even though some `commercial <https://engys.com/helyx/>`__ and `free <https://github.com/cfddose/Splash/tree/main>`__ efforts are underway).
