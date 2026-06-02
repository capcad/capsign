# CapSign Sign Description Schema — Specification

**Version 1.0**
**Status:** Stable
**License:** MIT (code/schema) · CC BY 4.0 (this document)

---

## 1. Purpose and scope

CapSign is a machine-readable description of the **compositional layout of a Brazilian road sign** — enough information to *redraw* the sign face, not merely to catalogue it.

It describes the **face** of a *static, fabricated* sign (the printed/painted aluminium guide signs, tourist boards, destination panels and coded warning/regulatory plates defined by CONTRAN and the DNIT *Manual de Sinalização Rodoviária*, IPR-743): its type, its colour category, and the panels, text lines, arrows, pictograms, route shields and badges that compose it.

### What this schema is

- An **interface contract** between a *producer* (anything that recognises or authors a sign — an image-recognition model, a manual editor, an import tool) and a *consumer* (anything that draws, validates, or stores the sign).
- A **layout** description: where things sit on the sign face and how they relate.

### What this schema is deliberately **not**

- **Not** an inventory format. It does not describe *where a sign is installed*, its GPS position, condition, retroreflectivity, material, or maintenance history. Inventory systems (GIS point layers with MUTCD/CONTRAN type codes) cover that; this schema is complementary to them, not a replacement. See §8.
- **Not** a rendering or CAD format. It contains no geometry, coordinates (except overall millimetre dimensions for opaque sign types), fonts, block references, layers, or drawing instructions. How a consumer turns this description into a drawing is entirely the consumer's concern.
- **Not** a model of dynamic/electronic Variable Message Signs. For programmable VMS content exchange between traffic-management systems, see DATEX II (CEN/TS 16157-4). CapSign shares some element vocabulary with it (see §9) but targets a different domain: permanent fabricated signage.

The normative source for the *visual* rules a sign must obey (colours per category, pictogram set, dimensional conventions, CONTRAN codes) is the DNIT IPR-743 manual and the CONTRAN signalling manuals. CapSign is a way to represent a sign that conforms to those rules; it does not restate them.

---

## 2. Document conventions

The key words **MUST**, **MUST NOT**, **SHOULD**, **MAY** are used in the sense of RFC 2119.

A conforming **instance** is a JSON document that validates against `capsign.schema.json` (JSON Schema 2020-12). The schema is the authoritative, machine-checkable definition; this prose document explains intent and the judgment rules a producer applies. Where prose and schema appear to differ, the schema governs structure and the prose governs interpretation.

---

## 3. Top-level shape

An instance describes either one sign or many.

- One sign: a single object under the key `sign`.
- Multiple signs (e.g. several signs in one photo): an array under the key `signs`.

```json
{ "sign":  { … } }
{ "signs": [ { … }, { … } ] }
```

A consumer MUST accept either form.

---

## 4. The sign object

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `type` | string | yes | One of the five sign types (§5). |
| `colorScheme` | string | for `modules`, `orientação`, `pictograms` | The sign's colour category (§6). |
| `confidence` | number 0–1 | yes | Producer's confidence (§7). |
| `modules` | array | for `modules`, `orientação` | The panels, top to bottom (§10). |
| `pictograms` | array of string | for `pictograms` | Ordered pictogram ids (§5.3). |
| `footer` | string | no | Footer text for `pictograms` type. |
| `width`, `height` | number (mm) | for `tree`, `special` | Overall dimensions only. |

No other top-level fields are permitted.

---

## 5. Sign types

### 5.1 `modules`
The general case: one or more stacked panels, each carrying text lines, and optionally arrows, pictograms, route shields, and coloured badges. Most guide and coded signs are this type. Detailed in §10.

### 5.2 `orientação`
A two-column destination sign: place names on the left, distances in kilometres on the right. Modelled as modules whose lines carry a `km` field. (The word *orientação* is also a colour category — §6 — but as a `type` it specifically denotes the km-column layout.)

### 5.3 `pictograms`
A row of pictograms (services, tourism) with an optional footer line. Carried directly on the sign object via `pictograms` (an ordered list of ids) and `footer`, without a `modules` array.

### 5.4 `tree`
A junction or roundabout diagram. Opaque in v1.0: a conforming instance provides only `width`, `height`, and `confidence`. The internal geometry of the diagram is out of scope for this version.

