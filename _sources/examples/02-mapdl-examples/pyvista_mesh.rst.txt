.. only:: html

    .. note::
        :class: sphx-glr-download-link-note

        Click :ref:`here <sphx_glr_download_examples_02-mapdl-examples_pyvista_mesh.py>`     to download the full example code
    .. rst-class:: sphx-glr-example-title

    .. _sphx_glr_examples_02-mapdl-examples_pyvista_mesh.py:


.. _ref_pyvista_mesh:

PyVista Mesh Integration
~~~~~~~~~~~~~~~~~~~~~~~~

Run a modal analysis on a mesh generated from pyvista within MAPDL.


.. code-block:: default

    # sphinx_gallery_thumbnail_number = 2

    import os
    import pyvista as pv
    import pyansys

    # launch MAPDL and run a modal analysis
    os.environ['I_MPI_SHM_LMT'] = 'shm'  # necessary for Ubuntu
    mapdl = pyansys.launch_mapdl(loglevel='WARNING', override=True)

    # Create a simple plane mesh centered at (0, 0, 0) on the XY plane
    mesh = pv.Plane(i_resolution=100, j_resolution=100)

    mesh.plot(color='w', show_edges=True)




.. image:: /examples/02-mapdl-examples/images/sphx_glr_pyvista_mesh_001.png
    :alt: pyvista mesh
    :class: sphx-glr-single-img


.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none


    [(1.5773502691896262, 1.5773502691896262, 1.5773502691896262),
     (0.0, 0.0, 0.0),
     (0.0, 0.0, 1.0)]



Write the mesh to an archive file


.. code-block:: default

    archive_filename = os.path.join(mapdl.directory, 'tmp.cdb')
    pyansys.save_as_archive(archive_filename, mesh)









mapdl = pyansys.launch_mapdl(prefer_pexpect=True, override=True)


.. code-block:: default


    # Read in the archive file
    response = mapdl.cdread('db', archive_filename)
    mapdl.prep7()
    print(mapdl.shpp('SUMM'))

    # specify shell thickness
    mapdl.sectype(1, "shell")
    mapdl.secdata(0.01)
    mapdl.emodif('ALL', 'SECNUM', 1)

    # specify material properties
    # using aprox values for AISI 5000 Series Steel
    # http://www.matweb.com/search/datasheet.aspx?matguid=89d4b891eece40fbbe6b71f028b64e9e
    mapdl.units('SI')  # not necessary, but helpful for book keeping
    mapdl.mp('EX', 1, 200E9)  # Elastic moduli in Pa (kg/(m*s**2))
    mapdl.mp('DENS', 1, 7800)  # Density in kg/m3
    mapdl.mp('NUXY', 1, 0.3)  # Poissons Ratio
    mapdl.emodif('ALL', 'MAT', 1)

    # Run an unconstrained modal analysis
    # for the first 20 modes above 1 Hz
    mapdl.modal_analysis(nmode=20, freqb=1)

    # you could have also run:
    # mapdl.run('/SOLU')
    # mapdl.antype('MODAL')  # default NEW
    # mapdl.modopt('LANB', 20, 1)
    # mapdl.solve()

    mapdl.exit()





.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none

    SUMMARIZE SHAPE TESTING FOR ALL SELECTED ELEMENTS

     ------------------------------------------------------------------------------
                <<<<<<          SHAPE TESTING SUMMARY           >>>>>>
                <<<<<<        FOR ALL SELECTED ELEMENTS         >>>>>>
     ------------------------------------------------------------------------------
                        --------------------------------------
                        |  Element count     10000 SHELL181  |
                        --------------------------------------

      Test                Number tested  Warning count  Error count    Warn+Err %
      ----                -------------  -------------  -----------    ----------
      Aspect Ratio              10000              0             0         0.00 %
      Parallel Deviation        10000              0             0         0.00 %
      Maximum Angle             10000              0             0         0.00 %
      Jacobian Ratio            10000              0             0         0.00 %
      Warping Factor            10000              0             0         0.00 %

      Any                       10000              0             0         0.00 %
     ------------------------------------------------------------------------------




Load the result file within ``pyansys`` and plot the 8th mode.


.. code-block:: default

    result = mapdl.result
    print(result)

    result.plot_nodal_displacement(7, show_displacement=True, displacement_factor=0.4)




.. image:: /examples/02-mapdl-examples/images/sphx_glr_pyvista_mesh_002.png
    :alt: pyvista mesh
    :class: sphx-glr-single-img


.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none

    PyANSYS MAPDL Result
    Units       : User Defined
    Version     : 20.2
    Cyclic      : False
    Result Sets : 20
    Nodes       : 10201
    Elements    : 10000


    Available Results:
    NSL : Nodal displacements


    [(1.601861535420676, 1.6018615354206405, 1.5911447281433078),
     (1.8707257964933888e-14, -1.6930901125533637e-14, -0.01071680727734959),
     (0.0, 0.0, 1.0)]



plot the 1st mode using contours


.. code-block:: default

    result.plot_nodal_displacement(0, show_displacement=True,
                                   displacement_factor=0.4, n_colors=10)




.. image:: /examples/02-mapdl-examples/images/sphx_glr_pyvista_mesh_003.png
    :alt: pyvista mesh
    :class: sphx-glr-single-img


.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none


    [(1.6007557758794801, 1.6007557758794804, 1.6007557758920548),
     (-2.7755575615628914e-16, 0.0, 1.2574281893495964e-11),
     (0.0, 0.0, 1.0)]



Animate a high frequency mode

Get a smoother plot by disabling movie_filename and increasing ``nangles``.
Enable a continous plot looping with ```loop=True```.


.. code-block:: default


    result.animate_nodal_displacement(18, loop=False, add_text=False,
                                      nangles=30, displacement_factor=0.4,
                                      show_axes=False, background='w',
                                      movie_filename='plane_vib.gif')



.. image:: /examples/02-mapdl-examples/images/sphx_glr_pyvista_mesh_004.png
    :alt: pyvista mesh
    :class: sphx-glr-single-img


.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none


    [(1.5773502691896262, 1.5773502691896262, 1.5773502691896262),
     (0.0, 0.0, 0.0),
     (0.0, 0.0, 1.0)]




.. rst-class:: sphx-glr-timing

   **Total running time of the script:** ( 0 minutes  8.485 seconds)


.. _sphx_glr_download_examples_02-mapdl-examples_pyvista_mesh.py:


.. only :: html

 .. container:: sphx-glr-footer
    :class: sphx-glr-footer-example



  .. container:: sphx-glr-download sphx-glr-download-python

     :download:`Download Python source code: pyvista_mesh.py <pyvista_mesh.py>`



  .. container:: sphx-glr-download sphx-glr-download-jupyter

     :download:`Download Jupyter notebook: pyvista_mesh.ipynb <pyvista_mesh.ipynb>`


.. only:: html

 .. rst-class:: sphx-glr-signature

    `Gallery generated by Sphinx-Gallery <https://sphinx-gallery.github.io>`_
