# Multi-Container Runtime (OS Project)

A lightweight container runtime implemented in C that supports multiple isolated containers using Linux namespaces, along with a supervisor process and a kernel-level memory monitoring module.

---

## 👥 Team Information

| Name          | SRN          |
| ------------- | ------------ |
| Komal S Sajja     | PES1UG24CS231     |
| Kabir Raju G | PES1UG24CS210 |

---

## ⚙️ Setup and Execution Guide

### 🔧 Requirements

* Ubuntu 22.04 / 24.04 (VM recommended)
* Secure Boot disabled
* Kernel headers installed

```bash
sudo apt update
sudo apt install -y build-essential linux-headers-$(uname -r)
```

---

### 📁 Project Setup

```bash
git clone https://github.com/<your-username>/OS-Jackfruit.git
cd OS-Jackfruit
```

Download and extract base filesystem:

```bash
mkdir rootfs-base
wget https://dl-cdn.alpinelinux.org/alpine/v3.20/releases/x86_64/alpine-minirootfs-3.20.3-x86_64.tar.gz
tar -xzf alpine-minirootfs-3.20.3-x86_64.tar.gz -C rootfs-base

cp -a rootfs-base rootfs-alpha
cp -a rootfs-base rootfs-beta
```

---

### 🛠️ Build

```bash
cd boilerplate
make clean
make
```

Copy workloads:

```bash
cp cpu_hog memory_hog io_pulse ../rootfs-alpha/
cp cpu_hog memory_hog io_pulse ../rootfs-beta/
```

---

### 🧠 Load Kernel Module

```bash
sudo insmod monitor.ko
ls /dev/container_monitor
```

(Optional verification)

```bash
dmesg | tail
```

---

### ▶️ Running the System

#### Terminal 1 — Supervisor

```bash
sudo ./engine supervisor ../rootfs-base
```

#### Terminal 2 — Container Commands

Start containers:

```bash
sudo ./engine start alpha ../rootfs-alpha /bin/sh
sudo ./engine start beta  ../rootfs-beta  /bin/sh
```

Check status:

```bash
sudo ./engine ps
```

View logs:

```bash
sudo ./engine logs alpha
cat logs/alpha.log
```

Stop containers:

```bash
sudo ./engine stop alpha
sudo ./engine stop beta
```

---

### 🧹 Cleanup

```bash
sudo rmmod monitor
```

---

## 📸 Demo Screenshots

### 1. Multiple containers running simultaneously

![Multi](screenshots/1-multi-container.jpeg)

---

### 2. Container metadata using `ps`

![Metadata](screenshots/2-metadata.jpeg)

---

### 3. Log output captured via logging pipeline

![Logging](screenshots/3-logging.jpeg)

---

### 4. CLI interaction with supervisor

![CLI](screenshots/4-cli-ipc.jpeg)

---

### 5. Soft memory limit warning

![Soft](screenshots/5-soft-limit.jpeg)

---

### 6. Hard limit enforcement (process killed)

![Hard](screenshots/6-hard-limit.jpeg)

---

### 7. Scheduling experiment using different priorities

Due to the short-lived nature of the workload, the measured execution time primarily reflects process startup overhead rather than scheduling differences. However, the experiment demonstrates the ability to assign different priorities using nice values.

![Scheduling](screenshots/7-scheduling.jpeg)

---

### 8. Proper cleanup and termination

![Cleanup](screenshots/8-cleanup.jpeg)

---

## 🧠 Engineering Concepts

### 🔹 Container Isolation

Containers are created using Linux namespaces (PID, UTS, mount). Each container sees its own process tree and filesystem using `chroot`. `/proc` is mounted inside the container for process visibility.

---

### 🔹 Supervisor Role

A persistent supervisor process manages container lifecycle. It keeps track of all running containers and ensures proper cleanup using `waitpid()` to avoid zombie processes.

---

### 🔹 IPC and Logging

Two communication paths are used:

* Pipes: capture container output
* UNIX socket: send CLI commands

A producer-consumer model is implemented using threads and a bounded buffer to handle logs safely.

---

### 🔹 Memory Monitoring

A kernel module tracks memory usage using RSS.

* Soft limit → warning message
* Hard limit → process termination

Kernel-level enforcement ensures timely and reliable monitoring.

---

### 🔹 Scheduling Behavior

Linux Completely Fair Scheduler (CFS) allocates CPU based on `nice` values. Lower nice values result in higher CPU share.

---

## ⚖️ Design Choices

| Component      | Approach           | Tradeoff                 |
| -------------- | ------------------ | ------------------------ |
| Isolation      | Namespace-based    | No network isolation     |
| Supervisor     | Central controller | Single point of control  |
| Logging        | Threaded buffer    | Slight overhead          |
| Memory control | Kernel module      | Requires root privileges |

---

## 🧾 Conclusion

This project demonstrates how core OS concepts such as process isolation, scheduling, IPC, and memory management work together in a practical container runtime system.
