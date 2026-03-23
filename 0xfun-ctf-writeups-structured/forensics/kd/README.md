# kd>

**Category:** Forensics  
**Source:** krauq  
**Flag:** `0xfun{wh0_n33ds_sl33p_wh3n_y0u_h4v3_cr4sh_dumps}`

## Challenge Summary

The challenge provided a very large Windows MiniDump file, `crypter.dmp`.

## Easy Solve

A quick strings search was enough:

```bash
strings crypter.dmp | grep 0xfun
```

That immediately revealed the flag.

## Intended Solve

The intended route was more methodical:

1. open the dump in WinDbg
2. run `.ecxr` to restore the exception context
3. inspect the `RBX` register
4. recover remaining bytes from the stack

## Takeaway

This is a good reminder that crash dumps often leak sensitive runtime data directly, and the simplest extraction path can sometimes beat the intended one.
