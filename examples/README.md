# Examples

Each file is a conforming CapSign instance, drawn from a real sign. All validate against `../capsign.schema.json`.

| File | Sign | Demonstrates |
|------|------|--------------|
| `01-orientacao-km-shield.json` | Curitiba / Ponta Grossa with distances | `orientação` type; km column on each line. |
| `02-inline-badge-segments.json` | Florianópolis / São Paulo / P. Alegre + BR-101 plate | Up-arrow module; a line as a **segment array** with an inline blue route badge. |
| `03-shield-as-element.json` | Paulo Frontin / Rebouças + PR-476 | Two modules; a **shield element** and a per-line arrow on one line; blue module override. |
| `04-multi-module-tourism.json` | CEMITÉRIO + Turismo Rural / Colônia… | `turístico` sign; blue header module over a brown body; per-line **pictograms** positioned `before`; module-level arrows. |
| `05-pictograms-row.json` | Fuel / restaurant / tyre / lodging, "A 300 m" | `pictograms` type; ordered ids + footer. |
| `06-coded-warning.json` | A-18 speed-bump diamond + down-left arrow | `advertência` sign; a module with **no text lines**, a CONTRAN-coded pictogram and an arrow. |
