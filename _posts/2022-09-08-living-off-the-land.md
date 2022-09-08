---
title: Living off the land.
---


We’re mostly cloud computing infrastructure here at SHU, but we do
have a department server. On which I’m not an admin (and shouldn’t
be!)

For my software engineering class however, students (and I) will just
need access to some software sometimes that isn’t installed. For
instance, the default version of `python` is 2.7.

However! `python3` is already installed! As `python3`. As you’d
expect. To be more specific, version 3.6. So that’ll probably suffice.

However, suppose not. Or, just that we wanted to build python3.10.7
from source.

# How to install python 3.10 on Red Hat Enterprise Linux Server
  release 7.9 (Maipo) without root

There were several missing modules that were not installed but
optional, one that the output of a failed `make` lists as mandatory,
and one that same failure lists as optional but seemed actually
necessary.

The configuration script says it’ll require a more recent version of
openssl than the one we have installed.

So we go `wget` ourselves a tarball of the most recent version of
openssl, and then try and configure _it_.

Well, the version of perl that we have on our system seems broken or
malconfigured, because perl can’t find some basic comes-out-of-the-box
install packages, e.g. `ExtUtils::MakeMaker` and `Test::Harness`. And
we don’t have CPAN installed either.

Those modules are supposedly core---but I cannot seem to find them
even though I have the standard perl5 installed.

So, the suggestion was to use something like
[perlbrew](https://perlbrew.pl/) to just manage, a la homebrew, to
install a local fresh installation of perl5. `perlbrew` is pretty
cool!

Out of the box, that failed. But the instructions suggested running
perlpatch, from the same download, to try and fix perl up before doing
the perlbrew of perl.

However, in order to run _that_ command, we need GNU patch which, you
guessed it, isn’t installed on our machine.

I had to download patch [from the GNU site](https://ftp.gnu.org/gnu/patch/)

# Installing in userspace

Since we’re having to install all of these in userspace, we’ll need to
have a directory convention for the download and install of all this
software. One of the keys is that most of these packages in their
configure script tell you about either a `PREFIX=` or `--prefix=`
configuration setting to let you install the software wherever you
want (in your own space ofc).

From there, I had to then perlbrew install-patchperl

From there, I had to install (some) right version of perl 5.16.3 in my case.

From there I could finish installing openssl 1.1 and add that on my
path.

Once I had that I was one step closer to installing python3, because
the next issue was that I do not have libffi installed.

The correct way to solve this problem is to download the release of
libffi, which has the installation script built in, and run that
script.

Okay, now these prerequisites are seemingly installed. Why isn’t the
install script running?

# Flags to the configure script

One of the keys was to read the configuration options for the
configure script. These are also [reproduced
online:](https://docs.python.org/3/using/configure.html#linker-options)

[This](https://bugs.python.org/msg381577) gave me the flags and
information for how to solve the missing libssl information.

These both describe the problem with the SSL installation, that may
have been a bug in the install script expecting a different directory
structure for openssl.
[And](https://github.com/pyenv/pyenv/issues/1183)
[also](https://github.com/pyenv/pyenv/issues/1183#issuecomment-605018391)
[these](https://bugs.python.org/msg321096).

## `LD_LIBRARY_PATH`

A little bit of cli-fu is just enough to hang yourself. Xah Lee gives
a nice history lesson on some of the ways this can bite you. I may or
may not have needed some settings for this install. But I did avail
myself.

# Scripting it!

Now that we have a way to build this, we’d sure like to not have to do
it all over again by hand. The right thing to do here is to script the
install. The expedient thing is diligent notes. We won’t expose *all*
of it here, but certainly this could be useful to the diligent student
reader.

```
./configure --prefix=$HOME/python \
  LDFLAGS="-L$SPOT/libffi/lib64/ -L$SPOT/openssl/lib/"
  CPPFLAGS="-I$SPOT/openssl/include/openssl/ -I$SPOT/libffi/include/"
  --with-openssl=$SPOT/openssl/
```

# Takeaways

1. Be kind to your sysadmin. Buy that person chocolates and Christmas
cards.

2. Read The Friendly Manual. The information you want is sitting
_right_ there. You can of course

3. Your browser is your friend. There was a time when you had mailing
lists and man pages and you felt good that you did. Now, you also have
the ol’ googler.

