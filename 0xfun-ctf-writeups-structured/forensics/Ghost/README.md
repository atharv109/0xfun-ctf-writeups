# Ghost

**Category:** Forensics  
**Source:** krauq  
**Flag:** `0xfun{l4y3r_pr0t3c710n_k3y}`

## Challenge Summary

The provided file looked like a normal PNG image, but it contained additional hidden data after the `IEND` chunk.

## Solve Flow

1. Inspect the PNG structure.
2. Notice extra trailing data after `IEND`.
3. Extract the appended blob.
4. Identify it as a password-protected `7z` archive.
5. Recover the password from visible SSTV-decoded text in the image.

The password was:

```text
1n73rc3p7_cOnf1rm3d
```

Using that password opened the archive and revealed the flag.

## Takeaway

This was a classic layered forensics challenge: image container, hidden payload, then password recovery from a visual clue.
