# Left of the Dial — Hardware Project

A dedicated streaming radio device to complement the web app at `tuner.redcontroldeck.com`. The phone is the UI. The box plays and glows.

---

## Concept

A small, focused audio object — not a computer with a screen. The physical device streams radio via WiFi, outputs audio to speakers, and shows a VFD spectrum display while playing. All station selection, preset management, browsing, and settings happen in the Left of the Dial web app on your phone. The box has one knob and glows.

---

## Architecture

```
iPhone (Left of the Dial app)
  └── WiFi → Pi WebSocket API → mpv audio stream
                                    └── HiFiBerry DAC
                                          ├── 3.5mm Y-splitter
                                          │     ├── AK2515 (spectrum display)
                                          │     └── Powered speakers / amp
                                          └── Bluetooth → wireless speaker (optional)
```

**Rotary encoder** on the box → Python reads GPIO → sends `navigateStation(+1/-1)` to the streaming layer → plays next/previous preset.

---

## Parts List

| # | Component | Purpose | Est. Cost | Notes |
|---|-----------|---------|-----------|-------|
| 1 | Raspberry Pi Zero 2W | Headless compute + WiFi + Bluetooth | $15 | Tiny, fanless, sufficient for audio streaming |
| 2 | HiFiBerry DAC Zero | Quality analog audio output (I²S DAC HAT) | $25 | Stacks directly on Pi Zero 2W |
| 3 | Nobsound AK2515 | VFD spectrum analyzer display | $50 | 25×15 VFD, USB-C powered, 3.5mm AUX in, shows spectrum + clock |
| 4 | KY-040 Rotary Encoder module | Physical station navigation knob | ~$5 | Reads via Pi GPIO, skip presets |
| 5 | Machined aluminum knob | The tactile soul of the device | ~$15–20 | Elma or Davies Molding; knurled grip; buy before designing hole |
| 6 | MicroSD card (32GB+) | Pi OS + software | ~$8 | Samsung or SanDisk |
| 7 | USB-C power supply (5V 3A) | Powers Pi (and Pi powers AK2515 via USB) | ~$10 | Single cable in |
| 8 | 3.5mm Y-splitter cable | Split Pi audio to AK2515 + speakers | ~$5 | Pi DAC → splitter → AK2515 input + speaker amp input |
| 9 | Walnut (end-caps) | Enclosure sides | ~$20 | ~¾" thick American black walnut; two pieces |
| 10 | Aluminum face plate | Front face with cutouts | ~$40–60 | 6061 aluminum, 0.125" thick; CNC via SendCutSend |
| 11 | Acrylic window (optional) | Protect AK2515 VFD | ~$5 | Clear or lightly smoked; fits in face cutout |
| 12 | Misc (wire, standoffs, screws, thermal paste) | Internal assembly | ~$15 | M2.5 standoffs for Pi/DAC stack |

**Estimated total (excl. speakers):** ~$215–235

---

## Enclosure Design

**Target dimensions:** ~9"W × 3"H × 4"D  
*(Driven by AK2515 board size — measure board first, design around it)*

**Front face layout (left → right):**
- AK2515 VFD window (dominant element, left-center)
- Small LED indicator (top-right corner, mirrors app LED state)
- Rotary encoder knob (right side)

**Material hierarchy:**
- Walnut end-caps (left and right sides, ~¾" thick)
- Brushed 6061 aluminum face plate (inset between walnut caps, like SA-101)
- Face plate slightly recessed from walnut edges — the shadow line reads as intentional

**Back panel:**
- USB-C power in
- 3.5mm or RCA audio out
- Nothing else visible

**Key design constraint:** Buy the aluminum knob first. Design the encoder hole to fit it precisely. The knob is the primary touch point and determines perceived quality.

---

## Audio Output Options

**Wired (primary):**
- HiFiBerry DAC Zero → 3.5mm out → powered bookshelf speakers or stereo receiver
- Quality analog signal, lowest latency

**Bluetooth (secondary):**
- Pi Zero 2W has built-in Bluetooth
- `pulseaudio` + `bluetoothctl` on Pi OS routes audio to any paired BT speaker
- Good for portable/wireless setup

**AirPlay:**
- Pi can run `shairport-sync` to RECEIVE AirPlay from iPhone (Pi becomes the speaker)
- Pi cannot easily be a certified AirPlay 2 SENDER
- Alternative: `owntone` server for sending to AirPlay receivers

---

## Software Stack (Pi)

| Component | Technology | Notes |
|-----------|-----------|-------|
| OS | Raspberry Pi OS Lite (64-bit) | Headless, no desktop |
| Audio playback | `mpv` or `vlc` | Stream radio URLs directly |
| Audio routing | `PulseAudio` | Handles DAC + Bluetooth routing |
| GPIO (encoder) | Python + `RPi.GPIO` or `gpiozero` | Reads encoder ticks |
| Device API | Python + `websockets` | Receives commands from phone app |
| AK2515 power | USB from Pi | Pi USB port → AK2515 USB-C |
| AK2515 audio | 3.5mm Y-splitter | Feeds spectrum from audio signal |

**Python server (~100 lines):**
- Reads rotary encoder → fires `next`/`prev` preset commands
- Exposes WebSocket endpoint on local network
- Receives station URL from app → passes to `mpv`
- Updates LED state via GPIO

**Left of the Dial app (future addition):**
- Local network discovery (mDNS/Bonjour via `avahi` on Pi)
- When detected, "Cast to device" option appears in app
- Tap station → sends stream URL to Pi WebSocket → Pi plays it

---

## Open Questions

- [ ] Exact AK2515 board dimensions (measure before designing face cutout)
- [ ] Speaker choice — powered bookshelf recommendation TBD
- [ ] Whether to include a volume knob (potentiometer via MCP3008 ADC, or second encoder)
- [ ] Case construction method: CNC, hand-cut, or hybrid
- [ ] LED wiring: 5mm through-hole LED on face, GPIO-controlled via transistor
- [ ] Pi Zero 2W vs Pi 3A+ (3A+ has full-size USB, easier for peripherals during dev)

---

## Reference

- Web app: `tuner.redcontroldeck.com`
- App repo: `github.com/mkudlacz/left-of-the-dial`
- AK2515 product: Nobsound/Douk Audio, Amazon ASIN lookup "AK2515"
- SendCutSend (aluminum face plate): sendcutsend.com
- Elma knobs: elma.com (search "21 series" rotary knobs)
- HiFiBerry DAC Zero: hifiberry.com/shop/boards/hifiberry-dac-zero
