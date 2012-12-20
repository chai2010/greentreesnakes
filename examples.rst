Examples of working with ASTs
=============================

Wrapping integers
-----------------

In Python code, ``1/3`` would normally be evaluated to a floating-point number,
that can never be exactly one third. Mathematical software, like `SymPy
<http://sympy.org/>`_ or `Sage <http://www.sagemath.org/>`_, often wants to use
exact fractions instead. One way to make ``1/3`` produce an exact fraction is
to wrap the integer literals ``1`` and ``3`` in a class::

    class IntegerWrapper(ast.NodeTransformer):
        """Wraps all integers in a call to Integer()"""
        def visit_Num(self, node):
            if isinstance(node.n, int):
                return ast.Call(func=ast.Name(id='Integer', ctx=ast.Load()),
                                args=[node], keywords=[])
            return node

    tree = ast.parse("1/3")
    tree = IntegerWrapper().visit(tree)
    # Add lineno & col_offset to the nodes we created
    ast.fix_missing_locations(tree)

    # The tree is now equivalent to Integer(1)/Integer(3)
    # We would also need to define the Integer class and its __truediv__ method.
