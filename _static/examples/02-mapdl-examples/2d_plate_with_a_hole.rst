.. only:: html

    .. note::
        :class: sphx-glr-download-link-note

        Click :ref:`here <sphx_glr_download_examples_02-mapdl-examples_2d_plate_with_a_hole.py>`     to download the full example code
    .. rst-class:: sphx-glr-example-title

    .. _sphx_glr_examples_02-mapdl-examples_2d_plate_with_a_hole.py:


.. _ref_plane_stress_concentration:

ANSYS 2D Plane Stress Concentration Analysis
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This tutorial shows how you can use pyansys and MAPDL to determine and
verify the "stress concentration factor" when modeling using 2D plane
elements and then verify this using 3D elements.

First, start MAPDL as a service and disable all but error messages.


.. code-block:: default

    # sphinx_gallery_thumbnail_number = 3

    import matplotlib.pyplot as plt
    import numpy as np

    import pyansys

    # os.environ['I_MPI_SHM_LMT'] = 'shm'  # necessary on Ubuntu without "smp"
    mapdl = pyansys.launch_mapdl(override=True, additional_switches='-smp',
                                 loglevel='ERROR')








Element Type and Material Properties
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
This example will use PLANE183 elements as a thin plate can be
modeled with plane elements provided that KEYOPTION 3 is set to 3
and a thickness is provided.

This example will use SI units.


.. code-block:: default


    mapdl.prep7()
    mapdl.units('SI')  # SI - International system (m, kg, s, K).

    # define a PLANE183 element type with thickness
    mapdl.et(1, "PLANE183", kop3=3)
    mapdl.r(1, 0.001)  # thickness of 0.001 meters)

    # Define a material (nominal steel in SI)
    mapdl.mp('EX', 1, 210E9)  # Elastic moduli in Pa (kg/(m*s**2))
    mapdl.mp('DENS', 1, 7800)  # Density in kg/m3
    mapdl.mp('NUXY', 1, 0.3)  # Poisson's Ratio

    # list currently defined material properties
    print(mapdl.mplist())





.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none

    LIST MATERIALS        1 TO        1 BY        1
      PROPERTY= ALL

     MATERIAL NUMBER        1

          TEMP        EX
                   0.2100000E+12

          TEMP        NUXY
                   0.3000000

          TEMP        DENS
                    7800.000




Geometry
~~~~~~~~
Create a rectangular area with the hole in the middle.  To correctly
approximate an infinite plate, the maximum stress must occur far
away from the edges of the plate.  A length to width factor can
approximate this.


.. code-block:: default


    length = 0.4
    width = 0.1

    ratio = 0.3  # diameter/width
    diameter = width*ratio
    radius = diameter*0.5


    # create the rectangle
    rect_anum = mapdl.blc4(width=length, height=width)

    # create a circle in the middle of the rectangle
    circ_anum = mapdl.cyl4(length/2, width/2, radius)

    # Note how pyansys parses the output and returns the area numbers
    # created by each command.  This can be used to execute a boolean
    # operation on these areas to cut the circle out of the rectangle.
    plate_with_hole_anum = mapdl.asba(rect_anum, circ_anum)

    # finally, plot the lines of the plate
    _ = mapdl.lplot(vtk=True, cpos='xy', line_width=10, font_size=26,
                    color_lines=True, background='w')




.. image:: /examples/02-mapdl-examples/images/sphx_glr_2d_plate_with_a_hole_001.png
    :alt: 2d plate with a hole
    :class: sphx-glr-single-img





Meshing
~~~~~~~
Mesh the plate using a higher density near the hole and a lower
density for the remainder of the plate by setting ``LESIZE`` for the
lines nearby the hole and ``ESIZE`` for the mesh global size.

Line numbers can be identified through inspection using ``lplot``


.. code-block:: default


    # ensure there are at 50 elements around the hole
    hole_esize = np.pi*diameter/50  # 0.0002
    plate_esize = 0.01

    # increased the density of the mesh at the center
    mapdl.lsel('S', 'LINE', vmin=5, vmax=8)
    mapdl.lesize('ALL', hole_esize, kforc=1)
    mapdl.lsel('ALL')

    # Decrease the area mesh expansion.  This ensures that the mesh
    # remains fine nearby the hole
    mapdl.mopt('EXPND', 0.7)  # default 1

    mapdl.esize(plate_esize)
    mapdl.amesh(plate_with_hole_anum)
    _ = mapdl.eplot(vtk=True, cpos='xy', show_edges=True, show_axes=False,
                    line_width=2, background='w')




