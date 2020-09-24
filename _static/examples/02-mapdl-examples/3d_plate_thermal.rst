.. only:: html

    .. note::
        :class: sphx-glr-download-link-note

        Click :ref:`here <sphx_glr_download_examples_02-mapdl-examples_3d_plate_thermal.py>`     to download the full example code
    .. rst-class:: sphx-glr-example-title

    .. _sphx_glr_examples_02-mapdl-examples_3d_plate_thermal.py:


.. _ref_3d_plate_thermal:

Basic Thermal Analysis with pyansys
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This example demonstrates how you can use MAPDL to create a plate,
impose thermal boundary conditions, solve, and plot it all within
pyansys.

First, start MAPDL as a service and disable all but error messages.


.. code-block:: default


    # sphinx_gallery_thumbnail_number = 2
    import os
    import pyansys

    os.environ['I_MPI_SHM_LMT'] = 'shm'  # necessary on Ubuntu without "smp"
    mapdl = pyansys.launch_mapdl(loglevel='ERROR')








Geometry and Material Properties
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Create a simple beam, specify the material properties, and mesh it.


.. code-block:: default

    mapdl.prep7()
    mapdl.mp('kxx', 1, 45)
    mapdl.et(1, 90)
    mapdl.block(-0.3, 0.3, -0.46, 1.34, -0.2, -0.2 + 0.02)
    mapdl.vsweep(1)
    mapdl.eplot()





.. image:: /examples/02-mapdl-examples/images/sphx_glr_3d_plate_thermal_001.png
    :alt: 3d plate thermal
    :class: sphx-glr-single-img


.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none


    [(2.1163550212232822, 2.5563550212232817, 1.9263550212232823),
     (0.0, 0.44000000000000006, -0.19),
     (0.0, 0.0, 1.0)]



Boundary Conditions
~~~~~~~~~~~~~~~~~~~
Set the thermal boundary conditions


.. code-block:: default

    mapdl.asel('S', vmin=3)
    mapdl.nsla()
    mapdl.d('all', 'temp', 5)
    mapdl.asel('S', vmin=4)
    mapdl.nsla()
    mapdl.d('all', 'temp', 100)
    out = mapdl.allsel()









Solve
~~~~~
Solve the thermal static analysis and print the results


.. code-block:: default

    mapdl.vsweep(1)
    mapdl.run('/SOLU')
    print(mapdl.solve())
    out = mapdl.finish()






