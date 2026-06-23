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

## 6. Skriv rapporten

Gem rapporten som Markdown i `reports/YYYY-MM-DD-forbrugsgennemgang.md` med sektionerne:
1. Cashflow-overblik (periode, indtægt, faste omk., variabelt forbrug, netto)
2. Udvikling over tid (sammenlignet med tidligere perioder)
3. Afvigelser der kræver opmærksomhed
4. Benchmark vs. dansk familie med 2 børn (med kildehenvisning)
5. Optimeringsforslag

## 7. Send til brugeren

Opret et Gmail-udkast (`mcp__Gmail__create_draft`) til michaelskouboe@gmail.com med
emnet "Forbrugsgennemgang <dato>" og rapportens indhold som brødtekst (brug htmlBody
for pæn formatering). Gmail-integrationen kan kun oprette udkast, ikke sende
automatisk — informer brugeren om, at udkastet ligger klar til afsendelse.

## Noter

- Hvis dette køres som en planlagt/automatisk trigger uden interaktiv bruger til stede,
  skal du stadig gennemføre alle trin og oprette udkastet — spørg ikke brugeren om
  bekræftelse undervejs.
- Commit nye/opdaterede filer i `reports/` og `data/category-rules.csv` til git, så
  historikken er bevaret til næste gennemgang.
