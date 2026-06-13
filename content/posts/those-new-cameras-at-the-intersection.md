+++
title = "PSA: About Those New Cameras at the Intersection"
author = ["Rod Morison"]
date = 2026-06-12T16:53:00-07:00
tags = ["privacy", "surveillance", "alpr", "pasadena", "altadena"]
draft = false
topics = ["Privacy", "Surveillance"]
description = "New license-plate readers are going up around Pasadena and Altadena. Here's what they are, where your data goes (much further than you'd think), and the four questions worth asking before the next council meeting."
+++

<!-- hand-authored Hugo markdown — NOT an ox-hugo export from org/all-posts.org.
     Do not create an org entry with EXPORT_FILE_NAME "those-new-cameras-at-the-intersection"
     or ox-hugo will overwrite this file. -->

If you drive around Pasadena or Altadena, you may have noticed new cameras appearing at intersections — small, solar-paneled, mounted on their own poles, aimed at traffic. I noticed them too, and went looking for what they actually do. This post is the summary I wish someone had handed me: what these cameras are, where the data goes, and what's worth asking your local officials. Consider it a neighbor-to-neighbor PSA.

## What they are {#what-they-are}

The cameras are almost certainly automated license-plate readers (ALPRs), and the dominant vendor in this market — by a wide margin — is [Flock Safety](https://www.flocksafety.com). Pasadena PD already operates a Flock network that local reporting puts at roughly 73 cameras and growing ([Pasadena Now](https://www.pasadenanow.com)).

Flock's "Falcon" cameras photograph every passing vehicle and log the plate, make, model, color, time, and location. You can be searched by "vehicle signature" even without a plate match. The feeds roll up into [FlockOS](https://www.flocksafety.com/products/flock-os), a real-time platform that fuses plate reads, live video, gunshot detection, and 911 data across connected agencies. Flock's own marketing claims 12,000+ communities and 4,800+ law-enforcement agencies.

This isn't a red-light camera or a traffic counter. It's a searchable, time-stamped record of where your car has been.

## The part most people don't know {#the-part-most-people-dont-know}

The local pitch is reassuring: data is retained for 30 days, the city owns it, it auto-deletes. Some of that is even contractually true in some cities. But two findings from the national reporting change the picture:

**First, the network.** Flock operates a [National LPR Network](https://www.flocksafety.com/products/national-lpr-network). Any enrolled agency can run a single query reaching **83,000+ cameras across roughly 6,800 networks** nationwide. Enrollment is a toggle — "Enable National Lookup" — and about three-quarters of Flock's law-enforcement customers have it on. One audit logged **450,000+ nationwide searches in a single 30-day window** in spring 2025. The [ACLU found](https://data.aclum.org) data can sometimes be shared even when an agency opts out. So "our city keeps its data 30 days" can quietly coexist with "7,000 agencies across the country can search it."

**Second, the fine print moved.** In early 2026, Flock rewrote its national Terms & Conditions. The prior pledge that "Flock does not own and shall not sell Customer Data" was removed, and Flock granted itself a perpetual, irrevocable license to use customer data — surviving even contract termination. Cities nominally "own" their data, but watchdog investigations found departing customers receiving altered, low-resolution, metadata-stripped exports rather than raw footage. Flock calls this standard SaaS practice. Maybe so — but it's not what city councils thought they approved in 2023.

It's worth saying plainly: in the fact-checking behind this post, two of Flock's own privacy reassurances — that access is "restricted to approved users based on job role" and that "nothing is shared unless the customer chooses" — did **not** survive verification against the documented evidence.

## Not hypothetical {#not-hypothetical}

The abuse cases are documented, not speculative:

- A Texas sheriff's office [searched 83,000+ cameras across 6,809 networks to track a woman suspected of having an abortion](https://www.eff.org/deeplinks/2025/05/she-got-abortion-so-texas-cop-used-83000-cameras-track-her-down) — the recorded search reason was "had an abortion, search for female" — including queries against cameras in Washington and Illinois, where abortion is legal.
- Flock allowed U.S. Customs and Border Protection to access Illinois plate data in violation of a 2023 Illinois law barring LPR sharing for immigration or abortion enforcement; the Illinois Secretary of State audited Flock and ordered the access shut off.
- By February 2026, [NPR reported](https://www.npr.org/2026/02/17/nx-s1-5612825/flock-contracts-canceled-immigration-survillance-concerns) at least 30 localities had deactivated cameras or canceled Flock contracts, largely over fears that local cameras were feeding federal immigration enforcement — with audit logs suggesting local departments running searches on behalf of federal agencies after direct federal access was cut.
- The EFF and ACLU of Northern California are [suing San Jose](https://www.eff.org) over its ~500-camera ALPR network, arguing warrantless mass surveillance violates the California Constitution.

California has real guardrails on paper — Civil Code §1798.90.5 limits ALPR retention and sharing, and Gov. Code §7284 restricts immigration cooperation. But the California DOJ itself has flagged local police departments as the weak link in enforcing the ALPR law, and [LAist](https://laist.com) has reported California agencies sharing plate data with ICE and Border Patrol unlawfully.

## Closer to home {#closer-to-home}

A few local threads worth following (these are from local reporting — verify the current numbers yourself):

- Pasadena PD's network is around **73 cameras**, expanded amid resident calls for transparency and limits.
- Altadena is unincorporated, so its policing — and any cameras there — falls to the **LA County Sheriff**, a different operator with different oversight than Pasadena PD. An LA County oversight panel has asked Altadena and Pasadena residents to weigh in on plate surveillance.
- Next door, some South Pasadena residents are pushing to remove that city's Flock readers entirely.

Crowdsourced maps can show you what's deployed near you: [DeFlock](https://deflock.me) maps ALPR locations nationwide, and [Have I Been Flocked?](https://haveibeenflocked.com) checks whether your plate appears in published Flock audit logs.

## Four questions worth asking {#four-questions-worth-asking}

If you take one thing from this post, make it these — at a council meeting, in a public-comment letter, or to the oversight panel:

1. **Who owns the new cameras** at a given intersection — Pasadena PD, LA County Sheriff, or a private party (HOA, business) feeding the same network?
2. **What's the retention period, and does the contract still guarantee city ownership and raw-data access** under Flock's post-2026 terms?
3. **Is the local network enrolled in National Lookup?** If so, out-of-state and federally-tasked queries can reach it. Are California's §1798.90.5 restrictions actually configured — and audited?
4. **What happens when the precedents land?** The San Jose lawsuit and the Illinois audit will shape what's legal here. Is our contract written to adapt, or are we locked in?

None of this requires believing your local police department acts in bad faith. The Texas search and the Illinois violations happened through the network's *design*: local cameras, approved locally for local purposes, searchable nationally for purposes no council ever voted on. That's the gap between what gets approved and what gets built — and it's exactly the kind of gap that closes only when residents ask precise questions early.

---

*Method note: this post distills a longer research brief assembled with an AI deep-research pipeline — 24 sources, 119 extracted claims, 25 of them put through a three-vote adversarial fact-check (22 confirmed, 3 refuted — including two of Flock's own privacy claims). National findings are verified; local Pasadena/Altadena specifics come from local news and are flagged as leads to confirm. Vendor scale figures are Flock's self-reported marketing.*
