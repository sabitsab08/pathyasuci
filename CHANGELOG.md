# পাঠ্যসূচি — Bug Fixes & Improvements

## 🐞 Bugs fixed

### 1. `index.html` was broken (critical)
It was an incomplete copy of the app: **no `<script>` at all**, missing the
Progress and My Goal pages, and missing the mobile bottom nav. Nothing was
clickable. Meanwhile all 28 course/syllabus sub-pages link back to
`pathyasuci.html` — the real app. Since a web server serves `index.html` by
default, the hosted site landed on a dead page.

**Fix:** `index.html` is now a redirect to `pathyasuci.html` (meta-refresh +
JS + a manual link fallback, so it works with JS disabled too).

### 2. Dashboard showed fake data from the wrong subject
The dashboard, activity feed, upcoming exams and all four goal cards were
hardcoded with **Pharmacology / Biochemistry / Pathology / Microbiology** — a
medical curriculum — while the actual courses are Chemistry, Physics and
Mathematics. Other invented figures: `মোট কোর্স = 8` (really 14),
`সম্পন্ন অধ্যায় = 47`, `পড়ার সময় = 24h`, `+5 এই সপ্তাহে`, and a weekly
reading-hours chart with entirely made-up bar heights.

**Fix:** every one of these now derives from real progress. See below.

### 3. Totals were wrong until a syllabus page was opened
`prog_<id>_total` only gets written after you visit that course's syllabus
page, so the dashboard under-reported the total topic count.

**Fix:** authoritative per-course topic counts live in `COURSES_META`, so
totals are correct on first load (231 topics across 14 courses).

### 4. Progress "done" count could go stale
Read only the `prog_<id>_done` mirror key.

**Fix:** `getCourseDone()` prefers the raw `prog_<id>` progress map and falls
back to the mirror, and clamps to the course total.

### Verified clean
All 14 syllabus files were checked: `TOTAL`, the `UNITS` array sums, and the
actual `.topic-item` counts all agree. All 29 internal links resolve.

---

## ✨ Now fully dynamic

One source of truth (`COURSES_META` + `STUDENT`) drives everything:

| Area | Before | After |
|---|---|---|
| Stat cards | hardcoded 8 / 47 / 24h | live course count, credits, done / total / remaining topics, overall % |
| Greeting | "স্বাগতম, Jayed" | time-of-day aware, name from `STUDENT` |
| Recent activity | 4 fake medical rows | real courses you've progressed on, most-complete first |
| Upcoming exams | 4 fake exams | **Focus panel** — least-complete courses, each links to its syllabus |
| My Goal | 4 hardcoded goals | one auto-goal per department with real % and status |
| Weekly chart | fabricated hours | real department-completion breakdown |
| Progress list | fixed order | sorted least-complete first (more useful) |
| Sidebar profile | hardcoded twice | single `STUDENT` object; initials auto-derived |

Bengali numerals are used throughout via a `toBn()` helper.

To change the student, edit the `STUDENT` object at the top of the script —
nothing else needs touching.

---

## 📱 Mobile improvements

**Main app (`css/home-style.css`)**
- Drawer + bottom nav breakpoint raised 768px → **900px**, so tablets get the
  touch layout too
- Tap targets enlarged (icon buttons 40px, course rows, topic rows ≥40px)
- Stats stack to a single column under 380px so numbers stay legible
- `overflow-x: hidden` guard + `min-width: 0` on cards — no more sideways scroll
- Progress hero stacks vertically and centres on mobile
- `-webkit-tap-highlight-color` removed; proper `:active` feedback on touch
- Empty-state styling (`.dash-empty`) for when there's no progress yet

**All 28 course & syllabus pages**
- Added a mobile block to each (they only had one minimal media query)
- **Syllabus topic rows are now 44px tall** — comfortable to tap when checking
  off topics, which is the main thing you do on a phone
- Compact topbar, hero, unit cards and typography at ≤680px and ≤380px

---

## ✅ Testing

Verified headlessly (jsdom) across three states — brand-new user, partial
progress, and everything complete:
- Dashboard, Progress and My Goal render correct figures in all three
- Empty states appear and disappear correctly
- **Integration:** ticking topics on `syllabus-ch7202.html` correctly flows
  through to the dashboard and progress bars
- Page navigation, search (hits and no-results), dark mode all working
- JS syntax clean; CSS braces balanced in all 29 stylesheets; no broken links

---

# Round 2 — onboarding, light default, footer

## 👋 Welcome popup with a mascot

Meet **পাতা** — an open-book character drawn as inline SVG (no image files, no
external requests). It floats gently, waves, and blinks. Three steps:

1. **Name first.** "হ্যালো! আমি পাতা 📖" then asks what to call them. Empty
   input is blocked with an inline message; Enter key works. The name saves to
   `localStorage` and immediately updates the sidebar, the avatar initials and
   the dashboard greeting.
2. **Features.** Courses, Syllabus (tap a topic to tick it), Progress, My Goal —
   one line each on what the screen is actually for.
3. **Dark mode tip.** Explains the app opens in light mode and to switch to dark
   if they're reading at night or their eyes get tired, pointing at the moon icon.

