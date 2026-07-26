---
title: "Example Machine — Writeup Template (replace me)"
date: "2026-07-26"
tags:
    - HTB
    - Linux
    - Medium
    - privesc
thumbnail: "/assets/img/thumbnail/sample.png"
bookmark: true
---

> **This is a template, not a real writeup.** Duplicate this file for every new box inside `_pages/Write-ups/`, rename it something like `machine-name.md`, update the front matter (`title`, `date`, `tags`, `thumbnail`) and replace every section below. Only publish once the machine is **retired** — that's HTB's policy for public write-ups. Add a new subfolder under `_pages/Write-ups/` (with its own empty `index.md`) if you want sub-categories like AD, Web, or Binary Exploitation.

# Overview

One or two lines: what the box was about, what made it interesting, and the high-level path from first foothold to root.

# Recon

```
nmap -sC -sV -oN nmap/initial.txt 10.10.10.10
```

List what was open and why it mattered.

# Enumeration

Walk through what you found poking at each open service. Keep commands in code blocks so they're copy-pasteable.

# Exploitation

The actual foothold: the vulnerability, why it works, and the exact steps/payload used to get a shell.

```
curl -X POST http://10.10.10.10/upload -F "file=@shell.php"
```

# Privilege Escalation

Path from low-priv shell to root/SYSTEM — enumeration you ran, what it turned up, and the final escalation step.

```
sudo -l
```

# Lessons Learned

What this box taught you, or a technique you want to remember for next time.
