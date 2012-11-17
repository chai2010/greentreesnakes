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

.. class:: UAdd
           USub
           Not
           Invert

   Unary operator tokens. :class:`Not` is the ``not`` keyword, :class:`Invert`
   is the ``~`` operator.

.. class:: BinOp(left, op, right)

   A binary operation (like addition or division). ``op`` is the operator, and
   ``left`` and ``right`` are any expression nodes.

.. class:: Add
           Sub
           Mult
           Div
           FloorDiv
           Mod
           Pow
           LShift
           RShift
           BitOr
           BitXor
           BitAnd

   Binary operator tokens.

.. class:: BoolOp(op, values)

   A boolean operation, 'or' or 'and'. ``op`` is :class:`Or` or
   :class:`And`. ``values`` are the values involved. Consecutive operations
   with the same operator, such as ``a or b or c``, are collapsed into one node
   with several values.
   
   This doesn't include ``not``, which is a :class:`UnaryOp`.

.. class:: And
           Or

   Boolean operator tokens.

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

.. class:: Eq
           NotEq
           Lt
           LtE
           Gt
           GtE
           Is
           IsNot
           In
           NotIn

   Comparison operator tokens.

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

.. class:: keyword(arg, value)
   
   A keyword argument to a function call or class definition. ``arg`` is a raw
   string of the parameter name, ``value`` is a node to pass in.

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

.. class AugAssign(target, op, value)

   Augmented assignment, such as ``a += 1``. In that example, ``target`` is a
   :class:`Name` node for ``a``, op is :class:`Add`, and ``value`` is a
   :class:`Num` node for 1.

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

Control flow
------------

.. note::
   Optional clauses such as ``else`` are stored as an empty list if they're
   not present.

.. class:: If(test, body, orelse)

   An ``if`` statement. ``test`` holds a single node, such as a :class:`Compare`
   node. ``body`` and ``orelse`` each hold a list of nodes.
   
   ``elif`` clauses don't have a special representation in the AST, but rather
   appear as extra :class:`If` nodes within the ``orelse`` section of the
   previous one.

.. class:: For(target, iter, body, orelse)

   A ``for`` loop. ``target`` holds the variable(s) the loop assigns to, as a
   single :class:`Name`, :class:`Tuple` or :class:`List` node. ``iter`` holds
   the item to be looped over, again as a single node. ``body`` and ``orelse``
   contain lists of nodes to execute. Those in ``orelse`` are executed if the
   loop finishes normally, rather than via a ``break`` statement.

.. class:: While(test, body, orelse)

   A ``while`` loop. ``test`` holds the condition, such as a :class:`Compare`
   node.

.. class:: Break
           Continue

   The ``break`` and ``continue`` statements.

.. class:: TryFinally(body, finalbody)
           TryExcept(body, handlers, orelse)
   
   ``try`` blocks. All attributes are list of nodes to execute, except for
   ``handlers``, which is a list of :class:`ExceptHandler` nodes.
   
   If a ``try`` block has both ``except`` and ``finally`` clauses, it is parsed
   as a :class:`TryFinally`, with the body containing a :class:`TryExcept`.

.. class:: ExceptHandler(type, name, body)

   A single ``except`` clause. ``type`` is the exception type it will match,
   typically a :class:`Name` node. ``name`` is a raw string for the name to hold
   the exception, or None if the clause doesn't have ``as foo``. ``body`` is
   a list of nodes.

.. class:: With(context_expr, optional_vars, body)

   A ``with`` block. ``context_expr`` is the context manager, often a
   :class:`Call` node. ``optional_vars`` is a :class:`Name`, :class:`Tuple` or
   :class:`List` for the ``as foo`` part, or ``None`` if that isn't used.

Function and class definitions
------------------------------

.. class:: FunctionDef(name, args, body, decorator_list, returns)

   A function definition. 
   
   * ``name`` is a raw string of the function name.
   * ``args`` is a :class:`arguments` node.
   * ``body`` is the list of nodes inside the function.
   * ``decorator_list`` is the list of decorators to be applied, stored outermost
     first (i.e. the first in the list will be applied last).
   * ``returns`` is the return annotation (Python 3 only).

