# Multi-Container Runtime

A lightweight Linux container runtime in C with a long-running supervisor and a kernel-space memory monitor.

Read [`project-guide.md`](project-guide.md) for the full project specification.

---
Team details:
| Name           |       SRN      | 
|----------------|----------------|
| Jagriti Mohan  | PES2UG24CS200  | 
| Joel William   | PES2UG24CS205  | 


# Multicontainer Runtime Project  
### Build, Load, Run, and Cleanup Guide (Ubuntu 22.04 / 24.04)

---

## Overview
This project implements a multicontainer runtime consisting of:
- A **kernel module** for low-level container handling  
- A **supervisor** (user-space controller)  
- A **CLI tool** for managing containers  

This guide provides **step-by-step instructions** to reproduce the setup from scratch.

---

##  1. System Requirements

- Ubuntu 22.04 / 24.04
- GCC, Make
- Linux Kernel Headers
- Root (sudo) access

---

### Architecture Overview

The runtime is a single binary (`engine`) used in two ways:

1. **Supervisor daemon** — started once with `engine supervisor ./rootfs-base`. It stays alive, manages containers, and owns the logging pipeline.
2. **CLI client** — each command like `engine start alpha ./rootfs-alpha /bin/sh` is a short-lived process that sends a request to the running supervisor and prints the response.

The CLI process connects to the supervisor over an IPC control channel, sends a command, receives a response, and exits. The supervisor, upon receiving a `start` command, calls `clone()` to create a new container child process with its own namespaces. Each container's stdout and stderr are connected back to the supervisor via pipes, which feed into the bounded-buffer logging pipeline.

There are two separate IPC paths in this project:

- **Path A (logging):** Container stdout/stderr → Supervisor, via pipes. Described in Task 3.
- **Path B (control):** CLI process → Supervisor, via a UNIX domain socket, FIFO, or shared-memory channel. Described in Task 2.

### CLI Contract

Use this exact command interface:

```bash
engine supervisor <base-rootfs>
engine start <id> <container-rootfs> <command> [--soft-mib N] [--hard-mib N] [--nice N]
engine run   <id> <container-rootfs> <command> [--soft-mib N] [--hard-mib N] [--nice N]
engine ps
engine logs <id>
engine stop <id>
```

## What to Do Next

Read [`project-guide.md`](project-guide.md) end to end. It contains:

- The six implementation tasks (multi-container runtime, CLI, logging, kernel monitor, scheduling experiments, cleanup)
- The engineering analysis you must write
- The exact submission requirements, including what your `README.md` must contain (screenshots, analysis, design decisions)

Your fork's `README.md` should be replaced with your own project documentation as described in the submission package section of the project guide. (As in get rid of all the above content and replace with your README.md)
