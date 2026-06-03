# CapSign Schema

**English** · [Português](README.pt-BR.md)

A machine-readable schema for describing the **layout of Brazilian road signs** (CONTRAN / DNIT IPR-743) — enough to redraw a sign's face, not merely to catalogue where it is installed.

CapSign describes the **composition** of a static, fabricated sign: its type, colour category, and the panels, text lines, arrows, pictograms, route shields and badges that make it up. It is an **interface contract** between anything that *recognises or authors* a sign and anything that *draws, validates, or stores* it.

It fills a gap no existing standard covers. Sign-inventory systems store signs as located points with condition attributes (*where* a sign is); DATEX II models dynamic electronic VMS content; the DNIT/CONTRAN manuals define the visual rules — but none describe the compositional layout of a custom guide sign in machine-readable form. CapSign does.

## Contents

| File | What it is |
|------|------------|
| [`capsign.schema.json`](capsign.schema.json) | The authoritative, machine-checkable definition (JSON Schema 2020-12). |
| [`SPECIFICATION.md`](SPECIFICATION.md) | The human-readable specification: intent, rules, and the judgment a producer applies. |
| [`examples/`](examples/) | Real signs expressed as conforming instances. |
| `LICENSE` | MIT (schema and code). The specification document is also offered under CC BY 4.0. |

## Quick look

A simple km-column guide sign:

```json
{
  "sign": {
    "type": "orientação",
    "colorScheme": "orientação",
    "confidence": 1.0,
    "modules": [
      { "lines": [
        { "text": "Curitiba", "km": 85, "fontSize": 150 },
        { "text": "Ponta Grossa", "km": 23, "fontSize": 150 }
      ] }
    ]
  }
}
```

A destination line ending in an inline blue route plate (segment array):

```json
[
  { "text": "P. Alegre", "fontSize": 150 },
  { "text": "BR-101", "fontSize": 150, "backColor": "blue" }
]
```

See [`examples/`](examples/) for six worked cases, including a multi-module tourist sign, a route-shield element, a pictogram row, and a coded warning sign.

## Core concepts (one paragraph)

A sign has a **type** (`modules`, `orientação`, `pictograms`, `tree`, `special`) and a **colorScheme** (CONTRAN colour category, read from the sign's *body*). A `modules`/`orientação` sign is a stack of **modules** (panels); a module holds **lines**; a line is either a text object or — when colour changes *within* the line — an array of **segments**. Lines and modules may carry **elements**: arrows, pictograms, or route shields, each with a `position`. A separate **module** exists only where a visible border divides panels; a coloured plate floating inside a panel is a line `backColor`, not a module. See the [specification](SPECIFICATION.md) for the full rules.

## Validate an instance

```bash
pip install jsonschema
```

```python
import json
from jsonschema import Draft202012Validator

schema = json.load(open("capsign.schema.json"))
instance = json.load(open("examples/01-orientacao-km-shield.json"))

errors = list(Draft202012Validator(schema).iter_errors(instance))
print("valid" if not errors else [e.message for e in errors])
```

## Recommended pictogram vocabulary

Pictogram `id` is either an official CONTRAN code (e.g. `A-18`, `R-19`) for standard warning/regulatory symbols, or a descriptive lowercase term for service/tourism symbols. The descriptive set is open; a consumer maps unknown terms to a placeholder. A recommended starter vocabulary:

`fuel`, `camping`, `airport`, `ferry`, `restroom`, `tyre`, `post-office`, `parking`, `heliport`, `hospital`, `information`, `port`, `restaurant`, `phone`, `mechanic`, `bus-stop`, `train-terminal`, `boat-terminal`, `toll`, `church`, `rural-tourism`, `yacht-club`, `waterfall`, `beach`, `museum`, `theater`, `monument`, `cycling`, `horse-riding`, `fishing`, `diving`, `surfing`, `viewpoint`, `cave`, `island`, `lighthouse`, `zoo`, `park`, `ruins`, `cultural-center`, `golf`, `soccer`, `handicraft`.

## Scope

**In scope:** the sign face — type, colour, panels, lines, text, arrows, pictograms, shields, badges, relative sizes.

**Out of scope:** installation location, condition, retroreflectivity, material (use an inventory system — CapSign pairs cleanly with one); rendering geometry, fonts, CAD detail (the consumer's concern); dynamic electronic VMS content (see DATEX II).

## Status & versioning

Version 1.0, stable. `tree` and `special` sign types are intentionally opaque in 1.0 (dimensions only). Backward-compatible additions bump the minor version; breaking changes bump the major version.

## License

Schema, examples, and any code: **MIT** (see `LICENSE`). The specification prose document is additionally available under **CC BY 4.0**. In short: use it, build on it, commercially or otherwise, with attribution.

## Contributing

Issues and proposals — new pictogram terms, additional sign types, clarifications — are welcome. Because instances are validated strictly, please include an example sign (and, where relevant, a photo or reference) with any proposed structural change.
