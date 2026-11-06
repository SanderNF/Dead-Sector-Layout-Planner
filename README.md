# Dead Sector Layout Planner

A floor planner for building data centres in the game **Dead Sector**. Place racks, PDUs, UPS chains, coolers and switches on a grid, wire them together, and see how much power, cooling and usable capacity each rack gets — including what's left when one or two of a rack's PDUs fail.

**Open the planner:** https://sandernf.github.io/Dead-Sector-Layout-Planner/

The whole app is a single `index.html`: no install, no build step, no server, no account. Your layout stays in your own browser.

> This is a fan-made tool and is not affiliated with the developers of Dead Sector. Device stats are taken from the game as best we can; if something is off, please [open an issue](../../issues) or see [CONTRIBUTING.md](CONTRIBUTING.md).

## Contents

- [Quick start](#quick-start)
- [Using it](#using-it)
- [Where data is stored](#where-data-is-stored)
- [Adding devices without code](#adding-devices-without-code)
- [Extending in code](#extending-in-code)
- [Hosting your own copy](#hosting-your-own-copy)
- [Contributing](#contributing)
- [License](#license)

## Quick start

1. Open the planner (link above), or download `index.html` and open it in any modern browser.
2. Press `A` and place a few **racks**, a **PDU** and a **UPS**.
3. Press `P` and draw power cables UPS → PDU → rack.
4. Read each rack's power, cooling and usable capacity straight off the floor. Click a block to change its ratings in the side panel.
5. Use **Import / export** to save a copy of your layout or share it with others.

## Using it

| Action | How |
| --- | --- |
| Place a device | **Add device** (or press `A`), pick one, click the floor. `R` rotates. |
| Draw a cable | `P` for power, `N` for network. Click a block's edge, click to bend, click another block's edge to finish. |
| Select | Click, Shift-click to add, or drag a box on empty floor. `Ctrl+A` selects all. |
| Move | Drag any selected block; the selection moves together and cables re-route. |
| Copy and paste | `Ctrl+C`, `Ctrl+V` (pastes at the cursor), `Ctrl+D` duplicates, `Ctrl+X` cuts. |
| Pan and zoom | Right-drag or Space-drag to pan, scroll to zoom, `F` to fit. |
| Undo | `Ctrl+Z`, redo with `Ctrl+Shift+Z`. |

### How the numbers work

- **Power** flows UPS chain → PDU → rack. UPS units joined by power cables form a chain whose output is the sum of its units, up to the chain limit. A UPS can't feed a rack directly.
- Each chain's output is **split evenly** between the racks it reaches. Each PDU passes at most its capacity; if the shares through it add up to more, they're scaled down.
- **1 failed / 2 failed** are the worst case when that many of the rack's own PDUs go down. The chain's output is then re-shared between the racks still connected.
- **Cooling**: each cooler splits its output evenly between the racks in its row whose centre is within its reach (measured from the cooler's centre, across its short side).
- **Usable** is the lower of power and cooling.

## Where data is stored

Each player's layout is saved in their own browser (`localStorage`, key `rack-power-layout-v1`). Nothing is sent anywhere, and players can't see each other's layouts. Clearing site data deletes the layout, so use **Import / export** to back up or share one.

### Layout format

```json
{
  "format": "rack-power-layout",
  "version": 4,
  "devices":   [ /* custom devices, same shape as below */ ],
  "overrides": { "pdu": { "kW": 150 } },
  "blocks": [ { "id": "b1", "type": "rack", "x": 4, "y": 12, "w": 4, "h": 5, "name": "R1" } ],
  "cables": [ { "id": "c2", "kind": "power", "pts": [[7, 2.5], [8, 2.5]],
                "a": { "block": "b1", "ox": 3, "oy": 1.5 }, "b": { "block": "b2", "ox": 0, "oy": 1.5 } } ],
  "seq": 3
}
```

- `overrides` holds rating changes made in the side panel, per device id.
- `ox`/`oy` are where a cable attaches, relative to the block's top-left corner, in cells.
- Older layouts (without `format`) are upgraded automatically when loaded.

## Adding devices without code

Open **Devices**, fill in the form, and choose what the device **works like**. To share devices, open **Share devices as a pack**, copy the JSON and post it. Others paste it and click **Import pasted pack**.

### Device pack format

```json
{
  "format": "rack-power-devices",
  "version": 1,
  "devices": [
    { "id": "small_ups", "label": "Small UPS", "short": "Small UPS", "cat": "Power",
      "prefix": "SU", "w": 2, "h": 2, "behavior": "ups",
      "stats": { "kW": 12, "chainMax": 6 }, "color": "#2f7d43" }
  ]
}
```

| Field | Meaning |
| --- | --- |
| `id` | Unique key (letters, digits, `_`). Layouts refer to it, so don't rename it. |
| `label`, `short` | Full name and toolbar name. |
| `cat` | Group in the Add device menu. |
| `prefix` | Automatic naming: `SU` gives SU1, SU2, … |
| `w`, `h` | Size in grid cells (1–40). |
| `behavior` | One of the behaviours below. |
| `stats` | Numbers for that behaviour. Missing values use the default. |
| `color` | Optional `#rrggbb` outline colour. |

### Behaviours

| `behavior` | What it does | `stats` (default) |
| --- | --- | --- |
| `rack` | Draws power from PDUs, gets cooling from coolers in reach | `maxPdus` (3) |
| `pdu` | Takes power from a UPS chain and passes it on to racks | `kW` (138) |
| `ups` | Chains with other UPS units and feeds PDUs | `kW` (24), `chainMax` (4) |
| `cooler` | Splits its cooling between racks within reach in its row | `kW` (40), `reach` (18) |
| `switch` | Takes network cables from racks and other switches | — |
| `none` | Takes up space only (walls, pillars, labels) | — |

## Extending in code

Everything lives in the `<script>` at the bottom of `index.html`. The parts you'll touch are at the top, under **DEVICE CATALOG**:

- **`BUILTIN_DEVICES`** — the devices that ship with the planner. Add an entry to add a built-in device.
- **`BEHAVIORS`** — each behaviour's name, help text and stats (with defaults, minimums and labels). The side panel and the device form are generated from this.
- **`RULES`** — which behaviours a power or network cable may connect, as alphabetically sorted pairs joined by `-` (for example `pdu-rack`).

A brand-new behaviour (not just a variant of an existing one) also needs handling in `analyze()`, which does the power and cooling maths, and a readout line in `blockSVG()`.

If you change the saved layout shape, bump `VERSION` and upgrade older layouts in `clean()` so existing saves keep working.

Test changes by opening `index.html` directly in a browser; no server is needed.

## Hosting your own copy

Forks can be served for free with GitHub Pages:

1. Push `index.html`, `README.md` and `.nojekyll` to the default branch.
2. In the repo, go to **Settings → Pages**.
3. Under **Build and deployment**, choose **Deploy from a branch**, pick the branch and `/ (root)`, and save.

The site appears at `https://<your-username>.github.io/Dead-Sector-Layout-Planner/` after a minute or so.

## Contributing

Bug reports, stat corrections, new devices and code changes are all welcome. Read [CONTRIBUTING.md](CONTRIBUTING.md) for how to get started. For questions, I'm usually reachable on [Discord](https://discord.com/users/612612141924614144).

## License

[MIT](LICENSE) © 2026 SanderNF
