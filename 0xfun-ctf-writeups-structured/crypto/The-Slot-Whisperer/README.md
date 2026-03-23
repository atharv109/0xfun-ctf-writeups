# The Slot Whisperer

**Category:** Crypto  
**Source:** krauq  
**Flag:** `0xfun{sl0t_wh1sp3r3r_lcg_cr4ck3d}`

## Challenge Summary

The backend used an LCG with known parameters, but only exposed `state % 100` as the visible output.

## Solve Idea

Because the multiplier, increment, and modulus were known, the only missing piece was the internal state.

The path was:

1. take the first observed spin
2. enumerate candidate internal states consistent with that output
3. advance each candidate through the recurrence
4. reject candidates quickly when later outputs do not match

With enough observations, only one state survived, and future spins became predictable.

## Why It Worked

The `% 100` output hid some state information, but not enough to stop brute force once the generator parameters were known.
