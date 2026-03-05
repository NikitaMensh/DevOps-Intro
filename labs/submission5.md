# Lab 5 — Virtualization & System Analysis

## Task 1 — VirtualBox Installation

### Host Operating System

Host system information:

```bash
$ cat /etc/os-release
PRETTY_NAME="Debian GNU/Linux 12 (bookworm)"
NAME="Debian GNU/Linux"
VERSION_ID="12"
VERSION="12 (bookworm)"
VERSION_CODENAME=bookworm
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"
```

### VirtualBox Version

```bash
$ VBoxManage --version
7.1.16r172425
```

### Installation Notes

VirtualBox was installed on the host system using the official installer.
The installation completed successfully and the application launched without issues.

No problems were encountered during installation.

---

# Task 2 — Ubuntu VM and System Analysis

## VM Configuration

The Ubuntu virtual machine was created using the following configuration:

* **Operating System:** Ubuntu 24.04.4 LTS (Noble Numbat)
* **RAM:** 4 GB
* **Storage:** 25 GB
* **CPU:** 2 cores
* **Virtualization platform:** VirtualBox

---

# Operating System Information

### Tool used

`cat /etc/os-release`

### Command

```bash
$ cat /etc/os-release
```

### Output

```text
PRETTY_NAME="Ubuntu 24.04.4 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.4 LTS (Noble Numbat)"
VERSION_CODENAME=noble
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=noble
LOGO=ubuntu-logo
```

### Analysis

This command reads system information from `/etc/os-release`, which contains metadata about the operating system.
The VM is running **Ubuntu 24.04.4 LTS**, which is a long-term support release.

---

# CPU Information

### Tool used

`lscpu`

### Command

```bash
$ lscpu
```

### Output

```text
Architecture:                    x86_64
CPU op-mode(s):                  32-bit, 64-bit
Address sizes:                   39 bits physical, 48 bits virtual
Byte Order:                      Little Endian
CPU(s):                          2
On-line CPU(s) list:             0,1
Vendor ID:                       GenuineIntel
Model name:                      Intel(R) Core(TM) i3-8130U CPU @ 2.20GHz
CPU family:                      6
Model:                           142
Thread(s) per core:              1
Core(s) per socket:              2
Socket(s):                       1
Stepping:                        10
BogoMIPS:                        4415.96
Flags:                           fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx rdtscp lm constant_tsc rep_good nopl xtopology nonstop_tsc cpuid tsc_known_freq pni pclmulqdq ssse3 fma cx16 pcid sse4_1 sse4_2 x2apic movbe popcnt aes xsave avx f16c rdrand hypervisor lahf_lm abm 3dnowprefetch pti fsgsbase bmi1 avx2 bmi2 invpcid rdseed adx clflushopt arat md_clear flush_l1d arch_capabilities
Virtualization features:
Hypervisor vendor:               KVM
Virtualization type:             full
Caches (sum of all):
L1d:                             64 KiB (2 instances)
L1i:                             64 KiB (2 instances)
L2:                              512 KiB (2 instances)
L3:                              8 MiB (2 instances)
NUMA:
NUMA node(s):                    1
NUMA node0 CPU(s):               0,1
```

### Analysis

The VM has **2 virtual CPU cores** assigned.
The underlying physical CPU is **Intel Core i3-8130U @ 2.20 GHz**.

The presence of the **hypervisor flag** indicates the system is running inside a virtual machine.

---

# Memory Information

### Tool used

`free -h`

### Command

```bash
$ free -h
```

### Output

```text
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       1.1Gi       1.4Gi        36Mi       1.6Gi       2.7Gi
Swap:             0B          0B          0B
```

### Analysis

The VM has approximately **3.8 GB of RAM**, which corresponds to the **4 GB configuration set in VirtualBox**.
The system currently uses about **1.1 GB**, leaving significant memory available.

---

# Network Configuration

### Tool used

`ip a`

### Command

```bash
$ ip a
```

### Output

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever

2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:0a:c9:3c brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic noprefixroute enp0s3
```

### Analysis

The VM uses a **VirtualBox NAT network adapter**.
The assigned IP address is **10.0.2.15**, which is typical for VirtualBox NAT networking.

---

# Storage Information

### Tool used

`df -h`

### Command

```bash
$ df -h
```

### Output

```text
Filesystem      Size  Used Avail Use% Mounted on
tmpfs           392M  1.7M  390M   1% /run
/dev/sda2        25G  5.4G   18G  24% /
tmpfs           2.0G     0  2.0G   0% /dev/shm
tmpfs           5.0M  8.0K  5.0M   1% /run/lock
tmpfs           392M  152K  392M   1% /run/user/1000
```

### Additional Disk Details

```bash
$ lsblk -f
```

```text
NAME   FSTYPE   FSVER LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
loop0  squashfs 4.0                                         0   100% /snap/bare/5
loop1  squashfs 4.0                                         0   100% /snap/core22/2292
loop2  squashfs 4.0                                         0   100% /snap/gnome-42-2204/247
loop3  squashfs 4.0                                         0   100% /snap/gtk-common-themes/1535
loop4  squashfs 4.0                                         0   100% /snap/snap-store/1270
loop5  squashfs 4.0                                         0   100% /snap/firefox/7766
loop6  squashfs 4.0                                         0   100% /snap/firmware-updater/210
loop7  squashfs 4.0                                         0   100% /snap/snapd/25935
loop8  squashfs 4.0                                         0   100% /snap/snapd-desktop-integration/343
sda
├─sda1
└─sda2 ext4  1.0       78fb1516-f517-4af0-b112-adf7666d60c4 17.8G   22% /
sr0
```

### Analysis

The virtual disk has **25 GB capacity**, which matches the VM configuration.
The main filesystem is **ext4**, mounted at `/`.

---

# Virtualization Detection

### Tool 1

```bash
$ systemd-detect-virt
oracle
```

### Tool 2

```bash
$ sudo dmidecode -s system-product-name
VirtualBox
```

### Analysis

Both tools confirm the system is running inside a **VirtualBox virtual machine**.

* `systemd-detect-virt` identifies the hypervisor vendor
* `dmidecode` reports the system product name as **VirtualBox**

---

# Reflection

Several command-line tools were used to analyze the virtual machine system:

| Tool                  | Purpose                                |
| --------------------- | -------------------------------------- |
| `lscpu`               | CPU architecture and processor details |
| `free -h`             | Memory usage                           |
| `ip a`                | Network interfaces and IP addresses    |
| `df -h`               | Disk usage                             |
| `lsblk`               | Disk structure and filesystems         |
| `cat /etc/os-release` | OS version                             |
| `systemd-detect-virt` | Virtualization detection               |

Among these tools, **lscpu**, **ip**, and **df** were particularly useful because they provide concise and detailed information about system hardware and resources.