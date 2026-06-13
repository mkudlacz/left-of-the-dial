# Left of the Dial

A single-file web app for discovering and streaming college, public, and community radio stations across the United States. No build step, no dependencies, no accounts — open `index.html` in any browser and tune in.

Named after the Replacements song.

---

## What It Is

Built for the moment when someone says "WKCR is doing a Sonny Rollins tribute" and you want to tune in immediately. Styled after vintage 70s/80s stereo equipment — walnut cabinet, brushed aluminum faceplate, one red power LED.

Live at **[tuner.redcontroldeck.com](https://tuner.redcontroldeck.com)**

---

## Features

- **101 verified stations** — college, public, NPR, freeform, jazz, classical, community
- **Browse by format** — College, Public/NPR, Freeform, Community, Jazz, Classical, Indie/Alt, News
- **Browse by metro** — New York, Boston, Chicago, Bay Area, LA, Seattle/PDX, Southeast, Midwest, Southwest, Mountain
- **Live search** — by call sign, city, state, org, genre, tags, and **now-playing metadata** (type "coltrane" or "afrobeat" to surface stations currently playing that)
- **Now-playing metadata** — artist/song shown while streaming, fetched from station APIs (KEXP, KCRW) or auto-derived Icecast status for any self-hosted station; polled every 60s
- **Tuning animation** — analog needle sweep while connecting
- **VU meter** — animated bars while streaming
- **Favorites** — star any station; saved to localStorage
- **Shareable favorites** — ⤴ Share copies a URL like `tuner.redcontroldeck.com/#favs=wkcr,wfmu`; anyone opening it gets those stations imported
- **Favorites importer** — paste a share URL into the Saved tab to migrate favorites across devices
- **Now-playing bar** — live status, metadata, direct link to station site
- **PWA** — installable on iPhone via Safari → Share → Add to Home Screen; home screen icon is a vintage radio dial photo

---

## Files

```
left-of-the-dial/
├── index.html      # The entire app
├── manifest.json   # PWA manifest
├── icon.jpg        # Home screen icon (vintage radio dial photo, 300×300)
├── icon.svg        # Browser tab favicon (drawn tuning dial)
└── README.md
```

---

## Running Locally

```bash
open index.html
```

No server required. Works as a local file in Chrome, Firefox, or Safari.

---

## Station Database

Stations live in the `DB` array in `index.html`:

```js
{ id:'wkcr', name:'WKCR 89.9', freq:'89.9 FM', city:'New York', state:'NY',
  org:'Columbia University', genre:'Jazz · Classical · Freeform',
  tags:['college','jazz','new york','tri-state'],
  url:'http://wkcr.streamguys1.com/live', site:'wkcr.org' }
```

### URL Verification

```bash
curl -sL --max-time 7 -H "User-Agent: Mozilla/5.0" -r 0-50 "$url" | wc -c
# >10 bytes = stream is live. Use -L to follow CDN redirects.
```

Finding a correct URL for a broken station — Radio Browser API is most reliable:

```bash
curl -s -H "User-Agent: LeftOfTheDial/1.0" \
  "https://de1.api.radio-browser.info/json/stations/search?name=WKNC&countrycode=US&limit=3" \
  | python3 -c "import sys,json; [print(s['name'],'|',s['url_resolved']) for s in json.load(sys.stdin)]"
```

**The `User-Agent` header is required** — the API returns empty without it.

### Status (June 2026)

- **101 active** — verified working
- **39 hidden** — commented out with `/* ... */`, pending working URLs

Hidden stations (39): WBRS, WMHC, WMUA, WZLY, WMHB, WHUS, WSOU, WVBR, WKRB, WBNY, WHCL, WRCU, WALF, WMUC, WVUD, WQHS, WVUM, WKNC, WVVS, WRFL, WNUR, WHPK, KMNR, KWUR, KURE, KRLX, KSDB, KBGA, KVRX, KTRU, KASC, KRUX, KUCR, KKJZ, LAist 89.3, KBVR, KMHD, KVCU, WMSE

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

## Now-Playing Metadata

Metadata is fetched while a station is streaming and shown in the now-playing bar. Sources (tried in order):

1. **Station-specific API** — KEXP (`api.kexp.org/v2/plays`), KCRW (`tracklist-api.kcrw.com/Simulcast`)
2. **Icecast status** — auto-derived as `<stream-origin>/status-json.xsl` for any self-hosted Icecast station

If CORS blocks the request or no metadata is available, nothing is shown. Polled every 60 seconds.

Metadata is cached per station in the session and included in search — searching "afrobeat" or "coltrane" will surface stations whose recent now-playing text matches.

---

## Shareable Favorites

The ⤴ Share button copies a URL like:

```
https://tuner.redcontroldeck.com/#favs=wkcr,wfmu,kalx
```

Opening that URL merges those stations into the recipient's saved list. The importer in the Saved tab accepts the full URL, bare hash (`#favs=...`), or a plain comma list.

---

## Deployment

GitHub `main` → Cloudflare Pages → `tuner.redcontroldeck.com`. Push to `main` and Cloudflare deploys in ~60 seconds. No build command, output dir `/`.

---

## Backlog

### Station coverage
- [ ] Restore 39 hidden stations — find working stream URLs
- [ ] Periodic re-verification of active URLs (CDNs rotate)
- [ ] Fix WNUR — only known URL is `.m3u` playlist, need actual Icecast endpoint

### Metadata enrichment
- [ ] Expand `META_API` to WFMU, WWOZ, WNYC/WQXR, WBUR, WBEZ (APIs exist, need correct endpoints + CORS check)
- [ ] Show name / program guide alongside song metadata where available
- [ ] Per-station program schedule panel

### Discovery
- [ ] Station notes — history, format, notable shows (e.g. WKCR's Festival Jazz each January)
- [ ] "Similar stations" suggestions based on shared tags
- [ ] Radio Browser API integration for discovering stations beyond the curated DB

### Navigation
- [ ] Keyboard shortcuts: Space = play/pause, F = favorite, Esc = stop
- [ ] Last-played station resumes on next open (save `currentId` to localStorage)
- [ ] Deep-link to a station: `tuner.redcontroldeck.com/#station=wkcr`
- [ ] Recently played history

### Sharing
- [ ] Share currently playing station (link that opens the app tuned to that station)

### Technical
- [ ] Service worker / offline shell
- [ ] HLS support via lightweight lib (unlocks ~8 hidden stations with m3u8-only streams)
