.. _numerics:

Numerics
========

.. questions::

   - What is 
   - What problem do 

.. objectives::

   - Explain 

.. instructor-note::

   - 15 min teaching
   - 0 min exercises

Numerical schemes
-----------------

OpenFOAM includes a wide range of solution and scheme controls, specified via dictionary files in the case **system** sub-directory. These are described by:

    - Numerical schemes: Transforming partial differential equations to a linear system of equations using the Finite Volume Method. The treatment of each term in the system of equations is specified in the ``fvSchemes`` dictionary. This enables fine-grain control of e.g. temporal, gradient, divergence and interpolation schemes. Additional run-time selectable physical modelling and general finite terms are prescribed in the ``fvOptions`` dictionary, targeting e.g. acoustics, heat transfer, momentum sources, multi-region coupling, linearised sources/sinks and many more
    - Solution methods: Case solution parameters are specified in the ``fvSolution`` dictionary. These include choice of linear equation solver per field variable, algorithm controls e.g. number of inner and outer iterations and under-relaxation.



OpenFOAM applications are designed for use with unstructured meshes, offering up
to second order accuracy, predominantly using collocated variable arrangements.
Most focus on the Finite Volume Method, for which the conservative form
of the general scalar transport equation for the transported quantity  :math:`\phi`  takes the
form:

.. math::
   \frac{\partial \rho \phi }{\partial t} +  \nabla \cdot \left(\rho \phi \mathbf{u} \right) =  \nabla \cdot \left(\Gamma_\phi  \nabla \phi \right) + S_\phi 


.. note:: 

    By setting different values for :math:`\phi`, :math:`\Gamma_\phi` and :math:`S_\phi`, one can obtain Navier-Stokes equations.
    For example, letting :math:`\phi = 1`, :math:`\Gamma_\phi = 0`, :math:`S_\phi = 0`, we will end up with the continuity equation



The Finite Volume Method requires the integration over a 3-D **control volume**,
such that:

.. math::
      \int_V \frac{\partial \rho \phi }{\partial t}  \mathrm{d} V
    + \int_V \nabla \cdot \left(\rho \phi \mathbf{u} \right) \mathrm{d} V
    = \int_V \nabla \cdot \left(\Gamma_\phi \nabla \phi \right) \mathrm{d} V
    + \int_V S_\phi \mathrm{d} V

Next step is to transform the volume integral to surface integral by using Gauss -Ostrogradsky Thereom.

.. math::
      \int_V \frac{\partial \rho \phi }{\partial t}  \mathrm{d} V
    + \oint_S \left(\rho \phi \mathbf{u \cdot n} \right) \mathrm{d} S  
    = \oint_S \left( \Gamma_\phi \nabla \phi \cdot \mathbf{n}\right)  \mathrm{d} S
    + \int_V S_\phi \mathrm{d} V


or 

.. math::
      \int_V \frac{\partial \rho \phi }{\partial t}  \mathrm{d} V
    + \sum_{F} \int_F \left(\rho \phi \mathbf{u \cdot n} \right) \mathrm{d} S  
    = \sum_{F} \int_F \left(\Gamma_\phi \nabla \phi \cdot \mathbf{n}\right)  \mathrm{d} S
    + \int_V S_\phi \mathrm{d} V



Up to this point, the integral form is valid for an arbitrary volume, and for each volume, the integral equations are valid.
One problem left is to interpolating the cell centred values (known quantities) to the face centres, and this interpolation is one of the source of errors.


.. callout:: The Gauss-Ostrogradsky Thereom

   Suppose :math:`V` is a volume in three-dimensional space, which is compact and has a piecewise smooth boundary :math:`S`. If :math:`\mathbf{F}` is a continuously differentiable vector field defined on a neighborhood of :math:`V`. The closed boundary :math:`S` is oriented by outward-pointing normals, and :math:`\mathbf{n}` is the outward pointing unit normal at each point on the boundary. 

   .. math::
         \iiint_V (\nabla \cdot \mathbf{F}) \mathrm{d} V = \oint_S (\mathbf{F} \cdot  \mathbf{n}) \mathrm{d} S 

   The Gauss-Ostrogradsky Theorem, also known as the Divergence Theorem, simply states that the outward flux of a vector field through a closed surface is equal to the volume integral of the divergence over the region inside the surface.



