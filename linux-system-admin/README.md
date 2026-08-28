# **Linux & System Administration - DevOps Interview Questions (200 Questions)**

Welcome to the **Linux & System Administration** master collection containing **200 comprehensive interview questions and detailed answers** covering Linux Kernel Internals, Systemd, Process & Memory Management, Virtual Memory, Storage & Filesystems, TCP/IP Network Socket Tuning, and Production Troubleshooting.

---

## 🟢 **Part 1: Linux Architecture, Processes & Memory Management (Questions 1–60)**

### **1. Explain the Linux Boot Process from power-on to user login.**
**Answer:**
1. **BIOS / UEFI:** Executes Power-On Self-Test (POST), checks hardware, and reads the bootloader from Master Boot Record (MBR) or EFI System Partition (ESP).
2. **Bootloader (GRUB2):** Loads the compressed Linux kernel image (`vmlinuz`) and initial RAM disk (`initramfs`) into memory.
3. **Kernel Initialization:** Mounts root filesystem in RAM, initializes hardware device drivers, and mounts real root filesystem (`/`).
4. **Init Process (`systemd` - PID 1):** Starts default system target (`multi-user.target` or `graphical.target`), spawning system daemons, network services, and login prompts (`agetty`).

### **2. What is `systemd` and what are Units, Targets, and Timers?**
**Answer:**
`systemd` is the Linux init system and service manager running as **PID 1**.
- **Unit:** A configuration file (`.service`, `.socket`, `.mount`, `.timer`) representing a managed system resource.
- **Target:** A synchronization point grouping units together (e.g., `multi-user.target` replaces SysV init runlevel 3).
- **Timer:** Native systemd replacement for cron, executing service units based on monotonic or calendar events.

### **3. Explain the Linux Process States (`R`, `S`, `D`, `Z`, `T`).**
**Answer:**
- **`R` (Running / Runnable):** Process is currently executing on CPU or waiting in the run queue.
- **`S` (Interruptible Sleep):** Process is waiting for an event or I/O; can be awakened by signals.
- **`D` (Uninterruptible Sleep - Disk Sleep):** Process is waiting on hardware I/O (disk, NFS); **cannot be killed by `SIGKILL`**.
- **`Z` (Zombie / Defunct):** Process has completed execution, but its exit status has not been reaped by its parent process.
- **`T` (Stopped / Traced):** Process suspended by `SIGSTOP` or debugger (`ptrace`).

### **4. What is a Zombie Process and how do you resolve it?**
**Answer:** A child process that has exited but remains in the process table because its parent has not invoked `wait()` / `waitpid()`. Zombies consume process table entries (PIDs) but zero CPU/RAM. Since zombies are already dead, they cannot be killed with `kill -9`.
- **Resolution:** Send `SIGCHLD` to the parent process, or kill the parent process so the zombie is orphaned and adopted by **PID 1 (`systemd`)**, which reaps it immediately.

### **5. Explain Linux Load Average (1, 5, and 15-minute metrics).**
**Answer:** The average number of processes that are in a **runnable state** (using or waiting for CPU) plus processes in **uninterruptible sleep (`D` state)** (waiting for disk or network I/O) over 1, 5, and 15 minutes.
- A load average of `4.0` on a 4-core machine means 100% CPU utilization. If load average is `12.0` on 4 cores, CPU or disk I/O is heavily saturated.

### **6. Compare `top`, `htop`, and `atop` for process monitoring.**
**Answer:**
- **`top`:** Built-in standard real-time process monitoring utility.
- **`htop`:** Interactive, colorized viewer supporting mouse clicks, tree views, and easy process signal dispatching.
- **`atop`:** Advanced performance monitor logging historical system-level metrics (CPU, memory, disk, network) and per-process activity to disk for retrospective troubleshooting.

### **7. What is Virtual Memory and how does Paging work?**
**Answer:** An abstraction providing each process with a private, contiguous address space. Memory is divided into fixed-size **Pages** (typically 4KB). The Memory Management Unit (MMU) uses **Page Tables** to translate virtual memory addresses to physical RAM addresses.

### **8. What is a Page Fault (Minor vs Major Page Fault)?**
**Answer:**
- **Minor Page Fault:** The requested memory page is already in physical RAM (e.g., shared library), requiring only updating the process page table (fast, no disk I/O).
- **Major Page Fault:** The requested page is not in RAM (must be read from swap space or disk filesystem), causing process latency stalls.

### **9. What is Swap Space and how does the `swappiness` kernel parameter work?**
**Answer:** Dedicated disk space used as an extension of physical RAM when memory is exhausted.
- **`vm.swappiness` (0–100):** Controls the kernel's aggressiveness in swapping anonymous memory pages to disk relative to reclaiming page cache (default: 60; recommended for Kubernetes/Databases: 0–10).

### **10. What is the Linux Out-Of-Memory (OOM) Killer?**
**Answer:** A kernel mechanism activated when physical RAM and swap are completely exhausted. It computes an `oom_score` for each process based on memory consumption and `oom_score_adj`, terminating the highest-scoring process with `SIGKILL` to prevent a full kernel panic.

### **11. What is `/proc` filesystem in Linux?**
**Answer:** A pseudo-filesystem (procfs) dynamically generated in memory by the kernel that acts as an interface to internal kernel data structures, process states, and runtime hardware metrics.

### **12. What is `/sys` filesystem (sysfs)?**
**Answer:** A pseudo-filesystem providing a structured, hierarchical tree of kernel objects, device drivers, and hardware subsystem settings (e.g., block devices, network interfaces, power states).

### **13. What is `systemctl` and how do you manage service lifecycles?**
**Answer:** CLI utility to manage systemd services:
- `systemctl start <service>` / `stop` / `restart` / `reload`
- `systemctl status <service>`
- `systemctl enable <service>` (creates symlinks in target directories to start on boot)
- `systemctl daemon-reload` (reloads unit files from disk)

### **14. What is `journalctl` and how do you query system logs?**
**Answer:** Queries the systemd binary structured journal:
- `journalctl -u nginx.service -f` (follow live service logs)
- `journalctl -p err -b` (show all error-level logs since current boot)
- `journalctl --since "1 hour ago"`

### **15. Explain File Permissions and Octal Notation (`chmod`).**
**Answer:** Read (`r` = 4), Write (`w` = 2), Execute (`x` = 1):
- `chmod 755 script.sh` $\rightarrow$ Owner: `rwx` (7), Group: `r-x` (5), Others: `r-x` (5).
- `chmod 644 file.txt` $\rightarrow$ Owner: `rw-` (6), Group: `r--` (4), Others: `r--` (4).

### **16. What are Special Permissions: SUID, SGID, and Sticky Bit?**
**Answer:**
- **SUID (`chmod u+s` / 4000):** Executes the binary with the file owner's permissions (e.g., `/usr/bin/passwd` runs as root).
- **SGID (`chmod g+s` / 2000):** New files created in directory inherit directory group ownership.
- **Sticky Bit (`chmod +t` / 1000):** In shared directories (`/tmp`), only file owners or root can delete files.

