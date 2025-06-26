---
title: Virtual Memory Manager
sidebar_position: 1
---

This module is responsible for setting up and managing the virtual memory system. It handles page table initialization, memory mapping, page fault handling, and virtual memory tracking using HHDM (Higher Half Direct Mapping).

## Overview

The Virtual Memory Manager:

* Initializes new page tables based on available memory.
* Maps physical memory into virtual address space using large pages (1 GiB or 2 MiB).
* Tracks used virtual memory ranges via `NoditSet`.
* Provides utilities for allocating and mapping/unmapping memory pages.
* Handles page faults using a custom interrupt handler.

## Key Structures

### `VirtualMemory`

Encapsulates the state of the virtual memory manager:

* `set`: A `NoditSet` that tracks used virtual memory regions.
* `cr3`: The root page table physical frame.
* `hhdm_offset`: The offset between physical and virtual memory in the higher-half.

### `VmmReturn`

Returned by the initialization function; contains:

* `virt_mem`: A fully initialized `VirtualMemory` instance.
* `new_cr3` and `new_cr3_flags`: For switching to the new address space.

### `AllocatedPages<'a, S>`

Represents a contiguous set of virtual pages allocated from `VirtualMemory`. Provides safe and unsafe methods for:

* Mapping physical frames.
* Unmapping and deallocating pages.

## Initialization Flow

### `init(...)`

* Allocates a new level 4 page table.
* Uses the `CpuId` instruction to check for 1 GiB page support.
* Initializes mappings using either 1 GiB or 2 MiB pages.
* Maps the top 512 GiB of address space for the kernel.
* Writes the new page table into `Cr3`, switching page tables.

Returns a `VmmReturn` object.

### `init_with_page_size<S>()`

* Generic function to map physical memory regions listed in the bootloader’s memory map using large pages (`S`: `Size1GiB` or `Size2MiB`):
* Filters usable memory types (`USABLE`, `EXECUTABLE_AND_MODULES`, etc.).
* Avoids remapping overlapping regions.
* Builds the virtual memory tracking set.
* Adds a permanent mapping of the kernel region (`0xFFFFFF8000000000`).

## Page Allocation and Mapping

### `allocate_contiguous_pages(...)`

Finds a contiguous region of unused virtual memory for a given number of pages, ensuring proper alignment, and reserves it in the `NoditSet`.

### `already_allocated(...)`

Manually constructs an `AllocatedPages` wrapper from an existing virtual memory range.

### `AllocatedPages::map_to(...)`

Maps a virtual page to a physical frame using provided flags and a frame allocator. Panics if the page is outside the allocated range.

### `AllocatedPages::unmap_and_deallocate()`

Unmaps all pages in the range and removes the interval from the memory set.
