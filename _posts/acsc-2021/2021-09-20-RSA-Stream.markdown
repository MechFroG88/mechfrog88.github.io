---
layout: single
title:  "RSA stream"
date:   2021-09-20 11:23:31 +0800
categories: acsc-2021
permalink: /acsc-2021/rsa-stream
---

This is the first crypto challenge from ASCS 2021. It used Stream cipher with RSA to encrypt a message.

## Statement

>I made a stream cipher out of RSA! But people say I made a huge mistake. Can you decrypt my cipher?

<button class="collapsible btn" id="data">chal.py</button>

<div class="content" id="datadata" style="display:none" markdown="1">

```python
import gmpy2
from Crypto.Util.number import long_to_bytes, bytes_to_long, getStrongPrime, inverse
from Crypto.Util.Padding import pad

from flag import m
#m = b"ACSC{<REDACTED>}" # flag!

f = open("chal.py","rb").read() # I'll encrypt myself!
print("len:",len(f))
p = getStrongPrime(1024)
q = getStrongPrime(1024)

n = p * q
e = 0x10001
print("n =",n)
print("e =",e)
print("# flag length:",len(m))
m = pad(m, 255)
m = bytes_to_long(m)

assert m < n
stream = pow(m,e,n)
cipher = b""

for a in range(0,len(f),256):
  q = f[a:a+256]
  if len(q) < 256:q = pad(q, 256)
  q = bytes_to_long(q)
  c = stream ^ q
  cipher += long_to_bytes(c,256)
  e = gmpy2.next_prime(e)
  stream = pow(m,e,n)

open("chal.enc","wb").write(cipher)
```
</div>

## Observation

The plaintext used for stream cipher is the file itself, which we already know.

Define $$f(m, e) = m^{e} \text{ mod } (N)$$

The OTP used for stream cipher are $$f(m, e_{1}), f(m, e_{2}), ..., f(m, e_{n})$$

where $$e_{i + 1} = \text{nextprime}(e_{i})$$

The objective is to find $$m$$

## Solution

Since the plaintext is known, it is easy to find out the OTP used by just xor the ciphertext with the plaintext.

$$f(m, e_{i}) = c_{i} \oplus p_{i}$$

Because $$e_{1}$$ and $$e_{2}$$ are coprime, by Bezout's lemma

$$a\cdot e_{1} + b\cdot e_{2} = 1$$

where $$a$$ and $$b$$ are integers.

$$a$$ and $$b$$ can be found using extended euclidean algorithm.

$$m$$ is then revealed by 

$$
\begin{aligned}
(m^{e_{1}})^{a} \cdot (m^{e_{2}})^{b} &\equiv m^{a\cdot e_{1}} \cdot m^{b\cdot e_{2}} \text{ mod} (N)\\
&\equiv m^{a\cdot e_{1} + b\cdot e_{2}} \text{ mod} (N)\\
&\equiv m^{1} \text{ mod} (N)\\
\end{aligned}
$$

I used sage to do extended euclidean algorithm, the code is shown below.

```python
from Crypto.Util.number import bytes_to_long, long_to_bytes

import gmpy2
from sage.all import *
from Crypto.Util.number import long_to_bytes

def xor(s1, s2):
    return bytes([i ^ j for i,j in zip(s1,s2)])

f1 = open("chal.py","rb").read()
f2 = open("chal.enc","rb").read()

n = 30004084769852356813752671105440339608383648259855991408799224369989221653141334011858388637782175392790629156827256797420595802457583565986882788667881921499468599322171673433298609987641468458633972069634856384101309327514278697390639738321868622386439249269795058985584353709739777081110979765232599757976759602245965314332404529910828253037394397471102918877473504943490285635862702543408002577628022054766664695619542702081689509713681170425764579507127909155563775027797744930354455708003402706090094588522963730499563711811899945647475596034599946875728770617584380135377604299815872040514361551864698426189453
e = 65537
primes = []
res = []

for a in range(0,len(f1),256):
    res.append(bytes_to_long(f1[a:a+256]) ^ bytes_to_long(f2[a:a+256]))
    primes.append(e)
    e = gmpy2.next_prime(e)

d,u,v = xgcd(primes[0],primes[1])
p1 = Zmod(n)(res[0])
p2 = Zmod(n)(res[1])
print(d, u, v)
print(long_to_bytes((p1**u) * (p2**v)))
```

flag : `ACSC{changing_e_is_too_bad_idea_1119332842ed9c60c9917165c57dbd7072b016d5b683b67aba6a648456db189c}`
