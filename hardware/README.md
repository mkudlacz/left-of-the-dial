# Left of the Dial — Hardware Project

A dedicated streaming radio device to complement the web app at `tuner.redcontroldeck.com`. The phone is the UI. The box plays and displays.

---

## Build Comparison

| Attribute | **Mk.1 — Desktop** | **Mk.2A — Portable** | **Mk.2B — Portable+** | **Mk.2C — Field Radio** ← canonical | Mk.3 — Desktop+ | Mk.4 — Statement |
|-----------|-------------------|--------------------|----------------------|-------------------------------------|-----------------|------------------|
| **Status** | Defined | Defined | Option | **Build this one** | Defined | Defined |
| **Est. Cost** | ~$135 | ~$150 | ~$160 | ~$175 | ~$175 | ~$260 |
| **Display/Audio HAT** | AK2515 VFD | WhisPlay HAT | Pirate Audio Line-Out | WhisPlay HAT + soldered 3.5mm | AK2515 VFD | AK2515 VFD |
| **Display** | VFD 25×15 | 1.69" 240×280 color | 1.3" 240×240 IPS | 1.69" 240×280 color | VFD 25×15 | VFD 25×15 |
| **Spectrum** | Hardware | Software FFT | Software FFT | Software FFT | Hardware | Hardware |
| **Audio out** | Bluetooth | BT + speaker | BT + 3.5mm | **BT + speaker + 3.5mm** | BT + RCA | BT + RCA |
| **Speaker** | None | Built-in (small) | Separate | **Built-in + panel 3.5mm jack** | None | None |
| **3.5mm jack** | None | None | Yes (line) | **Yes (headphone/line — soldered mod)** | RCA | RCA |
| **Buttons** | Knob only | Knob + 1 btn | Knob + 4 tactile | Knob + 1 btn | Knob only | Knob only |
| **Knob** | Davies plastic | Davies plastic | Davies plastic | **Big machined aluminum (30–35mm)** | Machined | Machined |
| **Battery** | None | PiSugar 3 Plus | PiSugar 3 Plus | **PiSugar 3 Plus 5000mAh** | None | PiJuice + LiPo |
| **Power** | Wall | Battery + USB-C | Battery + USB-C | Battery + USB-C | Wall | Battery + Wall |
| **Enclosure shape** | Landscape | Portrait/puck | Portrait | **Portrait — transistor radio form** | Landscape | Landscape |
| **Footprint** | ~220×80×30mm | ~80×65×35mm | ~80×65×35mm | ~90×65×40mm | ~220×80×30mm | ~200×70×35mm |
| **Soldering required** | No | No | No | **Yes — 3 pads on WM8960** | No | No |
| **Best for** | Desk/shelf | Bedside, kitchen | Richer audio chain | **Deck, travel, work, earphones** | Receiver pairing | Statement object |

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

### GPIO Pin Map (verified from WhisPlay source)

**Pins used by WhisPlay:**

| Board Pin | BCM GPIO | Function |
|-----------|----------|----------|
| 3, 5 | GPIO2, GPIO3 | I²C — WM8960 audio control |
| 7 | GPIO4 | LCD RST |
| 11 | GPIO17 | Button |
| 12, 35, 38, 40 | GPIO18, GPIO19, GPIO20, GPIO21 | I²S — WM8960 audio data |
| 13 | GPIO27 | LCD DC |
| 15 | GPIO22 | LCD backlight |
| 16 | GPIO23 | RGB LED blue |
| 18 | GPIO24 | RGB LED green |
| 19, 21, 23, 24, 26 | SPI0 | LCD data |
| 22 | GPIO25 | RGB LED red |

**Free pins — confirmed available for rotary encoder:**

| Board Pin | BCM GPIO | Assign to |
|-----------|----------|-----------|
| **29** | GPIO5 | Encoder CLK |
| **31** | GPIO6 | Encoder DT |
| **32** | GPIO12 | Encoder SW (button press) |
| 33 | GPIO13 | Spare |
| 36 | GPIO16 | Spare |
| 37 | GPIO26 | Spare |

Encoder wires to board pins 29, 31, 32 — no conflicts with WhisPlay whatsoever. Verified against `whisplay.py` source.

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

