+++
title = 'SHAllengers - working for the lowest SHA256 hash'
date = 2024-06-21T15:27:47-07:00
draft = false
+++

There I was, browsing Hacker News on a Tuesday afternoon, bored. All of a sudden, I chance upon [a post that catches my eye](https://news.ycombinator.com/item?id=40683564). No prizes, no end in sight, just generating hashes for fun and a silly leaderboard. Plus, it generates even less value than AI and Bitcoin. Of course I'm interested!

Seems like the challenge is to generate the lowest SHA256 hash (i.e. starts with the most zeroes). Since I've previously done a little bit of password cracking, I have some ideas how to tackle this. But first, I just want to place on the leaderboard somehow. I blast out a message to my friends who are better programmers than I and attempt to nerd-snipe them into joining me, while I run the browser-based miner.

It succeeds, and a couple friends tackle the challenge. One does what I did and asks ChatGPT to generate a program that generates the hashes. Not a bad approach, especially in the beginning when we're just trying to place on the leaderboard quickly. We both finally start generating valid hashes, and place on the leaderboard.

![a screenshot of me being in 86th place on the leaderboard](/images/posts/shallenge/86th.jpg)

One very smart friend begins writing CUDA code to generate hashes in parallel. He mentions he's getting about 5 MH/s (5 million hashes per second) on his laptop gpu. Not bad!

However, there's one more pal to reach out to. They're a mysterious entity known only as ████. We met doing password cracking together at a security conference a couple years ago, and I know they used to compete in stuff similar to this. I shoot them a message and they respond almost immediately. They're in.

████ is a wizard. By the time I reach out the next day they've already got a patch written for [hashcat](https://hashcat.net/hashcat/), a popular password cracking tool. Hashcat already supports generating sha256 hashes, and even have a hand-optimized kernel for doing so. Unfortunately, the patch doesn't work for the optimized kernel, but the pure kernel is good enough. I get it running on my desktop and we get about 140MH/s. As my grandfather would say, now we're cooking with gas!

 We decided to check the performance of the unmodified kernel against what ████ wrote. 

████'s kernel:
 ```bash
 Speed.#1.........:   139.1 MH/s (3.19ms) @ Accel:2048 Loops:1 Thr:32 Vec:1
```

hashcat's optimized kernel:
 ```bash
 Speed.#1.........: 9278.9 MH/s (73.59ms) @ Accel:256 Loops:1024 Thr:32 Vec:1
```

What could we be possibly doing so badly that the hashrate suffers **that** much? ████ dove back into the code while I played around with some different command line options. At one point I try `-S` which slows things down terribly but enough that hashcat pops out a helpful error message telling me that I probably need to try something else for a better hashrate, and links me to [this handy wiki page](https://hashcat.net/wiki/doku.php?id=frequently_asked_questions#how_to_create_more_work_for_full_speed). I try some things but we can't seem to get above 140MH/s. That is, until I find this in the hashcat repo next to the documentation for `-S`:

`hashcat/docs/performance.txt`
```
Performance Notes:
==================

- Always use the latest display driver
- To significantly improve performance, use -w 3
- Cracking with a mask with a fixed prefix can be slow.
  Instead, consider using a mode with salt as a prefix and set salt to that fixed prefix.

Also read this:

https://hashcat.net/wiki/doku.php?id=frequently_asked_questions#how_to_create_more_work_for_full_speed
```

oh! wait a second! we're cracking with a mask, and we're using a fixed prefix (the username)! ████ changes his patch to target the hashcat kernel that includes a salt.

```bash
Speed.#1.........:  6516.3 MH/s (52.28ms) @ Accel:16 Loops:1024 Thr:256 Vec:1
```

We're back in the game, and maybe we might even be able to reach the top!

## being confronted with reality

Of course this dream was short lived - other competitors were putting fixed prefixes at the beginning of their hashes on the leaderboard that showed they were using 4090s and 7900 XTXes with hashrates two or three times ours. My dreams of reaching the top are dashed for now, but just in case I get lucky, I let my 3090 run all night generating heat and noise while I'm trying to sleep. Many thanks to my beautiful wife for her patience with me this week while I've been generating hashes. I make it to 13th place overnight.

████ and I keep chatting while things run and I mention I wish i had more hardware. I briefly joke about asking my boss to let me run a GPU instance in AWS or reaching out to people I know have password cracking rigs. Then ████ asks, "what about [Vast?](https://vast.ai/)". I had heard of them before, but assumed they mostly just resold major cloud gpu instances or something. I had no idea anyone could sign up and host their GPU machines to be used. ████ mentions one of the other competitors mentioned in their comments that they were renting the 4090 they were using.

I sign up, set up a test instance, and put in $10 worth of credit, just to play around. I select an interruptible instance with 8x L40S GPUs. After all, I can just move the hashcat restorefile and easily resume work on another instance, so we can save some money this way. I run the same workload:

```
CUDA API (CUDA 12.4)
====================
* Device #1: NVIDIA L40S, 45158/45589 MB, 142MCU
* Device #2: NVIDIA L40S, 45158/45589 MB, 142MCU
* Device #3: NVIDIA L40S, 45158/45589 MB, 142MCU
* Device #4: NVIDIA L40S, 45158/45589 MB, 142MCU
* Device #5: NVIDIA L40S, 45158/45589 MB, 142MCU
* Device #6: NVIDIA L40S, 45158/45589 MB, 142MCU
* Device #7: NVIDIA L40S, 45158/45589 MB, 142MCU
* Device #8: NVIDIA L40S, 45158/45589 MB, 142MCU

Benchmark relevant options:
===========================
* --backend-devices-virtual=1
* --optimized-kernel-enable

--------------------------------------
* Hash-Mode 1420 (sha256($salt.$pass))
--------------------------------------

Speed.#1.........: 18701.1 MH/s (63.36ms) @ Accel:8 Loops:1024 Thr:1024 Vec:1
Speed.#2.........: 18611.8 MH/s (63.59ms) @ Accel:8 Loops:1024 Thr:1024 Vec:1
Speed.#3.........: 18619.6 MH/s (63.63ms) @ Accel:8 Loops:1024 Thr:1024 Vec:1
Speed.#4.........: 18950.9 MH/s (62.45ms) @ Accel:8 Loops:1024 Thr:1024 Vec:1
Speed.#5.........: 18744.9 MH/s (63.14ms) @ Accel:8 Loops:1024 Thr:1024 Vec:1
Speed.#6.........: 18465.7 MH/s (64.17ms) @ Accel:8 Loops:1024 Thr:1024 Vec:1
Speed.#7.........: 18971.0 MH/s (62.44ms) @ Accel:8 Loops:1024 Thr:1024 Vec:1
Speed.#8.........: 18776.7 MH/s (63.08ms) @ Accel:8 Loops:1024 Thr:1024 Vec:1
Speed.#*.........:   149.8 GH/s

Started: Thu Jun 20 19:01:18 2024
Stopped: Thu Jun 20 19:01:35 2024
```

Wow! and we end up getting about 120GH/s running the patched kernel. an hour and 15 minutes later, I'm in 9th place and rising quickly!

![a screenshot of me being in 9th place on the leaderboard](/images/posts/shallenge/9th.jpg)

at this point I just want to quickly apologize for changing usernames - didn't want to monopolize the leaderboard, but I just wanted to have my username line up with my hacker news username. I'm not trying to game the system, I promise!

After letting the vast instance run overnight and playing around with other GPUs and settings, I end up here:

![a screenshot of me being in 2nd place on the leaderboard](/images/posts/shallenge/2nd.png)

Even better, I'm still within my $10 budget! I'm so close to the top and I just crave that last zero, so I load vast up with more credit. I can't stop thinking about what I can do to hash faster and better utilize vast - right now i'm just manually starting the hashing jobs when the container starts, which isn't great when my instances start and stop automatically based on other people's bids for the machines I'm using.

The L40S instance is still the best bang for buck I got, and the best hashrate as well. I've also tried 10X 4070Ti Super and 8x A6000 Ada instances but nothing has compared to the performance of the L40S. Huge thanks to whoever it is in Turkey hosting these things for so cheap!

I got that far, and I'm pretty happy. Wish I could've touched first place, but oh well. what a fun challenge!

## The Battle for 100th

As an aside, ████ came up with a cool synthetic challenge i thought might be fun mentioning - trying to compute hashes not with the aim of staying in first place, but in 100th place, the last spot on the leaderboard. I wish I had the time/energy right now to do this, I think it might be fun but I'm still working on finishing up my degree. maybe if we get a re-run of the challenge or some future ctf does something similar.