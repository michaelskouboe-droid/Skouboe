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

**"Husholdning"-kategorien skal altid specificeres:** Kategorien "Husholdning" dækker
indkøb til hjemmet: møbler, rengøringsmidler, isenkram, byggematerialer, dekoration,
haveartikler og håndværkerudgifter. Da beløbet kan svinge meget, skal dashboardet
ALDRIG vise Husholdning som én lukket linje. Vis altid:
- Kategoritotal øverst
- En udfoldet liste med de 5 største transaktioner (forretning + beløb) direkte i
  kortet — ikke bag et fold-ud element
- Tooltip/label: "Husholdning = møbler, rengøring, byggematerialer, haveartikler, VVS"

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

**Kvartalsvise og årslige betalinger — amortisering:**
Detektér kvartalsvise og årslige engangsbetalinger, der ellers ville forvrænge den
aktuelle måneds tal. En betaling betragtes som kvartalvis/periodisk hvis:
- Transaktionsbeløbet er ≥ 2,5 × kategoriets historiske månedlige gennemsnit, OG/ELLER
- Beskrivelsen indeholder mønstre som "KVARTAL", "Q1", "Q2", "Q3", "Q4", "PERIODE",
  "HALVÅR", "HELÅR", "ÅRSOPKRÆVNING", eller lignende periodeangivelser.

Når en periodisk betaling detekteres:
1. Spread beløbet over 3 måneder (kvartalsvis) eller 12 måneder (årslig).
2. Brug den amortiserede månedlige andel i cashflow-diagrammer og kategoribjælker.
3. Vis begge tal i dashboardet: "Amortiseret: 3.200 kr/md (faktisk betalt: 9.600 kr)"
4. Tilføj et info-kort "Kvartalsvise/periodiske betalinger" der lister alle amortiserede
   poster med fuldt beløb og fordeling, så brugeren ved hvad der skjuler sig bag tallene.
5. I 12-måneders cashflow-diagrammet: brug altid amortiserede værdier for at undgå
   kunstige toppe.

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

## 6. Byg rapporten som interaktivt HTML-dashboard

Rapporten gemmes som en selvstændig HTML-fil i `reports/YYYY-MM-DD-forbrugsgennemgang.html`.
Dashboardet skal fungere i en browser uden netadgang — brug kun inline `<style>` og
`<script>` blokke. Ingen eksterne CDN, ingen billeder.

### Designprincipper (dark-mode dashboard)

Brug CSS-variabler med følgende farvepalette:
```css
--bg: #0f172a;        /* baggrund */
--card: #1e293b;      /* kortbaggrund */
--border: #334155;    /* kant */
--text: #e2e8f0;      /* primær tekst */
--muted: #94a3b8;     /* sekundær tekst */
--accent: #38bdf8;    /* accentfarve (blå) */
--green: #22c55e;     /* positiv/god */
--yellow: #eab308;    /* advarsel */
--red: #ef4444;       /* negativ/dårlig */
--orange: #f97316;    /* udgifter */
```

Layout: CSS Grid med `grid-template-columns: repeat(auto-fit, minmax(300px, 1fr))` for
responsive kortlayout. Brug `border-radius: 12px` og `padding: 20px` på kort.

### Sektioner i dashboardet (i rækkefølge)

**A. KPI-bjælke (altid synlig øverst)**
4 kort side om side: Indtægt, Faste omkostninger, Variabelt forbrug, Netto.
Hvert kort: stort tal (28px bold) + delta vs. 12-måneders snit (grøn ▲/rød ▼ + %).
Netto-kortet farves grønt hvis positivt, rødt hvis negativt.

**B. Afvigelsesadvarsler**
Maks. 5 badges i en vandret linje: "⚠️ Transport +28% vs. snit" — kun kategorier der
afviger >20% fra historisk snit. Brug amortiserede værdier.

**C. Kvartalsvise betalinger (vis kun hvis der er detekterede)**
Et informationskort med liste: "Realkredit Q2: 9.600 kr betalt → 3.200 kr/md amortiseret
over 3 måneder." Forklar at cashflow-tal bruger amortiserede værdier.

**D. 13-måneders cashflow-diagram**
Stacked SVG-søjlediagram (inline SVG, ingen canvas-API).
- X-akse: 13 måneder (indeværende + 12 tidligere)
- Y-akse: DKK, auto-skaleret
- Blå søjle = Indtægt, Orange søjle = Udgifter (amortiserede tal)
- Grønt tal under søjle = positiv netto, rødt = negativ netto
Byg diagrammet med JavaScript der injiceres med data som en JSON-konstant øverst i
script-blokken.

**E. Kategorifordeling**
For hver kategori: en bjælke (CSS `width: X%`) med:
- Kategori-navn + beløb (amortiseret) til venstre
- Procent af totalt variabelt forbrug til højre
- Farvekod: grøn = under/på benchmark, gul = let over, rød = markant over (>20%)
- En sparkline af de seneste 6 måneder som en række af 6 mini-bjælker

