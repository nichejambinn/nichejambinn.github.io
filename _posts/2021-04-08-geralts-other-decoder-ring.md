---
title: "Geralt's Other Decoder Ring"
date: 2021-04-08
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - ctf
  - portfolio
  - programming
---

(This is the second part of the full challenge write-up. It assumes you got as far as I or anyone else I spoke to who tried this challenge did before the end of the CTF, having built the decoder function based on the code for the encoder that was provided in the page source - hope that's okay!)

Alright, so we've got our decoder. We can even test it in the html page: if we type in a message and encode it, decoding the ciphertext gives us the original message back. Everything works. The flag is ours.

So what happens when we try and decode the `flag.txt` ciphertext?

![geralt flag decoded chinese](/assets/images/geralt-page-chinese.png "geralt-page-chinese")

...

This isn't like `doTheShuffle()`, I understand this code forwards and backwards - I had to in order to get this far. There's no room for tricks in the code we were given for the encoder. The decoder I've built works for any other message I throw at it. I'm stumped.

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

Let me say something here. I put hours into rebuilding that decoder. I love it. At this point in the challenge, as far as I'm concerned, there's no way it could ever do anything wrong. The ciphertext, however, is literally Chinese to me. It's stupid. This challenge was clearly designed by a hideous Nilfgaardian troll.

Sadly, the ciphertext doesn't care about my feelings. Someone *did* design this challenge, and unless they really screwed things up, this is exactly what should be playing out right now. So even if I know my decoder isn't broken, it's not working to decode this flag.

Wait a minute... what if this ciphertext was written using *another* encoder?

Suppose `f=1481` the value for the first digit in the array. Now let's assume we got all the values we would otherwise expect: `e=7`, `d=0`, `i=0`. What would have to change about my formula for `f` for the numbers I'm seeing to make sense?

Let's look at those strange values again, how are we actually calculating them in our decoder?

```js
i = Math.floor(f[j] / 137) - 1; // index of the char in the msg
d = Math.floor(f[j] % 137 / 13) - 1; // index of the digit in the char code
```

We've been thinking about `i` and `d` in terms of their interpretation by the old encoder, like I've added in the comments above. But what `i` actually represents is one less than the number of times `f` can be divided by `137`. And `d` is one less than the number of times that the remainder of `f` after that division can be divided by `13`.

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

Wow, okay, that's something. Even more because we soon discover that `104 ≡ 71 ≡ 38 ≡ 5 ≡ -28 ≡ 5 % 33` where we subtract one factor of `33` at every `i`-step.

Here's what I got as a formula for `f`

```js
f = e + 5+33*(3-i) + 137*(10*(i+1)+d)
```

Notice when `i=1`, `(71+7)/13 = 78/13 = 6`, which is why we were getting `e=0` according to the old way of decoding the ciphertext.

Looks like the kind of bizarre formula someone would cook up for the explicit purpose of creating just this sort of bewildering result when you try and plug the `flag.txt` heart count numbers into the decoder equations you've built based on the provided function for the encoder :P

Now that we have this everything else is just getting the equations for the variables right in our new decoder

```js
i = Math.round(f[j] / 137 / 10) - 1;
d = Math.floor((f[j]+33*(i-3)) / 137) - 10*(i + 1);

e = (f[j] - (5 + 33*(3-i))) % 137;
```

With these in place, running the decoder against the contents of `flag.txt` gives us the flag.

![geralt challenge flag](/assets/images/geralt-page-flag.png "geralt-page-flag")

This was a *very* frustrating challenge involving some Sherlock Holmes-level inferences about the existence of a second magical encoder. It went unsolved during the ISSessions 2021 CTF and no one I spoke to after the event got any further than I did - wondering what the hell to do with the results of running the flag ciphertext through the first rebuilt decoder.

If anything, I hope that by having solved the mystery of Geralt's *other* decoder ring, I've spared anyone else from having to deal with this challenge in a future ISSessions CTF.

To that end, I'd like to thank the entire ISSessions 2021 CTF event team for their hard work, ingenuity, and sheer diabolical wickedness in designing this year's challenges. As a first-timer, I couldn't have asked for a better CTF :)

And thank you for reading!



(This is the start of the full challenge write-up. I just wanted to make sure I finished the last part regarding how to solve the challenge before the write-up contest deadline. Mostly to rub it in Louai's face >:D Big thanks to Kurt whose portfolio site workshop made this page a reality, and to Yusef for giving me the link to the recording)

![geralt challenge page](/assets/images/geralt-page-blank.png "geralt-page-blank")

What could possibly go wrong?



<!--more-->

Let's see what this thing does. Typing in "gimme the flag!" (because why not?) and clicking *Encode* we get something that looks like 

>♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♣♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♣♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♣♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♣♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♣♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♥♣

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

We start popping elements from the back of `a` and appending to a string `r` the character ♥ (`\u2665`) repeated `a` times, followed by a ♣ (`\u2663`).

That explains our *ridiculous* ciphertext.

<!--more-->

Time to start working our back to undo this damage and rebuild the controller. Fortunately, the first part is easy. Each `f` value in the encoder function is just a count of the number of heart symbols that appear before a club in our ciphertext.

```js
function applyMagicalDecoder(input) {
    HEART = 9828 // ♥ 9829 \u2665
    CLUB = 9827 // ♣ 9827 \u2663

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


```js

    // unshuffle
    a.sort(function(n, m) { return n - m; }); // formula for f orders them
      

    // decode from the formula for f
    let char_codes = []; // b
    let char_digits = []; // c
    let i = 0;
    let d = 0;

    for (let j = 0; j < f_arr.length; j++) {

        i = Math.floor(a[j] / 137) - 1; // index of the char in the msg

        d = Math.floor(a[j] % 137 / 13) - 1; // index of the digit in the char code
        
        if ((d == 0 && char_digits.length > 0)) {
            char_codes.push(char_digits.join(''));
            char_digits = [];
        }

        let e = f_arr[j] % 137 % 13; // char digit
        
        char_digits.push(e);
        
        //console.log(`${f_arr[j]} with digit ${e} at index ${d} of char ${i}`);

        // don't miss the last one!
        if (j == f_arr.length - 1) {
            char_codes.push(char_digits.join(''));
        }
    }
    
    decoded_str = "";
    for (let i = 0; i < char_codes.length; i++) {
        decoded_str += String.fromCharCode(char_codes[i]);
    }

    return decoded_str;  
}
```