Introducing the interpolation, we obtain

.. math::
      \int_V \frac{\partial \rho \phi }{\partial t}  \mathrm{d} V
    + \sum_{F}  \left(\rho \phi \mathbf{u \cdot n} \right)_f  S_f  
    = \sum_{F}  \left(\Gamma_\phi \nabla \phi \cdot \mathbf{n}\right)_f  S_f
    + \int_V S_\phi \mathrm{d} V


During the spliting of the volume integrals across the volume faces/surfaces, note that 

 - The control volume is assumed to be a convex polyhedron, i.e. can be of any shape, only if it is convex and the faces made up the volume are planar
 - All values of all variables are computed and saved on the controid of the control volume, i.e. cell centred collocated arrangement
 - The value at each face centroid is approximated given the values at the centroids of the two cells sharing the face
 - To transform the quantity from the cell centre to the face centre, an interpolation scheme is required

.. math::
      \int_V \frac{\partial \rho \phi }{\partial t}  \mathrm{d} V
    + \sum_{F} \left(\rho_f \phi_f \mathbf{u}_f \cdot \mathbf{n}_f \right) S_f  
    = \sum_{F} ({\Gamma_\phi}_f  \nabla \phi_f \cdot \mathbf{n}_f)    S_f
    + \int_V S_\phi \mathrm{d} V


Interpolation schemes
---------------------

Interpolation schemes are specified in the ``fvSchemes`` file under the interpolationSchemes sub-dictionary using the syntax:

.. tabs::

   .. tab:: InterpolationSchemes

      .. code-block:: cpp

         interpolationSchemes
         {
             default         none;
             <equation term> <interpolation scheme>;
         }


A wide variety of interpolation schemes are available, ranging from those that are based solely on geometry, and others, e.g. convection schemes that are functions of the local flow:

   - Linear scheme: The most obvious option is linear interpolation, 2nd order accurate.  However, for convective fluxes it introduces oscillations
   - Convection scheme: Many options for interpolating the  convective flux exist. Often it is the most important numerical choice in the simulation. Many of the convection schemes available in OpenFOAM are based on the TVD and NVD: 

        - NVD/TVD convection schemes::
         
            - Limited linear divergence scheme
            - Linear divergence scheme
            - Linear-upwind divergence scheme
            - MUSCL divergence scheme
            - Mid-point divergence scheme
            - Minmod divergence scheme
            - QUICK divergence scheme
            - UMIST divergence scheme
            - Upwind divergence scheme
            - Van Leer divergence scheme
         
        - Non-NVD/TVD convection schemes::

            - Courant number blended divergence scheme
            - DES hybrid divergence scheme
            - Filtered Linear (2) divergence scheme
            - LUST divergence scheme



Temporal schemes
----------------

Now it is the time to choose a time integration scheme. Temporal schemes define how a field is integrated as a function of time. OpenFOAM includes a variety of schemes to integrate fields with respect to time. Time scheme properties are input in the ``fvSchemes`` file under the ``ddtSchemes`` sub-dictionary using the syntax:

.. tabs::

   .. tab:: Time scheme properties

      .. code-block:: cpp

         ddtSchemes
         {
             default         none;
             ddt(Q)          <time scheme>;
         }


Available **<time scheme>** include

    - Backward time scheme
    - Crank-Nicolson time scheme
    - Euler implicit time scheme
    - Local Euler implicit/explicit time scheme
    - Steady state time scheme


When choosing temporal scheme, here are a few things to consider:

 - Explicit or implicit: the latter means we have to solve a linear system at each time-step.
 - Order of accuracy
 - Numerical stability, and its implications for the time-step


