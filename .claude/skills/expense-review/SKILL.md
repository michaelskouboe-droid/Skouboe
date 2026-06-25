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

**"Øvrigt" skal holdes lav.** En stor uklassificeret post gør rapporten ubrugelig.
Før hver gennemgang: find de største enkeltposter der falder i "Øvrigt" og tilføj
regler for dem i `data/category-rules.csv` i stedet for at lade dem stå uklassificeret.
Mål: "Øvrigt" bør udgøre under ca. 20-25% af de samlede udgifter i en måned — hvis det
er højere, prioriter kategorisering af de største poster før resten af rapporten skrives.

**Eksklusion af intern omfordeling og værdipapirhandel (kategori `_internal_transfer`):**
Husholdningen har flere sammenkædede konti (løn-, budget-, opsparings- og diverse
formålskonti) samt et investeringsdepot. Følgende skal trækkes helt ud af cashflow-
beregningen, da det ikke er reelt forbrug eller indtægt:
- Overførsler mellem egne konti (Nemkonto, Lønkonto, Opsparing, Renoveringskonto,
  Madkonto, Budgetkonto, "Budget <navn>")
- Værdipapirhandel og depot-aktivitet: "Hdl.", "Udbytte", "Fonds", "Depot gebyr",
  "Aktiekøb", "Aktie opsparing", "Investering"
- Undtagelse: løntekster der indeholder "lønoverførsel" er ægte indtægt og skal ALDRIG
  ekskluderes, selv hvis de matcher et af mønstrene ovenfor.

**Kategori-sum bug at undgå:** Når du summerer beløb pr. kategori til søjlediagrammet,
sum kun udgiftssiden (negative beløb) pr. kategori. Bland ikke positive beløb (f.eks.
tilbagebetalinger, MobilePay-modtagelser, børnepenge) ind i samme sum som udgifter for
en kategori som "Øvrigt" — det kan give en useriøs/negativ sum. Uklassificerede
positive beløb skal i stedet vises i et separat "anden indtægt"-felt, ikke trækkes fra
udgiftskategorien.

## 3. Beregn cashflow og udvikling

- Sum af indtægter, faste omkostninger, variabelt forbrug, og nettoresultat for perioden.
- Sammenlign med foregående perioder (læs tidligere rapporter i `reports/`) for at vise
  udvikling over tid pr. kategori.
- Fremhæv afvigelser: kategorier der stiger/falder >15% fra deres historiske gennemsnit,
  eller enkeltstående usædvanligt store transaktioner.

**Historisk perspektiv (ikke kun måned-mod-måned):** Beregn for hver hovedkategori et
historisk gennemsnit over en længere baseline-periode (12 måneder, eller alt
tilgængelig historik hvis kortere). Sammenlign den aktuelle måned mod dette gennemsnit
(ikke kun mod forrige måned) og marker kategorier der afviger >20% fra deres egen
historiske norm som "skiller sig ud". Vis en 12-måneders sparkline/trendlinje pr.
hovedkategori, så brugeren kan se om en afvigelse er et engangsudsving (f.eks.
kvartalsvis realkreditbetaling) eller en ny vedvarende tendens.

**Dagligvarer pr. forretning (fast del af hver gennemgang):** Dagligvarer er et af de
områder familien kan ændre på kort bane, og brugeren vil løbende kunne se *hvor* pengene
går hen, ikke kun det samlede beløb. For hver gennemgang:
- Match dagligvare-transaktioner mod kædenavn (Nemlig, Netto, Føtex, Rema, Lidl, Coop,
  Meny, Irma, Kvickly, Bilka osv. — udbyg `data/category-rules.csv` med flere kæder
  efter behov).
- Vis en bjælke pr. forretning for seneste 12 måneder: beløb, andel af total dagligvarer,
  og antal køb. Farvekod: rød = online levering (Nemlig o.l. — typisk dyrere pr. vare pga.
  leveringsgebyr/minimumskøb), gul = mellem/specialbutik, grøn = discount/fysisk indkøb.
- Beregn snit pr. køb pr. forretning (beløb/antal) for at vise om en kæde bruges til få
  store indkøb eller mange små.
- Fremhæv i en kort tekstboks hvis online levering udgør en stor andel (>30%) af
  dagligvareforbruget, med et konkret besparelsesoverslag hvis en del af forbruget
  flyttes til discountbutik.
- Indsæt sektionen direkte i overblikket (ikke kun i en foldet drill-down), da dette er
  et prioriteret fokusområde for brugeren.

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
