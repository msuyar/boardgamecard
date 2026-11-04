# Spellcard Forge

A card editor for Magic-style spell cards that use dice for damage. Each card has
damage at the bottom left, upcast cost in the middle and upcast damage at the bottom right.

Online version: https://claude.ai/artifact/2RJSfrAuLrVAHKAF116Sm7

## Files

| Path | What it is |
|---|---|
| `index.html` | The whole app: markup, styles and script in one file. |
| `manifest.webmanifest`, `icons/` | Let Android (Chrome) install the app on the home screen. |
| `sw.js` | Saves the app's files on the device so it opens without internet. |
| `decks/spellcard-deck.json` | The deck exported from the online version on 2026-10-25. |

## Running it locally

Double-click `index.html` to open it in a browser. You don't need to install or build anything.

When you open it this way:

- The deck is saved in that browser (IndexedDB), separate from the online version.
  Clearing site data erases it, so use **Export** to keep a copy.
- Art is stored inside each card, shrunk to 1000 px.
- **Export PNG** and **Export** download files through the browser.
- Fonts (Google Fonts) and the PNG exporter (html2canvas from cdnjs) load from the internet.
  Opened from a file, the page still works offline with fallback fonts, but PNG export is unavailable.

## On an Android phone

The phone needs the folder served over HTTPS once (GitHub Pages, Netlify or similar; opening the file directly
doesn't work well on Android). Then:

1. Open the address in Chrome on the phone and choose **Add to Home screen** (or **Install app**) from the menu.
2. Open Spellcard Forge from the home screen. After the first visit it works offline: fonts and the PNG exporter
   are saved too.
3. Cards are saved on the phone only. Use **Export** now and then to keep a backup file.

Updates: when the hosted files change, the phone picks them up the next time it opens the app online.

## Merging decks

The online version is where decks meet. **Merge** (in the deck panel) adds a deck file to the deck you have open:

- Cards that aren't in the deck yet are added.
- When a card is in both, the one edited more recently wins, so merging the same file twice changes nothing.
- Deleting a card doesn't carry over. Delete it on each device, or it comes back on the next merge.

To combine the phone deck and the online deck:

1. On the phone: **Export**, which saves `spellcard-deck.json` to Downloads.
2. Open the online version (on the phone or the computer) and **Merge** that file.
3. In the online version: **Export**, then **Merge** that file on the phone so it gets the cards made elsewhere.

Cards copied from the online deck keep the id of their full-size online art, so merging them back reuses that
image instead of storing a smaller copy.

To bring the deck saved in `decks/` in, click **Merge** and choose `decks/spellcard-deck.json`.

## Colors

| Letter | Color | Theme |
|---|---|---|
| `N` | Nature | blue and green |
| `A` | Arcana | purple |
| `R` | Radiant | yellow and white |
| `V` | Violence | red |
| `C` | Calmness | colorless (silver) |
| `D` | Demonic | a chaotic mix of the other five |

### Card layout and mana

The frame takes its color from the elements in the casting cost: one element tints it (dark red for Violence,
deep teal for Nature, and so on), several elements blend their tints, and Demonic blends all five. A card with no
element in its cost keeps the navy frame. The gold borders stay gold on every card. The art sits in an oval ringed by six element medallions, always in the same seats:

| Seat | Element |
|---|---|
| top | Radiant |
| upper right | Calmness |
| lower right | Nature |
| bottom | Demonic |
| lower left | Violence |
| upper left | Arcana |

The six seats show the casting cost as orbs. Costs are written like `AAAAA`: each element letter is one orb.
An element's first orb sits in its own seat; more orbs alternate clockwise, then counter-clockwise, outward from
there. An element never takes the seat facing it, which belongs to its opposite, so it can have at most 5 orbs.
Opposites are the facing seats: Radiant–Demonic, Calmness–Violence, Arcana–Nature.
The card's main element (`color`) seats first; other elements then fill the free seats in their own order.
The seat facing the main element shows the opposite element as an outlined, empty orb (5 Arcana orbs plus a
Nature outline, for example). Any other unused seat shows its own element's dimmed medallion. Numbers in the
casting cost aren't shown, and orbs that don't fit trigger a warning in the editor.
Common cards can have no main element (`"color": ""`, **None** in the editor): no medallion glows, the frame stays navy
when the cost is empty, dice are bronze, and the empty art oval shows the card-type icon.

The top-left corner holds the expansion logo (an uploaded image) and nothing else. New cards start with the
logo of the card you were editing. The artist credit, if filled in, shows at the bottom right.

Below the art is the parchment: the name and a card-type icon (Lackey, Spell, Buff, Debuff or Item) at the top, then rules text, a divider, and the flavor
panel, which has a fixed height. The three plaques along the bottom show damage, upcast cost and upcast damage.
In rules text, `{2d6}` shows an inline die, `{R}` a color symbol and `*word*` italics.

## Deck file format

```json
{
  "app": "spellcard-forge",
  "version": 2,
  "cards": [
    {
      "id": "example-tide",
      "v": 2,
      "name": "Tidecaller's Lash",
      "color": "N",
      "cost": "1NN",
      "type": "Spell — Evocation",
      "rarity": "rare",
      "rules": "… It takes {3d4} cold damage …",
      "flavor": "",
      "hasDamage": true,
      "hasUpcast": true,
      "dmg": { "count": 3, "die": 4, "bonus": 0, "type": "cold" },
      "upCost": "1N",
      "up": { "count": 1, "die": 4, "bonus": 1 },
      "artData": "data:image/jpeg;base64,…",
      "artX": 50, "artY": 50,
      "artist": "", "number": "002"
    }
  ]
}
```

- `hasDamage: false` hides the damage plaque (bottom left) and the upcast damage plaque (bottom right), for armor, buffs and other cards without damage.
  `hasUpcast: false` hides the upcast plaques. With both off, the card has only rules and flavor text, and the text box takes the extra space.
  If a card leaves these out, `hasDamage` counts as true and `hasUpcast` is true when `upCost` isn't empty.
- `kind` is the card type shown as an icon: `lackey`, `spell`, `buff`, `debuff` or `item`. Cards without it get a type guessed from their old `type` text.
  `type` and `rarity` are still kept in the file but no longer shown on the card.
- `rarity` is one of `common`, `uncommon`, `rare` or `mythic`.
- `die` is one of 4, 6, 8, 10, 12 or 20.
- `dmg.type` is empty or one of the damage types in the editor's list.
- `logoData` holds the expansion logo as an embedded PNG. `number` is kept in the file but no longer shown on the card.
- `artData` holds the card art as an embedded image. `artX` and `artY` set which part of the image shows (0–100).
  A card can have both `artData` and `artId`: the image itself, plus the id of the same image in the online version.
- `updatedAt` is when the card was last edited (milliseconds since 1970). Merge keeps whichever copy has the larger value.
- Cards without `"v": 2` use the old color letters (W U B R G P). Merge converts them automatically.

## Changing the app

Everything is in `index.html`:

- `COLORS` near the top of the script holds each color's frame, bar, symbol and dice colors.
  The Demonic frame is built from the other five right after it.
- `symbolSVG` draws the color symbols, and `DIE_SHAPES` draws the dice.
- `renderCard` builds the card markup. The card's CSS is under `the card` in the stylesheet,
  with sizes in `cqw` (100cqw is the card's width).

These local edits don't change the online version, which is published separately on claude.ai.
After changing a hosted copy, bump `CACHE` in `sw.js` if you also changed the icons or manifest.
`index.html` itself always loads fresh when the phone is online.
