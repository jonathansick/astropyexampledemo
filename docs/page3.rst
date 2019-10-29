######
Page 3
######

.. :download:`Download the Astropy README from GitHub <https://raw.githubusercontent.com/astropy/astropy/master/README.rst>`.

.. .. example:: External file download example
..    :tags: includes
.. 
..    :download:`Download the Astropy README from GitHub <https://raw.githubusercontent.com/astropy/astropy/master/README.rst>`.


.. .. example:: Named equation
..    :tags: reference target
.. 
..    .. math:: e^{i\pi} + 1 = 0
..       :label: euler
.. 
..    The Euler equation :math:numref:`euler`.

.. .. math:: e^{i\pi} + 1 = 0
..    :label: euler
.. 
.. The Euler equation :math:numref:`euler`.


.. plot::

   import matplotlib
   import matplotlib.pyplot as plt
   import numpy as np

   # Data for plotting
   x = [1., 2., 3.]
   y = [2., 4., 9.]

   fig, ax = plt.subplots()
   ax.plot(x, y)

   ax.set(xlabel='x', ylabel='y',
          title='Matplotlib plot')
   ax.grid()

.. example:: Matplotlib plot
   :tags: images

   .. plot::

      import matplotlib
      import matplotlib.pyplot as plt
      import numpy as np

      # Data for plotting
      x = [1., 2., 3.]
      y = [2., 4., 9.]

      fig, ax = plt.subplots()
      ax.plot(x, y)

      ax.set(xlabel='x', ylabel='y',
             title='Matplotlib plot')
      ax.grid()
