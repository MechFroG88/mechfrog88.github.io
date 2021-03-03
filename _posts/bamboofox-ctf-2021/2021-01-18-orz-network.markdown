---
layout: single
title:  "ORZ network"
date:   2021-01-18 10:23:31 +0800
categories: bamboofox-ctf-2021
permalink: /bamboofox-ctf-2021/orz-network
---

This is the third challenge from the Cryptography section of BambooFox CTF 2021. The challenge wants us determine if a Diffie-hellman parameter is vulnerable find the MST of a network graph.

## Statement

```
Say hello to the newest orz network.

There are 420 computers in this network, labeled from #1 to #420.
For each pair of computers, there may be a secure connection between them.
Each connection was initialized with a Diffie-Hellman key exchange under a multiplicative group modulo some prime number.

Today you and your hacker friend are given a mission: hack into and control the entire orz network.

Your friend had already hacked into and gained access to computer #1.
If you have access to computer #A and have already hijacked the connection between computers #A and #B, then you can gain access to computer #B.
To hijack a connection, all you have to do is to steal that connection's shared secret key (from the key exchange).

However, the admins of orz network are very experienced, so you can only hijack at most 419 connections before getting caught.
If your attack takes over 5 seconds, then you'll also get caught.

Fortunately, you are given the details of all connections.
You speak to yourself, "Isn't this the classic graph algorithm problem that I've solved a million times?"
Well, yes. Find a list of 419 connections to hijack, so that you can gain access to every computer.
Complete the mission, and you will be awarded the flag.


Notice: timer starts *after* the server sends you all the logs. Generating logs may take a few seconds, please be patient.
```

## Observation

Our task in this problem is to construct a minimum spanning tree in the network. We can view computers as vertices, Diffie-Hellman key as edges. To include an edge in our graph, we will have to find out the Diffie-Hellman shared key between the 2 computers.

We can find out Diffie-Hellman shared key with Pohlig–Hellman algorithm.

## Solution

Since our attack cannot takes more than 5 seconds, we should prioritize the edges that take less time to attack i.e. The time taken by Pohlig–Hellman algorithm is short.

We know that the runtime of Pohlig–Hellman algorithm depends on the smoothness of the order of the group. Therefore I can define the weight of the edge as $$100-i$$ where $$i$$ is the largest value such that $$2^{i}$$ is a factor of the group order.

I can then use Kruskal algorithm to find the Minimum Spanning Tree.


Here's the python code for the complete solution. 

```python
from pwn import *
from sympy.ntheory.residue_ntheory import discrete_log
import hashlib
import sys

# Proof of Work START
difficulty = 20
zeros = '0' * difficulty

r = remote('chall.ctf.bamboofox.tw',10369,level='debug')

def is_valid(digest):
    if sys.version_info.major == 2:
        digest = [ord(i) for i in digest]
    bits = ''.join(bin(i)[2:].zfill(8) for i in digest)
    return bits[:difficulty] == zeros

r.recvuntil('sha256(')
prefix = str(r.recvuntil(' ')[:-1])[2:-1]
r.recvuntil(': ')
i = 0
while True:
    i += 1
    s = prefix + str(i)
    if is_valid(hashlib.sha256(s.encode()).digest()):
        r.sendline(str(i))
        break

time.sleep(2)
n = 420
r.recvuntil('computers and ')
e = int(r.recvuntil(' connections')[:4])
r.recvuntil(': ')
r.sendline('\n')

# Proof of Work END

print("start now")

edges = []

def order(n):
    a = 100
    n = (n-1)
    while(n % 2 == 0):
        a -= 1
        n = n//2
    return a

# Disjoin Set Union Start
id = [-1 for i in range(n+1)]

def find(i):
    if (id[i] < 0):
        return i
    id[i] = find(id[i])
    return id[i]

def unio(i,j):
    a = find(i)
    b = find(j)
    if (a == b):
        return
    if (id[a] > id[b]):
        a,b = b,a
    id[a] += id[b]
    id[b] = a
# Disjoin Set Union End

# Input Start
for i in range(e):
    l1 = r.recvline().split()
    l2 = r.recvline().split()
    u = int((l1[5])[1:])
    v = int((l1[9])[1:])
    mod = int(l2[4][:-1])
    base = int(l2[7][:-1])
    A = int(l2[12][:-1])
    B = int(l2[-1][:])
    edges.append((u,v,mod,base,A,B,order(mod)))

# Input End
print("Timer Start")

# Kruskal Start
edges.sort(key = lambda x: x[-1])

answer = ""
for edge in edges:
    u,v,mod,base,A,B,_ = edge
    if (find(u) != find(v)):
        unio(u,v)
    else :
        continue
    a = discrete_log(mod,A,base)
    ans = str(pow(B,a,mod))
    r.send(ans + " ")

# Kruskal end

print("end")
r.send('\n')
time.sleep(5)
while True:
    r.recvline()
```

flag : `flag{orz_0TZ_0rz_Orz_OTZ_oTZ}`
