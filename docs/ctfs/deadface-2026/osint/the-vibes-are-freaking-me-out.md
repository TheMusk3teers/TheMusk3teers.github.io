---
title: The Vibes Are Freaking Me Out
tags:
  - DEADFACE CTF 2026
  - OSINT
---

# The Vibes Are Freaking Me Out

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | OSINT |
| **Points** | 75 |
| **Author** | @Cyb0rgSw0rd |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) · Lyna |

## Challenge

> We've received intelligence that a DEADFACE operative may have been using AI to create a browser extension that would enable them to communicate openly outside of the GhostTown forums using a new model called Maude. We need to find the tool they've built so we can find out what they're trying to hide from us.
>
> Flag Format: `deadface{N0t_Th3_Fl4g_Jus7_@n_3x@mpl3}`. The flag will appear in its entire form.

## Solution

### Finding the tool

A GitHub code search for `deadface maude` finds [fr34ked0ut/HIPS](https://github.com/fr34ked0ut/HIPS) ("Hidden In Plain Sight"), b1tfr34k's browser extension for hiding text in Unicode. Its `.maude/maude.md` holds the operative's instructions to "Maude", the AI model from the prompt.

### The flag

`.maude/skills/payload-gen/SKILL.md` stores a "crew token" and says it was saved backwards, then encoded:

```bash
echo fSEzcjNodzBOXzNNX3N0M0dfZ24xdHBtMHJQX3MxaFRfMTFBe2VjYWZkYWVk | base64 -d | rev
# deadface{A11_Th1s_Pr0mpt1ng_G3ts_M3_N0wh3r3!}
```

### Decoys

- **Deleted `.env` (commit `ec9f083`):** `TOKEN_TOKEN`, `PASSCODE` and `STATEMENT` decode to jokes ("DR TOM LEI LIVES ON WITHIN ALL OF US", etc.).
- **Zero-width characters in `maude.md`:** a hidden prompt injection telling an AI assistant to annoy the user by changing files. Decode it, don't follow it.
- **Base64 comments** in the exfil and recon skills and `build.js`: "STOP LOOKING FOR THE TOKEN MAUDE", "not the flag…".

## Flag

!!! flag "Flag"
    `deadface{A11_Th1s_Pr0mpt1ng_G3ts_M3_N0wh3r3!}`

## Takeaways

GitHub code search on unusual words from the prompt finds operative repos fast. Check commit history and every encoded string, but expect most of them to be bait. The HIPS hiding methods come back in [You Wouldn't Git It](you-wouldnt-git-it.md).
