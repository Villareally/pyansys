.. only:: html

    .. note::
        :class: sphx-glr-download-link-note

        Click :ref:`here <sphx_glr_download_examples_02-mapdl-examples_transient_thermal.py>`     to download the full example code
    .. rst-class:: sphx-glr-example-title

    .. _sphx_glr_examples_02-mapdl-examples_transient_thermal.py:


.. _ref_thermal_transient:

Example Thermal Transient Analysis
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
This example shows how you can use `pyansys` to input a time dependent
temperature table to vary the temperature at a beam.  This uses
convection loads with independently varying convection coefficient and
bulk temperature.

Example adapted from:
https://www.simutechgroup.com/tips-and-tricks/fea-articles/97-fea-tips-tricks-thermal-transient

Thanks SimuTech!


.. code-block:: default

    # sphinx_gallery_thumbnail_number = 4

    import os
    import numpy as np

    import matplotlib.pyplot as plt
    import pyansys

    os.environ['I_MPI_SHM_LMT'] = 'shm'  # necessary on Ubuntu
    mapdl = pyansys.launch_mapdl()

    mapdl.clear()
    mapdl.prep7()

    # Material properties-- 1020 steel in imperial
    mapdl.units('BIN')  # U.S. Customary system using inches (in, lbf*s2/in, s, °F).
    mapdl.mp('EX', 1, 30023280.0)
    mapdl.mp('NUXY', 1, 0.290000000)
    mapdl.mp('ALPX', 1, 8.388888889E-06)
    mapdl.mp('DENS', 1, 7.346344000E-04)
    mapdl.mp('KXX', 1, 6.252196000E-04)
    mapdl.mp('C', 1, 38.6334760)

    # use a thermal element type
    mapdl.et(1, 'SOLID70')






.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none

    2020-09-23 20:37:10,376 [INFO] pyansys.mapdl: CLEAR ANSYS DATABASE AND RESTART

     ANSYS Mechanical Enterprise
    2020-09-23 20:37:10,377 [INFO] pyansys.mapdl: *** ANSYS - ENGINEERING ANALYSIS SYSTEM  RELEASE 2020 R2          20.2     ***
     DISTRIBUTED ANSYS Mechanical Enterprise

     88888888  VERSION=LINUX x64     20:37:10  SEP 23, 2020 CP=      0.386





              ***** ANSYS ANALYSIS DEFINITION (PREP7) *****
    2020-09-23 20:37:10,379 [INFO] pyansys.mapdl: U.S. CUSTOMARY INCH UNITS SPECIFIED FOR INTERNAL
      LENGTH      = INCHES (IN)
      MASS        = LBF-S**2/IN
      TIME        = SECONDS (SEC)
      TEMPERATURE = FAHRENHEIT
      TOFFSET     = 460.0
      FORCE       = LBF
      HEAT        = IN-LBF
      PRESSURE    = PSI (LBF/IN**2)
      ENERGY      = IN-LBF
      POWER       = IN-LBF/SEC

     INPUT  UNITS ARE ALSO SET TO BIN
    2020-09-23 20:37:10,381 [INFO] pyansys.mapdl: MATERIAL          1     EX   =  0.3002328E+08
    2020-09-23 20:37:10,382 [INFO] pyansys.mapdl: MATERIAL          1     NUXY =  0.2900000
    2020-09-23 20:37:10,382 [INFO] pyansys.mapdl: MATERIAL          1     ALPX =  0.8388889E-05
    2020-09-23 20:37:10,383 [INFO] pyansys.mapdl: MATERIAL          1     DENS =  0.7346344E-03
    2020-09-23 20:37:10,384 [INFO] pyansys.mapdl: MATERIAL          1     KXX  =  0.6252196E-03
    2020-09-23 20:37:10,385 [INFO] pyansys.mapdl: MATERIAL          1     C    =   38.63348
    2020-09-23 20:37:10,386 [INFO] pyansys.mapdl: ELEMENT TYPE       1 IS SOLID70      3-D THERMAL SOLID
      KEYOPT( 1- 6)=        0      0      0        0      0      0
      KEYOPT( 7-12)=        0      0      0        0      0      0
      KEYOPT(13-18)=        0      0      0        0      0      0

     CURRENT NODAL DOF SET IS  TEMP
      THREE-DIMENSIONAL MODEL

    1



