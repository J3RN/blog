---
title: "Nix Terminology"
layout: "simple"
---

Under the umbrella of "Nix", there are a variety of separate applications and ideas that get thrown at you when you start reading the documentation.  Here is my incomprehensive list of them, and descriptions of what they are:

## Topics

- **Nix**.  Nix, the project, is a *"purely functional" package manager*.  Everything here relates to Nix, the package manager.
- **NixOS**.  NixOS is a Linux distribution built around declarative definitions of packages, both which packages and configurations for each.
- **Nix Store**:  This is where Nix packages are actually installed to.  Loading packages to make them available consists of updating symlinks to point to packages in the Nix Store.  The Nix Store is able to contain multiple versions of the same package; the different versions will be persisted in different directories within the Nix store.

## Commands

- [`nix-shell`]: `nix-shell` allows you to run a shell with specific package loaded.  For instance, if you don't have Vim installed and want to use Vim temporarily, you can run `nix-shell -p vim` and a shell will be opened where `vim` is available.
- [`nix-env`]: `nix-env` mutates or queries the "user environment".  Many Nix users recommend against using `nix-env` directly, unless you are using Nix, the package manager, on a system that is not NixOS.  This is because `nix-env` is considered a "low-level command" whose functionality should generally be used by higher-level commands.  You can install software to the current user environment with `nix-env --install <package>`, but this does not create or update any kind of declarative package list.