Spatial schemes
---------------

At their core, spatial schemes rely heavily on interpolation schemes to transform cell-based quantities to cell faces, in combination with Gauss Theorem to convert volume integrals to surface integrals.

Gradient
++++++++

Gradient schemes are specified in the fvSchemes file under the gradSchemes sub-dictionary using the syntax:

.. tabs::

   .. tab:: gradSchemes

      .. code-block:: cpp

            gradSchemes
            {
                default         none;
                grad(p)         <optional limiter> <gradient scheme> <interpolation scheme>;
            }


Gradient schemes

   - Gauss gradient scheme
   - Least-squares gradient scheme

Interpolation schemes

   - linear: cell-based linear
   - pointLinear: point-based linear
   - leastSquares: Least squares

Gradient limiters

The limited gradient schemes attempt to preserve the monotonicity condition by limiting the gradient to ensure that the extrapolated face value is bounded by the neighbouring cell values.

   - Cell-limited gradient scheme
   - Face-limited gradient scheme
   - Multi-directional cell-limited gradient scheme
   - Multi-directional face-limited gradient scheme
   - clippedLinear: limits linear scheme according to a hypothetical cell size ratio


Divergence
++++++++++

Divergence schemes are specified in the fvSchemes file under the divSchemes sub-dictionary using the general syntax:

.. tabs::

   .. tab:: Time scheme properties

      .. code-block:: cpp

            divSchemes
            {
                default         none;
                div(Q)          Gauss <interpolation scheme>;
            }


A typical use is for convection schemes, which transport a property under the influence of a velocity field specified using:

.. tabs::

   .. tab:: divSchemes

      .. code-block:: cpp

            divSchemes
            {
                default         none;
                div(phi,Q)      Gauss <interpolation scheme>;
            }

The phi keyword is typically used to represent the flux (flow) across cell faces, i.e.
https://doc.openfoam.com/2312/tools/processing/numerics/schemes/divergence/
- volumetric flux:
- mass flux:


NVD/TVD convection schemes

Many of the convection schemes available in OpenFOAM are based on the TVD and NVD [PROVIDE REF] For further information, see the page invalid item schemes-divergence-nvdtvd

    Limited linear divergence scheme
    Linear divergence scheme
    Linear-upwind divergence scheme
    MUSCL divergence scheme
    Mid-point divergence scheme
    Minmod divergence scheme
    QUICK divergence scheme
    UMIST divergence scheme
    Upwind divergence scheme
    Van Leer divergence scheme

Non-NVD/TVD convection schemes

    Courant number blended divergence scheme
    DES hybrid divergence scheme
    Filtered Linear (2) divergence scheme
    LUST divergence scheme



Laplacian
+++++++++

Laplacian schemes are specified in the fvSchemes file under the laplacianSchemes sub-dictionary using the syntax:

.. tabs::

   .. tab:: laplacianSchemes

      .. code-block:: cpp

            laplacianSchemes
            {
                default         none;
                laplacian(gamma,phi) Gauss <interpolation scheme> <snGrad scheme>
            }

All options are based on the application of Gauss theorem, requiring an interpolation scheme to transform coefficients from cell values to the faces, and a surface-normal gradient scheme.


SnGrad
++++++

Surface-normal gradient schemes are specified in the fvSchemesfile under the snGradSchemes sub-dictionary using the syntax:

.. tabs::

   .. tab:: snGradSchemes

      .. code-block:: cpp
            
            snGradSchemes
            {
                default         none;
                snGrad(Q)       <snGrad scheme>;
            }

Options

    Corrected surface-normal gradient scheme
    Face-corrected surface-normal gradient scheme
    Limited surface-normal gradient scheme
    Orthogonal surface-normal gradient scheme
    Uncorrected surface-normal gradient scheme



Pressure-velocity coupling

    Introduction: Pressure-velocity algorithms
    Steady state: SIMPLE
    Transient: PISO
    Transient: PIMPLE

