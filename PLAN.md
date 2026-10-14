# Plan: Self-contained interactive holiday packing list (single HTML file)

## Context
The user has a family packing list (markdown: Main luggage → Clothes / Electronics / Other / Toiletries / Kids, Hand luggage, Morning to-do). They want an interactive HTML packing list where you choose a **holiday type** and the list changes to match. Every item from the markdown is included, plus sensible additions.

**Scope (simplified by user):**
- No submission or suggestion flow, no GitHub Actions, no backend.
- **Everything lives in one self-contained `index.html`**: inline CSS, inline JS and the item data embedded as a JS object.
- No external requests (no CDN, no web fonts), so the file can be emailed, AirDropped or WhatsApped and opened offline.
- The same file can also be served as-is from GitHub Pages.

Repo `jinalharia/interactive-packing-list` is empty (no commits). Branch: `claude/eloquent-brahmagupta-avddyt`.

## Decisions confirmed with user
- Trip types: **beach/pool, city break, ski/winter, business, camping, road trip** (multi-select, e.g. beach + business).
- Modifiers: with baby, with kids, flying, driving, abroad.
- Tick state: **per device (localStorage)**, plus a share-by-URL link for selections.

---

## 1. File layout
```
index.html   ← the whole app (HTML + <style> + <script> with DATA + logic)
README.md    ← what it is, how to open/share it, how to edit the item list, how to enable Pages
```

## 2. Data model (embedded in `index.html` as `const DATA = {...}`)
```js
const DATA = {
  tripTypes: [{id:"beach",label:"Beach / pool"}, {id:"city",…}, {id:"ski",…}, {id:"business",…}, {id:"camping",…}, {id:"road-trip",…}],
  modifiers: [{id:"baby",label:"With baby"}, {id:"kids",…}, {id:"flying",…}, {id:"driving",…}, {id:"abroad",…}],
  sections: [
    {id:"main", name:"Main luggage", groups:["Clothes","Electronics","Other","Toiletries","Kids","Documents & money","Health"]},
    {id:"hand", name:"Hand luggage"},
    {id:"morning", name:"Morning to-do"}
  ],
  items: [
    {id:"sun-cream", name:"Sun cream", section:"main", group:"Toiletries", trips:["beach","camping","road-trip"], mods:[], qty:null},
    …
  ]
};
```
- `trips: ["*"]` means the item always shows (e.g. toothbrush).
- `mods` means the item only shows when every listed modifier is on. For example, nappies need `baby` and passports need `abroad` (in hand luggage).
- `qty` is an optional formula string: `"n+1"` (nights + 1), `"n"` or a fixed number. It is shown as a "×8" hint.
- Composite lines in the md are split into separate items. For example, "suit, shirts, business cards, celebrations, smart shoes" becomes business-tagged items. "floatie / pool balls (rugby, football, skimmer)" stays one item with a note.
- The data block sits at the top of the `<script>` with a comment, so the owner can hand-edit the list without touching the logic.

## 3. Page behaviour (vanilla JS, no build step)
- **Trip setup bar:**
  - Holiday-type chips (multi-select).
  - Modifier toggles.
  - A nights stepper that drives the quantity hints.
- **Filtering:** an item shows when its `trips` match a selected type (or include `*`) and all of its `mods` are on.
- **List view:**
  - Sections and groups as in the md, collapsible.
  - Each section shows a "x / y packed" count and a progress bar, with an overall progress bar at the top.
- **Checkboxes:**
  - Ticks are keyed by item id and saved to `localStorage`.
  - A "hide packed" toggle.
  - "Reset ticks" asks for confirmation before clearing.
- **Personal extras (on-device only):** an optional "+ add item" field per group. Extras are saved to `localStorage` only and never shared.
- **Share link:** the "Copy link" button puts trip, mods and nights in the URL hash (`#trip=beach,business&mods=baby,flying&nights=7`). Opening the link restores those selections, but not the ticks.
- **Look and feel:**
  - Mobile-first layout with large tap targets.
  - Light and dark theme via `prefers-color-scheme`.
  - `@media print` styles for a clean paper checklist.
- **Robustness:** all `localStorage` access is wrapped in try/catch, so the page still works if storage is blocked (e.g. some browsers on `file://`).

## 4. Items: all originals + proposed additions
All md items are kept, in their original sections and groups. Proposed additions (trim during the build if desired):
- **Documents/money (new group):**
  - Travel insurance, GHIC card, driving licence
  - Boarding passes, booking confirmations, car hire docs
  - Foreign currency, travel card, passport copies
- **Health (new group):**
  - First-aid kit, plasters, antihistamine
  - Rehydration sachets, Imodium, prescription meds, thermometer
  - Calprofen, after-sun, SPF lip balm
- **Electronics:** phone, laptop + charger (business), travel extension lead, car phone mount, camera.
- **Clothes:** light jacket/cardigan, rain jacket, hoodie for the flight.
- **Kids/baby:**
  - Travel cot + sheets, pushchair, car seat, baby carrier
  - Sippy cup, milk/formula, bottles + steriliser, UV sun suit
  - Comforter, stickers/colouring
- **Beach:** beach towels, bucket & spade, pop-up beach tent, cool bag.
- **Ski:**
  - Ski jacket / salopettes, thermals, gloves, ski socks
  - Ski goggles, helmet, neck warmer, lift pass
- **City:** comfy walking shoes, day backpack, umbrella.
- **Camping:** tent, sleeping bags, head torch, camping stove, mallet.
- **Road trip:** car snacks, travel sickness bands, sunshades for car windows.
- **Hand luggage:** clear liquids bag, pen, refillable water bottle.
- **Morning to-do:**
  - Online check-in, check flight status
  - Empty the fridge, water the plants, unplug appliances
  - Timer lights, out-of-office

## 5. Verification
- **Coverage:** a one-off Node check, run from the scratchpad and not committed, extracts `DATA` from `index.html` and confirms every bullet in the source md maps to at least one item.
- **Self-contained:** grep confirms there are no `src=`/`href=` references to external URLs.
- **Behaviour (Playwright + pre-installed Chromium):** open `index.html` over `file://` and over a local static server, then:
  - Toggle trip types and modifiers and confirm the item counts change as expected (e.g. nappies appear only with "baby").
  - Tick items, reload and confirm the ticks persist.
  - Confirm a share-link hash restores the selections.
  - Confirm the print view renders.
  - Take screenshots on a mobile viewport in light and dark mode.
- **Delivery:** commit `index.html` + `README.md` to `claude/eloquent-brahmagupta-avddyt` and push.
