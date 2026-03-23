# What You Have

**Category:** Pwn  
**Source:** krauq  
**Flag:** `0xfun{g3tt1ng_schw1fty_w1th_g0t_0v3rwr1t3s_1384311_m4x1m4l}`

## Challenge Summary

This was a classic GOT overwrite challenge. The binary had both **No RELRO** and **No PIE**, which made the solve path very direct.

## Key Observations

- **No RELRO** meant the Global Offset Table stayed writable.
- **No PIE** meant function addresses in the binary were fixed and easy to target.

That immediately suggested overwriting a GOT entry with the address of a useful function already present in the binary.

## Exploitation Idea

The target was `puts@GOT`, and the replacement value was the address of `win()`.

At some later point, the program called:

```c
puts("Goodbye!");
```

After the overwrite, that call no longer went to `puts`. Instead, it jumped to `win()`, which printed the flag.

## Solve Flow

1. Identify that `puts@GOT` is writable.
2. Find the address of `win()`.
3. Overwrite `puts@GOT` with `win()`.
4. Trigger the code path that calls `puts("Goodbye!")`.

Because the relocation table was writable and addresses were fixed, the exploit was reliable.
