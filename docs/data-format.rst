.. highlight:: shell

==============
Formatting datasets
==============

This section describes how to format your own datasets for importing into HistomicsML. A datasets consists of whole-slide images (.tif), a slide description (.csv), object boundaries (.txt) and histomic features (.h5).

Whole-slide images
------------------

Whole-slide images need to be converted to a pyramidal .tif format that is compatible with the IIPImage server (http://iipimage.sourceforge.net/documentation/server/). We have used Vips (http://www.vips.ecs.soton.ac.uk/index.php?title=VIPS)
to perform this conversion for our datasets.

.. note:: The path to the image needs to be saved in the database.
   HistomicsML uses the database to get the path when forming a request for the IIPIMage server.


Slide description
------------------------------------
A table (.csv) needs to be created to capture the dimensions, magnification, and location of the files for each slide image:

.. code-block:: bash

  <slide name>,<width in pixels>,<height in pixels>,<path to the pyramid on IIPServer>,<scale>

where scale = 1 for 20x and scale = 2 for 40x.

For the sample data provided in the database container, our slide description file (GBM-pyramids.csv) has the following contents:

.. code-block:: bash

   TCGA-02-0010-01Z-00-DX4,32001,38474,/fastdata/pyramids/GBM/TCGA-02-0010-01Z-00-DX4.svs.dzi.tif,1



Object boundaries
----------
Boundary information is formatted as a tab-delimited text file where each line describes the centroids and boundary coordinates for one object:

.. code-block:: bash

  <slide name> \t <centroid x coordinate> \t <centroid y coordinate> \t <boundary points>

where \t is a tab character and <boundary points> are formatted as:
x1,y1 x2,y2 x3,y3 ... xN,yN (with spaces between coordinate pairs)

One line from the sample data boundaries file (GBM-boundaries.txt):

.. code-block:: bash

  TCGA-02-0010-01Z-00-DX4 2250.1 4043.0 2246,4043 2247,4043 2247 ... 2247,4043 2246,4043



Histomic features
--------

Features are stored in an HDF5 binary array format. The HDF5 file contains the following variables:

.. code-block:: bash

  /features - A D x N array of floats containing the feature values for each object in the dataset (N objects, each with D features). Each feature/row should be normalized by z-score.
  /slides -	Names of the slides/images in the dataset
  /slideIdx - N-length array containing the slide index of each object. These indices can be used with the 'slides' variable to determine what slide each object originates from.
  /x_centroid - N-length array of floats containing the x coordinate of object centroids.
  /y_centroid - N-length array of floats containing the x coordinate of object centroids.
  /dataIdx - Array containing the index of the first object of each slide in 'features', 'x_centroid', and 'y_centroid' (this information can also be obtained from 'slideIdx' and will be eliminated in the future).
  /mean - A D-length array containing the mean of each feature prior to normalization. This provides a record of z-score normalization parameters so that the data can be de-normalized if needed.
  /std_dev - A D-length array containing the standard deviation of each feature prior to normalization. This provides a record of z-score normalization parameters so that the data can be de-normalized if needed.

The sample file (GBM-features.h5) provided in the database docker container can be queried to examine the structure with the following the command.

.. code-block:: python

  >>> import h5py
  >>> file="GBM-features.h5"
  >>> contents = h5py.File(file)
  >>> for i in contents:
  ...     print i
  ...
  # for loop will print out the feature information under the root of HDF5.

  dataIdx
  features
  mean
  slideIdx
  slides
  std_dev
  x_centroid
  y_centroid

  #for further step, if you want to see the details.

  >>> contents['features'][0]
  array([ -7.30991781e-01,  -8.36540878e-01,  -1.07858682e+00,
         9.26770031e-01,  -9.31272805e-01,  -4.36136842e-01,
        -1.13033086e-01,   5.28297901e-01,   6.85962856e-01,
         5.07918596e-01,  -5.27561486e-01,  -7.48096228e-01,
        -6.84849143e-01,  -8.79032671e-01,  -1.41368553e-01,
        -3.24195564e-01,  -4.50991303e-01,  -1.32366025e+00,
         9.17324543e-01,   8.36400129e-03,  -2.92657673e-01,
         2.01028720e-01,  -1.93680093e-01,   8.68237793e-01,
         5.72155595e-01,   3.29810083e-01,  -3.63551527e-01,
        -2.87026823e-01,  -8.47819634e-03,  -4.55458522e-01,
         1.43787396e+00,   5.24487114e+00,  -9.62561846e-01,
         5.94001710e-01,   3.57634330e+00,  -2.94562435e+00,
        -9.18125820e+00,   2.87391472e+01,  -9.34123135e+00,
         2.55983505e+01,  -2.99653459e+00,  -1.17376029e-01,
        -5.40324259e+00,   1.01094952e+01,   5.87054205e+00,
         6.21094942e+00,  -2.59355903e+00,  -4.27142763e+00], dtype=float32)
