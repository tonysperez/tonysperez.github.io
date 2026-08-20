---
title: Building a Default-Deny Data Boundary for an LLM Pipeline
description: When you care about controlling your data
categories: [ai]
tags: [dshield, dshield prism]
---

When I started building DShield Prism, I wanted the benefits of LLM-assisted analysis without giving up control of my data.

Prism processes honeypot telemetry: mostly commands and IP addresses. Analysis first runs on a local model, but Prism can optionally use a frontier LLM for tougher analysis and external CTI services for enrichment.

That creates a fairly obvious security question:

**What data should be allowed to be sent to these external systems?**

I needed the answer to be consistent, regardless of which component wanted to send the data or which external service it was calling.

So I built the egress model around a simple rule:

> If data hasn't been explicitly approved to leave, it doesn't leave.

## One Way In: Classification at Ingest

Every sensor's records carry a classification: `public` or `confidential`.

That classification is stamped in the Elastic Stack pipeline, before the data reaches any component capable of sending it elsewhere. A sensor marked confidential can still participate in the local analysis pipeline, but its data can't flow to cloud LLMs or external threat-intelligence services.

More importantly, what happens when data isn't classified at all?

Prism doesn't assume that an untagged record is *probably* safe. It treats it as confidential. If a configuration mistake, malformed event, or future code change causes classification to disappear or otherwise not be recognized, the safe outcome is still the default outcome.

Missing metadata shouldn't be able to break a security boundary.

## One Way Out

The next problem was making the control enforceable. It would have been easy to sprinkle checks throughout the code:

```text
if record_is_public:
    call_cloud_service()
```

That works until, you know, reality strikes. Someone adds another integration or another path capable of egressing data and forgets the check.

Instead, Prism routes **all automated egress paths** through the same releasability decision. Cloud LLM escalation and CTI queries don't each invent their own rules about what can leave. They ask the same question:

**Is this record explicitly releasable?**

If the answer isn't explicitly yes, the data stays local.

In addition to making the control harder to bypass accidentally, centralization makes auditing easier. There's one security invariant to inspect, test, and monitor.

## Cloud Is Opt-In

I also didn't want enabling Prism to implicitly enable data egress.

The pipeline works without external services. External services start disabled and have to be deliberately enabled.

### Cost Controls Are Security Controls

Even when enabled, external service use isn't unlimited. Cloud LLM usage has a hard daily budget, and external threat-intelligence lookups have query caps.

Originally, I thought of the LLM budget mostly as a way of avoiding an unexpected bill. Then, as so often happens, the software did something I hadn't intended: it entered an endless cloud LLM loop. The cap did exactly what it ought to, it turned an unbounded failure into a bounded one. A failure that didn't impact the other services I had relying on that same LLM provider.

That reinforced a broader lesson for me. With externally metered services, cost controls can also be security and reliability controls. Alerts have their place, but they only tell you that damage is occurring. Hard limits cap that damage.

## Why Enforce It in Code?

None of these ideas are especially exotic. Classify data. Minimize egress. Fail closed. Put limits around external dependencies. The difference is where those rules live. A README saying "don't send confidential data to the cloud" isn't a boundary. Hope is not a security control.

For Prism, I wanted the invariant to survive my own future mistakes:

**Only explicitly public data can leave the box. Everything else stays local.**

That doesn't make the system perfectly secure. Prism still has documented limitations, and interactive analyst workflows have different access requirements than the automated pipeline.

But the automated egress boundary doesn't depend on hoping someone remembers the policy at exactly the right moment.

That's becoming a recurring theme for me in AI security: **don't settle for a promise about what a system *shouldn't* do. Constrain what it *can* do.**
