# Created Training

Training log for [Created](https://www.created.center), William McNeely’s 24/7 strength gym in Ozark, MO. The site is a static PWA (`index.html`, `manifest.json`, `sw.js`, `icon.svg`) published with GitHub Pages.

Each floor machine can have a printed Machine Guide placard. The QR code on that placard opens this app on that machine: the guide, the video demo, and a way into today’s set log.

## Open a machine

Placard QR codes should use the hash form. It stays on one cached page, including offline:

```text
https://wmmcneely.github.io/created-training/#/machine/L001
```

`L001` is the floor ID. Same pattern for every machine: `#/machine/U001`, `#/machine/T001`, and so on.

This also works. The app rewrites it to the hash form, then to the canonical floor ID:

```text
https://wmmcneely.github.io/created-training/?m=L001
```

Older placards that still say L1, L2, or L3 keep working. `?m=L1` and `#/machine/L1` open L001 (L2 opens L002, L3 opens L003), and the address bar updates to the new ID.

`https://wmmcneely.github.io/created-training/index.html#/machine/L001` is the same page. Prefer the shorter URL without `index.html` on the placard.

The **Machines** link in the header opens the whole sheet at `#/machines`. **Log a set** on a guide returns to Today and keeps a one-line reminder of which machine you came from. Sets are still stored on today’s session, the same way as before.

## Machine catalog

`machines.json` is the fleet. It was imported from the Created Training Center Equipment sheet. One JSON object per row, in sheet order, keyed by floor ID.

Every row has a floor ID: one section letter plus a three-digit number, numbered in sheet row order inside that section. `L001` is Lower Extremity row 1. The old sheet codes L1, L2, and L3 are the same machines as L001, L002, and L003. `aliases` at the top of `machines.json` maps those legacy codes, and each of those three records also has `legacyId`.

| Letter | Sheet tab | IDs |
| --- | --- | --- |
| L | Lower Extremity | L001–L026 |
| U | Upper Extremity | U001–U021 |
| T | Trunk Equipment | T001–T009 |
| B | Total Body Equipment | B001–B021 |
| F | Free Weight Equipment | F001–F006 |
| C | Cardio Equipment | C001–C009 |
| R | Restorative Equipment | R001–R004 |
| Z | Business Equipment | Z001–Z015 |

`machine-ids.csv` is the same list as `id,section,equipment`, for pasting the new ID Number back into the Google Sheet.

### Record fields

| Field | Meaning |
| --- | --- |
| `id` | Same as the object key. Floor ID, such as `L001`. |
| `legacyId` | Previous sheet ID when one existed (`L1` on L001). Omitted otherwise. |
| `flags` | `name-only`, `duplicate-name` when those apply. |
| `section` | Sheet tab name. |
| `sheetRow` | Row on that tab. |
| `equipment` | Equipment column. |
| `createdName` | Created name. |
| `nickname` | Nick name. |
| `type` | Machine, bench, free weight, and so on. |
| `brand` | Brand. |
| `muscle` | Muscle column. |
| `movement` | Movement pattern from the sheet (not the placard cue). |
| `simplyPut` | The sheet’s “Simply Put” line. |
| `status` | Sheet status, such as `Active (Not Filmed)` or `Paused`. |
| `paused` | `true` when status starts with Paused. |
| `video` | Demo URL from the sheet’s File column. |
| `guide` | Placard copy, or `null` until a placard is encoded. |

Empty strings mean the sheet cell was blank. Extra sheet columns (`footprint`, `amount`, `integration`, `details`, `method`, `notes`) are kept on the record too. `importNotes` at the top of the file records import caveats.

Rows that were only a name on the sheet are included and flagged `name-only`. Three identical “Nautilus Dumbbell Rack” lines are separate records flagged `duplicate-name`.

## Placard guide

`guide` is the member-facing card. L001 is the filled-in example (Nautilus 2ST Hip Extension, the Super Glute Machine, legacy placard ID L1). Sheet columns stay on the record. The sentences from the printed card live here.

```json
"guide": {
  "brandLine": "NAUTILUS 2ST",
  "title": "HIP EXTENSION MACHINE",
  "aka": "Super Glute Machine",
  "primary": "Glutes - Buns & Hamstrings",
  "secondary": "Hamstrings, Lower back (erector spinae)",
  "setup": "Pad at 11 o'clock. Hips at red dot. Belt on.",
  "movement": "Drive legs straight. Pause. Slow return.",
  "caution": "Control it. Challenge yourself. Smooth reps, no momentum.",
  "startImage": "",
  "finishImage": ""
}
```

`movement` inside `guide` is the placard cue (“Drive legs straight…”). `movement` on the record is the sheet pattern (“Hip Extension”).

`startImage` and `finishImage` are optional paths to the start and finish photos, relative to the site root, with no leading slash: `guides/L001-start.jpg`. Leave them as `""` until the files exist. The page hides the photo row when both are empty.

Members mostly read the name, the nickname, and the muscles. Keep new placard copy that short.

## Add a machine or placard

1. Take the next number in that section. Lower currently ends at L026, so the next lower machine is L027. Upper ends at U021, trunk at T009, total body at B021, free weight at F006, cardio at C009, restorative at R004, business at Z015.
2. In `machines.json`, add an object whose key and `id` are that floor ID. Keep sheet order within the section.
3. Add the same row to `machine-ids.csv` and copy the ID into the sheet’s ID Number column.
4. Fill `equipment`, `createdName`, `nickname`, `type`, `brand`, `muscle`, `movement`, `simplyPut`, `status`, and `video`. Set `paused` to `true` only when status is Paused.
5. When the printed placard exists, set `guide` using the L001 shape. Copy the placard, not a paraphrase. Print the new ID on the placard (`#/machine/L027`).
6. If you add start/finish images, save them under `guides/`, point `startImage` / `finishImage` at them, add those paths to the `SHELL` list in `sw.js`, and bump the cache name `C` (`ct-v6`, `ct-v7`, …) so phones pick up the new files.
7. Changing `machines.json` alone does not require a new image path, but still bump `C` in `sw.js` when you want installed copies to drop the previous precache promptly. The service worker is network-first, so an open phone will also pick up the new JSON on the next load while online.

## Brand

Black background. Gold `#B38234` stripe above rust `#9F5F2C` stripe. Gold stays on top. The site name on the guide is written `www.created.center`.

## Offline

`sw.js` precaches `./`, `index.html`, `manifest.json`, `icon.svg`, and `machines.json` under cache name `ct-v6`. A placard hash URL (`#/machine/L001`) is still just `index.html`, so the cached shell can open it. `?m=L001` and the legacy `?m=L1` are matched with `ignoreSearch` when the network is down.
