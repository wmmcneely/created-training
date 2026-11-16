# Created Training

Training log for [Created](https://www.created.center), William McNeely’s 24/7 strength gym in Ozark, MO. The site is a static PWA (`index.html`, `manifest.json`, `sw.js`, `icon.svg`) published with GitHub Pages.

Each floor machine can have a printed Machine Guide placard. The QR code on that placard opens this app on that machine: the guide, the video demo, and a way into today’s set log.

## Open a machine

Placard QR codes should use the hash form. It stays on one cached page, including offline:

```text
https://wmmcneely.github.io/created-training/#/machine/L1
```

`L1` is the floor ID. Same pattern for any assigned ID: `#/machine/L2`, `#/machine/U1`, and so on.

This also works, and the app rewrites it to the hash form:

```text
https://wmmcneely.github.io/created-training/?m=L1
```

`https://wmmcneely.github.io/created-training/index.html#/machine/L1` is the same page. Prefer the shorter URL without `index.html` on the placard.

The **Machines** link in the header opens the whole sheet at `#/machines`. **Log a set** on a guide returns to Today and keeps a one-line reminder of which machine you came from. Sets are still stored on today’s session, the same way as before.

## Machine catalog

`machines.json` is the fleet. It was imported from the Created Training Center Equipment sheet. One JSON object per row, in sheet order, keyed by floor ID.

Only the Lower Extremity tab has an ID Number column. The IDs on the sheet are **L1**, **L2**, and **L3**. Every other row is kept under a placeholder key such as `UNASSIGNED-UE-01` with `idMissing: true`. Those keys are stable pointers back to the sheet. They are not floor IDs and should not be printed on placards.

Placeholder shape: `UNASSIGNED-<section>-<row>`

| Code | Sheet tab |
| --- | --- |
| LE | Lower Extremity |
| UE | Upper Extremity |
| TR | Trunk Equipment |
| TB | Total Body Equipment |
| FW | Free Weight Equipment |
| CA | Cardio Equipment |
| RS | Restorative Equipment |
| BZ | Business Equipment |

`<row>` is the data-row number on that tab, starting at 1 under the header. `UNASSIGNED-LE-04` is Lower Extremity row 4.

### Record fields

| Field | Meaning |
| --- | --- |
| `id` | Same as the object key. Floor ID, or a placeholder. |
| `idMissing` | `true` when the sheet had no ID Number. |
| `flags` | `missing-id`, `name-only`, `duplicate-name` when those apply. |
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

`guide` is the member-facing card. L1 is the filled-in example (Nautilus 2ST Hip Extension, the Super Glute Machine). Sheet columns stay on the record. The sentences from the printed card live here.

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

`startImage` and `finishImage` are optional paths to the start and finish photos, relative to the site root, with no leading slash: `guides/L1-start.jpg`. Leave them as `""` until the files exist. The page hides the photo row when both are empty.

Members mostly read the name, the nickname, and the muscles. Keep new placard copy that short.

## Add a machine or placard

1. Put the floor ID in the sheet’s ID Number column when you have one (`L4`, `U1`, …).
2. In `machines.json`, add or rename the object so the key and `id` are that floor ID. Set `idMissing` to `false` and drop `missing-id` from `flags`. If you are replacing a placeholder, delete the old `UNASSIGNED-…` key so the machine is not listed twice.
3. Fill `equipment`, `createdName`, `nickname`, `type`, `brand`, `muscle`, `movement`, `simplyPut`, `status`, and `video`. Set `paused` to `true` only when status is Paused.
4. When the printed placard exists, set `guide` using the L1 shape. Copy the placard, not a paraphrase.
5. If you add start/finish images, save them under `guides/`, point `startImage` / `finishImage` at them, add those paths to the `SHELL` list in `sw.js`, and bump the cache name `C` (`ct-v5`, `ct-v6`, …) so phones pick up the new files.
6. Changing `machines.json` alone does not require a new image path, but still bump `C` in `sw.js` when you want installed copies to drop the previous precache promptly. The service worker is network-first, so an open phone will also pick up the new JSON on the next load while online.

Do not put an `UNASSIGNED-…` key on a placard.

## Brand

Black background. Gold `#B38234` stripe above rust `#9F5F2C` stripe. Gold stays on top. The site name on the guide is written `www.created.center`.

## Offline

`sw.js` precaches `./`, `index.html`, `manifest.json`, `icon.svg`, and `machines.json`. A placard hash URL (`#/machine/L1`) is still just `index.html`, so the cached shell can open it. `?m=L1` is matched with `ignoreSearch` when the network is down.
