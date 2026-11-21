---
name: app-audit-fix
description: Two-phase audit-and-fix skill for apps (single-file HTML apps, games, POS/accounting apps, Flutter/SQLite apps, any codebase). Phase 1 hunts for bugs, security holes, data-loss risks and logic errors; Phase 2 independently tries to DISPROVE every finding; only verified findings get fixed, with minimal, safe, reversible edits. Use whenever the user mentions fixing problems, bugs, errors, "حل مشاكل", audit, review, checking an app, or hands over a finished app (even one built by a friend or another AI) and wants it to work without breaking. Use it even if the user doesn't say "audit".
---

# App Audit & Fix (Two-Agent Method)

Core rule: **the one who finds a problem is never the one who confirms it.**
Finder = Agent 1. Verifier = Agent 2. Fixer = Agent 3. Switch roles explicitly and
do not carry assumptions from one role to the next.

Prime directive: **never break a working app.** A false alarm that gets "fixed" is
worse than a real bug left alone. When unsure, report; do not edit.

Talk to the user in the language they write in (Iraqi Arabic is fine and preferred
if they write it). Keep reports short, scannable, no heavy jargon.

---

## Step 0 — Understand the app first (mandatory, before any hunting)

1. Read the whole file/project. For large files, read in chunks until fully covered.
2. Write a 5-line map: what the app does, main screens/modules, where data lives
   (localStorage / IndexedDB / SQLite / server), external libs, entry points.
3. List the **critical flows** (e.g. save invoice, record payment, buy/sell, login,
   export, backup/restore, game start/save). These are what must never break.
4. Note constraints: single file? offline? RTL/Arabic? tablet/phone? Keep them.
5. If the user gave a bug description, treat it as a lead, not as the full scope.

Do not edit anything in Step 0.

---

## Phase 1 — Agent 1: FIND problems (hunter)

Be suspicious and thorough. For each category scan the actual code, not assumptions.

**A. Crashes & runtime errors**
- Undefined variables/functions, wrong IDs (`getElementById` of missing element),
  null/undefined access, typos, missing `await`, unhandled promise rejections.
- Syntax issues, unclosed tags/braces, duplicate function names, duplicate IDs.
- Event handlers bound twice or never bound; handlers on elements recreated later.

**B. Data integrity & loss**
- Saves that overwrite instead of merge; no validation before write.
- Storage quota / parse errors (`JSON.parse` on bad data) with no fallback.
- Backup/restore/import that can wipe existing data. IDs that can collide.
- Deleting records that other records still reference.

**C. Logic & money/number errors** (critical for POS, debts, accounting)
- Floating-point rounding on money; totals not matching line items.
- Wrong sign (debt vs payment), wrong order of operations, double counting.
- Currency/number formatting, Arabic-Indic digits breaking `parseFloat`.
- Date/timezone mistakes; sorting strings as numbers.
- Stock going negative; discounts > total; empty-cart actions.

**D. Security** (if the app has users, a server, or sensitive data)
- Hard-coded secrets/keys/passwords in the code.
- Missing access/permission checks (UI-only hiding of admin features).
- Unsafe input: `innerHTML`/`eval`/string-built SQL with user input (XSS / injection).
- Sensitive data stored or logged in plain text. Over-permissive CORS/rules.

**E. UI/UX breakage**
- Layout broken on phone/tablet sizes, RTL issues, overflow, unreachable buttons.
- Missing loading/empty/error states; buttons that do nothing; modals that can't close.
- Performance traps: loops creating thousands of DOM nodes, per-frame allocations
  (games), memory leaks (listeners/intervals never cleared).

**F. Game-specific** (if a game)
- Game loop errors, delta-time bugs, collision edge cases, input stuck states,
  audio/context not resumed, save/load mismatches, mobile touch controls.

