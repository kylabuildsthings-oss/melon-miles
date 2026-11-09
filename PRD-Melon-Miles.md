# Melon Miles — Product Requirements

**Version:** 1.0
**Date:** 2026-11-09
**Status:** Ready to build
**Product type:** Single-page web app (one HTML file, inline CSS and JavaScript)
**Working title in source notes:** Melon Soda Travel Oracle
**Product name:** Melon Miles

---

## 1. Summary

Melon Miles is a playful, pixel-art travel inspiration tool with a melon soda beach look. You say where you are flying from, which month you want to go, how long for, what you want to spend, and what climate and vibe you want. It returns your top 3 destinations, a Melon Meter match, a total cost estimate (flight plus daily spend), a packing list, and a shareable pixel card.

There is no login, no saving, and no booking. Open it, get ideas, close it. Nothing is stored.

---

## 2. Vision

> Open it, answer a few questions, get 3 destinations, a packing list, and a shareable pixel card — all in under 30 seconds.

It is for travellers who already know when they want to go and roughly what they want to spend, and need a push on where.

---

## 3. Audience

**Primary — The spontaneous planner**

- Age 24–35, young professional or digital nomad
- Wants inspiration quickly, cares about how it looks, and shares results
- Often on a phone

**Secondary — The friend-group organiser**

- Age 22–30, student or early career
- Needs 2–3 solid options to drop in a group chat
- Uses the pixel card to get a yes from the group

---

## 4. Product principles

- Inspiration only. Melon Miles does not search live flights, hotels, or weather, and it does not link out to book.
- Estimates only. Costs are hardcoded averages, labelled as estimates.
- Nothing persists. No cookies, `localStorage`, or `sessionStorage`. Closing the tab ends the session.
- One file. HTML, CSS, and JavaScript live in a single document. No frameworks. The only external request is the Press Start 2P font from Google Fonts.
- Fast. The page should load in under a second, search should feel instant (target under 50ms), and motion should stay smooth (target 60fps). The file should stay under 200KB.

---

## 5. User flows

### 5.1 Land

The user opens the page and sees the pixel form, already filled with defaults. They can search immediately.

### 5.2 Discover

The user changes month, budget, origin, temperature, vibe, or duration, then clicks **Find Destinations** or presses Enter. Up to 3 destination cards animate in below the form. They can change the inputs and search again. Each new search replaces the previous results.

### 5.3 Surprise

The user clicks **Surprise Me**. A 1–2 second slot-machine animation plays with pixel fruit, then one random destination from the filtered set is revealed. **Surprise Again** runs the same flow. A way back returns to the top 3, if a search has already been run.

### 5.4 Share

The user clicks **Share** on a destination. The app draws a pixel card on a canvas and downloads it as a PNG. If the browser has no canvas support, the share control is hidden and a short fallback line is shown instead.

### 5.5 Leave

The user closes the tab. No data is saved.

---

## 6. Inputs

| Input | Control | Options | Default | Rules |
| --- | --- | --- | --- | --- |
| Origin airport | Dropdown, grouped by zone | 20+ airports | London (LHR) | Required. Sets the flight-cost zone. |
| Month | Dropdown | January–December | The month the page is opened in | Required. Matched against each destination’s in-season months. |
| Temperature | Radio | Hot / Mild / Cold | Hot | Exact match. |
| Vibe | Radio | Sunny / Lush / Shoulder | Sunny | Exact match. Label on the control: “Shoulder”. |
| Budget | Number, USD | Minimum 100 | 1500 | Below 100: warn, and do not search until it is at least 100. |
| Duration | Number, days | Minimum 1 | 7 | 0, negative, or empty: snap back to 1. |

Pressing Enter in the form runs the same action as **Find Destinations**.

---

## 7. Data

All data is hardcoded in the page. Zones are cost zones, not a geography lesson: Dubai is priced on the Asia row, and Istanbul is priced on the Africa row, matching the airport list below.

### 7.1 Destination

```javascript
{
  name: "Bali",
  flag: "🇮🇩",
  temp: "hot",            // hot | mild | cold
  vibe: "lush",           // sunny | lush | shoulder
  region: "asia",         // europe | asia | americas | africa | oceania
  months: ["april", "may", "june", "july", "august", "september"],
  peakMonths: ["july", "august"],  // subset of months; used only as a tie-break
  dailyCost: 70,          // USD per day for stay and food
  packing: ["swimwear", "sunscreen", "light clothes", "rain jacket"]
}
```

`months` is the allow-list. If the chosen month is absent, the destination is out.

`peakMonths` is optional and must be a subset of `months`. A peak month ranks above an in-season month that is not peak. It never overrides temperature, vibe, or budget.

### 7.2 How many destinations

