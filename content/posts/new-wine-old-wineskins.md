+++
title = "New Wine, Old Wineskins: AI, Labor, and the Limits of Recombination"
author = ["Rod Morison"]
date = 2026-03-30T16:55:00-07:00
tags = ["ai", "software", "philosophy", "political-economy"]
draft = false
topics = ["AI", "Software", "Philosophy", "Political economy"]
description = "From vibe coding to Gödel, from Ricardo's recantation to your power bill: what AI actually does, what it probably can't, and why the economics of that distinction are as old as the industrial loom."
+++

<img src="/ox-hugo/scopio-70b0ce2f-67c5-4993-a5fd-00fb07850a8e.jpeg" style="float:right; margin:0 0 1em 1em; max-width:400px;" alt="Wine glass with splashing red wine">

Here's a pattern I keep seeing: someone ships a working prototype using an LLM, then hits a
wall when they try to scale or productize it. The code works until it doesn't, and when it
doesn't, neither the developer nor the model quite knows why.

This is the "vibe coding" problem — building on autocomplete and instinct, skipping the
practices that keep complexity under control: clear design, incremental validation, honest
accounting of what the system actually does. Those practices exist because software systems
accumulate hidden coupling and undocumented constraint. Bypass them early and you're not avoiding the reckoning, you're
compounding interest on it. It's real, and it points to something more interesting than a
skills gap.


## Best Chains, Cleverly Completed {#best-chains-cleverly-completed}

The standard dismissal of LLMs — _it's just predicting the next token_ — is technically
accurate and philosophically incomplete at the same time. The same reductive move applies to
human cognition: neurons firing along pathways reinforced by experience. The mechanism alone
doesn't settle the question.

What I'd say with more confidence: current LLMs are extraordinarily good at navigating the
landscape of their training data — a vast terrain of hills and valleys where nearly identical
answers score nearly identical probabilities. Stack Overflow, GitHub, Reddit, academic papers —
mountains of curated human thought, compressed and indexed. Within that landscape, the
navigation is brilliant. The model finds plausible paths across familiar ground at a speed no
human can match. More precisely, what looks like reasoning is weighted selection across that
terrain — best chains of prior answers, cleverly completed — rather than inference from first
principles.

The question is whether they can navigate _beyond_ that landscape — into genuinely unmapped
territory. When they try, the results are unreliable. Worse, the model has no felt sense of
when it has left familiar ground. It doesn't know it's confabulating.

I've watched this play out directly. When an AI encounters a bug of genuinely new provenance —
something outside its training threads — it cycles. Tries this, tries that, circles back.
Not because it's being careful. Because it has no real foothold. The training corpus doesn't
have enough threads to weave a solution, so it improvises badly and confidently.

Whether that's a temporary limitation or something more fundamental is genuinely contested.
There are serious AI researchers who believe it's an engineering problem — more data, better
architectures, longer context windows — and that the ceiling will keep rising. I'm not
convinced, but I hold that view loosely.


## Where the Work Actually Lives {#where-the-work-actually-lives}

Where the vibe coder's wall actually reveals something is in what breaks first. Managing
complexity is a craft skill — knowing how much abstraction a problem needs, where to draw
boundaries, when to slow down and when to push. Vibe coders are journeymen at best; they've
borrowed a senior engineer's output speed without the underlying judgment. LLMs handle the
syntactic and pattern layers of software engineering well. Where they struggle:

-   **Causal reasoning about systems** — understanding why something fails under conditions
    not present in training
-   **Constraint satisfaction across time** — holding an architectural decision from three months
    ago in tension with a decision being made today
-   **Recognizing genuine novelty** — knowing when a problem doesn't match any prior solved
    problem, instead of pattern-matching to a superficially similar one

That last one is the dangerous failure mode. A skilled engineer slows down when they're in
unfamiliar territory. LLMs accelerate into it.

This is the core argument for human-in-the-loop — not as a stopgap pending more training
data, but as a structural feature of what these systems are.


## The Novelty Question {#the-novelty-question}

