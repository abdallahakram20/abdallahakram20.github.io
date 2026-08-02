---
title: "Cosmic Explorer — Web Challenge Write-up"
thumbnail: "/assets/img/writeups/cosmic-explorer/burp-repeater-dual-key-flag.png"
layout: page
permalink: /Write-ups/cosmic-explorer.html
---

# Cosmic Explorer — Web Challenge Write-up

**Platform:** Hack The Box
**Category:** Web / API Logic Flaw (JSON Parser Differential)
**Certificate:** [View on HTB Achievements](https://labs.hackthebox.com/achievement/challenge/3670503/1317)

---

## 1. Overview

"Cosmic Explorer" presents a neon sci-fi UI with two radio options — **Scan Cosmic Anomaly** and a greyed-out, clearly-off-limits **Access Secure Database** — and a single **Initiate Scan** button. The obvious surface-level feature is the anomaly scanner; the secure-database option is styled to look locked, which is usually a hint that it isn't actually locked server-side, only hidden client-side.

![Cosmic Explorer UI showing a Dark Matter Veil anomaly result](/assets/img/writeups/cosmic-explorer/ui-scan-dark-matter-veil.png)
_First scan — `Initiate Scan` with the default "Scan Cosmic Anomaly" option selected returns a random anomaly, "Dark Matter Veil", rendered with its own stock image._

---

## 2. Mapping the Normal Behavior

Before touching the "secure" option, I ran the scan a few more times to see how the anomaly endpoint behaves normally. Each request returns a different randomly-picked anomaly, confirming the front end is just calling a backend action and rendering whatever comes back — nothing is hardcoded client-side.

![Cosmic Explorer UI showing a Pulsar Emission anomaly result](/assets/img/writeups/cosmic-explorer/ui-scan-pulsar-emission.png)
_A second scan returns a different anomaly, "Pulsar Emission" — confirming the result is generated server-side per request rather than being a fixed client-side value._

This is the same lesson as most of these challenges: whatever the UI shows you is just a rendering of a JSON action/response cycle. The real logic lives in the backend source, so the next step was reviewing it directly rather than guessing at the UI.

---

## 3. Reading the Source — Two Services, Two Languages

The challenge ships two backend services rather than one:

- **Go gateway** (port `8080`) — the public-facing entry point that the browser actually talks to.
- **Python/Flask backend** (port `8081`) — an internal service the Go gateway proxies to, holding the `FLAG` environment variable.

**Go gateway (`main.go`):** the `/execute` handler unmarshals the incoming JSON into a `RequestData` struct and switches on its `Action` field. Only when `Action == "getcosmic"` does it forward the **raw request body** on to the Python service at `http://localhost:8081/execute`. Any other value is rejected before ever reaching the backend.

![Go gateway source showing the switch statement on Action for getcosmic](/assets/img/writeups/cosmic-explorer/go-gateway-source-getcosmic-switch.png)
_`main.go` — the gateway only proxies the request onward to the Python service when the JSON field `Action` (capital A, matching Go's exported struct field) equals `"getcosmic"`._

**Python backend (`app.py`):** once a request lands here, it reads the JSON body again and branches on `data['action']`. If `action == "getcosmic"` it returns a random anomaly (what the UI normally shows). But if `action == "getSecureCode"`, it returns the actual flag straight from `os.getenv("FLAG")`.

![Python backend source showing the getSecureCode branch returning the flag from environment](/assets/img/writeups/cosmic-explorer/python-backend-source-getsecurecode.png)
_`app.py` — Flask's `request.get_json()` reads the JSON as a plain Python `dict`, and looks specifically for the lowercase key `"action"`. When its value is `"getSecureCode"`, the handler returns `os.getenv("FLAG")` directly in the response._

---

## 4. The Conflict — Two Different Conditions, One Request

Putting the two services side by side exposes the actual puzzle:

```
[Client] ──> (Go Gateway)  requires: Action == "getcosmic"   ──> (Python Backend) requires: action == "getSecureCode" ──> [Flag]
```

- The **Go gateway** will only forward a request if the field `Action` (capital A) equals `"getcosmic"`.
- The **Python backend** will only return the flag if the field `action` (lowercase a) equals `"getSecureCode"`.

Satisfying both conditions with a single static JSON body looks impossible at first — until you notice that "the field named `Action`" and "the field named `action`" don't have to be the *same* key, and the two languages don't parse JSON keys the same way:

| | Go (`encoding/json`) | Python (Flask `get_json()`) |
|---|---|---|
| Key matching | Case-insensitive by default — `Action`, `action`, and `ACTION` in the JSON all map onto the same exported struct field. | Case-sensitive — `data['action']` only ever matches the literal lowercase key; a differently-cased key is simply a different, unused dictionary entry. |
| Behavior with two keys of different case | Uses whichever cased key is present to fill the struct field. | Reads only the exact-case key it's coded to look for and ignores everything else. |

That difference is the whole challenge: if the JSON body carries **both** `"action": "getSecureCode"` **and** `"Action": "getcosmic"` as separate keys, each service reads the *one it's built to read* and is completely unaware of the other:

- Go's struct-based unmarshaling picks up `"Action": "getcosmic"` → the gateway's condition is satisfied → it forwards the **entire original raw body** (including the lowercase `action` key it never itself inspects) on to Python.
- Python's dict-based `get_json()` picks up `"action": "getSecureCode"` from that same forwarded body → its condition is satisfied → it returns the flag.

---

## 5. Exploitation in Burp Repeater

With the two-key body crafted, I sent it straight to `/execute` on the Go gateway using Burp Suite Repeater:

```json
{
    "action": "getSecureCode",
    "Action": "getcosmic"
}
```

![Burp Repeater request with duplicate-case action keys and the JSON response containing the flag](/assets/img/writeups/cosmic-explorer/burp-repeater-dual-key-flag.png)
_The gateway's `Action: getcosmic` check passes, so it proxies the raw body untouched to the Python service, which in turn matches its own `action: getSecureCode` check and returns `{"flag":"HTB{COSM1C-BYP4SS}"}` — same "Captain's Log" response shape as a normal scan, just with the real flag this time instead of a random anomaly._

---

## 6. The Core Idea Behind the Challenge

- The UI's disabled-looking "Access Secure Database" option is a distraction — the actual bypass has nothing to do with clicking it, and everything to do with how two backend services independently parse the same JSON body.
- This is a classic **parser differential** bug: two systems that are supposed to agree on what a piece of data means disagree because they were built in different languages with different default parsing rules. Security checks split across multiple services/languages need to normalize on identical semantics, not just "the same field name."
- Go's `encoding/json` case-insensitive struct matching is a well-known and often-overlooked default — convenient for typo-tolerant APIs, but dangerous when a downstream service that's case-sensitive is trusting the same raw body without re-validating it independently.
- Once the source for both services was in hand, the fix was pure logic — no fuzzing, no injection, just building a single JSON object that satisfies two mutually-different case-sensitivity assumptions at once.

---

## 7. Why Reading Both Source Files Mattered

Nothing about this was discoverable from the UI or from probing the Go gateway alone — the gateway's check on its own is perfectly sane (`Action == "getcosmic"`), and the Python backend's check on its own is also perfectly sane (`action == "getSecureCode"`). The vulnerability only exists in the *seam* between the two services, and that seam is invisible unless you read both codebases side by side and compare exactly how each one deserializes JSON. This is a good reminder that in a microservice setup, auditing one service in isolation can miss the entire bug class — the interesting stuff often lives in the boundary between two components that each individually "did nothing wrong."

---

## 8. Key Takeaways

- **Parser differentials are a real bug class, not a theoretical one.** Whenever two services in a chain independently parse the same raw payload, differences in case-sensitivity, key-duplication handling, or type coercion between their languages/libraries can be exploited to make each one see a different "truth" from the same bytes.
- **A gateway that blindly forwards the raw request body is a trust boundary, not a filter.** Re-serializing only the fields you actually validated — and dropping everything else — closes this class of bug outright.
- **Read backend source for every service in the chain, not just the one facing you.** The bug here was invisible from the Go gateway's perspective and invisible from the Python backend's perspective; it only existed in how they disagreed with each other.
- **A "locked" UI option is not a security boundary.** The greyed-out "Access Secure Database" radio button never mattered — the actual condition to satisfy lived entirely in the JSON body sent to the API, regardless of what the UI let you click.
