# Phantom

**Category:** Pwn  
**Source:** krauq  
**Flag:** `0xfun{r34l_k3rn3l_h4ck3rs_d0nt_unzip}`

## Challenge Summary

This challenge moved into kernel exploitation. The vulnerable component was a Linux kernel module with a physical page use-after-free.

## Bug

A physical page could be freed and then reused in an attacker-controlled way.

## Main Technique

The solve used the dirty pagetable technique:

1. free a physical page
2. reclaim that page as a page-table-entry page
3. gain arbitrary physical memory read capability

Once physical memory could be inspected, the remaining task was to search through memory until the flag was found.

## Why This Matters

Kernel challenges often stop being about a single neat “jump to win” moment. Instead, they become about turning one low-level primitive into broader memory access. That is exactly what happened here.
