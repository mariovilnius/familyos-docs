# Changelog

Visi reikšmingi dokumentacijos ir formatų pakeitimai.

Formatas remiasi [Keep a Changelog](https://keepachangelog.com/) principu.
Repozitorijaus versijos — [SemVer](https://semver.org/).

## [Nepaskelbta]

### Pridėta
- Licencija — CC BY 4.0 ([LICENSE](LICENSE)).

### Pakeista
- `spec/import-format.md` §9: **pilna importerio aprėptis** — visi 8 tipai
  įrašomi (patikrinta importuojant `examples/sample.familyos`: 8/8, 0 praleista,
  0 įspėjimų). Formato versija nesikeičia (1.0).
- (anksčiau) importerio aprėptis (fazė 1) — formatas galioja
  visiems 8 tipams; įrašomi `member`/`chore`/`shopping_list`/`shopping_item`,
  likę priimami+validūs, bet dar neįrašomi (skip+warn). Formato versija
  nesikeičia (1.0). Įrodyta e2e testais.

## [1.0.0] — 2026-07-17

### Pridėta
- `.familyos` importo formatas, versija **1.0** — [`spec/import-format.md`](spec/import-format.md).
  - Envelope: `familyos`, `kind`, `generatedAt`, `source`, `household`, `records[]`.
  - 8 įrašų tipai: `member`, `chore`, `shopping_list`, `shopping_item`, `event`,
    `account`, `transaction`, `purchase`.
  - Idempotentiškumas per `externalId`; vidinės nuorodos per `ref`.
  - Todoist CSV → `chore` atitikmenų lentelė (+ pasikartojimo frazės → RRULE).
- Pavyzdys [`examples/sample.familyos`](examples/sample.familyos) (12 įrašų, visi tipai).
