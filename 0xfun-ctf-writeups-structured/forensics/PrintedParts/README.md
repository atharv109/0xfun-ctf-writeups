# PrintedParts

**Category:** Forensics  
**Source:** krauq  
**Flag:** `0xfun{this_monkey_has_a_flag}`

## Challenge Summary

The challenge provided G-code for an Ultimaker S5. The hidden message was not plain text inside the file. Instead, it was physically encoded in the 3D print path.

## Solve Idea

The flag was embossed as raised text on Blender’s Suzanne monkey head. To reveal it, the G-code motion data had to be parsed and plotted.

## Solve Flow

1. Parse `G0` and `G1` motion commands.
2. Track extrusion moves and coordinates.
3. Filter points to the front-face Y range.
4. Plot X versus Z for those relevant moves.
5. Read the embossed text from the resulting shape.

## Why It Worked

A printer path is still geometric data. Once visualized correctly, hidden geometry becomes readable just like an image.
