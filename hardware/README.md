# Left of the Dial — Hardware Project

A dedicated streaming radio device to complement the web app at `tuner.redcontroldeck.com`. The phone is the UI. The box plays, glows, and pairs to any Bluetooth speaker.

---

## Concept

A small, focused audio object. Streams radio via WiFi, outputs audio via Bluetooth to any paired speaker, and shows a VFD spectrum display while playing. All station selection, browsing, and settings happen in the Left of the Dial web app on your phone. The knob navigates presets and selects Bluetooth speakers. The box has a battery — no cables during use.

---

## Architecture

```
iPhone (Left of the Dial app)
  └── WiFi → Pi WebSocket API → mpv audio stream
                                    └── PulseAudio
                                          ├── Bluetooth → paired BT speaker (audio)
                                          └── Pi onboard audio → AK2515 3.5mm in (spectrum display only)
```

**Knob interactions:**
- Turn = next / previous preset
- Long press = enter Bluetooth speaker select mode
- In BT mode: turn scrolls paired devices on VFD, press connects, long press returns to play

---

## Parts List

| # | Component | Purpose | Est. Cost | Notes |
|---|-----------|---------|-----------|-------|
| 1 | Raspberry Pi Zero 2W | Headless compute + WiFi + Bluetooth 4.2 | $15 | Built-in BT |
| 2 | HiFiBerry DAC Zero | Clean analog audio (I²S DAC HAT) | $25 | Required for RCA out; stacks on Pi Zero; also feeds AK2515 |
| 3 | PiJuice Zero | LiPo battery + charging management HAT | $45 | Stacks on Pi Zero form factor; 3000mAh cell recommended |
| 4 | LiPo battery 3000mAh | ~5–6 hrs runtime | ~$12 | Compatible with PiJuice Zero connector |
| 5 | Nobsound AK2515 | VFD spectrum analyzer display | $50 | 25×15 VFD, USB-C powered, 3.5mm AUX in |
| 6 | KY-040 Rotary Encoder module | Station nav + BT speaker select | ~$5 | Button built in; long press triggers BT menu |
| 7 | Machined aluminum knob | Primary tactile control | ~$15–20 | Buy first — design hole around it |
| 8 | MicroSD card (32GB+) | Pi OS + software | ~$8 | Samsung or SanDisk |
| 9 | 3.5mm Y-splitter (internal) | HiFiBerry out → AK2515 + RCA board | ~$3 | Splits clean DAC signal to both |
| 10 | 3.5mm to RCA board/cable | Internal DAC → rear RCA jacks | ~$5 | Short internal cable to panel-mount RCA jacks |
| 11 | Panel-mount RCA jacks (pair) | Rear audio output | ~$5 | Flush-mount in back panel |
| 12 | Walnut (end-caps) | Enclosure sides | ~$20 | ~¾" thick American black walnut |
| 13 | Aluminum face plate | Front face with cutouts | ~$40–60 | 6061 aluminum, 0.125"; CNC via SendCutSend |
| 14 | Acrylic window | Protect AK2515 VFD | ~$5 | Clear or lightly smoked |
| 15 | Misc (wire, standoffs, screws) | Internal assembly | ~$15 | M2.5 standoffs for Pi/HiFiBerry/PiJuice stack |

**Estimated total:** ~$270–300  
**Bluetooth speaker** (separate purchase — see Speaker Options below)

---

## Enclosure Design

**Target dimensions:** driven by AK2515 board — measure board first, design around it.

**Front face (left → right):**
- AK2515 VFD window — the dominant element, glows green with spectrum
- Small LED indicator (top-right corner) — mirrors app LED state (amber/green/red)
- Rotary encoder knob (right side)

**Back panel:**
- USB-C charging in
- RCA out (L/R) — for receiver / stereo system

