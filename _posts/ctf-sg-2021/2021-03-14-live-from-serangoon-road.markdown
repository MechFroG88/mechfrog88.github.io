---
layout: single
title:  "Live From Serangoon Road"
date:   2021-03-14 13:23:31 +0800
categories: ctf-sg-2021
permalink: /ctf-sg-2021/live-from-serangoon-road
---

This is the third challenge from the Crypto section of CTF SG 2021. The challenge needs us to break a Stream Cipher that uses 4 LFSR.

## Statement

>Damnit, this telco is down again. It's fine, we know exactly how they encrypted the message, maybe you might be able to help us decode it?

The encryption python code, encrypted bytes, first few words of the decrypted text is provided.

<button class="collapsible btn" id="encrypt">encrypt.py</button>

<div class="content" id="encryptdata" style="display:none" markdown="1">

```python
#! /usr/bin/python3
import random

KEYS = [8, 8, 24, 24]
TAPS = [96, 195, 1310720, 589824]

class LSFR:
    def __init__(self, state, taps, length):
        self.taps = taps
        self.state = state
        self.length = 2**length

    def getNext(self):
        out = 0
        xored = self.taps & self.state
        while xored > 0:
            if xored % 2 == 1:
                out = 1 - out
            xored //= 2
        self.state = (self.state*2 + out)%self.length
        return out

class StreamCipher:
    def __init__(self, func, *args):
        self.lsfr = list(args)
        self.func = func

    def encrypt(self, string):
        bits = []
        key = []
        for char in string:
            out = self.func(list(map(lambda x: x.getNext(), self.lsfr)))
            bits.append(char ^ out)
            key.append(out)
        enc = []
        while bits:
            enc.append(chr(int(''.join(map(str, bits[0:8])), 2)))
            bits = bits[8:]
        return ''.join(enc)

def main():
    # Unbruteforcable 64 bit key muahahaha
    while True:
        try :
            seed = random.randrange(2**sum(KEYS))

            # Seeding
            a = LSFR(seed % 2**KEYS[0], TAPS[0], KEYS[0]); seed //= 2**KEYS[0]
            b = LSFR(seed % 2**KEYS[1], TAPS[1], KEYS[1]); seed //= 2**KEYS[1]
            c = LSFR(seed % 2**KEYS[2], TAPS[2], KEYS[2]); seed //= 2**KEYS[2]
            d = LSFR(seed % 2**KEYS[3], TAPS[3], KEYS[3]); seed //= 2**KEYS[3]

            flag = "This is n\n Hello world can you hear me my name is bebebe"
            flag = map(int, ''.join('{0:08b}'.format(ord(x), 'b') for x in flag))

            cipher = StreamCipher(lambda l:((l[0]^l[1])&l[2])|(((~l[1])|l[2])&l[3]), a, b, c, d)
            file = open("out.out", "w")
            file.write(cipher.encrypt(flag))
            file.close()
            break
        except :
            continue
        
    

main()
```
</div>

<button class="collapsible btn" id="decrypted">decrypted.txt</button>

<div class="content" id="decrypteddata" style="display:none" markdown="1">

```
This is n

```

</div>

## Observation

The challenge provided us the taps, key length of all the 4 LFSR. 

The only thing that we don't know is the initial seed of the LFSR.

Let $$a[i],b[i],c[i],d[i]$$ be the i-th outputs of the 4 LFSR respectively

Let $$cip[i]$$ be the i-th bit of the ciphertext 

Let $$msg[i]$$ be the i-th bit of the plaintext 

Define a function 

$$f(a,b,c,d) = ((a \oplus b) \land c) \lor ((\neg b \lor c) \land d)$$

The key for i-th bit is $$f(a[i],b[i],c[i],d[i])$$

$$cip[i] = msg[i] \oplus f(a[i],b[i],c[i],d[i])$$

## Solution

I analyzed $$f(a,b,c,d)$$ and found some interesting property from it.

Here's the truth table of $$f(a,b,c,d)$$

$$
\begin{array}{|c|c|c|c|c|}
    \hline
    a & b & c & d & f(a,b,c,d)\\
    \hline
    \hline
    1 & 1 & 1 & 1 & 1\\
    \hline
    1 & 1 & 1 & 0 & 0\\
    \hline
    1 & 1 & 0 & 1 & 0\\
    \hline
    1 & 1 & 0 & 0 & 0\\
    \hline
    \hline
    1 & 0 & 1 & 1 & 1\\
    \hline
    1 & 0 & 1 & 0 & 1\\
    \hline
    1 & 0 & 0 & 1 & 1\\
    \hline
    1 & 0 & 0 & 0 & 0\\
    \hline
    \hline
    0 & 1 & 1 & 1 & 1\\
    \hline
    0 & 1 & 1 & 0 & 1\\
    \hline
    0 & 1 & 0 & 1 & 0\\
    \hline
    0 & 1 & 0 & 0 & 0\\
    \hline
    \hline
    0 & 0 & 1 & 1 & 1\\
    \hline
    0 & 0 & 1 & 0 & 0\\
    \hline
    0 & 0 & 0 & 1 & 1\\
    \hline
    0 & 0 & 0 & 0 & 0\\
    \hline
    \hline
\end{array}
$$

Notice that

if $$a = 1$$ and $$b = 1$$ then $$f(a,b,c,d) = c \land d$$

if $$a = 1$$ and $$b = 0$$ then $$f(a,b,c,d) = c \lor d$$

