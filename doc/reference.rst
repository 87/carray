-----------------
Library Reference
-----------------

First-level classes
===================

.. py:class:: cparams(clevel=5, shuffle=True)

    Class to host parameters for compression and other filters.

    Parameters:
      clevel : int (0 <= clevel < 10)
        The compression level.
      shuffle : bool
        Whether the shuffle filter is active or not.

    Notes:
      The shuffle filter may be automatically disable in case it is
      non-sense to use it (e.g. itemsize == 1).


Utility functions
=================

.. py:function:: blosc_set_nthreads(nthreads)

    Set the number of threads that Blosc can use.

    Parameters:
      nthreads : int
        The desired number of threads to use.

    Returns:
      out : int
        The previous setting for the number of threads.

.. py:function:: blosc_version()

    Return the version of the Blosc library.

.. py:function:: detect_number_of_cores()

    Return the number of cores on a system.

.. py:function:: eval(expression, **kwargs)

    Evaluate a Numexpr `expression` and return the result.

    Parameters:
      expression : string
        A string forming an expression, like '2*a+3*b'. The values for 'a' and
        'b' are variable names to be taken from the calling function's frame.
        These variables may be scalars, carrays or NumPy arrays.
      kwargs : list of parameters or dictionary
        Any parameter supported by the carray constructor.

    Returns:
      out : carray object
        The outcome of the expression.  You can taylor the
        properties of this carray by passing additional arguments
        supported by carray constructor in `kwargs`.

.. py:function:: fromiter(iterable, dtype, count=-1, **kwargs)

    Create a carray/ctable from an `iterable` object.

    Parameters:
      iterable : iterable object
        An iterable object providing data for the carray.
      dtype : numpy.dtype instance
        Specifies the type of the outcome object.
      count : int, optional
        Specifies the number of items to read from iterable. The
        default is -1, which means all data is read.
      kwargs : list of parameters or dictionary
        Any parameter supported by the carray/ctable constructors.

    Returns:
      out : a carray/ctable object

    Notes:
      Specify `count` to improve performance.  It allows `fromiter` to
      avoid looping the iterable twice (which is slooow).

.. py:function:: set_nthreads(nthreads)

    Set the number of threads to be used during carray operation.

    This affects to both Blosc and Numexpr (if available).

    Parameters:
      nthreads : int
        The number of threads to be used during carray operation.

    See also:
      :py:func:`blosc_set_nthreads`


The carray class
================

.. py:class:: carray(array, cparams=None, expectedlen=None, chunklen=None)

  A compressed and enlargeable in-memory data container.

  `carray` exposes a series of methods for dealing with the compressed
  container in a NumPy-like way.

  Parameters:
    array : an unidimensional NumPy-like object
      This is taken as the input to create the carray.  It can be any Python
      object that can be converted into a NumPy object.  The data type of
      the resulting carray will be the same as this NumPy object.
    cparams : instance of the `cparams` class, optional
      Parameters to the internal Blosc compressor.
    expectedlen : int, optional
      A guess on the expected length of this carray.  This will serve to
      decide the best `chunklen` used for compression and memory I/O
      purposes.
    chunklen : int, optional
      The number of items that fits on a chunk.  By specifying it you can
      explicitely set the chunk size used for compression and memory I/O.
      Only use it if you know what are you doing.


carray variables
----------------

.. py:attribute:: cbytes

    The compressed size of this object (in bytes).

.. py:attribute:: chunklen

    The number of items that fits into a chunk.

.. py:attribute:: cparams

    The compression parameters for this object.

.. py:attribute:: dtype

    The NumPy dtype for this object.

.. py:attribute:: len

    The length of this object.

.. py:attribute:: nbytes

    The original (uncompressed) size of this object (in bytes).

.. py:attribute:: shape

    The shape of this object.


carray methods
--------------

.. py:method:: append(array)

    Append a numpy `array` to this instance.

    Parameters:
      array : NumPy-like object
        The array to be appended.  Must be compatible with shape and type of
        the carray.

    Returns:
      out : int
        The number of elements appended.


.. py:method:: copy(**kwargs)

    Return a copy of this object.

    Parameters:
      kwargs : list of parameters or dictionary
        Any parameter supported by the carray constructor.

    Returns:
      out : carray object
        The copy of this object.


.. py:method:: iter(start=0, stop=None, step=1)

    Iterator with `start`, `stop` and `step` bounds.

    Parameters:
      start : int
        The starting item.
      stop : int
        The item after which the iterator stops.
      step : int
        The number of items incremented during each iteration.  Cannot be
        negative.

    Returns:
      out : iterator


.. py:method:: where(boolarr)

    Iterator that returns values of this object where `boolarr` is true.

    Parameters:
      boolarr : a carray or NumPy array of boolean type

    Returns:
      out : iterator

    See also:
      :py:func:`wheretrue`