.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none

    *** NOTE ***                            CP =       0.752   TIME= 15:29:15
     The automatic domain decomposition logic has selected the MESH domain
     decomposition method with 2 processes per solution.

     *****  ANSYS SOLVE    COMMAND  *****

     *** NOTE ***                            CP =       0.753   TIME= 15:29:15
     There is no title defined for this analysis.

     *** ANSYS - ENGINEERING ANALYSIS SYSTEM  RELEASE 2020 R2          20.2     ***
     DISTRIBUTED ANSYS Mechanical Enterprise

     88888888  VERSION=LINUX x64     15:29:15  SEP 23, 2020 CP=      0.756





                           S O L U T I O N   O P T I O N S

       PROBLEM DIMENSIONALITY. . . . . . . . . . . . .3-D
       DEGREES OF FREEDOM. . . . . . TEMP
       ANALYSIS TYPE . . . . . . . . . . . . . . . . .STATIC (STEADY-STATE)
       GLOBALLY ASSEMBLED MATRIX . . . . . . . . . . .SYMMETRIC

     *** NOTE ***                            CP =       0.757   TIME= 15:29:15
     Present time 0 is less than or equal to the previous time.  Time will
     default to 1.

     *** NOTE ***                            CP =       0.757   TIME= 15:29:15
     The conditions for direct assembly have been met.  No .emat or .erot
     files will be produced.



         D I S T R I B U T E D   D O M A I N   D E C O M P O S E R

      ...Number of elements: 450
      ...Number of nodes:    2720
      ...Decompose to 2 CPU domains
      ...Element load balance ratio =     1.004


                          L O A D   S T E P   O P T I O N S

       LOAD STEP NUMBER. . . . . . . . . . . . . . . .     1
       TIME AT END OF THE LOAD STEP. . . . . . . . . .  1.0000
       NUMBER OF SUBSTEPS. . . . . . . . . . . . . . .     1
       STEP CHANGE BOUNDARY CONDITIONS . . . . . . . .    NO
       PRINT OUTPUT CONTROLS . . . . . . . . . . . . .NO PRINTOUT
       DATABASE OUTPUT CONTROLS. . . . . . . . . . . .ALL DATA WRITTEN
                                                      FOR THE LAST SUBSTEP


     SOLUTION MONITORING INFO IS WRITTEN TO FILE= file.mntr


     Range of element maximum matrix coefficients in global coordinates
     Maximum = 13.6474747 at element 449.
     Minimum = 13.6474747 at element 105.

       *** ELEMENT MATRIX FORMULATION TIMES
         TYPE    NUMBER   ENAME      TOTAL CP  AVE CP

            1       450  SOLID90       0.017   0.000038
     Time at end of element matrix formulation CP = 0.796117067.

     DISTRIBUTED SPARSE MATRIX DIRECT SOLVER.
      Number of equations =        2606,    Maximum wavefront =     72

      Local memory allocated for solver              =      2.747 MB
      Local memory required for in-core solution     =      2.645 MB
      Local memory required for out-of-core solution =      1.597 MB

      Total memory allocated for solver              =      5.124 MB
      Total memory required for in-core solution     =      4.935 MB
      Total memory required for out-of-core solution =      3.034 MB

     *** NOTE ***                            CP =       0.861   TIME= 15:29:15
     The Distributed Sparse Matrix Solver is currently running in the
     in-core memory mode.  This memory mode uses the most amount of memory
     in order to avoid using the hard drive as much as possible, which most
     often results in the fastest solution time.  This mode is recommended
     if enough physical memory is present to accommodate all of the solver
     data.
     Distributed sparse solver maximum pivot= 33.3096879 at node 1885 TEMP.
     Distributed sparse solver minimum pivot= 0.710694082 at node 2032 TEMP.
     Distributed sparse solver minimum pivot in absolute value= 0.710694082
     at node 2032 TEMP.




Post-Processing using MAPDL
~~~~~~~~~~~~~~~~~~~~~~~~~~~
View the thermal solution of the beam by getting the results
directly through MAPDL.


.. code-block:: default

    mapdl.post1()
    mapdl.set(1, 1)
    mapdl.post_processing.plot_nodal_temperature()





.. image:: /examples/02-mapdl-examples/images/sphx_glr_3d_plate_thermal_002.png
    :alt: 3d plate thermal
    :class: sphx-glr-single-img


.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none


    [(2.116355073728637, 2.5563550862456124, 1.9263550686622422),
     (0.0, 0.4400000125169754, -0.1900000050663948),
     (0.0, 0.0, 1.0)]



Alternatively you could also use the result object that reads in the
result file using pyansys


.. code-block:: default


    nnum, temp = mapdl.result.nodal_temperature(0)
    # this is the same as pyansys.read_binary(mapdl._result_file)

    print(mapdl.result.filename)
    print(temp)




.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none

    /tmp/ansys_cfmhthlpwk/file0.rth
    [11.41342066 17.70960818 24.03472642 ... 96.77753462 96.76893653
     96.73371864]





.. rst-class:: sphx-glr-timing

   **Total running time of the script:** ( 0 minutes  3.306 seconds)


.. _sphx_glr_download_examples_02-mapdl-examples_3d_plate_thermal.py:


.. only :: html

 .. container:: sphx-glr-footer
    :class: sphx-glr-footer-example



  .. container:: sphx-glr-download sphx-glr-download-python

     :download:`Download Python source code: 3d_plate_thermal.py <3d_plate_thermal.py>`



  .. container:: sphx-glr-download sphx-glr-download-jupyter

     :download:`Download Jupyter notebook: 3d_plate_thermal.ipynb <3d_plate_thermal.ipynb>`


.. only:: html

 .. rst-class:: sphx-glr-signature

    `Gallery generated by Sphinx-Gallery <https://sphinx-gallery.github.io>`_
