# `.familyos` importo failo formatas — v1.0

Portable formatas šeimos duomenims įkelti į familyOS: darbai, pirkinių sąrašai,
kalendorius, finansai, pirkiniai, nariai. Vienas failas, viena versija, žmogaus
skaitomas.

> **Formato versija:** 1.0 · **Būsena:** beta
> Formalią schemą apibrėžia familyOS Zod kontraktas (`familyosImportSchema`);
> šis dokumentas — normatyvinis jo aprašymas + migracijų gidas.

---

## 1. Kodėl toks dizainas

Formatas sąmoningai artimas užduočių tvarkyklių eksportui (**Todoist**, Trello):
plonas *envelope* + plokščias `records[]` sąrašas, kur kiekvieną įrašą skiria
`type` laukas. Privalumai:

- **Lengva rašyti ranka** ir žiūrėti `git diff`.
- **Plečiama be lūžių** — pridėjus naują `type`, seni importeriai tiesiog
  ignoruoja tai, ko nesupranta (forward-compatible).
- **Idempotentiška** — `externalId` leidžia perimportuoti tą patį failą be
  dublikatų.
- **Vienas failas keliems moduliams** — ne po CSV kiekvienam.

Failo plėtinys: **`.familyos`** (turinys — JSON; `.json` irgi priimamas).
MIME: `application/json`.

---

## 2. Envelope

```jsonc
{
  "familyos": "1.0",              // formato versija (privaloma)
  "kind": "familyos.import",      // atpažinimo diskriminatorius (privaloma)
  "generatedAt": "2026-07-17T10:00:00Z",   // nebūtina
  "source": "todoist",           // kilmė: todoist | familyos-export | manual | laisvas tekstas
  "household": {                  // nebūtina užuomina; importas VISADA eina į dabartinį namų ūkį
    "name": "Kazlauskai",
    "baseCurrency": "EUR",
    "timezone": "Europe/Vilnius"
  },
  "records": [ /* ... įrašai ... */ ]   // 1..10000
}
```

- `familyos` — importeris priima savo **major** versiją (`1.x`). Nesutampant
  major — atmeta su aiškia klaida.
- `household` yra tik informacinis. Tikslinis namų ūkis = prisijungusio
  vartotojo. Importas niekada nekuria svetimo namų ūkio.

---

## 3. Įrašų tipai (`records[]`)

Kiekvienas įrašas turi `type`. Bendri (nebūtini) laukai visiems:

| Laukas | Paskirtis |
|---|---|
| `externalId` | Stabilus ID iš šaltinio. Perimportuojant upsert pagal (namų ūkis, type, externalId). |
| `ref` | Vietinis (tik šio failo) žymuo, kad kiti įrašai galėtų nurodyti šį (pvz. prekė → sąrašas). |

### 3.1 `member`
```jsonc
{ "type": "member", "nickname": "Jonas", "role": "CHILD" }
```
`role`: `ADULT | TEEN | CHILD | GUEST` (OWNER nekuriamas per importą). Kiti
įrašai nurodo narį per `nickname`.

### 3.2 `chore` (užduotis — Todoist stilius)
```jsonc
{
  "type": "chore",
  "content": "Išnešti šiukšles",      // Todoist: CONTENT
  "note": "Antradieniais ir penktadieniais",  // Todoist: DESCRIPTION
  "assignees": ["Jonas"],             // narių nickname; Todoist: RESPONSIBLE
  "due": "2026-07-18",                // Todoist: DATE
  "dueTime": "18:00",
  "rrule": "FREQ=WEEKLY;BYDAY=TU,FR", // RFC 5545 arba Todoist frazė (žr. §5)
  "priority": 1,                       // 1 = aukščiausias (Todoist p1)
  "points": 10,                        // familyOS: motyvacijos taškai
  "coins": 5,                          // familyOS: monetos
  "externalId": "todoist:6f21a"
}
```

### 3.3 `shopping_list` + `shopping_item`
```jsonc
{ "type": "shopping_list", "name": "Maxima", "ref": "maxima" }
{ "type": "shopping_item", "list": "maxima", "name": "Pienas", "quantity": "2", "unit": "l" }
```
`shopping_item.list` = sąrašo `ref` **arba** pavadinimas. Praleidus — dedama į
numatytąjį sąrašą.

### 3.4 `event` (kalendorius)
```jsonc
{
  "type": "event",
  "title": "Odontologas",
  "start": "2026-07-20T09:30:00+03:00",
  "end": "2026-07-20T10:15:00+03:00",
  "location": "Klinika",
  "attendees": ["Jonas", "Rūta"]
}
```
Visai dienai: `"allDay": true` ir `start` kaip data 00:00.

