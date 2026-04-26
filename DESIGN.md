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

## 11. Build Status — All Chapters + Boss Shipped (last update 2026-04-26)

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
| **Chapter 4: Boolean-Based Blind** | ✅ | 5 levels (Coin Flip, Single Letter, Binary Search, Table Name, Hash One Bit) + sandbox. Binary-search bisection taught explicitly. |
| **Chapter 5: Time-Based Blind** | ✅ | 5 levels (Heartbeat, Conditional Wait, Slow Letter, Heavy Computation, Patient Extraction) + sandbox covering MySQL/Postgres/MSSQL dialects. |
| **Chapter 6: Stacked Queries** | ✅ | 5 levels (Two for One, New Admin, Burning the Logs, Privilege Escalation, Clean Heist) + sandbox. Driver `multipleStatements` flag taught. |
| **Chapter 7: Second-Order** | ✅ | 5 levels (Sown Field, Detonation, Newsletter, Slow Burn, Sleeper Agent) + sandbox. Plant-then-detonate pattern. |
| **Chapter 8: Out-of-Band** | ✅ | 5 levels (Whisper Channel, DNS Smuggling, HTTP Beacon, File Drop, Quiet Exfil) + sandbox. UNC + DNS + http extension + COPY ... PROGRAM. |
| **Chapter 9: NoSQL (Bonus)** | ✅ | 5 levels (Operator, Lazy Probe, Server-Side Script, Aggregation, Type Confusion) + sandbox. Mongo `$ne`/`$regex`/`$where`/`$lookup`/`$set`. |
| **Chapter 10: Boss — The Archon's Vault** | ✅ | Single-node final encounter requiring ≥3 chained disciplines (UNION + blind + time + OOB + stacked + error-based). 50 XP, victory copy, Total Defense card. |
| Map: single-node chapter support | ✅ | `nodeCount` chapter property; boss renders one ankh node ☥ labeled "THE ARCHON". |
| Nav link rename | ✅ | `index.html`, `neural-heatmap.html`, `tutorial-aws.html`, `tutorial-neural.html` now link to "SQLi Dungeon". |

### 🔨 Optional polish remaining

| Area | Priority | Notes |
|---|---|---|
| Sprite art | Low | User approved earlier. Currently each chapter uses a unicode glyph + color. Real SVG/pixel sprites per vuln class (spider/ghost/dragon/etc.) on nodes are a future polish. Keep them inline so the file stays single-deploy. |
| Asset extraction | Low | `aws-attack-surface.html` is now ~2200 lines. If it grows past ~3000 (e.g., adding sprites/animations), pull `CHAPTERS` into `assets/sqli-levels.js` and `<script src>` it. Not needed yet. |
| Tutorial page for the dungeon | Low | `tutorial-sqli.html` — explainer/lore page intro to game mechanics. Currently the prologue copy in the game does this minimally. |
| Audio | Low | Optional. Key-press click on Cast, low chime on Defense Card open, gold flourish on boss kill. |
| Difficulty tiers | Low | Add per-level DC modifiers (Stealth/Insight DC scales with chapter). The hooks exist; values are flat. |
| Leaderboard for sandbox | Stretch | Requires backend or third-party service. Out of scope for static-page deploy. |
| Stronger payload validators | Polish | Some `check()` regexes are permissive (reward creative solutions). Tighten if false-positives become a teaching problem. |

### 🧭 How to Extend (now that the core game ships end-to-end)

1. **Run it**: open `aws-attack-surface.html` in a browser. No build step. localStorage persists progress; clear it via DevTools `Application → Local Storage → sqli-dungeon.save.v1` for a fresh run.
2. **Add a chapter**: insert a new entry in the `CHAPTERS` object. Pattern: `{ id, name, class, intro, levels: { 1..5 }, sandbox(seed) }`. Use Chapters 1–9 as templates.
3. **Add a level**: every level needs `{ id, name, endpoint, query, field, goal, reconNotes[], hints[], check(payload)→{ok, narrate}, unlockTokens[], defense: { title, body, code, tip } }`. `check()` returns `{ ok: true|'partial'|false, narrate }`.
4. **Add a special boss-style chapter**: set `nodeCount: 1` on the chapter object and `isBoss: true` on the level. Map renderer auto-handles single-node layout and the ☥ glyph.
5. **Tokens**: append to `KNOWN_TOKENS`, then reference in `unlockTokens`. Locked until a level unlocks them.
6. **Commit cadence**: one chapter per commit. Existing pattern: `"Chapter N: Topic (5 levels + sandbox)"`.
7. **DO NOT** rewrite scaffold, theme, or engine without reason — they are settled.

### 🐛 Known limitations

- The `mulberry32` seeded RNG is deterministic per seed — re-rolling the sandbox generates a new seed, but the same seed will always produce the same encounter. This is intentional so students can share seeds.
- `check()` functions use regex against the payload string — they approve any payload that matches the shape, not just the exact "canonical" payload. That's fine for learning (rewards creative solutions) but a determined student can pass with a borderline match. Tighten regexes if that matters.
- No audio / no animations beyond CSS transitions. Scope choice.
- Cheatsheet definitions are hardcoded in `showCheatsheet()`. If the token list grows, move definitions next to the tokens themselves.

---

*Edit this file to redirect the build. Anything marked "Open question" is a nudge for your call.*
