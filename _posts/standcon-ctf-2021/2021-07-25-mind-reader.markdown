---
layout: single
title:  "Mind Reader"
date:   2021-07-24 12:03:00 +0800
categories: standcon-2021
permalink: /standcon-2021/mind-reader
---

This is a challenge from Cryptography section of Standcon 2021. The challenge wants us to break the PRNG system.

## Statement

>So you think you are the best mind reader in the entire galaxy? We shall see about that...

<button class="collapsible btn" id="data">mind_reader_release.py</button>

<div class="content" id="datadata" style="display:none" markdown="1">

```python
#!/usr/bin/env python

import random
from string import ascii_lowercase

life = 3

def printFlag():
    print("[REDACTED]")
    exit(0)
    
def banner():
    print(" ╔══════════════════════════════════════════════════════╗")
    print(" ║                                                      ║")
    print(" ║           ███╗   ███╗██╗███╗   ██╗██████╗            ║")
    print(" ║           ████╗ ████║██║████╗  ██║██╔══██╗           ║")
    print(" ║           ██╔████╔██║██║██╔██╗ ██║██║  ██║           ║")
    print(" ║           ██║╚██╔╝██║██║██║╚██╗██║██║  ██║           ║")
    print(" ║           ██║ ╚═╝ ██║██║██║ ╚████║██████╔╝           ║")
    print(" ║           ╚═╝     ╚═╝╚═╝╚═╝  ╚═══╝╚═════╝            ║")
    print(" ║                                                      ║")                                            
    print(" ║   ██████╗ ███████╗ █████╗ ██████╗ ███████╗██████╗    ║")
    print(" ║   ██╔══██╗██╔════╝██╔══██╗██╔══██╗██╔════╝██╔══██╗   ║")
    print(" ║   ██████╔╝█████╗  ███████║██║  ██║█████╗  ██████╔╝   ║")
    print(" ║   ██╔══██╗██╔══╝  ██╔══██║██║  ██║██╔══╝  ██╔══██╗   ║")
    print(" ║   ██║  ██║███████╗██║  ██║██████╔╝███████╗██║  ██║   ║")
    print(" ║   ╚═╝  ╚═╝╚══════╝╚═╝  ╚═╝╚═════╝ ╚══════╝╚═╝  ╚═╝   ║")
    print(" ║                                                      ║")
    print(" ╚══════════════════════════════════════════════════════╝")     

def printMenu():     
    print("=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=")
    print("1. Play game")
    print("2. View Top players")
    print("3. Exit")
    print("Please select your option")
    choice=input("> ")
    print("=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=")
    return choice

def readFile():
    firstName = open('./first-names.txt', 'r')
    lastName = open('./last-names.txt', 'r')
    game = open('./game.txt', 'r')

    fnArr = firstName.read().splitlines()
    lnArr = lastName.read().splitlines()
    gameArr = game.read().splitlines()

    return fnArr, lnArr, gameArr

def gen(initSeed, fnArr, lnArr, gameArr):
    genNameArr = []
    genHighscoreArr = []
    toPredictArr = []
    seed = []
    r = random.Random()
    r.seed(initSeed)
    for count in range(730):

        x = r.getrandbits(32)
        seed.append(x)
        if (count < 350):

            fnIndex = x & 0xFFFF
            lnIndex = x >> 16
            Name = " ".join([fnArr[fnIndex], lnArr[lnIndex]])
            genNameArr.append(Name)

        if (count >= 350 and count < 700):
            genHighscoreArr.append(x)

        elif(count >= 700):

            toPredictArr.append(gameArr[x & 0xFFFF])
    return genNameArr, genHighscoreArr, toPredictArr

def hangman(tried_letters):
    print ("LIFE: {0}".format(life))
    if len(tried_letters) >= 1:
        print("Used letters: {}".format(", ".join(letters for letters in tried_letters)))

def print_error(why):
    print(why)

def gameLogic(letters, random_word):
    global life
    tried_letters = []

    while True:
        try:
            hangman(tried_letters)

            for letter in letters:
                print(letter, end=' ')
            letter_input = str(input("\nLetter: ")).lower()

            if  letter_input == random_word:
                print('\nThe word is {0}. You win!'.format(random_word))
                return

            if len(letter_input) > 1 or letter_input not in ascii_lowercase:
                print_error("Invalid Input\nOnly single letters")
                continue

            if letter_input in tried_letters:
                print_error("You already tried this letter!")
                continue 
    
            else:
                tried_letters.append(letter_input)
                if letter_input in random_word:
                    
                    letter_index = [index for index, value in enumerate(random_word) if value == letter_input]
                    
                    if len(letter_index) > 1:
                        for i in letter_index:
                            letters[i] = letter_input
                    elif letter_input in letters:
                        print_error("You already tried this letter!")
                        continue
                    else:
                        letters[letter_index[0]] = letter_input
                else:
                    life -= 1
                
                if ''.join(letters) == random_word:
                    hangman(tried_letters)                
                    for letter in letters:
                        print(letter, end=' ')                
                    print('\nThe word is {0}. You win!'.format(random_word))
                    return

                elif life == 0:
                    hangman(tried_letters)
                    print('\nThe word is {0}'.format(random_word))
                    print("The Hangman was hanged. You loose!")
                    exit(0)

        except IndexError:
            print_error("Invalid Input\nOnly single Letters")
            continue

def playGame(toPredictArr):
    print("So you think you are a mind reader huh!?")
    print("Try to guess what i am thinking 30 times in a row and win the grand prize")
    print("I am even going to give u 3 chances, not like that would help >:)")
    
    for idx, words in enumerate(toPredictArr):
        letters = []
        for _ in range(len(words)):
                letters.append("_")
        print("=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-")
        print("Round " + str(idx+1))
        print("=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-")
        gameLogic(letters,words)

    if idx == 29:
        printFlag()

def viewTopPlayers(nameArr, highscoreArr):
    print("Now showing the top 350 players!!!")
    print("══════════════════════════════════")
    print('{0:25} {1}'.format("Name", "ID"))
    print("=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-")
    for x in range (len(nameArr)):
        print('{0:25} {1}'.format(nameArr[x], highscoreArr[x]))

def main():
    initSeed = random.getrandbits(32)
    fnArr, lnArr, gameArr = readFile()
    nameArr, highscoreArr, toPredictArr = gen(initSeed, fnArr, lnArr, gameArr)

    banner()

    while True:

        choice = printMenu()
        if(choice == '1'):
            playGame(toPredictArr)
        elif(choice == '2'):
            viewTopPlayers(nameArr, highscoreArr)
        elif(choice == '3'):
            exit()

if __name__ == "__main__":
    main()
    
```
</div>

