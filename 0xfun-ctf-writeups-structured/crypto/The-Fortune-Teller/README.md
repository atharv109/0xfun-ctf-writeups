# The Fortune Teller

**Category:** Crypto  
**Source:** krauq  
**Flag:** `0xfun{trunc4t3d_lcg_f4lls_t0_lll}`

## Challenge Summary

The generator was a 64-bit LCG, but only the upper 32 bits of each state were shown.

## Solve Idea

The missing lower 32 bits of the first state were brute-forced. For each candidate, the LCG was advanced and checked against later observations.

Because later outputs quickly rejected wrong guesses, the brute-force search became manageable.

## Solve Flow

1. Take the first visible 32-bit glimpse.
2. Enumerate all possible lower 32-bit values.
3. Rebuild the full candidate state.
4. Advance the recurrence and compare against later glimpses.
5. Keep the single valid state.

The original note says the solve ran very quickly in C, around 0.6 seconds.
