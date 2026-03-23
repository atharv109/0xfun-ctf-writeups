0xFun CTF 2026 Writeups
This repository contains my organized writeups for selected challenges from 0xFun CTF 2026.
Instead of putting everything into a single file, I split the repo by category and then by challenge, so each writeup is easier to read, update, and reference later.
---
Repository Structure
```text
.
├── README.md
├── web/
├── pwn/
├── crypto/
├── forensics/
└── osint/
```
Inside each category folder, every challenge has its own subfolder with a dedicated `README.md`.
Example:
```text
web/
└── SkyPort-Ops/
    └── README.md
```
---
Categories
Web
Web exploitation challenges involving authentication flaws, request smuggling, path traversal, file upload bugs, and multi-step exploit chains.
Pwn
Binary exploitation challenges involving memory corruption, GOT overwrites, seccomp bypasses, heap attacks, VM escapes, and kernel exploitation.
Crypto
Cryptography challenges involving PRNG reversal, elliptic curve weaknesses, truncated state recovery, linear algebra over GF(2), and prediction attacks.
Forensics
Challenges involving file carving, hidden metadata, memory dumps, encoded artifacts, audio decoding, steganography, and hidden embedded data.
OSINT
Open-source intelligence challenges involving public infrastructure, geolocation, DNS records, Discord artifacts, Reddit traces, and malware intel pivots.
---
Challenge Index
Web
SkyPort Ops
Pwn
Warden
What You Have
Fridge
bit_flips
chaos
67
Phantom
Crypto
BitStorm
MeOwl ECC
The Slot Whisperer
The Roulette Conspiracy
The Fortune Teller
The Fortune Teller's Revenge
Forensics
Ghost
PrintedParts
kd>
Tesla
DTMF
Nothing Expected
OSINT
GeoSkill
Lookup 0xFUN
MultiVerse / Where's Franklin?
Tragedy
Insanity 1
Insanity Revenge
Malware Analysis 1
Malware Analysis 2
Malware Analysis 3
---
Why I Structured It This Way
I wanted this repo to be easy to navigate and useful as a long-term reference. A single giant README becomes hard to maintain, so this structure makes it easier to:
keep each writeup clean and detailed
update one challenge without touching the others
link directly to individual challenges
expand the repo later with scripts, screenshots, payloads, or solver files
---
Notes
Some writeups are very detailed exploit chains, while others are shorter solve summaries depending on how much detail I originally captured during the CTF.
This repo is mainly meant for:
documenting solves
revising techniques later
sharing writeups in a cleaner public format
---
Possible Future Improvements
Later, this repo could be expanded with:
screenshots for key steps
exploit scripts and solver code
payload samples
challenge binaries or sanitized artifacts where allowed
tags like `request-smuggling`, `heap`, `mt19937`, `osint`, etc.
---
Disclaimer
These writeups are for educational purposes and CTF learning only.
