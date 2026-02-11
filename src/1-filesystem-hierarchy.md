# Filesystem Hierarchy (FHS 1)

The *Filesystem Hierarchy Standard* (FHS) can be found in
[this document](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/index.html).

Why is this relevant? The FHS is a good guideline of where files are placed on your computer,
especially things at the root of the filesystem. This standard is what defines what goes
in `/etc` and `/dev` and all sorts of other stuff!
Here, we're mainly going to be concerned with the `/usr` directory.

Recall that in the last chapter, we made a `usr` directory and put a bunch of stuff into
it. The structure of `usr` should look something like this:
```
.
├── bin 
│   └── musl-gcc
├── include
│   └── ...
└── lib
    └── ... 
```

`musl-gcc` is irrelevant; the important directories are the other two. In fact,
you'll find them on your own system! Check them out!

The `/usr/include` directory contains tons of header files, a lot of them for libraries
that you may have installed: think about the `-dev` or `-devel` versions of a lot of
packages.

On the other hand, the `/usr/lib` directory contains tons of SO files, which, as mentioned
earlier, are libraries. We will implement this later, but for now, our lib
directory only contains our static libraries.

We will be creating our FHS-compliant layout within `fsroot`. In fact, this is what the
`sysroot` argument on the previous page referred to: `sysroot` expects a psuedo-root that
contains the headers and libraries in the expected places. `prefix` serves a similar purpose.

Before we proceed with putting additional stuff inside `fsroot`, let's first see what it
would look like if `fsroot` was actually the root.

## Chroot

Chroot is a program that allows you to imagine that a particular directory is the root
directory: nothing outside of that directory is accessible.

```admonish note title="Chroot security"
A standard disclaimer made when mentioning chroot is that it isn't a sandbox! There are
ways to get out of a simple chroot jail.
```

Copy the statically-linked busybox into `fsroot`. Then run `sudo chroot path/to/fsroot /busybox`.

You should see the standard help message of busybox. Now, to actually get a shell,
run `sudo chroot path/to/fsroot /busybox sh`.

You may notice that while shell builtins, like `cd` and `echo` work, programs like `ls` do not.
This is because these programs are usually binaries stored somewhere on PATH; check out your
PATH with `echo $PATH`. None of these directories exist except `/usr/bin`, which contains a
dynamically linked file anyways! So we effectively have nothing on path.

A temporary workaround is to prefix everything with `/busybox`: e.g. `/busybox ls`.
Let's put something on PATH so
that we can actually use the shell normally.

```admonish tip title="Busybox standalone shell"
Another workaround, which I'm not going to show here, is to enable busybox's standalone
shell options, directions for which can be found in the INSTALL file in busybox's repository.
This mode defaults to running busybox applets when the binary can't be found.
```

While still in the chroot environment, run these commands:
```bash
/busybox mkdir bin
/busybox --install -s bin
```
Here, we create a directory `/bin`, then run busybox's installation program, which creates
symbolic links to itself from the name of every applet it provides: e.g. it symlinks
`/bin/ls` to itself, `/busybox`. This works because busybox is a *multi-call binary*, which
analyze what name they are called under to determine what function to execute.

Now, we can interact normally in the chroot shell! Run `ls /bin -l` to see how the installed
applets are actually symlinks!

We have created a relatively isolated system filesystem here; now all that remains is to boot it!