## Observation

After reading the code, we can find out that the server is using Python library random to generate the namelist.

Looking at the python [documentation](https://docs.python.org/3/library/random.html) for random. They explicitly state that

>Warning : The pseudo-random generators of this module should not be used for security purposes. For security or cryptographic uses, see the secrets module.

Can we exploit the python random library?

## Solution

Yes we actually can! There's also an online [library](https://github.com/tna0y/Python-random-module-cracker) for that!

Internally python uses mersenne twister PRNG. 

I did not research on how the algorithm and the exploits work. 

Basically the PRNG is not cryptographically secure.

```python
import random
from randcrack import RandCrack
from pwn import *

def main():
    r = remote('20.198.209.142', 55003, level='debug')

    firstName = open('./files/first-names.txt', 'r').read().split('\n')
    lastName = open('./files/last-names.txt', 'r').read().split('\n')
    game = open('./files/game.txt', 'r').read().split('\n')

    dicFirst = {}
    dicLast = {}

    for i in range(len(firstName)):
        dicFirst[firstName[i]] = i

    for i in range(len(lastName)):
        dicLast[lastName[i]] = i

    arr = []
    highscore = []

    r.recvuntil('>')
    r.sendline('2')
    r.recvuntil('>')
    r.sendline('2')

    r.recvuntil('=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-\n')

    for i in range(350):
        rank = r.recvline().split()
        fn = rank[0].decode()
        ln = rank[1].decode()
        highscore.append(int(rank[2].decode()))
        arr.append((dicLast[ln] << 16) + dicFirst[fn])

    arr += highscore
    toPredictArr = []
    rc = RandCrack()
    for i in range(730):
        if (i < 624):
            rc.submit(arr[i])
        else:
            x = rc.predict_getrandbits(32)
            toPredictArr.append(game[x & 0xFFFF])

    toPredictArr = toPredictArr[-30:]
    r.recvuntil('>')
    r.sendline('1')

    for i in range(30):
        r.recvuntil('Letter:')
        r.sendline(toPredictArr[i])
    print(r.recvuntil(b'}'))

if __name__ == '__main__':
    main()        
```

flag : `STC{w0w_wHa7_An_UneXPec7eD_7Wi57}`
