---
layout: single
title:  "Protect the Vaccine"
date:   2021-02-28 12:23:31 +0800
categories: dso-nus-ctf-2021
permalink: /dso-nus-ctf-2021/protect-the-vaccine
---

This is the second challenge from the Web & Crypto section of DSO-NUS CTF 2021. The challenge introduced a new LSB attack on Special-Structured RSA Primes.

## Statement

A nation-supported hacker group is using their cutting edge technology to attack a company that develops vaccine. They roll their own crypto with a hope that it will be more secure. Luckily, we have got some of their crypto system information and also have found some research that is likely to break their crypto system. I heard you are a cipher breaker, could you help us to decrypt their secret and protect the vaccine from their plan?

A research paper on the attack, encryptor.py and data.txt is attached with the question.

[PDF](/assets/pdf/protect-the-vaccine.pdf)

<button class="collapsible btn" id="encryptor">encryptor.py</button>

<div class="content" id="encryptordata" style="display:none" markdown="1">
```python
from config import a,b,m,r_p,r_q,secret
from Crypto.Util.number import bytes_to_long

p = a**m + r_p
q = b**m + r_q
N = p*q
e = 65537

M = bytes_to_long(secret)
c = pow(M, e, N)

print('N:', N)
print('e:', e)
print('r_p:', r_p)
print('r_q:', r_q)
print('c:', c)
```
</div>

<button class="collapsible btn" id="data">data.txt</button>

<div class="content" id="datadata" style="display:none" markdown="1">
```python
N: 3275733051034358984052873301763419226982953208866734590577442123100212241755791923555521543209801099055699081707325573295107810120279016450478569963727745375599027892100123044479660797401966572267597729137245240398252709789403914717981992805267568330238483858915840720285089128695716116366797390222336632152162599116524881401005018469215424916742801818134711336300828503706379381178900753467864554260446708842162773345348298157467411926079756092147544497068000233007477191578333572784654318537785544709699328915760518608291118807464400785836835778315009377442766842129158923286952014836265426233094717963075689446543
e: 65537
r_p: 5555
r_q: 2021
c: 1556192154031991594732510705883546583096229743096303430901374706824505750761088363281890335979653013911714293502545423757924361475736093242401222947901355869932133190452403616496603786871994754637823336368216836022953863014593342644392369877974990401809731572974216127814977558172171864993498081681595043521251475276813852699339208084848504200274031750249400405999547189108618939914820295837292164648879085448065561197691023430722069818332742153760012768834458654303088057879612122947985115227503445210002797443447539212535515235045439442675101339926607807561016634838677881127459579466831387538801957970278441177712
```
</div>

## Observation

This is also a very generous crypto challenge. The description of the attack is clearly stated in the attached research paper. What I have to do is just follow the example and find the flag.

## Solution

I will briefly explain the attack and skip all the proofs.

Suppose in a RSA cryptosystem,

$$p = a^{m} + r_{p}$$

$$q = b^{m} + r_{q}$$

$$N = pq$$

where $$m$$ is a positive even number and $$N,r_{p},r_{q}$$ is known

We can efficiently find $$p$$ and $$q$$ with a very interesting method described in the paper. 

For $$i$$ in $$(0,1,...,10000)$$

Set 

$$d = round(\sqrt{N} - i)$$ 

$$z \equiv N - (r_{p} \cdot r_{q}) \text{ (mod } d)$$

And solve the equation

$$X^{2} - zX + dr_{p}r_{q} = 0$$

Let $$x_{1},x_{2}$$ be two roots of the equation.

If $$\dfrac{x_{1}}{r_{q}} + r_{p}$$ and $$\dfrac{x_{2}}{r_{p}} + r_{q}$$ are integers,

Then $$p = \dfrac{N}{\dfrac{x_{1}}{r_{q}} + r_{p}}$$ and $$q = \dfrac{N}{\dfrac{x_{2}}{r_{p}} + r_{q}}$$

By knowing the $$p$$ and $$q$$ we can easily break the cryptosystem.

I use python with sage library to solve the polynomial and find the roots.

Here's the python code for the complete solution. 

```python
from sage.all import *
import gmpy2
from Crypto.Util.number import inverse, long_to_bytes

N = 3275733051034358984052873301763419226982953208866734590577442123100212241755791923555521543209801099055699081707325573295107810120279016450478569963727745375599027892100123044479660797401966572267597729137245240398252709789403914717981992805267568330238483858915840720285089128695716116366797390222336632152162599116524881401005018469215424916742801818134711336300828503706379381178900753467864554260446708842162773345348298157467411926079756092147544497068000233007477191578333572784654318537785544709699328915760518608291118807464400785836835778315009377442766842129158923286952014836265426233094717963075689446543
e = 65537
rp = 5555
rq = 2021
c = 1556192154031991594732510705883546583096229743096303430901374706824505750761088363281890335979653013911714293502545423757924361475736093242401222947901355869932133190452403616496603786871994754637823336368216836022953863014593342644392369877974990401809731572974216127814977558172171864993498081681595043521251475276813852699339208084848504200274031750249400405999547189108618939914820295837292164648879085448065561197691023430722069818332742153760012768834458654303088057879612122947985115227503445210002797443447539212535515235045439442675101339926607807561016634838677881127459579466831387538801957970278441177712

x = gen(PolynomialRing(ZZ,'x'))

for i in range(10000):
    d = int(pow(gmpy2.isqrt(N) - i,2))
    z = (N - rp*rq)%d
    f = x**2 - z*x + d*rp*rq
    sol = f.roots()
    if (len(sol) >= 2):
        x1 = sol[0][0]
        x2 = sol[1][0]
        if (x1 % rq == 0 and x2 % rp == 0):
            p = N // ((x1 // rq) + rp)
            q = N // ((x2 // rp) + rq)
            break
        if (x1 % rp == 0 and x2 % rq == 0):
            p = N // ((x1 // rp) + rq)
            q = N // ((x2 // rq) + rp)
            break


phi = (p-1)*(q-1)
d = inverse(e,phi)
print(long_to_bytes(pow(c,d,N)))
```

flag : `DSO-NUS{851f6c328f2da456cbc410184c7ada365c6d1f69199f0f4fdcb9fd43101ce9ee}`