Geometry and Mesh
~~~~~~~~~~~~~~~~~
Create a block 5x1x1 inches in size and mesh it


.. code-block:: default

    mapdl.block(0, 5, 0, 1, 0, 1)
    mapdl.lesize('ALL', 0.2, layer1=1)

    mapdl.mshape(0, '3D')
    mapdl.mshkey(1)
    mapdl.vmesh(1)
    mapdl.eplot()





.. image:: /examples/02-mapdl-examples/images/sphx_glr_transient_thermal_001.png
    :alt: transient thermal
    :class: sphx-glr-single-img


.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none

    2020-09-23 20:37:10,393 [INFO] pyansys.mapdl: CREATE A HEXAHEDRAL VOLUME WITH
     X-DISTANCES FROM      0.000000000     TO      5.000000000
     Y-DISTANCES FROM      0.000000000     TO      1.000000000
     Z-DISTANCES FROM      0.000000000     TO      1.000000000

          OUTPUT VOLUME =     1
    2020-09-23 20:37:10,395 [INFO] pyansys.mapdl: SET DIVISIONS ON ALL SELECTED LINES
          FOR ELEMENT SIZE =  0.20000        SPACING RATIO =     1.0000
          (KFORCE = 0      PREVIOUS NONZERO VALUES WILL NOT BE ALTERED)
     SET LAYER-MESH CONTROLS ON ALL SELECTED LINES
          TO  LAYER1 =      1.0000   ABSOLUTE LENGTH UNITS
    2020-09-23 20:37:10,395 [INFO] pyansys.mapdl: PRODUCE ALL HEXAHEDRAL ELEMENTS IN 3D.
    2020-09-23 20:37:10,396 [INFO] pyansys.mapdl: USE THE MAPPED MESHER.
    2020-09-23 20:37:10,406 [INFO] pyansys.mapdl: GENERATE NODES AND ELEMENTS
           IN VOLUMES       1  TO      1  IN STEPS OF      1

     NUMBER OF VOLUMES MESHED   =         1
     MAXIMUM NODE NUMBER        =       936
     MAXIMUM ELEMENT NUMBER     =       625

    [(8.29555495773441, 6.295554957734411, 6.295554957734411),
     (2.5, 0.5, 0.5),
     (0.0, 0.0, 1.0)]



Setup the Solution
~~~~~~~~~~~~~~~~~~
Solve a transient analysis while ramping the load up and down.

Note the solution time commands in the above code fragment. The
final TIME is set to 1000 seconds. Time substep size is permitted to
range from a minimum of 2 seconds to a maximum of 50 seconds in the
DELTIM command. A first substep of 10 seconds is applied. Automatic
time substep sizing will vary substeps between the extremes.

A Table Array is used for the time-dependent Convection Coefficient
values. Times go in the Zeroth column, while associated Convection
Coefficients go in the First column.


.. code-block:: default


    mapdl.run('/SOLU')
    mapdl.antype(4)            # transient analysis
    mapdl.trnopt('FULL')       # full transient analysis
    mapdl.kbc(0)               # ramp loads up and down

    # Time stepping
    end_time = 1500
    mapdl.time(end_time)       # end time for load step
    mapdl.autots('ON')         # use automatic time stepping


    # setup where the subset time is 10 seconds, time
    mapdl.deltim(10, 2, 25)    # substep size (seconds)
    #                          -- minimum value shorter than smallest
    #                            time change in the table arrays below

    # Create a table of convection times and coefficients and trasfer it to MAPDL
    my_conv = np.array([[0, 0.001],      # start time
                        [120, 0.001],    # end of first "flat" zone
                        [130, 0.005],    # ramps up in 10 seconds
                        [700, 0.005],    # end of second "flat zone
                        [710, 0.002],    # ramps down in 10 seconds
                        [end_time, 0.002]])  # end of third "flat" zone
    mapdl.load_table('my_conv', my_conv, 'TIME')


    # Create a table of bulk temperatures for a given time and transfer to MAPDL
    my_bulk = np.array([[0, 100],      # start time
                        [120, 100],    # end of first "flat" zone
                        [500, 300],    # ramps up in 380 seconds
                        [700, 300],    # hold temperature for 200 seconds
                        [900, 75],     # temperature ramps down for 200 seconds
                        [end_time, 75]])   # end of second "flat" zone
    mapdl.load_table('my_bulk', my_bulk, 'TIME')






