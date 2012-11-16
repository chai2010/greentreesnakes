Meet the Nodes
==============

.. currentmodule: ast

Literals
--------

.. class:: Num(n)

   A number - integer, float, or complex. The ``n`` attribute stores the value,
   already converted to the relevant type.

.. class:: Str(s)

   A string. The ``s`` attribute hold the value. In Python 2, the same type
   holds unicode strings too.

.. class:: Bytes(s)

   A :class:`bytes` object. The ``s`` attribute holds the value. Python 3 only.

.. class:: List(elts, ctx)
           Tuple(elts, ctx)

   A list or tuple. ``elts`` holds a list of nodes representing the elements.
   ``ctx`` is :class:`Store` if the container is an assignment target (i.e.
   ``(x,y)=pt``), and :class:`Load` otherwise.

.. class:: Set(elts)

   A set. ``elts`` holds a list of nodes representing the elements.

.. class:: Dict(keys, values)

   A dictionary. ``keys`` and ``values`` hold lists of nodes with matching order
   (i.e. they could be paired with :func:`zip`).

.. class:: Ellipsis()

   Represents the ``...`` syntax for the ``Ellipsis`` singleton.

Variables
---------

.. class:: Name(id, ctx)

   A variable name. ``id`` holds the name as a string, and ``ctx`` is one of
   the following types.
   
.. class:: Load()
           Store()
           Del()

   Variable referemces can be used to load the value of a variable, to assign
   a new value to it, or to delete it. Variable references are given a context
   to distinguish these cases.

::

    >>> parseprint("a")      # Loading a
    Module(body=[
        Expr(value=Name(id='a', ctx=Load())),
      ])
    
    >>> parseprint("a = 1")  # Storing a
    Module(body=[
        Assign(targets=[
            Name(id='a', ctx=Store()),
          ], value=Num(n=1)),
      ])

    >>> parseprint("del a")  # Deleting a
    Module(body=[
        Delete(targets=[
            Name(id='a', ctx=Del()),
          ]),
      ])


.. note::
   The pretty-printer used in these examples is available `in the source repository
   <https://bitbucket.org/takluyver/greentreesnakes/src/default/astpp.py>`_ for
   Green Tree Snakes.


Expressions
-----------

.. class:: Expr(value)

   A container for an expression when it appears as a statement by itself.
   ``value`` holds one of the other nodes in this section, or a literal or name.

.. class:: UnaryOp(op, operand)

   A unary operation. ``op`` is the operator, and ``operand`` any expression
   node.

.. class:: BinOp(left, op, right)

   A binary operation (like addition or division). ``op`` is the operator, and
   ``left`` and ``right`` are any expression nodes.

.. class:: BoolOp(op, values)

   A boolean operation, 'or' or 'and'. ``op`` is :class:`Or` or
   :class:`And`. ``values`` are the values involved. Consecutive operations
   with the same operator, such as ``a or b or c``, are collapsed into one node
   with several values.
   
   This doesn't include ``not``, which is a :class:`UnaryOp`.

.. class:: Compare(left, ops, comparators)

   A comparison of two or more values. ``left`` is the first value in the
   comparison, ``ops`` the list of operators, and ``comparators`` the list of
   values after the first. If that sounds awkward, that's because it is::
   
      >>> parseprint("1 < a < 10")
      Module(body=[
        Expr(value=Compare(left=Num(n=1), ops=[
            Lt(),
            Lt(),
          ], comparators=[
            Name(id='a', ctx=Load()),
            Num(n=10),
          ])),
        ])


.. class:: Call(func, args, keywords, starargs, kwargs)

   A function call. ``func`` is the function, which will normally be a
   :class:`Name` object. Of the arguments:

   * ``args`` holds a list of the arguments passed by position.
   * ``keywords`` holds a list of :class:`keyword` objects representing
     arguments passed by keyword.%
   * ``starargs`` and ``kwargs`` each hold a single node, for arguments passed
     as ``*args`` and ``**kwargs``.
   
   When constructing a Call node, ``args`` and ``kwargs`` are required, but they
   can be empty lists. ``starargs`` and ``kwargs`` are optional.
   
   ::

       >>> parseprint("func(a, b=c, *d, **e)")
       Module(body=[
           Expr(value=Call(func=Name(id='func', ctx=Load()),
                           args=[Name(id='a', ctx=Load())],
                           keywords=[keyword(arg='b', value=Name(id='c', ctx=Load()))],
                           starargs=Name(id='d', ctx=Load()),
                           kwargs=Name(id='e', ctx=Load()))),
         ])

