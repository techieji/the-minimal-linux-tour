# Introduction

There are very many Linux tutorials out there. I have often found them to be a waste of time
since without doing useful actions, it's hard to keep the knowledge in my brain. At the
same time, some tutorials aimed at a thorough understanding of Linux, namely
[Linux From Scratch](https://www.linuxfromscratch.org/), are infuriatingly hard to get
started with. Finally, there are a lot of shorter, more reliable tutorials that don't
involve compiling hundreds of packages by hand, but they don't offer explanation of what's
going on.

So that is what this is meant to be. Here's why I'm writing this:
1. As a reference for myself in building a Linux setup
2. As a store of information on what exactly all the various terms mean
3. As a guide to others to replicate what I did and go further!

This guide is, above all, meant to be *minimal*, in that I'm not trying to create a new
distro, and *thorough*, meaning everything should be explained: not just commands, but
directories, kinds of filesystems, etc.
In addition, very few commands modify your system in a meaningful way: root is only used for
a few operations.

I hope you enjoy reading through it as much as I enjoyed writing it!

## Prerequisites

Here are some hard prereqs that I'd advise:
 - Linux system: you could probably use Windows with MSYS, but there are probably some limitations
   of that. You should also have standard tools, like `git`, installed.
 - Modern computer: we're going to be compiling some hefty stuff, without multicore compiling,
   you could end up having to wait for quite a while.
 - Virtualization software: I'm using qemu. With a VM, we can boot from the kernel image
   directly instead of fiddling around with the actual bootloader.

Here are some skill-based prereqs:
 - Building from source: for the majority of projects, just some variation on `./configure` and
   `make` should be enough, but debugging others' projects is a useful skill.
 - Comfort with command line: when we're running the kernel, all we have is a command line, so
   knowing common tools would be helpful. That being said, it's definitely possible to pick
   up while doing this tutorial.
 - Organization: **I won't tell you how to organize your stuff** unless it matters. Please be
   sane and remember where you put stuff!

This being said, even if you don't strictly meet the above,
I'd say see how far you get! 
I've tried to make this have a fairly low knowledge requirement.
Just trying to learn something leads down a lot
of other paths, so go for it!

## Typesetting

I know my writing style is verbose. Therefore, I will use some visual aids to break up
this block content and make it easier to understand:

```admonish note
Asides on specific components will be placed a box like this.
```

```admonish tip title="Fix"
Alternative fixes that I had to apply will be formatted like this.
```

```admonish faq
Questions that I asked and answered will be formatted like this.
```

```admonish danger
Dangerous actions (which are very rare) will be formatted like this.
```

## Sources

Every programmer builds on the work of others without citing them. I don't really remember
all the sources I've used, but here are a few of the most notable:

 - **A good tutorial in its own right**: [https://blinry.org/tiny-linux/](https://blinry.org/tiny-linux/)
 - **LFS, another useful reference**: [https://www.linuxfromscratch.org/~xry111/lfs/view/clfs-ng-systemd/index.html](https://www.linuxfromscratch.org/~xry111/lfs/view/clfs-ng-systemd/index.html)
 - Short instructions on compiling and booting the kernel: [https://github.com/mranv/minimalOS](https://github.com/mranv/minimalOS)
 - Another similar one with more detail: [https://github.com/bluedragon1221/minlinux2](https://github.com/bluedragon1221/minlinux2)
 - FHS reference: [https://refspecs.linuxfoundation.org/FHS\_3.0/](https://refspecs.linuxfoundation.org/FHS_3.0/)
 - And, of course, Wikipedia!

Minor references:
 - [Musl references on compiling busybox](https://wiki.musl-libc.org/building-busybox)
