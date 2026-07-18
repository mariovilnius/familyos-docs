# Changelog

Visi reikšmingi dokumentacijos ir formatų pakeitimai.

Formatas remiasi [Keep a Changelog](https://keepachangelog.com/) principu.
Repozitorijaus versijos — [SemVer](https://semver.org/).

## [1.0.0] — 2026-07-17

### Pridėta
- `.familyos` importo formatas, versija **1.0** — [`spec/import-format.md`](spec/import-format.md).
  - Envelope: `familyos`, `kind`, `generatedAt`, `source`, `household`, `records[]`.
  - 8 įrašų tipai: `member`, `chore`, `shopping_list`, `shopping_item`, `event`,
    `account`, `transaction`, `purchase`.
  - Idempotentiškumas per `externalId`; vidinės nuorodos per `ref`.
  - Todoist CSV → `chore` atitikmenų lentelė (+ pasikartojimo frazės → RRULE).
- Pavyzdys [`examples/sample.familyos`](examples/sample.familyos) (12 įrašų, visi tipai).