### 3.5 `account` + `transaction` (finansai)
```jsonc
{ "type": "account", "name": "Swedbank", "ref": "swed", "currency": "EUR", "isShared": true }
{
  "type": "transaction",
  "account": "swed",          // sąskaitos ref arba pavadinimas
  "amount": "-12.50",         // string, ženklas: minusas = išlaida
  "category": "Maistas",      // kategorija (sukuriama, jei nauja)
  "merchant": "Maxima",
  "date": "2026-07-15",
  "externalId": "bank:tx:99812"
}
```
**Pinigai — visada string** (`"-12.50"`), niekada float (kad nebūtų
slankiojo kablelio paklaidų; serverio pusėje — Decimal).

### 3.6 `purchase` (pirkiniai/garantijos)
```jsonc
{
  "type": "purchase",
  "item": "Dulkių siurblys",
  "amount": "189.00",
  "date": "2026-06-01",
  "warrantyUntil": "2028-06-01"
}
```

---

## 4. Nuorodų sprendimas (references)

Importeris įrašus tvarko **stabilia tvarka** ir jungia per vardus/ref:

1. `member` pirmiausia → `nickname` → `memberId` žemėlapis.
2. `shopping_list`, `account` → `ref`/pavadinimas → ID žemėlapiai.
3. Tada `chore.assignees`, `event.attendees` → memberId; `shopping_item.list`,
   `transaction.account` → ID.

Neišsprendus nuorodos (pvz. `assignees: ["Petras"]`, o tokio nario nėra):
įrašas praleidžiamas ir įrašomas į `warnings[]` — visas importas **nenutrūksta**.

---

## 5. Migracija iš Todoist

Todoist projektą eksportuok: **Project → ⋯ → Export → Export as CSV/Template**.
Stulpelių atitikmenys į `chore`:

| Todoist CSV | `.familyos` `chore` |
|---|---|
| `CONTENT` | `content` |
| `DESCRIPTION` | `note` |
| `PRIORITY` (1–4) | `priority` |
| `RESPONSIBLE` | `assignees[]` |
| `DATE` | `due` |
| pasikartojimo frazė (`every day`, `every mon,fri`) | `rrule` |
| `INDENT` (sub-tasks) | suplokštinama (be hierarchijos v1) |

Pasikartojimo frazės → RRULE (importeris supranta dažniausias):
`every day` → `FREQ=DAILY`; `every week` → `FREQ=WEEKLY`;
`every mon,fri` → `FREQ=WEEKLY;BYDAY=MO,FR`; `every month` → `FREQ=MONTHLY`.
Sudėtingesnes rašyk tiesiai RFC 5545 (`FREQ=...`).

> CSV → `.familyos` konvertavimas daromas kliento pusėje arba parankiniu
> skriptu; API priima tik `.familyos` (JSON) bundle.

---

## 6. Idempotentiškumas ir kartojimas

- Įrašas su `externalId` → **upsert** pagal (namų ūkis, type, externalId).
  Perimportavus tą patį failą — atnaujina, ne dubliuoja.
- Be `externalId` → visada **insert** (kartojant atsiras dublikatai).
- Rezultatas grąžinamas:
  ```jsonc
  { "created": { "chore": 12, "event": 3 }, "updated": { "chore": 2 },
    "skipped": 1, "warnings": ["Nerastas narys „Petras" (chore „Šuo")"] }
  ```

---

## 7. Validacija ir klaidos

- Failas validuojamas prieš bet kokį rašymą (Zod schema).
- Struktūros klaida (blogas JSON, trūksta `kind`, blogas `type`) → `422` su
  lauko keliu; **niekas neįrašoma** (viskas-arba-nieko validacijos lygyje).
- Nuorodų klaidos (§4) → įrašas praleidžiamas + `warnings[]`, importas tęsiasi.
- Limitas: 10 000 įrašų viename faile.

---

## 8. Versijos

- `familyos: "1.0"` — pradinė. `records[]` diskriminuota sąjunga.
- Suderinamumo taisyklė: **naujas `type` = minor** (seni importeriai praleidžia
  nežinomą). **Esamo `type` laukų šalinimas/reikšmės keitimas = major.**

Pakeitimų istorija — [../CHANGELOG.md](../CHANGELOG.md).

---

## 9. Importerio aprėptis (fazė 1)

Formatas (visi 8 tipai) galioja ir validuojamas. familyOS importeris
(`POST /import`) šiuo metu **įrašo**: `member`, `chore`, `shopping_list`,
`shopping_item` (su nuorodų sprendimu ir natūralaus rakto idempotentiškumu).
`event`, `account`, `transaction`, `purchase` — **priimami ir validūs**, bet dar
neįrašomi: grąžinami `warnings[]` ir suskaičiuojami `skipped`. Aprėptis
plečiama nekeičiant formato.

Įrodyta automatiniais testais: importas → įrašymas, skip+warn, idempotentiškas
perimportavimas be dublikatų, versijos ir rolės apsaugos.

## 10. Minimalus pavyzdys

Žr. [`../examples/sample.familyos`](../examples/sample.familyos) — 12 įrašų,
visi 8 tipai.