.. image:: /examples/02-mapdl-examples/images/sphx_glr_2d_plate_with_a_hole_002.png
    :alt: 2d plate with a hole
    :class: sphx-glr-single-img





Boundary Conditions
~~~~~~~~~~~~~~~~~~~
Fix the left-hand side of the plate in the X direction and set a
force of 1 kN in the positive X direction.



.. code-block:: default


    # Fix the left-hand side.
    mapdl.nsel('S', 'LOC', 'X', 0)
    mapdl.d('ALL', 'UX')

    # Fix a single node on the left-hand side of the plate in the Y
    # direction.  Otherwise, the mesh would be allowed to move in the y
    # direction and would be an improperly constrained mesh.
    mapdl.nsel('R', 'LOC', 'Y', width/2)
    assert mapdl.mesh.n_node == 1
    mapdl.d('ALL', 'UY')

    # Apply a force on the right-hand side of the plate.  For this
    # example, we select the nodes at the right-most side of the plate.
    mapdl.nsel('S', 'LOC', 'X', length)

    # Verify that only the nodes at length have been selected:
    assert np.unique(mapdl.mesh.nodes[:, 0]) == length

    # Next, couple the DOF for these nodes.  This lets us provide a force
    # to one node that will be spread throughout all nodes in this coupled
    # set.
    mapdl.cp(5, 'UX', 'ALL')

    # Select a single node in this set and apply a force to it
    # We use "R" to re-select from the current node group
    mapdl.nsel('R', 'LOC', 'Y', width/2)
    mapdl.f('ALL', 'FX', 1000)

    # finally, be sure to select all nodes again to solve the entire solution
    _ = mapdl.allsel()









Solve the Static Problem
~~~~~~~~~~~~~~~~~~~~~~~~
Solve the static analysis


.. code-block:: default

    mapdl.run('/SOLU')
    mapdl.antype('STATIC')
    mapdl.solve()





