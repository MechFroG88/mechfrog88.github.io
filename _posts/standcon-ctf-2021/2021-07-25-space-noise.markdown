---
layout: single
title:  "Space Noise"
date:   2021-07-24 12:06:00 +0800
categories: standcon-2021
permalink: /standcon-2021/space-noise
---

This is a challenge from Cryptography section of Standcon 2021. The challenge wants us to guess the encryption scheme

## Statement

>We just intercepted a secret transmission from the Secret Space Agency, but the traffic looks really weird... Wireshark shows so much red! Can you help us to figure out what's going on?

A pcap file is attached

## Observation

Once we open the file in Wireshark we see many different colors.

![Space Noise](/assets/images/standcon-2021/noise.png)

Observe that between 2 black lines there are bunch of blue and red lines. So the 2 black lines might be a seperator.

## Solution

Extract the packets to a json file.

I guessed that the red lines and blue lines actually representing 1 and 0.

By guessing the text starts with 'STC{', I found out that the encryption scheme used is actually  morse code. 

Decrypt it and we get hex data. Decode the hex data to get the flag.

某 crypto 大神曾经说过 :
> 没有 source 的 crypto 题目都是來闹的

```python
import json

f = open('packet.json')

jsonData = json.load(f)

data = []

for i in jsonData:
    color = i['_source']['layers']['frame']['frame.coloring_rule.name']
    if (color == 'Bad TCP'):
        data.append('2')
    elif (color == 'TCP'):
        data.append('0')
    else:
        data.append('1')

arr = (''.join(data)).split('22')

arr = arr[0].split('20') + arr[1:-1]

mapp = {
    '00000' : '0', 
    '10000' : '1',
    '11000' : '2',
    '11100' : '3', 
    '11110' : '4', 
    '11111' : '5',
    '01111' : '6',
    '00111' : '7', 
    '00011' : '8',
    '00001' : '9',    
    '0111'  : 'b',
    '0101'  : 'c',
    '011'   : 'd',
    '1'     : 'e', 
    '1101'  : 'f',
}

hexa = ''
for i in arr:
    hexa += mapp[i]

print(bytes.fromhex(hexa))
```

flag : `STC{I believe that this Nation should commit itself to achieving the goal, before this decade is out, of landing a man on the Moon and returning him safely to Earth.}`
