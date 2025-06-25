---
title: VFS – Virtual Filesystem Layer
sidebar_position: 1
---

This module provides the foundational infrastructure for the virtual filesystem (VFS). It enables the abstraction and management of multiple filesystems under a single, unified interface.

## Overview

The VFS acts as a router and interface for all mounted filesystems. It allows lookup and read operations through a common `Vnode` abstraction, enabling the OS to interact with different storage backends (e.g., RAM-based, disk-based) in a uniform way.

It supports:

* Mounting and unmounting filesystems.
* Resolving full paths to underlying filesystems.
* Delegating file operations through trait-based dispatch (`VnodeOps`).

## Core Concepts

### Vnode

Represents a file or directory in the virtual filesystem. Each `Vnode` has:

* A **name**: the relative or resolved path.
* A **vtype**: either a regular file or directory.
* A reference to **VnodeOps**, the implementation of operations for that vnode.

### VnodeType

Defines the type of a node:

* `Regular`: a standard file.
* `Directory`: a folder containing other files.

### VnodeOps (Trait)

All filesystem backends implement this trait. It defines:

* `lookup(path)`: Finds and returns a vnode by relative path.
* `read(file, buf, offset, length)`: Reads data from the file into a buffer.

This abstraction allows the VFS to forward operations to the correct filesystem backend transparently.

## Vfs Structure

The `Vfs` struct manages mounted filesystems using a thread-safe, ordered map:

* `mount(mount_point, ops)`: Registers a new filesystem at the given mount path.
* `unmount(mount_point)`: Removes a mounted filesystem by path.
* `lookuppn(full_path)`: Resolves a full path to a vnode by locating the appropriate mounted filesystem.

## Error Handling

The module defines an error enum:

* `VfsError::NotFound` - Returned when a lookup or unmount operation fails to locate the target.
