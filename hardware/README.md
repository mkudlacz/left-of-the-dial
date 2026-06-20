# Left of the Dial — Hardware Project

A dedicated streaming radio device to complement the web app at `tuner.redcontroldeck.com`. The phone is the UI. The box plays and displays.

---

## Build Comparison

| Attribute | **Mk.1 — Desktop** | **Mk.2A — Portable** ← committed | **Mk.2B — Portable+** | Mk.3 — Desktop+ | Mk.4 — Statement |
|-----------|-------------------|---------------------------------|----------------------|-----------------|------------------|
| **Status** | Defined | **Committed** | Option | Defined | Defined |
| **Est. Cost** | ~$135 | ~$150 | ~$160 | ~$175 | ~$260 |
| **Display/Audio HAT** | AK2515 VFD | WhisPlay HAT | Pirate Audio Line-Out | AK2515 VFD | AK2515 VFD |
| **Display** | VFD 25×15 hardware spectrum | 1.69" 240×280 color LCD | 1.3" 240×240 color IPS | VFD 25×15 | VFD 25×15 |
| **Spectrum** | Hardware (AK2515) | Software (FFT → color LCD) | Software (FFT → IPS) | Hardware | Hardware |
| **Audio out** | Bluetooth | Bluetooth + onboard speaker | Bluetooth + 3.5mm line | Bluetooth + RCA | Bluetooth + RCA |
| **Speaker** | None (BT only) | Built-in (small) | Separate (add-on) | None | None |
| **Buttons** | Knob only | Knob + 1 button | Knob + 4 tactile | Knob only | Knob only |
| **Battery** | None | PiSugar 3 Plus 5000mAh | PiSugar 3 Plus 5000mAh | None | PiJuice + LiPo |
| **Power** | Wall | Battery + USB-C charge | Battery + USB-C charge | Wall | Battery + Wall |
| **Enclosure complexity** | Low (Hammond + walnut) | Low (display + knob + 1 btn) | Medium (4 btn cutouts) | Low | High (CNC) |
| **Footprint** | ~220×80×30mm | ~80×65×35mm | ~80×65×35mm | ~220×80×30mm | ~200×70×35mm |
| **Best for** | Desk/shelf | Bedside, kitchen, travel | Richer feature set | Receiver pairing | Statement object |

---

## Mk.2A — Portable (Committed)

Battery-powered, self-contained. WhisPlay HAT gives a color LCD, onboard speaker, RGB LED, and one programmable button — all in the Pi Zero footprint. PiSugar 3 Plus attaches via back contact (no GPIO conflict). Knob handles station navigation. Phone handles everything else.

**Why WhisPlay over Pirate Audio:** single button is the right amount for a minimalist enclosure. Four tactile buttons require four cutouts, legends, and caps — enclosure complexity not worth it for a device whose primary UI is the phone.

### Parts List

| # | Component | Purpose | Est. Cost | Source |
|---|-----------|---------|-----------|--------|
| 1 | Raspberry Pi Zero 2W | Compute + WiFi + Bluetooth | $15 | raspberrypi.com |
| 2 | PiSugar WhisPlay HAT | 1.69" color LCD + speaker + RGB LED + button | $36 | pisugar.com |
| 3 | PiSugar 3 Plus 5000mAh | Battery + charging (back contact, no GPIO) | $50 | pisugar.com |
| 4 | KY-040 rotary encoder | Station nav + BT speaker select | $5 | Amazon |
| 5 | Davies Molding 1434 knob | Primary tactile control | $6 | mouser.com |
| 6 | MicroSD card 32GB | Pi OS + software | $8 | Amazon |
| 7 | USB-C wall adapter 5V 3A | Charging | $8 | Amazon |
| 8 | GPIO hammer header | No-solder GPIO attachment | $6 | Adafruit #3662 |
| 9 | Misc (standoffs, wire, screws) | Assembly | $10 | Amazon |
| | **Total (no enclosure)** | | **~$144** | |

### Stack (top to bottom, no conflicts)

```
WhisPlay HAT     ← GPIO header (display, speaker, LED, button)
Pi Zero 2W       ← compute
PiSugar 3 Plus   ← back contact pogo pins (battery, no GPIO use)
```

### Signal Chain

```
Pi Zero 2W
  ├── WiFi ←→ Left of the Dial app (station control)
  ├── Bluetooth → paired BT speaker (primary audio)
  ├── WhisPlay speaker → casual/bedside listening
  └── PulseAudio monitor source → Python FFT → WhisPlay LCD (spectrum)
```

### Knob + Button Interaction

```
PLAY MODE (default)
  LCD: spectrum bars + callsign + freq
  Turn knob → next / previous preset
  Short press (knob) → play / pause
  Long press (knob) → BT SPEAKER MODE
  WhisPlay button → reserved (e.g. toggle display mode)

BT SPEAKER MODE
  LCD: paired device name
  Turn → scroll paired BT devices
  Short press → connect → PLAY MODE
  Long press → cancel → PLAY MODE

IDLE MODE (stopped / no station)
  LCD: clock + last station name
```