Minimum 30.

| Region | Count |
| --- | --- |
| Europe | 8 |
| Asia | 8 |
| Americas | 6 |
| Africa / Middle East | 4 |
| Oceania | 4 |

| Temperature | Count |
| --- | --- |
| Hot | 12 |
| Mild | 10 |
| Cold | 8 |

| Vibe | Count |
| --- | --- |
| Sunny | 10 |
| Lush | 10 |
| Shoulder | 10 |

The three cuts are independent. A destination counts once in each table. Every destination needs a flag, a region, at least one in-season month, a daily cost, and 3–5 packing items.

Starter rows (expand these until the counts above are met):

| Name | Flag | Temp | Vibe | Region | Daily cost (USD) |
| --- | --- | --- | --- | --- | --- |
| Bali | 🇮🇩 | Hot | Lush | Asia | 70 |
| Phuket | 🇹🇭 | Hot | Sunny | Asia | 85 |
| Iceland | 🇮🇸 | Cold | Shoulder | Europe | 140 |
| Canary Islands | 🇪🇸 | Hot | Sunny | Europe | 95 |
| Costa Rica | 🇨🇷 | Hot | Lush | Americas | 100 |
| Chamonix | 🇫🇷 | Cold | Sunny | Europe | 130 |
| Kyoto | 🇯🇵 | Mild | Shoulder | Asia | 90 |
| Cape Town | 🇿🇦 | Mild | Sunny | Africa | 80 |

### 7.3 Airports

Minimum 20. Each airport has a city name, a code, and a zone used for flight cost.

| Zone | Airports |
| --- | --- |
| Europe | London (LHR), Paris (CDG), Berlin (BER), Rome (FCO), Amsterdam (AMS) |
| Americas | New York (JFK), Los Angeles (LAX), Miami (MIA), Toronto (YYZ), Mexico City (MEX) |
| Asia | Tokyo (NRT), Singapore (SIN), Dubai (DXB), Mumbai (BOM), Seoul (ICN) |
| Oceania | Sydney (SYD), Auckland (AKL) |
| Africa | Johannesburg (JNB), Cape Town (CPT), Istanbul (IST) |

The dropdown groups airports under those zone labels.

### 7.4 Flight cost matrix

Lookup is `originZone → destinationRegion → USD`. These are one-way-style planning estimates for the whole flight leg of the trip, not live fares. The same number is used regardless of month.

| Origin zone | Europe | Asia | Americas | Africa | Oceania |
| --- | --- | --- | --- | --- | --- |
| Europe | 150 | 700 | 600 | 500 | 900 |
| Americas | 500 | 900 | 300 | 700 | 1100 |
| Asia | 700 | 200 | 900 | 600 | 500 |
| Oceania | 900 | 500 | 1100 | 800 | 300 |
| Africa | 500 | 600 | 700 | 300 | 1000 |

Example: London is Europe, so London → Asia is $700 and London → Europe is $150.

---

## 8. Search logic

### 8.1 Cost

```
flightCost = matrix[origin.zone][destination.region]
stayCost   = destination.dailyCost × duration
totalCost  = flightCost + stayCost
```

### 8.2 Filters

A destination is a **match** only when all of these are true:

1. `temp` equals the selected temperature.
2. `vibe` equals the selected vibe.
3. The selected month is in `months`.
4. `totalCost ≤ budget × 1.2`.

Temperature and vibe are exact. There is no “close enough” on those.

### 8.3 Melon Meter

The meter describes how the total sits against the budget. Because matches stop at `budget × 1.2`, a normal result is only ever Perfect or Good.

| Meter | Rule | Meaning |
| --- | --- | --- |
| 🍉🍉🍉 Perfect | `totalCost ≤ budget × 1.1` | Inside the trip, with a little room |
| 🍉🍉 Good | `totalCost ≤ budget × 1.2` | Inside the match window, above the perfect line |
| 🍉 Okay | Shown only as a stretch | Over the match window; see empty state |

Good does **not** extend to `budget × 1.4`. That wider band would disagree with the budget filter, so V1 uses 1.1 and 1.2 only.

### 8.4 Ranking

Sort matches in this order:

1. **Melon Meter.** Perfect above Good.
2. **Month fit.** `peakMonths` containing the selected month above a regular in-season month.
3. **Value.** Lower `totalCost` first. If totals are equal, lower `dailyCost` first.

Return the first 3. If fewer than 3 match, show however many match. Do not pad with destinations that failed a filter.

### 8.5 Surprise pool

**Surprise Me** picks uniformly at random from the full match set, not only from the top 3. If the match set is empty, it uses the same empty state as search and does not play a fake result.

**Surprise Again** draws again from that same set. It may repeat the previous draw.

