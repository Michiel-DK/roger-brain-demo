# roger brain demo — feedback (André → Michiel)

Notes on the demo landing page, mapped against `index.html`. More detailed phrasing to follow by text.

---

## 1 · Hero header — make it restaurant-focused
**Where:** hero section (`<h1>` + `.sub`, ~lines 237–240)
**Now:** "An operations brain that grades its own work."

- Rewrite the header in restaurant language that describes the **whole process**, not the abstract "brain."
- Name the concrete loop: **uploading invoices → tracking inventory →** ordering/refilling.
- The current headline reads as internal/AI-lab framing; a restaurateur should recognize their own workflow in the first line.

## 2 · Core pitch — it gets smarter, so you touch it less
**Where:** hero `.sub` + carries through sections 03 (doubt) and 04 (ask)

- Lead the pitch with: **the system learns as you use it**, and needs **less interface reliance over time**.
- The mechanism is **random sampling** — the sampling is what makes it smarter and steadily reduces the questions it has to ask.
- Frame the direction of travel: more use → fewer prompts → less screen time. That's the promise.

## 3 · "What's needed for tomorrow" should reflect programmed parameters
**Where:** section 02 · refill ("Orders start from the forecast", ~lines 297–309)

- The mockup currently frames the brain as **just inventory tracking** — that's fine as far as it goes.
- But the short-term / "what's needed for tomorrow" view should **also reflect programmed parameters**, not only forecast math:
  - each **supplier's delivery lead time**
  - **which days** you'd actually contact/order from each supplier
- So the tomorrow view respects real supplier schedules, not just demand.

## 4 · "Connect everything" section — model the org
**Where:** section 01 · learn / the group-brain concept (~lines 283–288)

- Mention **defining relationships between venues**: **headquarters vs. multiple locations**.
- Add an **easy way to duplicate an existing venue setup** (clone a location's configuration to stand up a new one fast).

## 5 · Alerts should feel intentional, not random
**Where:** section 04 · ask (the pill, ~lines 393–400)

- The brain **shouldn't message at random**.
- Tie alerts to **morning runs of specific items**, prioritized by **how perishable** the item is.
- Goal: prompts read as **intentional and scheduled**, not arbitrary.

## 6 · Receiving — keep it, add a photo option
**Where:** section 02 door beat / receiving flow (~lines 311–336)

- Receiving looks **solid as-is**.
- Keep: **upload the invoice.**
- Add: an option to **snap a photo of what actually arrived** (alongside the invoice).
