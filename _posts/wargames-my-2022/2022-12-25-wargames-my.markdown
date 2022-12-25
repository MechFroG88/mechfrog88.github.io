---
layout: single
title:  "Wargames My 2022"
date:   2022-12-25 12:01:00 +0800
categories: wargames-2022
permalink: /wargames-2022
---

Writeup for the challenges I solved in Wargames My 2022

This is my third time participating in Wargames My. Thanks to my teammates [@DavidTan0527](https://github.com/DavidTan0527) and [@1001mei](https://github.com/1001mei) for carrying the team (`iMonkee Ultra Max`) 😆

## Misc

### Secure Dream 1.0



> Let me know your dreams. Could your dreams bypass my expectation?

<button class="collapsible btn" id="data2">server.py</button>

<div class="content" id="data2data" style="display:none" markdown="1">

```python
#!/usr/bin/env python3

print('''\033[94m
♡ ☆ .♡‧₊˚ 
╭◜◝ ͡ ◜◝╮  ㅤ ╭◜◝ ͡ ◜◝╮. 
(             )  ♡   (             )☆ ♡
╰◟◞ ͜ ◟◞╭◜◝ ͡ ◜◝╮ ͜ ◟◞╯♡ 
. ☆  ㅤㅤ(                )☆ ♡
♡        ╰◟◞ ͜ ◟◞╯ . ☆

    [Secure Dream v1.0]
\033[0m''')

payload = input("What is your dream in life?\n")
if any(filter(lambda c: c in 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"\'', payload)):
    print("\nA𝒘𝒘... W𝒆 𝒅𝒐𝒏'𝒕 𝒖𝒏𝒅𝒆𝒓𝒔𝒕𝒂𝒏𝒅 𝒚𝒐𝒖𝒓 𝒅𝒓𝒆𝒂𝒎 :(")
else:
    eval(payload)

```

</div>

#### Observation

For this challenge, we are given a server endpoint that will take in user input and run the `eval` function.

The payload, however, must not contain any lowercase or uppercase letter or string literal

The goal of this challenge is to write the flag from the server

#### Solution

At first, I was searching for a method to code python without alphabets, apparently, this is [possible](https://ctftime.org/writeup/10726) in python2 with backtick ``. But since the server was running on python3, this method will not work.

I got stuck for a while until the organizers released this hint

> Is print(1) == 𝒑𝒓𝒊𝒏𝒕(1) ? Maybe? Really?

Observe that the first `print` is a regular string, and the second `𝒑𝒓𝒊𝒏𝒕` was italic.

I thought that this might mean that python accepts Unicode character as the function name, so I can just provide italic characters to bypass the filter

I opened a python terminal to try it out

```python
>>> 𝒑𝒓𝒊𝒏𝒕("hello")
hello
```

It works!

Great! Now that I can use every lowercase character, I just need to find a way to bypass the string literal restriction.

It is not difficult to do this by using the `bytes()` function

First I convert the code

`print(open('flag.txt').read())` to bytes

```python
[112, 114, 105, 110, 116, 40, 111, 112, 101, 110, 40, 34, 102, 108, 97, 103, 46, 116, 120, 116, 34, 41, 46, 114, 101, 97, 100, 40, 41, 41]
```

Then I convert it to bytes with the `bytes()` function and wrap it around again with `eval()` to execute the code

Final payload

```python
𝒆𝒗𝒂𝒍(𝒃𝒚𝒕𝒆𝒔([112, 114, 105, 110, 116, 40, 111, 112, 101, 110, 40, 34, 102, 108, 97, 103, 46, 116, 120, 116, 34, 41, 46, 114, 101, 97, 100, 40, 41, 41]).𝒅𝒆𝒄𝒐𝒅𝒆())
```

### Secure Dream 2.0



> Can you really bypass another dreams?


<button class="collapsible btn" id="data">server.py</button>

<div class="content" id="datadata" style="display:none" markdown="1">


```python
#!/usr/bin/env python3

print('''\033[94m
♡ ☆ .♡‧₊˚ 
╭◜◝ ͡ ◜◝╮  ㅤ ╭◜◝ ͡ ◜◝╮. 
(             )  ♡   (             )☆ ♡
╰◟◞ ͜ ◟◞╭◜◝ ͡ ◜◝╮ ͜ ◟◞╯♡ 
. ☆  ㅤㅤ(                )☆ ♡
♡        ╰◟◞ ͜ ◟◞╯ . ☆

    [Secure Dream v1.0]
\033[0m''')

payload = input("What is your dream in life?\n")
if any(filter(lambda c: c in 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"\'+', payload)):
    print("\nA𝒘𝒘... W𝒆 𝒅𝒐𝒏'𝒕 𝒖𝒏𝒅𝒆𝒓𝒔𝒕𝒂𝒏𝒅 𝒚𝒐𝒖𝒓 𝒅𝒓𝒆𝒂𝒎 :(")
else:
    eval(payload)

```

</div>

#### Observation

Secure Dream v2.0 is just the same as v1.0 but with `+` character blacklisted

#### Solution

My solution for v1.0 did not use the `+` character so I solved this question with the same payload

Final payload

```python
𝒆𝒗𝒂𝒍(𝒃𝒚𝒕𝒆𝒔([112, 114, 105, 110, 116, 40, 111, 112, 101, 110, 40, 34, 102, 108, 97, 103, 46, 116, 120, 116, 34, 41, 46, 114, 101, 97, 100, 40, 41, 41]).𝒅𝒆𝒄𝒐𝒅𝒆())
```

## OSINT

### Where Am I



> Find the place. (Caught in a landside, No escape from reality)

![whereami.jpg](/assets/images/wargames-my-2022/whereami.jpg)

#### Solution

The organizer has hinted that all the flags are in `wgmy{}` format and we are not required to add the flag format ourselves.

So I started to think where can one provide a flag with the image as a hint? 

And I immediately thought of social media and google reviews.

But since the question was `Where Am I` so google review might be the more probable option.

I searched for all [Texas Chicken outlets](https://www.texaschickenmalaysia.com/nearest-texas-chicken-store/) in Malaysia to find the outlet that looked like the picture.

Anddd I found it! The flag was captioned below the image

![Texas Chicken midvalley](/assets/images/wargames-my-2022/whereami2.png)

### Who Am I

> Find me. (Is this the real life? Is this just fantasy?)

![whoami.png](/assets/images/wargames-my-2022/whoami.png)

#### Solution

This seems like a registration page, but the registration on the main website had already closed. 

Again, all the flags are in `wgmy{}` format and we are not required to add the flag format ourselves.

The best place to put the flag will be on social media.

So I looked through the [facebook page](https://www.facebook.com/wargames.my/) of wgmy and found the original image!

![whoami2.jpg](/assets/images/wargames-my-2022/whoami2.jpg)

Although there are no comments related to the flag on the post, the image contains weird symbols on the side.

I looked at it closely and guess that it was wingdings. (and it really was XD)

With some manual work of decoding it, I then got the flag.

## Web

### Christmas Wishlist


> Submit your wishlist at this website!

<button class="collapsible btn" id="server">server.py</button>

<div class="content" id="serverdata" style="display:none" markdown="1">

```python
from flask import Flask, request, render_template, render_template_string
from waitress import serve
import os
import subprocess

app_dir = os.path.split(os.path.realpath(__file__))[0]
app = Flask(__name__)
app.config['UPLOAD_FOLDER'] = f'{app_dir}/upload/'

@app.route('/', methods=['GET','POST'])
def index():
    try:
        if request.method == 'GET':
            return render_template('index.html')

        elif request.method == 'POST':
            f = request.files['file']
            filepath = os.path.join(app.config['UPLOAD_FOLDER'], f.filename)

            if os.path.exists(filepath) or ".." in filepath:
                return render_template_string("Hohoho.. No present for you")

            else:
                f.save(filepath)
                output = subprocess.check_output(
                    ["/bin/file",'-b', filepath], 
                    shell=False, 
                    encoding='utf-8',
                    timeout=1
                )
                
                if "ASCII text" not in output:
                    output=f"<p style='color:red'>Error: The file is not a text file: {output}</p>"
                else:
                    output = "You wish for "
                    with open(filepath,'r') as f:
                        lines = f.readlines()
                        output += ', '.join(lines[:-1]) + " and " + lines[-1]

                os.remove(filepath)
                return render_template_string(output)

    except:
        return render_template_string("Error")

if __name__ == '__main__':
    serve(app, host="0.0.0.0", port=3000, threads=1000, cleanup_interval=30)
```

</div>

#### Observation

This is a flask app that provides an endpoint to upload a file. 

On every upload, the command `/bin/file -b <filepath>` will be run.

The server will then read the uploaded file and pass the content to the jinja template.

#### Solution

I am really bad at web challenges and have no idea where to start. I thought of command injection but that is prevented by `shell=False` 

I was stuck here for a few hours until I found a [writeup](https://fireshellsecurity.team/rctf2022-easyupload-filechecker-ezbypass/#filechecker-mini) online that looks very similar to this question. Then only I know about template injection.

The gist of the solution is we provide `{{something...}}` and the jinja template will run the command in the bracket. I just copied the payload from the writeup and got the solution

Final payload (filename doesn't matter)

File content
```
{% raw %}{{ request.__class__._load_form_data.__globals__.__builtins__.open("/flag").read() }}{% endraw %}
```

### Christmas Wishlist 2

> Someone exploited the previous website, I have upgraded you can submit your wishlist at my website again!

<button class="collapsible btn" id="server2">server.py</button>

<div class="content" id="server2data" style="display:none" markdown="1">

```python
from flask import Flask, request, render_template, render_template_string
from waitress import serve
import os
import subprocess

app_dir = os.path.split(os.path.realpath(__file__))[0]
app = Flask(__name__)
app.config['UPLOAD_FOLDER'] = f'{app_dir}/upload/'

@app.route('/', methods=['GET','POST'])
def index():
    try:
        if request.method == 'GET':
            return render_template('index.html')

        elif request.method == 'POST':
            f = request.files['file']
            filepath = os.path.join(app.config['UPLOAD_FOLDER'], f.filename)

            if os.path.exists(filepath) or ".." in filepath:
                return render_template_string("Hohoho.. No present for you")
            
            else:
                f.save(filepath)
                output = subprocess.check_output(
                    ["/bin/file",'-b', filepath], 
                    shell=False, 
                    encoding='utf-8',
                    timeout=1
                )
                
                if "ASCII text" not in output:
                    output=f"<p style='color:red'>Error: The file is not a text file: {output}</p>"
                else:
                    output="Wishlist received. Santa will check out soon!"

                os.remove(filepath)
                return render_template_string(output)

    except:
        return render_template_string("Error")

if __name__ == '__main__':
    serve(app, host="0.0.0.0", port=3000, threads=1000, cleanup_interval=30)
```
</div>

#### Observation

It looks pretty much the same as v1.0 just that the file will not be read if the `ASCII text` is in the file output command

Also, only the file command output will be added when `ASCII text` is not in the output

#### Solution

The writeup I found in v1.0 already accounted for this scenario, basically, you can put the template in the shebang of the file so that the file command will output it.

To make sure that it does not contains the word `ASCII text`, just add a Unicode character like an emoji or characters from another language in the file body.

Final payload (filename doesn't matter)

File content
```
#!/usr/bin/{% raw %}{{ request.__class__._load_form_data.__globals__.__builtins__.open("/flag").read() }}{% endraw %}

手动
```

## Steganography

### Color

> Please message us on discord if you are colorblind (Because I'm easy come, easy go, Little high, little low,)

<img src="/assets/images/wargames-my-2022/color.png" alt="color.png" style="width:300px;"/>

#### Solution

We get a QR code with multiple colours. I guessed that each colour will represent one part of the flag

So I used [stegonline](https://stegonline.georgeom.net/upload) to filter out the 3 different colours and scan each of the QR separately.

And the QR codes indeed represented one part of the flag! Combine them and solve the question

### Puzzle

> Is the original always better?
> Maybe, should we check? (I'm just a poor boy, I need no sympathy,)

<img src="/assets/images/wargames-my-2022/puzzle.jpg" alt="puzzle.jpg" style="width:500px;"/>

#### Observation

Look at the top part of the image carefully and we can see pieces of QR code. 

The organizer also provided the [original image](https://upload.wikimedia.org/wikipedia/commons/thumb/e/ec/Mona_Lisa%2C_by_Leonardo_da_Vinci%2C_from_C2RMF_retouched.jpg/800px-Mona_Lisa%2C_by_Leonardo_da_Vinci%2C_from_C2RMF_retouched.jpg) as the hint.

#### Solution

I load two images into python and observe the difference between each pixel. 

I first create a new image.

For each of the pixels, if both are the same then set the pixel to white.

If the maximum of `(Puzzle pixel - Original pixel)` for each of the colors (r,g,b) is negative, then I set the pixel to full black.

Else I set it to blue

```python
from PIL import Image

ori = Image.open('ori.jpg', 'r')
width, height = ori.size
oripxl = list(ori.getdata())

puz = Image.open('puzzle.jpg', 'r')
oripuz = list(puz.getdata())

im = Image.new(mode="RGB", size=(width, height), )

for i in range(width * height):
    x = i % width
    y = i // width
    if (oripxl[i] != oripuz[i]):
        a = ((oripuz[i][0] - oripxl[i][0]), (oripuz[i][1] - oripxl[i][1]), (oripuz[i][2] - oripxl[i][2]))
        if (max(a) < 0):
            im.putpixel((x, y), (0,0,0))
        else:
            im.putpixel((x, y), (0,0,256))
    else:
        im.putpixel((x, y), (256,256,256))

im.save("out.jpg")
```

This is what I get in the end

!["puzzle.jpeg"](/assets/images/wargames-my-2022/puzzle.jpeg)

Rearranging the pieces with photoshop (manual labour),

!["puzzle2.jpeg"](/assets/images/wargames-my-2022/puzzle2.jpeg)

Tada! I got flag by scanning the QR

## Boot2root

### Sanity Check (TryHackMe)


> Please use the link below to learn how to use TryHackMe platform. Submit the root flag located in /root/root.txt

#### Solution

Trivial, just cat the file for the flag

### D00raemon (User)

> User flag located in /home//user.txt

#### Solution

First I did a port scan of the provided IP, 2 ports are open port 22 (ssh) and port 80 (http)

Port 80 was nothing but just a default page for apache.

![d00raemon1.png](/assets/images/wargames-my-2022/d00raemon1.png)

I then did a directory brute force on the IP and found that `/wordpress` was online.

Continue my directory brute force on `/wordpress`, I found `/wordpress/wp-content/uploads/`

![d00raemon2.png](/assets/images/wargames-my-2022/d00raemon2.png)

Moving further in, `/wordpress/wp-content/uploads/2022/12/notes.txt` was a random string. 

```
PvWu&q563b3cwctZjL
```

I guess that it was the ssh password for the user and I was right!

ssh to the user account and cat the file for the flag.

![d00raemon3.png](/assets/images/wargames-my-2022/d00raemon3.png)


### D00raemon (Root)

> Root flag located in /root/root.txt

#### Solution

Now that I got ssh access to the user account, I want to escalate our privilege to read `/root/root.txt`

First, let's check what command this user can run with `sudo -l`

![d00raemon4.png](/assets/images/wargames-my-2022/d00raemon4.png)

This is interesting, I can use the command 

```
/usr/bin/csvtool trim t * --help
```
as any user without a password

csvtool is a powerful command line tool that can read/write to a file

I can add any character I want to the command because of the `*` wildcard, just that the prefix and suffix must be the same.

Here, the organizers have hinted a lot that we need to bypass the `--help` flag

So to do this, I just add `-o` flag just before `--help` flag. Then `--help` will then be regarded as the output file.

Final command

```
sudo -u root /usr/bin/csvtool trim t /root/root.txt -o --help
cat -- --help
```

![d00raemon5.png](/assets/images/wargames-my-2022/d00raemon5.png)

## Cryptography

### HMAC master

<button class="collapsible btn" id="hmac">server.py</button>

<div class="content" id="hmacdata" style="display:none" markdown="1">

> Prove you're a HMAC master by Connect with nc 54.255.181.88 2000

```python
#!/usr/bin/env python3
import hashlib
import os
from secret import flag

try:
    print("Welcome to the HMAC master challenge!")
    print("Answer all 3 questions to get the flag!\n")

    print("Now give me two different messages that have the same MD5 hash!")
    a = input("Message A: ")
    b = input("Message B: ")
    if a == b:
        print("Both message cannot be the same! Aborting..")
        exit()
    elif hashlib.md5(bytes.fromhex(a)).hexdigest() != hashlib.md5(bytes.fromhex(b)).hexdigest():
        print("Oops.. See you next time!")
        exit()
    print("You solved the first question!\n")

    print("Now the second question")
    print("Give me a message, then I will give the MD5 hash after I preappend a secret key")
    print("Next you need to give me another message and the corresponding hash after preappended the secret key")
    KEY = os.urandom(8)
    a = input("Message A: ")
    h = hashlib.md5(KEY+bytes.fromhex(a)).hexdigest()
    print(f"MD5(KEY+A): {h}")
    b = input("Message B: ")
    h2 = hashlib.md5(KEY+bytes.fromhex(b)).hexdigest()
    if a == b: 
        print("Both messages cannot be the same! Aborting..")
        exit()
    elif input("MD5(KEY+B): ") != h2:
        print("Oops.. See you next time!")
        exit()
    print("Good Job!\n")

    print("Now the final question, is the another way around")
    print("Give me a message, then I will give the MD5 hash after I append a secret key behind")
    print("Next you need to give me another message and the corresponding hash after appended the secret key")

    KEY = os.urandom(8)
    a = input("Message A: ")
    h = hashlib.md5(bytes.fromhex(a)+KEY).hexdigest()
    print(f"MD5(A+KEY): {h}")
    b = input("Message B: ")
    h2 = hashlib.md5(bytes.fromhex(b)+KEY).hexdigest()
    if a == b:
        print("Both message cannot be the same! Aborting..")
        exit()
    elif input("MD5(B+KEY): ") != h2:
        print("Oops.. See you next time!")
        exit()
    print(f"Congrats HMAC master!! Here is your flag: {flag}")
except Exception as e:
    print("HACKER ALERT!! Aborting..")
```

</div>

#### Observation

For this HMAC question, we have 3 parts to solve.

1. Provide 2 different messages with the same signature
2. Given a signature of a chosen message prepended with a secret, provide a different message and the signature when is prepended with the same secret
3. Given a signature of a chosen message appended with a secret, provide a different message and the signature when is appended with the same secret

#### Solution

1. MD5 is known as a weak hash because collisions have been found for the hash algorithm. To solve this question, go to this [website](https://www.mathstat.dal.ca/~selinger/md5collision/) and take the 2 messages
2. MD5 is vulnerable to length extension attack because it is an [Merkle–Damgård hash function](https://en.wikipedia.org/wiki/Merkle%E2%80%93Damg%C3%A5rd_construction). To solve this question, use [hash_extender](https://github.com/iagox86/hash_extender) to construct the second message and the signature
3. This is more interesting compared to the previous 2.
   
Again, since MD5 is an [Merkle–Damgård hash function](https://en.wikipedia.org/wiki/Merkle%E2%80%93Damg%C3%A5rd_construction), the hash is basically done block by block. (similar to AES-CBC)

Each block is of a size of 512 bits. If the message is less than the length, it will then be padded to a certain length before hashing. 

So to solve this question, we can first take the 2 messages we got from the first question, and add padding to both of them. (same as how would md5 algorithm do it)

The padding looks something like this

```
08000....004000000000000
```

Now, these 2 messages will be 512 bits, which fits in as the first block. 

From part 1, we know the hash in the first block of both messages is the same. Since the second block has the same message (the appended key), the final hash will be equal

Final input for part 3
```
m1 = d131dd02c5e6eec4693d9a0698aff95c2fcab58712467eab4004583eb8fb7f8955ad340609f4b30283e488832571415a085125e8f7cdc99fd91dbdf280373c5bd8823e3156348f5bae6dacd436c919c6dd53e2b487da03fd02396306d248cda0e99f33420f577ee8ce54b67080a80d1ec69821bcb6a8839396f9652b6ff72a7080000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000004000000000000
m2 = d131dd02c5e6eec4693d9a0698aff95c2fcab50712467eab4004583eb8fb7f8955ad340609f4b30283e4888325f1415a085125e8f7cdc99fd91dbd7280373c5bd8823e3156348f5bae6dacd436c919c6dd53e23487da03fd02396306d248cda0e99f33420f577ee8ce54b67080280d1ec69821bcb6a8839396f965ab6ff72a7080000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000004000000000000
```

### E-Signature

> I made a E-Signature service! Try it now by connect with nc 54.255.181.88 3000

<button class="collapsible btn" id="esign">server.py</button>

<div class="content" id="esigndata" style="display:none" markdown="1">

```python
#!/usr/bin/env python3
from Crypto.Util.number import *
from secret import flag
p = getPrime(1024)
q = getPrime(1024)
e = 0x10001
n = p*q
d = pow(e,-1,(p-1)*(q-1))
msg = b"gimmetheflag"

print("Welcome to our E-Signature Service!")
while True:
    print("Choose an option")
    print("1. Sign message")
    print("2. Get flag")
    print("3. Show Public Key")
    print("4. Exit")
    option = input("Enter option: ")
    try:
        if option == '1':
            m = int(input("Enter message to sign: "),16)
            if bytes_to_long(msg) == m:
                print("Nope hacker hehe")
                break
            s = pow(m,d,n)
            print(f"Signature: {hex(s)[2:]}\n")
        elif option == '2':
            s = int(input("Give me the signature of the message 'gimmetheflag': "),16)
            if bytes_to_long(msg) == pow(s,e,n):
                print(f"Signature matches! Here's your flag: {flag}\n")
            else:
                print("Wrong signature!!\n")
        elif option == '3':
            print(f"n={n}")
            print(f"e={e}\n")
        elif option == '4':
            break
        else:
            print("Invalid option!\n")
    except:
        print("Nope hacker hehe")
        break
```
</div>

#### Observation

The goal of this challenge is to obtain the signature of `gimmetheflag`

Here we are given a signature oracle that will sign any message user provides as long as it is not the target message.

#### Solution

Let $$m$$ be the target message

Goal : Find $$m^d \equiv mod(n)$$

To solve this question,

1. Send $$2 \cdot m \mod(n)$$ to the oracle to obtain $$(2m)^d \mod(n)$$
2. Send $$2^{-1} \mod(n)$$ to the oracle to obtain $$2^{-d} \mod(n)$$
3. Multiply both answers

$$(2m)^d \cdot 2^{-d} \equiv m^d \mod(n)$$

Send the answer to the server to get the flag

Solution script

```python
from pwn import *
from Crypto.Util.number import bytes_to_long, long_to_bytes

msg = b"gimmetheflag"

r = remote("54.255.181.88", 3000)

r.sendline("3")
r.recvuntil("n=")
n = int(r.recvline().decode())

r.recvuntil("e=")
e = int(r.recvline().decode())

r.sendline("1")
r.sendline(long_to_bytes(bytes_to_long(msg) * 2).hex())
r.recvuntil("Signature: ")

m2 = int(r.recvline(), 16)

r.sendline("1")
r.sendline(hex(pow(2, -1, n)))
r.recvuntil("Signature: ")

s2 = int(r.recvline(), 16)

r.sendline("2")
r.sendline(long_to_bytes((m2 * s2) % n).hex())

r.interactive()
```

### Corrupted

> My private key was corrupted, luckily still got half is not corrupted Can you help me to recover the private key? I got an important file on my server
> Login my server using godam@178.128.106.114 port 2222

```
-----BEGIN RSA PRIVATE KEY-----
MIICXAIBAAKBgQC0hAS5rQUKGw6oaJ4+PUDJrq537SAtuINquhoZu17GeLJhMPxR
vVfYoy7SVqetqgE0ZCpxyz+DOh7fX0eLVJByoMDB4ljV4ipjP4tN+pCMOt1ZTi2x
mgzV1fnlU7cYF+s9C1SazDPAdzdVRQGxMGsKX5Y9nWqLe37Uju6x2bOOHwIDAQAB
????????????????????????????????????????????????????????????????
????????????????????????????????????????????????????????????????
????????????????????????????????????????????????????????????????
???????????????????????????????Xe3iK5RisoeJtgdOXHp0+6oC+zbbyzpS4
P6z0852lAkEA5zyeIqW0dBjZW/fRP3+ZhZ6BojWU40DCQygcZXk2vcGB????????
????????????????????????????????????????????????????????????????
????????????????????????????????????????????????????????????????
????????????????????????????????????????????????????????????????
????????????????????????????????????????????????????????????????
????????????????????????????????????????????
-----END RSA PRIVATE KEY-----
```

#### Observation

We are given a partial (PKCS#1) private RSA key in ASN.1 PEM.

Our goal is to obtain the $$p$$ and $$q$$ to connect the server

#### Solution

Previously I have been using the ASN.1 convertor as a black box, but for this question, I will have to deep dive into the structure of ASN.1.

In short, it uses the [Type–length–value](https://en.wikipedia.org/wiki/Type%E2%80%93length%E2%80%93value) encoding scheme.

The first part of the data will be the type, followed by length and value.

You can learn more about it [here](https://coolaj86.com/articles/asn1-for-dummies/)

With this in mind, we can then look at the PKCS#1 structure

```
RSAPrivateKey ::= SEQUENCE {
  version   Version,
  modulus   INTEGER,  -- n
  publicExponentINTEGER,  -- e
  privateExponent   INTEGER,  -- d
  prime1INTEGER,  -- p
  prime2INTEGER,  -- q
  exponent1 INTEGER,  -- d mod (p-1)
  exponent2 INTEGER,  -- d mod (q-1)
  coefficient   INTEGER,  -- (inverse of q) mod p
  otherPrimeInfos   OtherPrimeInfos OPTIONAL
}
```

The first 2 integer should be $$n$$ and $$e$$

By converting the first known part of the key from base64 to hex, 

![corrupted.png](../../assets/images/wargames-my-2022/corrupted.png)

Here we filter for the byte `0x02` because that stands for the type of integer.

Observe the part with `02 81 81 01`

This is the starting of the modulus $$n$$. The length of $$n$$ is 0x81 bytes which is 1032 bits. This means that the $$n$$ we have is most likely 1024 bit (the first hex byte of n is 01)

There is also a part that shows `02 03 01 00 01`

This correspond to our exponent $$e$$, which is `0x100001`

So, this first part of the key has leaked the public modulus and exponent to us. 

Now let's take a look at the second part.

Note: Before we convert from base64 to hex, we must make sure that the 

starting character position $$\times\ 6 \mod 8 = 0$$

This is because each base64 character corresponds to 6 bits, the formulation is to make sure that the hex is correctly aligned

So we start from `e3iK5R...` instead of `Xe3iK5R...`

![corrupted2.png](../../assets/images/wargames-my-2022/corrupted2.png)

Observe part `02 41 00`. 

We don't know whether the value before this was a private exponent or another prime, but we can be certain that the part after this will be one of the primes. This is because the byte length is 0x41, which is `512` bit :)

So now we know the higher bits of a prime `p`, we know the modulus `n` and the exponent `e`. 

We can then use the classical coppersmith method to recover the prime!

Sage Script to find `p`

```python
from Crypto.Util.number import bytes_to_long

n = [0xb4,0x84,0x04,0xb9,0xad,0x05,0x0a,0x1b,0x0e,0xa8,0x68,0x9e,0x3e,0x3d,0x40,0xc9,0xae,0xae,0x77,0xed,0x20,0x2d,0xb8,0x83,0x6a,0xba,0x1a,0x19,0xbb,0x5e,0xc6,0x78,0xb2,0x61,0x30,0xfc,0x51,0xbd,0x57,0xd8,0xa3,0x2e,0xd2,0x56,0xa7,0xad,0xaa,0x01,0x34,0x64,0x2a,0x71,0xcb,0x3f,0x83,0x3a,0x1e,0xdf,0x5f,0x47,0x8b,0x54,0x90,0x72,0xa0,0xc0,0xc1,0xe2,0x58,0xd5,0xe2,0x2a,0x63,0x3f,0x8b,0x4d,0xfa,0x90,0x8c,0x3a,0xdd,0x59,0x4e,0x2d,0xb1,0x9a,0x0c,0xd5,0xd5,0xf9,0xe5,0x53,0xb7,0x18,0x17,0xeb,0x3d,0x0b,0x54,0x9a,0xcc,0x33,0xc0,0x77,0x37,0x55,0x45,0x01,0xb1,0x30,0x6b,0x0a,0x5f,0x96,0x3d,0x9d,0x6a,0x8b,0x7b,0x7e,0xd4,0x8e,0xee,0xb1,0xd9,0xb3,0x8e,0x1f]
n = bytes_to_long(bytes(n))

ph = [0xe7,0x3c,0x9e,0x22,0xa5,0xb4,0x74,0x18,0xd9,0x5b,0xf7,0xd1,0x3f,0x7f,0x99,0x85,0x9e,0x81,0xa2,0x35,0x94,0xe3,0x40,0xc2,0x43,0x28,0x1c,0x65,0x79,0x36,0xbd,0xc1,0x81]
ph = bytes_to_long(bytes(ph))

F.<x> = Zmod(n)[]

f = ph * (2^248) + x

p = ph*(2^248) + f.small_roots(X = 2^248, epsilon = 0.015, beta=0.49)[0]

assert n % int(p) == 0
q = n // int(p)

print(p, q)
```

The part to generate a private key and ssh to the target is easy after finding the primes.

Connect to the server and cat the flag!

