# Left of the Dial — Hardware Project

A dedicated streaming radio device to complement the web app at `tuner.redcontroldeck.com`. The phone is the UI. The box plays and displays.

---

## Build Comparison

| Attribute | **Mk.1 — Desktop** | **Mk.2 — Portable** ← focus | Mk.3 — Desktop+ | Mk.4 — Statement |
|-----------|-------------------|------------------------------|-----------------|------------------|
| **Status** | Defined | **In development** | Defined | Defined |
| **Est. Cost** | ~$135 | ~$145 | ~$175 | ~$260 |
| **Display** | AK2515 VFD (25×15) | 1.3" OLED 128×64 | AK2515 VFD (25×15) | AK2515 VFD (25×15) |
| **Spectrum** | Hardware (AK2515) | Software (PulseAudio FFT → OLED) | Hardware (AK2515) | Hardware (AK2515) |
| **Audio out** | Bluetooth | Bluetooth | Bluetooth + RCA | Bluetooth + RCA |
| **Power** | Wall (USB-C) | Battery + Wall | Wall (USB-C) | Battery + Wall |
| **Battery** | None | PiJuice Zero + 3000mAh LiPo | None | PiJuice Zero + LiPo |
| **Enclosure** | Hammond 1455 + walnut | Small Hammond + walnut | Hammond 1455 + walnut | CNC aluminum + walnut |
| **Footprint** | ~220×80×30mm | ~100×60×30mm | ~220×80×30mm | ~200×70×35mm |
| **Knob** | Davies Molding plastic | Davies Molding plastic | Machined aluminum | Machined aluminum |
| **Best for** | Desk/shelf next to speakers | Travel, kitchen, bedside | Home receiver pairing | Statement object |

---

## Mk.2 — Portable (In Development)

A battery-powered streaming puck. Bluetooth audio to any paired speaker. 1.3" OLED shows a live software spectrum + callsign while playing, clock while idle. Fits in a coat pocket. No cables during use.

### Key Insight — Software Spectrum

The Pi doesn't need a physical audio input for the spectrum display. PulseAudio creates a **monitor source** — a software loopback of whatever `mpv` is playing. Python reads from this, runs FFT, draws bars to the OLED. No audio cables, no dongles, no extra hardware. Fully internal.

### Parts List

