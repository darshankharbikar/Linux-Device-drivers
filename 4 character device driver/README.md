# Character Device Driver – Major Number and Minor Number

This document explains the concepts of **major** and **minor** numbers in Linux character device drivers, and why they are essential for registering and accessing devices from user space.  
Original tutorial: [EmbeTronicX – Character Device Driver: Major Number and Minor Number](https://embetronicx.com/tutorials/linux/device-drivers/character-device-driver-major-number-and-minor-number/)

---

## 1. Why Devices Need Numbers

In Linux, every device file (like `/dev/ttyS0`, `/dev/mem`, `/dev/mychardev`) is associated with:

- A **major number**
- A **minor number**

These numbers tell the kernel:

- Which **driver** should handle operations on that device (via the major number).
- Which **specific device** or sub-device within that driver (via the minor number).

User-space programs do not talk directly to hardware; they open `/dev/...` files, and the kernel routes those calls to the correct driver using major/minor numbers [web:0].

---

## 2. Major Number

The **major number**:

- Identifies the **device driver** (or device class).
- Maps a device file to a specific driver in the kernel.

Examples:

- `ttyS` (serial port) has a specific major number.
- `mem` (memory devices) has another major number.
- Your custom character driver will get its own major number when registered.

When you register a character driver, you can:

- Request a **dynamic major number** (kernel assigns one).
- Or specify a **static major number** (if you know it’s free).

In code, this is often done with:

```c
int major = register_chrdev(0, "mydriver");  // 0 => dynamic
```

If you use `0` as the major number, the kernel returns a dynamically assigned value [web:0].

---

## 3. Minor Number

The **minor number**:

- Is used **within** a driver to distinguish multiple devices.
- Does **not** directly map to a different driver; the major number already did that.
- Is passed to the driver’s operations (e.g., `open`, `read`, `write`) so the driver can decide which hardware or sub-device to use.

Example:

- `/dev/mydev0` → major = N, minor = 0  
- `/dev/mydev1` → major = N, minor = 1  

Same driver (same major), but different minor numbers.

Inside your driver, you can read the minor number from the `inode` or `file` structure to choose the correct device.

---

## 4. Device File and /dev

When you create a character device:

1. You register the driver in the kernel (using `register_chrdev` or `alloc_chrdev_region` + `cdev_init` + `cdev_add`).
2. The kernel knows the major number.
3. You create a device file in `/dev`:

   ```bash
   mknod /dev/mydev c <major> 0
   ```

   or using `udev` rules in modern systems.

User programs then open:

```c
int fd = open("/dev/mydev", O_RDWR);
```

The kernel uses the major number to find your driver, and the minor number to know which device instance you want [web:0].

---

## 5. Dynamic vs Static Major Numbers

### Static Major Number

- You specify a fixed number (e.g., `major = 240`).
- Must ensure it’s not used by another driver.
- Less flexible; not recommended for most modern drivers.

### Dynamic Major Number

- You request `major = 0`.
- The kernel assigns a free number and returns it.
- More flexible and safer.

Modern code often uses:

```c
int major;
major = register_chrdev(0, "mydriver");  // dynamic
if (major < 0) {
    // registration failed
}
```

---

## 6. Conceptual Diagram

Here is a simple diagram showing how major/minor numbers connect user space, kernel, and hardware:

```text
+----------------------+
|      User Space      |
|  open("/dev/mydev0") |
+----------+-----------+
           |
           | File: /dev/mydev0
           | Major = N, Minor = 0
           v
+----------------------+       +---------------------------+
|      Linux Kernel    |<----->|  Character Driver (N)     |
|  VFS + Device Layer  |       |  Handles minor = 0, 1, ...|
+----------+-----------+       +---------------------------+
           |
           | Driver uses minor to select device
           v
+----------------------+
|       Hardware       |
| (Device 0, Device 1) |
+----------------------+
```

If you want a nicer image, link to your own diagram:

```markdown

```

---

## 7. What You Learned Here

- What **major numbers** and **minor numbers** are.
- How they map `/dev/...` device files to drivers and device instances.
- How to register a character driver with dynamic or static major numbers.
- Why minor numbers let a single driver manage multiple devices.

In the next steps, you’ll use this knowledge to write a full **character device driver** with `open`, `read`, `write`, and `release` operations.