.. class:: Attribute(value, attr, ctx)

   Attribute access, e.g. ``d.keys``. ``value`` is a node, typically a
   :class:`Name`. ``attr`` is a bare string giving the name of the attribute,
   and ``ctx`` is :class:`Load`, :class:`Store` or :class:`Del` according to
   how the attribute is acted on.

Subscripting
~~~~~~~~~~~~

.. class:: Subscript(value, slice, ctx)

   A subscript, such as ``l[1]``. ``value`` is the object, often a
   :class:`Name`. ``slice`` is one of :class:`Index`, :class:`Slice`
   or :class:`ExtSlice`. ``ctx`` is :class:`Load`, :class:`Store` or :class:`Del`
   according to what it does with the subscript.

.. class:: Index(value)

   Simple subscripting with a single value::
   
       >>> parseprint("l[1]")
       Module(body=[
         Expr(value=Subscript(value=Name(id='l', ctx=Load()),
                              slice=Index(value=Num(n=1)), ctx=Load())),
         ])

.. class:: Slice(lower, upper, step)

   Regular slicing::
   
       >>> parseprint("l[1:2]")
       Module(body=[
         Expr(value=Subscript(value=Name(id='l', ctx=Load()),
                         slice=Slice(lower=Num(n=1), upper=Num(n=2), step=None),
                         ctx=Load())),
         ])

.. class:: ExtSlice(dims)

   Advanced slicing. ``dims`` holds a list of :class:`Slice` and
   :class:`Index` nodes::
   
       >>> parseprint("l[1:2, 3]")
       Module(body=[
           Expr(value=Subscript(value=Name(id='l', ctx=Load()), slice=ExtSlice(dims=[
               Slice(lower=Num(n=1), upper=Num(n=2), step=None),
               Index(value=Num(n=3)),
             ]), ctx=Load())),
         ])

Statements
----------

.. class:: Assign(targets, value)

   An assignment. ``targets`` is a list of nodes, and ``value`` is a single node.
   
   Multiple nodes in ``targets`` represents assigning the same value to each.
   Unpacking is represented by putting a :class:`Tuple` or :class:`List`
   within ``targets``.
   
   >>> parseprint("a = b = 1")     # Multiple assignment
   Module(body=[
       Assign(targets=[
          Name(id='a', ctx=Store()),
          Name(id='b', ctx=Store()),
        ], value=Num(n=1)),
     ])
   
   >>> parseprint("a,b = c")       # Unpacking
   Module(body=[
       Assign(targets=[
           Tuple(elts=[
               Name(id='a', ctx=Store()),
               Name(id='b', ctx=Store()),
             ], ctx=Store()),
         ], value=Name(id='c', ctx=Load())),
     ])

.. class:: Print(dest, values, nl)

   Print statement, Python 2 only. ``dest`` is an optional destination (for
   ``print >>dest``. ``values is a list of nodes. ``nl`` (newline) is True or
   False depending on whether there's a comma at the end of the statement.

.. class:: Raise(exc, cause)

   Raising an exception, Python 3 syntax. ``exc`` is the exception object to be
   raised, normally a :class:`Call` or :class:`Name`, or ``None`` for
   a standalone ``raise``. ``cause`` is the optional part for ``y`` in
   ``raise x from y``.
   
   In Python 2, the parameters are  instead ``type, inst, tback``, which
   correspond to the old ``raise x, y, z`` syntax.

.. class:: Assert(test, msg)

   An assertion. ``test`` holds the condition, such as a :class:`Compare` node.
   ``msg`` holds the failure message, normally a :class:`Str` node.

.. class:: Delete(targets)

   Represents a ``del`` statement. ``targets`` is a list of nodes.

.. class:: Pass()

   A ``pass`` statement.

Other statements which are only applicable inside functions or loops are
described in other sections.

Imports
~~~~~~~

.. class:: Import(names)

   An import statement. ``names`` is a list of :class:`alias` nodes.

.. class:: ImportFrom(module, names, level)

   Represents``from x import y``. ``module`` is a raw string of the 'from' name,
   without any leading dots. ``level`` is an integer holding the level of the
   relative import (0 means absolute import).

.. class:: alias(name, asname)

   Both parameters are raw strings of the names. ``asname`` can be ``None`` if
   the regular name is to be used.

::

    >>> parseprint("from ..foo.bar import a as b, c as d")
    Module(body=[
        ImportFrom(module='foo.bar', names=[
            alias(name='a', asname='b'),
            alias(name='c', asname='d'),
          ], level=2),
      ])

   
