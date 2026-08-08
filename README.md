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
wget https://raw.githubusercontent.com/EvodiAaron/spawner-orchestrator/main/startup startup
reboot
```

> `wget` requires the server's ComputerCraft config to have HTTP enabled
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
   (e.g. `redstone_integrator_0`).
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
| T | turn the selected spawner on for a preset duration (live countdown) |
| A (or S) | open settings |
| Enter / Backspace | select / go back, in menus |
| Ctrl+T | quit (outputs keep their last state) |

Text entry: Enter accepts, Backspace on an empty field cancels.

**Advanced Monitor**

Tap a spawner row to toggle it; tap `[ Settings ]` to configure on the
terminal. While settings are open the monitor shows a notice instead of a
stale list.

## Timers

Press **T** on a spawner to run it for 1, 5, 10, 15, or 30 minutes, or 1 or
2 hours. The countdown shows on both displays and the spawner switches
itself off at zero. A manual toggle cancels the countdown ("on" via Space
or tap means on indefinitely). Countdowns survive reboots, resuming from
the remaining time recorded at the last save.

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
