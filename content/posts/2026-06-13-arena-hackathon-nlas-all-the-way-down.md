---
title: "ARENA Hackathon - NLAs all the way down"
date: 2026-06-13
tags: ["AI Safety", "NLA", "Interp"]
draft: false
---

# NLAs all the way down: a verbalizer's self-readout doesn't reveal when it's making things up

*ARENA hackathon, June 2026.*

## TL;DR

A [natural language autoencoder (NLA)](https://transformer-circuits.pub/2026/nla/index.html) is a tool that translates a model's internal activations into English. We (Me and Claude), used the NLA to explain it's own internal estate. When the verbalizer made up a fact, we read its own activations and passed them through the same NLA to see whether the readout would give signs of it being a fabrication. It didn't. A confabulating state and a faithful one produce the same confident description, with no hedging and no sign that anything was invented. Our reading: confabulation is confident, the model isn't internally flagging doubt, so a self-readout has nothing to surface.

## What we tested

An NLA is a pair of fine-tuned models. The verbalizer (AV) maps an activation vector to a sentence ("This activation is about X"); the reconstructor (AR) maps that sentence back to a vector, which lets you score how faithfully the words captured the original. Gluing these 2 LLMs together (They are both the same model), one can train them using reconstruction as their main optimization pressure.  

By *confabulation*, we mean the verbalizer asserting a specific detail that the source text doesn't support, like a made-up name, date, or number.

The verbalizer is itself a model with internal states. So we take the verbalizer's own layer-20 activations (The layer selection is actually a limitation of the NLA selected) while it is verbalizing, and feed them back through the same NLA, we call this second pass NLA2. NLA2's output is a readout of the verbalizer's own internal state. The question is: when the verbalizer confabulates, does NLA2's readout of that moment look any difference from when it's faithful?

To study this question we used Qwen-2.5-7B and its corresponding NLA.

## The result

![Six NLA2 readouts of the verbalizer's own state, confabulating on the left and faithful on the right, in the same confident register.](/blog/images/fig-readouts-emi3.png)

*Figure 1. What NLA2 writes when it reads the verbalizer's own state. Left: moments
where the verbalizer confabulated. Right: moments where it was faithful. The left column
narrates invented facts: "the official name for the landing was '12:05 PM'", "Plato's
father was named Theodoros", in exactly the same confident "structured-article"
register the right column uses for true ones ("the ampere … defined as one coulomb per
second"). Nothing in the register distinguishes them.*

Both columns are the same template: *"structured [wiki / article] format … establishing
… the sentence …."* The confabulating readouts narrate the made-up content as flatly as
any fact; none of them says "uncertain", "possibly fabricated", or "I'm guessing". Neither a human nor a model can tell which column a readout came from.

We can put a number on that absence of doubt. Scanning all 2,073 readouts for
fabrication-flagging words ("fabricated", "made up", "false", "invented", "unsupported"),
only about 1% contain any and the rate is the same in both sets (1.5% confabulating
vs 1.0% faithful). Generic hedges ("seems", "suggests", "likely") appear in ~90% of
readouts in both sets, but that is the verbalizer's descriptive style, not a
confabulation cue, and it never escalates to naming a fabrication.

## Were confabulating states even readable?

The first thought one may have when looking at the results is that maybe NLA2 simply couldn't read the confabulating states. But it can. The reconstructor recovers the verbalizer's own activations about equally well in both
sets (text→vector cosine ≈0.80 confabulating, ≈0.82 faithful), and the self-readout loop
holds together in the first place (transfer cosine 0.80). The states are legible.
