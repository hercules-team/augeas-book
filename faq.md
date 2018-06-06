# Frequently Asked Questions

[//]: # (was http://augeas.net/faq.html)

## Basic Questions

###### What license is Augeas distributed under ?

  Augeas is distributed under the GNU Lesser General Public License
  ([LGPL](http://www.gnu.org/licenses/old-licenses/lgpl-2.1.html)) version
  2.1. A copy of the license is included in the file `COPYING` in the
  source distribution.

###### Can I use Augeas from a proprietary application ?
  Yes. The LGPL allows you to use Augeas from a proprietary application. It
  would be very helpful to both you and the community if you send bug fixes
  and improvements as patches for possible incorporation in the main
  development tree. It will decrease your maintenance costs anyway if you
  do so.

###### How do I get Augeas ?

  The [download](download.html) page explains where you can find released
  RPM's and tarballs and how you can check the latest development version
  out from Mercurial

###### What's with that name ?
  To quote [Wikipedia](http://en.wikipedia.org/wiki/Augeas): "*In Greek
  mythology, Augeas ... was King of Elis ... He is best known ... for his
  stables, which housed the single greatest number of cattle in the country
  and had never been cleaned.*"

###### How do I contribute to Augeas ?
  The main means of communication is the
  [Augeas mailing list](https://www.redhat.com/mailman/listinfo/augeas-devel)
  `augeas-devel@redhat.com`. For more information on contributing, see
  [the contribute page](developers.html)

###### I don't know any C &mdash; can I still help ?
  Of course. Besides writing code, there are
  [many more areas](developers.html) where Augeas needs help: evaluating
  existing tree representations of configuration files, writing
  descriptions for new file formats, and documenting Augeas' tree structure
  are all areas where help is sorely needed.

###### Can I use Augeas for other text files ?
  Yes, there is actually nothing specific to configuration files in
  Augeas. If you have some other unstructured text file that you'd like to
  transform into a tree, all you need to do is write a file format
  description for it.

###### Can I change a file with Augeas and still edit it by other means ?
  Yes. One of the main design goals of Augeas is to not interfere with any
  other ways in which you might change your configuration files. Augeas
  works very hard to preserve comments and formatting details such as the
  number of spaces used to separate fields etc. It also does not depend on
  any additional information embedded in comments &mdash; you can always
  alternate between changing a configuration file with your favorite editor
  or custom script, and changing it with Augeas.

## Using Augeas

###### Can I use Augeas in a shell script ?
  Yes, the `augtool` command can be used to manipulate the tree and the
  underlying files with shell scripts. Run `augtool help` for a list of
  commands `augtool` supports.

###### What language bindings are available ?

  Bindings for Ruby, Python, OCaml, Perl, Haskell, PHP, Lua, Java, Node.js,
  Go and Tcl are available for [download](download.html). Patches/additions
  for other languages are eagerly anticipated.

###### Do I need to know how file formats are described ?
  No, that is only necessary if you want to change how files are mapped
  into the tree or map new files. For using Augeas, it is enough to
  understand the functionality provided by `augtool`, or the equivalent
  calls in C or the language bindings.

###### How is the Augeas tree structured ?
  Each node in the Augeas tree consists of three pieces of data: a *label*
  and a *value*, both strings, and a list of *child* nodes. The label and
  value are both strings. The label is used in the path to the node and
  child nodes, much as file names are used in the paths to files in a file
  system. The label must not contain the characters `/` or `*` or end
  with `[N]` where N is a positive number.

###### Can a node have multiple children with the same label ?

  Yes, that is the biggest difference between the Augeas tree and files in
  a filesystem (and makes it similar to DOM's for XML documents). A node
  can have any number of children called `foo`; in paths, a specific child
  can be picked out with `foo[N]` where `N` is a positive number &mdash;
  the first child with label `foo` is picked with `foo[1]`, and the last
  can be picked with `foo[last()]`. Just `foo` in a path picks *all* the
  children with that label. This convention follows
  [XPath](http://www.w3.org/TR/xpath)

###### What kind of data can be stored in the tree ?
  Augeas only stores strings in the tree. So far, there hasn't been a
  compelling reason to support other data types.

###### Where do a find the contents of a file ?
  The contents of a file `/some/path` are mapped into the tree as
  `/files/some/path`, which is usually a fine-grained tree view of that
  file; additional (meta-)information about each file is available in
  `/augeas/files/some/path`.

## Describing file formats

###### What are *lenses* ?
  *Lenses* are the building blocks of the file <-> tree transformation;
  they combine parsing a file and building the tree, the *get*
  transformation, with turning the tree back into an (updated) file, the
  *put* transformation.

###### Why are they called *lenses* ?
  The name comes from the
  [Harmony project](http://alliance.seas.upenn.edu/~harmony/). In Harmony's
  lingo, Augeas maps back and forth between a *concrete view* (the
  configuration file) and an *abstract view* (the tree representation). And
  calling something that goes back and forth between two views a 'lens'
  seems only natural, doesn't it ?

###### What is a *bidirectional language* ?

  A bidirectional language is one where the program expresses a
  transformation from input to output, and from (possibly modified) output
  back to the corresponding input. They are called *bidirectional* rather
  than *bijective* because there are generally many inputs for the same
  output and vice versa. In Augeas case, many input files (e.g.,
  `/etc/hosts` with varying amounts of whitespace between fields) map to
  the same tree, and there are many ways to map the same tree back to a
  file. Other examples of bidirectional languages are `Harmony` (mapping
  trees to trees) and `Boomerang` (mapping strings to strings), both from
  the [Harmony project](http://alliance.seas.upenn.edu/~harmony/).

###### How is a file parsed ?
  The *lenses* match regular expressions, different lenses do different
  things with the strings that match (create a new tree node with a certain
  label, store a value in a tree node, combine tree nodes into a larger
  tree); writing the regular expressions is the main task when describing a
  file format.
###### What is the regular expression syntax ?

  Augeas uses
  [extended POSIX regexps](http://www.regular-expressions.info/posix.html).
  There are a few more esoteric features of extended POSIX regexps that are
  not supported, the most notable of them being POSIX named character
  classes, collating elements and back references. Named character classes
  should be supported soon (patches welcome).

###### What is the language in which file format descriptions are written ?
  The file format descriptions are written in a small subset of ML (a
  popular functional language.) If you don't know ML, don't despair: there
  are only two or three constructs that you need to know to write file
  format descriptions. The Augeas language is statically typed; besides the
  'usual' checks of a type checker, checks also make sure that the
  transformations described by lenses are safe.
###### How do I test my file format description ?
  The Augeas language has a built-in `test` construct for running unit
  tests; additional round-trip tests can be found in the `tests/`
  directory in the source distribution. They are run by the `augtest`
  command in the same directory.
###### Where do I find existing file format descriptions ?
  File format descriptions are contained in files with the extension
  `.aug`. In the source distribution, they are in the `lenses`
  directory, which, by default, is installed into
  `/usr/share/augeas/lenses`.
###### Can I send you my file format description ?
  Yes, please. One of Augeas' goals is to collect file format descriptions,
  and build a library of them, so that eventually there will be a
  'canonical' tree representation of configuration data.

## `libfa` &mdash; Finita Automata

###### What is `libfa` ?

  [Libfa](libfa/) is a library for manipulating finite automata. A finite
  automaton is a representation of all strings matching a regular
  expression. Augeas uses it for typechecking lenses and ensuring they
  don't contain ambiguities.

###### Can I use `libfa` outside of Augeas ?
  Yes. `Libfa` is also licensed under the [LGPL][LGPL], and has no dependencies
  on Augeas.

###### What operations does `libfa` support ?
  A list of operations can be found [here][faops].

###### What regular expression syntax does `libfa` support ?
  It supports extended POSIX regular expressions, exactly the same as Augeas.

###### Why is there no operation to match a regular expression against a string ?

  The GNU C library (and [GNU Regex][gnu-regex]) contains a much better
  matcher than I would ever hope to write for `libfa` &mdash; being able to
  use that matcher is the main reason for using extended POSIX syntax.

[LGPL]: http://www.gnu.org/licenses/old-licenses/lgpl-2.1.html
[faops]: libfa/operations.html
[gnu-regex]: http://www.delorie.com/gnu/docs/regex/regex.html
