# Linux Device Driver – Part 2: First Device Driver

This document summarizes a simple example of writing and testing a first Linux device driver as a Loadable Kernel Module (LKM).  
Original tutorial: [EmbeTronicX – Linux Device Driver Tutorial Part 2: First Device Driver](https://embetronicx.com/tutorials/linux/device-drivers/linux-device-driver-tutorial-part-2-first-device-driver/)

---

## 1. Goal of this Example

The aim is to create a minimal Linux kernel module that can be:

- Compiled against the running kernel.
- Inserted into the kernel using `insmod`.
- Removed from the kernel using `rmmod`.
- Observed via `dmesg` or `/var/log/kern.log` messages.

This is usually the first step before writing a full character or block driver [web:0].

---

## 2. Basic Structure of a Kernel Module

A simple kernel module typically defines:

- **Initialization function**  
  Runs when the module is inserted (e.g. `module_init(my_module_init)`).

- **Exit function**  
  Runs when the module is removed (e.g. `module_exit(my_module_exit)`).

It also includes metadata like `MODULE_LICENSE`, `MODULE_AUTHOR`, and `MODULE_DESCRIPTION` [web:0].

---

## 3. Example Source File Layout

A common layout:

- `first_driver.c` – source code for the kernel module.
- `Makefile` – builds the module against the current kernel.

Example `Makefile` sketch (adapt to your needs):

```makefile
obj-m += first_driver.o

KDIR := /lib/modules/$(shell uname -r)/build
PWD  := $(shell pwd)

all:
	$(MAKE) -C $(KDIR) M=$(PWD) modules

clean:
	$(MAKE) -C $(KDIR) M=$(PWD) clean
```

You will run `make` in this directory to produce `first_driver.ko`.

---

## 4. Building and Loading the Module

Typical steps:

1. Build the module:

   ```bash
   make
   ```

2. Insert the module:

   ```bash
   sudo insmod first_driver.ko
   ```

3. Check kernel logs:

   ```bash
   dmesg | tail
   ```

4. Remove the module:

   ```bash
   sudo rmmod first_driver
   ```

5. Confirm removal in logs:

   ```bash
   dmesg | tail
   ```

The init and exit functions usually print messages like “Module loaded” and “Module removed” so you can see that the code ran [web:0].

---

## 5. Conceptual Diagram

Here is a simple diagram showing where your first driver sits in the system:

```text
+----------------------+
|      User Space      |
|   (Shell / Tools)    |
+----------+-----------+
           |
           | insmod / rmmod
           v
+----------------------+       +-----------------------+
|      Linux Kernel    |<----->|  First Driver Module  |
| (Core + Other LKMs)  |       | (first_driver.ko)     |
+----------+-----------+       +-----------------------+
           |
           | Hardware access (later drivers)
           v
+----------------------+
|       Hardware       |
+----------------------+
```

If you want a nicer image, you can link to a diagram hosted somewhere:

```markdown

```

Replace the URL above with your own diagram link.

---

## 6. What You Learned Here

- How to structure a minimal kernel module with init and exit functions.
- How to compile the module using a kernel-aware `Makefile`.
- How to insert and remove the module and inspect kernel messages.

From here, the next logical step is to extend this simple module into a basic **character driver** that implements `open`, `read`, `write`, and `release` operations.
