---
title: "Unlocking Cars With a Radio: My SDR Keyless-Entry Research"
date: 2026-10-08 14:30:00 +0530
description: "How I used a HackRF and GNU Radio to capture and replay car key-fob signals, which cars fell to it, and the simple fix every car should already have."
---

![A car at night under neon light, its key-fob signal being intercepted as glowing radio waves](/assets/img/car-hacking-hero.jpg)

Back in 2018-19 I spent a stretch of nights in a parking lot with a laptop, a little orange radio board, and a car key. By the end of it I could unlock cars I had permission to test without ever touching their keys. I wrote it up as a paper, [*Cyber Attacks on Smart Cars using SDR*](https://irjet.net/archives/V6/i12/IRJET-V6I1219.pdf), published in IRJET (Volume 6, Issue 12, Dec 2019). This post is the human version of that paper, plus the hands-on work that lives in my [Car-Hacking repo](https://github.com/Yashkenia/Car-Hacking).

A quick, important note before anything else: this was controlled research on vehicles I was allowed to test, done to show a real weakness so it can be fixed. Don't do this to cars you don't own. It's illegal, and that's not the point.

## How keyless entry actually works

The old way to unlock a car was boring and safe: a metal key in a metal lock. Keyless fobs traded that for convenience. Press the button and the fob transmits a short burst of radio at a specific frequency. If the car hears a burst it recognizes, the doors unlock.

In my testing that frequency sat around **433.9 MHz**, a common band for key fobs. And here's the catch that makes the whole attack possible: on the cars I looked at, the fob sent essentially the *same* code every time. If the code never changes, then anything that can record radio and play it back can impersonate the key.

## The gear: SDR, GNU Radio, and a HackRF

Software-Defined Radio (SDR) is the idea that a radio can be defined in software instead of fixed hardware. Instead of a circuit built to do one thing, you get a general-purpose radio and write the logic.

I used two pieces:

- **[GNU Radio Companion](https://www.gnuradio.org/)**, a visual toolkit where you wire up "blocks" into a flowgraph to process radio signals.
- **A HackRF** from Great Scott Gadgets, the actual radio hardware that listens and transmits, connected to the laptop over USB.

Together they let me treat a car key's signal like any other data: capture it, look at it, and send it back.

## Step one: capturing the key

The capture flowgraph was simple. The important blocks:

```text
Osmocom Source   ->  switches the HackRF into receive mode over USB
  center freq:   ~433.9 MHz   (the key fob band)
  sample rate:   2 MHz
QT GUI Waterfall Sink  ->  live visual of the spectrum
File Sink        ->  writes the captured signal to disk
```

The **Osmocom Source** is the abstraction layer that talks to the HackRF and tells it to start receiving. The **Waterfall Sink** draws the spectrum in real time, so when I pressed the fob button, a bright peak jumped up at the key's frequency. That peak *is* the key's transmission. The file sink saved the raw signal so I could replay it later.

## Step two: replaying it at the car

Replay is capture in reverse. The flowgraph reads the saved signal back from the file and pushes it out through the HackRF, with a **Throttle** block to emit it at a steady, repeated rate so it cleanly matches the frequency the car expects.

```text
File Source  ->  Throttle  ->  Osmocom Sink (HackRF, transmit)
```

Point the radio at the car, run the graph, and the recorded "unlock" plays back on the air. The car hears a code it trusts and opens. That's a **replay attack**: no cloning, no cracking, just record and re-send. The same approach extends to **man-in-the-middle** (sitting between fob and car) and, with enough noise on the band, **denial of service** by jamming the fob so the real key stops working.

## Which cars fell to it

The formal results in the paper were on three cars, with the Indian Honda City i-Vtec as the main test subject, plus the Toyota Innova Crysta and the Maruti Wagon R. On all three, capture-and-replay worked.

I also ran the same attack against a **2014 Hyundai Xcent**. The capture and replay flowgraphs for it (`xcentcap.grc` and `xcentreplay.grc`) are in the [repo](https://github.com/Yashkenia/Car-Hacking) alongside the Honda and Wagon R graphs. Same method, same result.

## Why it worked, and the fix

None of this is clever cryptography on my end. It worked because the fobs used **fixed codes**. A code that never changes can always be replayed.

The fix has existed for decades, and good cars already use it:

- **Rolling codes (hopping codes).** The fob and car share a cryptographically secure sequence. Every button press sends the *next* code, and the car accepts only codes ahead of the last one it saw (typically within a window of about 256, in case a few presses happen out of range). A replayed old code is simply rejected.
- **KeeLoq.** A classic code-hopping cipher: it encrypts a 32-bit block to produce a 32-bit hopping code (with a 32-bit initialization vector XORed in), combined with a fixed serial-number portion. Every press produces a different transmission, so a recording is useless a moment later.

The threat model is worth stating plainly. An attacker who owns the fob's radio link can unlock, steal, track, lock out a key, try to brute-force or clone the fob, or jam its signal. Rolling codes shut down the easy end of that list. My conclusion in the paper was blunt: *every car should at least implement a rolling-code mechanism.* In 2019 some still didn't.

## Takeaways

The lesson that stuck with me: convenience features quietly become attack surface. A key fob feels like a small thing, but it's a radio protocol, and radio protocols need the same rigor as anything else. A fixed code is a password you broadcast to everyone nearby.

If you want the formal details, read the [full paper](https://irjet.net/archives/V6/i12/IRJET-V6I1219.pdf). If you want the actual GNU Radio flowgraphs and code, they're in the [Car-Hacking repo](https://github.com/Yashkenia/Car-Hacking). Questions or corrections, find me on [X](https://x.com/VAporXdc).
