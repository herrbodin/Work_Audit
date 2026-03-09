# RevisionsPro – CLAUDE.md

Detta är instruktionsfilen för Claude Code. Läs den innan du rör något i projektet.

---

## Vad är RevisionsPro?

En webbapp för svenska revisorer (auktoriserade revisorer och revisorsassistenter) som hjälper till att skapa och hantera **revisionsberättelser** enligt **ISA for Less Complex Entities (ISA for LCE)** – standarden som gäller för räkenskapsår från och med 15 december 2025.

Målgrupp: Auktoriserade revisorer och revisorsassistenter i Sverige.
Regulatoriska parter: Revisorsinspektionen (RI), FAR, Bolagsverket.

---

## Arkitektur

**En enda fil: `index.html`**

- Ren HTML + vanilla JavaScript
- Ingen bundler, inga npm-paket, inga build-steg
- Data lagras i `localStorage` (nycklarna är `rev_clients`, `rev_templates`, `rev_rb_templates`)
- Deploy: GitHub Pages (`herrbodin.github.io/Work_Audit`)
- Externt beroende: `html2pdf.js` från cdnjs (för PDF-export)

Rör inte arkitekturen utan explicit instruktion. Lägg inte till React, Vite, npm eller liknande.

---

## Datamodell

### Client (`rev_clients`)
```json
{
  "id": "uid",
  "name": "AB Exempelbolaget",
  "orgnr": "556XXX-XXXX",
  "contact": "Namn",
  "auditor": "Förnamn Efternamn",
  "auditortype": "Auktoriserad revisor",
  "auditorEmail": "revisor@byrå.se",
  "firm": "Byråns namn",
  "regnr": "FAR XXXXX",
  "status": "ej påbörjad | klar",
  "notes": [{ "id": "uid", "date": "YYYY-MM-DD", "text": "..." }],
  "years": [
    {
      "id": "uid",
      "from": "YYYY-MM-DD",
      "to": "YYYY-MM-DD",
      "baldate": "31 december 2024",
      "signdate": "YYYY-MM-DD",
      "city": "Stockholm",
      "pages": "1–XX",
      "comments": "",
      "ansvarsfrihet": "tillstyrker | avstyrker",
      "periodText": "",
      "rbTemplateId": "uid | ''",
      "status": "ej påbörjad | klar"
    }
  ]
}
```

### Auditor Template (`rev_templates`)
```json
{
  "id": "uid",
  "label": "Min standardmall",
  "auditor": "...",
  "auditortype": "Auktoriserad revisor",
  "email": "...",
  "firm": "...",
  "regnr": "...",
  "city": "Stockholm"
}
```

### RB-mall (`rev_rb_templates`)
```json
{
  "id": "uid",
  "name": "Standard AB",
  "desc": "...",
  "sections": {
    "intro": "...",
    "uttalanden": "...",
    "grund": "...",
    "styransvar": "...",
    "revisoransvar": "...",
    "rapport": "..."
  }
}
```

---

## Domänterminologi (använd alltid på svenska)

| Term | Förklaring |
|------|-----------|
| Revisionsberättelse (RB) | Audit report – det centrala dokumentet |
| Räkenskapsår | Financial year |
| Balansdatum | Balance sheet date |
| Ansvarsfrihet | Discharge of liability (tillstyrker / avstyrker) |
| Auktoriserad revisor | Authorized auditor |
| Revisorsassistent | Audit assistant |
| ROMM | Risk of Material Misstatement |
| Väsentlighet | Materiality |
| ISA for LCE | ISA for Less Complex Entities |
| iXBRL | Inline XBRL – format för digital inlämning till Bolagsverket |
| se-nak-gar | Bolagsverkets XBRL-taxonomi |

---

## iXBRL / Bolagsverket

Appen exporterar revisionsberättelsen som iXBRL (`.xhtml`) med Bolagsverkets `se-nak-gar`-taxonomi.

Viktiga XBRL-element som används:
- `se-nak-gar:EntityName`
- `se-nak-gar:EntityRegistrationNumber`
- `se-nak-gar:AuditedPeriodDescription`
- `se-nak-gar:BalanceSheetDate`
- `se-nak-gar:AuditReportDate`
- `se-nak-gar:AuditorName`
- `se-nak-gar:AuditorQualification`
- `se-nak-gar:AuditFirmName`
- `se-nak-gar:AuditFirmRegistrationNumber`
- `se-nak-gar:AuditorSigningCity`
- `se-nak-gar:AuditorDischargeOpinion`

Schema-URL: `https://www.bolagsverket.se/se/taxonomy/2021-09-30/se-nak-gar.xsd`

---

## Vyer i appen

| Vy | ID | Beskrivning |
|----|----|-------------|
| Klientlista | `view-clients` | Översikt med statistik, sök, statusbytes |
| Revisionsberättelse | `view-report` | Dokumentredigering, export PDF/Word/iXBRL |
| Revisorsmallar | `view-templates` | Sparade revisoruppgifter för snabbifyllning |
| RB-mallar | `view-rb-templates` | Juridiska textmallar med platshållare |

---

## Platshållare i RB-mallar

Dessa ersätts automatiskt när en mall kopplas till ett räkenskapsår:

| Platshållare | Ersätts med |
|---|---|
| `{{BOLAG}}` | Bolagets namn |
| `{{PERIOD}}` | Räkenskapsårets period (text) |
| `{{BALDAT}}` | Balansdatum |
| `{{SIDOR}}` | Sidor i årsredovisningen |
| `{{REVISOR}}` | Revisorns namn |
| `{{BYRA}}` | Revisionsbyrå |

---

## Regler för ändringar

1. **Språk:** All UI-text ska vara på svenska. Inga engelska labels, knappar eller felmeddelanden.
2. **Datastruktur:** Ändra inte localStorage-nycklarna utan att migrera befintlig data.
3. **Enkelfil:** Allt CSS och JS ska vara inbakat i `index.html`. Skapa inte separata filer.
4. **Inga breaking changes:** Lägg till funktionalitet, ta inte bort befintlig utan explicit instruktion.
5. **ISA for LCE:** Terminologi och struktur ska följa standarden.

---

## Vanliga uppgifter

**Lägga till ett nytt fält i revisionsberättelsen:**
1. Lägg till fältet i `year`-objektet i `saveClient()` / `confirmAddYear()`
2. Rendera det i `buildReport()`
3. Lägg till `syncYear('fältnamn', this.value)` på input-elementet
4. Inkludera det i `getReportData()` och exportfunktionerna

**Lägga till en ny vy:**
1. Skapa `<div id="view-ny">` i HTML
2. Lägg till en `<button class="nav-btn">` i sidebar
3. Registrera vyn i `showView()`-funktionen

**Debugga localStorage:**
```javascript
// I webbläsarens konsol:
JSON.parse(localStorage.getItem('rev_clients'))
```
