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

## 2. Step 1 — Inspecting the Landing Page

Before touching anything else, I routed the browser through **Burp Suite's Proxy** and requested the root path to see exactly what the server sends back, not just what renders visually:

```
GET / HTTP/1.1
Host: 682081b3-5bee-4c40-a7dd-5aab379711a4.challs.scriptsorcerers.xyz
```

The rendered page itself was empty, but reading the raw HTML response in Burp — rather than just what the browser displays — turned up an HTML comment left in the source:

```html
<!-- Developers: Visit /the-best-robot for a surprise! -->
```

That's the actual discovery moment: a comment that never shows up on the rendered page but sits right there in the response body for anyone who reads the source instead of just the screen.

---

## 3. Step 2 — Confirming via robots.txt

Checking `/robots.txt` is something I do on **every** web target as a first-step habit, regardless of whether I already have a lead — it costs one request and regularly surfaces paths nobody linked anywhere. In this case it also happened to line up with, and confirm, the comment from Step 1:

```
GET /robots.txt HTTP/1.1
Host: 682081b3-5bee-4c40-a7dd-5aab379711a4.challs.scriptsorcerers.xyz
```

![Burp Repeater showing the robots.txt response disallowing /the-best-robot](/assets/img/writeups/the-best-robot/robots-txt-response.png)
_The response comes back with a `Disallow` rule pointing at `/the-best-robot` — the exact same path hinted at in the HTML comment, now confirmed from a second, independent source._

Two unrelated places on the same target pointing at the same path is a strong enough signal to go request it directly.

---

## 4. Step 3 — Requesting the Disallowed Path

With the path confirmed twice over, I sent a plain GET request to it from Repeater:

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

## 5. The Core Idea Behind the Challenge

- The rendered page is not the full response. Browsers hide HTML comments from the visible page, but an intercepting proxy (or plain view-source) shows the raw body exactly as the server sent it — that's where the first lead was.
- `robots.txt` is a **convention for crawlers**, not an access control mechanism. `Disallow` only asks well-behaved bots not to index a path — it does nothing to prevent a request from actually reaching it. Here it doubled as free confirmation of a path I already suspected.
- The challenge's name is itself a hint — "the best robot" plays directly on "robots.txt", a reminder that CTF titles are often worth reading as clues, not just flavor text.

---

## 6. Key Takeaways

- **Read the raw response, not just the rendered page.** HTML comments, unused JS variables, and debug markup routinely leak information that never reaches the visible UI.
- **Check `robots.txt` and `sitemap.xml` on every target as a standing habit** — before automated scanning, before fuzzing. They take seconds to request and sometimes independently confirm a lead you already have.
- **`Disallow` is not a security boundary.** Anything listed there should have its own proper authorization check on the server side — if it doesn't, it's exposed to anyone who reads the file.
- **Corroborate leads from more than one source when you can.** Finding the same path hinted at from two unrelated places (a comment and a robots rule) is a good signal it's worth pursuing before you request it.
