# Snore Signatures

- Description: These signatures are a bore!

ncat --ssl snore-signatures.chal.uiuc.tf 1337

<details>
  <summary>Attachments</summary>
	
  [chal.py](https://uiuctf-2024-rctf-challenge-uploads.storage.googleapis.com/uploads/b477a42de9e90bafc5bd16405fee97b4b532d9b96e25330eb55ae1c2aa45e4f0/chal.py)
</details>


# Intro

Looking at the source code, it's a basic implementation of Schnorr signatures. Looking at the challenge name, we can confirm this theory.

# Analysis

These 3 are the most important functions in the code:

```python
def snore_gen(p, q, g, N=512):
    x = randint(1, q - 1)
    y = pow(g, -x, p)
    return (x, y)


def snore_sign(p, q, g, x, m):
    k = randint(1, q - 1)
    r = pow(g, k, p)
    e = hash((r + m) % p) % q
    s = (k + x * e) % q
    return (s, e)


def snore_verify(p, q, g, y, m, s, e):
    if not (0 < s < q):
        return False

    rv = (pow(g, s, p) * pow(y, e, p)) % p
    ev = hash((rv + m) % p) % q

    return ev == e
```

Note: The hash function is also provided in the code, but it is not important as we cannot exploit it.

The code generates the basic values then provides us with p,q,g,y,s,e. It takes one `m` and signs it for us. Afterward, it tells us to forge a signature on our own, taking `m` and `s` as our input. However, we can't use the `m` we provided earlier. It calls snore_verify() to verify our signature. If it's true, it continues and does this for 10 times. Afterward, it gives us the flag.

# Vulnreability

Looking at snore_verify, we see that we can control `m` and `s`. If you look carefully, the `% p` is inside the hash function argument. We can use this to our advantage! Let's do a simple example: `(rv + 0) % p == rv % p` And `(rv + p) % p == rv + p % p == rv % p`. This is because any number modulo the same number is zero! This property still holds true even with addition, so `(x+p) % p` will always return `x` since p % p is zero! In the actual implementation of Schnorr signatures, it uses concatenation and not addition!

# Solve

We will provide m as `0` for the server to sign. We will store the signature. We will give the value of `p` when asked for `m`, and give the same `s` (signature). So, when it signs the first time, it signs:

`hash((rv+0)%p) = hash(rv%p)`. 
Then, when it verifies, it does:

`hash((rv+p)%p) = hash(rv%p)`

For any integer m, This statment holds true:

`hash((rv+m)%p) = hash((rv+m+p)%p)`

We can now easily verify! We just need to do this 10 times! There is a problem though. We can't use the same `m` everytime to verify. But we can give zero the first time, one the second time and so on! All we have to do is add `p` to `m` for the exploit to work.

# Script

Here's the final script:

```python
from pwn import *
io = remote("snore-signatures.chal.uiuc.tf","1337",ssl=True)
p = int(io.readline().strip()[4:].decode()) # Get p
q = int(io.readline().strip()[4:].decode()) # Get q
g = int(io.readline().strip()[4:].decode()) # Get g
y = int(io.readline().strip()[4:].decode()) # Get y
io.readline()
m=0
for i in range(10):
	print(f"Round {i}")
	m += 1 # Increment m each time since we can't use the same one.
	io.sendline(str(m).encode()) # Give m for it to sign
	s = int(io.readline().strip()[8:].decode()) # Get the signed values
	e = int(io.readline().strip()[4:].decode()) # Get the signed values
	(io.readline())
	io.sendline(str(m+p).encode()) # Send m+p since the modulo will get rid of p.
	io.sendline(str(s).encode()) # Send the same s
	assert b"correct" in (io.readline()) # Make sure that it is right!
	if i == 9:
		io.interactive() # Get flag!
	y = int(io.readline().strip()[4:].decode())
	(io.readline())
```

# Flag - uiuctf{add1ti0n_i5_n0t_c0nc4t3n4ti0n}
