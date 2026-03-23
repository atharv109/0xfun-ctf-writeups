# Tragedy

**Category:** OSINT  
**Source:** krauq  
**Flag:** `0xfun{UPS_Flight_2976_fall1n_d0wn}`

## Challenge Summary

This challenge continued the same identity trail as MultiVerse.

## Solve Flow

1. Follow `Massive-Equipment393` into a Reddit comment on an r/aviation UPS Flight 2976 crash thread.
2. Notice that the comment contains an unusual number of invisible Unicode characters.
3. Extract the zero-width characters.
4. Decode them as quaternary-style steganographic data.

The hidden characters used code points like `U+200C`, `U+200D`, `U+FEFF`, and `U+202C` to encode the flag.
