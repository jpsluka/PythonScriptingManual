Passing information between steppables
======================================

When you work with more than one steppable (and it is a good idea to
work with several steppables each of which has a well defined purpose) you
may sometimes need to access or change a variable of one steppable
inside the code of another steppable. The most straightforward method to implement exchange of
information between steppables is to create a global python module (global from the simulation's stand point),
lets call it, ``global_vars.py``. Lets declare a parameter ``shared_parameter`` inside ``global_vars.py`` :

.. code-block:: python

    shared_parameter = 10

then we declare two steppables (e.g. in two different files)

.. code-block:: python

    # file SteppableCommunication.py
    import global_vars

    class SteppableCommunicationSteppable(SteppableBasePy):
        def __init__(self, frequency=1):
            SteppableBasePy.__init__(self, frequency)

        def step(self, mcs):

            print ('global_vars.shared_parameters=', global_vars.shared_parameters)


.. code-block:: python

    # file ExtraSteppable.py
    import global_vars

    class ExtraSteppable(SteppableBasePy):
        def __init__(self, frequency=1):
            SteppableBasePy.__init__(self, frequency)
            global_vars.shared_parameters = 25

        def step(self, mcs):
            print ("ExtraSteppable: This function is called every 1 MCS")


``ExtraSteppable`` modifies ``global_vars.shared_parameter`` and ``SteppableCommunicationSteppable`` access it. 

The global_vars.py file is a standard python file and can include computation. For example, you might want to define the width of the model in pixels and then use that information to scale the size of your simulated cells. 

.. code-block:: python

    # this is for a 2D model
    model_width_pixel = 50 # in pixels
    model_width_cells = 10  # in cells
    cell_target_volume = (model_width_pixel/model_width_cells)**2  # in pixels^2
    cell_target_surface = (model_width_pixel/model_width_cells)*6. # in pixels
    
In the above the value of cell_target_volume is available to any python module that loads 
the ``global_vars.py``. Using this approach the modeler can use a low resolution model 
that runs quickly for model development, and then, by changing only one line in the 
parameters file, increase the resolution (and run time) of the model.

Note that using this approach you can only change values in your model that are defined in 
python modules such as steppable. However, you cannot  use this approach to change values 
in the project's xml file. 
To use this approach in a way that allows full control of all parmeters in a simulation you 
can use a python-only approach. The python-only approach would allow you to change model 
dimensions, number of MCS, cell paramters etc.
    
You may, come up with more complex use case but this simple one illustrates the core mechanism where we use additional python module (``global_vars.py``) to exchange information between different classes
