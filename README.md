#  Custom Linux Kernel 5.10.246 — Vedant Panari

This repository documents the **successful compilation and deployment of a custom Linux 5.10.246 kernel** on Ubuntu.
The kernel includes a custom `printk()` message inside `start_kernel()` to verify source modification.

---

##  System Configuration

| Parameter      | Details                   |
| -------------- | ------------------------- |
| OS             | Ubuntu 22.04 LTS          |
| Kernel Version | Linux 5.10.246 (LTS)      |
| Architecture   | x86_64                    |
| Hardware       | Dell OptiPlex 3050        |
| Compiler       | GCC via build-essential   |
| Swap Space     | 8 GB (increased manually) |

---

##  Objective

* Build the Linux 5.10.246 kernel from source on Ubuntu
* Insert a **custom boot-time message** using `printk()`
* Successfully compile, install, and verify the custom kernel
* Push the modified source tree to GitHub

---

##  Step-by-Step Procedure

### 1️⃣ Install Required Dependencies

```bash
sudo apt update
sudo apt install -y build-essential libncurses-dev bison flex libssl-dev libelf-dev bc
```

These tools are required for kernel compilation:

* `gcc`, `make` → compile source
* `libncurses-dev` → text-based menu configuration
* `bison`, `flex` → preprocess kernel configuration
* `libssl-dev`, `libelf-dev` → for ELF file handling and cryptography
* `bc` → required for math expressions in build scripts

---

### 2️⃣ Increase Swap Space to 8 GB

This ensures smooth compilation on systems with low physical memory.

```bash
sudo swapoff /swapfile
sudo rm /swapfile
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
swapon --show
```

Expected:

```
NAME      TYPE SIZE USED PRIO
/swapfile file   8G   0B   -2
```

---

### 3️⃣ Download and Extract Linux 5.10.246 Source Code

```bash
cd ~
wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.10.246.tar.xz
tar -xf linux-5.10.246.tar.xz
cd linux-5.10.246
```

---

### 4️⃣ Configure Kernel Build

Copy the current kernel configuration and modify it for this version.

```bash
cp /boot/config-$(uname -r) .config
yes "" | make oldconfig
scripts/config --set-str CONFIG_SYSTEM_TRUSTED_KEYS ""
scripts/config --set-str CONFIG_SYSTEM_REVOCATION_KEYS ""
scripts/config --disable MODULE_SIG
make olddefconfig
make localmodconfig
```

Explanation:

* Uses existing kernel config for compatibility.
* Disables invalid certificate checks.
* Builds only required drivers for your hardware (minimal kernel).

---

### 5️⃣ Insert Custom Print Statement

Edit `init/main.c`:

```bash
gedit init/main.c
```

Locate:

```c
asmlinkage __visible void __init __no_sanitize_address start_kernel(void)
{
    char *command_line;
    char *after_dashes;
```

Add:

```c
    printk(KERN_INFO "Vedant: Custom Linux 5.10 kernel booted successfully.\n");
```

Save → Exit 

---

### 6️⃣ Build the Kernel

Compile kernel image and modules using two threads (safe for low-end systems):

```bash
make -j2 bzImage
make -j2 modules
```

Expected message:

```
Kernel: arch/x86/boot/bzImage is ready
```

---

### 7️⃣ Install Kernel and Update GRUB

```bash
sudo make modules_install
sudo make install
sudo update-grub
sudo reboot
```

GRUB automatically detects and lists the new kernel:

```
Found linux image: /boot/vmlinuz-5.10.246
Found initrd image: /boot/initrd.img-5.10.246
Adding boot menu entry for UEFI Firmware Settings ...
done
```

---

### 8️⃣ Boot into the New Kernel

At GRUB menu:

```
Advanced options for Ubuntu → Ubuntu, with Linux 5.10.246
```

After logging in, verify:

```bash
uname -r
```

Output:

```
5.10.246
```

---

### 9️⃣ Verify Custom Print Statement

By default, only root can read kernel logs:

```bash
sudo dmesg | grep Vedant
```

Expected output:

```
[    0.xxxxxx] Vedant: Custom Linux 5.10 kernel booted successfully.
```

---

###  Step 10 — Clean Up (Optional)

To reclaim disk space:

```bash
sudo rm -rf ~/linux-5.10.246*
sudo swapoff /swapfile && sudo rm /swapfile
```

---

## Verification Summary

| Command                     | Expected Output           |                                                         |
| --------------------------- | ------------------------- | ------------------------------------------------------- |
| `uname -r`                  | `5.10.246`                |                                                         |
| `sudo dmesg                 | grep Vedant`              | `Vedant: Custom Linux 5.10 kernel booted successfully.` |
| `swapon --show`             | `/swapfile file 8G 0B -2` |                                                         |
| `ls /boot/vmlinuz-5.10.246` | File exists               |                                                         |

---

## References

* [The Linux Kernel Archives](https://www.kernel.org/)
* [Kernel Newbies: Building a Kernel](https://kernelnewbies.org/KernelBuild)
* Chatgpt 5 : 
    * Give me a step-by-step process of kernel compilation from start for a low end device.
    * Is there a way to shorten compilation process?
    * Increase swap to 8GB for smoother build

---

### Author

**Vedant Panari**

PRN : 22610073

B.Tech in Information Technology

Ubuntu Kernel Customization (OSS Lab)