.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none

    2020-09-23 20:37:10,922 [INFO] pyansys.mapdl: ***** ROUTINE COMPLETED *****  CP =         0.608



     *****  ANSYS SOLUTION ROUTINE  *****
    2020-09-23 20:37:10,924 [INFO] pyansys.mapdl: PERFORM A TRANSIENT ANALYSIS
      THIS WILL BE A NEW ANALYSIS
    2020-09-23 20:37:10,924 [INFO] pyansys.mapdl: PERFORM A FULL TRANSIENT ANALYSIS
      USING NEWMARK ALGORITHM
    2020-09-23 20:37:10,925 [INFO] pyansys.mapdl: STEP BOUNDARY CONDITION KEY= 0
    2020-09-23 20:37:10,926 [INFO] pyansys.mapdl: TIME=  1500.0
    2020-09-23 20:37:10,927 [INFO] pyansys.mapdl: USE AUTOMATIC TIME STEPPING THIS LOAD STEP
    2020-09-23 20:37:10,928 [INFO] pyansys.mapdl: USE INITIAL TIME STEP SIZE OF  10.00000     FOR ALL  DEGREES OF FREEDOM
     FOR AUTOMATIC TIME STEPPING:
       USE  2.000000     AS THE MINIMUM TIME STEP SIZE
       USE  25.00000     AS THE MAXIMUM TIME STEP SIZE
    2020-09-23 20:37:10,929 [INFO] pyansys.mapdl: SET PARAMETER DIMENSIONS ON  MY_CONV TYPE=TABL  DIMENSIONS=     6     1     1
      TIME
    2020-09-23 20:37:10,930 [INFO] pyansys.mapdl: TABLE READ OPERATION  *TREAD
     TABLE ARRAY PARAMETER MY_CONV  READ FROM FILE /tmp/nvbktczbxs.txt
    2020-09-23 20:37:10,931 [INFO] pyansys.mapdl: SET PARAMETER DIMENSIONS ON  MY_BULK TYPE=TABL  DIMENSIONS=     6     1     1
      TIME
    2020-09-23 20:37:10,932 [INFO] pyansys.mapdl: TABLE READ OPERATION  *TREAD
     TABLE ARRAY PARAMETER MY_BULK  READ FROM FILE /tmp/cucafcgbtr.txt




The Transient Thermal Solve
~~~~~~~~~~~~~~~~~~~~~~~~~~~
This model is to be solved in one time step. For this reason, a
``TSRES`` command is used for force the solver to include a
``SOLVE`` at every time point in the two Table Arrays above. This
ensures that the time-dependent curves are followed by the transient
analysis. Intermediate solutions between the ``TSRES`` time points
will be included according to the ``DELTIM`` command and the
automatic time stepping decisions of the ANSYS solver.

In this example, the times for the ``TSRES`` array illustrated above
have been determined manually. A set of APDL commands could be used
to automate this process for chosen Table Array entries, in more
complex modeling situations, including checks that no time intervals
are too short.

Results at substeps will be wanted if the intermediate solutions of
the time-transient analysis are to be available for post-processing
review. The ``OUTRES`` command is used to control how much is written to
the results file. In this example the OUTRES command will be used to
simply write out all results for all substeps. In work with large
models and may substeps, too much data will be written if such a
strategy is employed for ``OUTRES``, and other options will need to be
considered. Note that one option for the ``OUTRES`` command is to
control times at which results are written with a Table Array, much
as is used in the ``TSRES`` command, but typically for a larger number
of time points, although including those of the TSRES array.

