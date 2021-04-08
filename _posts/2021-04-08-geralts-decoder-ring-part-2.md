---
title: "Geralt's Decoder Ring, Part 2: The Two Encoders"
date: 2021-04-08
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - ctf
  - portfolio
  - programming
---

**Note:** This is the second part of the write-up on how to solve the challenge *Geralt's Decoder Ring* from the ISSessions 2021 CTF. It assumes you read the [first part](/blog/geralts-decoder-ring-part-1) or got as far as I or anyone else I spoke to who tried this challenge did before the end of the CTF and rebuilt built the decoder function before getting totally stumped when trying to decode the flag.
{: .notice--info}

Alright, so we've got our decoder. We can even test it in the html page: if we type in a message and encode it, decoding the ciphertext gives us the original message back. Everything works. The flag is ours.

So what happens when we try and decode the `flag.txt` ciphertext?

![geralt flag decoded chinese](/assets/images/geralt-page-chinese.png "geralt-page-chinese")

...

This isn't like `doTheShuffle()`, I understand this code forwards and backwards - I had to in order to get this far. There's no room for tricks in the code we were given for the encoder. The decoder I've built works for any other message I throw at it.

I'm stumped.

Since the beginning, I've been logging the values for `f` - the heart counts - while playing around to get the right equations for `i`, `d` and `e`. So what does the array of `f` values look like for the flag ciphertext?

`a = [1481, 1611, 2818, 2954, 4154, 4290, 5492, 5623, ... ]`

...

If you've been paying attention, you'll know these numbers make absolutely zero sense. When `d=0` and `i=0`, the formula for `f` is `e+13+137`, where `e` is a *digit*, a number between 0 and 9 and no more. That means the max value for `f` our encoder could spit out is `159`.

How in the *hell* are we getting a number like `1481`? There just shouldn't be that many hearts in the first encoded char code digit of the ciphertext.

Time to log some more values.

```
f=1481 has digit 7 (e) at index 7 (d) of char 9 (i)
f=1611 has digit 0 (e) at index 7 (d) of char 10 (i)
f=2818 has digit 0 (e) at index 5 (d) of char 19 (i)
f=2954 has digit 12 (e) at index 4 (d) of char 20 (i)
f=4154 has digit 5 (e) at index 2 (d) of char 29 (i)
f=4290 has digit 4 (e) at index 2 (d) of char 30 (i)
f=5492 has digit 12 (e) at index -1 (d) of char 39 (i)
f=5623 has digit 6 (e) at index -1 (d) of char 40 (i)
...
```

Again, not much we can make sense of here. `i` should start at `0` and be constant for the first two or three lines, as those correspond to the 2 or 3 digits of the char code for an alphanumeric character. `d` should also start at `0` and increment over the digits, but here we're getting all sorts of bizarre results. In what universe could the digit `e` be `12`!?

Compare that with the output for an input string like `"FLAG"`

```
f=157 has digit 7 (e) at index 0 (d) of char 0 (i)
f=163 has digit 0 (e) at index 1 (d) of char 0 (i) 'F'=70
f=294 has digit 7 (e) at index 0 (d) of char 1 (i)
f=306 has digit 6 (e) at index 1 (d) of char 1 (i) 'L'=76
f=430 has digit 6 (e) at index 0 (d) of char 2 (i)
f=442 has digit 5 (e) at index 1 (d) of char 2 (i) 'A'=65
f=568 has digit 7 (e) at index 0 (d) of char 3 (i)
f=575 has digit 1 (e) at index 1 (d) of char 3 (i) 'G'=71
```

This is what we *should* be getting, but it's not what we're seeing. Well... there's a hint in what I just said. We *know* what we should be getting, or at least we can be fairly confident about it. The contents of `flag.txt` should decode to match the flag pattern `FLAG\{[a-z0-9]{10}\}`.

When we compare the two outputs above, there are some patterns that emerge. Sure `i` is changing, but more than that it's *incrementing*, the same way `d` normally would. Whereas `d` is more or less constant, much like `i` used to be. And isn't that a `7` and a `0` at the start of the ciphertext? Aren't those the same as the digits in the code for `F`?

<!--more-->

Let me say something here. I put hours into rebuilding that decoder. I love it. At this point in the challenge, as far as I'm concerned, there's no way it could ever do anything wrong.

The ciphertext, however, is literally Chinese to me. It's stupid. This challenge was clearly designed by a hideous Nilfgaardian troll.

Sadly, the ciphertext doesn't care about my feelings. Someone *did* design this challenge, and unless they really screwed things up, this is exactly what should be playing out right now. So even if I know my decoder isn't broken, I also know that it's not working to decode this flag.

Wait a minute... what if this ciphertext was created using *another* encoder?

Suppose `f=1481` the value for the first digit in the array. Now let's assume we got all the values we would otherwise expect: `e=7`, `d=0`, `i=0`. What would have to change about my formula for `f` for the numbers I'm seeing to make sense?

