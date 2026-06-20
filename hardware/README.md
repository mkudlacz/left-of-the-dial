# Left of the Dial — Hardware Project

A dedicated streaming radio device to complement the web app at `tuner.redcontroldeck.com`. The phone is the UI. The box plays and glows.

---

## Build Comparison

| Attribute | **Mk.1 — Stripped** ✓ | Mk.2 — Standard | Mk.3 — Full |
|-----------|----------------------|-----------------|-------------|
| **Status** | **Committed** | Defined | Defined |
| **Est. Cost** | **~$132** | ~$165 | ~$225 |
| **Audio out** | Bluetooth only | Bluetooth + RCA | Bluetooth + RCA |
| **Power** | Wall (USB-C) | Wall (USB-C) | Battery + Wall |
| **Battery** | None | None | PiJuice Zero + 3000mAh LiPo |
| **Compute** | Pi Zero 2W | Pi Zero 2W | Pi Zero 2W |
| **DAC** | Pi onboard (display feed only) | HiFiBerry DAC Zero | HiFiBerry DAC Zero |
| **Display** | AK2515 VFD 25×15 | AK2515 VFD 25×15 | AK2515 VFD 25×15 |
| **Enclosure** | Hammond 1455 + walnut end-caps | Hammond 1455 + walnut end-caps | Custom CNC aluminum + walnut |
| **Knob** | Davies Molding plastic | Machined aluminum | Machined aluminum |
| **Back panel** | USB-C | USB-C, RCA L/R | USB-C, RCA L/R |
| **Portability** | Desk/shelf (tethered) | Desk/shelf (tethered) | Fully portable |
| **Best for** | Proving the concept, living with it | Home receiver pairing | Statement object |

---

## Mk.1 — Stripped (Committed)

The proof of concept. Bluetooth audio to any paired speaker, AK2515 VFD for spectrum display, rotary knob for station navigation and speaker switching. No cables during use except power.

### Parts List