### 5.5 `special`
A custom or non-standard sign (logos, unusual colours). Opaque in v1.0, like `tree`: dimensions and confidence only.

---

## 6. Colour categories (`colorScheme`)

`colorScheme` names the CONTRAN colour category of the sign, taken from the **body** of the sign — not from a header module.

| Token | CONTRAN category | Typical body colours |
|-------|------------------|----------------------|
| `orientação` | Orientação (guide) | green ground, white text |
| `indicativa` | Indicação (information/services) | blue ground, white text |
| `turístico` | Turística (tourist) | brown ground, white text |
| `advertência` | Advertência (warning) | yellow ground, black text |
| `regulamentação` | Regulamentação (regulatory) | white ground, red/black |

**Body, not header.** A green guide sign with a blue header band is still `colorScheme: "orientação"`. The blue header is a *module-level* `backColor` override, not the sign's category. A producer MUST classify by the dominant body, and represent a differently-coloured header as a module `backColor`.

---

## 7. Colour tokens, overrides, and confidence

### 7.1 Tokens
`backColor` and `fontColor` use **only** the named tokens `green`, `blue`, `white`, `yellow`, `brown`, `red`. Hex values (`#0000AA`) MUST NOT be used. Named tokens keep the description independent of any particular colour space or palette calibration; the consumer maps tokens to its own exact colours (per the CONTRAN palette).

### 7.2 Overrides only where colour changes
`backColor`/`fontColor` appear at module, line, or segment level **only where the colour differs from the parent**. A producer MUST NOT restate the sign-level palette as a redundant override. Absence of an override means "inherit".

### 7.3 Confidence
`confidence` is the producer's self-assessment, useful for routing low-certainty results to human review.

| Range | Meaning |
|-------|---------|
| `1.0` | Fully legible, unambiguous type and content |
| `≥ 0.8` | Minor uncertainty (angle, glare, small text) |
| `≥ 0.6` | Partial obscuration; some `?` characters present |
| `< 0.6` | Flag for human review |

---

## 8. Module vs. badge — the central distinction

The most important judgment a producer makes is **whether a coloured region is a separate module or a coloured line inside a module.**

- A **separate module** exists **only** where there is a **visible dividing border** — an actual line or frame separating one panel from another.
- A coloured shield, plate, or highlight that **floats inside** a panel with **no dividing border** around it is **not** a module. It is a **line with a `backColor`** (or, if the colour change is only part of the line, a *segment* — §11), kept in the same module as the surrounding text.

**Example.** A blue `BR-277` plate sitting inside a green panel above the destinations, with no divider between them, is one module whose first line is `{ "text": "BR-277", "backColor": "blue" }` followed by the destination lines — **not** a separate header module.

A producer MUST create a separate module only when a real border or divider is visible.

---

## 9. Elements: arrows, pictograms, shields

An **element** is a graphic attached to a module or a line. Three kinds exist.

### 9.1 `arrow`
```json
{ "type": "arrow", "direction": "up-left", "position": "before" }
```
`direction` is one of the eight compass values `left right up down up-right up-left down-right down-left`.

**Arrow placement rule:** an arrow that **spans multiple lines** belongs to the **module's** `elements`; an arrow that **belongs to a single line** belongs to **that line's** `elements`.

### 9.2 `pictogram`
```json
{ "type": "pictogram", "id": "rural-tourism", "position": "before" }
```
`id` is either a **descriptive lowercase term** (for service/tourism symbols) or an **official CONTRAN code** (for standard warning/regulatory symbols, e.g. `A-18`, `R-19`). Descriptive terms keep common symbols human-readable; codes anchor standard symbols unambiguously. The descriptive vocabulary is open; a consumer maps unknown terms to a placeholder. (A recommended starter vocabulary is listed in the README.)

### 9.3 `shield`
```json
{ "type": "shield", "network": "PR", "number": "476", "position": "before" }
```
A route shield/plate. `network` is `BR` (federal) or a state abbreviation; `number` is a string (preserving leading zeros); `state` is optional. A shield is a *graphic* element placed like a pictogram — distinct from an inline text badge (§11), which is boxed text, not a drawn plate.

### 9.4 `position`
`before` = leading edge (left, for left-to-right text); `after` = trailing edge (right); `behind` = layered behind the text. Aligns conceptually with DATEX II's pictogram-position vocabulary (on-left/on-right, above/below) for cross-standard legibility, without adopting that standard's model.

