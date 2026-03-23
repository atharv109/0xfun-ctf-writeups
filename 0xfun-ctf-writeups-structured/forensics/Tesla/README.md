# Tesla

**Category:** Forensics  
**Source:** krauq

## Challenge Summary

The provided file was a Flipper Zero `.sub` file. The BadUSB and Sub-GHz angle was a decoy. The real payload was hidden in `RAW_Data`.

## Solve Idea

The raw data contained an XOR-encoded script. The hint string:

```text
i could be something to this
```

served as the XOR key.

## Solve Flow

1. Extract the `RAW_Data` blob.
2. Convert the relevant bytes into a workable hex or byte sequence.
3. XOR the data with the hinted string.
4. Decode the recovered output.

The writeup notes do not include the final printed flag in the source notes, but the important solve path is the XOR recovery from `RAW_Data`.
