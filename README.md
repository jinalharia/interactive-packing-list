# 🧳 Holiday packing list

An interactive packing checklist in **one self-contained file**: [`index.html`](index.html).
There's no install, no build step and no external requests. It works offline, so you can email it, AirDrop it, send it on WhatsApp, or host it on GitHub Pages.

## Using it
- **Holiday type:** beach / pool, city break, ski / winter, business, camping, road trip. You can pick more than one, e.g. beach + business.
- **Who's coming:** baby, kids. **Travel:** flying, driving, abroad. These add or remove items. For example, nappies only show with *Baby* and passports only with *Abroad*.
- **Nights:** sets quantity hints on items like underwear, socks and nappies.
- **Ticking:** tick items as you pack. Ticks are saved in that browser on that device.
- **Your own items:** use "Add your own…" in any group to add items. They're also saved on that device only.
- **Buttons:**
  - **Hide packed** hides items you've already ticked.
  - **Collapse/Expand all** folds or opens every section.
  - **Print** gives a clean paper checklist.
  - **Untick all** clears your ticks.
- **Copy link** gives you a URL that opens with the same trip settings, e.g. `index.html#trip=beach&with=baby,kids&travel=flying,abroad&nights=7`. Ticks are not included.

## Editing the list
- Open `index.html` and edit the clearly marked **data block** at the top of the `<script>`. Each item is one line:

  ```js
  ["Sun cream", "beach city ski camping road", "", "", { note: "factor 50" }],
  //  name      holiday types ("*" = all)    who  travel  options
  ```

  - **who:** `""` for everyone, or any of `baby kids`.
  - **travel:** `""` for always, or any of `flying driving abroad`.
  - **options (optional):**
    - `note` shows a small note under the item.
    - `q: [perNight, extra]` shows a quantity hint of `ceil(nights × perNight + extra)`.
- Items below a `// added` comment are extras that weren't in the original list. Delete any you don't want.
- Item IDs come from section, group and name. Renaming an item resets its tick on devices that had it ticked.

## Hosting on GitHub Pages
In the repo, go to **Settings → Pages → Build and deployment**. Choose **Deploy from a branch**, select `main` and `/ (root)`, then save. The list will be at `https://<user>.github.io/interactive-packing-list/`.