| # | Component | Purpose | Est. Cost | Source |
|---|-----------|---------|-----------|--------|
| 1 | Raspberry Pi Zero 2W | Compute + WiFi + Bluetooth | $15 | raspberrypi.com |
| 2 | Nobsound AK2515 | VFD spectrum display | $50 | Amazon |
| 3 | Hammond 1455N2202 | Aluminum extrusion enclosure body | $20 | mouser.com / digikey.com |
| 4 | Walnut board (~6"×4"×¾") | End-caps | $10 | local hardwood supplier |
| 5 | KY-040 rotary encoder | Station nav + BT speaker select | $5 | Amazon |
| 6 | Davies Molding 1434 knob | Tactile control | $6 | mouser.com |
| 7 | MicroSD card 32GB | Pi OS + software | $8 | Amazon |
| 8 | USB-C wall adapter 5V 3A | Power | $8 | Amazon |
| 9 | Short 3.5mm cable (internal) | Pi headphone out → AK2515 AUX | $3 | Amazon |
| 10 | Misc (M2.5 standoffs, wire, screws) | Internal assembly | $10 | Amazon |
| | **Total** | | **~$135** | |

### Signal Chain

```
Pi Zero 2W
  ├── WiFi ←→ Left of the Dial app (station control)
  ├── Bluetooth → paired BT speaker (audio)
  └── Onboard 3.5mm → AK2515 AUX in (spectrum display only)
```

Pi onboard audio is noisy but the AK2515 only needs the signal for visualization — quality is irrelevant here.

### Enclosure Notes

The Hammond 1455N2202 is a brushed aluminum extrusion (220×80×20mm) with removable plastic end-caps. Replace end-caps with hand-cut walnut. Drill/rout openings for:
- AK2515 VFD window (front face, left-center)
- Encoder hole (front face, right)
- LED indicator hole (front face, top-right)
- USB-C passthrough (back, from Pi)

The aluminum extrusion profile has a subtle ribbed texture that reads as vintage rack equipment. No CNC required — a drill press and router handle everything.

---

## Mk.2 — Standard

Adds clean RCA output for connection to a home receiver or stereo system. Everything else identical to Mk.1.

**Adds over Mk.1:**
- HiFiBerry DAC Zero (+$25) — clean I²S audio; replaces Pi onboard audio
- 3.5mm Y-splitter (+$3) — splits DAC output to AK2515 + RCA
- Panel-mount RCA jacks L/R (+$5) — rear panel audio out
- Machined aluminum knob (+$8 over Davies Molding)

**Signal chain change:**
```
HiFiBerry DAC Zero
  ├── 3.5mm Y-splitter → AK2515 AUX in (spectrum display)
  └── 3.5mm Y-splitter → RCA jacks (rear panel → receiver)
```

---

## Mk.3 — Full

Adds battery for full portability. Otherwise identical to Mk.2 but with upgraded enclosure to justify the cost.

**Adds over Mk.2:**
- PiJuice Zero (+$45) — LiPo battery management HAT
- 3000mAh LiPo (+$12) — ~5–6 hrs runtime at Pi Zero 2W power draw
- Custom CNC aluminum face (+$20–40 over Hammond) — justified at this tier
- USB-C now serves dual purpose: charging + power passthrough

---

## Knob Interaction (all builds)

```
PLAY MODE (default)
  VFD: spectrum animation
  Turn → next / previous preset
  Short press → play / pause
  Long press → enter BT SPEAKER MODE

BT SPEAKER MODE
  VFD: scrolls paired device name
  Turn → scroll paired BT devices
  Short press → connect, return to PLAY MODE
  Long press → cancel, return to PLAY MODE
```

---

## Software Stack (all builds)

| Layer | Technology | Notes |
|-------|-----------|-------|
| OS | Raspberry Pi OS Lite 64-bit | Headless |
| Stream playback | `mpv` | Handles radio stream URLs |
| Audio routing | `PulseAudio` | BT sink + local sink simultaneously |
| Bluetooth | `bluez` + `pulseaudio-module-bluetooth` | `bluetoothctl` for device management |
| GPIO | Python + `gpiozero` | Encoder turns + button |
| Device server | Python + `asyncio` + `websockets` | WebSocket API for phone app |
| Network discovery | `avahi-daemon` | `leftofthedial.local` on LAN |

### Phone App Integration (future)

Pi advertises on local network via mDNS. Left of the Dial app detects it and shows a "Cast to device" option. Tap a station → sends stream URL to Pi → Pi plays it and updates VFD.

---

## Bluetooth Speaker Pairings

| Speaker | Size | Price | Notes |
|---------|------|-------|-------|
| Marshall Willen | 4.5"×2.7" | $80 | Vintage aesthetic; recommended for Mk.1 |
| Tribit StormBox Micro 2 | 3"×3"×1.5" | $45 | Pocketable; best value |
| Sony SRS-XB13 | 4"×2.8" cylinder | $60 | IP67; warm sound |
| B&O Beosound A1 | 4.7" puck | $250 | Premium pairing for Mk.3 |

---

## Open Questions (Mk.1)

- [ ] Confirm Hammond 1455N2202 internal clearance fits Pi Zero 2W + AK2515 board
- [ ] AK2515 exact board dimensions (measure before cutting face opening)
- [ ] LED indicator wiring: 5mm through-hole, GPIO via NPN transistor
- [ ] Volume control: knob press-and-turn gesture, or accept BT speaker's own volume
- [ ] Pi Zero 2W vs Pi 3A+ for initial development (3A+ easier to work with, swap to Zero later)

---

## Reference

- Web app: `tuner.redcontroldeck.com`
- App repo: `github.com/mkudlacz/left-of-the-dial`
- AK2515: search "Nobsound AK2515" on Amazon (~$50)
- Hammond 1455N2202: mouser.com or digikey.com
- Davies Molding 1434: mouser.com
- HiFiBerry DAC Zero: hifiberry.com/shop/boards/hifiberry-dac-zero
- PiJuice Zero: uk.pi-supply.com/products/pijuice-zero
- SendCutSend (Mk.3 CNC face): sendcutsend.com
- Marshall Willen: marshallheadphones.com
