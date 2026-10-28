---
name: bilingual-i18n-dictionary
description: Designs and maintains one English/Arabic translation dictionary for an app. Covers key strategy, ICU message format, all six Arabic plural categories, grammatical gender, safe interpolation, a missing and stale key audit, and a translator review workflow. Use when setting up i18n for a bilingual product, adding Arabic to an English app, cleaning up duplicated or drifting translation files, or when Arabic counts and plurals read wrong.
---

# Bilingual EN/AR i18n Dictionary

Keep exactly one source of truth for every user-facing string in English and Arabic, with
rules that keep both languages complete and grammatical as the app grows.

## When to use
- Starting i18n in a new product, or adding Arabic to an English one.
- Several translation files disagree, or strings are hard-coded in components.
- Arabic plurals, counts or gendered phrases are wrong in production.

## 1. Pick a key strategy (decide once, write it down)
- **Semantic keys** (`checkout.summary.total`) — stable when wording changes; best for
  large apps. Namespace by feature, not by page.
- **English-as-key** (`"Total"`) — fast to start and self-documenting; risky when the same
  English word needs different Arabic in two contexts. If you choose this, allow an
  optional context suffix (`"Open|verb"` vs `"Open|status"`).
- Never build keys dynamically (`"status." + value`) unless every possible value is
  listed in the dictionary; audits cannot see dynamic keys.
- One file per language per namespace, same key set in both; English is the reference.

## 2. Message format
Use ICU MessageFormat (or the stack's equivalent) for anything with a variable.
- Named placeholders only: `{count}`, `{name}` — never positional `%s`.
- Never concatenate fragments ("You have " + n + " items"); word order differs in Arabic.
- Keep punctuation inside the message so translators control it.

## 3. Arabic plurals — all six categories
CLDR plural categories for Arabic, by the integer `n`:
- `zero` — 0
- `one` — 1
- `two` — 2
- `few` — n mod 100 in 3–10 (3–10, 103–110, …)
- `many` — n mod 100 in 11–99 (11–99, 111–199, …)
- `other` — 100, 101, 102, 200, 1000 …, and decimals

English needs only `one` and `other`; Arabic must supply all six. Example:

    {count, plural,
      zero {لا توجد رسائل}
      one {رسالة واحدة}
      two {رسالتان}
      few {{count} رسائل}
      many {{count} رسالة}
      other {{count} رسالة}}

Note the counted noun changes form (plural after 3–10, singular after 11+), and the
`one`/`two` forms usually drop the digit entirely.

## 4. Gender
- Arabic verbs and adjectives agree with the addressee and the subject. Decide the default
  address (masculine singular is conventional; or rephrase neutrally with nouns).
- When the user's gender is known and consented, use `select`:
  `{gender, select, female {مرحبًا بكِ} male {مرحبًا بك} other {أهلًا وسهلًا}}`.
- For third-party names of unknown gender, prefer constructions that avoid agreement
  ("تمت الموافقة بواسطة {name}" rather than "وافق {name}").

## 5. Interpolation safety
- Wrap every interpolated value that can be Latin (names, emails, IDs, amounts) in a bidi
  isolate at render time so Arabic sentences do not scramble.
- Format numbers, dates and currency with the locale formatter before interpolation; do
  not pre-format in English.
- Escape or sanitise values; translations must never contain raw HTML from users.

## 6. Missing and stale key audit (run before every release)
Produce a report with four lists:
1. **Missing in AR** — keys present in EN but not AR.
2. **Orphans** — keys in either file that no code references.
3. **Unused-in-code / hard-coded** — visible strings in components not going through the
   translate function (search for literal text in JSX/templates).
4. **Stale** — keys whose English changed after the Arabic was last reviewed (track a
   source hash or "reviewed against" value per key).
Also flag: placeholder mismatch between EN and AR, missing plural categories, and Arabic
values that are identical to English (likely untranslated).

## 7. Review workflow
1. Developer adds the EN key and a description (where it appears, max length, variables).
2. Arabic is drafted (human or AI) and marked `needs-review`.
3. A native reviewer checks meaning, register, plurals and length on the real screen.
4. Status moves to `approved`; any later EN change resets it to `needs-review`.
5. CI fails on missing AR keys or placeholder mismatches; warns on `needs-review`.

## Output format
When asked to set up or audit, return: the chosen key convention, a sample namespace in
both languages, the audit tables (key | issue | EN | AR | suggested fix), and the CI rules.

## Pitfalls
- Splitting one sentence across several keys — impossible to translate correctly.
- Reusing a key because the English matches ("Close" a dialog vs "Closed" status).
- Supplying only `one`/`other` in Arabic; it compiles but reads wrong for 2, 3–10 and 11+.
- Storing translations inside the database and the code; pick one home.