if $$a = 0$$ and $$b = 1$$ then $$f(a,b,c,d) = c$$

if $$a = 0$$ and $$b = 0$$ then $$f(a,b,c,d) = d$$

Nice!! This can help me to brute force the key in a smarter way.

So to solve this challenge, I brute force for the seed for LFSR 1 and LFSR 2

This is doable because LFSR 1 and LFSR 2 only have key length of 8.

On the other hand, LFSR 3 and LFSR 4 have key length of 24.

Tap for LFSR 3 is 1310720 `(0b000101000000000000000000)`. Which means that

$$c[i+24] = c[i+3] \oplus c[i+5]$$

Tap for LFSR 4 is 589824 `(0b000010010000000000000000)`. Which means that

$$d[i+24] = d[i+4] \oplus d[i+7]$$

I can't just try every key from 0 to $$2^{24}$$ for LFSR 3 and LFSR 4 due to the long key length. However, the number of possible keys are greatly reduced with all these constraints based on the plaintext.

$$
c[i+24] = c[i+3] \oplus c[i+5]\\
d[i+24] = d[i+4] \oplus d[i+7]\\
f(a[i],b[i],c[i],d[i]) = msg[i] \oplus cip[i]\\
$$

My code can find solution in less than 2 minutes on my laptop. It uses a backtracking method similar to how we solve sudoku and 8 queen problem.

Z3 is also possible but somehow it just doesn't work for me.

Here's the python code for the complete solution.

```python
from Crypto.Util.number import long_to_bytes

KEYS = [8, 8, 24, 24]
TAPS = [96, 195, 1310720, 589824]

enc = open('encrypted.enc','rb').read()
pre = open('decrypted.txt','rb').read()

enc = list(map(int, ''.join('{0:08b}'.format(x, 'b') for x in enc)))
pre = list(map(int, ''.join('{0:08b}'.format(x, 'b') for x in pre)))

def f(a,b,c,d):
    return ((a^b)&c)|(((~b)|c)&d)

class LSFR:
    def __init__(self, state, taps, length):
        self.taps = taps
        self.state = state
        self.length = 2**length

    def getNext(self):
        out = 0
        xored = self.taps & self.state
        while xored > 0:
            if xored % 2 == 1:
                out = 1 - out
            xored //= 2
        self.state = (self.state*2 + out)%self.length
        return out

class StreamCipher:
    def __init__(self, func, *args):
        self.lsfr = list(args)
        self.func = func

    def encrypt(self, string):
        bits = []
        key = []
        for char in string:
            out = self.func(list(map(lambda x: x.getNext(), self.lsfr)))
            bits.append(char ^ out)
            key.append(out)
        enc = []
        while bits:
            enc.append(chr(int(''.join(map(str, bits[0:8])), 2)))
            bits = bits[8:]
        return ''.join(enc)

c = [-1 for i in range(len(enc))]
d = [-1 for i in range(len(enc))]

for i in range(2**KEYS[0]):
    print(i)
    for j in range(2**KEYS[1]):
        a = LSFR(i, TAPS[0], KEYS[0])
        b = LSFR(j, TAPS[1], KEYS[1])

        a = [a.getNext() for i in range(8+len(enc))][8:]
        b = [b.getNext() for i in range(8+len(enc))][8:]
        func(0)

def func(k) :
    q = k-8
    if (q >= 0 and q + 24 < len(pre)) :
        i = q + 24
        t1 = a[i]
        t2 = b[i]
        t3 = c[q+3] ^ c[q+5]
        t4 = d[q+4] ^ d[q+7]
        t = pre[i] ^ enc[i]
        if (f(t1,t2,t3,t4) != t):
            return

    if (k == 24):
        cc = c.copy()
        dd = d.copy()
        for i in range(len(enc)-24):
            cc[24+i] = cc[i+3]^cc[i+5]
            dd[24+i] = dd[i+4]^dd[i+7]

        ans = b''
        for i in range(0,len(enc)-8,8):
            ca = 0
            for j in range(8):
                ca = ca << 1
                key = f(a[i+j],b[i+j],cc[i+j],dd[i+j])
                ca += key ^ enc[i+j]
            ans += chr(ca).encode()
        if (b'This is n' in ans):
            print(ans)
        return

    t1 = a[k]
    t2 = b[k]
    t = pre[k] ^ enc[k]

    if (t1 and t2):
        if (t):
            c[k] = 1
            d[k] = 1
            func(k+1)
        else :
            c[k] = 1
            d[k] = 0
            func(k+1)
            c[k] = 0
            d[k] = 1
            func(k+1)
            c[k] = 0
            d[k] = 0
            func(k+1)
    if (t1 and not t2):
        if (t):
            c[k] = 1
            d[k] = 1
            func(k+1)
            c[k] = 1
            d[k] = 0
            func(k+1)
            c[k] = 0
            d[k] = 1
            func(k+1)
        else :
            c[k] = 0
            d[k] = 0
            func(k+1)
    if (not t1 and t2):
        c[k] = t
        d[k] = 1
        func(k+1)
        c[k] = t
        d[k] = 0
        func(k+1)
    if (not t1 and not t2):
        c[k] = 0
        d[k] = t
        func(k+1)
        c[k] = 1
        d[k] = t
        func(k+1)
    c[k] = -1
    d[k] = -1
```

flag : `CTFSG{C0rRel4t10N_AtT4cK$_0n_LsFR_101}`
