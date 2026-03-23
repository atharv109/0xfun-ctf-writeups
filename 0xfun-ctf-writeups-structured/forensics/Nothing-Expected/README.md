# Nothing Expected

**Category:** Forensics  
**Source:** krauq  
**Flag:** `0xfun{th3_sw0rd_0f_k1ng_4rthur}`

## Challenge Summary

The provided PNG contained a large `tEXt` chunk, which itself stored compressed Excalidraw JSON.

## Solve Flow

1. Inspect PNG chunks.
2. Extract the large `tEXt` chunk.
3. Decompress the Excalidraw JSON content.
4. Load or inspect the drawing data.
5. Look for hidden `freedraw` elements placed far off-canvas.

Those off-canvas strokes spelled the flag by hand.

## Why It Worked

Excalidraw files can contain many elements not visible in the normal canvas view. Hidden coordinates are a perfect place to stash data in a forensics challenge.
