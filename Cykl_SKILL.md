---
name: cykl-bikeshare
description: Use this skill whenever the user asks about Cykl bikes, bike availability, bike stations, docking stations, or renting bikes in Wageningen or the surrounding area. Also trigger for questions like "are there bikes available near X", "how many bikes at [station]", "where can I return a bike", "how much does Cykl cost", "how do I unlock the bike", "how do I register", or "show me Cykl stations". This skill fetches live data from the Cykl GBFS API and can display station maps, availability, pricing, and FAQ answers.
---

# Cykl Bike Share Skill

Cykl is a bike-sharing service in Wageningen, Netherlands. This skill fetches live data from the Cykl GBFS API to answer questions about bike availability, station locations, pricing, and common how-to questions.

## API Endpoints

All endpoints are public, no auth required.

| Feed | URL |
|------|-----|
| Root | `https://www.cykl.nl/gbfs/gbfs.json` |
| System info | `https://www.cykl.nl/gbfs/en/system_information.json` |
| Station locations | `https://www.cykl.nl/gbfs/en/station_information.json` |
| Live availability | `https://www.cykl.nl/gbfs/en/station_status.json` |
| Free-floating bikes | `https://www.cykl.nl/gbfs/en/free_bike_status.json` |

> Note: The GBFS pricing endpoint is outdated. Use the pricing section below instead.

## Data Model

**station_information** (static):
- `station_id`, `name`, `lat`, `lon`
- `rental_uris.web` — direct link to rent from this station

**station_status** (live, TTL 60s):
- `station_id`
- `num_bikes_available` — bikes ready to rent right now
- `num_docks_available` — free docks for returning
- `is_installed`, `is_renting`, `is_returning` — 1 = active

## Pricing (valid since 23-03-2026)

**Regular users:**
- €0.95 per 30 minutes
- Capped at €4.15 for 12 hours
- Capped at €4.75 for 24 hours
- Each additional 24h: €4.75
- After 72h: €9.50/24h (includes €4.75 penalty)
- Returning at a different Wageningen destination: Free
- Parking at Ede-Wageningen station: Free until 1 July 2026

**Pricing examples:**
- 30 min ride → €0.95
- 5 hours → €4.15 (12h cap)
- 24 hours → €4.75
- 48 hours → €9.50
- 72 hours → €14.25
- 96 hours → €23.75 (penalty applies after 72h)

**Frequent rider discounts** (after your second ride):
- Buy €19 credit → pay €18
- Buy €38 credit → pay €34

**Month Pass (30 min):** €8.50/month or €60/year — unlimited 30-min rides, 50% discount after
**Month Pass (12h):** €17/month — unlimited 12h rides, 50% discount after

## Workflows

### "Are there bikes available?" / "Find me a bike near X"
1. Fetch both `station_information` and `station_status`
2. Join on `station_id`
3. Filter for `num_bikes_available > 0` and `is_renting == 1`
4. If user mentioned a place, sort by proximity (use lat/lon from station_information)
5. Show top results with available bike count and a rent link
6. Optionally display a map using `places_map_display_v0` with station coordinates

### "Show me all stations" / "Map of Cykl stations"
1. Fetch `station_information` and `station_status`
2. Join them and display via `places_map_display_v0` — use station `lat`/`lon` and notes showing available bikes
3. Summarize total bikes available across the network

### "How much does it cost?"
Answer directly from the pricing section above.

### "Where can I return a bike?"
1. Fetch `station_status`
2. Filter for `num_docks_available > 0` and `is_returning == 1`
3. Join with `station_information` for names + coordinates
4. Present as list or map

## FAQ

### How do I unlock the bike?
All bikes have an electronic lock and a mechanical chain lock.

**Smaller electronic locks:**
- Tap the unlock button on your phone screen (visible after starting the rental)
- In winter/standby mode: press the white button on top of the lock, wait 30s, then try again
- Offline mode: tap unlock 3 times on your screen → app shows a color code → enter it using the white button on the lock

**Larger electronic locks:** Tap unlock on your phone screen. Push the lock button up if a spoke is blocking it.

**Mechanical chain lock:** The app shows a 5-digit code. Align the numbers between the two markings on the lock to open it. Take the lock with you during the ride.

### How do I return a bike?
Bring the bike back to any Cykl station, then finish the ride via the app or website:
1. Select a return destination on the map
2. Lock the bike at the station
3. Confirm return in the app/website

### How do I register?
Register at [cykl.nl/register.php](https://www.cykl.nl/register.php). Phone numbers from 188 countries are supported (all EU countries included).

### How many bikes can I rent at once?
Up to 4 bikes per account. Account must be activated with a payment authorization or a university/company email.

### How long can I keep a bike?
Maximum 72 hours at standard pricing. After 72h a penalty of €4.75/day is added (so €9.50/day total). Maximum rental period is 72h before penalties apply.

### Payment options
- Recurring payments (automatic debit) — required before first rental
- Pre-paid credit — available for students with a valid student email
- Top-up at [cykl.nl/shop/choose_amount.php](https://www.cykl.nl/shop/choose_amount.php) — minimum top-up is the 24h fee (€4.75)

### What if the bike lights don't work?
Most bikes have magnetically powered lights. Spin each wheel separately to test them — it can take a moment to activate. Contact support if lights are broken.

### How do I adjust the saddle?
Each bike has a lever. Pull it anticlockwise to loosen, adjust height, then re-tighten before riding.

### Can I add a new station location?
Cykl is open to adding locations in Wageningen and Ede. Contact them via support@cykl.nl.

### Can I share my own bike on Cykl?
Yes, via Cykl P2P. Add friends by phone number or colleagues by company email. Your bike appears in blue on the map for friends only.

### What traffic rules apply?
Dutch traffic rules apply. See the Dutch government traffic signs leaflet or [the Cykl FAQ](https://cyklhelp.zohodesk.com/portal/en/kb/articles/which-traffic-rules-do-apply) for details.

### What penalties can Cykl apply?
Repeated minor offences → temporary ban. Major offences → permanent ban. Additional fees apply for not returning to a Cykl station or intentional damage.

### How do I reset my password?
1. Go to cykl.nl and enter your phone number
2. Click 'Forgotten? Reset Password'
3. Enter the SMS verification code received

### Ede-Wageningen station (from March 2026)
Bikes can now be rented and returned at the secure parking facility on the south side of Ede-Wageningen station. Bikes must be returned to the secured bike parking only.

## Tips
- Data refreshes every 60 seconds — always fetch fresh when answering live questions
- Stations with `num_bikes_available: 0` are empty; mention this clearly
- Always include the rental web URI when recommending a specific station
- The Cykl app: iOS ([App Store](https://apps.apple.com/nl/app/cykl/id6496858422)) and Android (`nl.cykl.cykl_menu://`)
- Support: support@cykl.nl / +316-207-554-89
- Full FAQ: [cyklhelp.zohodesk.com/portal/en/kb/cykl/english](https://cyklhelp.zohodesk.com/portal/en/kb/cykl/english)
