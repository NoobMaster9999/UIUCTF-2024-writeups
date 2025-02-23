# X Marked the Spot

- Description: A perfect first challenge for beginners. Who said pirates can't ride trains...

<details>
  <summary>Attachments</summary>

  [public.py](https://uiuctf-2024-rctf-challenge-uploads.storage.googleapis.com/uploads/9122a5606d14d1ded3cbd5907af07c55f80db318f0ea761c16531d0327acd09b/public.py)
  <br>
  [ct](https://uiuctf-2024-rctf-challenge-uploads.storage.googleapis.com/uploads/150449f10624289342a0f5b2d427b3c09eda679b84c5062def1eccf114bdb505/ct)
</details>


# Intro

Looking at `public.py`, the flag's length is 48 bytes while the key's length is 8 bytes. The code is doing a simple XOR. Let's take a look at how XOR works: 

| A | B | A XOR B |
|---|---|---------|
| 0 | 0 |    0    |
| 0 | 1 |    1    |
| 1 | 0 |    1    |
| 1 | 1 |    0    |

When both the bits are the same, it's 0 and when both the bits are different, it's 1. In case of letters, it take's the corresponding ascii value and then converts to binary, then finally XOR them.

# But the key is only 8 bytes?

The key is only eight bytes but the flag is 48. how does that work? In XOR, the smaller variable (key or text), will be repeating.

If my 8 byte key is: "abcdefgh", and I XOR with 48 bytes, my new key will be "abcdefghabcdefghabcdefghabcdefghabcdefghabcdefgh". 

So looking at this, we only need the 8 bytes!

# How can we use this to our advantage?

Well, XOR is reversible. 

let's say the flag is "X", the key is "K", and the ciphertext is "C". Then: 

```
X XOR K = C
C XOR K = X
C XOR X = K
```

# Known Plaintext Attack

XOR is vulnerable to a known plaintext attack.

Let's say I know the first four bytes of X. Then, `(First four bytes of C) XOR (first four bytes of X) = (first four bytes of K)`

This is called a known plaintext attack! In our case, we know the first 7 bytes of the flag (`uiuctf{`) so we can find the first 7 bytes of the key!

# The first 7 bytes

Let's find the first 7 bytes of the key. XOR the first 7 bytes of the ciphertext with `uiuctf{`. Let's use Python!

```python
from itertools import cycle
ct = open("ct","rb").read()[:7] # First 7 bytes
known=b"uiuctf{"

first_7_bytes = bytes(x ^ y for x, y in zip(known, cycle(ct))) # Provided in `public.py`

print(first_7_bytes) # First 7 bytes of the key

```

The first 7 bytes of the key are: "hdiqbfj". 

# The Last Byte

Just like we know the first 7 bytes of the flag, we also know the last byte of the flag: `}`. In our case, the key aligns perfectly with the flag so that the last character of the repeating key is the last character of the flag. Note:

`(Last byte of flag) XOR (Last byte of ciphertext) = Last byte of key`

Use the script above but instead of the first 7 bytes, use only the last one.

Our entire key is: "hdiqbfjq". XOR the key with the ciphertext to get the flag!

Final script:

```python
from itertools import cycle
ct = open("ct","rb").read()
first_7_ct = ct[:7] # First 7 bytes of the ciphertext
known=b"uiuctf{" # First 7 bytes of the flag

first_7_bytes = bytes(x ^ y for x, y in zip(known, cycle(first_7_ct))) # Provided in `public.py`

last = chr(ct[-1]).encode() # Last byte of the ciphertext
last_known = b"}" # Last byte of the flag
last_byte = bytes(x ^ y for x, y in zip(last_known, cycle(last))) # Last byte of the key

key = first_7_bytes + last_byte # Entire key
flag = bytes(x ^ y for x, y in zip(ct, cycle(key)))
print(flag) # Flag!
```

# Flag - uiuctf{n0t_ju5t_th3_st4rt_but_4l50_th3_3nd!!!!!}
