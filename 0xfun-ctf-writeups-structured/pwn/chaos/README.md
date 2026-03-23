# chaos

**Category:** Pwn  
**Source:** krauq  
**Flag:** `0xfun{l00k5_l1k3_ch479p7_c0uldn7_50lv3_7h15_0n3}`

## Challenge Summary

The challenge implemented a custom virtual machine with seven opcodes. The main bug was in the `STORE` instruction, which allowed out-of-bounds writes.

## Root Cause

The VM used a signed comparison (`jle`) when validating a destination offset, but it never enforced a lower bound.

That meant negative offsets were allowed, which let the attacker write before the intended VM memory region.

## Exploitation Path

The useful target was the VM’s function pointer table. By writing outside bounds with a negative index, the attacker could overwrite handler pointers.

The cleanest route was:

1. overwrite the HALT handler `func_table[0]`
2. replace it with `system@plt`
3. make the relevant register point to the string `"sh"`
4. trigger opcode `0`

At that point, the VM invoked `system("sh")` instead of its original halt logic.

## Why It Worked

The core bug was memory corruption inside the interpreter itself. Once the opcode dispatch table became attacker-controlled, the VM effectively provided arbitrary indirect control flow.
