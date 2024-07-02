# Without a Trace

- Description: Gone with the wind, can you find my flag? ncat --ssl without-a-trace.chal.uiuc.tf 1337

<details>
  <summary>Attachments</summary>

  [server.py](https://uiuctf-2024-rctf-challenge-uploads.storage.googleapis.com/uploads/c17ae894b0920b181b2d753a693fc678e6d0806045e3d7ba6519111dce60898b/server.py)
</details>

# Intro

We are provided with a `server.py` and a remote connection. Let's analyze the source.

# Analysis

The main function is as follows:

```python
def main():
    print("[WAT] Welcome")
    M = inputs()
    if M is None:
        print("[WAT] You tried something weird...")
        return
    elif check(M) == 0:
        print("[WAT] It's not going to be that easy...")
        return

    res = fun(M)
    if res == None:
        print("[WAT] You tried something weird...")
        return
    print(f"[WAT] Have fun: {res}")
```

First, it takes input. Let's look at the input function as well:
```python
def inputs():
    print("[WAT] Define diag(u1, u2, u3. u4, u5)")
    M = [
        [0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0],
    ]
    for i in range(5):
        try:
            M[i][i] = int(input(f"[WAT] u{i + 1} = "))
        except:
            return None
    return M
```

It takes input and stores it diagonally based on our input. If I give 1,2,3,4,5, the matrix would look like:
```
M = [
        [1, 0, 0, 0, 0],
        [0, 2, 0, 0, 0],
        [0, 0, 3, 0, 0],
        [0, 0, 0, 4, 0],
        [0, 0, 0, 0, 5],
]
```

Afterward, it calls the `check()` function. It's highly complicated, but basically, it checks that all the numbers we provided should be greater than 0.
Then, It calls the `fun()` function and gives the output. The `fun()` function stores the flag in another matrix diagonally (converts to integer, splits the flag into 5 parts, stores). So something like:
```
F = [
        [10, 0, 0, 0, 0],
        [0, 20, 0, 0, 0],
        [0, 0, 30, 0, 0],
        [0, 0, 0, 40, 0],
        [0, 0, 0, 0, 50],
]

```
Here the numbers are the flag split into 5 parts, represented as integers. It then uses numpy's matrix multiplication and trace to generate "res".

# Trial-and-error

I make a flag on my own: `uiuctf{flag}` I give `1` as input for the M matrix. It looks like:
```
M = [
  [1,0,0,0,0],
  [0,1,0,0,0],
  [0,0,1,0,0],
  [0,0,0,1,0],
  [0,0,0,0,1]
]
```
Looking at "res", It is actually the flag parts added together! The flag matrix on the local flag is:
```
F= [
[504280474484, 0, 0, 0, 0],
[0, 440156974177, 0, 0, 0],
[0, 0, 26493, 0, 0],
[0, 0, 0, 0, 0],
[0, 0, 0, 0, 0]
]
```
Adding the three numbers together gives me "res"! Let's try again, but a little differently

For the 5 inputs for the M matrix, give `2` for the first one and `1` for the rest. The M matrix looks like:
```
M = [
  [2,0,0,0,0],
  [0,1,0,0,0],
  [0,0,1,0,0],
  [0,0,0,1,0],
  [0,0,0,0,1]
]
```
The output is 1448717949638. It's the same as `2*504280474484 + 440156974177 + 26493` (from the above flag matrix)! 

# Solve

We have 5 variables (let's call them `a` through `e`). If I give all ones, the equation becomes: `a+b+c+d+e = output_1` and when I give one `2` and the rest as `1`, the equation becomes: `2*a+b+c+d+e = output_2`. Subtracting these two equations leaves us with `a`! Subtract output_2 - output_1 = a (first part of the flag). Do this with the other ones as well, convert to bytes, then join them to get the flag!

# Flag - uiuctf{tr4c1ng_&&_mult5!}
