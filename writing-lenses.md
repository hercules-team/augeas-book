# Writing lenses

Tutorial with some stuff from http://augeas.net/docs/lenses.html
and http://augeas.net/docs/language.html
and http://augeas.net/docs/writing-schemas.html

[//]: # (was http://augeas.net/docs/lenses.html)

## Lenses

*Lenses* [1]_ are the basic building blocks for establishing the mapping
from files into the Augeas tree and back. You can think of a lens as a
record of three functions *get*, *put* and *create*, where the *get*
function takes the contents of a text file, parses it and produces part of
Augeas' tree. The *put* and *create* functions take a tree and transforms
it back into a text file. The difference is that *put* is used when the
part of the tree being transformed into the file corresponds to something
in the input file, and the *create* function is used when it does not.

Structuring the transformation this way has some important consequences:

1. There is no need to think about the text -> tree direction and the tree
   -> text direction separately, and it therefore becomes impossible for
   those two transformations to become out of sync.
2. Usually, it is enough to focus on the text -> tree transformation when
   describing a config file format; the tree -> text direction comes
   (almost) for free
3. Augeas expects the tree to have a certain structure, implicitly defined
   by the lens for each file. If the tree does not have that structure, for
   example, because there is no ``canonical`` entry for a host in
   ``/files/etc/hosts``, Augeas will refuse to transform such a tree back
   into the corresponding file.

Lenses come in two flavors: *lens primitives* and *lens combinators*. The
former take some piece of the file and process it in some way, the latter
combine smaller lenses to form one large lens.

Strings and regular expressions
-------------------------------

Two basic types in the schema language are strings and regular
expressions. String literals are enclosed in double quotes, and can contain
escape sequences, similar to the escape sequences in C strings. In
particular, the sequences ``\n``,  and ``\t`` are translated to the
new line and tab characters respectively. Therefore, the string literal
``"\t\n"`` denotes a two character string consisting of a tab and a newline
character.

Regular expression literals are enclosed in slashes (``/.../``). The regular
expression syntax is that of extended POSIX regular expressions, with the
small difference that ``.`` does *not* match newlines. To put a literal
``/`` into a regular expression, escape it as ``\/``. The same escape
sequences as for string literals are recognized, so that the regular
expresion literal ``/(\t|\n)/`` matches either a tab or a newline
character.

In a context where a regular expression is expected, but a string is
provided, the string is automatically coerced to a regular expression
matching that string exactly. Characters that have a special meaning inside
regular expressions are escaped, so that the string ``"()"`` is coerced to
the regular expression ``/\(\)/``.

Lens primitives
----------------

The following lens primitives are built into Augeas; the arguments to them
are either regular expressions, indicated by ``RE``, strings indicated by
``STR`` or another lens, indicated by ``LENS``.

``del RE STR``
  Match the regular expression ``RE`` in the *get* direction. The result
  of the match does not appear anywhere in the tree, but Augeas remembers
  the exact value it saw during parsing and restores it in the *put*
  direction. The *create* function produces the default ``STR`` value.
``store RE``
  Store whatever matches the regular expression ``RE`` as the value of the
  enclosing subtree.
``value STR``
  Store the string ``STR`` as the value of the enclosing tree node.
``counter STR``
  Declare a new counter with name ``STR`` and reset its value to
  1. Counters don't need to be declared, but using this statement makes it
  possible to reset counters.
``seq STR``
  Take the next value from the counter with name ``STR`` and use it as the
  label of the enclosing subtree.
``key RE``
  Match the reguler expression ``RE`` against the current position in the
  input, and use the result of the match as the label of the enclosing
  subtree.
``label STR``
  Use the string ``STR`` as the label of the enclosing subtree.

Lens combinators
----------------

In the following, ``L``, ``L1`` and ``L2`` are lenses.

Concatenation: ``L1 . L2``
  The ``.`` operator concatenates two lenses ``L1 . L2``, by applying
  ``L1`` and then ``L2``. Concatenation requires that any string (for the
  *get* direction) or any tree (for the *put*/*create* direction) that
  matches ``L1 . L2`` can be split in exactly one way into one part matching
  ``L1`` and one part matching ``L2``.
Union: ``L1 | L2``
  The union of two lenses ``L1 | L2`` is formed with the ``|`` operator. It
  determines whether ``L1`` or ``L2`` apply at the current position, and
  uses the one that does. The two lenses must be such that only one can
  ever apply to a string. For example, the union ``del /[a-z]+/ | store
  /[a-zA-Z]+/`` is not permissible since the string ``abc`` can be processed
  with either the ``del`` or the ``store``. The typechecker will report an
  error when it encounters such a lens.
Repetition: ``L*``, ``L+``, ``L?``
  A lens ``L`` can be repeated by using the postfix operators ``*``, ``+``,
  and ``?``, with the same meaning as for regular expressions. For example,
  ``(L)+`` matches one or more occurences of ``L``.
Subtree: ``[ L ]``
  For a lens ``L``, the subtree lens ``[ L ]`` constructs a new tree
  node. The label of the node is determined by a ``key``, ``label`` or
  ``seq`` lens inside ``L``, and the value by a ``store`` lens inside
  ``L``; either of these may be missing in which case the label or value
  are set to ``NULL``. The children of the new node are the tree(s)
  constructed by ``L``. ``L`` can contain at most one ``key``, ``label`` or
  ``seq`` lens and at most one ``store`` lens. The typechecker will reject
  violations.
Square: ``square LEFT BODY RIGHT``
  Where ``LEFT``, ``BODY`` and ``RIGHT`` are three lenses, and ``LEFT``
  and ``RIGHT`` accept the same language, and captured strings by ``LEFT``
  and ``RIGHT`` must match.
  This lens makes it possible to process (generalized) squares,
  words of the form `uvu`; a matching square lens would transform `uvu`
  into a tree with a root labelled with `u` and children produced
  by processing `v` with ``BODY``.
  This lens makes it possible to process SGML-style languages
  like XML (when ``LEFT`` is of the form ``key RE`` and ``RIGHT``
  is of the form ``del RE STR``) in a very general fashion;
  in particular, it is possible to
  write lenses that accept an infinite number of different tags.
  It can also used to parse symetric patterns such as quotes
  (see the ``Quote`` module) when both ``LEFT`` and ``RIGHT`` are
  of the form ``del RE STR`` (they can actually be identical in
  this case).

.. [1] The term *lens* was coined by `Harmony and Boomerang`_, systems for
   constructing bidirectional maps between trees and between strings,
   respectively.

.. _Harmony and Boomerang: http://alliance.seas.upenn.edu/~harmony/
