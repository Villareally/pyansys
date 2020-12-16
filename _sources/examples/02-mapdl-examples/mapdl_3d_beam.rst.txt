.. only:: html

    .. note::
        :class: sphx-glr-download-link-note

        Click :ref:`here <sphx_glr_download_examples_02-mapdl-examples_mapdl_3d_beam.py>`     to download the full example code
    .. rst-class:: sphx-glr-example-title

    .. _sphx_glr_examples_02-mapdl-examples_mapdl_3d_beam.py:


.. _ref_3d_beam:

MAPDL 3D Beam Example
~~~~~~~~~~~~~~~~~~~~~

This is a simple example that loads an archive file containing a beam
and then runs a modal analysis using the simplified ``modal_analysis``
method.


.. code-block:: default


    from pyansys import examples
    import pyansys


    mapdl = pyansys.launch_mapdl(loglevel='ERROR')

    mapdl.cdread('db', examples.hexarchivefile)
    mapdl.esel('s', 'ELEM', vmin=5, vmax=20)
    mapdl.cm('ELEM_COMP', 'ELEM')
    mapdl.nsel('s', 'NODE', vmin=5, vmax=20)
    mapdl.cm('NODE_COMP', 'NODE')

    # boundary conditions
    mapdl.allsel()

    # dummy steel properties
    mapdl.prep7()
    mapdl.mp('EX', 1, 200E9)  # Elastic moduli in Pa (kg/(m*s**2))
    mapdl.mp('DENS', 1, 7800)  # Density in kg/m3
    mapdl.mp('NUXY', 1, 0.3)  # Poissons Ratio
    mapdl.emodif('ALL', 'MAT', 1)

    # fix one end of the beam
    mapdl.nsel('S', 'LOC', 'Z')
    mapdl.d('all', 'all')
    mapdl.allsel()

    mapdl.mxpand(elcalc='YES')
    mapdl.modal_analysis(nmode=6)






.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none


    '*** NOTE ***                            CP =       0.448   TIME= 23:49:31\n The automatic domain decomposition logic has selected the MESH domain\n decomposition method with 2 processes per solution.\n\n *****  ANSYS SOLVE    COMMAND  *****\n\n *** NOTE ***                            CP =       0.449   TIME= 23:49:31\n There is no title defined for this analysis.\n\n *** SELECTION OF ELEMENT TECHNOLOGIES FOR APPLICABLE ELEMENTS ***\n                ---GIVE SUGGESTIONS ONLY---\n\n ELEMENT TYPE    1 IS SOLID186. KEYOPT(2) IS ALREADY SET AS SUGGESTED.\n\n\n\n *** ANSYS - ENGINEERING ANALYSIS SYSTEM  RELEASE 2020 R2          20.2     ***\n DISTRIBUTED ANSYS Mechanical Enterprise\n\n 88888888  VERSION=LINUX x64     23:49:31  NOV 16, 2020 CP=      0.450\n\n\n\n\n\n                       S O L U T I O N   O P T I O N S\n\n   PROBLEM DIMENSIONALITY. . . . . . . . . . . . .3-D\n   DEGREES OF FREEDOM. . . . . . UX   UY   UZ\n   ANALYSIS TYPE . . . . . . . . . . . . . . . . .MODAL\n      EXTRACTION METHOD. . . . . . . . . . . . . .BLOCK LANCZOS\n   NUMBER OF MODES TO EXTRACT. . . . . . . . . . .     6\n   GLOBALLY ASSEMBLED MATRIX . . . . . . . . . . .SYMMETRIC\n   NUMBER OF MODES TO EXPAND . . . . . . . . . . .ALL\n   ELEMENT RESULTS CALCULATION . . . . . . . . . .ON\n\n *** NOTE ***                            CP =       0.451   TIME= 23:49:31\n The conditions for direct assembly have been met.  No .emat or .erot\n files will be produced.\n\n\n\n     D I S T R I B U T E D   D O M A I N   D E C O M P O S E R\n\n  ...Number of elements: 40\n  ...Number of nodes:    321\n  ...Decompose to 2 CPU domains\n  ...Element load balance ratio =     1.000\n\n\n                      L O A D   S T E P   O P T I O N S\n\n   LOAD STEP NUMBER. . . . . . . . . . . . . . . .     1\n   THERMAL STRAINS INCLUDED IN THE LOAD VECTOR . .   YES\n   PRINT OUTPUT CONTROLS . . . . . . . . . . . . .NO PRINTOUT\n   DATABASE OUTPUT CONTROLS. . . . . . . . . . . .ALL DATA WRITTEN\n\n\n\n                         ***********  PRECISE MASS SUMMARY  ***********\n\n   TOTAL RIGID BODY MASS MATRIX ABOUT ORIGIN\n               Translational mass               |   Coupled translational/rotational mass\n         39000.        0.0000        0.0000     |     0.0000        97500.       -19500.\n         0.0000        39000.        0.0000     |    -97500.        0.0000        19500.\n         0.0000        0.0000        39000.     |     19500.       -19500.        0.0000\n     ------------------------------------------ | ------------------------------------------\n                                                |         Rotational mass (inertia)\n                                                |    0.33800E+06   -9750.0       -48750.\n                                                |    -9750.0       0.33800E+06   -48750.\n                                                |    -48750.       -48750.        26000.\n\n   TOTAL MASS =  39000.\n     The mass principal axes coincide with the global Cartesian axes\n\n   CENTER OF MASS (X,Y,Z)=   0.50000       0.50000        2.5000\n\n   TOTAL INERTIA ABOUT CENTER OF MASS\n         84500.      -0.49072E-11  -0.29301E-10\n       -0.49072E-11    84500.      -0.37526E-10\n       -0.29301E-10  -0.37526E-10    6500.0\n     The inertia principal axes coincide with the global Cartesian axes\n\n\n  *** MASS SUMMARY BY ELEMENT TYPE ***\n\n  TYPE      MASS\n     1   39000.0\n\n Range of element maximum matrix coefficients in global coordinates\n Maximum = 9.116809117E+10 at element 32.\n Minimum = 9.116809117E+10 at element 4.\n\n   *** ELEMENT MATRIX FORMULATION TIMES\n     TYPE    NUMBER   ENAME      TOTAL CP  AVE CP\n\n        1        40  SOLID186      0.008   0.000209\n Time at end of element matrix formulation CP = 0.480036974.\n\n  BLOCK LANCZOS CALCULATION OF UP TO     6 EIGENVECTORS.\n  NUMBER OF EQUATIONS              =          900\n  MAXIMUM WAVEFRONT                =          225\n  MAXIMUM MODES STORED             =            6\n  MINIMUM EIGENVALUE               =  0.00000E+00\n  MAXIMUM EIGENVALUE               =  0.10000E+31\n\n\n  Local memory allocated for solver              =      1.564 MB\n  Local memory required for in-core solution     =      1.491 MB\n  Local memory required for out-of-core solution =      1.208 MB\n\n  Total memory allocated for solver              =      2.815 MB\n  Total memory required for in-core solution     =      2.685 MB\n  Total memory required for out-of-core solution =      2.122 MB\n\n *** NOTE ***                            CP =       0.609   TIME= 23:49:32\n The Distributed Sparse Matrix Solver used by the Block Lanczos\n eigensolver is currently running in the in-core memory mode.  This\n memory mode uses the most amount of memory in order to avoid using the\n hard drive as much as possible, which most often results in the\n fastest solution time.  This mode is recommended if enough physical\n memory is present to accommodate all of the solver data.'