.. py:method:: wheretrue()

    Iterator that returns indices where this object is true.  Only useful for
    boolean carrays.

    Returns:
      out : iterator

    See Also:
      :py:func:`where`


The ctable class
================

.. py:class:: ctable(cols, names=None, **kwargs)

    This class represents a compressed, column-wise, in-memory table.

    Create a new ctable from `cols` with optional `names`.  The
    columns are carray objects.

    Parameters:
      cols : tuple or list of carray/ndarray objects, or structured ndarray
        The list of column data to build the ctable object.
        This can also be a pure NumPy structured array.
      names : list of strings
        The list of names for the columns.  If not passed, the names
        will be chosen as 'f0' for the first column, 'f1' for the
        second and so on so forth (NumPy convention).
      kwargs : list of parameters or dictionary
        Allows to pass additional arguments supported by carray
        constructors in case new carrays need to be built.

    Notes:
      Columns passed as carrays are not be copied, so their settings
      will stay the same, even if you pass additional arguments
      (cparams, chunklen...).


ctable variables
----------------

.. py:attribute:: cbytes

    The compressed size of this object (in bytes).

.. py:attribute:: cols

    The ctable columns (dict).

.. py:attribute:: cparams

    The compression parameters for this object.

.. py:attribute:: dtype

    The NumPy dtype for this object.

.. py:attribute:: len

    The length of this object.

.. py:attribute:: names

   The names of the columns (list).

.. py:attribute:: nbytes

    The original (uncompressed) size of this object (in bytes).

.. py:attribute:: shape

    The shpe of this object.


ctable methods
--------------

.. py:method:: addcol(newcol, name=None, pos=None, **kwargs)

    Add a new `newcol` carray or ndarray as column.

    Parameters:
      newcol : carray or ndarray
        If a carray is passed, no conversion will be carried out.
        If conversion to a carray has to be done, `kwargs` will
        apply.
      name : string, optional
        The name for the new column.  If not passed, it will
        receive an automatic name.
      pos : int, optional
        The column position.  If not passed, it will be appended
        at the end.
      kwargs : list of parameters or dictionary
        Any parameter supported by the carray constructor.

    Notes:
      You should not specificy both `name` and `pos` arguments,
      unless they are compatible.

    See also:
      :py:func:`delcol`


.. py:method:: append(rows)

    Append `rows` to this ctable.

    Parameters:
      rows : list/tuple of scalar values, NumPy arrays or carrays
        It also can be a NumPy record, a NumPy recarray, or
        another ctable.


.. py:method:: copy(**kwargs)

    Return a copy of this ctable.

    Parameters:
      kwargs : list of parameters or dictionary
        Any parameter supported by the carray/ctable constructor.

    Returns:
      out : ctable object
        The copy of this ctable.

.. py:method:: delcol(name=None, pos=None)

    Remove the column named `name` or in position `pos`.

    Parameters:
      name: string, optional
        The name of the column to remove.
      pos: int, optional
        The position of the column to remove.

    Notes:
      You must specify at least a `name` or a `pos`.  You should
      not specificy both `name` and `pos` arguments, unless they
      are compatible.

    See also:
      :py:func:`addcol`


.. py:method:: eval(expression, **kwargs)

    Evaluate the `expression` on columns and return the result.

    Parameters:
      expression : string
        A string forming an expression, like '2*a+3*b'. The values
        for 'a' and 'b' are variable names to be taken from the
        calling function's frame.  These variables may be column
        names in this table, scalars, carrays or NumPy arrays.
      kwargs : list of parameters or dictionary
        Any parameter supported by the carray constructor.

    Returns:
      out : carray object
        The outcome of the expression.  You can taylor the
        properties of this carray by passing additional arguments
        supported by carray constructor in `kwargs`.


.. py:method:: iter(start=0, stop=None, step=1, outcols=None)

    Iterator with `start`, `stop` and `step` bounds.

    Parameters:
      start : int
        The starting item.
      stop : int
        The item after which the iterator stops.
      step : int
        The number of items incremented during each iteration.  Cannot be
        negative.
      outcols : list of strings
        The list of column names that you want to get back in results.  If
        None, all the columns are returned.  If the special name
        '__nrow__' is present, the number of row will be included in
        output.

    Returns:
      out : iterable

.. py:method:: where(expression, outcols=None)

    Iterate over rows where `expression` is true.

    Parameters:
      expression : string or carray
        A boolean Numexpr expression or a boolean carray.
      outcols : list of strings
        The list of column names that you want to get back in results.  If
        None, all the columns are returned.  If the special name
        '__nrow__' is present, the number of row will be included in
        output.

    Returns:
      out : iterable
        This iterable returns rows as NumPy structured types (i.e. they
        support being mapped either by position or by name).

