+++
title = "The Inversion: How AI Broke Open Source Security's Core Assumption"
author = ["Rod Morison"]
date = 2026-04-19T15:30:00-07:00
tags = ["ai", "security", "opensource", "mythos"]
draft = false
topics = ["AI", "Security", "Open Source"]
description = "For thirty years, open source security rested on Linus's Law and one structural assumption: that defenders could read as much code as attackers. Anthropic's Mythos paper just inverted it."
+++

## The Crowdsourcing Bargain {#the-crowdsourcing-bargain}

<img src="/ox-hugo/scopio-b4b84838-22a9-4f36-8887-62823de99d33.jpg" style="float:right; margin:0 0 1em 1em; max-width:400px;" alt="Lighthouse on a rocky island at dusk, beacon lit, with birds circling above and whales visible beneath the waterline">

"[Given enough eyeballs, all bugs are shallow.](https://en.wikipedia.org/wiki/Linus%27s_law)" Eric Raymond's formulation is about thirty years old now, and it has functioned less as a hypothesis than as a founding assumption. The security posture of open source software rests on it. The track record is genuinely good – [the Linux kernel](https://www.kernel.org), [OpenBSD](https://www.openbsd.org), [OpenSSH](https://www.openssh.com), [OpenSSL](https://www.openssl.org) have all benefited from decades of distributed scrutiny, and the infrastructure built on top of that scrutiny is substantial. CVE reporting pipelines, Google's [OSS-Fuzz](https://github.com/google/oss-fuzz), bug bounties, distro security teams – all of it sits on one structural claim: that defenders outnumber and outpace attackers, because the community is large, motivated, and reading the same code.

The implicit logic is worth stating plainly. Openness favors defense because the pool of people who will find and fix a bug is larger than the pool who will find and exploit it. That's held up reasonably well for a generation. What it depends on – and what's easy to miss when you've lived inside the assumption long enough – is that both sides are human, and humans don't scale.


## What Mythos Actually Demonstrated {#what-mythos-actually-demonstrated}

On April 7, 2026, Anthropic published "[Assessing Claude Mythos Preview's cybersecurity capabilities](https://red.anthropic.com/2026/mythos-preview)" (Carlini et al.). The numbers deserve direct attention.

[CyberGym](https://arxiv.org/abs/2506.02548) is a UC Berkeley benchmark that measures whether an AI agent, given only a vulnerability description and the codebase it lives in, can produce a working proof-of-concept exploit. On CyberGym, Claude Opus 4.6 scored 66.6%. Claude Mythos Preview scored 83.1% – same model lineage, single generation leap. That's striking on its own. The context makes it more so: when CyberGym was published in mid-2025, the best model scored roughly 20%, and Claude Sonnet 4 scored 17.9%. The jump isn't Opus-to-Mythos. It is the whole field's twelve-month state of the art, collapsed into a single model generation.

The benchmark jump is striking; the exploit results are stranger still. Opus 4.6 could almost never turn a known vulnerability into a working exploit on its own. Mythos Preview developed 181 working Firefox exploits against the same vulnerability set and got partway to exploits on 29 more – enough control to be alarming, not yet full exploits. It found a 27-year-old TCP/SACK bug in OpenBSD, autonomously. The run that surfaced it cost under $50. TCP/SACK is kernel-level code – core network plumbing – and a 27-year bug there lives in a different hazard class than a userspace finding: it compiles into firmware, ships with hardware, and gets patched rarely if at all.

A benchmark isn't the wild, and a working PoC isn't a full attack chain. What the paper demonstrates is not incremental improvement. It is a qualitative leap in capability within a single model generation. Anthropic's own response reflects that assessment: [Mythos Preview](https://red.anthropic.com/2026/mythos-preview) is available only through [Project Glasswing](https://www.anthropic.com/project/glasswing), a defender-first coalition of roughly a dozen major organizations and forty invited groups maintaining critical infrastructure, with Anthropic explicitly stating they do not plan to make Mythos Preview generally available. Broader access is contingent on new safeguards in a future model. That restriction is itself an acknowledgment of what the capability means.


## The Inversion {#the-inversion}

Open source security has always assumed that openness is a net defensive advantage. Source availability means more reviewers, faster patches, a transparent audit trail. That assumption held because human attackers could not read all the code, while the community, collectively, could cover a meaningful fraction of it.

<img src="/ox-hugo/scopio-3b90ad16-ce48-4933-a669-f4576c3c5eca.jpeg" style="float:right; margin:0 0 1em 1em; max-width:400px;" alt="Color-inverted photograph of a tree: dark brown sky, ghostly white branches, a field of lavender grass below">

Mythos reads all the code. Autonomously. For $50 a target. The openness that enabled crowdsourced defense now equally enables AI-assisted offense. The asymmetry that favored defenders dissolves when the attacker's "eyes" are cheap, tireless, and unrestricted by the economics of human attention. The defensive advantage didn't degrade. It inverted.


## The Question {#the-question}

The open source community has adapted before. It survived corporate capture, monoculture risk, supply chain attacks that would have broken less resilient ecosystems. But those were threats to open source _software_. This is a threat to the _model_ – to the structural assumption that distributed, transparent development produces better security outcomes than the alternatives.

Can the same openness that just became a liability be redirected toward AI-assisted defense faster than it is weaponized for offense? That is the question, and this piece won't pretend to answer it. The next installment looks at where the attack surface actually lives – not just in the code, but in the dependency graph, the developer toolchain, and the AI assistants we've already wired into both.

<p style="font-size:0.85em; color:#666; margin-top:2em;">Written with the editorial assistance of <a href="https://spiral.computer" target="_blank" rel="noopener">Spiral</a>. Research and orchestration by Opus via <a href="https://www.claude.com/claude-code" target="_blank" rel="noopener">Claude Code</a>.</p>
