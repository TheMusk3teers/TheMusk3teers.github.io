---
title: You Wouldn't Git It
tags:
  - DEADFACE CTF 2026
  - OSINT
  - Web
---

# You Wouldn't Git It

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | OSINT / Web |
| **Points** | 125 |
| **Author** | @Cyb0rgSw0rd |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) · Lyna |

## Challenge

> Everybody wants to be a content creator these days, but nobody wants to create good content. I'm not even sure how they get paid to do this stuff, really... When I go on social media all I want to do is doom scroll until I fall asleep. Maybe I just don't get the memes anymore because I don't laugh at any of these.
>
> Note: this challenge is a continuation of The Vibes Are Freaking Me Out. Use context and artifacts found in that challenge.
>
> Submit the flag as: `deadface{flag-text}`

## Solution

### Finding the platform

GhostTown thread 58 has a reply "Already shared on DoomScroll lol", which matches the "doom scroll" wording. The site is `https://doomscroll.dctf.io` (its address came from the CTF platform; it doesn't show up in DNS or certificate searches).

### Scanning every post

[HIPS](the-vibes-are-freaking-me-out.md) hides text with zero-width characters, Unicode tag characters (U+E0000 + ASCII) and variation selectors. b1tfr34k's own posts only had a decoy, so we pulled all ~240 posts and comments from the public API (`/api/posts?offset=N`, `/api/posts/<id>/comments`) and decoded every channel.

Hidden text turned up in posts 239, 246 and 247. On gh0st404's post 247, the comment "GREEN GIRAFFES CHOCOLATE LEPERS…" carries tag characters:

```python
hidden = ''.join(chr(ord(c) - 0xE0000) for c in comment if 0xE0000 <= ord(c) <= 0xE007F)
# YES! YOU'RE ALL USING IT! deadface{13_h4ck3r5_walk3d_int0_a_bar_each_says_Ill_h4v3_a_r00t_b33r}
```

### Decoys

- Zero-width bits in posts 239, 246 and 247, and variation selectors in post 239, hold hacker jokes ("my password is so strong it has a gym membership…").
- lamia415's comment on b1tfr34k's profile hides "drinks later?".
- The `SecondBrainTrain` repo, forks, HIPS icons and the decoded `CSN_BACKDOOR` value led nowhere.

## Flag

!!! flag "Flag"
    `deadface{13_h4ck3r5_walk3d_int0_a_bar_each_says_Ill_h4v3_a_r00t_b33r}`

## Takeaways

When a tool's repo tells you its hiding methods, scan the whole site for every one of them. In this CTF the tag-character channel held the flag and the zero-width ones were jokes.
