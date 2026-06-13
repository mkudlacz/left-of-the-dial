# Left of the Dial

A single-file web app for discovering and streaming college, public, and community radio stations across the United States. No build step, no dependencies, no accounts — open `index.html` in any browser and tune in.

Named after the Replacements song.

---

## What It Is

Built for the moment when someone says "WKCR is doing a Sonny Rollins tribute" and you want to tune in immediately. Styled after vintage 70s/80s stereo equipment — walnut cabinet, brushed aluminum faceplate, one red power LED.

Live at **[tuner.redcontroldeck.com](https://tuner.redcontroldeck.com)**

---

## Features

- **~86 verified stations** — college, public, NPR, freeform, jazz, classical, community
- **Browse by format** — College, Public/NPR, Freeform, Community, Jazz, Classical, Indie/Alt, News
- **Browse by metro** — New York, Boston, Chicago, Bay Area, LA, Seattle/PDX, Southeast, Midwest, Southwest, Mountain West
- **Live search** — by call sign, city, state, org name, or genre (fires on keystroke)
- **Tuning animation** — analog needle sweep while connecting
- **VU meter** — animated bars while streaming
- **Favorites** — star any station; saved to localStorage
- **Shareable favorites** — tap ⤴ Share to copy a URL encoding your saved stations; anyone opening it gets those stations imported
- **Now-playing bar** — live status, direct link to station site
- **PWA** — installable on iPhone via Safari → Share → Add to Home Screen

---

## Files

```
left-of-the-dial/
├── index.html      # The entire app — open this in a browser
├── manifest.json   # PWA manifest
└── README.md
```

---

## Running Locally

```bash
open index.html
# or
open -a "Google Chrome" index.html
```

No server required.

---

## Station Database

Stations are defined in a JavaScript array at the top of `index.html`. Each entry:

```js
{ id:    'wkcr',
  name:  'WKCR 89.9',
  freq:  '89.9 FM',
  city:  'New York',
  state: 'NY',
  org:   'Columbia University',
  genre: 'Jazz · Classical · Freeform',
  tags:  ['college', 'jazz', 'new york', 'tri-state'],
  url:   'http://wkcr.streamguys1.com/live',
  site:  'wkcr.org' }
```

### Verified stream URLs

All URLs were tested with `curl -r 0-50` in June 2026. ~86 are confirmed live; ~54 are commented out pending working URLs.

To find a correct URL for a broken station, the Radio Browser API is reliable:

```bash
curl -s -H "User-Agent: LeftOfTheDial/1.0" \
  "https://de1.api.radio-browser.info/json/stations/search?name=WKCR&countrycode=US&limit=3" \
  | python3 -c "import sys,json; [print(s['name'],'|',s['url_resolved']) for s in json.load(sys.stdin)]"
```

The `User-Agent` header is required — the API returns empty without it.

### Adding a Station

1. Find a direct stream URL (MP3 or AAC — not `.m3u`, `.m3u8`, or `.pls`)
2. Verify it: `curl -s --max-time 5 -r 0-50 "URL" | wc -c` — should be > 10
3. Add an entry to the `DB` array in `index.html`
4. Add relevant tags

### Tags Reference

| Tag | Filter chip |
|---|---|
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

## Shareable Favorites

The ⤴ Share button copies a URL like:

```
https://tuner.redcontroldeck.com/#favs=wkcr,wfmu,kalx
```

Anyone opening that URL gets those stations merged into their saved list. No backend required.

---

## Deployment

GitHub repo → Cloudflare Pages (no build command, output dir `/`) → `tuner.redcontroldeck.com` custom domain. Push to `main` and Cloudflare deploys automatically within ~60 seconds.

---

## Roadmap

- [ ] Restore ~54 hidden stations once correct stream URLs are found
- [ ] Live "now playing" metadata from station APIs (WFMU, WKCR, KEXP all have them)
- [ ] Schedule / program guide panel per station
- [ ] Keyboard shortcuts (space = play/pause)
- [ ] Station notes / history / notable shows
