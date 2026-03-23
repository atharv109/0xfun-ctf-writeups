# MultiVerse / Where's Franklin?

**Category:** OSINT  
**Source:** krauq  
**Flag:** `0xfun{sp0t1fy_pl4yl1st_3xt3nd_M0R3_TR4X}`

## Challenge Summary

This challenge followed an identity trail across online services.

## Solve Flow

1. Start from the username `Massive-Equipment393`.
2. Pivot into that user’s Reddit activity.
3. Decode Base58 data found in a Reddit comment.
4. Use that to locate Spotify playlists.
5. Reconstruct the flag from multiple fragments.

The final flag was assembled using:

- Base64 in a playlist description
- Base58 data from Reddit
- first letters of songs in a second playlist

## Why It Worked

This is a classic multi-platform OSINT chain where no single clue is enough alone. The intended difficulty comes from recognizing how the breadcrumbs connect across services.
