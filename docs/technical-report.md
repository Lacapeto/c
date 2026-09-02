# C2P Tech Hub — Technical Report

**Course:** COEN 554 Web Programming
**Artefact:** Promotional website for the C2P Tech Hub (Concept to Production), Department of Computer Engineering, Ahmadu Bello University, Zaria
**Stack:** HTML5, hand-written CSS3, JSON. No JavaScript, no framework, no build step.
**Date:** August 2026

---

## Contents

1. [Visual design rationale](#1-visual-design-rationale)
2. [JSON data structure definitions](#2-json-data-structure-definitions)
3. [HTTP/HTTPS and MIME types](#3-httphttps-and-mime-types)
4. [CMS selection justification](#4-cms-selection-justification)

---

## 1. Visual design rationale

### 1.1 The constraint that shaped everything

The brief prohibits JavaScript, CSS frameworks, and external requests of any kind. That is not a cosmetic restriction — it decides the architecture. Three consequences run through every decision below:

- **No runtime data binding.** The four JSON files cannot be fetched and rendered. They are the canonical schema; the HTML is a hand-maintained projection of them. Section 2 covers what this costs.
- **No computed values.** Anything a script would normally derive — a monogram from a name, an "upcoming vs past" flag from today's date — has to be stored in the data or written into the markup.
- **No external assets.** No Google Fonts, no icon library, no image CDN. Typography falls back to the system stack, icons are hand-written inline SVG, and all photography is downloaded, processed and served locally (§1.6).

The site is therefore fully functional from a local folder with no network at all.

### 1.2 Colour system

The site is monochrome: white `#ffffff`, near-black `#111111`, and a pure neutral grey ramp between them. There is no hue anywhere in the interface, so nothing competes with the photography or the logo, and hierarchy is carried by weight, spacing and contrast instead of colour. All values are declared as custom properties in `:root` in `assets/css/main.css`, so the palette has exactly one definition site.

| Token | Value | Role |
|---|---|---|
| `--color-blue` | `#111111` | Accent. Primary fills, links, focus rings |
| `--color-blue-hover` | `#2f2f2f` | Charcoal step for hover on solid fills |
| `--color-blue-active` | `#000000` | True black for active/pressed |
| `--color-blue-tint` | `#f2f2f2` | Section washes, hover backgrounds |
| `--color-blue-tint-strong` | `#e0e0e0` | Borders inside washed areas |
| `--color-ink` | `#0f0f0f` | Headings |
| `--color-body` | `#474747` | Body copy |
| `--color-muted` | `#666666` | Muted labels, captions, metadata |
| `--color-border` | `#e4e4e4` | Hairline borders |
| `--color-surface` | `#f7f7f7` | Off-white page washes |

**On the token names.** The `--color-blue-*` names describe the *role* they have always had — the single filled accent — not the hue. When the palette moved from deep blue to near-black, renaming them would have meant editing roughly 400 declarations for no visual difference and a large diff to review, so the role names stayed and only the values changed. Read "blue" as "accent" throughout the stylesheet.

Because the accent is now the darkest value rather than a mid-tone, the hover/active logic inverted: hover steps *lighter* to charcoal and active steps *darker* to true black. A press still reads as a downward step regardless of which state preceded it, which was the original intent.

**The one exception to monochrome.** Form validation keeps colour, because it is the only place on the site where colour carries meaning rather than decoration, and greying it out would delete a real signal from the join and contact forms. Both values are deep and desaturated so they register as a state change rather than as brand colour. Colour is never the sole cue — an invalid field also gets `.error-text` beside it — which is what WCAG 1.4.1 actually requires.

**Contrast, measured rather than assumed.** Every combination the site actually produces was computed against the WCAG 2.1 relative-luminance formula:

| Combination | Ratio | Level |
|---|---|---|
| `--color-blue` on white | 18.88:1 | AAA |
| White on `--color-blue` | 18.88:1 | AAA |
| `--color-blue-hover` on white | 13.39:1 | AAA |
| `--color-ink` on white | 19.17:1 | AAA |
| `--color-body` on white | 9.29:1 | AAA |
| `--color-muted` on white | 5.74:1 | AA |
| `--color-muted` on `--color-surface` | 5.36:1 | AA |
| `--color-muted` on `--color-blue-tint` | 5.13:1 | AA |
| `--color-danger` `#a3231c` on white | 7.46:1 | AAA |
| `--color-success` `#176b40` on white | 6.53:1 | AA |

Moving to a monochrome ramp raised every ratio on the site — the previous palette's weakest combination was `--color-muted` on the accent tint at 4.56:1, a hair over the 4.5:1 AA threshold; the same combination now measures 5.13:1. The muted grey still has to clear 4.5:1 on three different grounds (white, the surface wash, and the accent tint) because `.data-note` and `.form-note` put 13px muted text on exactly those backgrounds, and `#666666` was chosen as the lightest value that does so.

### 1.3 Radii and elevation

Every interactive control is `--radius-btn: 15px`. Buttons and tags were previously full pills (`999px`), which read as soft and consumer-facing; 15px is the anchor value the rest of the scale is set against, so the whole UI reads as one family.

| Token | Value | Use |
|---|---|---|
| `--radius-btn` | 15px | Buttons, inputs, anything interactive |
| `--radius-sm` | 8px | Tags, small chips |
| `--radius-md` | 15px | Cards, panels, media frames |
| `--radius-lg` | 18px | Large containers |
| `--radius-xl` | 22px | Full-bleed bands |
| `--radius-pill` | 999px | Retained only for genuinely circular elements |

Minimalism leans on hairlines rather than drop shadows, so the elevation ramp is much lighter than a typical one and neutral-black rather than tinted. Shadows mark that something has *lifted* — a hover, an open panel — not that it is permanently floating.

### 1.4 Type scale

System font stack only, since web fonts would require an external request:

```
-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue",
Arial, "Noto Sans", sans-serif
```

Beyond the constraint, this is the fastest possible option: zero bytes downloaded, zero layout shift from font swapping, and native rendering on every platform.

The scale is roughly a 1.22 ratio, declared mobile-first and enlarged at breakpoints. The mobile baseline and the 1024px+ values:

| Token | Mobile | ≥1024px | Use |
|---|---|---|---|
| `--fs-eyebrow` | 12px | 12px | Eyebrows, uppercase labels |
| `--fs-xs` | 13px | 13px | Metadata, hints, captions |
| `--fs-sm` | 15px | 15px | Secondary copy, form fields |
| `--fs-base` | 16px | 16px | Body |
| `--fs-lg` | 20px | 21px | Card headings, lede |
| `--fs-xl` | 24px | 26px | Sub-headings |
| `--fs-2xl` | 30px | 36px | Section headings |
| `--fs-3xl` | 36px | 48px | Page titles |
| `--fs-4xl` | 42px | 56px | Hero (60px at ≥1280px) |

Only the display end of the scale grows. Body text stays at 16px at every width, because the comfortable reading size does not change with the viewport — only the amount of surrounding space does. Paragraphs are capped at `68ch`; the page banner lede at `60ch`; the CTA band at `52ch`.

Headings use `text-wrap: balance` and negative tracking (`-0.022em`), which the large sizes need to avoid looking loose.

### 1.5 Motion

The brief asked for slight animation, so the motion language is deliberately small: nothing loops, nothing bounces, and nothing travels more than about 14px. All of it lives in one place — section 16 of `main.css` — so the whole language can be read, audited and switched off together.

Three rules govern every animation on the site:

1. **Only `transform` and `opacity` are animated.** Both are compositor properties, so no animation here can trigger layout or paint. A hover that animated `width` or `top` would cost a reflow on every frame.
2. **Entrances are one-shot and short.** The hero staggers its eyebrow, headline, lede, buttons and fact list in over roughly 460ms total, using `animation-delay` rather than duration so each element still moves quickly.
3. **Elements animate *from* a displaced state, never *to* one.** With no JavaScript there is no scroll observer, so entrance animations run on load. Writing them the other way round would leave content permanently invisible if an animation never ran.

That last point is also why below-the-fold sections only fade rather than rise: without scroll position, a transform would be a lie about where the element is, and a long page would animate content the reader cannot see yet.

The named effects:

| Effect | Where | What it does |
|---|---|---|
| `rise-in` | Hero children | 14px rise + fade, staggered |
| `hero-drift` / `hero-drift-side` | Hero logo render | Settles into place from a 3–4% offset |
| Underline sweep | Inline links | `background-size` grows 0 → 100% on one axis |
| Nav hairline | Nav links | Grows outward from centre under the label |
| Arrow nudge | Buttons with an arrow icon | Icon slides 3px on hover |
| Lift | Buttons, cards, gallery items | `translateY(-3px)` plus a shadow step |
| Desaturate | All photography | `grayscale(1)` easing back toward tone on hover |

The link underline animates `background-size` on a linear-gradient rather than the width of a pseudo-element, because a pseudo-element sweep breaks when a link wraps across two lines. Keyboard focus gets the underline immediately with `transition: none` — a focus indicator should never be the slow one.

**`prefers-reduced-motion` is treated as a requirement, not a nicety.** Vestibular disorders make even small parallax genuinely unpleasant, so the query removes transforms and durations outright rather than merely shortening them, and explicitly resolves every animated entrance to its final state so no content is left invisible. Hover lifts and image zooms are disabled there too, since those are movement as well.

There is a matching `@media print` block: print engines may snapshot a page before an `animation-delay` has elapsed, and every animated element starts at `opacity: 0`, so without that block the hero could print as a blank rectangle.

### 1.6 Photography and image pipeline

Every image on the site was previously a locally generated SVG placeholder. Those have been replaced with real photography across the gallery, project and event slots — 19 images in total.

**Sourcing.** Images were retrieved from the [Openverse](https://openverse.org) API, which indexes openly licensed work, filtered to CC-licensed results. Automated search relevance was poor for several slots, so candidates were fetched in batches and selected by eye rather than taking the first hit; the shortlists and the final choices are recorded in `assets/images/credits.json`, which stores the title, creator, licence and source URL for each file.

> **Attribution is not yet rendered on the site.** CC-BY and CC-BY-SA both require visible credit. `credits.json` holds everything needed, but a credits block still has to be added to the gallery page — or the images swapped for photographs of the actual hub — before this site is published. See §1.6.1.

**Processing.** Each image is centre-cropped to its slot's aspect ratio with a slight upward bias (faces sit above centre in most photographs), resampled with Lanczos, and written as a progressive JPEG at quality 82. Slot dimensions are 1200×800 for gallery, 1000×625 for projects and 1000×560 for events, and the `width`/`height` attributes in the markup are set from the file's true pixel size so the browser reserves correct space and nothing shifts as images land.

**Desaturation is a CSS concern, not a baked-in one.** The files on disk are the colour originals; `filter: grayscale(1)` in the stylesheet is what makes them monochrome, easing back toward tone on hover. Deleting that one rule restores the site to full-colour photography without touching a single image or line of markup — which matters, because these are placeholders.

#### 1.6.1 Known limitation

The photographs are stand-ins showing the *kind* of activity each slot describes, not the hub itself, and several are visibly not Nigerian university settings. They are there so the layout can be judged with real images in it. Replacing them with photographs of the actual hub — which also removes the attribution obligation above — is outstanding work.

### 1.7 Layout strategy

**Mobile-first, genuinely.** `main.css` holds the complete small-screen baseline. Every rule in `responsive.css` sits inside a `min-width` media query and only ever adds. There are five media blocks:

| Breakpoint | What changes |
|---|---|
| 480px | Two-up card grids, event rows gain their date column, footer bottom goes horizontal |
| 768px | Two-up grids, split layouts, three-column footer, two-column form grid, larger type |
| 1024px | Three- and four-up grids, **horizontal nav replaces the checkbox menu**, hero annotations lift out and overlap the figure |
| 1280px | Header CTA button appears, final type scale |
| print | Strips nav, footer, CTAs and buttons; forces accordions open; appends URLs after external links |

Breakpoints were chosen from where *this* content breaks, not from device names. The clearest example is the navigation: nine top-level links plus the brand lockup do not fit on one row at tablet widths without shrinking the labels below a comfortable size, so the checkbox-hack panel is kept until 1024px rather than the more usual 768px.

**Grid and Flexbox by role.** Grid handles two-dimensional page structure — card grids, the footer, the `concept → production` arc, form field pairs. Flexbox handles one-dimensional component internals — the header row, button contents, icon-plus-text pairs, tag clusters. Grid columns use `minmax(0, 1fr)` rather than `1fr`, because the default `min-width: auto` on a grid item lets long content force a column wider than its share.

That distinction is not academic. A defect found during the responsive pass: the footer's "Reach Us" column contains `admin@c2phub.com`, a single token no browser will break. At 768px the footer link columns are about 120px wide, but the address gave that `<li>` a min-content width of 161px, which burst out of its column and pushed **every page** 6px wider than the viewport. `overflow-wrap: anywhere` on `.footer-group a` lets the address wrap to its column instead of dictating its width. All nine pages were then re-tested at 320, 480, 768, 1024 and 1440px with zero horizontal overflow.

### 1.8 CSS-only interactivity

Five interactive components, none using script:

| Component | Mechanism | Where |
|---|---|---|
| Mobile navigation | Hidden `<input type="checkbox">` + `<label>` + sibling combinator | Every page, below 1024px |
| FAQ / detail accordions | Native `<details>`/`<summary>` with `[open]` attribute selector | `about.html`, `join.html` |
| Project category tabs | Radio group + `:checked ~` sibling selector | `projects.html` |
| Image lightbox | `:target` pseudo-class on a fixed overlay | `gallery.html` |
| Form validation cues | `:valid`, `:invalid`, `:focus`, `:placeholder-shown` | `join.html`, `contact.html` |

Two are worth explaining.

**Radio tabs rather than `:target` tabs.** Radio buttons make the selection mutually exclusive by definition, and selecting a tab does not push a fragment into the URL or scroll the page — both of which `:target` would do. The radios are visually hidden but remain focusable, so the strip stays keyboard-operable as a native radio group.

**The `:placeholder-shown` guard.** The validation rule is `.input:not(:placeholder-shown):invalid`, not `.input:invalid`. Without the guard, every empty required field would paint red on first load, before the visitor had typed anything. This is why every text input carries a `placeholder` attribute: it is load-bearing for the styling, not decoration.

### 1.9 Accessibility

Semantics first: one `<h1>` per page and no skipped heading levels (verified across all nine pages), `<header>`/`<nav>`/`<main>`/`<footer>` landmarks throughout, `<article>` and `<section>` where the content warrants it, `aria-current="page"` on the active nav link, a skip link as the first focusable element, and `aria-labelledby` tying each section to its heading.

Every form control has an associated `<label>`; the 24 controls on `join.html` and 8 on `contact.html` were checked for label association. Icons are `aria-hidden="true"` with `focusable="false"` so they are skipped by assistive technology and by tab order in older engines. Icon-only links carry `.sr-only` text.

**Focus is never removed, only restyled.** A global `:focus-visible` rule provides a 3px blue outline at 3px offset. The components that hide their real control provide their own indicator: `.nav-toggle:focus-visible + .nav-toggle-label`, `.accordion__summary:focus-visible`, `.tabs__radio:focus-visible ~ .tabs__list [for=...]`, and `.gallery__item:focus-visible`. Form fields replace the outline with a blue border plus a 3px tinted ring, which is a stronger cue at 12:1 contrast.

Three further defects were found and fixed during the accessibility pass:

1. **Links inside running text were invisible.** `reset.css` strips `color` and `text-decoration` from every anchor so navigation and buttons can be styled from scratch. Only `.prose a` opted back in, which left links inside form hints, notes, table cells and contact values rendering as plain body text — with neither a colour nor an underline cue. That fails WCAG 1.4.1 (Use of Colour), and in fact more severely, since colour alone was not the only missing signal. Every component carrying sentences now opts back in.
2. **A link nested inside a `<label>`.** On the join form, the directory link sat inside the label bound to the consent checkbox. Clicking it would follow the link *and* toggle the checkbox, so the visitor would navigate away having silently changed an answer they never touched. The link was moved out to its own line.
3. **Breadcrumb touch targets.** The links were only as tall as their line box, about 17px, short of the 24px WCAG 2.2 asks for. Vertical padding brings the hit area to 25px without moving anything visually.

`prefers-reduced-motion: reduce` is honoured in `reset.css`, collapsing all transitions and animations and disabling smooth scrolling.

---

## 2. JSON data structure definitions

### 2.1 The central design decision

The brief requires JSON files modelling the site's content entities. It also prohibits JavaScript. Those two requirements are in direct tension, and how that tension is resolved is the most important architectural decision in the project.

**The JSON files are not fetched at runtime. They cannot be.** `fetch()` is JavaScript. So is `XMLHttpRequest`. There is no declarative HTML mechanism for reading a JSON file and rendering it. The files therefore serve as the **canonical data model and the editing surface**, while the HTML carries a hand-maintained projection of the same records as static markup.

This is stated explicitly rather than left implied: every page that mirrors records carries a `DATA MODEL NOTE` comment block in its markup, and a visible `.data-note` callout tells the reader the same thing. The editing workflow is: change the record in `data/*.json` first, then reflect the change in the HTML.

In a JavaScript-permitted build, the same files would drive rendering unchanged — the schema is designed to be consumed by a template, not to be convenient for hand-copying. That is the point of the exercise: the data model is complete and honest, and only the transport is missing.

**What this costs, stated plainly:** the projection can drift. There is no mechanism that fails when the HTML and the JSON disagree. In a build-step project this would be enforced by the template; here it is enforced by discipline and by the comment blocks. Section 4 returns to this as the single strongest argument for migrating to a CMS.

### 2.2 Envelope common to all four files

Every file shares the same wrapper, so a consumer can treat them uniformly:

| Key | Type | Purpose |
|---|---|---|
| `$comment` | string | Documentation. JSON has no comment syntax, so provenance, the no-JavaScript caveat and any per-file warnings are carried in a conventional `$`-prefixed key |
| `entity` | string | Singular entity name — `project`, `student`, `event`, `program` |
| `schema_version` | string | SemVer. `1.0.0` across all four. Lets a future consumer detect a breaking field change |
| `last_updated` | string | ISO 8601 date of the last edit |
| `record_count` | integer | Number of records, so a consumer can verify a truncated file |
| `fields` | object | Field-by-field schema: name → type, required/optional, allowed values and rationale. Self-documenting |
| *`<entity plural>`* | array | The records themselves — `projects`, `students`, `events`, `programs` |

Two conventions run through every entity:

- **`id` is a stable, prefixed, sortable string** (`proj-`, `stu-`, `evt-`, `prog-`). It doubles as the HTML element `id` and the in-page anchor target, so a record and its rendered fragment share one identifier.
- **`is_placeholder` is a required boolean on every record.** The site ships with generated sample content, and the difference between real and provisional data must be machine-checkable rather than a matter of memory. It also drives the visible "Placeholder record" tags in the UI, so the distinction reaches the reader.

### 2.3 `data/projects.json` — 5 records

| Field | Type | Purpose |
|---|---|---|
| `id` | string, required, unique | Slug key, prefix `proj-`. Also the in-page anchor |
| `title` | string, required | Display name |
| `date` | string, required | ISO 8601 `YYYY-MM-DD`. The most significant milestone date, not necessarily the start. ISO so records sort lexicographically without parsing |
| `category` | string, required | Human-readable label shown on the card |
| `category_id` | string, required | Machine key grouping the project under a filter tab. One of `ai`, `embedded`, `iot` |
| `description` | string, required | Two to four sentences on what it does and how it was built |
| `image_url` | string, required | Root-relative path to the illustration |
| `image_alt` | string, required | Written as though the real photograph were in place. Never empty, never a restatement of the title |
| `team` | array of string, required | Names of the students who built it |
| `team_note` | string, optional | Present only where the team list is not yet confirmed |
| `status` | string, required | One of `Award winner`, `In development`, `Prototype`, `Concept`, `Deployed` |
| `tags` | array of string, required, 2–5 items | Technologies and domains; the card's meta row |
| `is_flagship` | boolean, required | True for the single project featured on the homepage. Exactly one record has it |
| `is_placeholder` | boolean, required | True for the four illustrative records; false for ParaVision |

The split between `category` and `category_id` is deliberate. Display strings need to change without breaking anything; the tab filter needs a key that does not. Collapsing them would make renaming a category a structural change.

### 2.4 `data/students.json` — 12 records

| Field | Type | Purpose |
|---|---|---|
| `id` | string, required, unique | Prefix `stu-`, zero-padded to three digits so records sort correctly as strings |
| `name` | string, required | Full name as the member wishes it displayed |
| `initials` | string, required, 2 chars | Precomputed avatar monogram |
| `level` | string, required | Nigerian convention: `100L`–`500L` |
| `department` | string, required | Home department. Not a constant — the hub is open across departments |
| `focus_area` | string, required | Technical area; used for mentor matching and grouping |
| `project` | string, required | Title of the hub project the member is attached to. Corresponds to a `title` in `projects.json` |
| `is_placeholder` | boolean, required | **True for all twelve.** Every record must be replaced before publication |

**`initials` is the clearest illustration of the constraint shaping the schema.** Deriving `"AB"` from `"Aisha Muhammad Bello"` is two lines of JavaScript and would normally never be stored. Because scripting is prohibited, the derived value has to be precomputed and persisted — the data model absorbs work that would otherwise happen at runtime. The `fields` block says so explicitly, so a future maintainer does not "tidy it away".

Note also that `project` joins to `projects.json` by *title* rather than by `id`. This is a deliberate trade-off, and the weaker of the two options: a title is what a human editing the file by hand can actually verify, but renaming a project silently breaks the join. With a build step this would be `project_id`.

### 2.5 `data/events.json` — 6 records

| Field | Type | Purpose |
|---|---|---|
| `id` | string, required, unique | Prefix `evt-`, zero-padded |
| `title` | string, required | Event name |
| `date` | string, required | ISO 8601 start date. Maps directly onto schema.org `startDate` |
| `end_date` | string, required | ISO 8601 end date. Equal to `date` for single-day events, so consumers never special-case a null |
| `time` | string, required | Local start/end time, WAT (UTC+1) |
| `location` | string, required | Venue as a display string |
| `category` | string, required | One of `Milestone`, `Competition`, `Bootcamp`, `Hackathon`, `Research`, `Incubation`. Corresponds to a program in `programs.json` where one applies |
| `description` | string, required | Two to three sentences: what happens and who it is for |
| `image_url` / `image_alt` | string, required | Illustration and its alternative text |
| `status` | string, required | `past` or `upcoming` |
| `registration` | string, required | Where to sign up, or that registration is closed/not required |
| `is_placeholder` | boolean, required | True where scheduling is illustrative |
| `note` | string, optional | Present only where a detail needs verifying before publication |

**`status` is stored, not derived** — the second place the constraint reaches into the schema. Whether an event is past or upcoming is a comparison between `date` and today, and that comparison requires JavaScript. So the answer is persisted, and it follows that the Upcoming/Past split on `events.html` is maintained by hand: when a date passes, the record's `status` changes and the markup block moves between the two sections. A stored value that silently goes stale is a genuine weakness of this design, and it is documented in the file rather than hidden.

The choice of ISO 8601 over a localised display string is what makes `date` and `end_date` useful: records sort lexicographically without parsing, and the values map straight onto the `Event` JSON-LD without transformation.

### 2.6 `data/programs.json` — 7 records

| Field | Type | Purpose |
|---|---|---|
| `id` | string, required, unique | Slug key, prefix `prog-`. Used as the element `id` and the accordion/`:target` anchor |
| `name` | string, required | Display name |
| `icon` | string, required | Key naming the inline SVG icon. **Not a file path** — the markup lives inline in the HTML, because an icon library would require an external request |
| `tagline` | string, required | One-line positioning statement, ~60 chars, used as the card subtitle |
| `description` | string, required | Two to three sentences of concrete detail |
| `format` | string, required | How the program is delivered |
| `cadence` | string, required | How often it runs |
| `duration` | string, required | Typical length of one instance |
| `capacity` | string, required | Intake size; `Open` where uncapped |
| `entry_requirement` | string, required | What a member needs before joining |
| `outcomes` | array of string, required, 2–4 items | What a participant leaves with |
| `is_placeholder` | boolean, required | **False for all seven** — these are the hub's actual offerings |

`icon` naming a key rather than a path is a third fingerprint of the no-external-requests rule: the indirection exists because the asset cannot be a file.

### 2.7 Structured data in the `<head>`

Separately from `data/`, every page embeds Schema.org JSON-LD in a `<script type="application/ld+json">` block — the single script element the brief permits, because it is a declarative data island rather than executable code. There are exactly nine such blocks across the site and no other `<script>` tag of any kind.

| Page | Type |
|---|---|
| `index.html` | `EducationalOrganization` |
| `events.html` | **Array** of `Event` objects |
| `projects.html` | `CreativeWork` per project |
| `contact.html` | `EducationalOrganization` with `ContactPoint` and `PostalAddress` |
| `directory.html` | `CollectionPage` wrapping an `ItemList` of `Person` |
| `join.html` | `WebPage` with a nested `FAQPage` |
| `about.html`, `programs.html`, `gallery.html` | Page-appropriate types |

All nine parse as valid JSON, verified programmatically.

---

## 3. HTTP/HTTPS and MIME types

### 3.1 What "static hosting" actually means here

There is no application server, no database and no build output. The deployment artefact is the directory itself. Every URL maps to a file on disk, and the server's entire job is to locate that file, guess or look up its media type, and stream the bytes back with the right headers. Nothing is computed per request.

### 3.2 The request/response cycle for one page load

Loading `https://c2phub.com/contact.html`:

1. **DNS resolution.** `c2phub.com` resolves to an edge address.
2. **TCP + TLS.** A TCP connection is established, then a TLS handshake negotiates the cipher suite and validates the server certificate. Only after this does any HTTP traffic flow.
3. **The document request.**
   ```http
   GET /contact.html HTTP/1.1
   Host: c2phub.com
   Accept: text/html,application/xhtml+xml,*/*;q=0.8
   Accept-Encoding: gzip, br
   ```
4. **The document response.**
   ```http
   HTTP/1.1 200 OK
   Content-Type: text/html; charset=utf-8
   Content-Encoding: br
   Content-Length: 7194
   Cache-Control: public, max-age=0, must-revalidate
   ETag: "a3f1c9..."
   ```
   The browser selects its HTML parser **on the strength of `Content-Type`, not the `.html` extension**. The `charset=utf-8` parameter is what makes the em-dashes and the `&mdash;` entities decode correctly.
5. **Subresource discovery.** The parser hits three `<link rel="stylesheet">` elements in `<head>` and issues three more GETs. Stylesheets are render-blocking: the browser will not paint until they arrive, which is why they are three small files rather than one large one, and why none of them `@import` another (an `@import` would serialise a second round trip behind the first).
6. **Images.** `<img>` elements are not render-blocking. Each `width`/`height` pair in the markup lets the browser reserve the correct box before the bytes arrive, preventing cumulative layout shift.
7. **Idle.** Because there is no JavaScript, there is no post-load execution phase at all — no parse, compile, hydrate or bind step. The page is interactive the moment it is painted.

A cold load of the heaviest page issues roughly a dozen requests: one document, three stylesheets, two logos, and the page's SVG illustrations. On a warm cache, conditional requests carrying `If-None-Match` return `304 Not Modified` with no body.

### 3.3 The MIME types this site depends on

| Extension | Correct `Content-Type` | Consequence of getting it wrong |
|---|---|---|
| `.html` | `text/html; charset=utf-8` | As `text/plain`, the browser prints the source as text. As `application/octet-stream`, it downloads the file |
| `.css` | `text/css` | **The stylesheet is refused.** See below |
| `.json` | `application/json` | Served as `text/html`, a browser tries to parse markup out of it |
| `.svg` | `image/svg+xml` | As `text/plain`, an `<img>` fails to render and shows alt text instead |
| `.png` | `image/png` | Usually still sniffed correctly from magic bytes, but not guaranteed under `nosniff` |

**Why a wrong `Content-Type` on CSS is uniquely fatal.** In standards mode — which this site is in, having a full `<!DOCTYPE html>` — browsers apply strict MIME checking to stylesheets. A `<link rel="stylesheet">` whose response arrives as `text/plain` is **discarded entirely**, with a console warning and no fallback. The HTML still renders, so the page is not blank; it renders completely unstyled. Since all presentation, layout and the five CSS-only interactive components live in those three files, a single wrong header reduces the site to a column of unstyled text with a non-functional navigation menu. There is no JavaScript to notice or recover.

This is stricter than for other types. `X-Content-Type-Options: nosniff`, which most hosts including Vercel send by default, extends the same strictness to scripts and disables content sniffing generally, so a mislabelled SVG will not be rescued by inspecting its bytes.

### 3.4 Why HTTPS specifically

Beyond confidentiality, two reasons matter here:

**Integrity.** Over plain HTTP, any intermediary can modify the response body. The classic injection is an advertising or analytics script. For this project that is not merely unwelcome — a single injected `<script>` would violate the exam's central constraint on a page that is provably clean at origin. TLS makes the document that arrives the document that was sent.

**Protocol capability.** HTTP/2 and HTTP/3 are effectively HTTPS-only in browsers. Both multiplex many requests over one connection, which suits a site of this shape — many small files rather than a few bundled ones. Under HTTP/1.1 the dozen-odd requests would contend for six connections per origin; multiplexed, they overlap freely.

### 3.5 Caching, and a defect it caused

Static assets invite aggressive caching, and Vercel serves them with long `Cache-Control` lifetimes behind content-hashed edge invalidation.

This is not theoretical. During the responsive-design pass, changes to `main.css` appeared not to take effect: the browser was holding a cached copy, and the CSSOM inspection confirmed the *old* rule text was still in the sheet while the file on disk had the new one. The verification runs had to be repeated with cache-busting query strings appended to the stylesheet URLs before the results could be trusted.

The lesson generalises to deployment: because these filenames are stable and unversioned, a visitor who has seen the old `main.css` may keep it after a redeploy until its lifetime expires or the ETag is revalidated. Vercel handles this by invalidating at the edge on each deployment, which is one concrete reason to prefer it over copying files onto a plain web host.

---

## 4. CMS selection justification

This site is hand-written HTML, CSS and JSON with no build step. The alternative — WordPress, or a modern framework such as Astro or Next.js — was evaluated and rejected for the current requirements. The defence below is conditional, not absolute.

**Performance and hosting cost.** The site is nine documents, three stylesheets totalling 62 KB, inline SVG icons, and locally served JPEG photography. Every response is a static file from an edge cache; there is no PHP process, no database query and no server-side render. Hosting is free on Vercel's static tier and would remain trivially cheap anywhere. An equivalent WordPress install requires PHP and MySQL, making the cheapest credible hosting a small VPS or managed plan, and a typical theme ships jQuery and several plugin bundles — script the brief forbids outright and that this site does not need, since all five interactive components are pure CSS.

**Security surface.** A directory of static files has essentially no attack surface: no interpreter, no database, no login, no upload path. The only dynamic element is a `mailto:` form, which submits through the visitor's own mail client. WordPress by contrast is the most-attacked application on the web, and the risk is concentrated in plugins, which must be patched continuously. For a departmental site with no full-time maintainer, an unpatched plugin is the realistic failure mode.

**Version control and workflow.** The entire site is diffable text. A change to a colour token or a student record is a legible one-line diff, reviewable in a pull request and revertible with `git revert`. WordPress content lives in a database, so the meaningful history is invisible to Git; rolling back a bad content edit means restoring a database backup.

**Maintenance burden — where the argument turns.** The honest weakness is the one documented in Section 2: because JavaScript is prohibited, the JSON files cannot drive rendering, so every record exists twice — once as canonical data and once as hand-maintained markup. Nothing enforces agreement. Adding a student today means editing `students.json` and mirroring it into `directory.html`. That is acceptable for twelve records maintained by one developer; it scales badly and drifts silently.

**Conditions that would justify migrating.** Three thresholds, any of which is sufficient:

1. **Non-technical editors.** The moment someone who does not write HTML must publish — a coordinator posting events — the manual projection becomes untenable. This is the strongest trigger.
2. **Update frequency.** Beyond roughly weekly content changes, the duplication cost exceeds the cost of running a CMS.
3. **Team size and content volume.** Past two or three contributors, or a directory in the hundreds, hand-maintained markup produces merge conflicts and drift.

If the JavaScript constraint were lifted but the others held, the correct intermediate step is not WordPress but a static site generator: Astro or Eleventy would consume these exact JSON files as its data layer, eliminate the duplication, and still deploy as static files — preserving every performance and security advantage above.

**Word count: 447**

---

## Appendix: verification performed

| Check | Result |
|---|---|
| `.js` files in project | 0 |
| `<script>` tags | 9, all `type="application/ld+json"` |
| Inline event handler attributes | 0 |
| External resource requests | 0 (only social profile hyperlinks) |
| Pages present and cross-linked | 9/9 |
| JSON-LD blocks parsing as valid JSON | 9/9 |
| `data/*.json` parsing as valid JSON | 4/4 |
| Duplicate element IDs | 0 on every page |
| `<img>` elements without `alt` | 0 of 63 |
| Broken internal links or asset paths | 0 |
| Horizontal overflow at 320/480/768/1024/1440px | 0 on all nine pages |
| One `<h1>` per page, no skipped heading levels | 9/9 |
| Form controls with associated labels | 32/32 |
| CSS-only interactive components | 5, each exercised in-browser |