The initial condition starting temperature is controlled for this
example with the ``TUNIF`` command. Note that thermal transient
analyses can also have a starting temperature profile formed by a
static thermal ``SOLVE``. If a user neglects to set an initial
temperature in ANSYS Mechanical APDL, a value of zero will be used,
which is often not what is desired.

The thermal convective loads are applied with an SF family
command—in this example a convective load is applied to the end face
of the solid model by the SFA command, using the Table Array entries
for convection and bulk temperature that were developed above. The
Table Array names are surrounded with percent signs (%).  A SOLVE is
then performed.


.. code-block:: default


    # Force transient solve to include the times within the conv and bulk arrays
    # my_tres = np.unique(np.vstack((my_bulk[:, 0], my_conv[:, 0])))[0]  # same as
    mapdl.parameters['my_tsres'] = [120, 130, 500, 700, 710, 900, end_time]
    mapdl.tsres('%my_tsres%')

    mapdl.outres('ERASE')
    mapdl.outres('ALL', 'ALL')

    mapdl.eqslv('SPARSE')  # use sparse solver
    mapdl.tunif(75)        # force uniform starting temperature (otherwise zero)

    # apply the convective load (convection coefficient plus bulk temperature)
    # use "%" around table array names
    mapdl.sfa(6, 1, 'CONV', '%my_conv%', ' %my_bulk%')

    # solve
    mapdl.solve()





