---
title: Alice and Bob Would Like a Word
tags:
  - DEADFACE CTF 2026
  - OSINT
---

# Alice and Bob Would Like a Word

| | |
|---|---|
| **CTF** | DEADFACE CTF 2026 |
| **Category** | OSINT |
| **Points** | 75 |
| **Author** | @Cyb0rgSw0rd |
| **Writeup by** | [TheMusk3teers](../../../members.md)<br>[H0ussamCl4p](https://github.com/H0ussamCl4p) · [chaimae098](https://github.com/chaimae098) · [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it) · [imanebouzbiba](https://github.com/imanebouzbiba) |

## Challenge

> There's been an awful lot of chatter recently about excessive privacy invasions and surveillance happening. With such rampant privacy invasions, why on earth would you just leave your keys out in the open like that?
>
> Flag format: The flag will use `Z` and `Y` in place of opening and closing curly braces. Replace `Z` with `{` and `Y` with `}` so that the flag looks like `deadface{whatever_the_flag_is}`. The flag starts with `deadface`.

## Solution

### The forum thread

In the GhostTown thread "Privacy is officially dead and everyone's just cool with it?" (`/t/44`), people talk about public keys being exposed on repos. `b1tfr34k` gets a bit defensive when `daem0n` says public keys are published "by default on some repos".

We already knew from [The Vibes Are Freaking Me Out](the-vibes-are-freaking-me-out.md) that b1tfr34k's GitHub is `fr34ked0ut`.

### GitHub's `.keys` page

We checked the PGP keyservers (`keys.openpgp.org`, `keyserver.ubuntu.com`, `pgp.mit.edu`) and `github.com/fr34ked0ut.gpg` first, and found nothing. But GitHub also publishes everyone's SSH public keys at `github.com/<user>.keys`:

```bash
curl -s https://github.com/fr34ked0ut.keys
```

The fifth key has the flag written into the Base64 blob:

```text
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIDdpiFW3mHOZbTkfndeadfaceZPubM34n5Publ1cDuHY
```

Curly braces aren't valid Base64, which explains the weird `Z`/`Y` rule in the flag format. Swap them back and you get `deadface{PubM34n5Publ1cDuH}`, "pub means public, duh".

## Flag

!!! flag "Flag"
    `deadface{PubM34n5Publ1cDuH}`

## Takeaways

`github.com/<user>.keys` and `github.com/<user>.gpg` are public for every account. And when a flag format swaps out certain characters, ask what kind of field can't hold them. Here it was Base64.
