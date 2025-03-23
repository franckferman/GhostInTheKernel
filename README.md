<div id="top" align="center">

<!-- Shields Header -->
[![Contributors][contributors-shield]](https://github.com/franckferman/GhostInTheKernel/graphs/contributors)
[![Forks][forks-shield]](https://github.com/franckferman/GhostInTheKernel/network/members)
[![Stargazers][stars-shield]](https://github.com/franckferman/GhostInTheKernel/stargazers)
[![License][license-shield]](https://github.com/franckferman/GhostInTheKernel/blob/stable/LICENSE)

<!-- Logo -->
<a href="https://github.com/franckferman/GhostInTheKernel">
  <img src="https://raw.githubusercontent.com/franckferman/GhostInTheKernel/refs/heads/stable/docs/github/graphical_resources/Logo-GhostInTheKernel.png" alt="GhostInTheKernel Logo" width="auto" height="auto">
</a>

<!-- Title & Tagline -->
<h3 align="center">GhostInTheKernel</h3>
<p align="center">
    <em>Ghosts no longer haunt the shell — they reside in the kernel.</em>
    <br>
     A stealth-loaded, signal-driven LKM rootkit for total control beneath the surface.
</p>

</div>

## 📜 Table of Contents

<details open>
  <summary><strong>Click to collapse/expand</strong></summary>
  <ol>
    <li><a href="#-about">📖 About</a></li>
    <li><a href="#-usage">🎮 Usage</a></li>
    <li><a href="#-contributing">🤝 Contributing</a></li>
    <li><a href="#%EF%B8%8F-legal-disclaimer">⚖️ Legal Disclaimer</a></li>
    <li><a href="#-star-evolution">🌠 Star Evolution</a></li>
    <li><a href="#-license">📜 License</a></li>
    <li><a href="#-contact">📞 Contact</a></li>
  </ol>
</details>

## 📖 About

> *"In a system without borders, even the kernel can be haunted."*

**GhostInTheKernel** is a stealthy and modular **Loadable Kernel Module (LKM)** rootkit for Linux, written in C.  
It was designed as a proof-of-concept for advanced kernel-level concealment, command and control (C2), and privilege escalation — all controlled through POSIX signals.

Inspired by cyberpunk philosophy and systems introspection, this project explores how far one can manipulate visibility, authority, and persistence in a *post-userland* environment.

## I – 🧠 Introduction

GhostInTheKernel offers a covert presence inside the Linux kernel.

From syscall hijacking and selective process hiding to persistent module loading and signal-driven privilege escalation, it provides a foundation for red teaming, kernel experimentation, and low-level offensive research.

The rootkit interacts silently with its environment via signals, remaining invisible to common tools like `ps`, `lsmod`, `netstat`, and `dmesg`. Its modular architecture and clean API make it extendable and maintainable — even under scrutiny.

## II – ⚙️ Features

### A. 🧩 Syscall Hooking

GhostInTheKernel hooks a wide array of Linux syscalls across multiple files:

- **File access:** `access`, `faccessat`, `faccessat2`
- **Permissions & ownership:** `chmod`, `chown`, `fchmodat`, `fchownat`, `chdir`, `chroot`
- **Execution:** `execve`, `execveat`, `uselib`
- **Directory listing:** `getdents`, `getdents64`
- **Kernel accounting:** `acct`, `quotactl`, `syslog`
- **Process signals:** `kill`, `tkill`, `tgkill`, `pidfd_open`
- **Module loading:** `init_module`, `finit_module`
- **Mounting:** `mount`, `umount2`, `pivot_root`, `move_mount`, etc.
- **File management:** `open`, `unlink`, `rename`, `mknod`, `mkdir`, `truncate`, etc.

### B. 📡 Signal-Based Communication

| Signal         | Code | Purpose |
|----------------|------|---------|
| `SIGROOT`      | 42   | Grants root privileges to the calling process (with correct PID) |
| `SIGHIDE`      | 43   | Hides the calling process from userland tools |
| `SIGSHOW`      | 44   | Unhides a previously hidden process |
| `SIGAUTH`      | 45   | Marks a process as authorized to bypass hiding |
| `SIGMODHIDE`   | 46   | Removes the rootkit from `/proc/modules` and `dmesg` |
| `SIGMODSHOW`   | 47   | Makes the rootkit module visible again |
| `SIGPORTHIDE`  | 48   | Adds a port to the hidden port list |
| `SIGPORTSHOW`  | 49   | Removes a port from the hidden list |

### C. 🛡️ Process Authorization
The function `is_process_authorized()` uses a linked list (`authorized_pids_list`) to determine which processes can bypass the rootkit's hiding mechanisms.

## III – 👻 Stealth

### A. 📦 Module Hiding
The `hide_module()` function removes the rootkit from kernel module lists and obfuscates any traceable taints or dependencies.

### B. 🗂️ File Hiding
Intercepting `getdents64` allows selective hiding of entries in `/proc/`, including those related to the rootkit itself.

### C. 🌐 Port & Network Hiding
Using net hooks like `tcp4_seq_show_hooked`, the rootkit hides C2-related open ports from `/proc/net` and similar entries.

### D. 🧬 PID Hiding
The list `hidden_pids_list` and the function `is_pid_hidden()` ensure that select processes remain invisible to userland.

## IV – 🔒 Persistence
GhostInTheKernel ensures persistence via:
- Copying the module to `/lib/modules/rootkit_mod.ko`
- Creating a script at `/etc/local.d/rootkit_load.start` with the command `insmod /lib/modules/rootkit_mod.ko`

This is fully compatible with **OpenRC**-based systems.

#### Future Enhancements:

- Support for `systemd` unit persistence
- Firmware-level persistence exploration

## V – 🔼 Privilege Escalation

### A. 🧍 Root Access via Signal
The `SIGROOT` signal triggers `give_root()`, which uses `prepare_creds()` and `commit_creds()` to set UID/GID = 0 for the calling process.

### B. 📤 Module Lifecycle
- On load: `rootkit_init()` installs syscall/net hooks and hides the module
- On unload: `rootkit_exit()` restores everything and clears hidden state

> *"In a system without borders, even the kernel can be haunted."*

<p align="right">(<a href="#top">🔼 Back to top</a>)</p>

## 🎮 Usage

### 🔧 Usage Examples

```bash
kill -42 <pid>   # Gain root privileges
kill -43 <pid>   # Hide a process
kill -44 <pid>   # Unhide a process
kill -45 <pid>   # Authorize a process (bypass hiding)
kill -46 <pid>   # Hide the rootkit module
kill -47 <pid>   # Reveal the rootkit module
kill -48 <pid>   # Hide a port
kill -49 <pid>   # Unhide a port
```

### 📡 Signal Reference

| Signal Command       | Code | Purpose |
|----------------------|------|---------|
| `kill -42 <pid>`     | 42   | Grants root privileges to the calling process (requires authorized PID) |
| `kill -43 <pid>`     | 43   | Hides the specified process |
| `kill -44 <pid>`     | 44   | Reveals the specified hidden process |
| `kill -45 <pid>`     | 45   | Authorizes a process to bypass visibility filters |
| `kill -46 <pid>`     | 46   | Hides the rootkit module from `/proc/modules` and `dmesg` |
| `kill -47 <pid>`     | 47   | Reveals the previously hidden rootkit module |
| `kill -48 <pid>`     | 48   | Adds a TCP/UDP port to the hidden port list |
| `kill -49 <pid>`     | 49   | Removes a port from the hidden list |

<p align="right">(<a href="#top">🔼 Back to top</a>)</p>

## 🤝 Contributing

We truly appreciate and welcome community involvement. Your contributions, feedback, and suggestions play a crucial role in improving the project for everyone. If you're interested in contributing or have ideas for enhancements, please feel free to open an issue or submit a pull request on our GitHub repository. Every contribution, no matter how big or small, is highly valued and greatly appreciated!

<p align="right">(<a href="#top">🔼 Back to top</a>)</p>

## 🌠 Star Evolution

Explore the star history of this project and see how it has evolved over time:

<a href="https://star-history.com/#franckferman/GhostInTheKernel&Timeline">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/svg?repos=franckferman/GhostInTheKernel&type=Timeline&theme=dark" />
    <img alt="Star History Chart" src="https://api.star-history.com/svg?repos=franckferman/GhostInTheKernel&type=Timeline" />
  </picture>
</a>

Your support is greatly appreciated. We're grateful for every star! Your backing fuels our passion. ✨

## 📚 License

This project is licensed under the GNU Affero General Public License, Version 3.0. For more details, please refer to the LICENSE file in the repository: [Read the license on GitHub](https://github.com/franckferman/GhostInTheKernel/blob/stable/LICENSE)

<p align="right">(<a href="#top">🔼 Back to top</a>)</p>

## 📞 Contact

[![ProtonMail][protonmail-shield]](mailto:contact@franckferman.fr)
[![LinkedIn][linkedin-shield]](https://www.linkedin.com/in/franckferman)
[![Twitter][twitter-shield]](https://www.twitter.com/franckferman)

<p align="right">(<a href="#top">🔼 Back to top</a>)</p>

<!-- MARKDOWN LINKS & IMAGES -->
<!-- https://www.markdownguide.org/basic-syntax/#reference-style-links -->
[contributors-shield]: https://img.shields.io/github/contributors/franckferman/GhostInTheKernel.svg?style=for-the-badge
[contributors-url]: https://github.com/franckferman/GhostInTheKernel/graphs/contributors
[forks-shield]: https://img.shields.io/github/forks/franckferman/GhostInTheKernel.svg?style=for-the-badge
[forks-url]: https://github.com/franckferman/GhostInTheKernel/network/members
[stars-shield]: https://img.shields.io/github/stars/franckferman/GhostInTheKernel.svg?style=for-the-badge
[stars-url]: https://github.com/franckferman/GhostInTheKernel/stargazers
[issues-shield]: https://img.shields.io/github/issues/franckferman/GhostInTheKernel.svg?style=for-the-badge
[issues-url]: https://github.com/franckferman/GhostInTheKernel/issues
[license-shield]: https://img.shields.io/github/license/franckferman/GhostInTheKernel.svg?style=for-the-badge
[license-url]: https://github.com/franckferman/GhostInTheKernel/blob/stable/LICENSE
[protonmail-shield]: https://img.shields.io/badge/ProtonMail-8B89CC?style=for-the-badge&logo=protonmail&logoColor=blueviolet
[linkedin-shield]: https://img.shields.io/badge/-LinkedIn-black.svg?style=for-the-badge&logo=linkedin&colorB=blue
[twitter-shield]: https://img.shields.io/badge/-Twitter-black.svg?style=for-the-badge&logo=twitter&colorB=blue

