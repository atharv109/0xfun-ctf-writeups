# The Fortune Teller's Revenge

**Category:** Crypto  
**Source:** krauq  
**Flag:** `0xfun{r3v3ng3_0f_th3_f0rtun3_t3ll3r}`

## Challenge Summary

This was the harder sequel to The Fortune Teller. It used the same truncated 64-bit LCG idea, but introduced a 100,000-step jump between glimpses.

## Main Difficulty

Directly iterating from one visible output to the next was no longer practical because too many hidden transitions occurred in between.

## Solve Idea

The jump and next-state recurrence were combined into one affine transformation. Then a meet-in-the-middle approach over split state bits was used to recover the internal state.

## Why It Worked

Even with sparse observations, the recurrence was still algebraically structured. The large jump increased the difficulty, but did not remove the linear relationships that made state recovery possible.
