+++
title = "How to Share VS Code Server Across Multiple WSL Users with share_vscode.sh"
date = "2026-02-18"
author = "Subindev D"
tags = ["vscode", "wsl", "linux", "tooling"]
description = "A script to share VS Code Server across multiple WSL users"
+++

If you've ever set up VS Code with WSL (Windows Subsystem for Linux) and needed use it from other users in the same WSL instance, you'll know how tedious it can be.
There is no easy way to do it , that why I created `share_vscode.sh`.

---

## The Problem

When you connect to WSL via VS Code's Remote - SSH extension, VS Code automatically installs a component called `.vscode-server` in your home directory. This contains the VS Code server binary, all your installed extensions, and configuration files.
The problem is that `.vscode-server` is tied to your user account. Other users won't be able to run `code` command to open vscode, which will results in the error `Command 'code' not found`.

---

## The Solution

 We can simply **copy** the `.vscode-server` directory from one user to another and fix the ownership. Since the server binary and extensions are the same regardless of user, this works perfectly.

`share_vscode.sh` automates this entire process in a single script.

---

## What the Script Does

The script handles everything step by step:

1. **Lists available users** on the machine so you know who to target
2. **Prompts for the target username** and validates they exist
3. **Prompts for the `.vscode-server` source path** with a sensible default
4. **Auto-detects the VS Code binary path** using `which code` — no hardcoding needed
5. **Copies `.vscode-server`** to the target user's home directory
6. **Fixes ownership** so the target user actually has access to the files
7. **Adds VS Code to the target user's PATH** in their `.bashrc` (only if not already present)

---

## Prerequisites

Before using the script, make sure:

- You are on **WSL** (Windows Subsystem for Linux)
- VS Code is installed on **Windows** with the **"Add to PATH"** option enabled
- You have **sudo privileges** on the machine
- You have already connected to WSL via VS Code at least once (so `.vscode-server` exists)

To verify `.vscode-server` exists:

```bash
ls ~/.vscode-server
```

---

## How to setup
You need to run the script on the user which `code` command works
The easiest way to run the script is with a single curl command — no downloading or setup needed:

```bash
curl -fsSL https://raw.githubusercontent.com/subindevs/useful-scripts/refs/heads/main/share_vscode.sh | bash
```

Or if you prefer to download it first and inspect before running:

```bash
curl -O https://raw.githubusercontent.com/subindevs/useful-scripts/refs/heads/main/share_vscode.sh
chmod +x share_vscode.sh
./share_vscode.sh
```

---

## How to Use It

Once the script runs, just follow the prompts:

```
Available users:
  - subin
  - deva

Enter target username: deva
Enter path to .vscode-server (press Enter for default: /home/subin/.vscode-server):
✓ VS Code found at: /mnt/c/Users/subin/AppData/Local/Programs/Microsoft VS Code/bin/code

Summary:
  Target user    : deva
  Source dir     : /home/subin/.vscode-server
  Destination    : /home/deva/.vscode-server
  VS Code PATH   : /mnt/c/Users/subin/AppData/Local/Programs/Microsoft VS Code/bin/code

Proceed? (y/n): y

[1/3] Copying .vscode-server...
[2/3] Setting ownership...
[3/3] Adding VS Code to PATH...

✓ Setup complete for deva!
  run: source ~/.bashrc or restart terminal to apply PATH changes.
```

Once done, ask the target user to reload their terminal:

```bash
source ~/.bashrc
```

Now the target user can open VS Code with `code` command

---

## Troubleshooting

**`code .` gives an `Exec format error`**

This means WSL Interop is disabled. Fix it by running in PowerShell:

```powershell
wsl --shutdown
```

Then reopen WSL and try again. To confirm interop is restored:

```bash
cat /proc/sys/fs/binfmt_misc/WSLInterop
# Should say: enabled
```

**VS Code not found with `which code`**

Make sure VS Code was installed on Windows with "Add to PATH" checked. If not, reinstall or manually add it to your PATH:

```bash
export PATH="$PATH:/mnt/c/Users/YOUR_USERNAME/AppData/Local/Programs/Microsoft VS Code/bin"
```

**Ownership issues after copying**

Run manually:

```bash
sudo chown -R targetuser:targetuser /home/targetuser/.vscode-server
```

---

## Things to Keep in Mind

- The script copies extensions as-is. If any extensions use native binaries compiled for a specific user's environment, they may need to be reinstalled by the target user.
- If you install new extensions or update VS Code, you'll need to run the script again for other users.
- The PATH entry is only added once — the script checks for duplicates before writing to `.bashrc`.