---

## 10. Modules and lines

A **module** is a panel:

```json
{
  "justification": "left",
  "backColor": "blue",
  "elements": [ { "type": "arrow", "direction": "left", "position": "before" } ],
  "lines": [ … ]
}
```

| Field | Required | Notes |
|-------|----------|-------|
| `lines` | yes | The module's content lines (§11). May be empty for a coded sign that is purely a pictogram + arrow. |
| `justification` | no | `left` / `center` / `right`. |
| `elements` | no | Module-level elements (e.g. a multi-line arrow). |
| `backColor`, `fontColor` | no | Overrides where the module differs from the sign. |

A **line** (the `lineObject` form):

```json
{ "text": "Curitiba", "km": 85, "fontSize": 150,
  "elements": [ … ], "backColor": "blue", "fontColor": "white" }
```

| Field | Required | Notes |
|-------|----------|-------|
| `text` | yes | Use `?` per illegible character (§12). |
| `fontSize` | no* | Relative size; largest text on the sign = 150, others scaled to it. Not a point or mm size. |
| `km` | no | Integer distance, for `orientação` lines. |
| `elements` | no | Per-line arrow/pictogram/shield. |
| `backColor`, `fontColor` | no | Overrides. |

\* `fontSize` SHOULD be present on visible text lines; it is optional structurally to allow empty/placeholder lines.

---

## 11. Inline badge segments

When **part** of a single line has its own background — e.g. a destination followed by a coloured route plate **on the same line** — that line is written as an **array of segments** instead of a single line object. Each segment is `{ "text": …, "fontSize": … }` with an optional `backColor` on the boxed segment. Segments render inline, left to right.

```json
[
  { "text": "P. Alegre", "fontSize": 150 },
  { "text": "BR-101", "fontSize": 150, "backColor": "blue" }
]
```

**Rule.** Use a segment array **only** when the colour changes *inline within one line*. A whole line with a single background stays a normal line object with a line-level `backColor`. A segment array MUST contain at least two segments.

This is the distinction between an *inline text badge* (boxed text — §11) and a *shield element* (a drawn plate — §9.3): the BR-101 above is boxed text on the line; a route shield is a graphic placed like a pictogram.

---

## 12. Text fidelity

A producer that *recognises* a sign (rather than authoring one) MUST report only what is visually confirmed:

- If a character is faded, damaged, cropped, or obscured, emit `?` for each unreadable character.
- A partially recognised place name MUST be emitted with `?` characters, **never** silently completed from outside knowledge.
- Observation, not interpretation: correction is a separate, downstream concern.

(For authored signs this section does not apply — the author supplies known-correct text.)

---

## 13. Versioning

This specification is versioned. Backward-compatible additions (new optional fields, new enum members, new pictogram terms) increment the minor version. Breaking changes (removed/renamed fields, changed required-ness, changed structure) increment the major version. Consumers SHOULD ignore unknown optional fields they do not understand only if a future minor version explicitly relaxes `additionalProperties`; in v1.0 instances are strict (`additionalProperties: false`) to catch typos early.

---

## 14. Conformance

- A **conforming instance** validates against `capsign.schema.json`.
- A **conforming producer** emits only conforming instances.
- A **conforming consumer** accepts every conforming instance, and MUST NOT reject an instance for using an optional field or vocabulary term it does not recognise (it MAY fall back to a placeholder).

---

## Appendix A — relationship to other standards

| Standard / system | Domain | Overlap with CapSign |
|-------------------|--------|----------------------|
| DNIT IPR-743 / CONTRAN manuals | Visual/fabrication rules for Brazilian signs | **Normative source** for the rules CapSign-described signs obey. CapSign is a digital representation; the manuals are the authority. |
| DATEX II (CEN/TS 16157-4) | Dynamic electronic VMS content exchange | Shared *element decomposition* (text lines in sequence, pictograms with positions, supplementary panels) and position vocabulary. Different domain (programmable VMS, not fabricated signs); not adopted. |
| GIS sign inventories (MUTCD/CONTRAN point layers) | *Where* signs are and their condition | Complementary. An interop layer MAY pair a CapSign description with a location record (lat/long + type code + dimensions) for inventory ingestion. |

No existing open standard describes the compositional layout of a static custom guide sign for redrawing. CapSign occupies that gap.
