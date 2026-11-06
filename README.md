# Dead Sector Layout Planner

A floor planner for building data centres in the game **Dead Sector**. Place racks, PDUs, UPS chains, coolers and switches on a grid, wire them together, and see how much power, cooling and usable capacity each rack gets — including what's left when one or two of a rack's PDUs fail.

**Open the planner:** https://sandernf.github.io/Dead-Sector-Layout-Planner/

The app is one `index.html` plus `default_parts.json`, the devices that ship with it. There's no install, no build step and no account, and your layout stays in your own browser.

**Game version:** device stats and mechanics are based on Dead Sector [Upcoming Patch v0.2.1](https://store.steampowered.com/news/app/4736470/view/719046905302090028).

> This is a fan-made tool and is not affiliated with the developers of Dead Sector. If a number doesn't match the game, please [report a stat correction](../../issues/new/choose) or see [CONTRIBUTING.md](CONTRIBUTING.md).

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

1. Open the planner (link above). To run it yourself, see [Running it locally](#running-it-locally).
2. Press `A` and place a few **racks**, a **PDU** and a **UPS**.
3. Press `P` and draw power cables UPS → PDU → rack.
4. Read each rack's power, cooling and usable capacity straight off the floor. Click a block to change its ratings in the side panel.
5. Use **Import / export** to save your layout as a `.json` file, or open one someone shared with you.

## Using it

| Action | How |
| --- | --- |
| Place a device | **Add device** (or press `A`), pick one, click the floor. `R` rotates. |
| Draw a cable | `P` power, `N` network, `I` interlink, `S` signal. Click a block's edge, click to bend, click another block's edge to finish. |
| Select | Click, Shift-click to add, or drag a box on empty floor. `Ctrl+A` selects all. |
| Move | Drag any selected block; the selection moves together and cables re-route. |
| Copy and paste | `Ctrl+C`, `Ctrl+V` (pastes at the cursor), `Ctrl+D` duplicates, `Ctrl+X` cuts. |
| Pan and zoom | Right-drag or Space-drag to pan, scroll to zoom, `F` to fit. |
| Undo | `Ctrl+Z`, redo with `Ctrl+Shift+Z`. |

### How the numbers work

- **Power** flows UPS chain → PDU → rack. UPS units joined by power cables form a chain whose output is the sum of its units, up to the chain limit. A UPS can't feed a rack directly.
- Each chain's output is **split evenly** between the racks it reaches. Each PDU passes at most its capacity; if the shares through it add up to more, they're scaled down.
- **1 failed / 2 failed** are the worst case when that many of the rack's own PDUs go down. The chain's output is then re-shared between the racks still connected.
- **Cooling**: each cooler splits its output evenly between the racks it reaches. An aisle cooler reaches racks in its row whose centre is within its reach (measured from the cooler's centre, across its short side). A room cooler, such as the Room CRAC, reaches racks whose centre is inside a square area centred on it (24 × 24 cells by default).
- **Usable** is the lower of power and cooling.
- **Network**: racks connect to switches with network cables. Each switch's bandwidth is split evenly between its racks, and no rack gets more than its own max (1 Tbps for now). Whatever a rack can't use goes to the others.
- **Cluster Interlink**: switches connected to an interlink with interlink cables can lend spare bandwidth to each other's racks, up to the interlink's limit (3.2 Tbps). Switches can't be joined directly with network cables.
- **Failover Controller**: connect it to two switches with signal cables. If one switch fails, its racks move to the other. **1 switch failed** is the worst case when any one switch goes down.

## Where data is stored

Each player's layout is saved in their own browser (`localStorage`, key `rack-power-layout-v1`). Nothing is sent anywhere, and players can't see each other's layouts. Clearing site data deletes the layout, so use **Import / export** to back it up: **Save as file** downloads it as a `.json` file and **Open file…** loads one. You can also copy and paste the JSON.

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

Open **Devices**, fill in the form, and choose what the device **works like**. To share devices, open **Share devices as a pack** and use **Save pack as file**, or copy the JSON and post it. Others use **Import pack file…**, or paste it and click **Import pasted pack**.

### Device pack format

```json
{
  "format": "rack-power-devices",
  "version": 1,
  "devices": [
    { "id": "small_ups", "label": "Small UPS", "short": "Small UPS", "cat": "Power",
      "prefix": "SU", "w": 2, "h": 2, "behavior": "ups",
      "stats": { "kW": 6, "chainMax": 4 }, "color": "#2f7d43" }
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
| `rack` | Draws power from PDUs, gets cooling from coolers in reach and network from switches | `maxPdus` (3), `netGbps` (1000) |
| `pdu` | Takes power from a UPS chain and passes it on to racks | `kW` (138) |
| `ups` | Chains with other UPS units and feeds PDUs | `kW` (24), `chainMax` (4) |
| `cooler` | Splits its cooling between racks within reach in its row | `kW` (40), `reach` (18) |
| `roomcooler` | Splits its cooling between racks inside a square area centred on it | `kW` (35), `reach` (24, the square's width and height) |
| `switch` | Splits its bandwidth evenly between the racks cabled to it | `Gbps` (6400) |
| `interlink` | Joins switches with interlink cables so they share spare bandwidth | `Gbps` (3200) |
| `failover` | Links two switches with signal cables; if one fails, its racks move to the other | — |
| `none` | Takes up space only (walls, pillars, labels) | — |

## Extending in code

The devices that ship with the planner are in **`default_parts.json`**, in the same format as a [device pack](#device-pack-format), plus an optional one-key shortcut `key`. Add an entry there to add a built-in device; no code change needed.

The code lives in the `<script>` at the bottom of `index.html`. The parts you'll touch are at the top, under **DEVICE CATALOG**:

- **`BEHAVIORS`** — each behaviour's name, help text and stats (with defaults, minimums and labels). The side panel and the device form are generated from this.
- **`RULES`** — which behaviours a power or network cable may connect, as alphabetically sorted pairs joined by `-` (for example `pdu-rack`).

A brand-new behaviour (not just a variant of an existing one) also needs handling in `analyze()`, which does the power and cooling maths, and a readout line in `blockSVG()`.

If you change the saved layout shape, bump `VERSION` and upgrade older layouts in `clean()` so existing saves keep working.

### Running it locally

Browsers won't let a page opened straight from disk (`file://`) read `default_parts.json`, so serve the folder instead:

```sh
python3 -m http.server
```

Then open http://localhost:8000/ and reload after each change.

## Hosting your own copy

Forks can be served for free with GitHub Pages:

1. Push `index.html`, `default_parts.json` and `.nojekyll` to the default branch.
2. In the repo, go to **Settings → Pages**.
3. Under **Build and deployment**, choose **Deploy from a branch**, pick the branch and `/ (root)`, and save.

The site appears at `https://<your-username>.github.io/Dead-Sector-Layout-Planner/` after a minute or so.

## Contributing

Bug reports, stat corrections, new devices and code changes are all welcome. Read [CONTRIBUTING.md](CONTRIBUTING.md) for how to get started. For questions, I'm usually reachable on [Discord](https://discord.com/users/612612141924614144).

## License

[MIT](LICENSE) © 2026 SanderNF
