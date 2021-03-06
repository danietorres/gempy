
Chapter 1: GemPy Basic
======================

In this first example, we will show how to construct a first basic model
and the main objects and functions. First we import gempy:

.. code:: ipython3

    # These two lines are necessary only if gempy is not installed
    import sys, os
    sys.path.append("../")
    
    # Importing gempy
    import gempy as gp
    
    # Embedding matplotlib figures into the notebooks
    %matplotlib inline
    
    # Aux imports
    import numpy as np

All data get stored in a python object InputData. This object can be
easily stored in a Python pickle. However, these files have the
limitation that all dependecies must have the same versions as those
when the pickle were created. For these reason to have more stable
tutorials we will generate the InputData from raw data---i.e. csv files
exported from Geomodeller.

These csv files can be found in the input\_data folder in the root
folder of GemPy. These tables contains uniquely the XYZ (and poles,
azimuth and polarity in the foliation case) as well as their respective
formation name (but not necessary the formation order).

.. code:: ipython3

    # Importing the data from csv files and settign extent and resolution
    geo_data = gp.create_data([0,2000,0,2000,-2000,0],[ 50,50,50],
                             path_f = os.pardir+"/input_data/FabLessPoints_Foliations.csv",
                             path_i = os.pardir+"/input_data/FabLessPoints_Points.csv")

With the command get data is possible to see all the input data.
However, at the moment the (depositional) order of the formation and the
separation of the series (More explanation about this in the next
notebook) is totally arbitrary.

.. code:: ipython3

    gp.get_data(geo_data).head()


::


    ---------------------------------------------------------------------------

    NameError                                 Traceback (most recent call last)

    <ipython-input-1-ceab18c1653c> in <module>()
    ----> 1 gp.get_data(geo_data).head()
    

    NameError: name 'gp' is not defined


To set the number of (depositional) series and which formation belongs
to which, it is possible to use the function set\_series. Here, there
are two important things to notice:

-  set\_series requires a dictionary. In Python dictionaries are not
   order (keys dictionaries since Python 3.6 are) and therefore there is
   not guarantee of having the right order.

   -  The order of the series are vital for the method (from younger to
      older).
   -  The order of the formations (as long they belong to the correct
      series) are only important for the color code. If the order of the
      pile differs from the final result the color of the interfaces and
      input data will be different

-  Every fault is treated as an independent series and **have to be at
   the top of the pile**. The relative order between the distinct faults
   represent the tectonic relation between them (from younger to older
   as well).

The order of the series (for Python < 3.6, otherwise passing the correct
order of keys in the dictionary is enough) can be given as attribute as
in the cell below. For the order of formations can be passed as
attribute as well or using the specific function set\_order\_formations.

.. code:: ipython3

    # Assigning series to formations as well as their order (timewise)
    gp.set_series(geo_data, {"fault":'MainFault', 
                          "Rest": ('SecondaryReservoir','Seal', 'Reservoir', 'Overlying')},
                           order_series = ["fault", 'Rest'],
                           order_formations=['MainFault', 
                                             'SecondaryReservoir', 'Seal','Reservoir', 'Overlying',
                                             ]) 



.. parsed-literal::

    <IPython.core.display.Javascript object>



.. raw:: html

    <img src="data:image/png;base64,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" width="747.9999777078635">




.. parsed-literal::

    <gempy.strat_pile.StratigraphicPile at 0x7f6e21b43048>



As an alternative the stratigraphic pile is interactive given the right
backend (try %matplotlib notebook or %matplotlib qt5). These backends
sometimes give some trouble though. Try to execute the cell twice:

.. code:: ipython3

    %matplotlib notebook
    gp.get_stratigraphic_pile(geo_data)



.. parsed-literal::

    <IPython.core.display.Javascript object>



.. raw:: html

    <img src="data:image/png;base64,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" width="747.9999777078635">




.. parsed-literal::

    <gempy.strat_pile.StratigraphicPile at 0x7f6e21c1d470>



Notice that the colors depends on the order and therefore every time the
cell is executed the colors are always in the same position. Be aware of
the legend to be sure that the pile is as you wish!! (In the future
every color will have the annotation within the rectangles to avoid
confusion)

