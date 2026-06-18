---
title: "ARENA 8.0 Hackathon - NLAs all the way down"
date: 2026-06-13
tags: ["AI Safety", "NLA", "Interp"]
draft: false
---

# NLAs all the way down: a verbalizer's self-readout doesn't reveal when it's making things up

*ARENA hackathon, June 2026.*

## TL;DR

A [natural language autoencoder (NLA)](https://transformer-circuits.pub/2026/nla/index.html) is a tool that translates a model's internal activations into English. We (Me and Claude), used the NLA to explain its own internal state. When the verbalizer made up a fact, we read its own activations and passed them through the same NLA to see whether the readout would give signs of it being a fabrication. It didn't. A confabulating state and a faithful one produce the same confident description, with no hedging and no sign that anything was invented. Our reading: confabulation is confident, the model isn't internally flagging doubt, so a self-readout has nothing to surface.

## What we tested

An NLA is a pair of fine-tuned models. The verbalizer (AV) maps an activation vector to a sentence ("This activation is about X"); the reconstructor (AR) maps that sentence back to a vector, which lets you score how faithfully the words captured the original. Gluing these two LLMs together (They are both the same model), one can train them using reconstruction as their main optimization pressure.  

By *confabulation*, we mean the verbalizer asserting a specific detail that the source text doesn't support, like a made-up name, date, or number.

The verbalizer is itself a model with internal states. So we take the verbalizer's own layer-20 activations (The layer selection is actually a limitation of the NLA selected) while it is verbalizing, and feed them back through the same NLA, we call this second pass NLA2. NLA2's output is a readout of the verbalizer's own internal state. The question is: when the verbalizer confabulates, does NLA2's readout of that moment look any different from when it's faithful?

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
any fact; none of them says "uncertain", "possibly fabricated", or "I'm guessing". Neither a human nor a model can tell which column a readout came from once you strip the topic words.

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

## How did we test for confabulation?

We ran an independent model judge (Gemma-4-12B) rating each readout 1-5 for how ungrounded it is. It found no difference between confabulating (3.73) and faithful (3.89) readouts.

![Distribution of the judge's 1–5 ungroundedness rating for both sets; the two are nearly identical and both pile up at 4–5.](/blog/images/fig-ceiling.png)

*Figure 2. The judge rates almost every readout 4 or 5 out of 5 for "ungroundedness", in
both sets, about 82% overall. The two distributions sit on top of each other, so the
rating has no headroom to separate the sets. NLA2's readouts are uniformly generic
("format/pattern"-like), which is why none of them carry a confabulation-specific signal.*

## What this means

The simplest reading is that confabulation is confident, or at least that the model doesn't have any trace of uncertainty when confabulating. The moment the verbalizer invents a fact, its internal state encodes "I am emitting a structured article about this topic", not "I am unsure" or "I am inventing this". There is no internal uncertainty to surface, so NLA2 just reports "structured format about X".

There are a few things we can't be sure about though, for one, a trace might exist but the readout can't express it. The verbalization is lossy and generic descriptions could be its ceiling. The second one, Qwen 2.5 7B being a small model, could maybe not contain any traces of self-awareness about not knowing while bigger models could have them, that's open to more experimentation. We are not claiming that the underlying state contains no signal, only that this readout does not surface one. 

## Building the dataset
 
The pipeline is straightforward to state. Run the target model over 150 Wikipedia articles, capture its layer-20 activations, and have the verbalizer explain each one, giving about 1,500 explanations. Then label each explanation by grounding: pull out every concrete claim it asserts (named entities, numbers, specific attributes) and check each against the full source article. An explanation is faithful if all of its asserted specifics are supported by the source, confabulated if it asserts a specific the source does not contain, and neutral otherwise.
 
"Checked against the source" sounds simple and is where all the danger lives, because every loose way of checking lets a confabulation slip through as faithful. Three leaks we had to close:
 
1. Numbers match by value, not by digits appearing somewhere. "$5 million" must not count as grounded because a stray "5" sits in some table, so numeric grounding parses the actual value (scale words, years, centuries as ranges) and matches within a tight tolerance.
2. Entities match on content words, not function words. "the Mojave Desert" (a confabulation for the Sahara) must not ground itself on "the" and "desert," so a stopword filter removes the words that match everything.
3. The tolerance stays tight enough not to tile a dense table of figures, where loose matching grounds almost any number.
Each of these, before it was fixed, leaked real confabulations into the faithful arm: the exact contamination that fakes a result.
 
A rule on its own is still a proxy, so the final labels are the intersection of the rule and an independent judge (Gemma-4-12B). The judge is not a better labeler. On its own it scores lower than the rule (64% vs 83%), because it is stingy and pushes grounded explanations into neutral. Its value is disagreement: an explanation counts as confabulated only when rule and judge agree, and faithful only when the rule finds grounded content and the judge sees no fabrication. Anything they split on goes to neutral and is dropped. That discards data, but it routes each labeler's false positives out of the two arms, which is the trade that matters: faithful-arm precision comes out near 100% on a hand-adjudicated gold sample. The gold set is small, so read that as "no contamination we could find," not a proven zero.
 
What remains is 118 confabulated/faithful pairs matched on length and topic, the set every test in the next section runs on.
 
## The signal that wasn't: topic, not confabulation
 
Could it be possible that we simply didn't look hard enough? Surely a classifier trained on the full readout text would find a signal the eye misses? It doesn't. Because confabulating and faithful readouts come from different articles, a classifier can separate them by subject matter instead of by confabulation, and that is all it does. A classifier on the pooled readouts clears our pre-registered bar at ROC-AUC 0.72 (0.5 = chance, 1.0 = perfect), but a classifier trained only to recognize the article's topic scores higher (0.77–0.84), and every test that removes the topic shortcut comes out flat: topic-agnostic style features (matched-pair p=0.53), holding the article fixed (within-article difference ≈ 0), and the judge above. 

## Lessons

- The control set is the experiment. Every near-miss false positive we caught lived in
  how we decided a readout was faithful, not in the modeling.
- A verbalizer that is fabricating doesn't
  necessarily represent itself as uncertain, so don't expect its self-report to flag it. (Whether this holds for a model confabulating during its own generation is untested, and the obvious next experiment.)
