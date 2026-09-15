# Maasin City Website — Prototype Handover

**File:** `maasin-city-website.html` — one self-contained file. Open it in any browser, no build step, no server, no dependencies.

This is a **design and content reference**, not production code. It exists so the design decisions, page structure and content model are settled before you write the real thing.

---

## What's in it

| Route              | Page                                                                                      |
| ------------------ | ----------------------------------------------------------------------------------------- |
| `#/`               | Landing page — trip-stage guide (Before You Arrive / During Your Stay / If You Need Help) |
| `#/destinations`   | Tourist destinations gallery, filter + search, 27 sites                                   |
| `#/transportation` | Three tabs: By Sea, By Land, Around the City (motorcab + PUJ)                             |
| `#/accommodation`  | 22 establishments, filter + search, links to Google and Facebook                          |
| `#/emergency`      | Emergency hotlines grouped by urgency, `tel:` links                                       |
| `#/devnotes`       | Implementation notes (also summarised below)                                              |

---

## Content model

All content sits in plain arrays at the top of the `<script>` block, deliberately separated from rendering. **These are what you replace with CMS collections or API endpoints.**

```js
DESTINATIONS; // name, brgy, cat, status, desc, photo
SEA_ROUTES; // op, type, route, freq, port, office
LAND_ROUTES; // dest, mode, time, note
HOTELS; // n, b, t, rooms, tel, gbp, fb
EMERGENCY; // grouped: g, items[{n, w, t, prime}]
VISITOR_TYPES; // welcome survey options
```

The existing site runs on **Joomla**, so these map naturally onto categories with custom fields. Nothing in the rendering functions needs to change when the data source does.

### Field notes

- `HOTELS.gbp` / `HOTELS.fb` — Google Business Profile and Facebook Page URLs. When empty, the renderer falls back to a Google Maps **search** link built from the establishment name and barangay. Those work today, but the real profile URL is better. Suggested collection method: add both fields to the annual business permit renewal form for accommodation establishments.
- `DESTINATIONS.status` — `"open"` or `"soon"`. Sites marked `soon` must stay labelled that way until the facility is genuinely open to visitors.
- `EMERGENCY.items[].t` — phone number. Empty renders an `ADD NUMBER` tag. **Only 911 is filled in, deliberately.**

---

## Welcome survey

Appears once per visitor, locks scrolling until answered, stores the choice in `localStorage` under `maasin_visitor_type`.

`recordAnswer()` has the POST call commented in — wire it to an endpoint that stores **only** the choice, a timestamp and the landing page. No personal data.

```js
fetch("/api/visitor-survey", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    type: value,
    at: new Date().toISOString(),
    landing: location.hash,
  }),
});
```

**Two rules that must survive implementation:**

1. The survey **never** appears when someone lands directly on `#/emergency`, and the modal itself carries an emergency escape link. A person looking for a rescue number must not meet a questionnaire first.
2. Add a Data Privacy Act notice linked from the modal before launch. Responses stay anonymous.

Watch the bounce rate for the first month. A forced gate collects from nearly everyone but costs traffic; if bounce climbs sharply, make the modal dismissible and accept a lower response rate.

---

## Build properly

The prototype cuts corners that production must not:

- **Real routing**, not hash-based. Server-rendered pages so search engines index them.
- **Image optimisation** — responsive `srcset`, lazy loading, compressed. Every dashed box in the prototype marks a photo slot and names the shot required.
- **Alt text on every image.** This is a government site; accessibility is a legal expectation.
- **Sitemap and structured data** — `LocalBusiness` / `TouristAttraction` schema on destination and hotel entries.
- **"Last updated" date** visible on Transportation, Accommodation and Emergency.

### Performance

Many residents across the 60 rural barangays browse on mobile data, some over shared Starlink links. Keep pages under 1 MB. The **Emergency page especially** must load fast on a slow connection — consider making it the one page that works offline via a service worker.

### Nothing operational may be hard-coded

Fares, schedules, room counts and hotlines all change. Every one of them must be editable by city staff without a developer.

---

## Design tokens

