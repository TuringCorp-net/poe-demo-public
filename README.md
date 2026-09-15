# poe-demo-public

Data for the **Decider** canvas app on Poe. One file, served over a CDN, so the app can show
worked examples **without sending a message, without charging anyone and without a redeploy**.

- **Canvas app**: <https://poe.com/TuringCorp-Decider>
- **Loaded by the app as**: `https://cdn.jsdelivr.net/gh/TuringCorp-net/poe-demo-public@main/examples.json`

## Why this repo exists

A canvas app is a single-file HTML page with no server and no database. Poe's canvas CSP allows
resources from a fixed list of trusted origins, and `cdn.jsdelivr.net` is on it, so the app can read
a JSON file from here directly. That means the examples can be **updated without touching the app
and without asking the user for anything**.

## Hard rules

1. **JSON only.** jsDelivr serves `.json` and `.js` from GitHub and returns a `301` to
   `raw.githubusercontent.com` for everything else (`.md`, `.zip`, …). `raw.githubusercontent.com`
   is *not* on Poe's trusted-origin list, so the browser blocks the redirect and the app sees a
   failed fetch. Never point the app at a `.md` file.
2. **Real recorded runs only.** Every entry is the verbatim output of an actual Decider run
   (task, both options, the pick, the confidence, the reason). No hand-written verdicts, no
   "example" numbers that were never produced.
3. **No internal fields.** No model or channel names, no providers, no costs, no internal IDs —
   this file is public and is downloaded by the browser.
4. **Nothing personal.** Only self-authored or explicitly cleared tasks and options.

## Format

```json
{
  "schema": 1,
  "updated": "YYYY-MM-DD",
  "examples": [
    {
      "id": "stable-slug",
      "domain": "Research",
      "recorded": "YYYY-MM-DD",
      "task": "the question both options answer",
      "option_A": "first candidate",
      "option_B": "second candidate",
      "betterOption": "option_A",
      "confidence": "84.2%",
      "reason": "the reason the run produced"
    }
  ]
}
```

`domain` is the one category field: it is what a user picks by ("I have a research question, show me
one of those"), so it has to read like a kind of question, not like an internal label. Keep the set
small and stable, and keep them user-facing: `Research`, `Work & career`, `Personal life`,
`Relationships` (extend only when there is a real example behind a new one). The app walks the list
in order, so extra entries in the same domain simply come up as the user keeps clicking.

`betterOption` is exactly `option_A` or `option_B` — the same form the model contract uses.
`confidence` is the string the run reported, one decimal.

## Updating

```bash
# edit examples.json, then
git commit -am "examples: add ..." && git push

# jsDelivr caches @main for ~12h; purge immediately after a content change
curl https://purge.jsdelivr.net/gh/TuringCorp-net/poe-demo-public@main/examples.json
```

Verify what the app will actually see:

```bash
curl -sI https://cdn.jsdelivr.net/gh/TuringCorp-net/poe-demo-public@main/examples.json
# expect: 200, content-type: application/json, access-control-allow-origin: *
```
