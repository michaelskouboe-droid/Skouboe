---
name: portfolio-review
description: Biweekly review of the user's full stock portfolio across multiple trading platforms (Nordnet, Saxo Bank, etc.) — consolidated overview, allocation, concentration/FX risk, trend over time, and rebalancing recommendations. Use when the user asks for their portfolio review, depot-status, aktieoverblik, or on the recurring schedule for "porteføljegennemgang".
---

# Portfolio Review (Porteføljegennemgang)

Du udfører en periodisk gennemgang af brugerens samlede aktieportefølje, som er
spredt over flere handelsplatforme/depoter (i dag: Nordnet med flere depoter, og
Saxo Bank). Formålet er et samlet overblik, udvikling over tid, og konkrete
anbefalinger — ikke bare et dump af tal.

## 1. Find datakilden

Nye eksports ligger i Google Drive (brugeren eksporterer CSV fra hver platforms
depotoversigt/beholdning). Søg efter dem med `mcp__Google_Drive__search_files`,
f.eks. `title contains 'aktie' or title contains 'portfolio' or title contains
'depot' or title contains 'beholdning'`, og afgræns til filer nyere end seneste
snapshot i `data/portfolio/snapshots/`. Hent indholdet med
`mcp__Google_Drive__download_file_content` (kommer som base64 — afkod).

**Kendte formater (match efter bedste evne, nye varianter kan dukke op):**

- **Nordnet "Aktietabel"-eksport** (tabulator-separeret, UTF-16-kodet): kolonner
  `Navn, Valuta, Antal, GAK, I dag %, Seneste kurs, Belåningsværdi DKK, Værdi,
  Værdi DKK, Ureal.afkast %, Afkast DKK`.
- **Nordnet "Beholdning"-eksport** (semikolon-separeret, UTF-8 med BOM): kolonner
  `Dato;Papirtype;Papirnavn;ISIN;Kurs;Valuta;Depotnavn;Depotnummer;Mængde;
  Kursværdi DKK;Anskaffelsesværdi;Urealiseret gevinst/tab;Anskaffelsesværdi DKK
  (Gennemsnit);Valutakurs;Anskaffelsesvaluta;Forventet udtrækning`. Denne har
  kostbasis og er den vigtigste kilde til regnskabsmæssigt afkast.
- **Saxo Bank "Portfolio"-eksport** (komma-separeret): kolonner `Produkt,
  Fondskode/ISIN, Antal, Lukkekurs, Lokal værdi,, Værdi i DKK`. Ingen kostbasis —
  kun øjebliksværdi. Indeholder typisk en cash-linje ("CASH & CASH FUND...").

Alle tal bruger dansk decimalkomma og kan være tusind-separeret med punktum —
konverter til float (fjern punktum, erstat komma med punktum) før beregning.

Hvis der ikke findes nye filer siden sidste gennemgang, sig det klart i
rapporten i stedet for at generere en rapport på gamle data uden at nævne det.

## 2. Normaliser til ét samlet datasæt

Tilføj hver ny snapshot til `data/portfolio/holdings-history.csv` (opret hvis den
ikke findes) med kolonnerne:

```
dato,platform,depotnavn,depotnummer,navn,isin,valuta,antal,kurs,vaerdi_dkk,anskaffelse_dkk,ureal_dkk
```

- `platform`: "Nordnet" eller "Saxo Bank" (udled fra filnavn/kolonner).
- `depotnavn`/`depotnummer`: fra Nordnet-filerne direkte; for Saxo og
  Nordnet-aktietabel-filen uden depotnummer, brug et stabilt navn som "Portfolio"
  hhv. "Konto <kontonummer fra filnavnet>".
- `anskaffelse_dkk`/`ureal_dkk`: kun udfyldt hvor kilden har kostbasis (Nordnet
  Beholdning). Lad stå tomt for Saxo og Nordnet-aktietabel.
- Gem også de rå kildefiler under `data/portfolio/snapshots/YYYY-MM-DD/` så
  historikken er revisionssikker.

Dette akkumulerede datasæt er grundlaget for trend-visning over tid — slet aldrig
gamle rækker, kun tilføj nye.

## 3. Beregn overblik

- **Samlet værdi** (sum `vaerdi_dkk`) og udvikling vs. forrige snapshot-dato i
  `holdings-history.csv` (beløb og %).