**Material hierarchy:**
- Walnut end-caps (left and right, ~¾" thick)
- Brushed 6061 aluminum face plate inset between walnut caps
- Face plate slightly recessed from walnut edges (shadow line = intentional)

**Key constraint:** Buy the aluminum knob first. Design the encoder hole around it.

---

## Knob + VFD Interaction Design

```
PLAY MODE (default)
  VFD: spectrum animation
  Turn knob → navigate presets (next / prev)
  Short press → play / pause
  Long press → enter BT SPEAKER MODE

BT SPEAKER MODE
  VFD: scrolls paired device name (e.g., "MARSHALL WILLEN")
  Turn knob → scroll through paired BT devices
  Short press → connect selected device, return to PLAY MODE
  Long press → cancel, return to PLAY MODE
```

Paired devices are managed once (via CLI or companion app) and then permanently available on the knob menu. Adding a new speaker = pair via CLI, it appears in the menu.

---

## Software Stack (Pi)

| Component | Technology | Notes |
|-----------|-----------|-------|
| OS | Raspberry Pi OS Lite (64-bit) | Headless, no desktop |
| Audio playback | `mpv` | Streams radio URLs; output via PulseAudio |
| Audio routing | `PulseAudio` | Combined sink: BT speaker + HiFiBerry (RCA out + AK2515 feed) simultaneously |
| Bluetooth | `bluez` + `pulseaudio-module-bluetooth` | `bluetoothctl` for device management |
| GPIO (encoder) | Python + `gpiozero` | Reads turns and button presses |
| Device server | Python + `asyncio` + `websockets` | WebSocket API for phone app |
| mDNS discovery | `avahi-daemon` | Phone app detects Pi on local network |
| Battery management | PiJuice Python library | Read charge %, trigger safe shutdown |
| AK2515 power | USB from Pi | Pi USB-A → AK2515 USB-C |
| AK2515 audio | HiFiBerry DAC → 3.5mm Y-splitter | Clean signal; one branch to AK2515, one to RCA jacks |

**PulseAudio combined sink** routes audio to Bluetooth (wireless speaker) and HiFiBerry DAC (RCA out + AK2515 feed) simultaneously. Use Bluetooth when portable, RCA when connected to a receiver — both can be active at once.

---

## Phone App Integration (Future)

- Pi advertises itself via mDNS (`leftofthedial.local`)
- Left of the Dial web app detects Pi on local network
- "Cast to device" option appears when Pi is found
- Tapping a station sends the stream URL to Pi via WebSocket
- Pi confirms playback; app shows "Playing on device"

---

## Bluetooth Speaker Options

| Speaker | Size | Price | Notes |
|---------|------|-------|-------|
| Marshall Willen | 4.5"×2.7" | $80 | Vintage aesthetic pairs naturally with walnut/aluminum box |
| Tribit StormBox Micro 2 | 3"×3"×1.5" | $45 | Pocketable, good bass for size |
| Sony SRS-XB13 | 4"×2.8" cylinder | $60 | IP67, warm sound |
| B&O Beosound A1 | 4.7" puck | $250 | Premium; exceptional sound |

Marshall Willen recommended for aesthetic coherence — vintage leather + brass sits naturally next to walnut + aluminum.

---

## Open Questions

- [ ] AK2515 board exact dimensions (measure before designing face cutout)
- [ ] Pi Zero 2W vs Pi 3A+ for development (3A+ has full-size USB, easier to work with initially)
- [ ] Volume control — knob press-and-turn gesture, or second knob?
- [ ] LED wiring: 5mm through-hole on face plate, GPIO-controlled via NPN transistor
- [ ] Case construction: CNC aluminum face + hand-cut walnut, or full CNC
- [ ] PiJuice Zero vs simpler LiPo + TP4056 + boost converter (~$15 DIY vs $45 turnkey)

---

## Reference

- Web app: `tuner.redcontroldeck.com`
- App repo: `github.com/mkudlacz/left-of-the-dial`
- AK2515: Nobsound/Douk Audio — search "AK2515" on Amazon (~$50)
- PiJuice Zero: uk.pi-supply.com/products/pijuice-zero
- SendCutSend (aluminum face): sendcutsend.com
- Elma knobs: elma.com — "21 series" rotary knobs
- Marshall Willen: marshallheadphones.com