## Mk.2C — Field Radio (Canonical Build)

The one to build. **No soldering required.** External speaker via PH2.0 JST, Bluetooth for AirPods and wireless speakers. Battery all day. Big knob. Landscape orientation like the CCrane — sits on a table, holds in a hand, sounds good either way.

**Design references:** CCrane Solar Observer (form factor, speaker-left layout) + iPod nano (knob as dominant UI element, screen as informational, restrained controls).

**No 3.5mm jack in v1.** Bluetooth covers AirPods and wireless earphones. 3.5mm can be added in a future revision with help.

**AirPlay / HomeKit note:** AirPods pair directly via standard Bluetooth A2DP — works great. HomeKit audio is not supported (requires Apple MFi certification). AirPlay 2 sending is not available on Pi.

### Parts List

| # | Component | Purpose | Est. Cost | Source |
|---|-----------|---------|-----------|--------|
| 1 | Raspberry Pi Zero 2W | Compute + WiFi + Bluetooth | $15 | raspberrypi.com |
| 2 | PiSugar WhisPlay HAT | 1.69" color LCD + WM8960 + RGB LED + button | $36 | pisugar.com |
| 3 | PiSugar 3 Plus 5000mAh | Battery (back contact, no GPIO conflict) | $50 | pisugar.com |
| 4 | 50mm full-range speaker driver 4–8Ω | External speaker via PH2.0 JST | $4–6 | Amazon "50mm speaker 4 ohm" |
| 5 | KY-040 rotary encoder | Station nav + BT device select | $5 | Amazon |
| 6 | Machined aluminum knob 30–35mm 6mm D-shaft | The dominant UI element | $10–15 | Amazon or Rean |
| 7 | GPIO hammer header | No-solder GPIO install | $6 | Adafruit #3662 |
| 8 | MicroSD card 32GB | Pi OS + software | $8 | Amazon |
| 9 | USB-C wall adapter 5V 3A | Charging | $8 | Amazon |
| 10 | Short PH2.0 JST cable | WhisPlay → speaker driver | $2 | Amazon |
| 11 | Female-to-female jumper wires | Encoder → GPIO | $6 | Amazon |
| 12 | MicroSD USB reader | Writing Pi OS image | $8 | Amazon |
| 13 | Misc (standoffs, wire, screws) | Assembly | $10 | Amazon |
| | **Total (no enclosure)** | | **~$168–173** | |

### Stack (top to bottom, zero soldering)

```
WhisPlay HAT      ← GPIO header
  └── PH2.0 JST ──────────────→ 50mm speaker driver (mounted in enclosure)
Pi Zero 2W        ← compute
PiSugar 3 Plus    ← back contact pogo pins (battery)
  └── USB-C ─────────────────→ charging port (enclosure wall)

Encoder CLK/DT/SW → jumper wires → board pins 29/31/32
```

### Audio Routing

| Output | How | Use case |
|--------|-----|---------|
| Enclosure speaker | PH2.0 JST → 50mm driver | Table, deck, kitchen |
| AirPods | Bluetooth A2DP | Wireless earphones |
| BT speaker | Bluetooth A2DP | Marshall Willen, Tribit, etc. |

### Enclosure — Landscape Field Radio

**References:** CCrane Solar Observer (speaker-left landscape) + iPod mini (knob proportionally dominant, compact refined body)
**Orientation:** Landscape — sits on a table, comfortable handheld

```
  ┌────────────────────────────────────────────┐  ~130mm wide
  │                                            │  ~ 75mm tall
  │  ┌──────────────┐  ┌─────────┐            │
  │  │              │  │  WKCR   │   ╭─────╮  │
  │  │  ░ ░ ░ ░ ░   │  │  89.9   │   │     │  │
  │  │  ░ ░ ░ ░ ░   │  │  ▁▃▇▃▁  │   │  ◎  │  │ ← 35mm knob
  │  │  ░ ░ ░ ░ ░   │  └─────────┘   │     │  │
  │  └──────────────┘      · btn      ╰─────╯  │
  │   50mm speaker    LCD landscape             │
  └────────────────────────────────────────────┘
  walnut                                  walnut
  end-cap                                 end-cap

  BOTTOM: USB-C (centered, hidden when on table)
  BACK: plain
```

