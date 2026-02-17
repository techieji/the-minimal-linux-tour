# (Pseudo)-Filesystems (FHS 2)

Now, we're going to start adding some more directories to the root filesystem. Unlike before,
these directories won't actually hold files: instead, they are *pseudo-filesystems*, meaning
that their contents will be dynamically created and managed by the kernel. We will first be looking at
the `/proc` and `/sys` directories.

## Proc and Sys

We need to enable support for generating these directories in the Linux kernel; enable
**File Systems → Pseudo filesystems → /proc file system support** and 
**File Systems → Pseudo filesystems → sysfs filesystem support**. Then, recompile your kernel.
Make a `/proc` and a `/sys` directory at the root of your filesystem, and recompress it into a ramdisk.
Then, enter your system.

If you type `ls /sys` or `ls /proc`, you will see nothing. This is because, like any external filesystem,
we need to *mount* it; most familiarly, we need to mount USB drives, but the mount command is fairly general,
and we need to use it in this case to actually access the "files" in the sys and proc filesystems. Run
these commands:

```bash
mount -t sysfs sys sys
mount -t proc proc proc
```

If you look at the manual, you'll see that `mount` takes a type argument `-t`, a device, and a mount point;
although the type argument is usually optional, we need to provide it here.
In any case, you can verify that the `/proc/` and `/sys` directories are no longer unpopulated. So, what do they
contain?

`/proc` is the more relevant of the two directories: this directory contains information about every executing
process, and a little more besides. For example, we can list all the mounted objects by looking at the file
`/proc/mounts` (easily accessed by just running `mount` with no arguments). There is also `/proc/kmsg` which
contains kernel logs (much like `/dev/kmsg` discussed earlier).

Inside each numbered directory, we can find details about each executing process by PID. You can find a list
of executing processes and their PIDs with `ps`. Here are some interesting
files:
 - `cmdline`: contains the command executed (if there was one) to start this process
 - `cwd`: a symlink to the processes "current working directory"
 - `exe`: a symlink to the process that is executing
 - `fd`: a directory containing the *file descriptors* a process is holding
and so on. Here is the [full documentation](https://www.kernel.org/doc/html/latest/filesystems/proc.html)
of the contents of the `/proc` directory.

What about `/sys`? This directory contains less useful data; it is mainly a way to access data from the
kernel. Since this tutorial aims to go from the kernel up, we're not going to be looking in detail at what
it contains. Similar to the previous directory, the full documentation can be found on the kernel website
[here](https://www.kernel.org/doc/html/latest/filesystems/sysfs.html).

## fstab

In any case, note that we had to manually mount these two filesystems. These aren't the only filesystems there
are: at some point, we have to mount the hardware to get off of the RAM! Although we can simply put everything in
the init script, there is a more modular way to do it: an `fstab`.

Create the file `/etc/fstab` with the following content:
```fstab
# device-spec	mount-point	fs-type		options		dump	fsck
proc		/proc		proc		defaults	0	0
sysfs		/sys		sysfs		defaults	0	0
```
(the first line isn't strictly necessary). You can see that this file contains a lot of the information we
had manually provided to the `mount` command: we give it a point to mount to, its type, and the name of the
filesystem. If this file exists, running `mount -a` will mount everything in the `fstab`. Therefore, after
making this file, we can edit our `/etc/init.d/rcS` to run `mount -a`, and this will let the machine
automatically mount these two pseudo-filesystems before we even receive access to the terminal.

TODO: add tmpfs and devpts
