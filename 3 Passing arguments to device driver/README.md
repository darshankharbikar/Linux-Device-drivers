# Linux Device Driver – Part 3: Passing Arguments to Device Driver

This document summarizes how to pass arguments (parameters) into a Linux kernel module so the driver can be configured at load time instead of being hard-coded.  
Original tutorial: [EmbeTronicX – Linux Device Driver Tutorial Part 3: Passing Arguments to Device Driver](https://embetronicx.com/tutorials/linux/device-drivers/linux-device-driver-tutorial-part-3-passing-arguments-to-device-driver/)

---

## 1. Goal of this Example

In Part 2, you built a minimal kernel module that just prints messages on load/unload. In this part, you extend it so that:

- The module accepts **parameters** (e.g. an integer, a string, or a boolean).
- These parameters are set when you run `insmod`:

  ```bash
  sudo insmod first_driver.ko my_int=42 my_string="hello"
  ```

- The driver uses those values in its init function (or later operations) instead of fixed constants [web:0].

This makes drivers more flexible and easier to test without changing source code.

---

## 2. Declaring Module Parameters in C

The typical pattern in Linux uses:

- **Global variables** to store the parameter values.
- **`module_param()`** macro to expose them as load-time parameters.
- Optional **`MODULE_PARM_DESC()`** or the `description` field in `module_param()` to provide help text.

Example sketch (adapt to your exact code):

```c
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/init.h>

static int my_int = 0;
static char *my_string = "default";

module_param(my_int, int, 0644);
module_param(my_string, charp, 0644);

MODULE_PARM_DESC(my_int, "An example integer parameter");
MODULE_PARM_DESC(my_string, "An example string parameter");

static int __init my_module_init(void)
{
    printk(KERN_INFO "Module loaded with my_int=%d, my_string=%s\n",
           my_int, my_string);
    return 0;
}

static void __exit my_module_exit(void)
{
    printk(KERN_INFO "Module removed\n");
}

module_init(my_module_init);
module_exit(my_module_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Your Name");
MODULE_DESCRIPTION("Example driver with parameters");
```

Key points:

- `my_int` and `my_string` are global but **static** to this file.
- `module_param()` makes them visible to `insmod` as `my_int` and `my_string`.
- `0644` is the permission for viewing/changing via `/sys/module/...` (optional detail) [web:0].

---

## 3. Passing Parameters at Load Time

You set parameters when inserting the module:

```bash
sudo insmod first_driver.ko my_int=10 my_string="test"
```

You can also check current values after loading:

```bash
cat /sys/module/first_driver/parameters/my_int
cat /sys/module/first_driver/parameters/my_string
```

This shows how the kernel exposes module parameters as sysfs files.

---

## 4. Why Use Parameters?

Reasons to use module parameters:

- Configure behavior without changing source code.
- Test different configurations quickly (e.g. different timeouts, IDs, debug flags).
- Make the driver more reusable across different hardware or environments [web:0].

For example, instead of hard-coding a port number or a debug level, you pass it at load time.

---

## 5. Conceptual Diagram

Here is a simple diagram showing how parameters flow into the driver:

```text
+----------------------+
|      User Space      |
|  (Shell / insmod)    |
+----------+-----------+
           |
           | insmod first_driver.ko my_int=10 my_string="test"
           v
+----------------------+       +-----------------------+
|      Linux Kernel    |<----->|  Driver with Params   |
| (Core + Other LKMs)  |       | my_int, my_string     |
+----------+-----------+       +-----------------------+
           |
           | Driver uses my_int, my_string in init / runtime
           v
+----------------------+
|       Hardware       |
+----------------------+
```

If you want a nicer image, link to your own diagram:

```markdown

```

---

## 6. What You Learned Here

- How to declare module parameters using `module_param()`.
- How to pass parameters with `insmod`.
- How the kernel exposes parameters via `/sys/module/.../parameters/`.
- Why parameters make drivers more flexible and testable.

Next, you can build on this by adding real device operations (like `open`, `read`, `write`) to turn this into a basic **character driver**.
