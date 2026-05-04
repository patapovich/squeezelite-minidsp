# squeezelite-minidsp

Helper scripts for using [squeezelite](https://github.com/ralph-irving/squeezelite) with a [miniDSP](https://www.minidsp.com/) device (e.g. DDRC-24).

## Scripts

### squeezelite-volume

Maps LMS volume commands to minidsp master gain. Used with the squeezelite `-w` option.

LMS controls volume in a -72..0 dB range, which squeezelite linearises to 0–100 before calling the script. This script maps that to minidsp's -127..0 dB master gain range using the same linear dB scale.

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
