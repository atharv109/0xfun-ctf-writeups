# DTMF

**Category:** Forensics  
**Source:** krauq  
**Flag:** `0xfun{Mu1t1_t4p_plu5_dtmf}`

## Challenge Summary

The WAV file contained 288 DTMF tone bursts. Each burst encoded either `0` or `1`, so the audio had to be decoded into a bitstream first.

## Solve Flow

1. Detect and decode the DTMF tones.
2. Translate the tones into binary.
3. Convert the binary into Base64.
4. Decode the Base64 data.
5. Apply Vigenère decryption.

The Vigenère key was found in the WAV metadata comment:

```text
uhmwhatisthis
```

That final decryption step produced the flag.
