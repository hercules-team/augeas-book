# Quick Tour

This tutorial gives a brief overview of using Augeas, and `augtool` in
particular. It's highly recommended that you follow along on your own; to
do so, first [download and install](/download.html) Augeas. Then, create a
sandbox so that you can safely modify files without affecting your
system. The commands below create the sandbox from `/etc` on your system,
but you might want to copy the files from
[tests/root](https://github.com/hercules-team/augeas/tree/master/tests/root)
in the source tarball &mdash; all the examples below were run against those
files.

```bash
  export AUGEAS_ROOT=/tmp/augeas-sandbox
  mkdir $AUGEAS_ROOT
  sudo cp -pr /etc $AUGEAS_ROOT
  sudo chown -R $(id -nu):$(id -ng) $AUGEAS_ROOT
  augtool -b
```

The `-b` option tells `augtool` to preserve the original file with an
extension of `.augsave` whenever it makes a change.

In the `augtool` shell, type `help` to get a list of commands. The
`print` command is particularly useful to explore what data is present in
the tree. The commands to run can either be typed interactively into
`augtool` (with TAB completion on commands and tree paths), or piped in
from standard input.

## Adding an entry to `/etc/hosts`

To warm up, let's do something very simple: adding a new entry to
`/etc/hosts`. In the `augtool` shell, type the following commands:

```
  augtool> set /files/etc/hosts/01/ipaddr 192.168.0.1
  augtool> set /files/etc/hosts/01/canonical pigiron.example.com
  augtool> set /files/etc/hosts/01/alias[1] pigiron
  augtool> set /files/etc/hosts/01/alias[2] piggy
  augtool> save
```

The four `set` commands create four nodes underneath
`/files/etc/hosts/01`, and assign them the values passed as the second
argument. The data parsed from a file with full path `FILE` in the
filesystem is stored in the tree underneath `/files/FILE`; with that the
`set` commands manipulate an entry in the file `/etc/hosts`. A second
hierarchy underneath `/augeas/files/FILE` contains metadata about the
file, such as an indication of any errors encountered when the file was
read.

The tree for `/etc/hosts` puts each host entry into its own subtree,
numbered sequentially from 1, so that the first host entry appears under
`/files/etc/hosts/1`, the second under `/files/etc/hosts/2`, etc. We
use the label `01` and put the details of our new host entry underneath
`/files/etc/hosts/01`. This works because reading in a `/etc/hosts`
file will never use labels starting with 0, and tree labels are strings,
their numeric value is irrelevant. The order in which host entries are
written back to file is determined by the order in which they appear in the
tree.

The `set` command creates non-existant nodes as needed. There is a second
command, `ins`, to create new nodes in the tree that provides more
control over where exactly a new node shows up, in particular where in the
list of its siblings it appears. Because files are inherently sequential,
the order of sibling nodes in the tree matters.

The notation `alias[1]` and `alias[2]` tells Augeas to set the value
for the first and second child of `/files/etc/hosts/01`. Both nodes
are called `/files/etc/hosts/01/alias`; the new host entry can be
listed with

```bash
  augtool> ls /files/etc/hosts/01
```

or searched for with

```bash
  augtool> match /files/etc/hosts/*/ipaddr 192.168.0.1
```

which prints `/files/etc/hosts/01/ipaddr`.

Running `(cd ${AUGEAS_ROOT} && diff -u ./etc/hosts ./etc/hosts.augsave)`
confirms that `/etc/hosts` now has one additional entry at the end:

```bash
  diff -up ./etc/hosts.augsave ./etc/hosts
  --- ./etc/hosts.augsave
  +++ ./etc/hosts
  @@ -4,3 +4,4 @@
   #172.31.122.254   granny.watzmann.net granny puppet
   #172.31.122.1     galia.watzmann.net galia
   172.31.122.14   orange.watzmann.net orange
  +192.168.0.1    pigiron.example.com pigiron piggy
```

Changing `/etc/grub.conf`
---------------------------

Modifying a slightly more complex file, `/etc/grub.conf` [#grub_conf]_,
follows the same lines as modifying `/etc/hosts`. One of the big
advantages of Augeas is that configuration data is manipulated in the same
way, regardless of how it is stored in the native config files.

While the above change to `/etc/hosts` can be easily achieved with
standard shell tools (`cat` comes to mind), and `/etc/hosts` in general
can be manipulated relatively easily with `sed`, `grep`, `awk`, or
tools readily available in most scripting languages, manipulating
`/etc/grub.conf` with those tools is quite a bit harder.

We want to change `/etc/grub.conf` so that `grub` boots the second boot
entry by default, and remove the third boot record completely. The
following two commands do that &mdash; you should look at the tree before and
after the change using `print /files/etc/grub.conf`:

```bash
  augtool> set /files/etc/grub.conf/default 1
  augtool> rm /files/etc/grub.conf/title[3]
  augtool> save
```

The diff between the old and new file confirms that Augeas changed what was
intended:

```bash
  diff -up ./etc/grub.conf.augsave ./etc/grub.conf
  --- ./etc/grub.conf.augsave     2008-04-09 16:14:06.000000000 -0700
  +++ ./etc/grub.conf     2008-04-15 15:32:09.000000000 -0700
  @@ -7,7 +7,7 @@
   #          kernel /vmlinuz-version ro root=/dev/vg00/lv00
   #          initrd /initrd-version.img
   #boot=/dev/sda
  -default=0
  +default=1
   timeout=5
   splashimage=(hd0,0)/grub/splash.xpm.gz
   hiddenmenu
  @@ -19,11 +19,6 @@ title Fedora (2.6.24.3-50.fc8)
          root (hd0,0)
          kernel /vmlinuz-2.6.24.3-50.fc8 ro root=/dev/vg00/lv00
          initrd /initrd-2.6.24.3-50.fc8.img
  -title Fedora (2.6.21.7-3.fc8xen)
  -       root (hd0,0)
  -       kernel /xen.gz-2.6.21.7-3.fc8
  -       module /vmlinuz-2.6.21.7-3.fc8xen ro root=/dev/vg00/lv00
  -       module /initrd-2.6.21.7-3.fc8xen.img
   title Fedora (2.6.24.3-34.fc8)
          root (hd0,0)
          kernel /vmlinuz-2.6.24.3-34.fc8 ro root=/dev/vg00/lv00
```

## Making `sshd` accept an additional environment variable

The config file for `sshd` is deceptively simple, but has some subtle
intricacies. One of them are `Match` blocks at the end of the file,
another is the fact that some settings can be repeated in the file, and
values are accumulated, and that their values are best viewed as arrays.

To illustrate this, we will add a new environment variable `FOO` to the
`AcceptEnv` setting in `/etc/ssh/sshd_config`. The schema that Augeas
uses for `sshd_config` [#sshd_config]_ by default maps multiple
`AcceptEnv` lines into multiple tree nodes with the name `AcceptEnv`,
similar to how multiple aliases in `/etc/hosts` are handled. Since each
line contains a list of environment variables, the schema splits that list
into separate tree nodes numbered starting at one.  The following lines in
`sshd_config`

```
  AcceptEnv LANG LC_CTYPE
  AcceptEnv LC_IDENTIFICATION LC_ALL
```

are mapped into a tree

```
  AcceptEnv[1]/1 = "LANG"
  AcceptEnv[1]/2 = "LC_CTYPE"
  AcceptEnv[2]/1 = "LC_IDENTIFICATION"
  AcceptEnv[2]/2 = "LC_ALL"
```

This allows for very fine-grained access to individual value in the
`sshd_config` file, and preserves enough information so that Augeas can
decide which entries go on the same line. To see all `AcceptEnv` entries,
run

```
  augtool> print /files/etc/ssh/sshd_config/AcceptEnv/*
```

To add a new variable `FOO` at the end of the last `AcceptEnv` line, we
perform

```
  augtool> set /files/etc/ssh/sshd_config/AcceptEnv[last()]/01 FOO
  augtool> save
```

The addition of `[last()]` to `AcceptEnv` in the path tells Augeas that
we are talking about the last node named `AcceptEnv`. Augeas requires
that for a `set` the path expression corresponds either to an existing
node, or to no node at all (in which case a new node is created). Writing
the above command as

```
  augtool> set /files/etc/ssh/sshd_config/AcceptEnv/01 FOO
```

results in an error when there are multiple `AcceptEnv` entries in the
tree. In that case, Augeas can not figure out under which of them you want
to put the entry labelled `01`.

Again, the above commands lead to the expected diff, the addition of
`FOO` to the last `AcceptEnv` line:

```bash
  diff -up ./etc/ssh/sshd_config.augsave ./etc/ssh/sshd_config
  --- ./etc/ssh/sshd_config.augsave
  +++ ./etc/ssh/sshd_config
  @@ -93,7 +93,7 @@ UsePAM yes
   # Accept locale-related environment variables
   AcceptEnv LANG LC_CTYPE LC_NUMERIC LC_TIME LC_COLLATE LC_MONETARY LC_MESSAGES
   AcceptEnv LC_PAPER LC_NAME LC_ADDRESS LC_TELEPHONE LC_MEASUREMENT
  -AcceptEnv LC_IDENTIFICATION LC_ALL
  +AcceptEnv LC_IDENTIFICATION LC_ALL FOO
   #AllowTcpForwarding yes
   #GatewayPorts no
   #X11Forwarding no
```

.. [#grub_conf] some distributions do not have a `/etc/grub.conf`, but
   only a `/boot/grub/menu.lst` or similar. The simplest way to follow
   these examples is to create a symbolic link
   `$AUGEAS_ROOT/etc/grub.conf` that points to the real grub config in
   your sandbox. In real life, you would change paths like
   `/files/etc/grub.conf/...` to `/files/boot/grub/menu.lst/...` when
   interacting with Augeas
.. [#sshd_config] all schemas can be found in the `lenses/` subdirectory
   in the source tarball, which by default is installed into
   `/usr/share/augeas/lenses`
