# Freeform — Terrestrial Radio Streamer

A single-file web app for discovering and streaming college, public, and community radio stations across the United States. No build step, no dependencies, no accounts — open `freeform.html` in any browser and tune in.

---

## What It Is

Built for the moment when someone says "WKCR is doing a Sonny Rollins tribute" and you want to tune in immediately. Modeled after the [No Tape Left Behind archive player](https://player.redcontroldeck.com) — dark, fast, stays out of the way.

---

## Features

- **~160 stations** covering all regions of the US — college, public, NPR affiliates, community, and freeform
- **Browse by filter** — College, Public/NPR, Freeform, Community, Jazz, Classical, Indie/Alt, News
- **Browse by metro** — New York, Boston, Chicago, Bay Area, LA, Seattle/PDX, Southeast, Midwest, Southwest, Mountain West
- **Live search** — by call sign, city, state, org name, or genre
- **Tuning animation** — analog needle sweep while connecting
- **VU meter** — animated bars while streaming live
- **Favorites** — star any station; saved to localStorage
- **Now-playing bar** — volume control, live status, direct link to station site

---

## Files

```
Terrestial Radio Streamer/
├── freeform.html       # The entire app — open this in a browser
└── README.md           # This file
```

---

## Running It

```bash
open freeform.html
# or
open -a "Google Chrome" freeform.html
```

No server required. Works as a local file in Chrome, Firefox, or Safari.

To install as a home screen app on iPhone: open in Safari → Share → Add to Home Screen.

---

## Station Database

Stations are defined in a JavaScript array at the top of `freeform.html`. Each entry:

```js
{
  id:    'wkcr',
  name:  'WKCR 89.9',
  freq:  '89.9 FM',
  city:  'New York',
  state: 'NY',
  org:   'Columbia University',
  genre: 'Jazz · Classical · Freeform',
  tags:  ['college', 'jazz', 'new york', 'tri-state'],
  url:   'https://live.wkcr.org/wkcr.mp3',   // direct stream URL
  site:  'wkcr.org'
}
```

### Adding a Station

1. Find the station's direct stream URL (usually on their website under "Listen Live" or "Stream")
2. Add an entry to the `DB` array in `freeform.html`
3. Add relevant tags — these drive the filter chips

### Tags Reference

| Tag | Filter chip |
|-----|------------|
| `college` | College |
| `public`, `npr` | Public/NPR |
| `freeform` | Freeform |
| `community` | Community |
| `jazz` | Jazz |
| `classical` | Classical |
| `indie` | Indie/Alt |
| `news` | News |
| `new york`, `tri-state` | New York |
| `boston`, `new england` | Boston |
| `chicago` | Chicago |
| `bay area` | Bay Area |
| `los angeles` | LA |
| `pacific northwest`, `seattle` | Seattle/PDX |
| `southeast`, `south` | SE/South |
| `midwest` | Midwest |
| `southwest` | Southwest |
| `mountain west` | Mountain |

---

## Stream URL Notes

Stream URLs occasionally rotate — radio stations update their infrastructure. If a station won't connect:

1. Visit the station's site directly (linked in the now-playing bar)
2. Find their "Listen Live" page
3. Right-click the player → Inspect → Network tab → look for `.mp3`, `.aac`, or `.ogg` requests
4. Update the `url` field in the `DB` entry

Most college stations use Icecast and follow the pattern:
```
https://[station-domain]:8000/[callsign].mp3
```

---

## Roadmap / Ideas

- [ ] Pull live "now playing" metadata from station APIs (WFMU, WKCR, KEXP all have them)
- [ ] Schedule / program guide panel per station
- [ ] Radio Browser API integration as a fallback for undiscovered stations
- [ ] PWA manifest for installable home screen app
- [ ] Station notes / descriptions (history, format, notable shows)
- [ ] "Similar stations" suggestions
- [ ] Keyboard shortcuts (space = play/pause, arrow keys = prev/next favorite)
- [ ] Export / share a station link

---

## Credits

Built with Claude. Station data sourced from Wikipedia's list of US college radio stations, station websites, and the Radio Browser directory. Stream URLs verified against each station's live streaming page.
