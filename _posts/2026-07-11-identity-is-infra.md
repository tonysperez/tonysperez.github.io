---
title: Identity Is Infrastructure - Lessons From Operating Hybrid AD and Entra ID
description: How bad could messing up your identity system really be?
categories: [iam]
tags: []
--------

For a long time, I thought of identity as one part of systems administration. Active Directory handled users and computers. Entra ID handled cloud identity. SSO connected applications. MFA added another layer of protection.

After owning identity in a hybrid, multi-forest Active Directory and Entra ID environment, I started looking at it differently:

**Identity isn't just another workload. It's core infrastructure.**

A network outage may take down a site. An identity problem can prevent people from reaching email, applications, administrative tools, servers, or even the systems you need to fix the identity problem itself. Or, arguably worse, it can grant improper access to sensitive information or systems.

That changed how I think about IAM.

## AuthN Is Only the Beginning

It's easy to reduce IAM to authentication:

**Can this person prove who they are?**

That's important, but it's just a fraction of the problem. The harder question is authorization:

**Okay, I know who you are. What should you actually be allowed to do?**

In practice, that means thinking about several layers together:

* MFA
* Conditional Access
* contextual access conditions
* just in time access
* group membership
* role assignment and RBAC
* identity lifecycle
* exceptions

A user successfully authenticating is an entirely different beast from a user being appropriately authorized. This distinction sounds obvious until you're dealing with dozens of applications, changing job responsibilities, inherited group memberships, and access that has accumulated over time.

## Groups Become Security Policy

Group-based RBAC looks simple on a diagram.

User goes into group. Group gets access. Done.

Reality has a funny way of messing with ideals.

Some groups represent departments or job functions. Others grant access to applications, file shares, or administrative capabilities. Some exist because of a migration three years ago and nobody is entirely sure whether they're still needed.

One pattern I came to appreciate is Microsoft's **AGDLP**:

**Accounts → Global Groups → Domain Local Groups → Permissions**

While AGDLP is specific to the way Active Directory groups work, I find the broader idea useful because it separates **identity** from **access**.

An identity group answers:

**What role does this person perform?**

An access group answers:

**What resource or permission does this role receive?**

Instead of assigning permissions directly to users—or even directly to broad organizational groups—you place users into groups that describe their role, then nest those groups into groups that describe access.

For example:

```text
Alice
  ↓
GG_Finance_Analysts
  ↓
DLG_FinanceReports_Read
  ↓
Read access to Finance Reports
```

That separation matters. Or, as LLMs seem contractually obligated to say, "it's the whole ball game."

If Alice moves from Finance to another department, I shouldn't have to hunt through file servers and applications looking for permissions assigned directly to her account. Her role changes, so her membership in the Finance identity group changes. The access associated with that role follows naturally.

The directory starts to express policy:

**These people perform this role. This role receives this access.**

Of course, clean models have a way of getting messy over time. Groups accumulate, nesting becomes harder to follow, exceptions appear, and temporary access sometimes outlives the reason it was granted. That's why I've come to think of group membership as security policy, not just directory administration.

Adding someone to an identity group is an authorization decision.

Mapping an identity group to an access group is an authorization decision.

Leaving either relationship in place is also an authorization decision.

And like most "just remember to audit it later" things, the last one is the easiest to forget.

Least privilege isn't a destination. It's an ongoing process of evaluating whether the identity, role, and access relationships you have today still reflect how the organization actually operates.

## Federation Moves the Boundary

SAML and OIDC make application access dramatically easier to manage.

They also move security decisions into your identity platform.

Once an application trusts your identity provider, configuration choices around claims, groups, roles, MFA, and Conditional Access become part of *that application's* security model.

That's useful because it gives you a central place to enforce policy. It also means mistakes can scale.

A poorly scoped access rule isn't necessarily isolated to a single workstation or server. Depending on the design, it can affect an entire application or population of users.

On a side note, this is one reason I like guardrails that limit the maximum impact of a bad authorization decision. In cloud environments, concepts such as AWS Control Policies and permissions boundaries serve a similar purpose: even when an individual policy is wrong, broader controls can limit how wrong it is allowed to be.

That made me much more cautious about treating identity changes as routine configuration. They're production security changes.

## Conditional Access: More Context

Phishing-resistant MFA is one of the strongest controls we have, but authentication decisions aren't always binary.

Who is signing in?

What are they trying to access?

From where?

Under what conditions?

Conditional Access gives you a way to make those decisions with more context than a simple username and password.

The hard part isn't creating a policy. It's operating a growing collection of policies without losing track of how they interact. And as environments grow, exceptions accumulate, applications behave differently, and legitimate business requirements don't always line up neatly with the cleanest security design.

A control that is technically correct but routinely prevents people from doing their jobs will eventually create pressure for bypasses.

The goal isn't simply to make access harder.

It's to make legitimate access predictable while making inappropriate access difficult.

## Exceptions Have Gravity

One lesson that applies well beyond identity is that temporary exceptions, like most things labeled "temporary" in IT, have a strange tendency to become permanent.

An MFA exception gets created for a workflow that can't support it yet.

A broader group gets used because a granular role doesn't exist.

An old authentication method stays enabled because one application still depends on it.

Each decision can make sense individually. But the danger comes when nobody revisits them—which, naturally, never happens in IT because every aspect of architecture and configuration is regularly reviewed, perfectly documented, and immediately corrected.

Exceptions accumulate weight.

Eventually, other systems and processes start depending on them, which makes removing them much harder than creating them ever was. I've learned to treat an exception as something that should have an owner, a business justification, documentation, and, whenever possible, an expiration date.

"Temporary" should describe a lifecycle, not a hope. Hope is not a security control.

## Identity Failures Have a Large Blast Radius

The more centralized identity becomes, the more important it becomes to understand its failure modes.

A bad endpoint configuration might affect one system.

A bad identity policy can affect an entire organization.

That means IAM work deserves many of the same disciplines we apply elsewhere in infrastructure:

* understand the blast radius
* make changes deliberately
* preserve a recovery path
* test assumptions
* document why the control exists
* know what happens when a dependency fails

Identity systems are security systems, but unlike some other security controls, they're also availability systems.

A perfectly secure environment that prevents legitimate users from doing their jobs isn't successful by any useful metric. Well, maybe account compromise rate. It's hard to compromise an account nobody can use. But I digress.

## What Changed for Me

The biggest shift in my thinking was realizing that IAM sits underneath almost everything else.

Applications depend on identity.

Administrators depend on identity.

Cloud services depend on identity.

Security controls depend on identity.

And because identity connects all of them, small decisions can have very large consequences.

I still think about IAM in terms of MFA, federation, RBAC, Conditional Access, and lifecycle management. But I no longer think of those as separate features. They're pieces of the same question:

**How do you make sure the right identity has the right access, under the right conditions, for exactly as long as it needs it?**

That's why I now think of identity as infrastructure.