.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none


    'One or more COMPONENTS exist that do not have all underlying entities selected.  Issuing an ALLSEL or other select commands before CDWRITE will ensure all underlying entities are selected.  These COMPONENTS were not written to the CDWRITE file.\n *****  ANSYS SOLVE    COMMAND  *****\n\n *** NOTE ***                            CP =       1.054   TIME= 21:45:55\n There is no title defined for this analysis.\n\n *** SELECTION OF ELEMENT TECHNOLOGIES FOR APPLICABLE ELEMENTS ***\n                ---GIVE SUGGESTIONS ONLY---\n\n ELEMENT TYPE    1 IS PLANE183 WITH PLANE STRESS OPTION. NO SUGGESTION IS\n AVAILABLE.\n\n\n\n *** ANSYS - ENGINEERING ANALYSIS SYSTEM  RELEASE 2020 R2          20.2     ***\n ANSYS Mechanical Enterprise\n 88888888  VERSION=LINUX x64     21:45:55  SEP 28, 2020 CP=      1.059\n\n\n\n\n\n                       S O L U T I O N   O P T I O N S\n\n   PROBLEM DIMENSIONALITY. . . . . . . . . . . . .2-D\n   DEGREES OF FREEDOM. . . . . . UX   UY\n   ANALYSIS TYPE . . . . . . . . . . . . . . . . .STATIC (STEADY-STATE)\n   GLOBALLY ASSEMBLED MATRIX . . . . . . . . . . .SYMMETRIC\n\n *** NOTE ***                            CP =       1.061   TIME= 21:45:55\n Present time 0 is less than or equal to the previous time.  Time will\n default to 1.\n\n *** NOTE ***                            CP =       1.062   TIME= 21:45:55\n The conditions for direct assembly have been met.  No .emat or .erot\n files will be produced.\n\n                      L O A D   S T E P   O P T I O N S\n\n   LOAD STEP NUMBER. . . . . . . . . . . . . . . .     1\n   TIME AT END OF THE LOAD STEP. . . . . . . . . .  1.0000\n   NUMBER OF SUBSTEPS. . . . . . . . . . . . . . .     1\n   STEP CHANGE BOUNDARY CONDITIONS . . . . . . . .    NO\n   PRINT OUTPUT CONTROLS . . . . . . . . . . . . .NO PRINTOUT\n   DATABASE OUTPUT CONTROLS. . . . . . . . . . . .ALL DATA WRITTEN\n                                                  FOR THE LAST SUBSTEP\n\n\n SOLUTION MONITORING INFO IS WRITTEN TO FILE= file.mntr\n\n\n\n\n            **** CENTER OF MASS, MASS, AND MASS MOMENTS OF INERTIA ****\n\n  CALCULATIONS ASSUME ELEMENT MASS AT ELEMENT CENTROID\n\n  TOTAL MASS =  0.30649\n\n                           MOM. OF INERTIA         MOM. OF INERTIA\n  CENTER OF MASS            ABOUT ORIGIN        ABOUT CENTER OF MASS\n\n  XC =  0.20000          IXX =   0.1024E-02      IXX =   0.2576E-03\n  YC =  0.49997E-01      IYY =   0.1642E-01      IYY =   0.4156E-02\n  ZC =   0.0000          IZZ =   0.1744E-01      IZZ =   0.4414E-02\n                         IXY =  -0.3065E-02      IXY =   0.8905E-09\n                         IYZ =    0.000          IYZ =    0.000\n                         IZX =    0.000          IZX =    0.000\n\n\n  *** MASS SUMMARY BY ELEMENT TYPE ***\n\n  TYPE      MASS\n     1  0.306487\n\n Range of element maximum matrix coefficients in global coordinates\n Maximum = 1.265116826E+09 at element 67.\n Minimum = 359465553 at element 773.\n\n   *** ELEMENT MATRIX FORMULATION TIMES\n     TYPE    NUMBER   ENAME      TOTAL CP  AVE CP\n\n        1       977  PLANE183      0.060   0.000062\n Time at end of element matrix formulation CP = 1.12946904.\n\n SPARSE MATRIX DIRECT SOLVER.\n  Number of equations =        6124,    Maximum wavefront =     48\n  Memory allocated for solver              =     8.424 MB\n  Memory required for in-core solution     =     8.123 MB\n  Memory required for out-of-core solution =     4.307 MB\n\n *** NOTE ***                            CP =       1.190   TIME= 21:45:55\n The Sparse Matrix Solver is currently running in the in-core memory\n mode.  This memory mode uses the most amount of memory in order to\n avoid using the hard drive as much as possible, which most often\n results in the fastest solution time.  This mode is recommended if\n enough physical memory is present to accommodate all of the solver\n data.\n Sparse solver maximum pivot= 1.958386732E+09 at node 1937 UY.\n Sparse solver minimum pivot= 3839644.71 at node 960 UY.\n Sparse solver minimum pivot in absolute value= 3839644.71 at node 960\n UY.'



Post-Processing
~~~~~~~~~~~~~~~
The static result can be post-processed both within MAPDL and
outside of MAPDL using ``pyansys``.  This example shows how to
extract the von Mises stress and plot it using the ``pyansys``
result reader.


.. code-block:: default


    # grab the result from the ``mapdl`` instance
    result = mapdl.result
    result.plot_principal_nodal_stress(0, 'SEQV', lighting=False,
                                       cpos='xy', background='w',
                                       text_color='k', add_text=False)

    nnum, stress = result.principal_nodal_stress(0)
    von_mises = stress[:, -1]  # von-Mises stress is the right most column

    # Must use nanmax as stress is not computed at mid-side nodes
    max_stress = np.nanmax(von_mises)




.. image:: /examples/02-mapdl-examples/images/sphx_glr_2d_plate_with_a_hole_003.png
    :alt: 2d plate with a hole
    :class: sphx-glr-single-img





Compute the Stress Concentration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
The stress concentration :math:`K_t` is the ratio of the maximum
stress at the hole to the far-field stress, or the mean cross
sectional stress at a point far from the hole.  Analytically, this
can be computed with:

:math:`\sigma_{nom} = \frac{F}{wt}`

Where

- :math:`F` is the force
- :math:`w` is the width of the plate
- :math:`t` is the thickness of the plate.

Experimentally, this is computed by taking the mean of the nodes at
the right-most side of the plate.