| # | Component | Purpose | Est. Cost | Source |
|---|-----------|---------|-----------|--------|
| 1 | Raspberry Pi Zero 2W | Compute + WiFi + Bluetooth | $15 | raspberrypi.com |
| 2 | PiJuice Zero | LiPo battery management HAT | $45 | pi-supply.com |
| 3 | LiPo 3000mAh (PiJuice compatible) | ~5–6 hrs runtime | $12 | pi-supply.com |
| 4 | 1.3" OLED SH1106 128×64 | Spectrum + status display | $8 | Amazon |
| 5 | KY-040 rotary encoder | Station nav + BT speaker select | $5 | Amazon |
| 6 | Davies Molding 1434 knob | Tactile control | $6 | mouser.com |
| 7 | MicroSD card 32GB | Pi OS + software | $8 | Amazon |
| 8 | USB-C wall adapter 5V 3A | Charging | $8 | Amazon |
| 9 | Hammond 1455K1201 (smaller) | Aluminum extrusion body ~120×51×31mm | $15 | mouser.com |
| 10 | Walnut (~4"×3"×¾") | End-caps | $8 | local hardwood |
| 11 | 5mm LED | Status indicator (amber/green/red) | $1 | Amazon |
| 12 | Misc (M2.5 standoffs, wire, screws, NPN transistor for LED) | Assembly | $10 | Amazon |
| | **Total** | | **~$141** | |

### Signal Chain

```
Pi Zero 2W
  ├── WiFi ←→ Left of the Dial app (station control)
  ├── Bluetooth → paired BT speaker (audio)
  └── PulseAudio monitor source → Python FFT → OLED (spectrum display)
```

### GPIO Wiring

| GPIO | Pin | Connected to |
|------|-----|--------------|
| GPIO2 (SDA) | 3 | OLED SDA + PiJuice SDA (I²C shared bus) |
| GPIO3 (SCL) | 5 | OLED SCL + PiJuice SCL (I²C shared bus) |
| 3.3V | 1 | OLED VCC |
| GND | 6 | OLED GND |
| GPIO17 | 11 | Encoder CLK |
| GPIO18 | 12 | Encoder DT |
| GPIO27 | 13 | Encoder SW (button) |
| GPIO22 | 15 | LED (via 330Ω resistor + NPN transistor) |

PiJuice I²C address: `0x14`. OLED address: `0x3C` or `0x3D`. No conflict.

### OLED Display Modes

```
PLAY MODE (streaming)
  Top line: WKCR  89.9
  Below: spectrum bars (software FFT from PulseAudio monitor)

IDLE MODE (stopped)
  Top: current time HH:MM
  Bottom: last station or "Left of the Dial"

BT SPEAKER MODE (knob long press)
  Scrolls: "MARSHALL WILLEN"
  Turn → next paired device
  Press → connect, return to PLAY

ERROR MODE
  "STREAM ERROR"
  last station name
```

### Software — OLED Spectrum

```python
# Rough sketch — core loop
import sounddevice as sd
import numpy as np
from luma.oled import device as oled_device
from luma.core.render import canvas

# Read from PulseAudio monitor source (loopback of mpv output)
def audio_callback(indata, frames, time, status):
    fft = np.abs(np.fft.rfft(indata[:, 0], n=256))
    bars = fft[:64]  # 64 frequency bins → 64 OLED columns
    draw_spectrum(bars)

# Draw to OLED
def draw_spectrum(bars):
    with canvas(display) as draw:
        draw.text((0, 0), current_callsign, fill='white')
        for i, val in enumerate(bars):
            h = int(val / max_val * 52)  # 52px height for bars
            draw.rectangle([i*2, 64-h, i*2+1, 64], fill='white')
```

Full implementation: ~100–150 lines of Python. Libraries: `luma.oled`, `sounddevice`, `numpy`.

### Enclosure Notes

Hammond 1455K1201 is 120×51×31mm — about half the width of Mk.1's 1455N2202. Removable plastic end-caps replaced with walnut. Front face openings:
- OLED window (~35mm×18mm cutout, centered left)
- Encoder hole (right side)
- LED hole (top-right corner)

Back:
- USB-C (charging only)

At ~120mm wide the whole thing is smaller than a deck of cards in two dimensions.

---

## Mk.1 — Desktop

Desk/shelf unit. AK2515 handles spectrum display in hardware — no software FFT needed. Wall powered. Bluetooth audio to nearby speaker.

**Parts over Mk.2:** AK2515 ($50) replaces OLED ($8). No battery. Larger Hammond 1455N2202 enclosure.

**AK2515 power:** Internal USB-A → USB-C pigtail, tapped from the same 5V supply rail as the Pi.

**AK2515 audio:** Pi Zero 2W has no native 3.5mm jack. Options in order of preference:
1. PCM5102 I2S DAC board (~$5 on GPIO) → 3.5mm → AK2515. Clean signal, small board.
2. USB audio dongle via OTG adapter → 3.5mm → AK2515. No soldering but occupies OTG port.

AK2515 only needs signal for visualization — audio quality is irrelevant.

---

## Mk.3 — Desktop+

Adds clean RCA out for home receiver pairing. Otherwise identical to Mk.1.

**Adds over Mk.1:**
- HiFiBerry DAC Zero (+$25) — clean I²S DAC; replaces PCM5102 workaround
- 3.5mm Y-splitter — DAC → AK2515 + RCA jacks
- Panel-mount RCA jacks L/R (+$5)
- Machined aluminum knob (+$8)

---

## Mk.4 — Statement

Battery-powered desktop object. Mk.3 hardware in a custom CNC aluminum + walnut enclosure.

**Adds over Mk.3:**
- PiJuice Zero + LiPo (+$57)
- CNC aluminum face plate via SendCutSend (+$40–60 over Hammond)

---

## Knob Interaction (all builds)

```
PLAY MODE
  Turn → next / previous preset
  Short press → play / pause
  Long press → BT SPEAKER MODE

BT SPEAKER MODE
  Display: paired device name
  Turn → scroll paired BT devices
  Short press → connect → PLAY MODE
  Long press → cancel → PLAY MODE
```

---

## Software Stack (all builds)

| Layer | Technology | Notes |
|-------|-----------|-------|
| OS | Raspberry Pi OS Lite 64-bit | Headless |
| Stream playback | `mpv` | Radio stream URLs |
| Audio routing | `PulseAudio` | BT sink; monitor source for Mk.2 FFT |
| Bluetooth | `bluez` + `pulseaudio-module-bluetooth` | `bluetoothctl` for pairing |
| GPIO | Python + `gpiozero` | Encoder + LED |
| Display (Mk.1/3/4) | PulseAudio → AK2515 audio in | Hardware spectrum |
| Display (Mk.2) | `luma.oled` + `sounddevice` + `numpy` | Software FFT → OLED |
| Device server | Python + `asyncio` + `websockets` | WebSocket API for phone app |
| Network discovery | `avahi-daemon` | `leftofthedial.local` on LAN |

---

## Bluetooth Speaker Pairings

| Speaker | Size | Price | Notes |
|---------|------|-------|-------|
| Marshall Willen | 4.5"×2.7" | $80 | Vintage aesthetic; good with Mk.1/3/4 |
| Tribit StormBox Micro 2 | 3"×3"×1.5" | $45 | Best portable companion for Mk.2 |
| Sony SRS-XB13 | 4"×2.8" cylinder | $60 | IP67; kitchen/bathroom use |
| B&O Beosound A1 | 4.7" puck | $250 | Premium; pairs with Mk.4 aesthetically |

---

## Open Questions

**Mk.2 (active):**
- [ ] Confirm Hammond 1455K1201 fits Pi Zero 2W + PiJuice stack (check clearance: ~25mm internal height)
- [ ] PiJuice Zero + OLED I2C address conflict check (should be fine: 0x14 vs 0x3C)
- [ ] Knob press-and-turn for volume, or leave volume to BT speaker?
- [ ] Pi Zero 2W GPIO header needs soldering — consider Pi 3A+ for prototyping (standard USB-A, soldered GPIO)
- [ ] Test PulseAudio monitor source latency on Pi Zero 2W — FFT needs to stay <50ms for responsive spectrum

**Mk.1 (backlog):**
- [ ] PCM5102 vs USB audio dongle for AK2515 signal — test both
- [ ] Confirm AK2515 exact board dimensions

---

## Reference

- App: `tuner.redcontroldeck.com` · repo: `github.com/mkudlacz/left-of-the-dial`
- Pi Zero 2W: raspberrypi.com
- PiJuice Zero: uk.pi-supply.com/products/pijuice-zero
- 1.3" OLED SH1106: search "1.3 inch OLED SH1106 I2C" on Amazon (~$8)
- luma.oled library: luma-oled.readthedocs.io
- Hammond 1455K1201 (Mk.2): mouser.com
- Hammond 1455N2202 (Mk.1/3): mouser.com
- HiFiBerry DAC Zero (Mk.3/4): hifiberry.com
- SendCutSend (Mk.4 CNC face): sendcutsend.com
- Marshall Willen: marshallheadphones.com
