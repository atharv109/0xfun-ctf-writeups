# bit_flips

**Category:** Pwn  
**Source:** krauq  
**Flag:** `0xfun{3_b1t5_15_4ll_17_74k35_70_g37_RC3_safhu8}`

## Challenge Summary

The program allowed exactly three arbitrary bit flips. The whole exploit had to be built within that hard limit.

## Solve Idea

The intended solution used those three flips in a very constrained way:

- two flips changed the `FILE` structure’s `_fileno` field from `3` to `0`
- one final flip changed the return address so control landed in `cmd()`

## Why the `_fileno` Flip Mattered

Changing `_fileno` from `3` to `0` redirected later `fgets()` input from some other file descriptor to standard input.

That meant the attacker could now feed the command string interactively.

## Final Control Transfer

The last bit flip redirected the function return into `cmd()`, which was already a useful code path inside the binary.

After that, the attacker simply supplied:

```text
cat flag
```

through stdin and got the flag.

## Takeaway

This challenge was less about a long exploit chain and more about precision. The difficulty came from making exactly three modifications that each had to carry maximum value.
