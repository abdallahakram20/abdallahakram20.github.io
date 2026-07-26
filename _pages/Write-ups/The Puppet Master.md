---
title: "The Puppet Master — OSINT Challenge Write-up"
thumbnail: "/assets/img/writeups/puppet-master/bushmaster-front-view.jpg"
layout: page
permalink: /Write-ups/the-puppet-master.html
---

# The Puppet Master — OSINT Challenge Write-up

**Platform:** Hack The Box
**Category:** OSINT / Open-Source Intelligence
**Difficulty:** Very Easy
**Certificate:** [View on HTB Achievements](https://labs.hackthebox.com/achievement/challenge/3670503/977)

---

## 1. Overview

The scenario frames the player as an analyst reviewing evidence submitted by a field source, but the actual skill being tested is a transferable one: turning a photo into a confirmed identity through careful, source-checked research rather than guesswork.

![Investigation platform evidence page](/assets/img/writeups/puppet-master/evidence-page.png)
_Caption (EN): The evidence page as presented on the investigation platform._

![Evidence assessment / dashboard](/assets/img/writeups/puppet-master/evidence-assessment.png)
_Caption (EN): The platform's evidence assessment view, listing confirmed visual elements and the investigation vectors required to solve the challenge._

---

## 2. High-Level Approach (No Exploit Details — This Is a Research Task, Not an Exploit)

This challenge has no technical exploitation component, so the approach below is a research methodology rather than an attack chain:

1. **Observe first, search second.** Before touching a search engine, note down every distinguishing visual trait in the photo: vehicle silhouette, wheel arrangement, hull shape, camouflage pattern, visible mounts or antennas, and the general terrain/environment.
2. **Reverse image search.** Run the photo (or close crops of distinctive parts) through multiple reverse-image tools rather than relying on just one — different engines index different sources.
3. **Cross-reference specifications.** Once a strong candidate emerges, verify it against independent public sources (manufacturer pages, defense reference sites, encyclopedic entries) rather than a single page, to avoid anchoring on the first plausible match.
4. **Reconcile details into the answer format.** Combine the confirmed, cross-checked facts into whatever structured format the challenge requests.

![Evidence photo — confirmed vehicle](/assets/img/writeups/puppet-master/bushmaster-front-view.jpg)
**The evidence photo used as the starting point for the reverse-image search.**

---

## 3. Confirmed Identity

- **Name:** Bushmaster Protected Mobility Vehicle (PMV)
- **Manufacturer:** Thales Australia
- **Country of Origin:** Australia
- **In Service:** 1997–present
- **Type:** Infantry Mobility Vehicle

![Wikipedia infobox reference](/assets/img/writeups/puppet-master/wikipedia-infobox.png)
_Caption (EN): Public reference entry confirming the vehicle's identity and service history._

### 3.1 Supporting Cross-References

![NZ Army official equipment page](/assets/img/writeups/puppet-master/nzarmy-bushmaster-page.jpg)
_Caption (EN): An operator's official equipment page (New Zealand Army) describing the same vehicle in public service._

![Public specifications reference](/assets/img/writeups/puppet-master/wikipedia-specs-table.png)
_Caption (EN): Publicly documented technical specifications used to cross-check the identification._

---

## 4. Data Dictionary — What the "Evidence" Represents

This is a conceptual description of the artifact type used in the challenge, not a reproduction of any real leaked or sensitive dataset:

| Field                  | Description                                                                                                      |
| ---------------------- | ---------------------------------------------------------------------------------------------------------------- |
| `asset_type`           | General category of the object shown (e.g., wheeled armored vehicle)                                             |
| `visual_markers`       | Distinguishing features used for identification (hull geometry, wheel count, mounts, paint scheme)               |
| `environment_context`  | Setting of the photo (terrain, lighting, apparent purpose — training vs. operational)                            |
| `candidate_matches`    | Shortlist of publicly documented equipment types resembling the asset                                            |
| `confirmed_identity`   | The single candidate supported by consistent cross-referenced sources                                            |
| `specification_fields` | Publicly published technical facts about the confirmed identity (origin, manufacturer, era of service, capacity) |

No private, personal, or sensitive data is involved anywhere in this challenge — every fact used to solve it is drawn from openly published, public-domain reference material.

---

## 5. Deriving the Flag — Conceptual Method

Rather than walking through the specific answer, here's the reasoning pattern that gets a player there:

- **Pattern recognition:** matching the photographed object's shape and layout against known silhouettes of publicly documented equipment.
- **Metadata triangulation:** treating each independently confirmed spec (origin country, manufacturer, in-service date, capacity figures) as one data point, and only trusting a fact once it appears consistently across more than one independent public source.
- **Format assembly:** once every required field is independently confirmed, assembling them in the exact structure/casing the challenge specifies (this challenge is picky about exact formatting — re-read the submission instructions carefully before submitting).

---

## 6. Completion Confirmation

![Challenge solved confirmation](/assets/img/writeups/puppet-master/pwned-success.png)
"The Puppet Master has been Pwned" — confirmation screen after submitting the correct flag.\_

---

## 7. Key Takeaways

- A structured research loop — **Observe → Search → Shortlist → Cross-verify → Assemble** — beats jumping straight to a search engine.
- Never trust a single source for a technical fact; corroboration across independent references is what actually confirms an identification.
- Isolating distinctive visual markers (shape, proportions, distinctive add-ons) dramatically improves reverse-image search accuracy over searching the whole image at once.
