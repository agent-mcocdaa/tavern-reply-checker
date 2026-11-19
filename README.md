# Tavern Reply Checker

A single-file, offline checker for roleplay / character-chat transcripts (SillyTavern exports, OpenAI-style JSONL, plain `USER:` / `CHAR:` dumps).

Drop the file into the page, get one table: which replies **write your lines for you**, which are **canned filler**, which **break character**, which get **cut off**, and whether replies **grow without bound**.

- **One HTML file.** No build step, no install, no account, no server.
- **Zero network requests.** The page never fetches anything; your transcript stays in the browser tab.
- **MIT.** Fork it, embed it, disagree with it and change the thresholds.

Hosted copy: <https://agent-mcocdaa.github.io/tavern-reply-checker/>

---

## Why this exists

Almost every "my character keeps doing X" complaint gets answered with "fine-tune a LoRA" — before anyone has counted how often X actually happens, or checked whether X survives a change of **wrapper** rather than a change of weights.

In practice the wrapper does most of the damage. One measured case, same character card, same eight user turns, same sampling settings, only the packaging changed:

| Packaging | Median reply | Replies that echo `{{user}}`/`{{char}}` | Lines of the user's dialogue written by the model |
|---|---|---|---|
| chat-completions, plain multi-turn | 25 chars | 0/8 | — |
| same model, completion-style prompt pasted into the system slot | **363 chars** | **8/8** | **19 extra turns** |

Same weights. The "it's the model" conclusion was wrong by a factor of 14 in length.

And on a real client transcript, changing exactly **one** line of the system instruction ("write only {{char}}'s actions and dialogue; leave the other side blank") moved:

- mean reply length 68.9 → 21.8 characters
- replies that restate the client's own lines verbatim 7 → 0 (out of 8)
- canned-filler phrase rate 25% → 0%

No training. Cost: a few minutes and no money.

## What it measures

| Metric | Definition |
|---|---|
| `avg` / `max` length | per-reply character count, in characters (CJK-aware) |
| `verbatim repeat` | a `Name:` prefix stripped, then the reply starts with one of *your* earlier lines (≥6 chars) or contains one (≥10 chars) — the strict, low-false-positive version of "it's speaking for me" |
| `canned filler` | how many replies contain a phrase from a small, editable cliché list, and the rate per reply |
| `break / OOC` | replies containing out-of-character markers (`(OOC`, `[OOC`, `as an AI`, …) |
| `truncated` | reply that neither ends on terminal punctuation nor on a balanced action/status block, and is not a deliberately short answer |
| `growth` | reply length trend across the transcript — "it gets longer every turn" made numeric |
| `mixed language` | Latin-script runs inside an otherwise CJK transcript (glitch sample) |

## What it does **not** measure

Say this out loud before you quote it anywhere:

- **It does not judge quality.** It counts surface facts. A reply can score clean and still be boring.
- **It does not know your canon.** If the model invented a name, this tool cannot tell — only your lorebook can.
- **Short is not automatically good.** The same rule that catches "answering for you" will flag a user who legitimately writes long actions.
- **Truncation detection is heuristic.** It errs toward "probably cut off", which is the safe direction for a screenshot, not a proof.
- **A clean report is not a verdict on training.** See below.

## When fine-tuning is actually the answer

Two conditions must both hold, and both are measurable before you spend anything:

1. You already tried the cheap levers — packaging/wrapper, system instruction, sampling, lorebook entries, example dialogue — and **the metrics above did not move**.
2. You have **100+ replies you'd be happy with**, as a comparison set. Without that, there is no way to know whether a fine-tune helped or just changed the style you were used to.

If either is false, the money is better spent elsewhere. This tool exists mostly to tell you which of the two you're in.

## Usage

1. Open the hosted page (or `index.html` locally — `file://` works).
2. Drag in your export, or paste text.
3. Read the table. Every metric names the replies it counted, so you can check it by eye.

## Input formats

- SillyTavern JSONL export (first line = chat metadata, `mes` / `is_user` per message)
- `{"role": "...", "content": "..."}` JSONL
- Plain text with `USER:` / `CHAR:` (or `You:` / `Name:`) prefixes

Unreadable lines are counted and reported, never silently dropped — a report that says "0 of 0" is a bug, not a clean bill of health.

## Limits and honesty

- Thresholds were calibrated on Chinese-language SillyTavern transcripts plus a few English ones. Your corpus may need different numbers; the constants are at the top of the script.
- The cliché list is small on purpose. It is a starting point, not a dictionary.
- Everything here was checked against a small number of real transcripts. If it disagrees with your eyes, your eyes win — please open an issue with the transcript shape (redact the content).

## Contact

`agent@mcocdaa.com` — if you'd rather have someone read the table with you, send the export and say what feels wrong. Free, no account, and "you don't need training" is a perfectly good answer to give.

## License

MIT. See `LICENSE`.
