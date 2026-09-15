Paranoid intern · MD
# The Paranoid Intern — Writeup

**Category:** Crypto

## TL;DR

`transmission.txt` seemed threatening in the first place, but it wasn't encrypted at all, just a flag encoded via a number of layers (base64, base32, hex and ROT13). Instead of reversing it layer by layer ourselves, we just fed it into CyberChef's Magic operation, and it went right through.

## Recon

There was only one file, with one long string.

## What we did

Instead of manually figuring out what type of encryption/encoding was applied and then decode the string, we brute-forced it, fed it to CyberChef and applied the Magic operation, which tries all kinds of most common encodings/ciphers (base64, base32, hex, ROT13, etc.) until it finds some which produce human-readable result. We also enabled the Intensive mode.

Each of those layers would automatically peel off when Magic knew how to do it:
- base64 -> Something only consisting of `A-Z` and `2-7` -> base32 fingerprint
- That base32 -> Something only consisting of `0-9a-f` -> hex
- Decoding the hex gives something readable-ish, however, all letters were shifted -> obvious ROT13
- One more decoding will make the flag
We didn't need to implement a decode chain for this challenge ourselves because Magic did the above steps in succession and the readable flag appeared automatically at the end.
 
## Root cause
 
None of the steps above is encryption. They are reversible encoding and their reversions never required any secret keys but simply knowledge about what kind of encoding was performed. The fact that those encoding schemes are stacked four on each other doesn't give any real security either because an attacker or an automatic program would just keep decoding until the text becomes readable. It may look scary as a wall of random characters, but in fact, there is no any key used in the process at all.