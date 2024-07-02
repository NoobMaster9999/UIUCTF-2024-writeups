# Determined

- Description: "It is my experience that proofs involving matrices can be shortened by 50% if one throws the matrices out."

    Emil Artin

ncat --ssl determined.chal.uiuc.tf 1337

<details>
  <summary>Attachments</summary>

  [server.py](https://uiuctf-2024-rctf-challenge-uploads.storage.googleapis.com/uploads/7eca7b6525ba52d872109a5296ec74f3a4e58df6c534bfc5f3b54db0a83413f4/server.py)
  [gen.py](https://uiuctf-2024-rctf-challenge-uploads.storage.googleapis.com/uploads/d0be34e73989114ae65eadcf8c1c7a16770c17cfe293e9d12245ecaf54cbac30/gen.py)
  [gen.txt](https://uiuctf-2024-rctf-challenge-uploads.storage.googleapis.com/uploads/abd71bfb434d295415aed9b5f1f3cd5d84652605cf272461c83ed0b09160bafd/gen.txt)

</details>

# Intro

It is recommended to read the write-up on "Without a Trace" first, since this is like a part 2. Read it [here](https://github.com/NoobMaster9999/UIUCTF-2024-writeups/tree/main/Crypto/Without%20a%20Trace)

Let's look at gen.py first.

```python
from SECRET import FLAG, p, q, r
from Crypto.Util.number import bytes_to_long
n = p * q
e = 65535
m = bytes_to_long(FLAG)
c = pow(m, e, n)
# printed to gen.txt
print(f"{n = }")
print(f"{e = }")
print(f"{c = }")
```
It looks like normal RSA but usually the `e` is `65537` and not `65535`, but it doesn't really matter. However, if you take a look, `r` is an extra (unused variable). `gen.txt` has `n`,`e`,`c` (provided to us). We need "p" and "q" to be able to decrypt the ciphertext

# Server

Looking at `server.py`, it is pretty much the same as last time except for a few. This time, the input function is:
```python
def inputs():
    print("[DET] First things first, gimme some numbers:")
    M = [
        [0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0],
    ]
    try:
        M[0][0] = p
        M[0][2] = int(input("[DET] M[0][2] = "))
        M[0][4] = int(input("[DET] M[0][4] = "))

        M[1][1] = int(input("[DET] M[1][1] = "))
        M[1][3] = int(input("[DET] M[1][3] = "))

        M[2][0] = int(input("[DET] M[2][0] = "))
        M[2][2] = int(input("[DET] M[2][2] = "))
        M[2][4] = int(input("[DET] M[2][4] = "))

        M[3][1] = q
        M[3][3] = r

        M[4][0] = int(input("[DET] M[4][0] = "))
        M[4][2] = int(input("[DET] M[4][2] = "))
    except:
        return None
    return M
```
`M[0][0]` has p, `M[3][1]` has q and `M[3][3]` has r. Some of the rest have our inputs (and the rest are just `0`). Let's try an example where all my inputs are `1`.
 The matrix would look like:
```
    M = [
        [p, 0, 1, 0, 1],
        [0, 1, 0, 1, 0],
        [1, 0, 1, 0, 1],
        [0, q, 0, r, 0],
        [1, 0, 1, 0, 0],
    ]
```
An interesting pattern.

# Deeper analysis 

Afterward, it calls the `fun()` function on our input and gives us the value of `res`. The function is the same as the `check` function from the previous challenge.
```python
def fun(M):
    def sign(sigma):
        l = 0
        for i in range(5):
            for j in range(i + 1, 5):
                if sigma[i] > sigma[j]:
                    l += 1
        return (-1)**l

    res = 0
    for sigma in permutations([0,1,2,3,4]):
        curr = 1
        for i in range(5):
            curr *= M[sigma[i]][i]
        res += sign(sigma) * curr
    return res
```
This is very complicated. I will only go through the stuff I will use. In this case, looking at the `sign` function, it returns `(-1)**l`. This means that the output will always be `-1` or `1`.
Let's look at the rest. It uses all permutations of `0` through `4`. It multiples current based on the permutations and our input. `res += sign(sigma) * curr`. It signs each permutation, multiples by current, and then does this for all permutations.

# Useless permutations

If you look carefully, if the `curr` is multiplied by zero, it would be zero. `res += sign(sigma) * curr`. Since the curr is 0, `sign(sigma) * curr` is also zero. This means that nothing will be added to `res`. And remember, it accesses the matrix that we gave input on. 
The first permutation would be `(0,1,2,3,4)`. So in this case it would multiply: `M[0][0] * M[1][1]* M[2][2] * M[3][3] * M[4][4]` But some of these are zeroes (M[4][4]), meaning that this will not affect our calculation for `res`. Let's find the permutations that actually affect the calculation.
```python
from itertools import permutations
actual_perms=[]
p=100
q=101
r=1337
M = [
        [p, 0, 1, 0, 1],
        [0, 1, 0, 1, 0],
        [1, 0, 1, 0, 1],
        [0, q, 0, r, 0],
        [1, 0, 1, 0, 0],
    ]

for sigma in permutations([0,1,2,3,4]):
    curr = 1
    for  i in range(5):
        curr *= M[sigma[i]][i]
    if curr != 0:
        actual_perms.append(sigma)

print(actual_perms)
```

`[(0, 1, 4, 3, 2), (0, 3, 4, 1, 2), (2, 1, 4, 3, 0), (2, 3, 4, 1, 0), (4, 1, 0, 3, 2), (4, 1, 2, 3, 0), (4, 3, 0, 1, 2), (4, 3, 2, 1, 0)]` Only eight permutations that actually do the math. 

# Equations

We make up example values:
```
p=100
q=101
r=1337
```
Now let's calculate the "res" ourselves using the function provided to us: `-122364`. (Assuming all our inputs are `1`. Note: This is for all permutations combined). Let's now calculate ourselves, without the function. We know that there are only 8 permutations. 

1. The first one is `(0,1,4,3,2)`. This means that `M[0][0] * M[1][1] * M[4][2] * M[3][4] * M[2][4]` which is `133700`. We know that it also multiples it by `sign(sigma)`. Let's calculate that for this permutation as well: `-1`. The first output is then `-133700`, which, is `-1*100*1337` = `-(p * r)`
2. The second one is `(0,3,4,1,2)`. This means that `M[0][0] * M[3][1] * M[4][2] * M[1][3] * M[2][4]` which is `10100`. `sign(sigma)` = 1 = `10100` = `(p*q)`. Do this for all the permutations. I will only list the equations now.
3. 1337 = `r`
4. -101 = `-q`
5. 1337 = `r`
6. -1337 = `-r`
7. -101 = `-q`
8. 101 = `q`
Adding all of them, it becomes `-122364` (the output!). The equations becomes: `-(p * r) + (p*q) + r - q + r - r - q + q`. Simplified: `-(p * r) + (p*q) + r - q`. (Looking at this, we can ignore 5 through 8!)

 Let's do the exact same thing, but give inputs of 4 every time, so the M matrix would look like:
```
M = [
        [p, 0, 4, 0, 4],
        [0, 4, 0, 4, 0],
        [4, 0, 4, 0, 4],
        [0, q, 0, r, 0],
        [4, 0, 4, 0, 0],
    ]
```

I am gonna do the exact same thing as above:
1. -8556800 = `-(p * 64 * r)`
2. 646400 = `p * q * 64`
3. 342272 = ` 256 * r`
4. 25856 = `256 * q`

Let's add them. It's the same as "res"! We did'nt have to do 5 through 8 cause it's just adding and then subtracting the same variable, canceling it out.
The e
