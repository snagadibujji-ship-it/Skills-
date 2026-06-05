---
name: linux-kernel-module-development
description: Expert guidance on writing native Linux Kernel Modules (LKMs), interacting with kernel APIs, memory management, and character device drivers. Triggers on "kernel module", "lkm", "character device", "kmalloc", "kernel space", "copy_to_user".
---

# Linux Kernel Module Development

## Purpose
This skill provides systems engineers with strict guidelines for writing, compiling, and debugging Linux Kernel Modules (LKMs) and device drivers securely and efficiently in C.

## When to Use
- Writing custom drivers for novel hardware.
- Creating character devices to expose kernel features to userspace.
- Implementing low-level network packet filters (Netfilter) or filesystem hooks.
- Debugging kernel panics or oopses caused by out-of-tree modules.

## Key Information

### 1. The Kernel Environment
The kernel is a vastly different environment from userspace:
*   **No Standard Library**: You cannot use `libc` (no `printf`, `malloc`). You must use kernel equivalents (`printk`, `kmalloc`).
*   **No Floating Point**: Floating-point operations are generally prohibited in kernel space to avoid corrupting FPU state.
*   **Concurrency Everywhere**: The kernel is heavily multi-threaded. Interrupts can happen at any time.

### 2. Module Basics
*   **`module_init()` / `module_exit()`**: The entry and exit points for your module.
*   **`printk()`**: The kernel print function. Use log levels like `KERN_INFO`, `KERN_ERR`. View output via `dmesg`.

### 3. Memory Management
*   **`kmalloc()` / `kfree()`**: Allocates physically contiguous memory. Extremely fast, but limited in size. Always specify flags (e.g., `GFP_KERNEL` for normal allocations, `GFP_ATOMIC` if in an interrupt handler where you cannot sleep).
*   **`vmalloc()` / `vfree()`**: Allocates virtually contiguous memory. Can allocate large chunks, but slower.
*   **Userspace Memory**: Never dereference a userspace pointer directly! Always use `copy_from_user()` and `copy_to_user()` to safely transfer data and handle page faults.

### 4. Character Devices
The most common way to communicate with userspace (via `/dev/my_device`).
*   Requires registering a Major and Minor number.
*   **`file_operations` struct**: Define function pointers for `open`, `read`, `write`, `release`, and `unlocked_ioctl`.

### 5. Concurrency and Synchronization
*   **Spinlocks**: Use for short critical sections. Disables preemption. You *cannot sleep* (e.g., call `kmalloc(GFP_KERNEL)`) while holding a spinlock.
*   **Mutexes**: Use for longer critical sections. The thread will sleep if the lock is contested.

## Best Practices
- ✅ **Handle Errors Explicitly**: Kernel functions often return negative error codes (e.g., `-ENOMEM`). Always check return values and clean up allocated resources using `goto` error labels to prevent memory leaks on failure.
- ✅ **Use Out-of-Tree Makefiles**: Use Kbuild to compile your module (`make -C /lib/modules/$(uname -r)/build M=$(PWD) modules`).
- ✅ **GPL License**: Declare `MODULE_LICENSE("GPL")` to access the full range of exported kernel symbols (`EXPORT_SYMBOL_GPL`).
- ❌ **Never Sleep in Interrupts**: If your code is in an interrupt context or holding a spinlock, you must not call any function that might block or sleep.