This geo\_data object contains essential information that we can access
through the correspondent getters. Such a the coordinates of the grid.

.. code:: ipython3

    print(gp.get_grid(geo_data))


.. parsed-literal::

    [[    0.             0.         -2000.        ]
     [    0.             0.         -1959.18371582]
     [    0.             0.         -1918.36730957]
     ..., 
     [ 2000.          2000.           -81.63265228]
     [ 2000.          2000.           -40.81632614]
     [ 2000.          2000.             0.        ]]


The main input the potential field method is the coordinates of
interfaces points as well as the orientations. These pandas dataframes
can we access by the following methods:

Interfaces Dataframe
^^^^^^^^^^^^^^^^^^^^

.. code:: ipython3

    gp.get_data(geo_data, 'interfaces').head()




.. raw:: html

    <div>
    <style>
        .dataframe thead tr:only-child th {
            text-align: right;
        }
    
        .dataframe thead th {
            text-align: left;
        }
    
        .dataframe tbody tr th {
            vertical-align: top;
        }
    </style>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>X</th>
          <th>Y</th>
          <th>Z</th>
          <th>formation</th>
          <th>series</th>
          <th>annotations</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>1000</td>
          <td>1000</td>
          <td>-1000</td>
          <td>MainFault</td>
          <td>fault</td>
          <td>${\bf{x}}_{\alpha \,{\bf{1}},0}$</td>
        </tr>
        <tr>
          <th>1</th>
          <td>800</td>
          <td>1000</td>
          <td>-1600</td>
          <td>MainFault</td>
          <td>fault</td>
          <td>${\bf{x}}_{\alpha \,{\bf{1}},1}$</td>
        </tr>
        <tr>
          <th>2</th>
          <td>1200</td>
          <td>1000</td>
          <td>-400</td>
          <td>MainFault</td>
          <td>fault</td>
          <td>${\bf{x}}_{\alpha \,{\bf{1}},2}$</td>
        </tr>
        <tr>
          <th>3</th>
          <td>1100</td>
          <td>1000</td>
          <td>-700</td>
          <td>MainFault</td>
          <td>fault</td>
          <td>${\bf{x}}_{\alpha \,{\bf{1}},3}$</td>
        </tr>
        <tr>
          <th>4</th>
          <td>900</td>
          <td>1000</td>
          <td>-1300</td>
          <td>MainFault</td>
          <td>fault</td>
          <td>${\bf{x}}_{\alpha \,{\bf{1}},4}$</td>
        </tr>
      </tbody>
    </table>
    </div>



Foliations Dataframe
^^^^^^^^^^^^^^^^^^^^

Now the formations and the series are correctly set.

.. code:: ipython3

    gp.get_data(geo_data, 'foliations').head()




.. raw:: html

    <div>
    <style>
        .dataframe thead tr:only-child th {
            text-align: right;
        }
    
        .dataframe thead th {
            text-align: left;
        }
    
        .dataframe tbody tr th {
            vertical-align: top;
        }
    </style>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>X</th>
          <th>Y</th>
          <th>Z</th>
          <th>dip</th>
          <th>azimuth</th>
          <th>polarity</th>
          <th>formation</th>
          <th>series</th>
          <th>annotations</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>917.45</td>
          <td>1000.0</td>
          <td>-1135.398</td>
          <td>71.565</td>
          <td>270.0</td>
          <td>1</td>
          <td>MainFault</td>
          <td>fault</td>
          <td>${\bf{x}}_{\beta \,{\bf{1}},0}$</td>
        </tr>
        <tr>
          <th>1</th>
          <td>1450.00</td>
          <td>1000.0</td>
          <td>-1150.000</td>
          <td>18.435</td>
          <td>90.0</td>
          <td>1</td>
          <td>Reservoir</td>
          <td>Rest</td>
          <td>${\bf{x}}_{\beta \,{\bf{4}},0}$</td>
        </tr>
      </tbody>
    </table>
    </div>



It is important to notice the columns of each data frame. These not only
contains the geometrical properties of the data but also the
**formation** and **series** at which they belong. This division is
fundamental in order to preserve the depositional ages of the setting to
model.

