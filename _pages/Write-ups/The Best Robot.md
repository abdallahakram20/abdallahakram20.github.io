---
title: "The Best Robot — Web Challenge Write-up"
thumbnail: "/assets/img/writeups/the-best-robot/flag-response.png"
layout: page
permalink: /Write-ups/the-best-robot.html
---

# The Best Robot — Web Challenge Write-up

**Platform:** ScriptSorcerers CTF
**Category:** Web / Information Disclosure
**Difficulty:** Very Easy

---

## 1. Overview

"The Best Robot" is a minimal web challenge with no visible UI puzzle at all — just a blank landing page. With nothing to click on and no obvious form or parameter to attack, the challenge is really testing whether you go through the basic reconnaissance checklist before assuming there's nothing there.

---

## 2. Checking robots.txt First

Before touching anything else, I routed the browser through **Burp Suite's Proxy** and requested `robots.txt` directly from Repeater — it costs nothing to check and it's one of the first things worth ruling out on any target, CTF or real-world.

```
GET /robots.txt HTTP/1.1
Host: 682081b3-5bee-4c40-a7dd-5aab379711a4.challs.scriptsorcerers.xyz
```

![Burp Repeater showing the robots.txt response disallowing /the-best-robot](/assets/img/writeups/the-best-robot/robots-txt-response.png)
_The response comes back with a `Disallow` rule pointing straight at `/the-best-robot` — the site is explicitly telling crawlers not to index a path that isn't linked anywhere on the page itself._

That single line is the whole challenge, really: a path that was never meant to be *found by browsing*, only excluded from search indexing — which of course means a human (or a proxy) can still request it directly.

---

## 3. Requesting the Disallowed Path

With the path in hand, I sent a plain GET request to it from Repeater:

```
GET /the-best-robot HTTP/1.1
Host: 682081b3-5bee-4c40-a7dd-5aab379711a4.challs.scriptsorcerers.xyz
```

![Burp Repeater response showing the flag returned in plain text](/assets/img/writeups/the-best-robot/flag-response.png)
_The server responds `200 OK` and returns the flag directly in the body — no auth, no extra parameters, nothing else needed._

```
scriptCTF{r0b07s_4r3_t4k1ng_0v3r_52711980a099}
```

---

## 4. The Core Idea Behind the Challenge

- `robots.txt` is a **convention for crawlers**, not an access control mechanism. `Disallow` only asks well-behaved bots not to index a path — it does nothing to prevent a request from actually reaching it.
- Listing a sensitive or unlinked path in `robots.txt` is a classic way to accidentally advertise its existence to anyone who bothers to check the file, which is exactly the opposite of what teams usually intend when they add entries to it.
- The challenge's name is itself a hint — "the best robot" plays directly on "robots.txt", which is a nice reminder that CTF titles are often worth reading as clues, not just flavor text.

---

## 5. Key Takeaways

- **Always check `robots.txt` and `sitemap.xml` early in recon** — before automated scanning, before fuzzing, they take seconds to request and regularly leak paths nobody linked anywhere.
- **`Disallow` is not a security boundary.** Anything listed there should be treated as a path that needs its own proper authorization check on the server side — if it doesn't have one, it's exposed to anyone who reads the file.
- **Simple challenges test discipline, not skill.** This one didn't need Intruder, fuzzing, or any exploit — just following the standard recon checklist in order instead of assuming an empty page means there's nothing to find.
