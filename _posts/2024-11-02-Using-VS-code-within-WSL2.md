---
title: "How to launch VS code from within WSL (and also for kubectl)"
date: 2024-11-02
tags: wsl kubectl
---

I've found several stackoverflow posts or blog posts talking about updating changing and using `code` (Microsoft Visual Studio Code) instead of your default linux text editor (`vim` in my case) and what can be a very useful upgrade for editing for instance `kubectl` stuff. Nice one here but it is applicable only to Linux or Mac (AFAIK) but missing the key part for Linux OS under Windows (within WSL). Commands and aliases mentioned there didn't work on my laptop.

It took me a bit more googling and testing to come up with a working solution to trigger VS Code (Windows instance) from within WSL terminal and

```
export PATH=$PATH:"/mnt/c/Program\ Files/Microsoft\ VS\ Code/bin/"
```

and

```
export KUBE_EDITOR='code --wait --new-window'
```
