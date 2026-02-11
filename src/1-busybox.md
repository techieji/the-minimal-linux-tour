# Busybox

[Busybox](https://busybox.net/) is essentially all of your standard binary tools wrapped
up into a single binary: from basic tools like `ls`, `mv`, `cat`, to more complex
utilities like `sh`, `vi`, `awk`, and others, to system administration tools, like
`chown`, `crond`, and `ip`. It is incredibly versatile.

Clone it and change to the most recent release (as of writing, 1.36.1):

```bash
git clone https://github.com/mirror/busybox.git
git checkout 1_36_1    # switching to the 1_36_1 tag
```

Now, let's compile it and mess around!

```bash
cd busybox
make defconfig    # setup the default config
make
```

```admonish note title="Speed"
If you have a multicore machine, run `nproc` to get the number of cores on
your machine and then run `make -jn`, with `n` being replaced with the number of
cores: e.g., `make -j16`. This should significantly speed up your compilation time!
```

```admonish tip title="Traffic Control Compilation Errors"
If you are encountering errors in `networking/tc.c`, this is due to a recent version
of Linux changing the symbols for traffic control. We have no need for traffic control,
so disable it by running `make menuconfig`, navigating to `Networking Utilities` and
disabling `tc`; save your new configuration and run `make` again.

If you run into an error when running `make menuconfig`, look for the fix below about that.
```

You should now have a file labeled `busybox`: run it with `./busybox`. This gives you a whole
ton of output.

Busybox contains a set of *applets*. Each of these can be run with `busybox <applet>`. Start
off by running `busybox sh`, and see what else it can do!

## Independence

That being said, can we just copy over this file to any old system and expect it to work?
It turns out that this executable is not independent of where you're executing it!

Run `file ./busybox`. Somewhere in the middle, you should see:
`dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2`. This is a hallmark of
a *dynamically linked program*. What does that mean? Let's see:

Run `ldd ./busybox`. You should see a list of lines of the format `library => path`.
`ldd` prints out the *shared object dependencies* of a program. On my system, I see this:
```
prajasekar@pradtop busybox $ ldd busybox
	linux-vdso.so.1 (0x00007fc677414000)
	libm.so.6 => /lib64/libm.so.6 (0x00007fc6772f3000)
	libresolv.so.2 => /lib64/libresolv.so.2 (0x00007fc6772e1000)
	libc.so.6 => /lib64/libc.so.6 (0x00007fc6770ef000)
	/lib64/ld-linux-x86-64.so.2 (0x00007fc677416000)
```
What this means is that `busybox` expects to find `libm` (the math library) at
`lib64/libm.so.6`, and similarly for the other libraries listed! This means that if any of these
libraries are in the wrong place or aren't even on the system, the executable wouldn't
work!

This is a pretty strong constraint on what the new system should look like, and can cause
a lot of headaches in setting up a minimal system. The solution to this is *static linking*. 

So what is the difference between static and dynamic?
Remember the interpreter line above? Dynamically linked programs use that interpreter to
find library functions at runtime, while statically linked programs have the library function
compiled into the final executable. This means that statically linked programs are self-sufficient,
which is ideal in this situation.

```admonish question title="Can I use dynamic linking instead?"
You could by placing the appropriate SO files
in the right locations. However, I'll leave that as an
exercise to the reader since conceptually, static linking is simpler.

In any case, dynamic linking will be done in the future.
```

To compile busybox with static linking, run `make menuconfig`, enter `Settings`, and enable
"Build static binary".

```admonish tip title="Ncurses-devel not being found"
This seems to be a bug in one of the scripts that improperly checks whether or not ncurses
is installed.

To fix, open `scripts/kconfig/lxdialog/check-lxdialog.sh` and change line 50 from
`main() {}` to `int main() {}`.
```

At this point, you could run `make` and see if that works. It doesn't work on my system,
but we're going to use advantage of this to download some other repositories that we're going
to use: a libc and the Linux kernel. We're going to be compiling busybox with the help of
these packages.

For a libc, I used [Musl](http://wiki.musl-libc.org), a standards-compliant, performant, and
lightweight libc implementation. Download it from
[https://git.musl-libc.org/cgit/musl/](https://git.musl-libc.org/cgit/musl/)
by downloading the `musl-<version>.tar.gz`. Extract it on your computer using
`tar xvf <filename>`.

Make a directory called `fsroot` somewhere; the name really doesn't matter, as long as it's
empty.
Now, make a directory for the library called `usr` inside `fsroot`;
this directory has to be called `usr`.
Then, run these commands in the musl source folder:
```bash
# we don't need shared library support:
./configure --prefix=<path/to/fsroot>/usr --disable-shared
make
make install
```
Now, there is a directory in `usr` called `lib`: this should contain the desired libraries,
the files that end in a `.a` suffix.

The values of the paths will be explained in the next chapter.

We're not done yet: busybox also requires the Linux headers, and right now,
`usr/include` only contains the libc headers.
So download
the kernel, which we're going to need later anyways.

Run these commands to get the Linux headers:
```bash
git clone --depth 1 https://github.com/torvalds/linux.git
make defconfig
make headers
make header_install INSTALL_HDR_PATH=<path/to/fsroot>/usr
```

After running `make headers`, there should be a bunch of header files in `usr/include` in
the linux repository. `make header_install ...` installs them in the given location with
`/include` added on effectively. So now, we have all the headers ready!

Now, we have to show busybox where all the library files are: go back into busybox,
run `make menuconfig`, go
to Settings, and set the field "Path to sysroot" to `path/to/fsroot`.

Finally, you can just run `make`.

Now, run `file` on the new busybox binary: it should say `statically linked` in the middle.
If you run it, it should behave exactly the same.

We can demonstrate that it's truly independent from the host system using the `chroot`
command; however,
we first have to start setting up a filesystem hierarchy.