**F. Husholdning — altid udfoldet**
Udover den normale kategoribjælke: et udvidet kort specifikt for Husholdning der viser:
- Total med amortisering hvis relevant
- Top 5 transaktioner (forretning + beløb) listet direkte — IKKE bag et fold-ud
- Tooltip under kortets titel: "Møbler · Rengøring · Byggematerialer · Haveartikler · VVS"

**G. Forbrug pr. forretning — månedsoverblik**
Et interaktivt kort med to tab-knapper: "Denne måned" og "Seneste 12 måneder".
JavaScript toggle viser/skjuler de relevante data.
For hver visning: rangeret liste med bjælker — forretning, beløb, antal transaktioner,
snit pr. køb. Brug inline JavaScript med data embeddet som JSON-objekt i script-blokken.
Sorter efter beløb (højeste først). Vis top 20 forretninger.

**H. Dagligvarer pr. forretning pr. måned — 12 måneder**
Et stacked SVG-søjlediagram (inline SVG):
- X-akse: 12 måneder
- Y-akse: DKK
- Hver søjle opdelt i farvelagte segmenter pr. butikskæde
- Farvepalette pr. kæde: Nemlig=#ef4444, HelloFresh=#f97316, Netto=#22c55e,
  Rema=#16a34a, Lidl=#15803d, Føtex=#3b82f6, Bilka=#2563eb, Meny=#eab308,
  Coop=#a3a3a3, Øvrige=#64748b
- En legend under diagrammet med farve + kædenavn
- Under diagrammet: eksisterende enkelt-periode breakdown med bjælker pr. forretning
  (rød=online levering, gul=mellemkæde, grøn=discount)

**I. Benchmark-sammenligning**
Kompakt tabel: Kategori | Dit forbrug | Benchmark | Forskel. Kilde noteret.

**J. Optimeringsforslag**
Nummereret liste (1-5): fed overskrift per forslag + én linje begrundelse.

**K. Drill-down detaljer (foldet som udgangspunkt)**
`<details><summary>` for:
- "Se alle transaktioner pr. kategori"
- "Se fuld benchmark-sammenligning med kilde"
- "Se beregningsgrundlag for optimeringsforslag"

### JavaScript-data injection

Øverst i `<script>`-blokken defineres alle data som konstanter:
```js
const DATA = {
  months: [...],           // 13 måneds-labels
  income: [...],           // 13 tal (amortiseret)
  expenses: [...],         // 13 tal (amortiseret)
  categories: [...],       // [{name, amount, benchmark, sparkline:[6]}]
  merchants: {
    thisMonth: [...],      // [{name, amount, count}]
    last12: [...]          // [{name, amount, count}]
  },
  groceryByStore: {        // [{store, color, months:[12 tal]}]
    stores: [...],
    months: [...]
  },
  amortized: [...]         // [{description, actual, monthly, months}]
};
```

Skriv så lidt løbende tekst som muligt — foretræk tal, bjælker, farver og korte
labels over sætninger. Brug dansk sprog og danske tal-/valutaformater (kr., 1.234,56).

## 7. Send til brugeren

Gem den fulde HTML i `reports/YYYY-MM-DD-forbrugsgennemgang.html`.

Opret et Gmail-udkast (`mcp__Gmail__create_draft`) til michaelskouboe@gmail.com med
emnet "Forbrugsgennemgang <dato>". Gmail-udkastet skal være en **kompakt opsummering**
(ikke det fulde dashboard — Gmail understøtter ikke komplekse scripts):

- 4 KPI-kort som en enkel HTML-tabel (inline styles, ingen scripts)
- Top 5 afvigelser som en punktliste
- En tekstlinje: "Det fulde interaktive dashboard ligger i reports/YYYY-MM-DD-forbrugsgennemgang.html"
- Optionelt: 1-2 vigtigste optimeringsforslag

Gmail-integrationen kan kun oprette udkast, ikke sende automatisk —
informer brugeren om, at udkastet ligger klar til afsendelse.

## Noter

- Hvis dette køres som en planlagt/automatisk trigger uden interaktiv bruger til stede,
  skal du stadig gennemføre alle trin og oprette udkastet — spørg ikke brugeren om
  bekræftelse undervejs.
- Commit nye/opdaterede filer i `reports/` og `data/category-rules.csv` til git, så
  historikken er bevaret til næste gennemgang.
- KRITISK: Opret aldrig mere end ét Gmail-udkast pr. kørsel. Kontrollér om et udkast
  allerede eksisterer med samme dato i emnefeltet (brug `mcp__Gmail__list_drafts`)
  inden du opretter et nyt. Hvis et eksisterende udkast findes, opdater det i stedet
  eller spring oprettelsen over og informer brugeren.
