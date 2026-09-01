---
title: README
type: note
permalink: tinkering/retro-player/readme
---

# retro-player — software

Agents for the retro NFC music player. See the [[Design and Project Plan - v1|Design & Project Plan]] for the hardware side.

| File | Purpose |
|------|---------|
| `nfc_agent.py` | NFC card → MPD album playback; re-tap = pause toggle; `--pair` mode |
| `vu_agent.py` | peppyalsa FIFO → VU ballistics → hardware PWM on GPIO 12/13; `--test` sweep for trimmer calibration |
| `asound.conf` | ALSA chain: MPD → level meter tap → dmix → Amp2 |
| `systemd/` | Service units for both agents |
| `cards.example.json` | Card map format (real file: `cards.json`, created by `--pair`) |

## Setup (Phase 1–3 of the project plan)

### 1. config.txt (`/boot/firmware/config.txt`)

```ini
# Amp2 (disable onboard audio)
#dtparam=audio=on
dtoverlay=hifiberry-dacplus-std

# Hardware PWM for the VU meters on GPIO12/13 (NOT the default 18/19 —
# those belong to I2S). func=4 selects ALT0.
dtoverlay=pwm-2chan,pin=12,func=4,pin2=13,func2=4

# Buses
dtparam=i2c_arm=on
dtparam=spi=on
```

### 2. System packages + peppyalsa

```bash
sudo apt update
sudo apt install -y mpd mpc git build-essential autoconf automake libtool libasound2-dev libfftw3-dev python3-venv i2c-tools
git clone https://github.com/project-owner/peppyalsa.git
cd peppyalsa && aclocal && libtoolize && autoconf && automake --add-missing
./configure && make && sudo make install   # installs /usr/local/lib/libpeppyalsa.so
```

### 3. ALSA + MPD

Copy `asound.conf` to `/etc/asound.conf`. In `/etc/mpd.conf`:

```
music_directory  "/home/pi/music"
audio_output {
    type   "alsa"
    name   "Amp2 via meter"
    device "retro_out"
    mixer_type    "hardware"
    mixer_device  "default"
    mixer_control "Digital"    # verify with: amixer -c sndrpihifiberry scontrols
}
```

Then `sudo systemctl restart mpd`, drop a test album under `music_directory`, and check `mpc update && mpc add / && mpc play` makes sound *and* `cat /var/run/peppyalsa/meter.fifo | xxd | head` shows data flowing (create the FIFO first: `sudo mkdir -p /var/run/peppyalsa && sudo mkfifo /var/run/peppyalsa/meter.fifo`).

### 4. Python environment

```bash
cd /home/pi/retro-player
python3 -m venv venv
venv/bin/pip install adafruit-circuitpython-pn532 adafruit-blinka python-mpd2 rpi-hardware-pwm
```

### 5. Bring-up order

1. `i2cdetect -y 1` → PN532 at 0x24.
2. `venv/bin/python nfc_agent.py -v` → tap a card, see the UID logged as unknown.
3. `venv/bin/python nfc_agent.py --pair "Albums/Artist/Album" --name "Album"` → pair it.
4. Run the agent again, tap → album plays. Tap again → pauses.
5. `venv/bin/python vu_agent.py --test` → needles sweep; set each trimmer so full sweep = full scale.
6. Play music, `venv/bin/python vu_agent.py --debug` → needles dance; confirm raw frame values peak near 100 (if they top out lower, lower `METER_MAX` or raise MPD volume).
7. Install services:
   ```bash
   sudo cp systemd/*.service /etc/systemd/system/
   sudo systemctl daemon-reload
   sudo systemctl enable --now nfc-agent vu-agent
   ```

## Notes & gotchas

- **PWM permissions:** `rpi-hardware-pwm` writes to `/sys/class/pwm/pwmchip0`. Either run vu-agent as root, or add a udev rule granting the `pi` user access:
  `SUBSYSTEM=="pwm", ACTION=="add", RUN+="/bin/chgrp -R gpio /sys/class/pwm", RUN+="/bin/chmod -R g+w /sys/class/pwm"` in `/etc/udev/rules.d/99-pwm.rules`.
- **Pi 5:** set `PWM_CHIP = 2` in `vu_agent.py` (RP1 numbering) — GPIO12/13 remain correct.
- **FIFO frame format:** `vu_agent.py` assumes 4-byte frames (two uint16 LE, 0–100). If `--debug` shows garbage, dump the FIFO with `xxd` and adjust `FRAME_SIZE`/parsing — peppyalsa's format is set at build time by its version; this matches current master.
- **`/var/run` is tmpfs** — the FIFO disappears on reboot. The vu-agent service unit recreates it (`ExecStartPre`), so peppyalsa finds it as long as vu-agent starts before audio plays; the unit orders itself before/wants mpd accordingly.
- **Card map hot-reload:** nfc-agent watches `cards.json` mtime, so `--pair` (or the future web admin) takes effect without a restart.