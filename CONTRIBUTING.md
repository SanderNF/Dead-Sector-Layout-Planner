# Contributing

Thanks for helping improve the Dead Sector Layout Planner! Whether you've spotted a wrong number, want a new device in the catalogue, or want to change the code, this page explains how.

## Talk to me first

If you're not sure where to start, or want to check an idea before spending time on it, just ask. I'm usually reachable on **Discord**: https://discord.com/users/612612141924614144

Message me there for anything from "is this a bug?" to "would you take a PR for X?". Ideas, feedback and game knowledge (stats, mechanics, new devices) are just as valuable as code.

## Ways to contribute

Each type below has an issue template that asks for the right details. Pick one when you [open an issue](../../issues/new/choose).

### Report a bug

Open a **Bug report** and include:

- What you did, what you expected, and what happened instead.
- Your browser and version.
- If you can, an exported layout (**Import / export** → copy the JSON) that shows the problem. That makes it much easier to reproduce.

### Correct game stats or mechanics

The planner is only useful if its numbers match the game. Stats are currently based on [Upcoming Patch v0.2.1](https://store.steampowered.com/news/app/4736470/view/719046905302090028). If a device rating, chain limit, cooling reach or power-sharing rule is wrong, or a newer patch changes it:

- Open a **Stat correction** (or message me) with the correct value and where it comes from: a screenshot, an in-game test, or a patch note.
- Mention the game version or patch if you know it, since stats can change between updates.

### Suggest or add a device

The quickest way is to build it in the planner itself: open **Devices**, create it, then **Share devices as a pack** and paste the JSON into a **New device** issue. See [Adding devices without code](README.md#adding-devices-without-code) for the format.

If it should ship as a built-in device, you can also add it to `BUILTIN_DEVICES` yourself in a pull request (see below).

### Request a feature

Open a **Feature request** describing what you want to do in the planner and why. Describing the problem you're solving matters more than a specific solution.

## Working on the code

The whole app is a single file, `index.html`, with plain HTML, CSS and JavaScript. There are no dependencies, no build step and no test runner.

### Setup

```sh
git clone https://github.com/SanderNF/Dead-Sector-Layout-Planner.git
cd Dead-Sector-Layout-Planner
```

Then open `index.html` in your browser. After each change, reload the page.

### Where things live

All the code is in the `<script>` at the bottom of `index.html`. The README's [Extending in code](README.md#extending-in-code) section is the map. In short:

| What you want to change | Where |
| --- | --- |
| Built-in devices | `BUILTIN_DEVICES` |
| A behaviour's stats, defaults or help text | `BEHAVIORS` |
| Which devices a cable may connect | `RULES` |
| Power and cooling maths | `analyze()` |
| How a block and its readout are drawn | `blockSVG()` |
| Loading and upgrading saved layouts | `clean()` |

### Ground rules

- **Keep it a single file with no build step.** Anyone should be able to download `index.html` and open it. Avoid adding external libraries or CDN scripts unless we've talked about it first.
- **Don't break people's saved layouts.** Layouts live in players' browsers, so a change to the saved shape must bump `VERSION` and upgrade older layouts in `clean()`. Never rename an existing device `id`; layouts refer to it.
- **Keep data local.** The planner doesn't send anything anywhere, and it should stay that way.
- **Match the existing style.** Follow the formatting and naming around the code you're changing, and keep changes focused: one fix or feature per pull request.

### Before you open a pull request

Test by hand in the browser:

- [ ] Place, move, rotate, copy/paste and delete blocks; draw and remove power and network cables.
- [ ] Check that the power, failure and cooling numbers are what you expect for a small test layout.
- [ ] Undo and redo still work after your change.
- [ ] Export a layout, reload the page, import it again, and it comes back the same.
- [ ] An existing saved layout from before your change still loads.
- [ ] No errors in the browser console.
- [ ] If you changed how something works, update `README.md` to match.

### Pull requests

1. Fork the repo and create a branch from `main`.
2. Make your change and test it as above.
3. Open a pull request that says what changed and why. Screenshots help for anything visual. Link the issue if there is one.

I'll review it as soon as I can. If it's been a while, feel free to nudge me on Discord.

## Be kind

Be respectful and patient with each other. Everyone here is helping out in their free time.

## License

By contributing, you agree that your contributions are licensed under the [MIT License](LICENSE) that covers this project.