### 8.6 Empty state and stretch ideas

When nothing matches:

- Heading: **No matches found**
- Body: **Try increasing your budget or changing the month.**

Under that, show up to 3 **stretch** destinations so the page is not a dead end. A stretch candidate passes temperature, vibe, and month, and fails only the budget gate. Sort stretches by `totalCost` ascending (closest to affordable first) and label each **🍉 Okay**. If temperature, vibe, and month together still produce nothing, show the empty copy only, with no cards.

### 8.7 Budget warning

If budget is under $100, block the search and show a warning. Suggest a practical floor: the cheapest `dailyCost` in the whole dataset multiplied by the current duration, plus the cheapest flight in the matrix from the selected origin zone. Round that suggestion up to the nearest $50, and never suggest less than $100. The user can still type; the warning clears once budget is 100 or more.

---

## 9. Outputs

### 9.1 Top 3 cards

Each card shows:

- Flag and destination name
- Melon Meter (the watermelon glyphs, plus the word Perfect, Good, or Okay)
- Total estimated cost
- Breakdown: flight cost and stay cost (`dailyCost × duration`)
- Temperature and vibe labels
- A mini packing list of 3–5 items (the destination’s `packing` array)

Costs display as whole US dollars, for example `$1,240`.

### 9.2 Full packing list

One checklist for the destination in focus (the first result, or the surprise result). It starts from `packing`, then adds items from climate and length:

| Condition | Add |
| --- | --- |
| Hot | Sunglasses, sandals, hat |
| Mild | Light layers, comfortable shoes |
| Cold | Warm coat, thermal layers, gloves, beanie |
| Lush | Insect repellent, quick-dry towel |
| Duration of 10 days or more | Laundry kit, extra underwear |
| Duration of 14 days or more | A second pair of shoes, small day bag |

Render it as a pixel checklist: a ticked or empty pixel box beside each line. Items already on the destination list are not duplicated. Cap the full list at 12 lines.

### 9.3 Shareable pixel card

A PNG from the Canvas API, offered as a download. The card contains:

- Destination name and flag
- Month
- Melon Meter
- Total estimated cost
- A pixel border in the melon soda palette

File name: `melon-miles-{destination}-{month}.png`, lower case, spaces as hyphens.

If `canvas.getContext` is missing, hide every Share control and show: **Sharing needs a browser that can draw images.**

Copy-to-clipboard is out of V1. Download is the share action.

---

## 10. Interface

### 10.1 Regions of the page

1. Header: the name **Melon Miles**, a one-line description, and a small melon mark.
2. Form: the six inputs, **Find Destinations**, and **Surprise Me**.
3. Results: top 3 cards, or the empty state, plus the packing checklist.
4. Surprise overlay: slot animation, then the revealed card, with **Surprise Again** and **Back**.

### 10.2 Surprise animation

About 1.2–1.8 seconds. Three reels of fruit glyphs (🍉 🍊 🍋 🍒 🍇) cycle and then stop. Motion uses `transform` only. Respect `prefers-reduced-motion`: if it is set, skip the reels and show the result immediately.

### 10.3 Copy tone

Short, warm, and a little silly. Buttons say what they do: **Find Destinations**, **Surprise Me**, **Surprise Again**, **Share**, **Back**. Costs are always marked as estimates, once near the results: **Costs are rough estimates, not live prices.**

---

## 11. Visual design

### 11.1 Palette

| Name | Hex | Use |
| --- | --- | --- |
| Melon Green | `#d7f0f0` | Page background |
| Melon Soda | `#ffe8d4` | Cards |
| Pixel Shadow | `#3b2b1f` | Borders and offset shadows |
| Ocean Teal | `#2f4b4b` | Text and accents |
| Coral Pink | `#f7cfb0` | Buttons |
| Sand Beige | `#fdf3e7` | Input backgrounds |
| Melon Fruit | `#ff6b4a` | Accents and the meter |

Flat colours only. No gradients.

### 11.2 Type

- Primary: `Press Start 2P` from Google Fonts
- Fallback: `Courier New`, monospace
- Sizes between `0.7rem` and `1.8rem`
- On small screens, body copy can sit at the low end of that range so lines stay readable

### 11.3 Pixel rules

- Borders of 3px or 4px, solid Pixel Shadow
- Box shadows are offset only, with no blur (for example `4px 4px 0 #3b2b1f`)
- Corner radius at most 32px
- `image-rendering: pixelated` on the generated card and any drawn art
- Icons are emoji or drawn pixels, not icon-font SVGs from a network

### 11.4 Layout

- Desktop and tablet: one column, max width 820px, centred
- Mobile: inputs stack, buttons go full width, type steps down
- The form must be usable at 375px wide without horizontal scroll

