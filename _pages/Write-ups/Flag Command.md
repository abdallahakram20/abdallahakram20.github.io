---
title: "Flag Command — Web Challenge Write-up"
thumbnail: "/assets/img/writeups/flag-command/terminal-start-fresh.png"
layout: page
permalink: /Write-ups/flag-command.html
---

# Flag Command — Web Challenge Write-up

**Platform:** Hack The Box
**Category:** Web / Client-Side Logic
**Difficulty:** Very Easy
**Certificate:** [View on HTB Achievements](https://labs.hackthebox.com/achievement/challenge/3670503/646)

---

## 1. Overview

"Flag Command" drops you into a fake in-browser terminal that tells the story of the *Dimensional Escape Quest*: you wake up in a bizarre alien forest and have to navigate a text-adventure maze using terminal-style commands (`start`, `head north`, `head south`, etc.) to escape.

![The terminal boot-up story text and first set of options](/assets/img/writeups/flag-command/terminal-start-fresh.png)
_Fresh session — the story intro plays, `start` is typed, and the game presents its first branch: four directions to head in._

On the surface it reads like a straightforward "pick the right path" puzzle. Four options, no obvious catch. So that's exactly how I started — by actually playing it.

---

## 2. First Attempt — Playing It Straight

I picked a direction and followed where it led.

![Heading north leads to a dead end and a game-over screen](/assets/img/writeups/flag-command/terminal-died-head-north.png)
_`HEAD NORTH` → a small tavern → `GO DEEPER INTO THE FOREST` → ambushed by fairies → game over. Each branch narrows down to more options, and a wrong pick anywhere along the chain just resets you._

That run ended in death after two choices. So I restarted and tried again — different directions, different branches, working through the story's various forks by hand. Every path I tried eventually either looped back to a dead end or killed me off with a different flavor-text death. None of the visible branches led anywhere close to an actual flag.

At that point it was clear the "correct" sequence — if one even existed — wasn't something worth brute-forcing by hand. The story tree was a distraction from what actually mattered: how the game was getting its options in the first place.

---

## 3. Switching Approach — Intercepting Traffic with Burp Suite

Instead of continuing to guess, I routed the browser through **Burp Suite's Proxy** and let the app run normally, then worked through the traffic it generated.

1. **Let Burp's Target → Site map build passively.** Just using the app normally is enough for Burp to map out every endpoint it touches — not just the ones you'd notice from clicking around.

![Burp Suite Target site map showing the /api/options endpoint](/assets/img/writeups/flag-command/burp-target-sitemap-options.png)
_The Site map surfaces `/api/options` as a distinct JSON endpoint sitting alongside the static JS/CSS assets — a good example of how a proxy gives you a full inventory of an app's surface, not just what you happened to click on._

2. **Send the interesting request to Repeater.** This decouples the request from the game's live state — you can hit the endpoint as many times as you want and pick apart the response at your own pace, instead of racing the terminal's UI.

3. **Read the response in full using the Inspector panel**, not just the parts the front-end renders. Structured JSON responses often carry more than what the UI is coded to display.

![Burp Repeater showing the JSON response with the extra field highlighted in the Inspector](/assets/img/writeups/flag-command/burp-repeater-response-secret.png)
_Repeater's response pane plus the Inspector panel makes it easy to isolate and copy a field cleanly — there's a key in this response that has nothing rendered for it anywhere in the game's UI, sitting right alongside the numbered stages that do map to visible options._

---

## 4. The Core Idea Behind the Challenge

- The visible game only ever shows you `HEAD NORTH / SOUTH / EAST / WEST` and similar branching prompts — a normal player spends their time trying every visible combination of those, exactly like I did in Section 2.
- Because the options are fetched from a backend API rather than hardcoded purely client-side, inspecting that traffic reveals the **full data model**, including entries the front-end code was never told to render.
- That's the pattern this challenge is really teaching: a UI only shows you what its designer *intended* you to see; the underlying data or API often carries more than the interface exposes. This shows up constantly in real bug bounty work — hidden API fields, unused parameters, or debug flags that never made it into the UI but are still live on the backend.
- Once you know that extra entry exists as plain text in the API response, it's simply a matter of feeding that exact string back into the terminal as a command — same input box, same terminal, just a value nobody put a button for.

![The hidden command typed into the browser terminal, returning the flag](/assets/img/writeups/flag-command/browser-terminal-flag-captured.png)
_Submitting the hidden command in the actual browser terminal returns the flag and the "you escaped the forest" win state._

---

## 5. Why This Took Longer Than It Should Have

Honestly, the maze framing is a genuine red herring, and I fell for it — I spent a real chunk of time actually trying to navigate the forest logically (heading in different directions, dying repeatedly, restarting) before it clicked that the interesting part wasn't the story tree at all, it was the raw API response sitting quietly in Burp's traffic. The lesson: when a challenge dresses up a simple client/server data exposure as a game, don't get pulled into playing the game on its own terms — go look at what the app is actually saying to itself first.

---

## 6. Key Takeaways

- **Client-side ≠ hidden.** Anything a JavaScript app can render, it first had to receive or compute — so the data is always inspectable via an intercepting proxy or DevTools.
- **Read API responses in full, not just what the UI renders from them.** Extra keys, debug fields, or unused branches in a JSON payload are a common — and very real-world — source of information disclosure.
- **Don't let a challenge's narrative framing dictate your methodology.** A "maze" or "story" skin doesn't change the fact that it's still a web app making HTTP requests; approach it like one.
- **Proxy first, guessing second.** For any interactive web challenge, watching what's actually being requested and returned (Burp Suite, or DevTools' Network tab) is almost always faster than brute-forcing the visible UI — as the dead-end run in Section 2 makes clear.
