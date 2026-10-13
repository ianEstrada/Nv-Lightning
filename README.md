# Nv-Lightning

Windows tuning and monitoring tool for NVIDIA GPUs. Talks to the driver directly, verifies
every value it writes, and works on **Turing and newer** (verified on an RTX 2080).

## What it does

| | |
| --- | --- |
| **Tuning** | Core, memory, **XBAR** and **SYS** clock offsets · power limit · fan (auto / fixed % / curve) · voltage lock (including *lowest available point*) |
| **Monitoring** | Core, memory and XBAR clocks · temperature · power draw and limit · utilisation |
| **Profiles** | Save your tuning under a name, overwrite an existing one, apply in one click |
| **Startup** | Optional: your last applied tuning comes back at logon |
| **Diagnostics** | Telemetry ring (one line per second, flushed to disk, survives a hard hang) · ReBAR status |
| **Extras** | Two NVIDIA Profile Inspector profiles bundled (`nip/`) · shutdown timer |

Every write is read back. If the driver ignores it, you get a failure message and the previous
state is restored — no silent "done".

## Install and use

1. Download `Nv-Lightning.exe` from the **[releases page](https://github.com/ianEstrada/Nv-Lightning/releases)**.
2. Run it **as administrator** (needed to apply tuning; monitoring works without it).
3. **TUNING** → set what you want → **Apply All**.
4. **PROFILES → Save as…** → type a name → your tuning is stored.
5. Enable the logon task if you want it reapplied automatically at boot.

`0 + Apply` means *back to default*: it clears the stored offset.

## Profiles

A fresh install has **no profiles at all** — you create them, with your own names. They live in
`%ProgramData%\Nv-Lightning\profiles.json` (one file, easy to back up). If that file is ever
unreadable it is parked as `profiles.json.bad-<timestamp>`, never deleted.

## Why this tool

- **Turing and newer** — covers the generations several tuning tools skip.
- **XBAR and SYS** — the clock domains almost nothing else exposes.
- **It tells you the truth** — every value verified, every failure reported.
- **It leaves evidence** — the telemetry ring survives a hard hang.

Inspired by **[mVolt+](https://github.com/b00nz/mVolt)**, which targets Blackwell (RTX 50) and
showed what talking to the driver directly can expose. Note: these are undocumented driver
interfaces, and NVIDIA can break them with a driver update — it already happened to mVolt+ once.

## Credits

- **Lightning** — the tool, the XBAR/SYS work and the driver research.
- **Succes** — collaboration on the NIP / ReBAR profile research and the bundled `.nip` profiles.
- **Orbmu2k** — author of NVIDIA Profile Inspector, bundled inside the tool (its own license).

## License

**CC BY-ND 4.0** — free to use and share, no modified redistributions, attribution required.
Full text: [LICENSE.md](LICENSE.md).