.. code-block:: default


    # We use nanmean here because mid-side nodes have no stress
    mask = result.mesh.nodes[:, 0] == length
    far_field_stress = np.nanmean(von_mises[mask])
    print('Far field von mises stress: %e' % far_field_stress)
    # Which almost exactly equals the analytical value of 10000000.0 Pa





.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none

    Far field von mises stress: 9.999966e+06




Since the expected nominal stress across the cross section of the
hole will increase as the size of the hole increases, regardless of
the stress concentration, the stress must be adjusted to arrive at
the correct stress.  This stress is adjusted by the ratio of the
width over the modified cross section width.


.. code-block:: default

    adj = width/(width - diameter)
    stress_adj = far_field_stress*adj

    # The stress concentration is then simply the maximum stress divided
    # by the adjusted far-field stress.
    stress_con = (max_stress/stress_adj)
    print('Stress Concentration: %.2f' % stress_con)






.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none

    Stress Concentration: 2.34




Batch Analysis
~~~~~~~~~~~~~~
The above script can be placed within a function to compute the
stress concentration for a variety of hole diameters.  For each
batch, MAPDL is reset and the geometry is generated from scratch.


.. code-block:: default


    def compute_stress_con(ratio):
        """Compute the stress concentration for plate with a hole loaded
        with a uniaxial force.
        """
        mapdl.clear('NOSTART')
        mapdl.prep7()
        mapdl.units('SI')  # SI - International system (m, kg, s, K).

        # define a PLANE183 element type with thickness
        mapdl.et(1, "PLANE183", kop3=3)
        mapdl.r(1, 0.001)  # thickness of 0.001 meters)

        # Define a material (nominal steel in SI)
        mapdl.mp('EX', 1, 210E9)  # Elastic moduli in Pa (kg/(m*s**2))
        mapdl.mp('DENS', 1, 7800)  # Density in kg/m3
        mapdl.mp('NUXY', 1, 0.3)  # Poisson's Ratio
        mapdl.emodif('ALL', 'MAT', 1)

        # Geometry
        # ~~~~~~~~
        # Create a rectangular area with the hole in the middle
        diameter = width*ratio
        radius = diameter*0.5

        # create the rectangle
        rect_anum = mapdl.blc4(width=length, height=width)

        # create a circle in the middle of the rectangle
        circ_anum = mapdl.cyl4(length/2, width/2, radius)

        # Note how pyansys parses the output and returns the area numbers
        # created by each command.  This can be used to execute a boolean
        # operation on these areas to cut the circle out of the rectangle.
        plate_with_hole_anum = mapdl.asba(rect_anum, circ_anum)

        # Meshing
        # ~~~~~~~
        # Mesh the plate using a higher density near the hole and a lower
        # density for the remainder of the plate

        mapdl.aclear('all')

        # ensure there are at least 100 elements around the hole
        hole_esize = np.pi*diameter/100  # 0.0002
        plate_esize = 0.01

        # increased the density of the mesh at the center
        mapdl.lsel('S', 'LINE', vmin=5, vmax=8)
        mapdl.lesize('ALL', hole_esize, kforc=1)
        mapdl.lsel('ALL')

        # Decrease the area mesh expansion.  This ensures that the mesh
        # remains fine nearby the hole
        mapdl.mopt('EXPND', 0.7)  # default 1

        mapdl.esize(plate_esize)
        mapdl.amesh(plate_with_hole_anum)

        ###############################################################################
        # Boundary Conditions
        # ~~~~~~~~~~~~~~~~~~~
        # Fix the left-hand side of the plate in the X direction
        mapdl.nsel('S', 'LOC', 'X', 0)
        mapdl.d('ALL', 'UX')

        # Fix a single node on the left-hand side of the plate in the Y direction
        mapdl.nsel('R', 'LOC', 'Y', width/2)
        assert mapdl.mesh.n_node == 1
        mapdl.d('ALL', 'UY')

        # Apply a force on the right-hand side of the plate.  For this
        # example, we select the right-hand side of the plate.
        mapdl.nsel('S', 'LOC', 'X', length)

        # Next, couple the DOF for these nodes
        mapdl.cp(5, 'UX', 'ALL')

        # Again, select a single node in this set and apply a force to it
        mapdl.nsel('r', 'loc', 'y', width/2)
        mapdl.f('ALL', 'FX', 1000)

        # finally, be sure to select all nodes again to solve the entire solution
        mapdl.allsel()

        # Solve the Static Problem
        # ~~~~~~~~~~~~~~~~~~~~~~~~
        mapdl.run('/SOLU')
        mapdl.antype('STATIC')
        mapdl.solve()


        # Post-Processing
        # ~~~~~~~~~~~~~~~
        # grab the stress from the result
        result = mapdl.result
        nnum, stress = result.principal_nodal_stress(0)
        von_mises = stress[:, -1]
        max_stress = np.nanmax(von_mises)

        # compare to the "far field" stress by getting the mean value of the
        # stress at the wall
        mask = result.mesh.nodes[:, 0] == length
        far_field_stress = np.nanmean(von_mises[mask])

        # adjust by the cross sectional area at the hole
        adj = width/(width - diameter)
        stress_adj = far_field_stress*adj

        # finally, compute the stress concentration
        return max_stress/stress_adj









