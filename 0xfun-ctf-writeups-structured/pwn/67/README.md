# 67

**Category:** Pwn  
**Source:** krauq  
**Flag:** `0xfun{p4cm4n_Syu_br0k3_my_xpl0it_btW}`

## Challenge Summary

This was a heap challenge centered on a use-after-free. The freed pointer was not cleared, which left a dangling reference and opened the door to tcache poisoning.

## Core Bug

Freeing an object did not null the stored pointer. That allowed later operations to interact with a chunk that had already been returned to the allocator.

## Exploit Direction

The solve used:

- use-after-free
- tcache poisoning
- House of Apple 2 / FSOP

The target path eventually reached `_IO_wfile_overflow`, which was redirected toward a `system(" sh")` style execution path.

## Environment Note

The challenge used glibc 2.42 with safe-linking, so the exploit needed to respect modern heap hardening. That is what made the challenge more advanced than a basic UAF.

## Final Outcome

By chaining heap corruption into an FSOP-style attack, execution reached `system`, which produced the flag.
