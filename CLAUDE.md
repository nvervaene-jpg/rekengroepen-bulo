# Rekengroepen — BuLO Sint-Franciscus

## Wat is dit project?

Een webtool waarmee leerkrachten van BuLO Sint-Franciscus leerlingen indelen in homogene rekengroepen op basis van wiskundescores over meerdere domeinen. Gehost via GitHub Pages.

- **Live URL**: https://nvervaene-jpg.github.io/rekengroepen-bulo/
- **Wachtwoord**: MeesterNils
- **Eigenaar**: Nils (ICT-leerkracht en coach)

## Architectuur

Eén HTML-bestand (`index.html`) met inline CSS en JS. Geen framework, geen build-stap. Data wordt opgehaald via `fetch('data.json')` bij het laden.

### Bestanden

- `index.html` — De volledige tool (HTML + CSS + JS, ~52KB)
- `data.json` — Alle leerlingdata en doelbeschrijvingen (~61KB)

## data.json structuur

```json
{
  "db": {
    "2_GK": ["doelbeschrijving 1", "doelbeschrijving 2", ...],
    "2_HR": [...],
    ...
  },
  "ll": [
    {
      "i": "Naam||Klas",       // unieke ID
      "n": "Naam",             // leerlingnaam
      "k": "Klas",             // leerkracht/klasnaam
      "l": 2,                  // leerjaar
      "d": {                   // domeinen
        "GK": {
          "p": 83,             // totaalpercentage
          "k": "2_GK",         // key naar db-doelen
          "ps": [100, 80, ...],// per-doel percentages (matcht db-array)
          "lj": 2              // leerjaar voor dit domein
        },
        "HR": { ... },
        "MK": { ... },
        "MT": { ... }
      }
    }
  ]
}
```

### Domeinen

| Code | Naam | Vanaf LJ |
|------|------|----------|
| GK | Getallenkennis | 1 |
| HR | Hoofdrekenen | 1 |
| MK | Meetkunde | 1 |
| MT | Meten | 1 |
| CY | Cijferen | 3 |

### db-keys

Format: `"{leerjaar}_{domein}"` (bijv. `"2_GK"`, `"3_CY"`).

## Opslaan/Laden (JSON-bestand)

De tool slaat een JSON-bestand op met:

```json
{
  "v": "3",
  "d": "2026-09-02T...",
  "data": [
    { "i": "Naam||Klas", "b": "doorstroom", "dl": 3, "f": false, "nt": "notitie" }
  ],
  "verwijderd": ["Naam||Klas", ...],
  "defGroepen": [
    { "id": "...", "naam": "Groepsnaam", "kleur": "#...", "leerlingen": ["id1", "id2"] }
  ]
}
```

Bij laden worden beslissingen (`b`, `dl`, `f`, `nt`) teruggekoppeld aan de leerlingen uit `data.json`, verwijderde leerlingen gefilterd, definitieve groepen hersteld, en het groepsvoorstel automatisch gegenereerd.

## Excel → data.json pipeline

Leerkrachten leveren Excel-bestanden aan per domein: `{Domein}_{Leerjaar}_{Leerkracht}.xlsx`

### Excel-structuur

- **Sheet "Overzicht ..."**: rij 0 bevat leerlingnamen vanaf kolom 4
- **Sheets "LL1", "LL2", ...**: individuele leerlingdata
- Per sheet: kolom 2 = doelbeschrijving, kolom 3 = max score, kolom 4 = behaalde score
- Laatste rijen: totaal, maximum, en percentage

### Verwerkingsstappen

1. Parse met Python + `openpyxl` (read_only=True, data_only=True)
2. Lees per student-sheet de doelen, scores en percentage
3. Bouw `ps`-array (per-doel percentages) die matcht met de `db`-doelen (match op eerste 30 tekens)
4. Filter bestaande leerlingen van dezelfde leerkracht uit: `[ll for ll in data['ll'] if ll.get('k') != KLAS]`
5. Voeg nieuwe entries toe
6. Sla op als `data.json`

### Bekende dataproblemen

- Sommige leerlingen hebben alle nullen (waarschijnlijk niet getest)
- `#REF!`-fouten in Excel → worden als 0 behandeld
- Naaminconsisenties tussen bestanden (hoofdletters, spaties, spellingvarianten)

## De 4 tabs

1. **① Leerlingoverzicht** — Per leerling: scorebalkjes, advies, beslissing (doorstroom/herhaling), leerjaar-keuze, functioneel-toggle, notitie, verwijderknop
2. **② Groepsvoorstel** — Auto-gegenereerd op basis van beslissingen: groepeert op doelleerjaar en gedeelde zwakke domeinen
3. **③ Automatisch voorstel** — Groepeert puur op scores met instelbare drempel (65-80%)
4. **④ Definitieve groepen** — Drag-and-drop groepen, importeerbaar vanuit ② of ③, vrij aanpasbaar

## Belangrijke functies in de code

| Functie | Doel |
|---------|------|
| `mkKrt(ll)` | Bouwt een leerlingkaart voor tab ① |
| `renderLL()` | Rendert alle kaarten met filters |
| `genGr()` | Genereert groepsvoorstel (tab ②) |
| `genAu()` | Genereert automatisch voorstel (tab ③) |
| `renderDefGroepen()` | Rendert definitieve groepen (tab ④) |
| `expJS()` | Exporteert naar JSON-bestand |
| `impJS(e)` | Importeert JSON-bestand |
| `verwijderLLbyKaart(btn)` | Verwijdert een leerling |
| `calcAdv(ll, drempel)` | Berekent doorstroom/herhalingsadvies |
| `togDet(btn)` | Lazy-load doeldetails per leerling |

## Huidige staat (september 2026)

- **94 leerlingen** over 17 klassen/leerkrachten
- Leerjaren 1 t/m 4 + 3 manuele LJ5-leerlingen
- Alle Anneleen LJ2-data is volledig (7 leerlingen, 4 domeinen)
- Invoerfunctie voor nieuwe leerlingen is nog niet gebouwd (toekomstige feature)

## Conventies

- Geen frameworks, geen build tools — alles is vanilla HTML/CSS/JS
- CSS-variabelen voor theming (light/dark mode via `prefers-color-scheme`)
- Compacte variabelenamen in JS (`ll` = leerling, `d` = domeinen, `b` = beslissing, etc.)
- Deploy = push naar `main` branch → GitHub Pages serveert automatisch
