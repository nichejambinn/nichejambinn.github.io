---
title: "Geralt's Decoder Ring, Part 1: The Fellowship of the Decoder Ring"
date: 2021-04-08
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - ctf
  - portfolio
  - programming
---

![geralt challenge page](/assets/images/geralt-page-blank.png "geralt-page-blank")

What could possibly go wrong?

{% capture notice-2 %}
#### *Geralt of Rivia's Magical Decoder*

* 100pts max difficulty programming challenge from the 2021 ISSessions CTF hosted and run by the ISSessions infosec club at Sheridan. Nobody, myself included, was able to solve this challenge during the CTF but a couple weeks later I came back to it and figured the problem out so I'm sharing my process
* if you want to try and solve the challenge for yourself, you can find the original challenge files as well as my solutions on [GitHub](https://github.com/nichejambinn/geralts-other-decoder-ring)
{% endcapture %}

<div class="notice">{{ notice-2 | markdownify }}</div>
<!--more-->

Let's see what this thing does. Typing in "gimme the flag!" (because why not?) and clicking *Encode* we get something that looks a little bit like 

>♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♣♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♣♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♣

And that's *truncated* - oh god, oh no.

Clicking *Decode* does nothing. So let's check the page source and see what the hell is going on.

There's a function `applyMagicalEncoder()` as well as

```js
/* Rebuild Me */
function applyMagicalDecoder(input) {
  return "";
}
```

Stupid me, lost the decoder after all. Well, challenge accepted.

So what's the encoder look like?

```js
function applyMagicalEncoder(input) {
  var a=[];
  for (var i=0; i<input.length; i++) {
    var b = input.charCodeAt(i);
    var c = ('' + b).split('');
    for (var d=0; d<c.length; d++) {
      e = parseInt(c[d]);
      f = e + 13*(d+1) + 137*(i+1);
      a.push(f);
    }    
  }

  var g = doTheShuffle(a);
  
  var r = '';
  while (a.length) {
    var t = a.pop();
    r = r + "\u2665".repeat(t) + "\u2663";
  }
  return r;
}
```

Looks like we're gonna have to figure out what this function is doing so we can reverse each step and build our own decoder.

There's only one thing to do now, and that's go through the code line by line.

Suppose we wanted to encode the string `FLAG{abcdefg123}` ;)

Starting with the for-loop, we iterate on each character in the input string. For that character, we find the char code. A quick peek at the [MDN docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/charCodeAt) tells us that 

> The `charCodeAt()` method returns an integer between `0` and `65535` representing the UTF-16 code unit at the given index.

Okay, the first character in our string was `F`, so we're getting the UTF-16 code as an integer, which in this case is just the ASCII value: so `b=70`.

Next we're splitting that value by its digits into an array `c=['7', '0']` and iterating *again* on the entries in that array. We parse the string value, getting the digit back as an integer, which gives us `e=7` on the first iteration, then we do a bit of math to get `f = 7 + 13*(0+1) + 137*(0+1) = 7 + 150 = 157` and push that value to an array `a`.

When we're done, `a` looks something like `[157, 163, 294, 306, 430, 442, 568, 575, ...]`

And then this happens

```js
var g = doTheShuffle(a);
```

Personally, this is where I start crying. `doTheShuffle()` really is shuffling the array, there's a call to `Math.random()` and it's being used to swap random elements. Looking at the code for the function, there's definitely some patterns we might be able to exploit, but there's no way we're going to be able to programmatically reverse the steps of `doTheShuffle()` for our decoder.

When I find something I really don't like in a problem, I ignore it. At least until I understand everything else well enough that I can come back and make a decision on how to deal with it.

So what happens next?

We start popping elements from the back of `a` and appending to a string `r` the character ♥ (`\u2665`) repeated `a` times, followed by a ♣ (`\u2663`). So that's 157 ♥s, then a ♣, 163 ♥s, another ♣, etc.

That explains our *ridiculous* ciphertext.

<!--more-->

Time to start working our way back to undo this damage and rebuild the decoder. Fortunately, the first part is easy. Each `f` value in the encoder function is just a count of the number of heart symbols that appear before a club in our ciphertext.

```js
function applyMagicalDecoder(input) {
    const HEART = 9828 // ♥ 9829 \u2665
    const CLUB = 9827 // ♣ 9827 \u2663

    let a = [];
    let count = 0;
    
    // each f value is the count of hearts before a club appears
    for (let i = 0; i < input.length; i++) {
        if (input[i].charCodeAt() == CLUB) {
            a.push(count);
            count = 0;
        } else {
            count++;
        }
    }
  ...
```

I'm still ignoring that shuffle problem, so what's next is to somehow get the digit `e` from the heart count value `f`. Recall the formula for `f` in the decoder is

```js
f = e + 13*(d+1) + 137*(i+1);
```

Hey now... `e` is a digit right? That means it's somewhere between 0 and 9 inclusive. But at each step of the loop, we're adding at least one group of `13` and `137` to the next digit in order to encode that value of `f`. `a[0]` is at least `4` less than `a[1]`, `a[1]` is at least `4` less than `a[2]`, and if we're dealing with alphanumeric characters only, `a[3]` is at least `102` less than `a[4]`. In fact, for *any* `n`, `a[n] > a[n-1]` by at least `4`!

They're naturally ordered! Who *cares* if `a` is shuffled, all we have to do is sort the array from lowest to highest and we've recovered the original order of the char code digits in the array!

We'll add that sort to our decoder

```js
    // unshuffle - char code digits are ordered from lowest to highest value of f
    a.sort(function(n, m) { return n - m; });
``` 

The main thing to realize now is that `i` is just the number of multiples of `137` in any value of `f` minus 1. While `d` is just the number of multiples of `13` that are left once you've taken care of `i`, again minus 1. Finally, `e` is whatever's leftover after all those multiples have been divided out. That is to say

```js
    i = Math.floor(a[j] / 137) - 1; // index of the char in the msg
    d = Math.floor(a[j] % 137 / 13) - 1; // index of the digit in the char code
    e = f[j] % 137 % 13; // char digit
```

We can use these equations and iterate over the length of the array of `f` values to join our digits together as char codes. Each time `d==0`, that means we're at the start of a new char code.

```js 
  if (d == 0 && char_digits.length > 0) {
    char_codes.push(char_digits.join(""));
    char_digits = [];
  } 
```

Pushing all those numbers into an array, we can build the decoded message string!

```js
  decoded_str = "";
  for (let i = 0; i < char_codes.length; i++) {
      decoded_str += String.fromCharCode(char_codes[i]);
  }

  return decoded_str; 
}
```

There you go! We've rebuilt the decoder!

That's as far as I got during the CTF. When I tried to decode the contents of the file containing the flag ciphertext, I hit an obstacle that stumped me for the rest of the event.

It was only until I came back to challenge after a couple of weeks that I was able to solve the mystery of Geralt's *Other* Decoder Ring. But for that, you'll have to check out [Part 2](/blogs/geralts-decoder-ring-part-2)!