.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none

    2020-09-23 20:37:10,938 [INFO] pyansys.mapdl: RESET TIME STEPS AT TIME POINTS GIVEN BY ARRAY :MY_TSRES
    2020-09-23 20:37:10,940 [INFO] pyansys.mapdl: ERASE THE CURRENT DATABASE OUTPUT CONTROL TABLE.
    2020-09-23 20:37:10,941 [INFO] pyansys.mapdl: WRITE ALL  ITEMS TO THE DATABASE WITH A FREQUENCY OF ALL
       FOR ALL APPLICABLE ENTITIES
    2020-09-23 20:37:10,942 [INFO] pyansys.mapdl: USE SPARSE MATRIX DIRECT SOLVER
    2020-09-23 20:37:10,943 [INFO] pyansys.mapdl: UNIFORM TEMPERATURE=    75.000  (TREF=     0.000)
    2020-09-23 20:37:10,944 [INFO] pyansys.mapdl: SURFACE LOAD ON AREA      6
         LOAD KEY =1         LOAD LABEL = CONV
         VALUES = MY_CONV MY_BULK
    2020-09-23 20:37:14,054 [INFO] pyansys.mapdl: One or more COMPONENTS exist that do not have all underlying entities selected.  Issuing an ALLSEL or other select commands before CDWRITE will ensure all underlying entities are selected.  These COMPONENTS were not written to the CDWRITE file.
     *** NOTE ***                            CP =       0.622   TIME= 20:37:10
     The automatic domain decomposition logic has selected the MESH domain
     decomposition method with 2 processes per solution.

     *****  ANSYS SOLVE    COMMAND  *****

     TRANSFER SOLID MODEL BOUNDARY CONDITIONS TO FINITE ELEMENT MODEL
          SURFACE LOADS  TRANSFERRED FROM AREAS         =     25

     *** NOTE ***                            CP =       0.624   TIME= 20:37:10
     There is no title defined for this analysis.

     *** ANSYS - ENGINEERING ANALYSIS SYSTEM  RELEASE 2020 R2          20.2     ***
     DISTRIBUTED ANSYS Mechanical Enterprise

     88888888  VERSION=LINUX x64     20:37:10  SEP 23, 2020 CP=      0.625





                           S O L U T I O N   O P T I O N S

       PROBLEM DIMENSIONALITY. . . . . . . . . . . . .3-D
       DEGREES OF FREEDOM. . . . . . TEMP
       ANALYSIS TYPE . . . . . . . . . . . . . . . . .TRANSIENT
          SOLUTION METHOD. . . . . . . . . . . . . . .FULL
       EQUATION SOLVER OPTION. . . . . . . . . . . . .SPARSE
       GLOBALLY ASSEMBLED MATRIX . . . . . . . . . . .SYMMETRIC

     *** NOTE ***                            CP =       0.626   TIME= 20:37:10
     The conditions for direct assembly have been met.  No .emat or .erot
     files will be produced.



         D I S T R I B U T E D   D O M A I N   D E C O M P O S E R

      ...Number of elements: 625
      ...Number of nodes:    936
      ...Decompose to 2 CPU domains
      ...Element load balance ratio =     1.003


                          L O A D   S T E P   O P T I O N S

       LOAD STEP NUMBER. . . . . . . . . . . . . . . .     1
       TIME AT END OF THE LOAD STEP. . . . . . . . . .  1500.0
       AUTOMATIC TIME STEPPING . . . . . . . . . . . .    ON
          STARTING TIME STEP SIZE. . . . . . . . . . .  10.000
          MINIMUM TIME STEP SIZE . . . . . . . . . . .  2.0000
          MAXIMUM TIME STEP SIZE . . . . . . . . . . .  25.000
       TIME STEP RESET ARRAY . . . . . . . . . . . . .  MY_TSRES
       STEP CHANGE BOUNDARY CONDITIONS . . . . . . . .    NO
       TRANSIENT (INERTIA) EFFECTS
          THERMAL DOFS . . . . . . . . . . . . . . . .    ON
       TRANSIENT INTEGRATION PARAMETERS
          THETA. . . . . . . . . . . . . . . . . . . .  1.0000
          OSCILLATION LIMIT CRITERION. . . . . . . . . 0.50000
          TOLERANCE. . . . . . . . . . . . . . . . . .  0.0000
       PRINT OUTPUT CONTROLS . . . . . . . . . . . . .NO PRINTOUT
       DATABASE OUTPUT CONTROLS
          ITEM     FREQUENCY   COMPONENT
           ALL        ALL


     SOLUTION MONITORING INFO IS WRITTEN TO FILE= file.mntr




                **** CENTER OF MASS, MASS, AND MASS MOMENTS OF INERTIA ****

      CALCULATIONS ASSUME ELEMENT MASS AT ELEMENT CENTROID

      TOTAL MASS =  0.36732E-02

                               MOM. OF INERTIA         MOM. OF INERTIA
      CENTER OF MASS            ABOUT ORIGIN        ABOUT CENTER OF MASS

      XC =   2.5000          IXX =   0.2424E-02      IXX =   0.5877E-03
      YC =  0.50000          IYY =   0.3181E-01      IYY =   0.7934E-02
      ZC =  0.50000          IZZ =   0.3181E-01      IZZ =   0.7934E-02
                             IXY =  -0.4591E-02      IXY =   0.1626E-16
                             IYZ =  -0.9183E-03      IYZ =   0.3732E-17
                             IZX =  -0.4591E-02      IZX =   0.1789E-16


      *** MASS SUMMARY BY ELEMENT TYPE ***

      TYPE      MASS
         1  0.367317E-02

     Range of element maximum matrix coefficients in global coordinates
     Maximum = 5.168130667E-05 at element 375.
     Minimum = 4.168130667E-05 at element 226.

       *** ELEMENT MATRIX FORMULATION TIMES
         TYPE    NUMBER   ENAME      TOTAL CP  AVE CP

            1       625  SOLID70       0.009   0.000014
     Time at end of element matrix formulation CP = 0.658279002.

     ALL CURRENT ANSYS DATA WRITTEN TO FILE NAME= file.rdb
      FOR POSSIBLE RESUME FROM THIS POINT

     DISTRIBUTED SPARSE MATRIX DIRECT SOLVER.
      Number of equations =         936,    Maximum wavefront =     25

      Local memory allocated for solver              =      0.615 MB
      Local memory required for in-core solution     =      0.592 MB
      Local memory required for out-of-core solution =      0.473 MB

      Total memory allocated for solver              =      1.196 MB
      Total memory required for in-core solution     =      1.152 MB
      Total memory required for out-of-core solution =      0.926 MB

     *** NOTE ***                            CP =       0.685   TIME= 20:37:11
     The Distributed Sparse Matrix Solver is currently running in the
     in-core memory mode.  This memory mode uses the most amount of memory
     in order to avoid using the hard drive as much as possible, which most
     often results in the fastest solution time.  This mode is recommended
     if enough physical memory is present to accommodate all of the solver
     data.
     Distributed sparse solver maximum pivot= 3.561556377E-04 at node 883
     TEMP.
     Distributed sparse solver minimum pivot= 4.451945471E-05 at node 188
     TEMP.
     Distributed sparse solver minimum pivot in absolute value=
     4.451945471E-05 at node 188 TEMP.

    'One or more COMPONENTS exist that do not have all underlying entities selected.  Issuing an ALLSEL or other select commands before CDWRITE will ensure all underlying entities are selected.  These COMPONENTS were not written to the CDWRITE file.\n *** NOTE ***                            CP =       0.622   TIME= 20:37:10\n The automatic domain decomposition logic has selected the MESH domain\n decomposition method with 2 processes per solution.\n\n *****  ANSYS SOLVE    COMMAND  *****\n\n TRANSFER SOLID MODEL BOUNDARY CONDITIONS TO FINITE ELEMENT MODEL\n      SURFACE LOADS  TRANSFERRED FROM AREAS         =     25\n\n *** NOTE ***                            CP =       0.624   TIME= 20:37:10\n There is no title defined for this analysis.\n\n *** ANSYS - ENGINEERING ANALYSIS SYSTEM  RELEASE 2020 R2          20.2     ***\n DISTRIBUTED ANSYS Mechanical Enterprise\n\n 88888888  VERSION=LINUX x64     20:37:10  SEP 23, 2020 CP=      0.625\n\n\n\n\n\n                       S O L U T I O N   O P T I O N S\n\n   PROBLEM DIMENSIONALITY. . . . . . . . . . . . .3-D\n   DEGREES OF FREEDOM. . . . . . TEMP\n   ANALYSIS TYPE . . . . . . . . . . . . . . . . .TRANSIENT\n      SOLUTION METHOD. . . . . . . . . . . . . . .FULL\n   EQUATION SOLVER OPTION. . . . . . . . . . . . .SPARSE\n   GLOBALLY ASSEMBLED MATRIX . . . . . . . . . . .SYMMETRIC\n\n *** NOTE ***                            CP =       0.626   TIME= 20:37:10\n The conditions for direct assembly have been met.  No .emat or .erot\n files will be produced.\n\n\n\n     D I S T R I B U T E D   D O M A I N   D E C O M P O S E R\n\n  ...Number of elements: 625\n  ...Number of nodes:    936\n  ...Decompose to 2 CPU domains\n  ...Element load balance ratio =     1.003\n\n\n                      L O A D   S T E P   O P T I O N S\n\n   LOAD STEP NUMBER. . . . . . . . . . . . . . . .     1\n   TIME AT END OF THE LOAD STEP. . . . . . . . . .  1500.0\n   AUTOMATIC TIME STEPPING . . . . . . . . . . . .    ON\n      STARTING TIME STEP SIZE. . . . . . . . . . .  10.000\n      MINIMUM TIME STEP SIZE . . . . . . . . . . .  2.0000\n      MAXIMUM TIME STEP SIZE . . . . . . . . . . .  25.000\n   TIME STEP RESET ARRAY . . . . . . . . . . . . .  MY_TSRES\n   STEP CHANGE BOUNDARY CONDITIONS . . . . . . . .    NO\n   TRANSIENT (INERTIA) EFFECTS\n      THERMAL DOFS . . . . . . . . . . . . . . . .    ON\n   TRANSIENT INTEGRATION PARAMETERS\n      THETA. . . . . . . . . . . . . . . . . . . .  1.0000\n      OSCILLATION LIMIT CRITERION. . . . . . . . . 0.50000\n      TOLERANCE. . . . . . . . . . . . . . . . . .  0.0000\n   PRINT OUTPUT CONTROLS . . . . . . . . . . . . .NO PRINTOUT\n   DATABASE OUTPUT CONTROLS\n      ITEM     FREQUENCY   COMPONENT\n       ALL        ALL\n\n\n SOLUTION MONITORING INFO IS WRITTEN TO FILE= file.mntr\n\n\n\n\n            **** CENTER OF MASS, MASS, AND MASS MOMENTS OF INERTIA ****\n\n  CALCULATIONS ASSUME ELEMENT MASS AT ELEMENT CENTROID\n\n  TOTAL MASS =  0.36732E-02\n\n                           MOM. OF INERTIA         MOM. OF INERTIA\n  CENTER OF MASS            ABOUT ORIGIN        ABOUT CENTER OF MASS\n\n  XC =   2.5000          IXX =   0.2424E-02      IXX =   0.5877E-03\n  YC =  0.50000          IYY =   0.3181E-01      IYY =   0.7934E-02\n  ZC =  0.50000          IZZ =   0.3181E-01      IZZ =   0.7934E-02\n                         IXY =  -0.4591E-02      IXY =   0.1626E-16\n                         IYZ =  -0.9183E-03      IYZ =   0.3732E-17\n                         IZX =  -0.4591E-02      IZX =   0.1789E-16\n\n\n  *** MASS SUMMARY BY ELEMENT TYPE ***\n\n  TYPE      MASS\n     1  0.367317E-02\n\n Range of element maximum matrix coefficients in global coordinates\n Maximum = 5.168130667E-05 at element 375.\n Minimum = 4.168130667E-05 at element 226.\n\n   *** ELEMENT MATRIX FORMULATION TIMES\n     TYPE    NUMBER   ENAME      TOTAL CP  AVE CP\n\n        1       625  SOLID70       0.009   0.000014\n Time at end of element matrix formulation CP = 0.658279002.\n\n ALL CURRENT ANSYS DATA WRITTEN TO FILE NAME= file.rdb\n  FOR POSSIBLE RESUME FROM THIS POINT\n\n DISTRIBUTED SPARSE MATRIX DIRECT SOLVER.\n  Number of equations =         936,    Maximum wavefront =     25\n\n  Local memory allocated for solver              =      0.615 MB\n  Local memory required for in-core solution     =      0.592 MB\n  Local memory required for out-of-core solution =      0.473 MB\n\n  Total memory allocated for solver              =      1.196 MB\n  Total memory required for in-core solution     =      1.152 MB\n  Total memory required for out-of-core solution =      0.926 MB\n\n *** NOTE ***                            CP =       0.685   TIME= 20:37:11\n The Distributed Sparse Matrix Solver is currently running in the\n in-core memory mode.  This memory mode uses the most amount of memory\n in order to avoid using the hard drive as much as possible, which most\n often results in the fastest solution time.  This mode is recommended\n if enough physical memory is present to accommodate all of the solver\n data.\n Distributed sparse solver maximum pivot= 3.561556377E-04 at node 883\n TEMP.\n Distributed sparse solver minimum pivot= 4.451945471E-05 at node 188\n TEMP.\n Distributed sparse solver minimum pivot in absolute value=\n 4.451945471E-05 at node 188 TEMP.'