### **17. What is a Hard Link vs a Soft Link (Symlink)?**
**Answer:**
- **Hard Link:** A direct directory reference pointing to the existing inode on the same filesystem. Deleting the original file does not delete the data; data is removed only when link count reaches 0.
- **Soft Link (Symlink):** A separate file containing a path reference to the target file. Can cross filesystems; breaks (dangling link) if the target file is deleted or moved.

### **18. What is an Inode in Linux?**
**Answer:** A filesystem data structure storing metadata about a file: file size, ownership UID/GID, access permissions, timestamps (`atime`, `mtime`, `ctime`), and data block pointers (does **not** store filename or actual file content).

### **19. How do you troubleshoot "No space left on device" when `df -h` shows disk space available?**
**Answer:** The filesystem has **exhausted its Inodes** (caused by millions of zero-byte temporary files or mail queue files). Verify via `df -i`. Delete small orphaned files (`find /tmp -type f -delete`) to free inodes.

### **20. What is the difference between `atime`, `mtime`, and `ctime`?**
**Answer:**
- **`atime` (Access Time):** Last time file content was read.
- **`mtime` (Modification Time):** Last time file content was modified.
- **`ctime` (Change Time):** Last time file metadata or inode attributes (permissions, ownership) changed.

### **21. Explain the Linux Filesystem Hierarchy Standard (FHS).**
**Answer:**
- `/bin`, `/usr/bin`: Core user binaries.
- `/sbin`, `/usr/sbin`: System administration binaries.
- `/etc`: Host-specific configuration files.
- `/var`: Variable data (logs, spool, caches).
- `/tmp`: Ephemeral temporary files (cleared on reboot).
- `/dev`: Device nodes representing hardware.
- `/proc`, `/sys`: Virtual kernel filesystems.
- `/opt`: Add-on third-party software packages.

### **22. What is LVM (Logical Volume Manager) and what are PV, VG, and LV?**
**Answer:** Storage abstraction allowing dynamic disk resizing without repartitioning:
- **Physical Volume (PV):** Physical block devices (e.g., `/dev/nvme0n1`).
- **Volume Group (VG):** Pool of storage aggregating multiple PVs.
- **Logical Volume (LV):** Virtual partitions allocated from a VG, formatted with filesystems (ext4, XFS).

### **23. How do you extend an ext4 and XFS filesystem dynamically using LVM?**
**Answer:**
```bash
# 1. Extend the Logical Volume by 50GB
lvextend -L +50G /dev/vg_prod/lv_data

# 2. Resize filesystem online:
resize2fs /dev/vg_prod/lv_data   # For ext4
xfs_growfs /mnt/data             # For XFS
```

### **24. Compare ext4 vs XFS vs Btrfs.**
**Answer:**
- **ext4:** Battle-tested, rock-solid, supports shrinking and online expansion.
- **XFS:** High-performance 64-bit journaling filesystem optimized for large files and parallel multi-threaded I/O (default in RHEL/CentOS); cannot be shrunk.
- **Btrfs:** Copy-on-Write (CoW) filesystem with built-in subvolumes, snapshots, and software RAID.

### **25. What is `/etc/fstab` and what are the 6 fields?**
**Answer:** Defines static filesystem mounting rules on boot:
1. `Device/UUID` (e.g., `UUID=1234-...`)
2. `Mount Point` (e.g., `/mnt/data`)
3. `Filesystem Type` (e.g., `ext4`, `xfs`)
4. `Mount Options` (e.g., `defaults,noatime`)
5. `Dump` (0 = disable dump backup)
6. `Pass / Fsck Order` (1 = root filesystem, 2 = other filesystems, 0 = skip fsck).

### **26. What is `noatime` mount option and why is it used for database performance?**
**Answer:** Disables updating the inode `atime` whenever a file is read, eliminating write I/O overhead on disk during heavy read workloads.

### **27. What is `tmpfs` and `/dev/shm`?**
**Answer:** Memory-backed temporary filesystems storing files directly in virtual memory (RAM/swap) with ultra-low latency; wiped completely on reboot.

### **28. What is `strace` and how do you trace a running process?**
**Answer:** Diagnostic tool intercepting and recording system calls made by a process:
```bash
strace -p <PID> -f -e trace=open,read,write,network
```

### **29. What is `lsof` and how do you find open files/network sockets?**
**Answer:** Lists open files:
- `lsof -i :8080` (find process listening on port 8080)
- `lsof -p <PID>` (list all files/sockets opened by a process)
- `lsof +D /var/log` (list open files in a directory).

### **30. What is `fuser`?**
**Answer:** Identifies processes using specific files, directories, or sockets (`fuser -k 8080/tcp` kills the process holding port 8080).

### **31. What is `vmstat` and what do `bi`, `bo`, `wa`, `si`, `so` mean?**
**Answer:** Reports virtual memory statistics:
- `bi`/`bo`: Blocks received from (`bi`) / sent to (`bo`) block devices (Disk I/O).
- `wa`: CPU I/O Wait percentage (CPU idle while waiting on disk).
- `si`/`so`: Memory swapped in (`si`) / out (`so`) from disk per second ($> 0$ indicates RAM exhaustion).

### **32. What is `iostat` and what does `%util` indicate?**
**Answer:** Monitors I/O device loading:
- `%util`: Percentage of CPU time during which I/O requests were issued to the device. Approaching 100% indicates storage saturation and disk bottlenecks.

### **33. What is `netstat` vs `ss`?**
**Answer:** `ss` is the modern, fast replacement for legacy `netstat`. It dumps socket statistics directly from kernel memory (`sock_diag`) rather than parsing slow `/proc/net/` files.
- `ss -tulpn`: Show listening TCP/UDP sockets with process names and PIDs.

### **34. What is `ip` command vs `ifconfig`?**
**Answer:** `ip` (iproute2) replaces legacy `ifconfig`:
- `ip addr show` (replaces `ifconfig`)
- `ip route show` (replaces `route -n`)
- `ip link set eth0 up`

### **35. What is `sar` (System Activity Reporter)?**
**Answer:** Part of the `sysstat` package that collects, logs, and reports historical CPU, memory, network, and disk performance data.

### **36. What is `nice` vs `renice`?**
**Answer:** Controls process CPU scheduling priority (niceness ranges from `-20` highest priority to `19` lowest priority, default `0`).
- `nice -n 10 ./script.sh`
- `renice -n -5 -p <PID>`

### **37. What is `kill` vs `pkill` vs `killall`?**
**Answer:**
- `kill <PID>`: Dispatches a signal to a specific process ID.
- `pkill <pattern>`: Sends signals to processes matching a regex name.
- `killall <name>`: Sends signals to all processes matching exact binary name.