### LCD Display Modes

```
PLAYING — software FFT spectrum
  ┌─────────────────┐
  │ WKCR  89.9      │
  │ ▁▃▅▇▅▃▁▂▄▆▇▆▄▂ │
  │ ▁▂▃▄▅▆▇▆▅▄▃▂▁▂ │
  └─────────────────┘
  Amber bars on dark — matches app aesthetic

IDLE — clock
  ┌─────────────────┐
  │    12:47 PM     │
  │   Last: WKCR    │
  └─────────────────┘

BT SELECT
  ┌─────────────────┐
  │ ◄ MARSHALL      │
  │   WILLEN      ► │
  └─────────────────┘
```

### Software — LCD Spectrum

```python
# Core concept — reads PulseAudio monitor, draws to WhisPlay LCD
import sounddevice as sd
import numpy as np
from PIL import Image, ImageDraw
import ST7789  # or equivalent SPI LCD driver

def draw_spectrum(bars, callsign, freq):
    img = Image.new('RGB', (240, 280), (0, 0, 0))
    draw = ImageDraw.Draw(img)
    draw.text((4, 4), f"{callsign}  {freq}", fill=(201, 162, 39))
    for i, val in enumerate(bars[:120]):
        h = int(val / max_val * 240)
        draw.rectangle([i*2, 280-h, i*2+1, 280], fill=(201, 162, 39))
    display.display(img)
```

Full implementation: ~150 lines Python. Libraries: `luma.lcd` or `ST7789`, `sounddevice`, `numpy`, `Pillow`.

---

## Mk.2B — Portable+ (Option)

Same as Mk.2A but uses **Pimoroni Pirate Audio Line-Out** instead of WhisPlay. Adds a 3.5mm stereo line-out (useful for feeding an external spectrum display or wired speaker). Trades built-in speaker and single button for the 3.5mm jack and four tactile buttons.

**Enclosure consideration:** four buttons require four cutouts on the face panel. Could be designed as a vintage car-radio row of chrome-capped buttons — period-correct if done well, but more work than Mk.2A.

**Replaces WhisPlay ($36) with:**
- Pimoroni Pirate Audio Line-Out (~$23) — display + 3.5mm line-out
- Separate 40mm speaker driver (~$4) + Pimoroni Audio Amp SHIM (~$11) if speaker wanted

**Net cost:** ~$10 cheaper, more capable audio chain, more complex enclosure.

---

## Mk.1 — Desktop

Desk/shelf unit. AK2515 VFD handles spectrum in hardware. Wall powered. BT audio to nearby speaker.

**Key issue resolved:** Pi Zero 2W has no native 3.5mm. Audio to AK2515 best handled by a **PCM5102 I2S DAC board** (~$5) on the GPIO header. Tiny board, no USB conflicts, clean analog signal out to AK2515 3.5mm AUX in.

**Parts over Mk.2A:** AK2515 ($36 net swap), PCM5102 DAC ($5), larger Hammond 1455N2202 enclosure. No battery. No WhisPlay.

---

## Mk.3 — Desktop+

Mk.1 + clean RCA out for home receiver pairing.

**Adds over Mk.1:**
- HiFiBerry DAC Zero ($25) replaces PCM5102 — cleaner I²S, proper RCA out
- 3.5mm Y-splitter → AK2515 + RCA jacks
- Machined aluminum knob

---

## Mk.4 — Statement

Mk.3 hardware in a fully custom CNC aluminum + walnut enclosure. Battery added.

**Adds over Mk.3:** PiJuice Zero + LiPo ($57), CNC face plate via SendCutSend (~$50).

---

## Display/Audio HAT Options (all evaluated)

| HAT | Display | Audio out | Speaker | Buttons | Price | Verdict |
|-----|---------|-----------|---------|---------|-------|---------|
| PiSugar WhisPlay | 1.69" 240×280 color | PH2.0 JST only | Built-in | 1 | $36 | **Mk.2A — chosen** |
| Pimoroni Pirate Audio Line-Out | 1.3" 240×240 IPS | 3.5mm stereo line | None | 4 | $23 | Mk.2B option |
| Pimoroni Pirate Audio Mini Speaker | 1.3" 240×240 IPS | Speaker terminals | 1W onboard | 4 | $23 | 4-btn enclosure issue |
| Pimoroni Pirate Audio Headphone Amp | 1.3" 240×240 IPS | 3.5mm headphone | None | 4 | $23 | 4-btn enclosure issue |
| Waveshare WM8960 | None | 3.5mm + speaker | None | None | ~$15 | No display |
| RPi Codec Zero | None | Speaker terminals | None | None | $20 | No display, input-only 3.5mm |

---

## Battery Options (all evaluated)