A projection of the aforementioned data can be visualized in to 2D by
the following function. It is possible to choose the direction of
visualization as well as the series:

.. code:: ipython3

    %matplotlib inline
    gp.plot_data(geo_data, direction='y')



.. image:: ch1_files/ch1_19_0.png


GemPy supports visualization in 3D as well trough vtk. These plots are
interactive. Try to drag and drop a point or interface! In the
perpendicular views only 2D movements are possible to help to place the
data where is required.

.. code:: ipython3

    gp.plot_data_3D(geo_data)

The ins and outs of Input data objects
--------------------------------------

As we have seen objects DataManagement.InputData (usually called
geo\_data in the tutorials) aim to have all the original geological
properties, measurements and geological relations stored.

Once we have the data ready to generate a model, we will need to create
the next object type towards the final geological model:

.. code:: ipython3

    interp_data = gp.InterpolatorInput(geo_data, u_grade=[3,3])
    print(interp_data)


.. parsed-literal::

    Level of Optimization:  fast_compile
    Device:  cpu
    Precision:  float32
    <gempy.DataManagement.InterpolatorInput object at 0x7f6e219505c0>


.. code:: ipython3

    interp_data.get_formation_number()




.. parsed-literal::

    {'DefaultBasement': 0,
     'MainFault': 1,
     'Overlying': 5,
     'Reservoir': 4,
     'Seal': 3,
     'SecondaryReservoir': 2}



By default (there is a flag in case you do not need) when we create a
interp\_data object we also compile the theano function that compute the
model. That is the reason why takes long.

gempy.DataManagement.InterpolatorInput (usually called interp\_data in
the tutorials) prepares the original data to the interpolation algorithm
by scaling the coordinates for better and adding all the mathematical
parametrization needed.

.. code:: ipython3

    gp.get_kriging_parameters(interp_data)


.. parsed-literal::

    range 0.8882311582565308 3464.1015172
    Number of drift equations [2 2]
    Covariance at 0 0.01878463476896286
    Foliations nugget effect 0.009999999776482582


These later parameters have a default value computed from the original
data or can be changed by the user (be careful of changing any of these
if you do not fully understand their meaning).

At this point, we have all what we need to compute our model. By default
everytime we compute a model we obtain:

-  Lithology block model

   -  with the lithology values in 0
   -  with the potential field values in 1

-  Fault block model

   -  with the faults zones values (i.e. every divided region by each
      fault has one number) in 0
   -  with the potential field values in 1

.. code:: ipython3

    lith_block, fault_block = gp.compute_model(interp_data)

This solution can be plot with the correspondent plotting function.
Blocks:

.. code:: ipython3

    %matplotlib inline
    gp.plot_section(geo_data, lith_block[0], 25, plot_data=True)



.. image:: ch1_files/ch1_30_0.png


Potential field:

.. code:: ipython3

    gp.plot_potential_field(geo_data, lith_block[1], 25)



.. image:: ch1_files/ch1_32_0.png


From the potential fields (of lithologies and faults) it is possible to
extract vertices and simpleces to create the 3D triangles for a vtk
visualization.

.. code:: ipython3

    ver, sim = gp.get_surfaces(interp_data,lith_block[1], fault_block[1], original_scale=True)

.. code:: ipython3

    gp.plot_surfaces_3D(geo_data, ver, sim, alpha=1)

Additionally is possible to update the model and recompute the surfaces
in real time. To do so, we need to pass the data rescaled. To get an
smooth response is important to have the theano optimizer flag in
fast\_run and run theano in the gpu. This can speed up the modeling time
in a factor of 20.

.. code:: ipython3

    ver_s, sim_s = gp.get_surfaces(interp_data,lith_block[1],
                                   fault_block[1],
                                   original_scale=False)

.. code:: ipython3

    gp.plot_surfaces_3D_real_time(interp_data, ver_s, sim_s)

In the same manner we can visualize the fault block:

.. code:: ipython3

    gp.plot_section(geo_data, fault_block[0], 25)



.. image:: ch1_files/ch1_40_0.png


.. code:: ipython3

    gp.plot_potential_field(geo_data, fault_block[1], 25)



.. image:: ch1_files/ch1_41_0.png