Defined as CSS custom properties in `:root` at the top of the stylesheet — lift these as the site's design system. Light and dark themes are both defined; the page follows the visitor's OS setting.

```
--sea      #07414F   primary, headers, nav active
--sea-2    #0E7A8C   links, icons, interactive
--sun      #E07B39   primary action buttons
--alert    #B3261E   emergency only — never decorative
--ink      #0E2027   body text
--bg       #F6F6F3   page ground
```

**Type:** Gabarito (headings) + Instrument Sans (body), both Google Fonts, with system fallbacks declared.

**Breakpoints:** 980px (3-col → 2-col) and 620px (→ single column, nav scrolls horizontally).

---

## Outstanding content

Everything marked with an amber `ADD …` tag in the prototype:

- Ship schedules, fares and booking links
- Bus/van travel times, fares, terminal bay assignments
- **Motorcab fare matrix** (LTFRB-approved, with approval date)
- **PUJ route list** — route name, barangays served, fare, first/last trip
- Room counts for 18 establishments (BPLO permit records hold these)
- Verified contact numbers for all 22 establishments
- Every emergency hotline except 911
- Lonoy cave details; Ajonay Festival date
- Airport, sports centre and port construction status
- City Hall trunkline, official email, office hours

---

## One content decision to settle first

The header greets visitors in Bisaya and the rest of the site is English. Decide **now** whether the site is bilingual throughout. If yes, build the language switcher into the header and add a language field to every content record — retrofitting this later means touching every single entry.

Also confirm the spelling: the prototype uses **"Dajon Kamo!"** as supplied. The more common Cebuano spelling is **"Dayon Kamo!"**

<div class="note"><b>For the content team:</b> every card above needs one landscape photo at 1200×800 or larger, plus a caption and photographer credit. The dashed boxes show exactly which shot goes where. Sites marked <em>Opening soon</em> must stay labelled that way until the facility is actually open to visitors.</div>

<div class="note"><b>Needs filling in:</b> departure and arrival times, fares by class, and booking links for each operator. ${todo("ADD SCHEDULES")} ${todo("ADD FARES")} — these change often, so the developers should build this table as an editable CMS collection, not as hard-coded markup.</div>

 <div class="note"><b>Three things before this page goes live.</b> First, room counts: only four establishments publish one, so the rest show “—”. The BPLO holds declared room counts on every business permit application — pull them and fill the column. Second, the contact numbers here came from a provincial tourism listing and have not been verified with the establishments themselves; call each one to confirm before publishing. Third, the “Find on Maps” links are generic Google Maps searches — they work, but they are a fallback. Ask each establishment for its actual Google Business Profile and Facebook Page URL and drop them into the <code>gbp</code> and <code>fb</code> fields; a claimed profile shows their real photos, hours and reviews, which is far better for them and for the visitor.</div>
<div class="note"><b>A simple way to collect those links:</b> add the two URL fields to the annual business permit renewal form for accommodation establishments. The BPLO already contacts every one of them once a year, and the list stays current without anyone chasing it.</div>

<div class="note"><b>Needs filling in:</b> confirmed travel times, fares, terminal bay assignments and first/last trip times from the Maasin Integrated Bus Terminal. ${todo("ADD FARES & TIMES")}</div>

<div class="note"><b>Publish the official fare matrix here.</b> Visitors are most often overcharged on the port-to-hotel trip, and a fare table on the city's own website is the simplest protection there is. Source it from the City Transport / LTFRB-approved matrix and show the approval date so it carries authority.</div>

TO BE ADDED BACK:

<div class="sechead" style="margin-top:34px"><div><div class="eyebrow">Terminal</div><h2>Maasin Integrated Bus Terminal</h2></div></div>
      <div class="portals">
        <div class="portal">${photo("Bus terminal frontage and bays")}<div class="pbody"><h3>Location &amp; hours</h3><p>${todo("ADD ADDRESS")} ${todo("ADD OPERATING HOURS")}</p></div></div>
        <div class="portal">${photo("Terminal bay signage")}<div class="pbody"><h3>Bay assignments</h3><p>Which bay serves which route — ${todo("ADD BAY MAP")}</p></div></div>
      </div>
