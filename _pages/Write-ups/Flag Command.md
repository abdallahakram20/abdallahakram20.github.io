---
title: "Flag Command — Web Challenge Write-up"
thumbnail: "/assets/img/writeups/flag-command/terminal-full-session.png"
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

![The terminal boot-up story text](/assets/img/writeups/flag-command/terminal-full-session.png)
_The terminal's intro sequence — a lucid-dream framing device for a browser-based text adventure._

On the surface, it looks like a pure "guess the right sequence of commands" puzzle. In practice, the real skill being tested is **not** guessing — it's recognizing that a JavaScript-driven terminal like this has to *know* every possible branch of the story somewhere in the code it ships to your browser, and that a client-side app can never fully hide its own logic from the client.

---

## 2. Methodology — Intercepting Traffic with Burp Suite

Instead of clicking through the visible options one by one, I routed the browser through **Burp Suite's Proxy** and let the app run normally, then worked through the traffic it generated.

1. **Let Burp's Target → Site map build passively.** Just browsing/using the app normally is enough for Burp to map out every endpoint it touches — not just the ones you'd notice from clicking around.

![Burp Suite Target site map showing the /api/options endpoint](/assets/img/writeups/flag-command/burp-target-sitemap-options.png)
_The Site map surfaces `/api/options` as a distinct JSON endpoint sitting alongside the static JS/CSS assets — a good example of how a proxy gives you a full inventory of an app's surface, not just what you happened to click on._

2. **Send the interesting request to Repeater.** This decouples the request from the game's live state — you can hit the endpoint as many times as you want and pick apart the response at your own pace, instead of racing the terminal's UI.

3. **Read the response in full using the Inspector panel**, not just the parts the front-end renders. Structured JSON responses often carry more than what the UI is coded to display.

![Burp Repeater showing the JSON response with the extra field highlighted in the Inspector](/assets/img/writeups/flag-command/burp-repeater-response-secret.png)
_Repeater's response pane plus the Inspector panel makes it easy to isolate and copy a field cleanly — there's a key in this response that has nothing rendered for it anywhere in the game's UI._

4. **Feed the value found there back into the terminal**, in the actual browser tab, exactly as the game expects input.

![The command typed into the browser terminal, returning the flag](/assets/img/writeups/flag-command/browser-terminal-flag-captured.png)
_Submitting the hidden command returns the flag and the "you escaped the forest" win state._

The workflow here — passive proxying to map an app's endpoints, Repeater to isolate and replay a specific request without disturbing app state, and the Inspector panel to pull a field out cleanly — is the same loop used on far less trivial targets, so it's worth practicing even on a "very easy" challenge like this one.

---

## 3. The Core Idea Behind the Challenge

- The visible game only ever shows you `HEAD NORTH / SOUTH / EAST / WEST` and similar branching prompts — a normal player would spend their time trying every visible combination of those.
- Because the options are fetched from a backend API rather than hardcoded purely client-side, inspecting that traffic reveals the **full data model**, including entries the front-end code was never told to render.
- That's the pattern this challenge is really teaching: a UI only shows you what its designer *intended* you to see; the underlying data or API often carries more than the interface exposes. This shows up constantly in real bug bounty work — hidden API fields, unused parameters, or debug flags that never made it into the UI but are still live on the backend.
- Once you know that extra entry exists as plain text in the API response, it's simply a matter of feeding that exact string back into the terminal as a command.

---

## 4. Why This Took Longer Than It Should Have

Honestly, the maze framing is a bit of a red herring — I spent a chunk of time actually trying to navigate the forest logically (heading in different directions, trying to "solve" it like a real adventure game) before it clicked that the interesting part wasn't the story tree at all, it was the raw API response sitting quietly in Burp's traffic. The lesson: when a challenge dresses up a simple client/server data exposure as a game, don't get pulled into playing the game on its own terms — go look at what the app is actually saying to itself first.

---

## 5. Completion Confirmation

![Flag captured in the terminal](/assets/img/writeups/flag-command/browser-terminal-flag-captured.png)
_Flag captured: `HTB{...}` (redacted here — see your own HTB dashboard for the value tied to your instance)._

---

## 6. Key Takeaways

- **Client-side ≠ hidden.** Anything a JavaScript app can render, it first had to receive or compute — so the data is always inspectable via an intercepting proxy or DevTools.
- **Read API responses in full, not just what the UI renders from them.** Extra keys, debug fields, or unused branches in a JSON payload are a common — and very real-world — source of information disclosure.
- **Don't let a challenge's narrative framing dictate your methodology.** A "maze" or "story" skin doesn't change the fact that it's still a web app making HTTP requests; approach it like one.
- **Proxy first, guessing second.** For any interactive web challenge, watching what's actually being requested and returned (Burp Suite, or DevTools' Network tab) is almost always faster than brute-forcing the visible UI.