Run the batch and record the stress concentration


.. code-block:: default

    k_t_exp = []
    ratios = np.linspace(0.001, 0.5, 20)
    print('    Ratio  : Stress Concentration (K_t)')
    for ratio in ratios:
        stress_con = compute_stress_con(ratio)
        print('%10.4f : %10.4f' % (ratio, stress_con))
        k_t_exp.append(stress_con)






.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none

        Ratio  : Stress Concentration (K_t)
        0.0010 :     2.9957
        0.0273 :     2.9116
        0.0535 :     2.8363
        0.0798 :     2.7701
        0.1061 :     2.7081
        0.1323 :     2.6506
        0.1586 :     2.5967
        0.1848 :     2.5420
        0.2111 :     2.4939
        0.2374 :     2.4496
        0.2636 :     2.4040
        0.2899 :     2.3683
        0.3162 :     2.3339
        0.3424 :     2.2998
        0.3687 :     2.2732
        0.3949 :     2.2457
        0.4212 :     2.2224
        0.4475 :     2.1963
        0.4737 :     2.1775
        0.5000 :     2.1609




Analytical Comparison
~~~~~~~~~~~~~~~~~~~~~
Stress concentrations are often obtained by referencing tablular
results or polynominal fits for a variety of geometries.  According
to Peterson's Stress Concentration Factors (ISBN 0470048247), the analytical
equation for a hole in a thin plate in uniaxial tension:

:math:`k_t = 3 - 3.14\frac{d}{h} + 3.667\left(\frac{d}{h}\right)^2 - 1.527\left(\frac{d}{h}\right)^3`

Where:

- :math:`k_t` is the stress concentration
- :math:`d` is the diameter of the circle
- :math:`h` is the height of the plate

As shown in the following plot, ANSYS matches the known tabular
result for this geometry remarkably well using PLANE183 elements.
The fit to the results may vary depending on the ratio between the
height and width of the plate.


.. code-block:: default


    # where ratio is (d/h)
    k_t_anl = 3 - 3.14*ratios + 3.667*ratios**2 - 1.527*ratios**3

    plt.plot(ratios, k_t_anl, label=r'$K_t$ Analytical')
    plt.plot(ratios, k_t_exp, label=r'$K_t$ ANSYS')
    plt.legend()
    plt.xlabel('Ratio of Hole Diameter to Width of Plate')
    plt.ylabel('Stress Concentration')
    plt.show()



.. image:: /examples/02-mapdl-examples/images/sphx_glr_2d_plate_with_a_hole_004.png
    :alt: 2d plate with a hole
    :class: sphx-glr-single-img






.. rst-class:: sphx-glr-timing

   **Total running time of the script:** ( 0 minutes  22.687 seconds)


.. _sphx_glr_download_examples_02-mapdl-examples_2d_plate_with_a_hole.py:


.. only :: html

 .. container:: sphx-glr-footer
    :class: sphx-glr-footer-example



  .. container:: sphx-glr-download sphx-glr-download-python

     :download:`Download Python source code: 2d_plate_with_a_hole.py <2d_plate_with_a_hole.py>`



  .. container:: sphx-glr-download sphx-glr-download-jupyter

     :download:`Download Jupyter notebook: 2d_plate_with_a_hole.ipynb <2d_plate_with_a_hole.ipynb>`


.. only:: html

 .. rst-class:: sphx-glr-signature

    `Gallery generated by Sphinx-Gallery <https://sphinx-gallery.github.io>`_