### **38. Explain common Linux Signals (`SIGTERM`, `SIGKILL`, `SIGHUP`, `SIGINT`, `SIGQUIT`).**
**Answer:**
- **`SIGTERM` (15):** Graceful termination request; can be caught and handled.
- **`SIGKILL` (9):** Immediate forceful termination by kernel; **cannot be caught or ignored**.
- **`SIGHUP` (1):** Hangup signal; commonly used to trigger configuration reloads without process restart.
- **`SIGINT` (2):** Interrupt signal sent via `Ctrl+C`.
- **`SIGQUIT` (3):** Quit signal generating a core dump via `Ctrl+\`.

### **39. What is a Core Dump?**
**Answer:** An on-disk snapshot of a process's memory space and register states captured at the exact moment it crashed (e.g., segmentation fault), analyzed using `gdb` for debugging.

### **40. What is `gdb` (GNU Debugger)?**
**Answer:** Interactive debugger for inspecting memory, call stacks, and execution points of C/Go/Rust binary programs and core dumps (`gdb /bin/app core.dump`).

### **41. What is `/etc/resolv.conf`?**
**Answer:** Configuration file for the Linux DNS stub resolver specifying nameservers (`nameserver 8.8.8.8`), search domains, and lookup options (`ndots:5`).

### **42. What is `/etc/hosts`?**
**Answer:** Static local table mapping IP addresses to hostnames evaluated before querying external DNS servers.

### **43. What is `/etc/nsswitch.conf`?**
**Answer:** Name Service Switch configuration defining the lookup precedence for databases (e.g., `hosts: files dns` checks `/etc/hosts` before querying DNS).

### **44. What is `systemd-resolved`?**
**Answer:** A system service providing local network name resolution, DNS caching, and LLMNR/DNS-over-TLS resolving.

### **45. What is `iptables` Architecture (Tables, Chains, Targets)?**
**Answer:**
- **Tables:** `filter` (packet filtering), `nat` (address translation), `mangle` (header modification), `raw`.
- **Chains:** `PREROUTING`, `INPUT`, `FORWARD`, `OUTPUT`, `POSTROUTING`.
- **Targets:** `ACCEPT`, `DROP`, `REJECT`, `SNAT`, `DNAT`, `MASQUERADE`.

### **46. What is `nftables`?**
**Answer:** The modern framework replacing `iptables`, `ip6tables`, and `arptables` with a unified syntax, single CLI (`nft`), and kernel bytecode engine.

### **47. What is `ufw` (Uncomplicated Firewall) vs `firewalld`?**
**Answer:**
- **`ufw` (Ubuntu/Debian):** Simplified CLI frontend for managing iptables/nftables firewall rules.
- **`firewalld` (RHEL/CentOS):** Dynamic zone-based firewall daemon supporting runtime rule changes without dropping active connections.

### **48. What is `tcpdump` and how do you capture packets?**
**Answer:** Command-line packet analyzer:
```bash
tcpdump -i eth0 -nn -s0 -w capture.pcap port 443
```
- `-i eth0`: Interface.
- `-nn`: Do not resolve hostnames or ports.
- `-s0`: Capture full packet payload.
- `-w file.pcap`: Write to Wireshark PCAP file.

### **49. What is `iperf3`?**
**Answer:** Network bandwidth testing tool measuring maximum achievable TCP/UDP throughput and packet loss between client and server.

### **50. What is `traceroute` vs `mtr`?**
**Answer:**
- **`traceroute`:** Sends packets with incrementing TTLs to identify each hop in a network route.
- **`mtr` (My Traceroute):** Combines `traceroute` and `ping` into a live real-time network path diagnostics tool displaying packet loss and latency per hop.

### **51. What is `/etc/security/limits.conf` (PAM Limits)?**
**Answer:** Sets user and group resource limits (max open file descriptors `nofile`, max user processes `nproc`).

### **52. What is `chroot` Jail?**
**Answer:** Changes the perceived root directory (`/`) for a running process, isolating it from the rest of the host filesystem (precursor to Linux containers).

### **53. What is `systemd-cgls` and `systemd-cgtop`?**
**Answer:**
- **`systemd-cgls`:** Displays the cgroup process hierarchy as a tree.
- **`systemd-cgtop`:** Monitors real-time CPU, memory, and I/O consumption per systemd cgroup service unit.

### **54. What is `sysctl` and `/etc/sysctl.conf`?**
**Answer:** CLI and configuration interface for viewing and modifying Linux kernel runtime parameters at `/proc/sys/` (`sysctl -p` applies changes).

### **55. What is `journalctl -k` (dmesg)?**
**Answer:** Displays kernel ring buffer messages from systemd journal.

### **56. What is `logrotate`?**
**Answer:** Linux utility that automates rotation, compression, and removal of application log files.

### **57. What is `rsyslog` vs `systemd-journald`?**
**Answer:**
- **`systemd-journald`:** Captures and stores binary structured logs for systemd units.
- **`rsyslog`:** Traditional syslog daemon that receives logs from journald and formats/routes them to text files (`/var/log/syslog`) or remote syslog servers.

### **58. What is `auditd` (Linux Audit Daemon)?**
**Answer:** Kernel auditing subsystem logging security events (file access, permission modifications, system calls) for compliance and intrusion detection.

### **59. What is `cron` vs `systemd` Timers?**
**Answer:** systemd timers provide sub-second accuracy, dependency management, execution logging in journald, and monotonic timers (e.g., run 10 minutes after boot).

### **60. What is `sudo` vs `su`?**
**Answer:**
- **`su` (Switch User):** Switches user session, requiring the target user's password.
- **`sudo` (SuperUser Do):** Executes commands with elevated privileges, requiring the calling user's password, authenticated against `/etc/sudoers`.

---

## 🟡 **Part 2: Kernel Tuning, Sockets & Performance Troubleshooting (Questions 61–130)**

### **61. How do you tune the Linux Kernel for a High-Throughput Web Server (100k Concurrent Connections)?**
**Answer:**
Configure `/etc/sysctl.conf`:
```ini
fs.file-max = 2097152
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.ip_local_port_range = 1024 65535
net.ipv4.tcp_tw_reuse = 1
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr
```

### **62. Explain the TCP 3-Way Handshake at the Linux Socket Layer (`listen()`, `accept()`, `connect()`).**
**Answer:**
1. Server creates socket, calls `bind()`, and enters listening mode with `listen()`, initializing the **SYN Queue** (`tcp_max_syn_backlog`) and **Accept Queue** (`somaxconn`).
2. Client calls `connect()`, sending a `SYN` packet.
3. Server kernel places connection in `SYN_RECV` state inside the SYN Queue and returns `SYN-ACK`.
4. Client kernel returns `ACK` (`ESTABLISHED` state).
5. Server kernel moves connection to the Accept Queue.
6. Server application calls `accept()` to pull the established socket from the Accept Queue and begin data I/O.

### **63. What is TCP Accept Queue Overflow and how do you detect it?**
**Answer:** When the application process is slow in calling `accept()`, the Accept Queue (`somaxconn`) fills up. The Linux kernel silently drops incoming client `ACK` packets.
- **Detection:** `netstat -s | grep -i "listenqueue"` or `ss -lnt` (look for `Send-Q` matching `Recv-Q`).

### **64. What is `epoll` vs `select` vs `poll` in Linux Socket Programming?**
**Answer:**
- `select` / `poll`: Iterates linearly over all registered file descriptors ($O(N)$), degrading severely at scale.
- `epoll` (`epoll_create`, `epoll_ctl`, `epoll_wait`): Uses kernel callbacks and a ready list scaling $O(1)$, notifying the application *only* of the specific file descriptors that have ready I/O events.

### **65. What is Zero-Copy in Linux (`sendfile()` / `splice()`)?**
**Answer:** Eliminates CPU data copying between kernel space and user space. `sendfile()` streams data directly from disk page cache to the network socket buffer in kernel space, achieving line-rate throughput with minimal CPU overhead.

### **66. What is Transparent Huge Pages (THP) and why should it be disabled for Redis/MongoDB?**
**Answer:** THP allocates 2MB memory pages instead of standard 4KB pages to reduce TLB misses. However, for fine-grained random memory access databases, THP causes extreme memory allocation latency spikes and write amplification. Disabled via `echo never > /sys/kernel/mm/transparent_hugepage/enabled`.

### **67. What is Dirty Memory (`vm.dirty_ratio` vs `vm.dirty_background_ratio`)?**
**Answer:** Memory pages in page cache that have been modified by applications but not yet flushed to physical disk:
- `vm.dirty_background_ratio` (default: 10%): Kernel flushes dirty pages asynchronously in the background.
- `vm.dirty_ratio` (default: 20%): Hard limit where application write operations are blocked and forced to flush dirty pages synchronously to disk.

### **68. What is Linux Kernel Ring Buffer (`dmesg`)?**
**Answer:** A fixed-size circular buffer in kernel memory where kernel drivers, hardware alerts, OOM-killer events, and segfaults are logged.

### **69. What is eBPF (Extended Berkeley Packet Filter) in Linux Systems?**
**Answer:** An in-kernel virtual machine that executes verified, sandboxed bytecode at Linux kernel tracepoints, kprobes, and socket layers without modifying kernel source code or loading unstable third-party kernel modules.

### **70. What is `perf` for Linux Performance Profiling?**
**Answer:** The official Linux profiling tool analyzing hardware performance counters, CPU instruction cycles, cache misses, and kernel tracepoints to generate Flame Graphs.

### **71. Scenario: A Linux server load average is 45.0 on an 8-core CPU, but CPU utilization is only 15%. What is the bottleneck and how do you diagnose it?**
**Answer:**
- **Cause:** High load average with low CPU utilization indicates processes are blocked in **Uninterruptible Sleep (`D` state)** waiting on slow **Disk I/O** or a hanging **Network File System (NFS)** mount.
- **Diagnosis:**
  1. Check CPU I/O Wait in `top` (`%wa`).
  2. Check disk device saturation in `iostat -xz 1 5` (look for `%util` approaching 100%).
  3. Find processes in `D` state: `ps aux | awk '$8 ~ /D/'`.
  4. Inspect what syscalls are blocked: `cat /proc/<PID>/stack`.

### **72. Scenario: A critical process crashes with "Too many open files" (`EMFILE`). How do you fix it permanently?**
**Answer:**
1. Check current process file limits: `cat /proc/<PID>/limits`.
2. Check system-wide file allocation: `cat /proc/sys/fs/file-nr`.
3. Fix for systemd service: Add `LimitNOFILE=65535` under `[Service]` in `/etc/systemd/system/<service>.service` and run `systemctl daemon-reload && systemctl restart <service>`.
4. Fix for interactive users: Edit `/etc/security/limits.conf`:
   ```ini
   * soft nofile 65535
   * hard nofile 65535
   ```

### **73. Scenario: A deleted log file is still consuming 100GB of disk space in `df -h`. Why and how do you reclaim the space immediately?**
**Answer:**
- **Why:** An active running process (e.g., Java / Nginx) still holds an open file descriptor pointing to the deleted file inode. The OS cannot free data blocks until all file descriptors close.
- **Identify Process:** `lsof | grep deleted` (find PID and file descriptor number, e.g., FD 4).
- **Reclaim Space Immediately:** Truncate the file descriptor to zero bytes without restarting the process:
  ```bash
  > /proc/<PID>/fd/<FD_NUM>
  ```

### **74. Scenario: A multi-threaded application suffers from extreme memory fragmentation and allocation latency. How do you replace glibc malloc with `jemalloc` or `tcmalloc`?**
**Answer:**
Inject high-performance multi-threaded allocators via dynamic linker preload:
```bash
LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libjemalloc.so ./my_application
```
`jemalloc` uses thread-specific memory arenas, eliminating mutex contention during high concurrency memory allocations.

### **75. Scenario: A Linux server becomes completely unresponsive over SSH due to memory exhaustion. How do you trigger Magic SysRq keys?**
**Answer:**
Send kernel commands directly via hardware console or cloud serial console:
- **`R-E-I-S-U-B`:**
  - `r`: Put keyboard in raw mode.
  - `e`: Send `SIGTERM` to all processes.
  - `i`: Send `SIGKILL` to all processes.
  - `s`: Sync all dirty filesystems to disk.
  - `u`: Remount all filesystems as read-only.
  - `b`: Reboot the system safely without filesystem corruption.

### **76. What is NUMA (Non-Uniform Memory Access) and NUMA Node Balancing?**
**Answer:** Multi-socket server architecture where each CPU socket has dedicated local memory channels. Accessing remote memory across interconnects incurs latency penalties. Tuned using `numactl --interleave=all` or pinning processes to local NUMA nodes (`numactl --cpunodebind=0`).

### **77. What is Cgroup v2 Unified Hierarchy (`/sys/fs/cgroup`)?**
**Answer:** Replaces separate v1 controllers with a unified process tree, providing accurate memory writeback accounting, memory pressure stalls (PSI), and rootless delegation.

### **78. What is Pressure Stall Information (PSI) in Linux?**
**Answer:** Kernel metric measuring resource starvation across CPU, memory, and I/O (`/proc/pressure/memory`), quantifying the percentage of time tasks were stalled waiting for resources.

### **79. What is Linux Capabilities Table (`getpcaps`, `setcap`)?**
**Answer:** Viewing and assigning granular capabilities to executable files (e.g., `setcap 'cap_net_bind_service=+ep' /usr/bin/node` allows binding to port 80 without root).

### **80. What is SELinux Permissive vs Enforcing Mode?**
**Answer:**
- **Enforcing:** Blocks access and logs violations.
- **Permissive:** Allows access but logs violations for policy debugging (`audit2allow`).

### **81. What is Linux Network Interface Bonding (LACP / Mode 4)?**
**Answer:** Aggregating multiple physical network interfaces into a single virtual interface for bandwidth scaling and automatic failover using IEEE 802.3ad.

### **82. What is Linux VLAN Tagging (`802.1Q`)?**
**Answer:** Creating sub-interfaces (e.g., `eth0.100`) to tag outgoing Ethernet frames with a 12-bit VLAN identifier.

### **83. What is Linux Generic Segmentation Offload (GSO)?**
**Answer:** Delays packet segmentation until the packet reaches the network interface driver, reducing CPU processing overhead.

### **84. What is Linux Large Receive Offload (LRO) / GRO?**
**Answer:** Reassembles consecutive incoming TCP packets into a single large buffer before passing them up the TCP/IP stack.

### **85. What is Linux Kernel Module Blacklisting (`/etc/modprobe.d/blacklist.conf`)?**
**Answer:** Preventing specific kernel drivers (e.g., `nouveau`, `floppy`) from loading automatically on boot.

### **86. What is `lsmod` and `modprobe`?**
**Answer:**
- `lsmod`: Lists currently loaded kernel modules.
- `modprobe <module>`: Dynamically loads or unloads a module along with its dependencies.

### **87. What is `dkms` (Dynamic Kernel Module Support)?**
**Answer:** Automatically recompiles third-party kernel modules (NVIDIA drivers, ZFS) whenever a new Linux kernel update is installed.

### **88. What is Linux Kernel Panic and Kdump?**
**Answer:** A fatal unrecoverable system error. Kdump uses `kexec` to boot a secondary recovery kernel and capture the crashed kernel memory dump (`vmcore`) to disk for crash analysis.

### **89. What is Linux Swap Partition vs Swap File?**
**Answer:** A swap partition is a dedicated raw block partition; a swap file is a pre-allocated file on an existing filesystem (`dd if=/dev/zero of=/swapfile bs=1M count=4096`). Both offer identical performance on modern Linux kernels.

### **90. What is Linux Inode Exhaustion Mitigation?**
**Answer:** Find directories containing millions of small files:
```bash
find / -xdev -printf '%h\n' | sort | uniq -c | sort -k1 -n
```

### **91. What is `taskset` for CPU Pinning (CPU Affinity)?**
**Answer:** Restricting a process to execute strictly on designated CPU cores (`taskset -c 0,1 ./app`) to maximize CPU L1/L2 cache hits.

### **92. What is `chrt` for Real-Time Scheduling?**
**Answer:** Sets real-time scheduling policies (`SCHED_FIFO`, `SCHED_RR`) and priorities for latency-critical Linux processes.

### **93. What is Linux CFS Bandwidth Control (`cpu.cfs_quota_us`)?**
**Answer:** The cgroup mechanism enforcing hard container CPU limits over a quota period (usually 100,000 microseconds).

### **94. What is Linux Memory Ballooning in VMs?**
**Answer:** A hypervisor driver that expands inside the guest OS memory space to reclaim physical RAM from idle VMs.

### **95. What is `blkdiscard` / `fstrim` (SSD TRIM)?**
**Answer:** Informs underlying SSD flash storage controllers which data blocks are no longer in use, maintaining write performance.

### **96. What is RAID 0, 1, 5, 10 in Linux (`mdadm`)?**
**Answer:**
- **RAID 0:** Striping (high speed, zero fault tolerance).
- **RAID 1:** Mirroring (high redundancy, 50% capacity).
- **RAID 5:** Striping with distributed parity (survives 1 drive failure).
- **RAID 10:** Mirrored stripe sets (high speed and redundancy).

### **97. What is ZFS Zpool vs Dataset?**
**Answer:**
- **Zpool:** Storage pool aggregating raw disks with software RAID-Z.
- **Dataset:** Virtual filesystem created within a Zpool supporting snapshotting, compression, and quotas.

### **98. What is Linux Epoll Starvation?**
**Answer:** When an epoll worker loop gets stuck processing high-volume I/O on a single socket, starving other active sockets of CPU attention.

### **99. What is Linux Socket Buffer Autotuning?**
**Answer:** The Linux kernel dynamically adjusts TCP receive and transmit buffers based on the calculated Bandwidth-Delay Product (BDP).

### **100. What is an Enterprise Linux System Hardening Checklist?**
**Answer:**
1. Disable SSH root login and password authentication (enforce SSH keys).
2. Configure automated security patching (`unattended-upgrades`).
3. Enable and enforce SELinux / AppArmor in enforcing mode.
4. Minimize open ports and enforce host firewalls (ufw/firewalld).
5. Enforce auditd logging for privileged system calls.
6. Disable unused filesystems (cramfs, freevxfs, jffs2, hfs).

---

## 🔴 **Part 3: Advanced Linux Internals, Storage & Low-Level Troubleshooting (Questions 101–200)**

### **101. What is `/etc/shadow` vs `/etc/passwd`?**
**Answer:** `/etc/passwd` contains general user metadata (username, UID, GID, home dir, shell) and is world-readable; `/etc/shadow` contains cryptographic password hashes and account expiration policies, readable strictly by root.

### **102. What is `/etc/group` and `/etc/gshadow`?**
**Answer:** `/etc/group` defines user group memberships; `/etc/gshadow` holds secure group passwords and administrators.

### **103. What is Pluggable Authentication Modules (PAM)?**
**Answer:** A flexible mechanism for authenticating users across Linux services (`/etc/pam.d/`), allowing dynamic integration of MFA, LDAP, and password complexity rules.

### **104. What is Linux Name Service Caching Daemon (`nscd`)?**
**Answer:** Caches lookups for name service databases (`passwd`, `group`, `hosts`) to reduce load on LDAP and NIS servers.

### **105. What is `glibc` and what is `musl libc`?**
**Answer:** Standard C library implementations providing core system calls and basic facilities:
- `glibc` (GNU C Library): Feature-complete, standard on Debian/Ubuntu/RHEL.
- `musl libc`: Lightweight, minimal footprint, standard on Alpine Linux.

### **106. What is Linux Dynamic Linker (`ld.so` / `ld-linux.so`)?**
**Answer:** Locates and loads shared libraries (`.so`) needed by an executable into memory at runtime (`ldd binary` lists dependencies).

### **107. What is `/etc/ld.so.conf` and `ldconfig`?**
**Answer:** `/etc/ld.so.conf` lists directories containing shared libraries; `ldconfig` builds the binary search cache (`/etc/ld.so.cache`).

### **108. What is `chattr` and the Immutable Bit (`chattr +i`)?**
**Answer:** Sets filesystem attributes: `chattr +i file.txt` makes a file **immutable** (cannot be modified, deleted, renamed, or linked, even by the root user).

### **109. What is `lsattr`?**
**Answer:** Lists second extended filesystem attributes (like immutable `i` or append-only `a`) of files.

### **110. What is Linux File System Quota (`edquota`, `repquota`)?**
**Answer:** Restricts the amount of disk space (blocks) and number of inodes that individual users or groups can consume on a filesystem.

### **111. What is `/dev/urandom` vs `/dev/random`?**
**Answer:**
- `/dev/random`: Blocks when kernel entropy pool is low.
- `/dev/urandom`: Non-blocking cryptographically secure pseudo-random number generator, safe for all standard cryptographic purposes.

### **112. What is Linux Kernel Entropy Pool?**
**Answer:** A collection of hardware noise (keyboard clicks, mouse movements, disk seek times, hardware RNG) used to generate random numbers.

### **113. What is `tune2fs` in ext4?**
**Answer:** Adjusts tunable filesystem parameters (reserved block percentage, maximum mount count between fscks, volume labels) on ext2/ext3/ext4 filesystems.

### **114. What is `dumpe2fs`?**
**Answer:** Prints detailed superblock and block group metadata for ext4 filesystems.

### **115. What is `xfs_info` and `xfs_repair`?**
**Answer:**
- `xfs_info`: Displays geometry and allocation group details of XFS filesystems.
- `xfs_repair`: Checks and repairs damaged XFS filesystems (must be run on unmounted filesystems).

### **116. What is `fsck` (File System Consistency Check)?**
**Answer:** Verifies the integrity of filesystems and repairs corrupted inode tables and unlinked data blocks.

### **117. What is Linux Block Device Major and Minor Numbers?**
**Answer:**
- **Major Number:** Identifies the device driver associated with the device (e.g., 8 for SCSI/SATA disks).
- **Minor Number:** Identifies the specific physical device or partition managed by that driver (e.g., 0 for `sda`, 1 for `sda1`).

### **118. What is `udev` and `/etc/udev/rules.d/`?**
**Answer:** The Linux kernel device manager that dynamically creates device nodes in `/dev/` when hardware is connected and executes custom rule scripts.

### **119. What is `lsblk` and `blkid`?**
**Answer:**
- `lsblk`: Displays visual tree of all block storage devices and mount points.
- `blkid`: Prints unique UUIDs and filesystem formats of block devices.

### **120. What is `hdparm` and `smartctl`?**
**Answer:**
- `hdparm`: Tunes and tests SATA/IDE disk read speeds.
- `smartctl`: Queries S.M.A.R.T. disk telemetry to predict physical drive failures (bad sectors, temperature, reallocated sectors).

### **121. What is Linux NVMe CLI (`nvme-cli`)?**
**Answer:** Utility for interacting with high-performance NVMe SSDs (formatting, firmware updates, namespace creation, error logs).

### **122. What is I/O Scheduler in Linux (mq-deadline, kyber, bfq, none)?**
**Answer:** Determines the order in which block I/O operations are dispatched to storage hardware:
- `none`: Optimal for NVMe SSDs (delegates scheduling to fast hardware controller).
- `mq-deadline`: Simple, deterministic deadline scheduler for SATA SSDs.
- `bfq` (Budget Fair Queueing): Desktop interactive responsiveness.

### **123. What is `ionice`?**
**Answer:** Sets process I/O scheduling class and priority (`Idle`, `Best-effort`, `Real-time`): `ionice -c 3 ./backup.sh` runs disk backups only when disk is idle.

### **124. What is Linux Direct I/O (`O_DIRECT`)?**
**Answer:** A file open flag that bypasses the OS page cache entirely, reading and writing directly between userspace memory buffers and storage devices (used by databases to avoid double caching).

### **125. What is Asynchronous I/O (AIO) vs `io_uring`?**
**Answer:**
- **AIO:** Legacy asynchronous kernel I/O interface; limited to Direct I/O on block devices.
- **`io_uring`:** High-performance Linux 5.1+ asynchronous interface using shared lockless ring buffers between kernel and userspace, supporting all file and network I/O with zero syscall overhead.

### **126. What is `kswapd` Daemon?**
**Answer:** The background kernel thread that monitors memory watermarks and reclaims page cache or swaps memory pages when free memory drops below `pages_low`.

### **127. What is Memory Compaction in Linux?**
**Answer:** Rearranges allocated memory pages to create contiguous blocks of free physical memory, satisfying high-order allocation requests (HugePages).

### **128. What is SLAB, SLUB, and SLOB Allocator?**
**Answer:** Kernel memory allocators managing small, frequently used kernel object caches (inodes, dentry caches):
- `SLUB`: Modern default scalable allocator with minimal metadata overhead.

### **129. What is `/proc/meminfo` and what do `MemAvailable`, `Buffers`, `Cached`, `Dirty` mean?**
**Answer:**
- `MemAvailable`: Estimated memory available to launch new applications without swapping (includes reclaimable caches).
- `Buffers`: In-memory raw disk block metadata cache.
- `Cached`: Page cache holding files read from disk.
- `Dirty`: Modified pages waiting to be written to disk.

### **130. What is `/proc/slabinfo` and `slabtop`?**
**Answer:** Displays real-time kernel slab memory cache consumption per object type (dentry, inode_cache).

### **131. What is Linux Dentry Cache (Directory Entry Cache)?**
**Answer:** In-memory kernel cache mapping directory path lookups (`/var/log/nginx/access.log`) to inode numbers, accelerating file lookups.

### **132. What is `drop_caches` (`/proc/sys/vm/drop_caches`)?**
**Answer:** Forces the kernel to immediately clean and free non-dirty page cache, dentries, and inodes: `echo 3 > /proc/sys/vm/drop_caches`.

### **133. What is Linux HugePages (`vm.nr_hugepages`)?**
**Answer:** Pre-allocating dedicated 2MB or 1GB static memory blocks in physical RAM, locked against swapping (used for Oracle, PostgreSQL, and DPDK).

### **134. What is POSIX Shared Memory (`shm_open`, `/dev/shm`)?**
**Answer:** Inter-process communication mechanism allowing multiple independent processes to map the exact same physical RAM segment into their address spaces for microsecond data exchange.

### **135. What is Memory-Mapped Files (`mmap`)?**
**Answer:** Maps a file directly into a process's virtual address space; reading or writing to memory automatically triggers page cache reads and lazy writes to disk.

### **136. What is Copy-on-Write (CoW) in Process Forking (`fork()`)?**
**Answer:** When `fork()` spawns a child process, the child shares the parent's physical memory pages marked read-only. Memory pages are duplicated only when parent or child writes to a page.

### **137. What is `vfork()` vs `fork()` vs `clone()`?**
**Answer:**
- `fork()`: Creates child process with CoW virtual memory.
- `vfork()`: Suspends parent; child borrows parent address space until `exec()`.
- `clone()`: Underlying syscall allowing fine-grained sharing of namespaces, memory, and file descriptors (used to create both threads and containers).

### **138. What is a POSIX Thread (Pthread) vs Linux Process?**
**Answer:** In the Linux kernel, both processes and threads are represented by the exact same data structure (`task_struct`). Threads are simply `clone()` tasks that share the exact same virtual memory space (`CLONE_VM`) and file descriptor table (`CLONE_FILES`).

### **139. What is Linux Thread-Local Storage (TLS)?**
**Answer:** A dedicated memory area where each thread in a multi-threaded process maintains its own private instance of static or global variables.

### **140. What is Context Switch (Voluntary vs Involuntary)?**
**Answer:**
- **Voluntary:** Process yields CPU while waiting for I/O (`sleep()`, network read).
- **Involuntary:** Kernel scheduler preempts process when its CPU time slice expires or a higher-priority task wakes up.

### **141. What is `/proc/cpuinfo`?**
**Answer:** Contains processor architecture, core counts, cache sizes (L1/L2/L3), CPU flags (`sse4_2`, `avx512`), and hardware vulnerability mitigations (Spectre/Meltdown).

### **142. What is CPU Governor (`cpufreq`)?**
**Answer:** Controls dynamic CPU clock frequency scaling: `performance` (maximum clock speed), `powersave`, `ondemand`.

### **143. What is CPU C-States vs P-States?**
**Answer:**
- **C-States:** Idle power-saving states (C0 = active; C1-C6 = sleeping with powered-down CPU components).
- **P-States:** Active operational performance frequency/voltage states (P0 = maximum turbo frequency).

### **144. What is Interrupt Request (IRQ) and `/proc/interrupts`?**
**Answer:** Hardware signals sent to the CPU when devices (NIC, disk) require attention. `/proc/interrupts` shows per-CPU interrupt counts.

### **145. What is SMP IRQ Affinity (`/proc/irq/<N>/smp_affinity`)?**
**Answer:** Binding network card hardware interrupts to specific CPU cores to prevent interrupt processing contention across cores.

### **146. What is SoftIRQ (Software Interrupt)?**
**Answer:** Deferred kernel interrupt handlers running in `ksoftirqd/N` kernel threads that process heavy tasks (network packet reception) scheduled by fast hardware IRQs.

### **147. What is Network RX / TX Ring Buffer (`ethtool -g`)?**
**Answer:** Circular memory buffers on physical NICs that queue incoming (RX) and outgoing (TX) packets before the kernel driver processes them. Increasing ring buffer size prevents packet drops during traffic bursts.

### **148. What is `ethtool` for Network Interface Tuning?**
**Answer:** CLI tool to view and configure network card driver parameters, link speeds, auto-negotiation, and hardware offloading features.

### **149. What is MTU Mismatch and IP Fragmentation?**
**Answer:** If a packet exceeds the MTU of an intermediate router and the `DF` (Don't Fragment) bit is set, the router drops the packet and returns an ICMP "Fragmentation Needed" message. If ICMP is blocked, the connection hangs (Black Hole).

### **150. What is Network Namespace (`ip netns`)?**
**Answer:** Isolates physical/virtual network devices, IP routing tables, firewall rules, and port bindings into independent network sandboxes.

### **151. What is Linux Veth (Virtual Ethernet) Device?**
**Answer:** A virtual point-to-point network pipe acting as a bi-directional tunnel between two network namespaces.

### **152. What is Linux Bridge (`ip link add br0 type bridge`)?**
**Answer:** Software-defined Layer 2 switch forwarding Ethernet frames between attached interfaces based on MAC address tables.

### **153. What is Linux MACVLAN vs IPVLAN?**
**Answer:**
- **MACVLAN:** Assigns a unique MAC address and IP to each sub-interface on top of a single physical NIC.
- **IPVLAN:** Multiple virtual interfaces share the exact same physical MAC address but have unique IP addresses.

### **154. What is Linux TUN / TAP Device?**
**Answer:** Virtual software network devices:
- **TUN (Layer 3):** Simulates network-layer devices and processes raw IP packets (used by OpenVPN, WireGuard).
- **TAP (Layer 2):** Simulates Ethernet devices and processes raw Ethernet frames (used by QEMU VMs).

### **155. What is Linux Dynamic Routing with FRRouting (FRR)?**
**Answer:** Open-source IP routing protocol suite providing BGP, OSPF, and IS-IS daemon implementations for Linux servers.

### **156. What is Linux TCP SYN Cookies Internal Algorithm?**
**Answer:** When the SYN queue is full, the kernel computes the initial sequence number of `SYN-ACK` as a cryptographic hash of (Client IP, Server IP, Client Port, Server Port, Timestamp, MSS). The connection is instantiated only upon receiving the client `ACK`.

### **157. What is Linux TCP Fast Open (`net.ipv4.tcp_fastopen=3`)?**
**Answer:** Enables data exchange directly within the initial TCP `SYN` packet, reducing handshake latency for returning clients.

### **158. What is Linux TCP SACK and D-SACK?**
**Answer:** Selective Acknowledgment allows receivers to acknowledge out-of-order packets; Duplicate SACK informs senders of duplicate packet receptions.

### **159. What is Linux TCP CUBIC vs BBR Congestion Control?**
**Answer:**
- **CUBIC:** Standard loss-based congestion control; scales back window aggressively on packet drops.
- **BBR:** Bottleneck Bandwidth and Round-trip propagation time model; maximizes bandwidth and minimizes packet queueing delays.

### **160. What is Linux Socket Option `SO_BINDTODEVICE`?**
**Answer:** Binds a network socket strictly to a specific physical network interface (e.g., `eth1`), routing all traffic through that device regardless of system route tables.

### **161. What is Linux Socket Option `TCP_NODELAY` (Disabling Nagle's Algorithm)?**
**Answer:** Sends packets immediately without buffering small payloads to wait for ACK acknowledgments, mandatory for low-latency RPC and gaming.

### **162. What is Linux Socket Option `TCP_CORK`?**
**Answer:** Buffers and merges partial packets into full MSS frames before transmitting over the network, maximizing throughput for file transfers.

### **163. What is `/proc/net/dev`?**
**Answer:** Displays real-time byte and packet transmission/reception statistics and error drop counts for all network interfaces.

### **164. What is `/proc/net/snmp` and `/proc/net/netstat`?**
**Answer:** System-wide TCP/UDP/IP protocol telemetry (TCP retransmissions, resets, connection aborts, listen queue overflows).

### **165. What is Linux Kernel Module Signature Verification (`CONFIG_MODULE_SIG`)?**
**Answer:** Enforces that the kernel will only load kernel modules (`.ko`) that have been cryptographically signed with a trusted private key, blocking malicious rootkits.

### **166. What is Kernel Lockdown Mode (`integrity` vs `confidentiality`)?**
**Answer:** Restricts even the root user from modifying running kernel memory, accessing raw PCIe/MSR registers, or reading kernel memory via `/dev/mem` when UEFI Secure Boot is active.

### **167. What is Linux Kernel Address Space Layout Randomization (KASLR)?**
**Answer:** Randomizes the memory location of the kernel code and data structures at boot time, mitigating kernel exploit vulnerabilities.

### **168. What is Linux Seccomp-BPF Syscall Filtering Architecture?**
**Answer:** Compiles system call filter rules into Berkeley Packet Filter bytecode evaluated inside the kernel before each syscall executes.

### **169. What is Linux Landlock LSM (Linux Security Module)?**
**Answer:** Unprivileged application sandboxing module allowing programs to restrict their own filesystem and network access without requiring root permissions.

### **170. What is Linux IMA (Integrity Measurement Architecture)?**
**Answer:** Measures and verifies the cryptographic SHA256 hashes of executable files and kernel modules before allowing execution.

### **171. What is Linux TPM 2.0 (Trusted Platform Module) and Measured Boot?**
**Answer:** Hardware security chip storing cryptographic measurements (PCR hashes) of each boot stage (UEFI, GRUB, Kernel), sealing disk encryption keys against tampering.

### **172. What is LUKS (Linux Unified Key Setup) and `cryptsetup`?**
**Answer:** Standard disk encryption specification providing full-disk block encryption using AES-XTS.

### **173. What is Clevis and Tang for Network-Bound Disk Encryption (NBDE)?**
**Answer:** Automates unlocking LUKS-encrypted root disks on boot over a trusted private network without human passphrase entry.

### **174. What is Systemd Service Sandboxing Directives?**
**Answer:** Hardening directives inside unit files:
```ini
[Service]
ProtectSystem=strict
ProtectHome=true
PrivateTmp=true
NoNewPrivileges=true
ProtectKernelTunables=true
RestrictAddressFamilies=AF_INET AF_INET6
```

### **175. What is Systemd Socket Activation?**
**Answer:** Systemd listens on network sockets on behalf of services. When a client connects, systemd spawns the service on-demand and hands over the established socket, enabling zero idle memory usage.

### **176. What is Systemd Slice (`*.slice`) and Resource Control?**
**Answer:** Groups service units into cgroup resource allocation slices (`system.slice`, `user.slice`) with hard memory and CPU limits (`MemoryMax=2G`, `CPUQuota=200%`).

### **177. What is Systemd Transient Units (`systemd-run`)?**
**Answer:** Spawns ad-hoc, isolated background commands as tracked systemd services with explicit cgroup resource limits.

### **178. What is Systemd Journal Remote (`systemd-journal-remote`)?**
**Answer:** Streams binary structured systemd journal logs securely over HTTPS to a centralized remote log collector server.

### **179. What is Linux Sysctl `kernel.pid_max`?**
**Answer:** The maximum number of process IDs the Linux kernel can allocate (default: 32768 on 32-bit; can be raised to 4 million on 64-bit systems to prevent PID exhaustion).

### **180. What is Linux Sysctl `fs.inotify.max_user_watches`?**
**Answer:** Limits the maximum number of filesystem directories a user process (IDE, build tool, Prometheus) can monitor for file changes.

### **181. What is Linux Sysctl `net.ipv4.ip_local_port_range`?**
**Answer:** Defines the range of ephemeral client ports used for outbound TCP/UDP connections (default: `32768 60999`; tuned to `1024 65535` for proxies).

### **182. What is Linux Sysctl `net.ipv4.tcp_fin_timeout`?**
**Answer:** The duration a TCP socket remains in the `FIN_WAIT_2` state before being forcibly closed by the kernel (default: 60s; tuned to 15s to reclaim sockets).

### **183. What is Linux Sysctl `vm.overcommit_memory` (0, 1, 2)?**
**Answer:**
- `0`: Heuristic overcommit (kernel estimates available RAM).
- `1`: Always overcommit memory (standard for Redis).
- `2`: Strict non-overcommit; allocations cannot exceed `Swap + (RAM * vm.overcommit_ratio)`.

### **184. What is Linux Sysctl `vm.panic_on_oom`?**
**Answer:**
- `0`: Kernel OOM Killer terminates offending processes.
- `1`: Kernel immediately panics and reboots when memory is exhausted (used in clustered HA nodes to trigger hardware failover).

### **185. What is Linux Sysctl `kernel.hung_task_timeout_secs`?**
**Answer:** The duration a process can remain in uninterruptible sleep (`D` state) before the kernel logs a warning stack trace to dmesg (default: 120s).

### **186. What is Linux Sysctl `fs.file-nr`?**
**Answer:** Displays three values: allocated file handles, allocated unused file handles, and maximum file handles system-wide.

### **187. What is Linux File Descriptor Leaks and how are they detected?**
**Answer:** When applications open files/sockets without calling `close()`. Detected via `ls -l /proc/<PID>/fd | wc -l` or monitoring Prometheus metric `process_open_fds`.

### **188. What is Linux Thread Leaks and how are they diagnosed?**
**Answer:** When applications spawn worker threads without joining or terminating them. Diagnosed via `ps -o nlwp <PID>` (number of lightweight processes).

### **189. What is Linux Memory Leak vs Memory Fragmentation?**
**Answer:**
- **Leak:** Application allocates memory and loses reference pointers, preventing garbage collection or freeing.
- **Fragmentation:** Total free memory is sufficient, but divided into small non-contiguous blocks, preventing large memory allocations.

### **190. What is `valgrind` (Memcheck)?**
**Answer:** Memory debugging tool that executes compiled binaries to detect memory leaks, buffer overflows, and uninitialized variable reads.

### **191. What is `bpftrace` for High-Level Dynamic Tracing?**
**Answer:** High-level tracing language for Linux eBPF allowing one-liner scripts to inspect function arguments, latencies, and syscall counts in live production kernels.

### **192. What is `bcc-tools` (BFF Profiling Tools)?**
**Answer:** Collection of specialized eBPF diagnostics scripts (`execsnoop`, `opensnoop`, `biolatency`, `tcpconnect`, `tcpretrans`).

### **193. What is `sysstat` Package Architecture?**
**Answer:** Automated daemon (`sysstat.service`) collecting SAR binary activity data every 10 minutes stored in `/var/log/sysstat/saDD`.

### **194. What is Linux `nfsstat`?**
**Answer:** Displays NFS client and server statistics (RPC calls, caching hit rates, network timeouts).

### **195. What is Linux Automounter (`autofs`)?**
**Answer:** Daemon that automatically mounts remote network filesystems (NFS/CIFS) on-demand when directories are accessed and unmounts them after inactivity.

### **196. What is Linux Core Dump Configuration (`/proc/sys/kernel/core_pattern`)?**
**Answer:** Template defining the filename and storage path where core dumps are written upon process crashes (`/var/crash/core.%e.%p.%t`).

### **197. What is Linux Network Namespace Exec (`ip netns exec`)?**
**Answer:** Executes a command inside a target network namespace (e.g., `ip netns exec net0 ping 8.8.8.8`).

### **198. What is Linux Kernel Livepatching (kpatch / Canonical Livepatch)?**
**Answer:** Applies critical Linux kernel security patches to memory while the system is running **with zero reboots and zero downtime**.

### **199. What is Linux Rootkit Detection (`rkhunter` / `chkrootkit`)?**
**Answer:** Scans system binaries, kernel modules, and network ports for known rootkits, backdoors, and local exploits.

### **200. What is the Golden Troubleshooting Order for Production Linux Outages?**
**Answer:**
1. **Scope the Outage:** Determine affected users and services.
2. **USE & RED Method Metrics:** Check CPU, Memory, Disk I/O, Network Saturation (`uptime`, `free -h`, `iostat -xz 1`, `ss -tulpn`).
3. **Kernel & Service Logs:** Inspect `dmesg -T` for OOM kills / disk stalls; inspect `journalctl -u <service> -e`.
4. **Deep System Call Tracing:** Use `strace`, `lsof`, or `bpftrace` to identify blocked system calls.
5. **Remediate & Post-Mortem:** Apply mitigation, restore service, and capture telemetry for blameless post-mortem analysis.