| Battery | Capacity | Runtime est. | Interface | Price | Verdict |
|---------|----------|-------------|-----------|-------|---------|
| PiSugar 3 Plus 5000mAh | 5000mAh | ~8–10 hrs | Back contact (no GPIO) | $50 | **Mk.2A/2B — chosen** |
| PiSugar 3 1200mAh | 1200mAh | ~2–3 hrs | Back contact | $40 | Too small |
| PiSugar S Plus 5000mAh | 5000mAh | ~8–10 hrs | Back contact | $30 | Check compatibility with WhisPlay stack |
| PiJuice Zero + 3000mAh | 3000mAh | ~5–6 hrs | GPIO (conflicts with HATs) | $57 | Use only if no HAT on GPIO |

**Note:** PiSugar battery attaches via pogo pins to Pi Zero back — zero GPIO conflict regardless of what's on top. PiJuice uses GPIO header — can't stack with WhisPlay or Pirate Audio.

---

## Software Stack (all builds)

| Layer | Technology | Notes |
|-------|-----------|-------|
| OS | Raspberry Pi OS Lite 64-bit | Headless |
| Stream playback | `mpv` | Radio stream URLs |
| Audio routing | `PulseAudio` | BT sink; monitor source for FFT |
| Bluetooth | `bluez` + `pulseaudio-module-bluetooth` | `bluetoothctl` for pairing |
| GPIO | Python + `gpiozero` | Encoder + button + LED |
| Display (Mk.1/3/4) | Audio signal → AK2515 | Hardware spectrum |
| Display (Mk.2A/2B) | `sounddevice` + `numpy` FFT → LCD | Software spectrum, amber-on-black |
| Device server | Python + `asyncio` + `websockets` | WebSocket API for phone app |
| Network discovery | `avahi-daemon` | `leftofthedial.local` on LAN |

---

## Bluetooth Speaker Pairings

| Speaker | Size | Price | Best with |
|---------|------|-------|-----------|
| Marshall Willen | 4.5"×2.7" | $80 | Mk.2A (aesthetic match) |
| Tribit StormBox Micro 2 | 3"×3"×1.5" | $45 | Mk.2A/2B travel use |
| Sony SRS-XB13 | 4"×2.8" cylinder | $60 | Mk.2A kitchen/bath |
| B&O Beosound A1 | 4.7" puck | $250 | Mk.4 |

---

## Open Questions

**Mk.2A (active):**
- [ ] PiSugar S Plus ($30) vs PiSugar 3 Plus ($50) — confirm both use same back-contact pogo pins and are compatible with WhisPlay stack
- [ ] WhisPlay LCD driver — confirm `luma.lcd` or ST7789P3 Python library works on Pi Zero 2W
- [ ] PulseAudio monitor source latency on Pi Zero 2W — FFT must stay <50ms for responsive spectrum
- [ ] Knob encoder connects to which GPIO pins when WhisPlay occupies header — check WhisPlay pinout for passthrough
- [ ] Pi Zero 2W GPIO header needs soldering — use hammer header (Adafruit #3662) for no-solder install
- [ ] Prototype before enclosure — verify stack physically fits and software works

**Mk.1 (backlog):**
- [ ] PCM5102 I2S DAC board — confirm Pi Zero 2W GPIO passthrough available when stacking
- [ ] AK2515 exact board dimensions for face cutout

---

## Mk.2A Shopping List (no enclosure)

Ready to order. Buy Pi + WhisPlay + PiSugar from their respective makers; everything else from Amazon.

| Item | Where | ~Price |
|------|-------|--------|
| Raspberry Pi Zero 2W | raspberrypi.com or Adafruit | $15 |
| PiSugar WhisPlay HAT | pisugar.com | $36 |
| PiSugar 3 Plus 5000mAh | pisugar.com (same order) | $50 |
| KY-040 rotary encoder | Amazon | $5 |
| Davies Molding 1434 knob (prototype) | mouser.com | $6 |
| MicroSD 32GB (Samsung Endurance) | Amazon | $8 |
| USB-C wall adapter 5V 3A | Amazon | $8 |
| GPIO hammer header | Adafruit #3662 | $6 |
| Female-to-female jumper wires | Amazon | $6 |
| MicroSD USB reader | Amazon | $8 |
| Misc (resistors, standoffs, wire) | Amazon | $10 |
| **Total** | | **~$158** |

---

## Reference

- App: `tuner.redcontroldeck.com` · repo: `github.com/mkudlacz/left-of-the-dial`
- PiSugar WhisPlay: pisugar.com/products/whisplay-hat-for-pi-zero-2w-audio-display
- PiSugar 3 Plus: pisugar.com/products/pisugar-3-raspberry-pi-zero-battery
- PiSugar S Plus: pisugar.com/collections/all
- Pimoroni Pirate Audio Line-Out: shop.pimoroni.com/en-us/products/pirate-audio-line-out
- Pimoroni Audio Amp SHIM: shop.pimoroni.com/en-us/products/audio-amp-shim-3w-mono-amp
- Adafruit hammer header #3662: adafruit.com/product/3662
- luma.lcd docs: luma-lcd.readthedocs.io
- Hammond 1455N2202 (Mk.1/3): mouser.com
- HiFiBerry DAC Zero (Mk.3/4): hifiberry.com
- SendCutSend (Mk.4 CNC): sendcutsend.com
- Marshall Willen: marshallheadphones.com
