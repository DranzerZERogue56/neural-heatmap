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

Level chapter per class. Order is pedagogical (each builds on the last).

1. **Classic / In-band — Auth Bypass** (`' OR '1'='1`)
2. **Error-Based** (extract data via forced DB errors)
3. **UNION-Based** (column count, data types, cross-table extraction)
4. **Boolean-Based Blind** (true/false inference)
5. **Time-Based Blind** (`SLEEP()` / `WAITFOR DELAY`)
6. **Stacked Queries** (`;` chaining, where the DB allows it)
7. **Second-Order** (payload stored now, fires later on a different query)
8. **Out-of-Band** (DNS / HTTP exfiltration — simulated)

> **Open question:** include NoSQL injection as a bonus chapter, or keep strictly SQL? *(edit here)*

---

## 4. Game Loop (D&D-style)

Each encounter = one vulnerable endpoint on the AWS server graph (reuse existing node model).

**Turn structure:**
1. **Recon phase** — player inspects the endpoint: request shape, visible response, error verbosity. Costs 0–1 action.
2. **Craft phase** — player composes a payload from SQL "spell components" (tokens / snippets they've unlocked).
3. **Cast phase** — payload is submitted; the simulated DB responds. Success/partial/fail is narrated.
4. **Loot phase** — on success: XP, unlocks (new tokens, new endpoints), and a **defense card** explaining the fix.

**Stats to track per run:** HP (detections before you're IP-banned), MP (rate-limit budget), XP, payload inventory, discovered schema.

**Dice / randomness:** WAF checks and blind-inference oracles roll against player "Stealth" and "Insight" stats — keeps it D&D-flavored without being unfair (failures always teach something).

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

> **Open question:** how many levels per chapter before sandbox unlocks — 3? 4? *(edit here)*

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

> **Open question:** keep it fully text/parchment, or allow pixel-art sprites for nodes/bosses? *(edit here)*

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

*Edit this file to redirect the build. Anything marked "Open question" is a nudge for your call.*