**Output of Phase 1:** a numbered list. Every finding MUST have:
- `ID` (F1, F2…), `Severity` (HIGH / MEDIUM / LOW)
- `Where` (function name + line or unique code snippet)
- `What` (one sentence), `Why it matters` (what the user would experience)
- `Evidence` (the exact code that proves it) and `How to trigger` (steps/inputs)

No evidence = not a finding. Drop it.

---

## Phase 2 — Agent 2: VERIFY (skeptic — try to prove each finding WRONG)

Forget that you wrote the list. For each finding, actively look for reasons it is
NOT a real problem:

- Is the code path actually reachable? Is it guarded elsewhere (earlier check,
  try/catch, default value, caller already validates)?
- Does the trigger steps really produce the failure? Trace it line by line with
  concrete values. If a runtime is available, **run it** (node, headless test,
  a small script) instead of guessing.
- Is it intended behavior (the app's design), not a bug?
- Is the severity inflated?

Give each finding one verdict:
- **CONFIRMED** — reproduced or traced with concrete values; failure is real.
- **REJECTED** — disproved; state why in one line.
- **UNCERTAIN** — cannot prove either way; state what info is missing.

Only CONFIRMED items move on. UNCERTAIN items are reported to the user, never fixed
silently.

---

## Phase 3 — Agent 3: FIX (surgeon)

Work in priority order: HIGH → MEDIUM → LOW. For each CONFIRMED finding:

1. **Smallest possible change** that removes the root cause. No refactors, no
   renaming, no "while I'm here" cleanups, no style changes.
2. **Preserve everything else**: same UI, same data format, same file structure.
   If a fix changes stored data format, add a migration so old data still loads.
3. Use targeted edits (`str_replace`), not whole-file rewrites, unless the file is
   tiny. Keep a copy of the original before editing.
4. After each fix, re-trace the critical flows from Step 0 that touch that code.
5. Never remove a feature to "fix" a bug. Never add new dependencies/libraries
   unless unavoidable (and then say so).
6. If a fix is risky or large, stop and ask the user before applying it.

---

## Phase 4 — Regression check (before saying "done")

- Re-read the final file once, end to end, looking only for syntax errors and
  broken references caused by the edits.
- Run what can be run: syntax check (`node --check` for JS extracted from HTML),
  lint if available, any existing tests, a quick smoke run of critical flows.
- Confirm each critical flow from Step 0 still works. Mark each ✅ or ❌.
- If anything is ❌, fix or revert that edit. Do not hand over a broken file.

---

## Final report format (short, in the user's language)

1. **الخلاصة**: one sentence — app state before/after.
2. **تم إصلاحه** (confirmed + fixed): `ID – problem – what was changed`.
3. **ما تم رفضه**: findings that turned out to be false alarms (one line each) —
   shows the verification really happened.
4. **يحتاج قرارك**: UNCERTAIN or risky items, with a plain explanation and a
   recommendation.
5. **فحص أخير**: the critical-flow checklist with ✅/❌.
6. **الملف النهائي**: deliver the fixed file(s) and keep the original name unless
   told otherwise.

---

## Self-interrogation before finishing (answer each honestly, yes/no)

- Did I read the entire app, not just the part that looked suspicious?
- Does every reported problem have real evidence and a trigger?
- Did I genuinely try to disprove each finding as Agent 2?
- Did any fix touch more than needed? Did it change the UI or data format?
- Do old saved data and existing features still work after my edits?
- Did I test or trace every critical flow after the last edit?
- Did I tell the user clearly what I did NOT fix and why?

If any answer is "no", go back and fix that gap before delivering.

---

## Hard rules

- Never claim something was tested if it was only read. Say "traced", not "tested".
- Never invent a bug to look useful. An honest "I found 2 real problems" beats 20 guesses.
- Never delete user data or touch backups while testing.
- Never expose or print secrets found in the code; just flag where they are.
- If the app is far too big to fully review in one pass, say so, split it into parts,
  and finish each part completely before moving on.
