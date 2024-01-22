---
title: "Potty Training"
date: 2022-11-10 00:00:00 +0800
categories: [Walkthrough, Snyk]
tags: [Fetch The Flag]
---

## Details
- Platform: [Snyk](/categories/snyk/)
- Event: [Fetch The Flag](/tags/fetch-the-flag/)

## Walkthrough
For this challenge, the only resource you have is a PNG image, along with a title and some text for possible hints.
![challenge hint](/assets/img/Pasted image 20221111153425.png)

Lets download the image and take a look.

![steganography image](/assets/img/potty.png)
_The single resource file - An image of a dog_

It just looks like a standard PNG file. No apparent clues in the image.

If there is only an image, then the image file itself contains the flag, or contains hidden files that will lead to the flag.

The title doesn't seem to be a hint, but the subtitle says it all.

Firstly, stegano is referring to steganography, which is the art of hiding secret messages in non-secret files or messages.

There is actually a python library called exactly "[stegano](https://stegano.readthedocs.io/en/latest/)"!
So that seems like a good place to start.

Let's download the image and make a python script to reveal any hidden message.

```python
import requests

from stegano import lsb

message = lsb.reveal("./potty.png")

print(message)
```

And we got something! ↓

```bash
▶ python3 main.py

        import requests
        r = requests.get('http://potty-training.c.ctf-snyk.io/')
        print(r.text)
```

It appears to be some python code that is simply making a request to some endpoint and printing out the text from the response.

Upon running the code, or just going to the URL in the browser, you get some html with the flag inside.

```html
<html>
  <head>
    <title>Good puppy!</title>
  </head>
  <body>
    <h1>Good puppy! 🐕</h1>
    <p>Here's your flag 🦴</p>
    <p>SNYK{the_flag_was_here}</p>
    <a href="https://snyk.io/blog/snyk-international-dog-day-recap/">Snyk + Dogs = 💜</a>
  </body>
</html>
```