Let's look at those strange values again, how are we actually calculating them in our decoder?

```js
i = Math.floor(f[j] / 137) - 1; // index of the char in the msg
d = Math.floor(f[j] % 137 / 13) - 1; // index of the digit in the char code
```

We've been thinking about `i` and `d` in terms of their interpretation by the old encoder, what I've indicated in the comments above. But what `i` actually represents is one less than the number of times `f` can be divided by `137`. And `d` is one less than the number of times that the remainder of `f` after that division can be divided by `13`.

Assuming our current formula for decoding the digit `e` is totally flawed, then perhaps what we're really seeing is more along the lines of

```
f=1481 has term 7 (e) plus 13 x 8 (d?) plus 137 x 10 (i?)
f=1611 has term 0 (e) plus 13 x 8 (d?) plus 137 x 11 (i?)
f=2818 has term 7 (e) plus 13 x 6 (d?) plus 137 x 20 (i?)
f=2954 has term 6 (e) plus 13 x 5 (d?) plus 137 x 21 (i?)
f=4154 has term 6 (e) plus 13 x 3 (d?) plus 137 x 30 (i?)
f=4290 has term 5 (e) plus 13 x 3 (d?) plus 137 x 31 (i?)
...
```

I'm still not confident about the so-called `d` term here. But there's a distinct pattern to the multiples of `137`. At each step, we're multiplying `137` by `10*(i+1)+d`, where `i` and `d` start from `0` and track the same indices they did in the old encoder.

Let's do a little more math. Assuming the ciphertext matches the flag format

```
char:  F   L   A   G    {         }
code: 70, 76, 65, 71, 123, ..., 125

  f     e       ?        (i+1)+d
 1481 = 7 +    104     +  137*10,  i=0, d=0
 1611 = 0 +    104     +  137*11,  i=0, d=1

 2818 = 7 +     71     +  137*20,  i=1, d=0
 2954 = 6 +     71     +  137*21,  i=1, d=1

 4154 = 6 +     38     +  137*30,  i=2, d=0
 4290 = 5 +     38     +  137*31,  i=2, d=1

 5492 = 7 +      5     +  137*40,  i=3, d=0
 5623 = 1 +      5     +  137*41,  i=3, d=1

 6823 = 1 +    -28     +  137*50,  i=4, d=0
 6961 = 2 +    -28     +  137*51,  i=4, d=1
 7099 = 3 +    -28     +  137*52,  i=4, d=2
 ...
21530 = 1 +   -391     +  137*160, i=15, d=0
21668 = 2 +   -391     +  137*161, i=15, d=1
21808 = 5 +   -391     +  137*162, i=15, d=1
```

Wow, okay, that's definitely something. Even moreso because we quickly discover that `104 ≡ 71 ≡ 38 ≡ 5 ≡ -28 ≡ 5 % 33` where we're subtracting one factor of `33` at every `i`-step.

Here's what I got as a formula for `f`

```js
f = e + 5+33*(3-i) + 137*(10*(i+1)+d)
```

Notice that, when `i=1`, `(71+7)/13 = 78/13 = 6`, which is why we were getting `e=0` according to the old way of decoding the ciphertext.

This looks like exactly the kind of bizarre formula someone would cook up for the explicit purpose of creating just this sort of bewildering result when someone tries to plug the `flag.txt` heart count numbers into the decoder equations they've built based on the function for the encoder that was provided in the challenge :P

Now that we have this formula everything else is just getting the equations for the other variables right in a new decoder

```js
// replace previous assignments for i, d, and e with these ones
i = Math.round(f[j] / 137 / 10) - 1;
d = Math.floor((f[j]+33*(i-3)) / 137) - 10*(i + 1);

e = (f[j] - (5 + 33*(3-i))) % 137;
```

With all that in place, running the decoder against the contents of `flag.txt` gives us the flag.

![geralt challenge flag](/assets/images/geralt-page-flag.png "geralt-page-flag")

This was a *very* frustrating challenge involving some Sherlock Holmes-level inferences about the existence of a second magical encoder. It went unsolved during the ISSessions 2021 CTF and no one I spoke to after the event got any further than I did - wondering what the hell to do with the results we got from running the flag ciphertext through the first rebuilt decoder.

If anything, I hope that by having solved the mystery of Geralt's *other* decoder ring, I'll spare anyone else from ever having to deal with this challenge in a future ISSessions CTF :X

To that end, I'd like to thank the entire ISSessions 2021 CTF event team for their hard work, ingenuity, and sheer diabolical wickedness in designing this year's challenges. As a first-timer, I couldn't have asked for a better CTF :D

And thank *you* for reading!

[GitHub](https://github.com/nichejambinn/geralts-other-decoder-ring) repo with the original challenge files and my solutions