Details:
- Shows **once**, on first visit only (`ps_onboarded` flag)
- "এখন থাক" skips; Escape closes
- Clicking the sidebar profile card **replays** it, so the name can be changed later
- Respects `prefers-reduced-motion` — the mascot holds still for those who ask
- Background scroll is locked while it's open

### Name privacy
Previously the name **"Jayed H." was hardcoded**, so every visitor saw it. The
fallback is now the neutral **"শিক্ষার্থী"**, and each visitor's own name lives
in their own browser's `localStorage`. Nobody sees anyone else's name.

## ☀️ Light mode is now the default

Every page used to read the OS setting (`prefers-color-scheme`), so anyone whose
phone was in dark mode got a dark app on first load. All **29 pages** now open in
light mode; dark is only used when the visitor turns it on themselves, and that
choice is remembered.

## 🏷️ Footer on every page

`Developed By: নিপাতনে সিদ্ধ Softwares` — added to all 29 pages. On the main app
it sits inside `<main>`, so it shows on all five tabs. Theme-aware and centred.

## ✅ Testing
- All 29 pages: open in light mode **even when the OS prefers dark**, footer present
- Onboarding: first visit, empty-name validation, name persistence, returning
  visit (no re-show), skip path, Escape
- Syllabus topic ticking and the dark-mode toggle still work after the changes

---

# Round 3 — professional tone

The onboarding copy was written in the informal **তুমি** register while the rest
of the app already used the formal **আপনি**. For an honours-level project the
whole interface should read as academic software, so all copy is now formal and
consistent.

| | Before (তুমি) | After (আপনি) |
|---|---|---|
| Step 1 title | হ্যালো! আমি পাতা 📖 | পাঠ্যসূচিতে স্বাগতম |
| Step 1 body | ...বলো তো, তোমাকে কী নামে ডাকব? | ...শুরু করার আগে আপনার নামটি লিখুন। |
| Placeholder | তোমার নাম লিখো | আপনার নাম |
| Error | একটা নাম লিখো, তাহলেই এগোতে পারব। | এগিয়ে যেতে একটি নাম লিখুন। |
| Step 2 title | যা যা করতে পারবে | প্রধান সুবিধাসমূহ |
| Step 3 title | চোখ আরাম চাইলে | প্রদর্শন মোড |
| Final button | চলো শুরু করি | শুরু করুন |
| Skip / Back | এখন থাক / পেছনে | এড়িয়ে যান / পূর্ববর্তী |

Also changed:
- **Terminology tightened** — শেখার ফলাফল → **শিখনফল**, ট্যাপ করো → **ক্লিক করুন**,
  নিজেই আপডেট হয় → **স্বয়ংক্রিয়ভাবে হালনাগাদ**
- **The mascot no longer introduces itself in the first person.** It's still
  there as a visual, but the copy speaks as the application, not as a character.
- **Emoji removed from interface copy** (👋 📖 🎉 🙂). The avatar fallback is now
  the initial শি instead of an emoji.
- **Courses page placeholder** rewritten — it said
  "এই কোর্সের details এখানে দেখাবে — তুমি বললে add করব", which was informal,
  mixed English into Bengali, and addressed the developer rather than the user.
  Now: "কোর্সের বিস্তারিত তথ্য দেখতে উপরের তালিকা থেকে একটি কোর্স নির্বাচন করুন।"
- **Empty states** rewritten in the same register.

Register audit across all 29 pages: no informal forms remain.

---

# Round 4 — syllabus tab fixed, mascot named

## 🐞 The Syllabus tab had a dead accordion (real bug)

Reported symptom: Chemical Thermodynamics was always expanded on the Syllabus
tab. The cause was `class="syllabus-item open"` hardcoded on the first item.

But removing `open` exposed a bigger problem. The click handler was:

```js
const sub = h.querySelector('[data-sylhref]');
if (sub) { window.location.href = sub.dataset.sylhref; }
else { h.closest('.syllabus-item').classList.toggle('open'); }
```

**All 14 rows have `data-sylhref`**, so `if (sub)` was always true and the
`else` toggle branch was unreachable. The 14 `.syllabus-topics` preview blocks —
**82 unit lines of real syllabus content** — could never be shown. The first row
appeared to work only because it was hardcoded open, and clicking it still
navigated away rather than collapsing.

**Fix — two clear affordances per row:**
- **The row** expands an inline preview of that course's units
- **The arrow (→)** opens the full syllabus page

Also: only one row stays open at a time so the list stays scannable; the arrow
stops propagation so it never toggles; both controls are keyboard reachable
(`role`, `tabindex`, `aria-expanded`, Enter/Space) with visible focus rings.

## 📖 The mascot is called পাতা again

The name is back, in the formal register: *"আমি পাতা — আপনার কোর্স ও সিলেবাসের
অগ্রগতি সংরক্ষণে সহায়তা করব।"* A small name label also sits under the mascot on
the green stage, so the character is identified without childish first-person copy.

## ✅ Testing
- No row auto-opens; row click expands (5 unit lines on course 1); a second row
  click closes the first; clicking again collapses
- Arrow navigates and does **not** toggle; `role`/`aria-label`/`tabindex` all set
- Regression: 29 pages still open in light mode with the footer; onboarding's
  three steps, name persistence and initials still work; syllabus topic ticking
  still writes progress; no informal Bengali remains
