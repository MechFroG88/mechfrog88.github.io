---
layout: single
title:  "Gamer's Cipher"
date:   2021-01-18 11:23:31 +0800
categories: bamboofox-ctf-2021
permalink: /bamboofox-ctf-2021/gamers-cipher
---

This is the first challenge from the Cryptography section of BambooFox CTF 2021.

## Statement

We are given haskell implementation of a custom made Cipher that is based on Nim.

<button class="collapsible btn" id="nim">Nim.hs</button>

<div class="content" id="nimdata" style="display:none" markdown="1">
```haskell
module Nim where

import Data.Bits
import Math.NumberTheory.Logarithms


nimSize = 8
maxNim  = (shiftL 1 nimSize) - 1

newtype Nim = Nim Integer deriving (Eq)


bitMask :: Int -> Nim
bitMask n = Nim $ (shiftL 1 n) - 1

nimLength :: Nim -> Int
nimLength (Nim a) = bit $ intLog2 $ integerLog2 a

nimSplit :: Nim -> Int -> (Nim, Nim)
nimSplit a n =
    let higher = shiftR a n
        lower  = a .&. bitMask n
    in (higher, lower)


fromNim :: Nim -> Integer
fromNim (Nim a) = a


instance Num Nim where
    Nim a + Nim b = Nim (xor a b)
    Nim a - Nim b = Nim (xor a b)

    0 * _ = 0
    _ * 0 = 0
    a * 1 = a
    1 * b = b
    a * b =
        let (a1, a0) = nimSplit a k
            (b1, b0) = nimSplit b k
        in a1 * b1 * p2 + (shiftL (a0 * b1 + a1 * b0) k) + a0 * b0
        where
            k  = max (nimLength a) (nimLength b)
            p  = Nim $ bit k
            p2 = Nim $ bit k + bit (k - 1)

    abs = id
    signum 0 = 0
    signum _ = 1
    negate = id
    fromInteger a
        | a > maxNim = error "Nim overflow"
        | a < 0      = error "Nim cannot be negative"
        | otherwise  = Nim a

instance Bits Nim where
    (Nim a) .&. (Nim b) = Nim (a .&. b)
    (Nim a) .|. (Nim b) = Nim (a .|. b)
    xor (Nim a) (Nim b) = Nim (xor a b)

    complement (Nim a) = Nim (complement a)

    shiftR (Nim a) b = Nim (shiftR a b)
    shiftL (Nim a) b
        | c <= maxNim = Nim c
        | otherwise   = error "Nim overflow"
        where c = shiftL a b

    rotateR = shiftR
    rotateL = shiftL

    bitSize _ = nimSize
    bitSizeMaybe _ = Just nimSize
    isSigned _ = False

    bit a = Nim $ bit a

    testBit  (Nim a) = testBit a
    popCount (Nim a) = popCount a

instance Show Nim where
    show (Nim a) = show a
```
</div>



<button class="collapsible btn" id="main">Main.hs</button>

<div class="content" id="maindata" style="display:none" markdown="1">

```haskell
import Nim
import Data.Char (ord)


readNim :: IO Nim
readNim = fromInteger <$> read <$> getLine

encode :: [Char] -> [Nim]
encode x = map (fromIntegral . ord) x

isValidKey :: Nim -> Int -> Bool
isValidKey key n = isUnityRoot && isPrimitive
    where
        isUnityRoot = key ^ n == 1
        isPrimitive = not $ elem 1 [ key ^ i | i <- [1..n-1] ]

encrypt :: Nim -> [Nim] -> [Nim]
encrypt key message
    | isValidKey key n = zipWith (+) keys [ calc x | x <- keys ]
    | otherwise        = error "Invalid key"
    where
        n    = length message
        keys = pows key
        coef = reverse $ zipWith (*) message keys
        pows = \val -> [ val ^ i | i <- [0..n-1] ]
        calc = \val -> sum $ zipWith (*) coef (pows val)

main :: IO ()
main = do
    putStrLn "Enter flag to encrypt:"
    flag <- getLine
    putStrLn "Enter your key:"
    key <- readNim
    putStrLn "Your encrypted flag:"
    print $ encrypt key (encode flag)

```
</div>


<button class="collapsible btn" id="log">encryption.log</button>

