---
title: "How to launch VS code from within WSL (and also for kubectl)"
date: 2024-11-02
tags: wsl kubectl
---

When working with WSL (Windows Subsystem for Linux), you might want to use Windows VS Code as your default editor instead of traditional Linux text editors like `vim` or `nano`. This is particularly useful when editing Kubernetes configurations with `kubectl edit` or working with Git commits.

## The Problem

Many online tutorials show how to set VS Code as the default editor on native Linux or Mac systems, but they don't address the specific setup needed for WSL on Windows. The standard approaches typically fail because WSL needs to access the Windows VS Code installation.

## The Solution

After some research and testing, I found a working solution to launch VS Code from within the WSL terminal. Here's what you need to do:

### 1. Add VS Code to your PATH

First, add the VS Code binary location to your PATH (change accordingly to your Windows VS Code location):

```bash
export PATH=$PATH:"/mnt/c/Program\ Files/Microsoft\ VS\ Code/bin/"
```

This allows WSL to find the `code` command by pointing to the Windows VS Code installation.

### 2. Set VS Code as your Kubernetes editor

For `kubectl` operations, set VS Code as the default editor:

```bash
export KUBE_EDITOR='code --wait --new-window'
```

The `--wait` flag ensures that `kubectl` waits for you to close the file before proceeding, and `--new-window` opens the file in a fresh VS Code window.

## Making it Permanent

To make these changes persistent, add both export commands to your `~/.bashrc` or `~/.zshrc` file:

```bash
echo 'export PATH=$PATH:"/mnt/c/Program\ Files/Microsoft\ VS\ Code/bin/"' >> ~/.bashrc
echo 'export KUBE_EDITOR="code --wait --new-window"' >> ~/.bashrc
```

Now you can use `code filename` to open files directly in VS Code from your WSL terminal, and `kubectl edit` will open resources in VS Code instead of a terminal-based editor.
