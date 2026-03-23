# BitStorm

**Category:** Crypto  
**Source:** krauq  
**Flag:** `0xfun{L1n34r_4lg3br4_W1th_Z3_1s_Aw3s0m3}`

## Challenge Summary

This challenge used a custom PRNG initialized from a 256-byte seed and an internal state of 32 64-bit words. Although the design looked complicated at first, all of its state transitions were linear over GF(2).

## Key Observation

The generator only used operations like:

- XOR
- shifts
- rotations

Over GF(2), these are linear transformations. That meant the whole generator could be modeled as matrix multiplication.

## Solve Idea

1. Express the PRNG state update as a linear transformation over GF(2).
2. Build equations from the observed outputs.
3. Recover enough output bits to create a solvable linear system.
4. Solve the system with Gaussian elimination.

The writeup notes a `3840 x 2048` linear system built from 60 outputs.

## Why It Worked

Even though the PRNG looked custom and unfamiliar, its structure was still fully linear. Once that was recognized, the problem turned into a linear algebra exercise rather than a black-box cryptanalysis task.
