# The Roulette Conspiracy

**Category:** Crypto  
**Source:** krauq  
**Flag:** `0xfun{m3rs3nn3_tw1st3r_unr4v3l3d}`

## Challenge Summary

This challenge used the Mersenne Twister PRNG. Each observed output was XORed with `0xCAFEBABE`, but that masking did not fundamentally protect the generator.

## Solve Flow

1. Collect 624 outputs.
2. Undo the XOR with `0xCAFEBABE`.
3. Recover the raw MT outputs.
4. Untemper each output.
5. Reconstruct the full internal MT state.
6. Clone the generator.
7. Predict the next 10 outputs exactly.

## Why 624 Outputs Matter

MT19937 has a state of 624 32-bit words. Once enough outputs are available, the internal state can be reconstructed exactly.

## Takeaway

Masking predictable PRNG output with a fixed XOR does not meaningfully improve security. The structure of MT still leaks completely once enough outputs are observed.