- **Fordeling pr. platform/depot** (DKK og % af total).
- **Fordeling pr. valuta** — sum `vaerdi_dkk` grupperet på `valuta`. Flag hvis en
  enkelt ikke-DKK-valuta udgør >40% af porteføljen (uafdækket FX-risiko).
- **Koncentrationsrisiko**: kombiner positioner i samme værdipapir tværs af
  depoter (match på `isin` hvor muligt, ellers `navn`). Beregn andel af total for
  hver position. Flag enkeltpositioner >10% og de samlede top 3/top 5 positioners
  andel af porteføljen.
- **Samlet urealiseret gevinst/tab** (kun for rækker med `anskaffelse_dkk` udfyldt)
  — vis beløb, % og nævn at det ikke dækker hele porteføljen, hvis Saxo/aktietabel-
  delen mangler kostbasis.

## 4. Trend over tid

Når der findes mere end én snapshot-dato i `holdings-history.csv`:
- Vis udvikling i total porteføljeværdi pr. snapshot (simpel CSS-bjælke-sparkline,
  som i forbrugsgennemgangens stil).
- Vis udvikling pr. platform/depot over de seneste 3-4 snapshots.
- Fremhæv positioner der har ændret sig markant i værdi eller andel siden sidste
  gennemgang (>15%).

Med kun én snapshot: sig det klart i rapporten ("dette er baseline — trend vises
fra næste gennemgang") i stedet for at opfinde en trend.

## 5. Anbefalinger / korrektioner

Giv 3-5 konkrete, prioriterede forslag baseret på faktiske observationer i data —
ikke generisk investeringsrådgivning. Typiske udløsere:
- Enkeltposition eller -sektor over en koncentrationsgrænse (>10% i én aktie,
  >25% i top 3).
- Skæv valutaeksponering (f.eks. >50% USD uden DKK/EUR-modvægt).
- Positioner med stort urealiseret tab, der er vokset over flere gennemgange
  (kan indikere behov for at tage stilling, ikke en automatisk "sælg"-anbefaling).
- Cash-positioner der står ubrugte over flere snapshots.

Vær tydelig: dette er observationer til brugerens egen beslutning, ikke
finansiel rådgivning.

## 6. Byg rapporten som visuel HTML

Følg samme princip som `expense-review`: overblik på under 30 sekunder, detaljer
i `<details>`. Gem som `reports/portfolio/YYYY-MM-DD-portefoljegennemgang.html`
og brug samme HTML som Gmail-udkastets `htmlBody`.

**A. Overblik**
- KPI-kort: Samlet værdi, Udvikling siden sidste gennemgang, Urealiseret
  gevinst/tab, Antal positioner/platforme.
- Op til 3 vigtigste flags (koncentration, FX, store bevægelser).

**B. Allokering**
- Bjælker for fordeling pr. platform/depot og pr. valuta.
- Top 10-positioner som bjælker med andel af total.

**C. Trend (når der er historik)**
- CSS-sparkline for total værdi over snapshots.

**D. Drill-down (`<details>`)**
- Fuld positionsliste pr. depot.
- Beregningsgrundlag for koncentration/FX-flags.

**E. Anbefalinger**
- Nummereret liste, fed overskrift + én linje begrundelse pr. punkt.

Brug dansk sprog og danske tal-/valutaformater (kr., 1.234,56). Ingen eksterne
billed-/chart-API'er — byg visualisering med inline-stylede `<div>`-bjælker.

## 7. Send til brugeren

Opret et Gmail-udkast (`mcp__Gmail__create_draft`) til michaelskouboe@gmail.com
med emnet "Porteføljegennemgang <dato>" og HTML-rapporten som `htmlBody`. Gmail-
integrationen kan kun oprette udkast — informer brugeren om at det ligger klar.

## Noter

- Hvis dette køres som en planlagt/automatisk trigger uden interaktiv bruger til
  stede, gennemfør alle trin og opret udkastet uden at spørge om bekræftelse.
- Commit nye/opdaterede filer i `data/portfolio/` og `reports/portfolio/` til git,
  så historikken bevares til næste gennemgang.
- Aktiekurser i CSV-eksportet er et øjebliksbillede ved eksport-tidspunktet — søg
  ikke efter "opdaterede" kurser via web, brug altid kildefilens tal.
