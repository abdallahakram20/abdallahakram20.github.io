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

## 2. High-Level Approach (Concept, Not a Step-by-Step Walkthrough)

1. **Treat the terminal as a web app, not a game.** Anything typed into the input box gets handled by JavaScript running in your own browser — which means the full command tree, including "hidden"/secret branches, has to exist somewhere in that JS or in a request it makes.
2. **Open DevTools before typing random guesses.** The Network and Sources tabs matter more here than the terminal prompt itself.
3. **Watch what the terminal fetches as you play.** As the story progresses, the front-end calls a backend endpoint to pull the next batch of valid options for the current story "stage."
4. **Read the response, not just the rendered text.** The API response is structured data (a JSON object keyed by stage number) — and one key in that object doesn't correspond to any button or hint shown in the UI at all.

![DevTools Network tab showing the /api/options response](/assets/img/writeups/flag-command/devtools-network-options-endpoint.png)
_The `/api/options` response: numbered stages (`"1"`, `"2"`, `"3"`, `"4"`) map to the visible on-screen choices — but there's an extra `"secret"` key sitting alongside them that the UI never displays as a button._

---

## 3. The Core Idea Behind the Challenge

- The visible game only ever shows you `HEAD NORTH / SOUTH / EAST / WEST` and similar branching prompts — a normal player would spend their time trying every visible combination of those.
- Because the options are fetched from a backend API rather than hardcoded purely client-side, inspecting that traffic reveals the **full data model**, including entries the front-end code was never told to render.
- That's the pattern this challenge is really teaching: a UI only shows you what its designer *intended* you to see; the underlying data or API often carries more than the interface exposes. This shows up constantly in real bug bounty work — hidden API fields, unused parameters, or debug flags that never made it into the UI but are still live on the backend.
- Once you know the "secret" entry exists as plain text in the API response, it's simply a matter of feeding that exact string back into the terminal as a command.

![Flag returned after issuing the secret command](/assets/img/writeups/flag-command/terminal-flag-result.png)
_Submitting the secret command string returns the flag and the "you escaped the forest" win state._

---

## 4. Why This Took Longer Than It Should Have

Honestly, the maze framing is a bit of a red herring — I spent a chunk of time actually trying to navigate the forest logically (heading in different directions, trying to "solve" it like a real adventure game) before it clicked that the interesting part wasn't the story tree at all, it was the raw API response sitting quietly in the Network tab. The lesson: when a challenge dresses up a simple client/server data exposure as a game, don't get pulled into playing the game on its own terms — go look at what the app is actually saying to itself first.

---

## 5. Completion Confirmation

![Flag captured in the terminal](/assets/img/writeups/flag-command/terminal-flag-result.png)
_Flag captured: `HTB{...}` (redacted here — see your own HTB dashboard for the value tied to your instance)._

---

## 6. Key Takeaways

- **Client-side ≠ hidden.** Anything a JavaScript app can render, it first had to receive or compute — so the data is always inspectable via DevTools.
- **Read API responses in full, not just what the UI renders from them.** Extra keys, debug fields, or unused branches in a JSON payload are a common — and very real-world — source of information disclosure.
- **Don't let a challenge's narrative framing dictate your methodology.** A "maze" or "story" skin doesn't change the fact that it's still a web app making HTTP requests; approach it like one.
- **Network tab first, guessing second.** For any interactive web challenge, watching what's actually being requested and returned is almost always faster than brute-forcing the visible UI.
