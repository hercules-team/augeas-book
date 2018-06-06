# Lens language reference

Reference for the core language + `builtin.c`

Stuff from http://augeas.net/docs/builtins.html
and http://augeas.net/docs/language.html

[//]: # (was http://augeas.net/docs/builtins.html)

## Builtin Functions

Augeas comes with a number of builtin functions. They can be used anywhere,
without qualifying them with a module name.

Lens functions
--------------

These functions construct lenses. They are described on the `lenses page`_

Tree manipulation
-----------------

These functions are mostly useful for making changes to a tree during a
`put test`_

``set PATH VALUE TREE -> TREE``
  Set the value of the node ``PATH`` to ``VALUE`` in the input tree
  ``TREE``, and return the modified tree. Both ``PATH`` and ``VALUE`` must
  be strings, and ``PATH`` must be a path expression that matches one
  node.
``clear PATH TREE -> TREE``
  Set the value of the node ``PATH`` to ``NULL`` in the input tree
  ``TREE``, and return the modified tree. ``PATH`` must be a string,
  denoting a path expression that matches on node.
``rm PATH TREE -> TREE``
  Remove the subtree at ``PATH`` in the input tree ``TREE`` and return the
  resulting tree. ``PATH`` must be a string, denoting a path expression.
``insa LABEL PATH TREE -> TREE``
  Insert a new tree node with label ``LABEL`` into the tree. Inserts the
  new node after the existing node at ``PATH`` in the
  input tree ``TREE`` and returns the modified tree. ``LABEL`` must be a
  string not containing ``/``, and ``PATH`` must be a string containing a
  path expression that matches exactly one node in ``TREE``.
``insb LABEL PATH TREE -> TREE``
  Same as ``insa``, but inserts the new node before the existing node at
  ``PATH``.

Filters and transforms
----------------------

Filters can be built up from simpler filters through concatenation, for
example, ``(incl "/dir/*") . (excl "/dir/*.bak")`` lists all files in
``/dir`` that do not have the extension ``.bak``. A file is included by a
filter if it matches at least one ``incl`` filter and no ``excl`` filter.

``incl GLOB -> FILTER``
  Produce a new filter that includes files that match the shell glob
  ``GLOB``. ``GLOB`` must be a string.
``excl GLOB -> FILTER``
  Produce a new filter that excludes files that match the shell glob
  ``GLOB``. ``GLOB`` must be a string.
``transform LENS FILTER -> TRANSFORM``
  Create a new transform that applies the lens ``LENS`` to all files that
  are included by the filter ``FILTER``.

Printing
--------

Printing functions are useful for debugging lenses. They are generally used
with an anonymous let, for example ``let _ = print_endline "Hello,
World"``.

``print_string STRING -> UNIT``
  Print the string ``STRING`` and return nothing.
``print_regexp RE -> UNIT``
  Print the regular expression ``RE`` and return nothing.
``print_endline STR -> UNIT``
  Print the string ``STR`` followed by a newline, and return nothing.
``print_tree TREE --> TREE``
  Print the tree ``TREE`` and return it. This function is mostly useful in
  the list of commands in ``put`` unit tests, since it makes it possible to
  look at the tree the test is operating on.

Lens inspection
---------------

These functions pick apart a lens, and are sometimes useful in debugging.

``lens_ctype LNS -> REGEXP``
  Return the *concrete* type of a lens, the regular expression that is
  matched against an input string to produce the tree.
``lens_atype LNS -> REGEXP``
  Return the *abstract* type of a lens, the regular expression that is
  matched against one level in the tree to turn the tree back into a
  string.
``lens_ktype LNS -> REGEXP``
  Return the regular expression that matches the key/label of the tree node
  produced by the lens.
``lens_vtype LNS -> REGEXP``
  Return the regular expression that matches the value of the tree node
  produced by the lens.
``lens_format_atype LNS -> STRING``
  Return a human-readable representation of the *abstract* type of the
  lens.

Tree transformation
-------------------

These functions are not all that useful by themselves, they are usually
used in `unit tests`_.

``get LENS STRING -> TREE``
  Apply the get direction of lens ``LENS`` to the string ``STRING`` and
  return the resulting tree.
``put LENS TREE STRING -> STRING``
  Apply the put direction of lens ``LENS`` to the tree ``TREE``, assuming
  it was initially generated from string ``STRING`` with a ``get``, and
  return the resulting string.

Regular expression functions
----------------------------

``regexp_match REGEXP STRING -> STRING``
  Match ``REGEXP`` against ``STRING`` and return the resulting match. If
  ``REGEXP`` does not match at all, return the empty string.

System functions
----------------

These functions are in the ``Sys`` module, and therefore need to be
prefixed with ``Sys.`` when they are used

``getenv VAR -> STRING``
  Return the value of the environment variable ``VAR``, or the empty string
  if ``VAR`` is not defined. ``VAR`` must be a string.
``read_file FILE -> STRING``
  Return the contents of file ``FILE`` as a string. ``FILE`` must be a
  string containing the absolute path to a file.

.. _lenses page: lenses.html
.. _put test: language.html#unit-tests
.. _unit tests: language.html#unit-tests
