# Insanity 1

**Category:** OSINT  
**Source:** krauq  
**Flag:** `0xfun{1ns4n1ty_d15c0rd_1_thr0ugh_r0l3s}`

## Solve Flow

1. Join or inspect the Discord invite `RUwDQBsSxt`.
2. Discover that the flag is not visible in normal chat content.
3. Enumerate the Discord server structure via the API.
4. Find the flag hidden in a role name.

The main trick was realizing that roles, not messages, held the answer.