[Thomas Kuhn's](https://plato.stanford.edu/entries/thomas-kuhn/) _Structure of Scientific Revolutions_ is useful here. Kuhn's observation was
that scientific fields don't progress smoothly — they accumulate friction. Experiments produce
results that don't quite fit the accepted model. Researchers find workarounds, patch the
theory, move on. The anomalies pile up quietly until the accumulated weight of them forces a
break — a new framework that reorders everything that came before. Kuhn also noticed that
these breaks tend to happen in parallel: when the conditions are ripe, multiple people reach
the same rupture point independently. Newton and Leibniz on calculus. Darwin and Wallace on
natural selection. The idea was, in some sense, waiting to be found.

If that's true — if major discoveries are largely the product of accumulated preconditions
rather than singular genius — then a model trained on enough scientific literature might
genuinely handle the _recombination_ step of discovery. Pattern-match across enough anomalies
and you might surface the connection a human would have found anyway, just faster.

[AlphaFold](https://www.deepmind.com/research/highlighted-research/alphafold) is the strongest evidence for this. It didn't reason about protein folding from
first principles. It found patterns in existing structural data that decades of human
researchers had missed — and in doing so, effectively solved a problem that had resisted the
field for fifty years. That's not a party trick. It's a genuine scientific result, and it
meaningfully advances medicine. The recombination argument has real teeth.

But the harder cases are breakthroughs that required _rejecting_ the existing framework
entirely. Special relativity didn't extend Newtonian mechanics — it revealed that Newtonian
mechanics was an approximation, valid only within a range of conditions nobody had previously
tested. Quantum mechanics didn't refine classical physics — it demolished the intuition that
physical systems have definite states independent of observation. [Gödel's incompleteness
theorems](https://plato.stanford.edu/entries/goedel/) didn't find a gap in mathematics — they proved that any sufficiently powerful formal
system _must_ contain true statements it cannot prove. Each of these required not just a new
answer, but a new conception of what the question was.

Could an LLM derive any of these? It's hard to see how. A system trained to predict within an
existing vocabulary has no mechanism for generating the insight that the vocabulary itself is
wrong. It can navigate the hills and valleys of what's known. The rupture points — the places
where the map has to be thrown out — seem to require something else.

There's an interesting concession worth making here, though. Einstein famously described his
path to relativity as rooted in physical intuition — the thought experiment of riding alongside
a beam of light, _feeling_ what that would mean. Human sensory experience, embodied in the
world, feeds an imaginative intuition that working purely with text and symbols doesn't obviously
replicate. LLMs lack that grounding entirely. But it's not inconceivable that future systems
with rich sensory input — robotics, continuous environmental feedback, genuine physical
interaction — might develop something closer to it. That's a different kind of AI than what
we have today. Whether it would be sufficient is a genuinely open question.


## Capital, Labor, and the Machine — New Wine, Old Wineskins {#capital-labor-and-the-machine-new-wine-old-wineskins}

None of this is new. The question of who profits from technological displacement has been
running since the first industrial looms.

[David Ricardo](https://www.econlib.org/library/Ricardo/ricP.html) initially believed the disruption would be self-correcting — that the money
saved on labor would get reinvested, creating new jobs to replace the ones the machines took.
He changed his mind, quietly, in a late chapter of his _Principles_ titled "On Machinery"
(1821), concluding that the machines could do permanent damage to working people. It reads
uncomfortably well today.

[Marx](https://plato.stanford.edu/entries/marx/) took that observation and built a structural argument from it. He wasn't simply making a
moral case for workers — he was arguing that the system runs the way it runs by design.
Owners benefit when labor is cheap and plentiful. Unemployed workers aren't a bug; they're
leverage. Technology that puts people out of work serves that logic whether anyone intends it
to or not. You don't have to be a Marxist to find the framework useful for thinking about
what happens when AI absorbs entire job categories.

[Mill](https://plato.stanford.edu/entries/mill/) offered the more optimistic read — that _how_ the gains from technology get distributed
is a choice societies make, not a law of nature. The machine creates wealth; who gets it is
a political question, not an economic inevitability. That distinction matters now more than
ever. It's the implicit argument behind every proposal for AI dividends, universal basic
income, or profit-sharing mandates — and, more immediately, behind whatever regulatory
frameworks governments are currently fumbling toward.

There's a useful historical precedent here. When industrial monopolies ran roughshod over
American workers and consumers at the turn of the 20th century, the answer wasn't to
dismantle capitalism — it was Theodore Roosevelt breaking up the trusts. The technology was
fine; the power concentration wasn't. We've done this before. The question is whether the
institutions move fast enough this time, and whether the political will exists to use them.

The debate hasn't moved as far as we'd like to think. What's changed is the speed, the scale,
and the fact that this time the machinery is targeting cognitive labor rather than physical.


## The Grounding Problem {#the-grounding-problem}

In 1980, philosopher John Searle proposed a thought experiment: imagine a person locked in a
room, following rules to manipulate Chinese symbols they don't understand, producing responses
that look fluent to a native speaker outside. The room passes the language test. The person
inside understands nothing. [Searle's point](https://plato.stanford.edu/entries/chinese-room/) was that processing symbols correctly is not the
same thing as understanding what they mean — and that no amount of sophistication in the
rule-following changes that fundamental gap.

The argument translates surprisingly well onto LLMs. When I reason about a system failing at
scale, I'm drawing on physical intuition, memory of specific past failures, the texture of
similar problems — a dense web of meaning built from years of living in and working on real
systems. The model has processed more text about system failures than I will ever read. But
processing descriptions of experience is not the same as having it.

LLMs have statistical shadows of that grounding. Not the thing itself.


## What It Means in Practice {#what-it-means-in-practice}

<img src="/ox-hugo/scopio-e4b94397-e8fa-4124-b459-858dcb4a6fe2.jpg" style="float:right; margin:0 0 1em 1em; max-width:400px;" alt="Leather wineskin">

The productivity gap AI creates is not incremental. A skilled engineer working with LLMs
doesn't do the same job somewhat faster — the difference is closer to swapping a horse and
buggy for a jet. Tasks that once required a team now require one person and an afternoon.
That compression is real, and anyone not adapting to it is watching the gap widen from the
wrong side.

Which brings the economic argument back around. Ricardo worried that the gains from machinery
would not find their way to labor. So far in the AI era, the engineers who understand the
tools deeply are capturing a disproportionate share of the value — not because the tools are
scarce, but because the judgment to use them well is. That's the arbitrage window, and it
won't stay open indefinitely.

The deeper question — whether AI can cross from recombination into genuine novelty, whether
it can throw out the map rather than just navigate it faster — remains open. We've been
trying to define human creativity for a few thousand years without resolution. We're unlikely
to settle it here.

What we can say is this: the industrial revolutions didn't end human work, they transformed
it. The wineskins stretched. The question for this one is the same as it always was — not
whether the technology changes everything, but who shapes the change, and who it changes for.


## Postscript: And Then There's the Power Bill {#postscript-and-then-there-s-the-power-bill}

The human brain runs on 20 watts. The inference load of a major LLM across its reported
user base runs to roughly the annual consumption of a medium-sized city. The meter is running.

---

_Further reading:_

-   [Joseph Schumpeter](https://www.schumpeter.info/) on creative destruction and innovation waves
-   [Brynjolfsson](https://www.brynjolfsson.com/research), on productivity and the technology diffusion lag
-   [Searle, "Minds, Brains, and Programs"](https://plato.stanford.edu/entries/chinese-room/) (1980)
-   Kuhn, _The Structure of Scientific Revolutions_ (1962)
-   Heilbroner, _The Worldly Philosophers_ (1953) — for the broader political economy backdrop
-   Ricardo, _Principles of Political Economy and Taxation_, Ch. 31: "On Machinery" (1821)
-   Mill, _Principles of Political Economy_ (1848) — on production vs. distribution as separable questions

---

<p style="font-size:0.85em; color:#666; margin-top:2em;">Original content written with the editorial assistance of <a href="https://claude.ai" target="_blank" rel="noopener">Claude</a>.</p>