Post-Processing
~~~~~~~~~~~~~~~
Animate the temperature as a function of time.  Disable writing to
disk to speed up the animation.


.. code-block:: default


    # Animate every 5th result
    result = mapdl.result
    rnums = range(0, result.nsets, 5)
    result.animate_nodal_solution_set(rnums, stitle='Temperature',
                                      movie_filename='animation.gif',
                                      loop=False)





.. image:: /examples/02-mapdl-examples/images/sphx_glr_transient_thermal_002.png
    :alt: transient thermal
    :class: sphx-glr-single-img


.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none

    Caching scalars:   0%|          | 0/36 [00:00<?, ?it/s]    Caching scalars: 100%|##########| 36/36 [00:00<00:00, 2465.75it/s]

    [(8.29555495773441, 6.295554957734411, 6.295554957734411),
     (2.5, 0.5, 0.5),
     (0.0, 0.0, 1.0)]



Visualize a Slice
~~~~~~~~~~~~~~~~~
Visualize a slice through the dataset using ``pyvista``
for more details visit <https://docs.pyvista.org/>`_.


.. code-block:: default


    # get the temperature of a result set
    nnum, temp = result.nodal_temperature(30)

    # Load this result into the underlying VTK grid
    grid = result.grid
    grid['temperature'] = temp

    # generate a single horizontal slice slice along the XY plane
    single_slice = grid.slice(normal=[0, 0, 1], origin=[0, 0, 0.5])
    single_slice.plot(scalars='temperature')