**Dimensions:**
- **Width:** 130mm (~5.1") — between CCrane and iPod mini proportionally
- **Height:** 75mm (~3")
- **Depth:** 40mm (stack ~23mm + speaker driver + clearance)

**Face zones (left → right):**
- 8mm margin
- 50mm speaker grille (circular punched holes, 50mm driver behind)
- 12mm gap
- 45mm LCD window (WhisPlay, landscape-rotated)
- 10mm gap
- 35mm knob (proportionally dominant — the iPod mini move)
- 8mm margin, button small + flush below knob

**Enclosure candidate:** Hammond 1455L1301 (~130×76mm) or measure stack and pick closest.
Walnut end-caps left and right. Cutouts in aluminum face: speaker grille, LCD window, knob hole.

### Display Rotation

WhisPlay LCD is 240×280 portrait by default. In landscape enclosure, set `USE_HORIZONTAL = 0` or `2` in `whisplay.py` to rotate 90°. Display becomes 280×240 — slightly wider than tall, perfect for spectrum bars.

```python
USE_HORIZONTAL = 0  # landscape — change from default 1
```

### GPIO Assignments (confirmed free from WhisPlay source)

| Board Pin | BCM | Function |
|-----------|-----|----------|
| 29 | GPIO5 | Encoder CLK |
| 31 | GPIO6 | Encoder DT |
| 32 | GPIO12 | Encoder SW |
| 33, 36, 37 | GPIO13/16/26 | Spare |

### Knob Interaction

```
PLAY
  Turn → next / previous preset
  Short press → play / pause
  Long press → BT DEVICE MODE

BT DEVICE MODE
  LCD: paired device name
  Turn → scroll devices
  Short press → connect → PLAY
  Long press → cancel

IDLE
  LCD: clock + last station
```

---

## Mk.2A — Portable (Simpler Option)

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

**Mk.2C (canonical — active):**
- [x] Encoder GPIO confirmed free: board pins 29/31/32 (BCM 5/6/12) — verified against whisplay.py source
- [ ] Locate WM8960 HPOUTL/HPOUTR/GND pads on WhisPlay PCB — confirm accessible before ordering
- [ ] PiSugar 3 Plus vs S Plus — confirm back-contact pogo pin compatibility with WhisPlay stack
- [ ] WhisPlay LCD driver — whisplay.py (PiSugar GitHub) is the library; confirm runs on Pi OS Lite Zero 2W
- [ ] PulseAudio FFT latency — must stay <50ms on Zero 2W for responsive spectrum display
- [ ] Hammond enclosure — measure full stack height before ordering (Pi ~5mm + WhisPlay ~10mm + PiSugar ~8mm + tolerance)
- [ ] WM8960 jack-detect auto-mute — configure via amixer; test earphone insert/remove behavior
- [ ] Always prototype before cutting enclosure

**Mk.1 (backlog):**
- [ ] PCM5102 I2S DAC board GPIO passthrough when stacking
- [ ] AK2515 exact board dimensions for face cutout

---

## Mk.2C Shopping List (canonical build, no enclosure)

Buy Pi + WhisPlay + PiSugar from their respective makers in one pass. Everything else Amazon.

| Item | Where | ~Price |
|------|-------|--------|
| Raspberry Pi Zero 2W | raspberrypi.com or Adafruit | $15 |
| PiSugar WhisPlay HAT | pisugar.com | $36 |
| PiSugar 3 Plus 5000mAh | pisugar.com (same order) | $50 |
| KY-040 rotary encoder | Amazon | $5 |
| Machined aluminum knob 30–35mm 6mm D-shaft | Amazon "6mm D shaft aluminum knob large" or Rean | $10–15 |
| Panel-mount 3.5mm stereo jack | Amazon "3.5mm panel mount stereo jack" | $2 |
| GPIO hammer header | Adafruit #3662 | $6 |
| MicroSD 32GB (Samsung Endurance) | Amazon | $8 |
| USB-C wall adapter 5V 3A | Amazon | $8 |
| Female-to-female jumper wires | Amazon | $6 |
| MicroSD USB reader | Amazon | $8 |
| Misc (standoffs, 28AWG wire, solder, screws) | Amazon | $12 |
| **Total** | | **~$166–171** |

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