View the results using the pyansys result object


.. code-block:: default

    result = mapdl.result
    print(result)






.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none

    PyANSYS MAPDL Result
    Units       : User Defined
    Version     : 20.2
    Cyclic      : False
    Result Sets : 6
    Nodes       : 321
    Elements    : 40


    Available Results:
    EMS : Miscellaneous summable items (normally includes face pressures)
    ENF : Nodal forces
    ENS : Nodal stresses
    ENG : Element energies and volume
    EEL : Nodal elastic strains
    ETH : Nodal thermal strains (includes swelling strains)
    EUL : Element euler angles
    EPT : Nodal temperatures
    NSL : Nodal displacements
    RF  : Nodal reaction forces





Access nodal displacement values


.. code-block:: default

    nnum, disp = result.nodal_displacement(0)

    # print the nodes 50 - 59
    for i in range(49, 59):
        print(nnum[i], disp[i])






.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none

    50 [-0.00102212  0.00216823  0.00130867]
    51 [-0.00123704  0.0026504   0.00140731]
    52 [-0.00146622  0.0031634   0.00149216]
    53 [-0.00170751  0.00370246  0.00156362]
    54 [-0.00195885  0.00426307  0.00162258]
    55 [-0.00221832  0.00484085  0.00166975]
    56 [-0.00248416  0.00543187  0.00170626]
    57 [-0.00275472  0.00603243  0.00173306]
    58 [-0.00302861  0.00663935  0.00175155]
    59 [-0.00330456  0.00724977  0.00176291]




Plot a modal result


.. code-block:: default

    result.plot_nodal_displacement(0, show_edges=True)





.. image:: /examples/02-mapdl-examples/images/sphx_glr_mapdl_3d_beam_001.png
    :alt: mapdl 3d beam
    :class: sphx-glr-single-img


.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none


    [(6.295554957734411, 6.295554957734411, 8.29555495773441),
     (0.5, 0.5, 2.5),
     (0.0, 0.0, 1.0)]



Animate a modal result
result.animate_nodal_solution(0, show_edges=True, loop=False, displacement_factor=10,


.. code-block:: default

                                  # movie_filename='demo.gif')









Cleanup
~~~~~~~
Close mapdl when complete


.. code-block:: default

    mapdl.exit()









.. rst-class:: sphx-glr-timing

   **Total running time of the script:** ( 0 minutes  2.995 seconds)


.. _sphx_glr_download_examples_02-mapdl-examples_mapdl_3d_beam.py:


.. only :: html

 .. container:: sphx-glr-footer
    :class: sphx-glr-footer-example



  .. container:: sphx-glr-download sphx-glr-download-python

     :download:`Download Python source code: mapdl_3d_beam.py <mapdl_3d_beam.py>`



  .. container:: sphx-glr-download sphx-glr-download-jupyter

     :download:`Download Jupyter notebook: mapdl_3d_beam.ipynb <mapdl_3d_beam.ipynb>`


.. only:: html

 .. rst-class:: sphx-glr-signature

    `Gallery generated by Sphinx-Gallery <https://sphinx-gallery.github.io>`_
