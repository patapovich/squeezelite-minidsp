# squeezelite-minidsp

Helper scripts for using [squeezelite](https://github.com/ralph-irving/squeezelite) with a [miniDSP](https://www.minidsp.com/) device (e.g. DDRC-24).

## Scripts

### squeezelite-volume

Maps LMS volume commands to minidsp master gain. Used with the squeezelite `-w` option.

LMS controls volume in a -72..0 dB range, which squeezelite linearises to 0–100 before calling the script. This script maps that to minidsp master gain via:

```
gain = FLOOR_DB * (1 - (vol/100)^CURVE_K)
```

Slider 100 is always 0 dB. **Slider 0 is a no-op** — the script exits without touching the device. LMS sends `vol=0` on pause/stop, and on a multi-input MiniDSP (e.g. DDRC-24 with USB + Toslink) hard-muting the master also silences any other input that's currently selected. Squeezelite is paused so no audio is flowing from it anyway; leaving the master gain alone keeps Toslink (or whatever input is selected) playing at the previous level, and any external HA / MQTT volume display doesn't snap to 0 on every pause.

Two tunables (override via env vars):

| Var | Default | Effect |
|---|---|---|
| `FLOOR_DB` | `-50` | dB at slider 1. Lower magnitude → less dynamic range, hotter minimum. Higher magnitude (e.g. `-72`) softens the mute→audible cliff. |
| `CURVE_K` | `1` | Curve exponent. `1` = linear (matches a simple FLOOR_DB mapping). `>1` = bottom-heavy: slider 1–30 stays near floor, listening sweet spot moves up the slider, top steepens. `<1` = top-heavy (audio-taper / log-pot feel). |

Examples:

```sh
FLOOR_DB=-50 CURVE_K=1   squeezelite-volume 50   # linear, slider 50 → -25 dB
FLOOR_DB=-72 CURVE_K=2   squeezelite-volume 50   # squared, slider 50 → -54 dB
FLOOR_DB=-72 CURVE_K=3   squeezelite-volume 50   # cubed, slider 50 → -63 dB
```

For systemd-managed installs add `FLOOR_DB="-72"` and `CURVE_K="2"` to `/etc/default/squeezelite` and ensure the unit's launcher exports the env (e.g. `set -a; . /etc/default/squeezelite; set +a; exec squeezelite ...`).

A short debounce (0.3 s) is applied so LMS's fade-in ramp on power-on is collapsed to a single gain command.

Fade-out on power-off is suppressed entirely in squeezelite (see the `-w` patch) by skipping the volume script when the player output state is `OUTPUT_OFF`.

### squeezelite-source

Switches the minidsp input source based on squeezelite power state. Used with the squeezelite `-S` option.

- Power on → `minidsp source usb`
- Power off / init → `minidsp source toslink`

## Requirements

- [minidsp-rs](https://github.com/mrene/minidsp-rs) CLI (`minidsp` in PATH)
- squeezelite built with `-DGPIO` and the `-w` volume script patch from [PR #258](https://github.com/ralph-irving/squeezelite/pull/258)

## Installation

```sh
sudo cp squeezelite-volume squeezelite-source /usr/local/bin/
sudo chmod +x /usr/local/bin/squeezelite-volume /usr/local/bin/squeezelite-source
```

## Usage

```sh
squeezelite -o hw:CARD=DDRC24,DEV=0 \
    -w /usr/local/bin/squeezelite-volume \
    -S /usr/local/bin/squeezelite-source
```