<div class="content" id="logdata" style="display:none" markdown="1">


```
Enter flag to encrypt:
[DATA EXPUNGED]
Enter your key:
[REDACTED]
Your encrypted flag:
[13,1,114,230,244,145,218,78,204,36,81,48,148,35,40,50,78,40,88,43,122,39,41,149,208,208,191,68,65,61,224,140,18,239,104,210,110,119,178,27,173,253,15,237,85,192,82,74,148,15,250]
```

</div>

## Observation

I spent a lot of time analyzing the multiplication defined in the code only to realized that it is something well known online. The Nim group has well defined addition and multiplication which has additive inverse, multiplicative inverse, and addition and multiplication are both associative and commutative. There are also multiplicative identity 1 and additive identity 0. Here the wiki link, [Nimber](https://en.wikipedia.org/wiki/Nimber)

## Solution

The encrypted flag is basically means 

$$ flag[i] = key^{i} + f(key,i)$$

$$ f(key,i) = \sum_{j = 0}^{n-1} m[j] \cdot key^{j} \cdot (key^{i})^{n-1-j} $$

Note: The $$+$$ and $$ \cdot $$ are nim's addition and multiplication.

Now that's really great! Because we have n unknowns (which is each character of the flag) and n equations. We can then solve this using gaussian elimination.

Here's the python code for the complete solution. 

```python
import math

# Start Nim Definition

add = lambda x,y :x^y

def nimLength(a):
    return 2 ** int(math.log2(int(math.log2(a))))

def nimSplit(a,n):
    higher = a>>n
    lower = a & (2**n - 1)
    return (higher,lower)

def mul(a,b):
    if a == 0 or b == 0:
        return 0
    if a == 1:
        return b
    if b == 1:
        return a
    k = max(nimLength(a),nimLength(b))
    p = 2**k
    p2 = 2**k + 2**(k-1)
    a1, a0 = nimSplit(a,k)
    b1, b0 = nimSplit(b,k)

    ans1 = mul(mul(a1,b1),p2)
    ans2 = add(mul(a0,b1),mul(a1,b0)) << k
    ans3 = mul(a0,b0)

    return add(add(ans1,ans2),ans3)

def pow(a,b):
    ans = 1
    while (b > 0) :
        if (b & 1):
            ans = mul(ans,a)
        a = mul(a,a)
        b = b//2
    
    return ans

# End Nim Definition

c = [13,1,114,230,244,145,218,78,204,36,81,48,148,35,40,50,78,40,88,43,122,39,41,149,208,208,191,68,65,61,224,140,18,239,104,210,110,119,178,27,173,253,15,237,85,192,82,74,148,15,250]
n = len(c)

keys = []

# Generate all the valid keys
for key in range(2**8):
    t = [pow(key,i) for i in range(1,n)]
    if (pow(key,n) == 1 and t.count(1) == 0):
        keys = t
        break

# Generate the multiplicative inverse
inv = [0 for i in range(256)]
for i in range(256):
    for j in range(256):
        if (mul(i,j) == 1):
            inv[i] = j
            inv[j] = i

for key in keys:
    print(key)

    # Gaussian Elimination
    cc = []
    M = []
    for i in range(n):
        cc.append(add(c[i],pow(key,i)))
        arr = [pow(key,(i*j+n-1-j)%n) for j in range(n)]
        M.append(arr)
    
    for i in range(n):
        inve = inv[M[i][i]]
        cc[i] = mul(inve,cc[i])
        for k in range(n):
            M[i][k] = mul(M[i][k],inve)
        for j in range(i+1,n):
            cc[j] = add(cc[j],mul(M[j][i],cc[i]))
            for k in range(i+1,n):
                M[j][k] = add(M[j][k],mul(M[j][i],M[i][k]))
    
    cc.reverse()
    ans = []
    anss = ""

    # Backward Substitution
    for i in range(n):
        for j in range(i):
            cc[i] = add(cc[i],mul(ans[j],M[n-1-i][n-1-j]))
        ans.append(cc[i])     
        anss += chr(cc[i])
        
    if "flag" in anss:
        print(anss)    
   
```

flag : `flag{did_you_solve_with_dft_or_lagrange_polynomial}`