.. class:: Lambda(args, body)

   ``lambda`` is a minimal function definition that can be used inside an
   expression. Unlike :class:`FunctionDef`, ``body`` holds a single node.

.. class:: arguments(args, vararg, varargannotation, kwonlyargs, kwarg, \
                     kwargannotation, defaults, kw_defaults)
   
   The arguments for a function. In **Python 3**:
   
   * ``args`` and ``kwonlyargs`` are lists of :class:`arg` nodes.
   * ``vararg`` and ``kwarg`` are raw strings referring to the ``*args, **kwargs``
     parameters.
   * ``varargannotation`` and ``kwargannotation`` - see :class:`arg`
   * ``defaults`` is a list of default values for arguments that can be passed
     positionally. If there are fewer defaults, they correspond to the last n
     arguments.
   * ``kw_defaults`` is a list of default values for keyword-only arguments. If
     one is ``None``, the corresponding argument is required.

   In **Python 2**, the attributes for annotations and keyword-only arguments
   are not needed.

.. class:: arg(arg, annotation)

   A single argument in a list; Python 3 only. ``arg`` is a raw string of the
   argument name, ``annotation`` is its annotation, such as a :class:`Str` or
   :class:`Name` node.
   
   In Python 2, arguments are instead represented as :class:`Name` nodes, with
   ``ctx=Param()``.

::

    In [52]: %%dump_ast
       ....: @dec1
       ....: @dec2
       ....: def f(a: 'annotation', b=1, c=2, *d, e, f=3, **g) -> 'return annotation':
       ....:   pass
       ....: 
    Module(body=[
        FunctionDef(name='f', args=arguments(args=[
            arg(arg='a', annotation=Str(s='annotation')),
            arg(arg='b', annotation=None),
            arg(arg='c', annotation=None),
          ], vararg='d', varargannotation=None, kwonlyargs=[
            arg(arg='e', annotation=None),
            arg(arg='f', annotation=None),
          ], kwarg='g', kwargannotation=None, defaults=[
            Num(n=1),
            Num(n=2),
          ], kw_defaults=[
            None,
            Num(n=3),
          ]), body=[
            Pass(),
          ], decorator_list=[
            Name(id='dec1', ctx=Load()),
            Name(id='dec2', ctx=Load()),
          ], returns=Str(s='return annotation')),
      ])

.. class:: Return(value)

   A ``return`` statement.

.. class:: Yield(value)

   A ``yield`` expression. Because ``yield`` is now an expression, it must be
   wrapped in a :class:`Expr` node if the value sent back is not used.

.. class:: Global(names)
           Nonlocal(names)

   ``global`` and ``nonlocal`` statements. ``names`` is a list of raw strings.

.. class:: ClassDef(name, bases, keywords, starargs, kwargs, body, decorator_list)

   A class definition.
   
   * ``name`` is a raw string for the class name
   * ``bases`` is a list of nodes for explicitly specified base classes.
   * ``keywords`` is a list of :class:`keyword` nodes, principally for 'metaclass'.
     Other keywords will be passed to the metaclass, as per `PEP-3115
     <http://www.python.org/dev/peps/pep-3115/>`_.
   * ``starargs`` and ``kwargs`` are each a single node, as in a function call.
     starargs will be expanded to join the list of base classes, and kwargs will
     be passed to the metaclass.
   * ``body`` is a list of nodes representing the code within the class
     definition.
   * ``decorator_list`` is a list of nodes, as in :class:`FunctionDef`.

::

    In [59]: %%dump_ast
       ....: @dec1
       ....: @dec2
       ....: class foo(base1, base2, metaclass=meta):
       ....:   pass
       ....: 
    Module(body=[
        ClassDef(name='foo', bases=[
            Name(id='base1', ctx=Load()),
            Name(id='base2', ctx=Load()),
          ], keywords=[
            keyword(arg='metaclass', value=Name(id='meta', ctx=Load())),
          ], starargs=None, kwargs=None, body=[
            Pass(),
          ], decorator_list=[
            Name(id='dec1', ctx=Load()),
            Name(id='dec2', ctx=Load()),
          ]),
      ])
