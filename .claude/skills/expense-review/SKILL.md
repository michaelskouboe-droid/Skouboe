---
name: expense-review
description: Biweekly review of personal spending and fixed costs (cashflow overview, trend/deviation detection, comparison to a Danish 2-child family benchmark). Use when the user asks for their expense review, spending check, or when triggered on the recurring Monday schedule for "forbrugsgennemgang".
---

# Expense Review (Forbrugsgennemgang)

Du udfører en periodisk gennemgang af brugerens forbrug. Brugeren er en privatperson
i Danmark med en familie (2 børn), og vil have et løbende overblik over cashflow,
udvikling i omkostninger, afvigelser, og forslag til besparelser sammenlignet med
en gennemsnitlig dansk familie med 2 børn.

## 1. Find datakilden

Transaktionsdata ligger som CSV-filer i `data/transactions/`. Hver fil dækker typisk
en måned eller en periode og er eksporteret fra netbank. Forventede kolonner (navne
kan variere lidt mellem banker — match efter bedste evne):

- Dato (dato for transaktionen)
- Tekst / beskrivelse
- Beløb (negativ = udgift, positiv = indtægt)
- evt. Saldo

Hvis `data/transactions/` er tom eller der ikke er nye filer siden sidste gennemgang
(se `reports/` for seneste rapport-dato), så sig det klart til brugeren i rapporten
i stedet for at gætte på tal.

## 2. Kategoriser forbrug

Del transaktioner op i:

- **Fast forbrug / faste omkostninger**: husleje/realkredit, el/vand/varme, forsikringer,
  abonnementer, internet/mobil, daginstitution/SFO, lån/afdrag.
- **Privat forbrug / variabelt**: dagligvarer, restauranter/take-away, tøj, fritid,
  transport (brændstof/parkering/offentlig transport), gaver, øvrigt.

Brug transaktionsteksten til at gætte kategori. Hvis der findes en tidligere kategori-
mapping i `data/category-rules.csv` (opret den hvis den ikke findes, med kolonner
`tekst_mønster,kategori`), så brug og udbygge den, så kategoriseringen bliver mere
præcis over tid.

## 3. Beregn cashflow og udvikling

- Sum af indtægter, faste omkostninger, variabelt forbrug, og nettoresultat for perioden.
- Sammenlign med foregående perioder (læs tidligere rapporter i `reports/`) for at vise
  udvikling over tid pr. kategori.
- Fremhæv afvigelser: kategorier der stiger/falder >15% fra deres historiske gennemsnit,
  eller enkeltstående usædvanligt store transaktioner.

## 4. Benchmark mod en dansk familie med 2 børn

Slå op i offentlige danske kilder for et referencebudget for en familie med 2 børn,
f.eks. Danmarks Statistik (forbrugsundersøgelsen / FU-tabeller) eller Forbrugerrådet
Tænk Penge's standardbudget. Brug WebFetch/WebSearch til at hente aktuelle tal —
gæt ikke tal fra hukommelsen, da de ændrer sig år for år. Notér i rapporten hvilken
kilde og hvilket år tallene stammer fra.

Sammenlign brugerens kategorier (husleje, mad, transport, fritid osv.) med
referencebudgettet og fremhæv hvor de ligger markant over eller under.

## 5. Optimeringsforslag

Giv 3-5 konkrete, prioriterede forslag til at reducere omkostninger, baseret på:
- Kategorier der ligger over benchmark
- Stigende trends
- Abonnementer/faste omkostninger der ikke er blevet brugt for nylig (hvis det kan ses)

## 6. Byg rapporten som visuel HTML — ikke som tekstblokke

Rapporten skal kunne overskues på under 30 sekunder, med mulighed for at folde
detaljer ud. Brug HTML + inline CSS (ingen eksterne billeder/chart-APIs, da data
er finansielle og private — byg visualiseringer som rene `<div>`-bjælker/farver).

Gem den fulde HTML i `reports/YYYY-MM-DD-forbrugsgennemgang.html` og brug samme
indhold som `htmlBody` i Gmail-udkastet. Følg denne struktur:

**A. Overblik (altid synligt, ingen scrolling for at forstå helheden)**
- 4 KPI-"kort" side om side (brug en HTML-tabel eller flex-div med `display:inline-block`,
  da Gmail har begrænset CSS-støtte): Indtægt, Faste omkostninger, Variabelt forbrug, Netto.
  Hvert kort: stort tal + lille delta vs. forrige periode (grøn ▲/rød ▼ med procent).
- Et lille badge/ikon-linje med op til 3 vigtigste afvigelser ("⚠️ Transport +28%
  vs. sidste periode"), kun de vigtigste — ikke en fuld liste.

**B. Visuel kategori-fordeling**
- For hver kategori: en bjælke bygget af en `<div>` med `background` og `width:XX%`
  der viser andel af totalt forbrug, plus beløb. Farvekod efter benchmark-status:
  grøn = under/på niveau med benchmark, gul = let over, rød = markant over (>20%).
- Brug samme bjælke-stil til at vise trend over de sidste 3-4 perioder per kategori
  (en række af korte bjælker = simpel "sparkline" i CSS).

**C. Drill-down detaljer (skal være foldet sammen som udgangspunkt)**
Brug `<details><summary>...</summary>...</details>` for hver sektion herunder, så
e-mailen er kort ved første åbning, men detaljerne er ét klik væk:
- "Se alle transaktioner pr. kategori"
- "Se fuld benchmark-sammenligning med kilde"
- "Se beregningsgrundlag for optimeringsforslag"
(Gmail folder `<details>` sammen visuelt i de fleste klienter, men hvis det ikke
understøttes, er det acceptabelt at det falder tilbage til synligt — vigtigst er
at overblikket (A+B) står først og er kort.)

**D. Optimeringsforslag**
- Vis som en kort, nummereret liste med fed overskrift per forslag og én linje
  begrundelse — ikke lange afsnit. Detaljeret begrundelse hører under drill-down (C).

Skriv så lidt løbende tekst som muligt — foretræk tal, bjælker, farver og korte
labels over sætninger. Brug dansk sprog og danske tal-/valutaformater (kr., 1.234,56).

## 7. Send til brugeren

Opret et Gmail-udkast (`mcp__Gmail__create_draft`) til michaelskouboe@gmail.com med
emnet "Forbrugsgennemgang <dato>" og den visuelle HTML-rapport fra trin 6 som
`htmlBody`. Gmail-integrationen kan kun oprette udkast, ikke sende automatisk —
informer brugeren om, at udkastet ligger klar til afsendelse.

## Noter

- Hvis dette køres som en planlagt/automatisk trigger uden interaktiv bruger til stede,
  skal du stadig gennemføre alle trin og oprette udkastet — spørg ikke brugeren om
  bekræftelse undervejs.
- Commit nye/opdaterede filer i `reports/` og `data/category-rules.csv` til git, så
  historikken er bevaret til næste gennemgang.
