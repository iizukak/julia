.. currentmodule:: Base

.. _devdocs-meta:

Talking to the compiler (the ``:meta`` mechanism)
=================================================

In some circumstances, one might wish to provide hints or instructions
that a given block of code has special properties: you might always
want to inline it, or you might want to turn on special compiler
optimization passes.  Starting with version 0.4, julia has a
convention that these instructions can be placed inside a ``:meta``
expression, which must be the first expression in the body of a
function.

``:meta`` expressions are created with macros. As an example, consider
the implementation of the ``@inline`` macro::

    macro inline(ex)
        esc(_inline(ex))
    end
    
    _inline(ex::Expr) = pushmeta!(ex, :inline)
    _inline(arg) = arg

Here, ``ex`` is expected to be an expression defining a function.
A statement like this::

    @inline function myfunction(x)
        x*(x+3)
    end

gets turned into an expression like this::

    quote
        function myfunction(x)
            Expr(:meta, :inline)
            x*(x+3)
        end
    end

``pushmeta!(ex, :symbol)`` appends ``:symbol`` to the end of the
``:meta`` expression, creating a new ``:meta`` expression if
necessary.

To use the metadata, you have to parse these ``:meta`` expressions.
If your implementation can be performed within Julia, ``popmeta!`` is
very handy: ``popmeta!(body, :symbol)`` will scan a function *body*
expression (one without the function signature) for a ``:meta``
expression; if ``:symbol`` is present, it will return ``true`` and
remove ``:symbol`` from the arguments of the ``:meta`` expression
(deleting the expression altogether if there are no more arguments).

Not yet provided is a convenient infrastructure for parsing ``:meta``
expressions from C++.
