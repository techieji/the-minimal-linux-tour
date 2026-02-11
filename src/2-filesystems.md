# (Pseudo)-Filesystems (FHS 2)

Enter your Linux kernel directory and enable **/proc file system support**
and **sysfs filesystem support** under **File Systems -> Pseudo filesystems**.

Enable the block layer (top level)?

KERNEL CAN AUTOMOUNT DEV!

Then, recompile your kernel.

Make fstab.
Make `/proc`. Make `/sys`.

TODO: compile with CONFIG\_HOTPLUG??? for mdev


TEMPORARILY ENABLED BUNCH OF VIRTIO AND BLOCK