---

## 12. Technical specification

| Item | Choice |
| --- | --- |
| Delivery | One `.html` file |
| Languages | HTML5, CSS3, vanilla JavaScript |
| Frameworks | None |
| Fonts | Press Start 2P (Google Fonts). Everything else is local. |
| APIs | None |
| Storage | None |
| Analytics | None in V1 |

### 12.1 Without JavaScript

The product cannot rank destinations without a script. If JavaScript is off, a `<noscript>` block in the page explains that Melon Miles needs JavaScript and restates the one-line description. The form is not shown as if it would work.

### 12.2 Browsers

- Current Chrome, Firefox, Safari, and Edge
- Mobile Safari on iOS 15 and later
- Chrome on Android 10 and later

### 12.3 Accessibility

- Text and background pairs from the palette should meet WCAG AA for the size they are used at. Ocean Teal on Melon Green, Melon Soda, and Sand Beige is the default text pairing. Melon Fruit is for marks and short accents, not for paragraphs.
- Every control has a visible label. Radios are grouped in a fieldset with a legend.
- Focus states are visible (offset pixel outline, not colour alone).
- Buttons and inputs are reachable and operable by keyboard. Enter submits the form. Escape closes the surprise overlay.
- The surprise overlay traps focus while it is open and returns focus to **Surprise Me** when it closes.
- Meter results include the word Perfect, Good, or Okay, not only the glyphs.
- `prefers-reduced-motion` skips the slot reels and shortens the results entrance.

### 12.4 Performance

- First load under 1 second on a normal connection, after the font
- Search work under 50ms for a dataset of this size (compute on the click; do not pre-render every destination)
- Animations stay on `transform` and `opacity`
- File size under 200KB including the destination data

---

## 13. Edge cases

| Case | Behaviour |
| --- | --- |
| No destination passes every filter | Empty heading and the budget/month suggestion. Stretch cards only if temperature, vibe, and month matched something over budget. |
| Budget under $100 | Inline warning and a suggested minimum. Search does not run. |
| Duration is 0, negative, or blank | Set the field to 1 and continue. |
| Budget or duration is not a number | Treat as invalid, show a one-line hint, and do not search. |
| Canvas is unavailable | Hide Share. Show the fallback sentence in section 9.3. |
| Fewer than 3 matches | Show 1 or 2 cards. Do not invent fillers. |
| Surprise with an empty match set | Same empty state. No animation. |
| Font request fails | Courier New. Layout does not depend on the webfont metrics. |

---

## 14. Success signals

V1 does not record these. There is no analytics call and no storage, on purpose. The numbers below are the product goals for a later version that adds privacy-conscious measurement. They are not V1 acceptance tests.

| Signal | Goal |
| --- | --- |
| Time on page | Over 60 seconds |
| Searches per session | Over 1.5 |
| Sessions that use Surprise Me | Over 30% |
| Sessions that use Share | Over 20% |
| Sessions that search more than once | Over 40% |
| Return visitors | Over 25% |

V1 is accepted when the flows in section 5 work, the rules in section 8 match the UI, the data counts in section 7 are met, and the edge cases in section 13 behave as written.

---

## 15. Out of scope for V1

- Accounts, saved trips, and history
- Live flight, hotel, or weather data
- Booking links
- Currency conversion
- Comparing more than the top 3 side by side in a separate view
- Dark mode
- Copy-to-clipboard for the card
- Offline install / PWA
- Cookies and any browser storage

---

## 16. Later (V2)

- Currency selector: USD, EUR, GBP
- More destinations (50+) and extra vibes such as romantic, adventure, and cultural
- Seasonal events
- A real flight-price source, still shown as a range
- Dark mode
- Installable offline shell
- GIF export of the pixel card
- A way for people to suggest destinations

---

## 17. Build checklist

- [ ] 30 or more destinations, with the region, temperature, and vibe counts in section 7.2
- [ ] 20 or more airports and the flight matrix in section 7.4
- [ ] Form defaults, including “this month”
- [ ] Filter, Melon Meter, and sort match section 8
- [ ] Top 3 cards, packing checklist, and PNG share card
- [ ] Surprise Me animation, with a reduced-motion path
- [ ] Empty, low-budget, bad-number, and no-canvas states
- [ ] Layout checked at 375px and at the 820px column
- [ ] Keyboard path: labels, focus, Enter, Escape
- [ ] `<noscript>` message
- [ ] No storage APIs and no network calls except the font
- [ ] File under 200KB

---

## 18. Constraints to remember while building

- Static file. It must run by opening the HTML, with no server logic.
- Flight costs and “good months” are averages. The UI must say they are estimates.
- The app has to stay usable on a phone and by keyboard.
