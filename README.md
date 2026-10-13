# Nv-Lightning

**Nv-Lightning** is a Windows tuning and monitoring tool for NVIDIA GPUs. It exposes the
controls vendor utilities hide — memory, crossbar (**XBAR**) and system-domain (**SYS**) clock
offsets, the power limit, fan behaviour and a fixed-voltage lock — plus live telemetry, named
profiles and an optional apply-at-logon.

It was built and verified on **Turing (RTX 2080)** and works on newer generations. Every value
it writes is read back and verified: if the driver ignores a request, the tool says so and
reverts instead of pretending it worked.

Everything you need to know is on this page. The only other file you have to read is
[LICENSE.md](LICENSE.md).

**Contents**

- [What it does](#what-it-does)
- [Requirements](#requirements)
- [Install](#install)
- [Quick start](#quick-start)
- [The app, section by section](#the-app-section-by-section)
- [Profiles](#profiles)
- [Command line](#command-line)
- [Troubleshooting](#troubleshooting)
- [Safety](#safety)
- [Why this tool](#why-this-tool)
- [Credits](#credits)
- [License](#license)
- [Release notes](#release-notes)

---

## What it does

| Area | What you get |
| --- | --- |
| **Live monitoring** | Core clock, memory clock, XBAR clock, GPU temperature, power draw, power limit, utilisation, fan speed |
| **Core / memory tuning** | Clock offsets in MHz, verified by readback |
| **XBAR / SYS tuning** | Crossbar and system-domain clock offsets — the domains most tuning tools never expose |
| **Power limit** | The wattage ceiling, inside the range your card reports |
| **Fan control** | Auto, fixed percentage, or a temperature → speed curve with hysteresis |
| **Voltage lock** | Hold the core at a fixed voltage point, including *lowest available point* |
| **Profiles** | Save your tuning under a name, overwrite an existing one, apply in one click |
| **Apply at logon** | Optional: your last applied tuning comes back after a reboot |
| **Telemetry ring** | One line per second flushed to disk, so a crash still leaves evidence |
| **ReBAR status** | Whether Resizable BAR is active, plus two bundled driver profiles |
| **Shutdown timer** | Optional: power the machine off after N minutes |

## Requirements

- Windows 10 or 11 (x64)
- An NVIDIA GPU whose driver exposes the clock, power and fan interfaces (Turing and newer
  recommended; older cards may expose fewer domains)
- **Administrator rights** to apply tuning. Monitoring alone works without elevation.
- No .NET runtime installation needed: the release is a single self-contained executable

## Install

1. Download `Nv-Lightning.exe` from the **[releases page](https://github.com/ianEstrada/Nv-Lightning/releases)** (or take it from
   `dist/` in this repository).
2. Put it wherever you like, for example `C:\Tools\Nv-Lightning\`.
3. Run it as administrator the first time (right click → *Run as administrator*).

Nothing else is installed into the system beyond:

| What | Where |
| --- | --- |
| Your profiles | `%ProgramData%\Nv-Lightning\profiles.json` |
| Last applied tuning (for the logon apply) | `HKLM\SOFTWARE\Nv-Lightning` |
| Telemetry ring | `%ProgramData%\Nv-Lightning\telemetry-ring.log` and `-prev.log` |
| Optional logon task | Task Scheduler → `NvLightningUnlock` |

All of it can be removed by hand — see [Troubleshooting](#how-to-remove-everything).

## Quick start

1. Open the app as administrator.
2. Go to **TUNING** and set the offsets you want (core, memory, XBAR, SYS), the power limit and
   the fan mode. Press **Apply All**.
3. Watch the monitoring tiles: they must move to what you asked. If a value does not take, the
   status line tells you instead of leaving you guessing.
4. Open **PROFILES → Save as…**, type a name, and your tuning is stored. Press **Apply** on that
   profile any time to come back to it.
5. If you want it applied at every logon, enable the logon task (the tool offers it once you
   have something applied).

---

## The app, section by section

### LIVE MONITORING — the tiles at the top

Read-only; nothing here changes your GPU.

| Tile | What it shows |
| --- | --- |
| **CORE CLOCK** | The core clock the GPU runs right now (MHz) |
| **MEM CLOCK** | The memory clock right now (MHz) |
| **GPU TEMP** | Core temperature (°C) |
| **GPU UTIL** | How busy the GPU is (%) |
| **POWER** | Current power draw (W) |
| **PWR LIMIT** | The power limit in effect (W) |
| **XBAR** | The live crossbar clock (MHz) — measured, not estimated |

If a value cannot be read, the tile says so instead of showing a made-up number.

### TUNING — where you change things

Nothing is applied until you press a button.

| Control | What it changes |
| --- | --- |
| **Core offset** | Adds or subtracts MHz from the core clock target. Positive = overclock, negative = underclock. |
| **Memory offset** | Same for the memory clock. Memory overclocking is where most of the gains — and most of the instability — live on NVIDIA cards. |
| **XBAR offset** | The crossbar domain, the interconnect inside the GPU. Raising it can help in bandwidth-bound situations. |
| **SYS offset** | The system domain: parts of the GPU outside the core. |
| **Power limit** | The wattage ceiling. Raising it stops early throttling; lowering it cuts heat and noise. |
| **Apply All** | Applies everything you set, in one pass, verifying each value. |
| **Revert** | Puts back what was there before you started changing values. |
| **0 + Apply** | The documented "back to default": `0` means *no offset*, so setting 0 and applying clears the stored tuning. This is the clean way to undo an overclock. |

The panel also shows, in plain words, whether each domain is **ready** or **not available** on
your card, so you know what your GPU exposes before you start.

**How it behaves**

- A value is only reported as applied when it has been **read back** and matches. If the driver
  discards the request, you get a failure message with both numbers.
- If a write would leave the card in a strange state, the tool **reverts** to the previous value
  instead of leaving it half-applied.
- The status line always says what happened. It never reports success for something that failed.

### FAN — cooling behaviour

| Mode | What it does |
| --- | --- |
| **Auto** | Hands the fan back to the card's own curve |
| **Fixed %** | Holds the fan at a percentage you choose |
| **Curve** | You define temperature → speed points, how often to re-check, and how much hysteresis to allow so the fan does not hunt around a threshold |

The fan readback is checked after applying, so you can see whether the card obeyed.

### Shutdown timer

Pick a number of minutes and press **Start** to schedule a shutdown, or **Cancel** to abort it.
Handy for long benchmarks or downloads.

### ReBAR and the bundled driver profiles

The **ReBAR status** line tells you whether Resizable BAR is active and what window size the
driver reports. Resizable BAR lets the CPU address video memory in larger windows; when it is
active some games gain performance. It depends on the motherboard firmware, the GPU's vBIOS and
the driver agreeing, which is why the tool reports it instead of assuming it.

Two NVIDIA Profile Inspector profiles are included with the tool and also live in the `nip/`
folder of this repository:

| File | Profile name | What it is |
| --- | --- | --- |
| `nip/lightning-rebar-base.nip` | **Lightning** | The base profile used alongside the ReBAR work: driver-level rendering and caching settings (shader cache, texture filtering, refresh rate, pre-rendered frames, vertical sync, shader disk cache and more). |
| `nip/Edge.nip` | **Base Profile** | A profile aimed at the Microsoft Edge browser, same family of settings. |

NVIDIA Profile Inspector itself is **bundled inside Nv-Lightning** and launched from there, so
you do not need a separate download. You can also import the `.nip` files into your own copy of
Inspector. These profiles are **not** overclocking: they change driver behaviour, not clocks or
voltages. Test them and keep a copy of your previous profile so you can go back.

### Telemetry and logs

The tool writes one line per second with GPU temperature, power, voltage and clocks, and flushes
it to disk immediately. If the machine ever hangs, the last line on disk is the last state it
reported — that is the evidence. The previous session's file is kept aside instead of being
overwritten, so the crash record survives.

## Profiles

- **You create them.** A fresh install has **zero profiles and zero stored tuning** — nothing is
  shipped with the tool, and the profile list stays empty until you make the first one.
- **Save as…** asks for a **name**. Type a new name to create a profile, or pick an existing
  profile from the list to **overwrite** it — you get a confirmation first, and overwriting
  replaces the values instead of duplicating the entry.
- **Apply** applies every value stored in the profile, verified, and marks it as the tuning to
  replay at logon.
- **Delete** removes the selected profile.
- A profile stores the complete tuning: core, memory, XBAR and SYS offsets, power limit, fan
  mode/percentage/curve, and the voltage lock.
- Profiles live in a single JSON file you can back up or copy to another machine. If that file is
  ever unreadable, the tool does **not** delete it: it parks it as `profiles.json.bad-<timestamp>`
  and starts with an empty list.

## Command line

The app also accepts a few flags — useful for scripting, for the logon task, and for looking at
what happened after a crash. All of them except `--nvapply` are read-only.

| Flag | What it does |
| --- | --- |
| `--diag` | Prints a diagnostic report: whether an NVIDIA GPU was found, which driver interfaces respond, the ReBAR state, and which tuning domains your card exposes. |
| `--nvapply` | Applies the tuning stored in the registry **without opening the window**. This is what the logon task runs. Needs administrator rights. |
| `--profiles-check` | Shows where the profiles file is and prints each profile with its values. No GPU access. |
| `--profiles-selftest` | Self-tests the profile storage in a throwaway folder (never touches your real profiles) and reports OK/FAIL. |
| `--ring-tail [prev] [N]` | Prints the last N lines of the telemetry ring (default 60). Add `prev` to read the **previous** session's file — the one that holds the evidence if the machine crashed. |

```bat
:: what does my card expose, and is ReBAR on?
Nv-Lightning.exe --diag

:: reapply my saved tuning without opening the window (what the logon task runs)
Nv-Lightning.exe --nvapply

:: GPU temperature / power / clocks in the last seconds before a crash
Nv-Lightning.exe --ring-tail prev 200

:: which profiles do I have and what do they contain?
Nv-Lightning.exe --profiles-check
```

Flags are case-insensitive. Exit code `0` means the operation reported success; anything else
means it reported a problem, and the output says which.

## Troubleshooting

### A value does not apply

The status line tells you what happened. The usual causes, in order:

1. **Not running as administrator.** Applying tuning needs elevation. Reading works without it,
   which is why the tiles can look fine while nothing applies.
2. **Another tuning tool is running.** Two utilities writing the same driver state overwrite each
   other. Close everything else.
3. **A driver update changed the interface.** These are undocumented driver interfaces and NVIDIA
   does change them — see [Why this tool](#why-this-tool). Run `--diag` to see which domains your
   card still reports.
4. **The value is outside what your card allows.** The power limit has a real range and offsets
   have practical limits; try a smaller step.

### A tile shows nothing

Some cards do not expose every sensor. If your GPU does not report power draw through the driver,
the tool falls back to another source; if that also returns nothing you get a dash instead of an
invented number. The XBAR tile reads a live measurement — if your card does not expose it, the
tile stays empty. That is a hardware/driver limitation, not a bug.

### The fan does not follow my setting

1. Check the **FAN** mode: in *Auto* the card's own curve wins, by design.
2. Some AIB cards handle the fan in their own vBIOS and do not answer the standard path. The tool
   falls back to a second interface; if your card ignores both, the fan stays under firmware
   control.
3. Very low percentages may be clamped by the card. Try 40–60% to confirm the path works.

### The screen went black or the driver reset after tuning

1. **Reboot.** Nothing the tool writes is stored in the GPU's firmware, so a reboot clears the
   applied tuning.
2. If it happens again at boot because the logon task replays it: disable the `NvLightningUnlock`
   task in Task Scheduler, or start the tool and apply **0** on the values you changed, or apply a
   known-good profile.
3. Redo the tuning in smaller steps and save a profile at the last stable point.

### Where is the evidence after a crash?

```bat
Nv-Lightning.exe --ring-tail prev 200
```

Look at the last lines: a temperature or power spike right before the end points at thermals or
power; lines that simply stop, with nothing unusual, point at a hard lockup instead.

### My profiles disappeared

Check `%ProgramData%\Nv-Lightning\profiles.json`. If the file was unreadable, the tool parked it
next to the original as `profiles.json.bad-<timestamp>` and started empty — nothing is deleted.

### The logon apply did not run

Check the `NvLightningUnlock` task in Task Scheduler: it must be enabled and its path must still
point at the exe (if you moved the file, fix the task or recreate it from the tool). Test it by
hand with `Nv-Lightning.exe --nvapply` as administrator. The task replays **the last tuning you
applied in the UI**, so apply a profile at least once.

### SmartScreen or antivirus complains

The executable is not code-signed (certificates cost money and this is a free tool), so
SmartScreen warns about it. Download it only from this repository's releases.

### How to remove everything

1. Delete `Nv-Lightning.exe` (and its folder).
2. Delete `%ProgramData%\Nv-Lightning` (profiles and telemetry logs).
3. Delete the registry key `HKLM\SOFTWARE\Nv-Lightning`.
4. Delete the `NvLightningUnlock` task in Task Scheduler, if you created it.

Nothing else is installed: no service, no driver, no background process.

## Safety

- Overclocking is never risk-free. Change one thing at a time, apply, and test.
- Save a profile of your working configuration **before** pushing further.
- Watch temperatures and power while you test.
- If the screen goes black or the driver resets, reboot and apply `0` on what you changed, or
  apply a known-good profile.
- Do not run two tuning tools at once.
- Administrator rights are required to apply tuning; monitoring alone does not need them.

## Why this tool

- **Turing and newer.** It was developed and verified on an RTX 2080 — an architecture several
  tuning tools do not cover — and works on newer cards through the same interfaces.
- **The hidden domains.** Core and memory offsets are the easy part. XBAR and SYS are where extra
  headroom lives, and almost nothing exposes them.
- **It tells you the truth.** Every write is verified; failures are reported with both numbers and
  the previous state is restored.
- **It remembers.** Named profiles with create/overwrite, plus an optional apply-at-logon.
- **It leaves evidence.** The telemetry ring survives a hard hang, so "it crashed and I don't know
  why" becomes "here is the last second before it died".

### Inspiration

The direct inspiration is **[mVolt+](https://github.com/b00nz/mVolt)**, which showed that talking
to the driver's own tuning interfaces — instead of a vendor utility — can expose the domains most
tools hide, including XBAR and SYS. mVolt+ targets **Blackwell (RTX 50 series)** and is tested on
RTX 5090 / 5080 / 5070 Ti. Nv-Lightning covers the generations it does not (verified on Turing)
and adds named profiles, fan curves, apply-at-logon, the crash-surviving telemetry ring and the
ReBAR readout.

A caution that comes with this lineage: these are **undocumented driver interfaces**, and NVIDIA
can change or break them with a driver update — that already happened to mVolt+ once
([report](https://www.tomshardware.com/pc-components/gpu-drivers/nvidias-latest-driver-update-breaks-mvolt-overclocking-functionality-nifty-open-source-app-allowed-users-to-increase-the-power-limit-to-700w-on-their-rtx-50-series-gpus-without-hardware-mods)).
If a value that used to apply stops applying after a driver update, that is the reason.

## Credits

- **Lightning** — Nv-Lightning, the XBAR/SYS domain work and the driver research behind it.
- **Succes** — collaboration on the NVIDIA Profile Inspector (NIP) / ReBAR profile research and
  the two `.nip` profiles included here.
- **Orbmu2k** — author of **NVIDIA Profile Inspector**, which is bundled inside the tool and keeps
  its own license. The tool launches it; it does not modify it.

## License

**CC BY-ND 4.0** — free to use and share, **no modified redistributions**, attribution required.

In short: use it as much as you want, on as many machines as you want, even commercially.
Redistribute the unmodified binary with credit. Do not publish a modified version of it without
the author's permission. No warranty: use at your own risk.

The full license text is in [LICENSE.md](LICENSE.md) and governs.

## Release notes

### 1.2.0 — first public release

**Tuning:** core, memory, XBAR and SYS clock offsets verified by readback; power limit; fan
control (auto / fixed / curve with hysteresis); voltage lock including *lowest available point*,
resolved from the card's own table at apply time.

**Monitoring:** live tiles for core, memory, XBAR, temperature, utilisation, power draw and power
limit. Values that cannot be read are shown as unavailable, never guessed.

**Profiles:** created by you, with a name you choose; *Save as…* lets you create or overwrite
(with confirmation); a profile stores the complete tuning. Nothing is shipped with the tool — a
fresh install has zero profiles — and an unreadable profiles file is parked instead of deleted.

**Startup:** optional logon task that reapplies the last tuning you applied.

**Diagnostics:** telemetry ring (one line per second, flushed to disk, previous session kept
aside) and command line flags for diagnostics, headless apply, profile inspection and reading the
ring.

**Other:** ReBAR status readout, two bundled NVIDIA Profile Inspector profiles, optional shutdown
timer.

**Notes:** verified on Turing (RTX 2080); the executable is not code-signed; licensed CC BY-ND 4.0.
