---
title: UStar RamFS
sidebar_position: 1
---

This module implements a read-only, in-memory filesystem. It is designed to work with a tar-formatted archive embedded as a boot module (ramdisk), allowing early-stage file access before any persistent filesystems are available.

## Overview

RamFS provides minimal virtual filesystem (VFS) operations over a static tar archive:

* Path-based file lookup.
* Buffered file reading.
* Mounting at boot time (usually at `/ramdisk`).

It is primarily intended for loading early configuration, init scripts, or embedded binaries during system initialization.

## Key Components

### `RamFs`

A lightweight struct that wraps a static tar archive (`&'static [u8]`). This archive is parsed dynamically to emulate a filesystem structure.

### `VnodeOps` Implementation

RamFs integrates with the virtual filesystem layer through the `VnodeOps` trait. It supports:

* `lookup(path)`: Resolves a file or directory path into a vnode.
* `read(file, buf, offset)`: Reads file contents into a buffer, supporting basic offset handling.

### `init(vfs)`

Initializes the RAM filesystem by:

* Accessing the boot module containing the archive.
* Extracting its memory location and size.
* Mounting it under the virtual path `/ramdisk`.

System logs include details such as:

* Module memory address.
* Size in bytes.
* Provided module path.

## Internal Utility: `tar_lookup`

The `tar_lookup` function performs linear scanning over the tar archive to find a matching file. It:

* Normalizes filenames to match tar-style paths (`./filename`).
* Identifies file or directory entries.
* Extracts file size, data slice, and vnode type.

Only regular files and directories are supported. No symbolic links, special device files, or metadata extensions are handled.

## Usage & Limitations

* **Read-only:** This filesystem does not support writing or modifying files.
* **Early-boot only:** Ideal for systems before external storage drivers are initialized.
* **Simple TAR support:** Assumes POSIX ustar format; no compression or complex features.
