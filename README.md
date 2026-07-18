# familyOS — dokumentacija

Vieša familyOS formatų ir integracijų dokumentacija.

**familyOS** — šeimos operacinė sistema: finansai, namų darbai, kalendorius,
pirkiniai, sveikata ir kt. viena vieta visai šeimai.

---

## Turinys

| Dokumentas | Aprašymas | Versija |
|---|---|---|
| [`.familyos` importo formatas](spec/import-format.md) | Portable failo formatas šeimos duomenims įkelti (darbai, sąrašai, kalendorius, finansai…). Modeliuota pagal Todoist/Trello eksportą. | **1.0** |

Pavyzdys: [`examples/sample.familyos`](examples/sample.familyos) — pilnas, patvirtintas prieš schemą.

---

## Versijavimas

- Šis repozitorijus versijuojamas per **git tag'us** (`v<major>.<minor>.<patch>`).
- Kiekvienas formatas turi savo **formato versiją** (pvz. `.familyos` → `1.0`).
- Pakeitimai — [CHANGELOG.md](CHANGELOG.md).

Suderinamumo taisyklė kiekvienam formatui:
- **Naujas įrašo tipas / nebūtinas laukas = minor** (seni skaitytuvai praleidžia nežinomą).
- **Esamo lauko šalinimas ar reikšmės keitimas = major.**

---

## Statusas

Beta. Formatai gali pildytis; esami laukai keičiami tik keliant major versiją.

Klausimai / pasiūlymai — [Issues](../../issues).
