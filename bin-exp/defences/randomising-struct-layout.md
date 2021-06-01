# Randomising struct layout

## Linux VFS Structure <a id="linux-vfs-structure"></a>

Let's take a look inside Linux's Virtual File System \(VFS\) Structure

{% code title="include/linux/fs.h" %}
```c
struct file_operations {
    int (*open) (struct inode *, struct file *);
    ssize_t (*read) (struct file *, char __user *,
                      size_t loff_t *);
    ssize_t (*write) (struct file *, const char __user *,
                      size_t loff_t *);
    loff_t (*llseek) (struct file *, loff_t, int);
    int (*flush) (struct file *, fl_owner_t id);
} __randomize_layout;
```
{% endcode %}

What this does is it acts as a level of indirection.

A concrete filesystem, such as ext4, defines concrete implementations for these functions. When a process issues a read or open call, the OS can use this higher-level abstraction of a file operation to figure out which specific file implementation to call.

We see a load of function pointers here. As an attacker, we can be jumping with joy here, because we can find the location of some of these and overwrite them with a buffer overflow, allowing us to redirect control-flow to wherever we want to go to.

If we look down at the bottom, however, we find the `__randomize_layout` incantation.

## What does "\_\_randomize\_layout" do?  <a id="what-does-__randomize_layout-do"></a>

It instructs GCC to generate different field offsets \(for things like the open pointer and the read pointer etc.\) during each compilation. So each time it's compiled, you'll get a different layout for each particular `struct`

## Why is this helpful? <a id="why-is-this-helpful"></a>

This makes it harder for evil code to read and write known values at known offsets. Every time someone compiles the Linux kernel, this particular `struct` will have a different order for each of these `structs` here. In general, adding non-determinism to the environment under attack makes it more difficult for the attacker to actually successfully exploit that environment.

