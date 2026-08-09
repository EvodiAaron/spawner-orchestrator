# Spawner Orchestrator

A control panel for Ender IO Powered Spawners, written for ComputerCraft
(CC: Tweaked on Minecraft 1.12.2 / FTB Revelation). It runs on an Advanced
Computer, optionally mirrors to an Advanced Monitor as a touch panel, and
drives redstone outputs — either the computer's own faces or networked
Plethora Redstone Integrators — to switch spawners on and off.

Single file, zero dependencies.

## Install

On the Advanced Computer, run:

```
wget https://raw.githubusercontent.com/EvodiAaron/spawner-orchestrator/main/install install
install
```

The `install` script deletes any existing `startup`, downloads the latest
one, and reboots the computer to start the panel. It fetches before it
deletes, so a failed download never leaves the computer without a program
(and it only reboots on success).

**Updating later:** just run `install` again.

If you'd rather skip the installer, a one-off manual fetch works too:

```
wget https://raw.githubusercontent.com/EvodiAaron/spawner-orchestrator/main/startup startup
reboot
```

> Both require the server's ComputerCraft config to have HTTP enabled
> (`http_enable = true`, the default on most packs). If it's disabled, copy
> the file by hand into `saves/<world>/computercraft/computer/<id>/startup`.

The program must be named `startup`: a computer reboot resets every redstone
output to off, and the panel's first job on boot is to re-apply each
spawner's saved state.

## Hardware

- **Required:** one Advanced Computer (colour screen + arrow keys)
- **Optional:** an Advanced Monitor for a touch panel — place it against the
  computer or connect it over wired modems. A basic monitor works too, but
  is display-only (no touch). Only the first attached monitor is used.
- **Optional:** Plethora Redstone Integrators for spawners that aren't
  adjacent to the computer

The panel is fully usable on a bare computer with no monitor and no
integrators attached.

## Adding your first integrator

1. Place a Redstone Integrator against the Powered Spawner. Integrators
   emit on absolute sides: `up/down/north/south/east/west` (the computer's
   own faces use relative sides: `top/bottom/left/right/front/back`).
2. Connect the integrator and the computer with wired modems and networking
   cable, then right-click each modem so it lights up red.
3. On the panel: **A** (settings) → **Manage integrators** →
   **Scan for integrators**, and add the one that appears
   (e.g. `redstone_integrator_0`). Selecting a known integrator lets you
   give it a friendly label ("cave wall") or remove it.
4. Back in settings, add a spawner, set its target to that integrator, pick
   the side facing the spawner, and set **Inverted** to match the spawner's
   own redstone mode (active with signal vs. active without signal). The
   panel always shows the logical state — "ON" means spawning — regardless
   of electrical polarity.

## Controls

**Terminal**

| Key | Action |
| --- | --- |
| Up / Down | move selection |
| Space | toggle the selected spawner |
| Tab | toggle all at once — master switch: anything on → all off, else all on |
| T | turn the selected spawner on for a one-off duration (live countdown) |
| M | cycle the default timer: Off → 1m → 5m → 10m → 15m → 30m → 1h → 2h |
| A (or S) | open settings |
| Enter / Backspace | select / go back, in menus |
| Ctrl+T | quit (outputs keep their last state) |

Text entry: Enter accepts, Backspace on an empty field cancels.

**Advanced Monitor**

Tap a spawner row to toggle it; tap `[ Toggle All ]` for the master switch;
tap `[ Timer: … ]` to cycle the default timer (see below). Settings are
configured on the terminal only. The monitor automatically
uses the largest text scale that still fits the spawner list, so a wall
display stays readable from a distance. While settings are open the
monitor shows a notice instead of a stale list.

## Timers

**Default timer** — the `[ Timer: … ]` control at the bottom of the monitor
(or **M** on the terminal) cycles a persistent default: **Off**, 1m, 5m,
10m, 15m, 30m, 1h, or 2h. Every toggle-on (row tap, Space, or Toggle All)
runs for that duration; Off means no timer — stay on until switched off.
The choice is saved in the config and survives reboots.

**One-off timer** — press **T** on a spawner to run it for a picked
duration regardless of the default mode.

**Interval mode** — each spawner can optionally pulse instead of holding a
steady on state: pick an interval in settings (1s, 3s, 5s, 10s, 15s, 30s,
60s, 120s) and, while the spawner is on, its output cycles that long on,
then that long off, starting with the on phase. The row still reads **ON**,
but the text shades faintly during the off half of the pulse so you can see
the phase at a glance. Set the interval back to "off" for a steady signal.

**Ignore Timer** — each spawner has an "Ignore timer" option in settings.
When set, no timer ever attaches to it: it always stays on until switched
off manually, regardless of the default mode or Toggle All.

**Bypass Toggle All** — a second per-spawner option that makes the master
switch skip it entirely, in both directions: Toggle All neither turns it
on nor off. Useful for a spawner that should never be caught by the big
red button.

Either way the countdown shows on both displays and the spawner switches
itself off at zero. A manual toggle cancels a running countdown.
Countdowns survive reboots, resuming from the remaining time recorded at
the last save.

## Per-spawner options and layout

Each spawner's settings screen also offers:

- **Menu position (Top/Bottom)** — Bottom spawners render as a group
  anchored to the bottom of the monitor, just above the `[ Toggle All ]`
  button (when the list is too long to fit, it falls back to one scrolled
  list). On the terminal, Bottom spawners simply sort after the Top group.
- **Move up / Move down** — reorder spawners within the list. Reordering
  also works BIOS-style directly in the settings list: select a spawner
  row and press **+** / **-** to move it up or down.

The header bar colour on both displays is selectable in settings
(**Header colour**), with a picker that previews each colour.

## Persistence

State lives in `config` next to the program and is saved after every
change, written atomically via a temp file. A corrupt or unreadable config
is moved aside to `config.bak` and the panel starts fresh — it never
crashes on bad data. A missing integrator marks its spawners as offline
(`!`) but the panel keeps running and re-applies states when the
peripheral reattaches.

## Development

The program targets Lua 5.1 (Cobalt) and uses no APIs beyond standard
CC: Tweaked ones. It runs a single event loop — `read()` is deliberately
avoided so monitor touches are never queued behind text input.