.. image:: /examples/02-mapdl-examples/images/sphx_glr_transient_thermal_003.png
    :alt: transient thermal
    :class: sphx-glr-single-img


.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none


    [(8.187217274930124, 6.187217274930123, 6.187217274930123),
     (2.5, 0.5, 0.5),
     (0.0, 0.0, 1.0)]



Visualize Several Slices
~~~~~~~~~~~~~~~~~~~~~~~~
This shows how you can visualize a series of slices through a dataset


.. code-block:: default


    # get the temperature of a different result set
    nnum, temp = result.nodal_temperature(120)

    # Load this result into the underlying VTK grid
    grid = result.grid
    grid['temperature'] = temp

    # generate a single horizontal slice slice along the XY plane
    slices = grid.slice_along_axis(7, 'y')
    slices.plot(scalars='temperature', lighting=False, show_edges=True)





.. image:: /examples/02-mapdl-examples/images/sphx_glr_transient_thermal_004.png
    :alt: transient thermal
    :class: sphx-glr-single-img


.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none


    [(8.291303324594558, 6.291303324594559, 6.291303324594559),
     (2.5, 0.5, 0.5),
     (0.0, 0.0, 1.0)]



Temperature at a Single Point
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Extract the temperature at a single node and plot it with respect to
the input temperatures using ``pyansys``


