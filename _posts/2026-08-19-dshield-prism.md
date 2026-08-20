---
title: DShield Prism - Refract the Noise, Resolve the Behavior
description: What was this attacker actually doing?
categories: [projects, ai]
tags: [dshield prism, dshield, sans isc]
pin: true
---

An internet-facing honeypot sees a firehose of mostly identical attacks. Buried somewhere in that deluge is the part worth reading: a novel technique, a quiet drift on a known campaign, a payload that shows up somewhere it shouldn't.

The usual answer is another dashboard. I built a few. They were great at telling me *how much* was happening and almost useless at telling me **what was happening**. Charts over raw IPs, commands, and timestamps don't survive the only question I actually cared about:

**What was this attacker doing?**

DShield Prism is my attempt at answering that. I built it during an apprenticeship with the SANS Internet Storm Center, where the work was mostly manual pivoting through honeypot logs, and I wanted to spend my time dissecting an attack instead of assembling it.

## Behavior Over Artifacts

The core bet is that behavior is the durable signal and everything else is disposable.

IPs rotate. Payload URLs move. Commands get lightly reshuffled. But the *sequence* (land, look around, disable something, fetch, persist) tends to survive all of that, because it reflects what the operator is actually trying to accomplish.

So Prism turns commands, sessions, and source IPs into behavioral fingerprints, matches new activity against a library of named behaviors, and flags whatever matches nothing. Recurring unknowns become new named behaviors. Once a behavior has a stable name, you can watch it change.

## A Boundary I Couldn't Forget To Enforce

Honeypot capture is not automatically safe to share. A single session can hold real credentials, a victim's data, or something that identifies the sensor. The moment a pipeline can reach a cloud model or an external intel feed, that's an egress problem.

Prism classifies every record at ingest and routes every path that leaves the box through one check. Untagged data is treated as confidential, so a missing tag can't open the door. I wrote about the reasoning in more detail in [Building a Default-Deny Data Boundary for an LLM Pipeline](https://tonystech.net/posts/llm-default-deny-egress/).

TL;DR - I didn't want a control that depended on me remembering it.

## Measured, Not Assumed

It would be very easy to build something like this, look at the output, decide it seems reasonable, and ship it.

Prism gates on a labeled evaluation set in CI instead. Some measurements gate the build and some are diagnostic only, and I try to keep that line honest, because a metric that can't fail a build isn't a control. The eval set is small and I don't pretend otherwise. It exists to catch regressions on this project, not to be a public benchmark.

The most useful thing that discipline did was tell me what to delete. At one point Prism asked the model for MITRE ATT&CK technique IDs. The model invented them faster than schema validation could catch, so I pulled the feature entirely. A confidently wrong TTP ID is worse than no TTP ID, and no amount of prompting was going to fix a model doing exactly what it was built to do. Cost got the same treatment. I put a hard daily cap on cloud spend from day one, which felt like paranoia right up until a bug put me in an endless query loop and the cap ate it.

## On Building It With AI

Prism was built with the assistance of generative AI, directed and reviewed by me. I held it to the same bar Prism applies to its own models: ground it, check the output against a schema, measure it against a baseline, and throw out what doesn't survive. The decisions, the measurements, and the calls on what *not* to ship are mine.

## The Details Live in the Repo

- [**github.com/tonysperez/dshield_prism**](https://github.com/tonysperez/dshield_prism)
- [Architecture and evaluation docs](https://github.com/tonysperez/dshield_prism/tree/main/docs)
- [Security model and known limitations](https://github.com/tonysperez/dshield_prism/blob/main/SECURITY.md)
