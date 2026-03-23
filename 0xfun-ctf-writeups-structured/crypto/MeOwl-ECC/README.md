# MeOwl ECC

**Category:** Crypto  
**Flag:** `0xfun{n0n_c4n0n1c4l_l1f7s_r_c00l}`

## Challenge Summary

This challenge used an anomalous elliptic curve where the group order equaled the field characteristic `p`. That made the system vulnerable to Smart’s attack.

## Core Observation

The hint claimed that Smart’s attack was broken, but the intended twist was that non-canonical lifts still made the attack viable.

## Solve Flow

1. Identify that the curve is anomalous (`#E(F_p) = p`).
2. Apply Smart’s attack to recover the secret scalar `d`.
3. Derive symmetric material through SHA-256.
4. Use the derived keys to decrypt DES-CBC and AES-CBC layers.

## Result

Once the private scalar was recovered, the rest of the challenge collapsed into standard symmetric decryption steps.
