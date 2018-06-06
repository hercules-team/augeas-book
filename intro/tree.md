# The tree

The tree that Augeas maintains is very similar to the tree of files and
directories in a filesystem: the components of a path are strings that can,
and nodes have an optional value associated with them, which can be an
arbitrary string.

Augeas' tree deviates from the files-in-a-filesystem model in one important
way: multiple children in a tree can have the same label. For example, the
schema for ``/etc/hosts`` uses that to map multiple aliases for the same
host to multiple nodes, all with the path ``/files/etc/hosts/N/alias``,
where ``N`` is the number of the host entry in the file. All numbering in
Augeas starts with one, not zero.

With that, a tree node consists of three pieces of data:

a *label*
  a string that is part of the path of the node. The string can contain any
  characters except for ``/``, ``*``, or ``[``. Strictly speaking, nodes
  can also have ``NULL`` as a label, but such nodes are not accessible
  through the public API |---| they are sometimes used by the lenses
  processing the files to store important formatting information.
a *value*
  an arbitrary string or ``NULL``
the *children*
  a list of tree nodes, where multiple children can have the same
  label. Since files are inherently sequential, ordering in the list of
  children matters.

The tree can be searched in all kinds of ways by using `path expressions`_

.. _path expressions: /page/Path_expressions

Important paths
---------------

The Augeas tree consist of two separate hierarchies: ``/augeas`` used for
reporting important metadata, in particular information about how files
were processed, and ``/files`` containing the result of parsing
configuration files.

Configuration data in ``/files``
****************************************

By convention, all configuration files get parsed into the ``/files``
hierarchy. The contents of a file that has the absolute path ``/PATH`` in
the file system appears at ``/files/PATH`` in the tree. For example, the
contents of ``/etc/hosts`` appear at ``/files/etc/hosts``.

Metainformation in ``/augeas``
******************************

The ``/augeas`` hierarchy reports important metadata about Augeas' inner
workings. Most entries under ``/augeas`` are readonly |---| it is possible
to change them by issuing ``set`` commands, but those changes have no
influence on how Augeas goes about its business.

The entries ``/augeas/root`` and ``/augeas/save`` report the file system
root from which files are read, and how files are saved. The save modes,
corresponding to starting ``augtool`` with the ``-n``, the ``-b``, or
neither of these options are ``newfile``, ``backup``, and
``overwrite``. Changing ``/augeas/save`` before saving the tree changes how
updated files are saved.

For each file ``/PATH`` that was processed, some metadata is presented in
``/augeas/files/PATH``. That information is due to be revised soon; the
entries mentioned here will remain, but other entries may be removed.

Underneath ``/augeas/files/PATH``, the entry ``error`` has no value if
processing of the file was successful. Values that indicate errors are

``read_failed``
  Reading the file failed; this means that the file exists, but can not be
  read, for example, because of insufficient permissions
``parse_failed``
  Parsing the file into the tree failed. This usually means that the file
  does not match the lens that was applied to it.
``put_read``
  Reading the file before saving changes of it failed, for similar reasons
  as ``read_failed`` indicates. Before Augeas writes a changed file, it
  has to read the original file back in to extract comments etc.
``rename_augsave``
  Renaming the original file to one ending with ``.augsave`` failed; this
  can only happen when the save mode is ``backup``
``rename_augnew``
  Renaming the new file with extension ``.augnew`` to the original file
  name failed; this can only happen when the save mode is ``backup``

The entry ``lens/info`` underneath ``/augeas/files/PATH`` indicates the lens
that is used to process the file. The value contains the path to the
``.aug`` schema definition, and the line and column number where that lens
was defined. For example, ``/augeas/files/etc/hosts/lens/info`` might
have the value ``/usr/share/augeas/lenses/hosts.aug:18.12-.34``.

Features under ``/augeas/version``
..................................

The node ``/augeas/version`` contains the version number of the Augeas
library. Underneath it are a few entries that indicate a few features of
the API:

``save``
  The behavior of ``aug_save`` can be changed dynamically at runtime by
  changing the entry ``/augeas/save``. The possible values that can be
  assigned to ``/augeas/save`` are listed in ``/augeas/version/save/mode``
``defvar``
  Variables can be defined for later use in path expressions
``defvar/expr``
  The expression used to define a variable ``var`` is stored in
  ``/augeas/variables/var``

.. _XPath: http://www.w3.org/TR/xpath
.. |---| unicode:: U+2014  .. em dash
