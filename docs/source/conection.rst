Usage
=====

.. _ssh:

Conecting to the LVMM HPC cluster
------------

The conectión to the LVMM computational cluster of the Nanomaterial Modeling Department is via the `OpenSSH <https://www.openssh.com>`_ protocol; While in the CNyN's internal network you can connect with the following command:

.. code-block:: console
   $ ssh nick@192.168.100.237

where `nick` is the username with which we are loging in LVMM, if the comunicatión is sucsesfull we will be prompted for the user's password:

.. code-block:: console
   By accessing this system, you consent to the following conditions:
   - This system is for authorized use only.
   - Any or all uses of this system and all files on this system may be monitored.
   - Communications using, or data stored on, this system are not private.
   
   usuario@192.168.100.237's password:

.. code-block:: console





   (.venv) $ pip install lumache

Creating recipes
----------------

To retrieve a list of random ingredients,
you can use the ``lumache.get_random_ingredients()`` function:

.. autofunction:: lumache.get_random_ingredients

The ``kind`` parameter should be either ``"meat"``, ``"fish"``,
or ``"veggies"``. Otherwise, :py:func:`lumache.get_random_ingredients`
will raise an exception.

.. autoexception:: lumache.InvalidKindError

For example:

>>> import lumache
>>> lumache.get_random_ingredients()
['shells', 'gorgonzola', 'parsley']

