---
title: "Social Media Investigation Hub — Cross-Platform OSINT Challenge Write-up"
thumbnail: "/assets/img/writeups/social-media-investigation-hub/twitter-profile.png"
layout: page
permalink: /Write-ups/social-media-investigation-hub.html
---

# Social Media Investigation Hub — Cross-Platform OSINT Challenge Write-up

**Platform:** Hack The Box
**Category:** OSINT / Cross-Platform Investigation
**Difficulty:** Easy
**Certificate:** [View on HTB Achievements](https://labs.hackthebox.com/achievement/challenge/3670503/975)

---

## 1. Scenario

A tech company reports a wave of suspicious negative reviews for their new product, the **"XyloPhone Pro,"** spreading across multiple social platforms at once. The reviews read as authentic and well-written, but the consistently negative sentiment and the coordinated timing across platforms suggest this isn't organic feedback — it looks like a competitor-run campaign. The task: investigate the handle **`TechReviewer2024`** across platforms and build a case for coordinated, inauthentic activity using cross-platform OSINT correlation.

---

## 2. High-Level Approach (Methodology, Not a Walkthrough)

Cross-platform username investigations follow a repeatable loop rather than a single trick:

1. **Anchor on the identifier.** Start from the one constant across platforms — the handle/username — and check every major platform for a matching account rather than assuming it only exists where it was first spotted.
2. **Profile each account independently first.** Capture bio, join date, location, follower/following counts, and post history for each platform *before* comparing them — this avoids anchoring bias from whichever platform you found first.
3. **Look for identity leakage.** Casual, "professional" platforms (LinkedIn especially) tend to leak far more verifiable detail — real name, employment history, dates — than social platforms optimized for anonymity. This is usually where a persona's cover story cracks.
4. **Correlate timelines.** Account creation dates, first posts, and campaign-relevant posts that cluster tightly in time across platforms are a strong signal of a single operator running multiple fronts, rather than independent, organic accounts.
5. **Follow the social graph.** Accounts a persona interacts with (mentions, "connections," cross-links) often reveal a wider ring of coordinated sockpuppets, not just a single bad actor.
6. **Corroborate, don't assume.** Every claim in the final report should be traceable to a specific screenshot/post — reconstruct the case from evidence, not from the suspicion you started with.

---

## 3. Platform-by-Platform Findings

### 3.1 X / Twitter — `@TechReviewer2024`

![X/Twitter profile for TechReviewer2024](/assets/img/writeups/social-media-investigation-hub/twitter-profile.png)
*The `@TechReviewer2024` profile: bio "Honest tech reviews | DM for collaborations | #TechCommunity," located in San Francisco, joined February 2024.*

Key observations:
- **Very young account** (joined Feb 2024) with a low follower count (67) relative to its stated "reviewer" niche — inconsistent with an established, trusted reviewer voice.
- A post explicitly announces networking with two other accounts, **`@ReviewMaster_Bob`** and **`@TechTruth_Sally`**, framed as "other honest reviewers" — this is the first thread connecting a *group* of accounts rather than a lone actor.
- A follow-up post ("Working on some big reviews coming soon. The truth needs to be told!") is vague and pre-announces upcoming "reviews" days before any product-specific content appears — consistent with a campaign being staged in advance.

### 3.2 LinkedIn — Alex Morgan ("Tech Reviewer")

![LinkedIn profile for Alex Morgan, Tech Reviewer](/assets/img/writeups/social-media-investigation-hub/linkedin-profile.png)
*LinkedIn profile tying the "Tech Reviewer" persona to a real name, "Alex Morgan," and a work history.*

This is where the persona's cover story breaks down:
- Profile created **January 2024** — a month *before* the X/Twitter account, and with only 89 connections despite the "Product Analysis Specialist" title.
- Current role listed: **"Freelance Technology Reviewer,"** Jan 2024–Present — matching the timing of the whole persona's launch almost exactly.
- **Previous role: "Marketing Specialist" at RivalTech Inc.**, Jan 2021–Dec 2023 — ending literally the same month the "independent reviewer" persona begins. This single field is the strongest piece of evidence in the whole case: it directly links the supposedly neutral reviewer to a marketing background at a named company with an obvious commercial motive to discredit a rival product.

### 3.3 Reddit — `u/TechReviewer2024`

![Reddit profile and posts for u/TechReviewer2024](/assets/img/writeups/social-media-investigation-hub/reddit-profile.png)
*Reddit post history, including a thread literally titled "XyloPhone Pro Campaign Coordination."*

Reddit is where the coordination stops being circumstantial and becomes explicit:
- **"Building a Network of Honest Reviewers"** — a recruitment post inviting other "genuine" reviewers to DM and "coordinate our efforts," posted in r/TechReviews. Framing a coordinated messaging operation as grassroots authenticity is a classic astroturfing pattern.
- **"XyloPhone Pro Campaign Coordination"** — posted in r/ProductTesting, listing specific negative talking points to push ("Battery life issues — emphasize 2-hour claim," "Overheating problems during gaming"). This is a direct, dated artifact of a scripted negative-messaging campaign, not independent reviewer opinion.
- **"Effective Negative Review Strategies"** — posted in r/MarketingTips, openly asking how to "highlight product weaknesses without seeming biased" for a "competitive analysis project." Posting this kind of request under the *same* handle used for the "reviewer" persona is the account's biggest operational-security mistake, and the one that ties everything together.

---

## 4. Cross-Platform Correlation Summary

| Signal                              | X/Twitter                   | LinkedIn                              | Reddit                                    |
| ------------------------------------ | ---------------------------- | -------------------------------------- | ------------------------------------------ |
| Account age                          | Created Feb 2024             | Created Jan 2024                       | Cake day Jan 15, 2024                      |
| Location                             | San Francisco, CA            | San Francisco Bay Area                 | (not shown, but content matches same case) |
| Stated role                          | "Honest" reviewer            | Freelance Technology Reviewer          | Reviewer / campaign organizer              |
| Hidden conflict of interest          | —                             | Ex-Marketing Specialist, **RivalTech Inc.** | —                                       |
| Coordination evidence                | Names 2 other "reviewer" accounts | —                                  | Recruits reviewers, shares talking points  |

The pattern across all three platforms — near-identical creation dates, matching location, a persona built around "honesty" while quietly recruiting other accounts and scripting negative talking points, and a direct employment link to a named competitor — is consistent with a single operator (or small team) running a coordinated, inauthentic negative-review campaign against the XyloPhone Pro on behalf of RivalTech Inc.

---

## 5. Completion

Challenge solved on **27 Jul 2026** — *"Social Media Investigation Hub"* on Hack The Box (130 XP). [Achievement certificate.](https://labs.hackthebox.com/achievement/challenge/3670503/975)

---

## 6. Key Takeaways

- **Professional-network platforms leak the most.** LinkedIn's employment history was the single field that turned a vague suspicion into a documented conflict of interest — always check the "boring" professional platforms, not just the loud social ones.
- **Timeline correlation is a cheap, powerful signal.** Multiple accounts created within days or weeks of each other, all tied to the same campaign, rarely happens organically.
- **Bad actors often out themselves through reuse.** Posting "how do I write biased reviews that don't look biased" from the *same* account used to publish "honest reviews" is the kind of overlap that collapses an entire cover story.
- **Build the case from evidence, not narrative.** Every conclusion here traces back to a specific screenshot — that discipline is what separates an OSINT report from a hunch.
