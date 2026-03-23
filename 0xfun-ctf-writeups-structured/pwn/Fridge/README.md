# Fridge

**Category:** Pwn  
**Source:** krauq  
**Flag:** `0xfun{4_ch1ll1ng_d1sc0v3ry!...}`

## Challenge Summary

This was a 32-bit ELF with a straightforward stack overflow through `gets()`, but there was one detail that could break the exploit if overlooked.

## Main Bug

The binary used `gets()`, which meant arbitrary-length input could overflow the stack buffer and overwrite saved control data.

## Useful Properties

- `system@plt` was already available in the binary.
- A `/bin/sh` string also existed in the binary.

That made the intended path a simple ret2libc-style chain using existing program symbols.

## Important Caveat

The tricky part was preserving the saved `ebx` value at offset 40.

Because the binary used PIC-relative instructions before returning, corrupting `ebx` caused the program to crash before execution ever reached the overwritten return address.

## Solve Flow

1. Overflow the buffer.
2. Preserve the correct saved `ebx` value.
3. Overwrite the saved return address.
4. Return into `system@plt` with the binary’s `/bin/sh` string as argument.

Without preserving `ebx`, the exploit failed early. Once that was handled, the challenge became a clean stack-based ret2libc solve.