.. code-block:: default


    # get the index of node 12
    idx = np.nonzero(result.mesh.nnum == 12)[0][0]

    # get the temperature at that index for each result
    node_temp = [result.nodal_temperature(i)[1][idx] for i in range(result.nsets)]

    # plot this as a function of time
    plt.plot(result.time_values, node_temp, label='Node 12')
    plt.plot(my_bulk[:, 0], my_bulk[:, 1], ':', label='Input')
    plt.legend()
    plt.xlabel('Time (seconds)')
    plt.ylabel('Temperature ($^\circ$F)')
    plt.show()



.. image:: /examples/02-mapdl-examples/images/sphx_glr_transient_thermal_005.png
    :alt: transient thermal
    :class: sphx-glr-single-img


.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none

    Matplotlib is currently using agg, which is a non-GUI backend, so cannot show the figure.





.. rst-class:: sphx-glr-timing

   **Total running time of the script:** ( 0 minutes  10.679 seconds)


.. _sphx_glr_download_examples_02-mapdl-examples_transient_thermal.py:


.. only :: html

 .. container:: sphx-glr-footer
    :class: sphx-glr-footer-example



  .. container:: sphx-glr-download sphx-glr-download-python

     :download:`Download Python source code: transient_thermal.py <transient_thermal.py>`



  .. container:: sphx-glr-download sphx-glr-download-jupyter

     :download:`Download Jupyter notebook: transient_thermal.ipynb <transient_thermal.ipynb>`


.. only:: html

 .. rst-class:: sphx-glr-signature

    `Gallery generated by Sphinx-Gallery <https://sphinx-gallery.github.io>`_
