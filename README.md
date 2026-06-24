# Skouboe – Forbrugsgennemgang

Dette repo driver en tilbagevendende økonomisk gennemgang: hver anden mandag
analyseres privat forbrug og faste omkostninger, der laves en cashflow-/trend-
rapport, afvigelser fremhæves, og forbruget sammenlignes med en gennemsnitlig
dansk familie med 2 børn (baseret på offentlige statistikker). Resultatet
leveres som et Gmail-udkast.

## Sådan virker det

1. **Eksporter transaktioner** fra din netbank som CSV og læg filen i
   `data/transactions/` (en fil pr. periode/måned er fint).
2. Skill'en `.claude/skills/expense-review` indeholder alle instrukser til
   analysen: kategorisering, cashflow, trendanalyse, afvigelser, benchmark
   og rapportgenerering.
3. Rapporter gemmes i `reports/` som Markdown, så historikken bevares og
   bruges til at vise udvikling over tid i fremtidige gennemgange.
4. Et Gmail-udkast oprettes automatisk med rapporten — du skal selv trykke
   "send", da integrationen ikke sender e-mails direkte.

## Sådan sætter du den tilbagevendende kørsel op

Dette repo bygger selv ikke en cron-scheduler. Opsæt i stedet en **Trigger**
i Claude Code on the web (se docs: https://code.claude.com/docs/en/claude-code-on-the-web):

- Skema: hver anden mandag
- Prompt: "Brug skill'en expense-review og gennemfør forbrugsgennemgangen"
- Repo: `michaelskouboe-droid/skouboe`, branch: `claude/dazzling-albattani-nhffi1` (eller main, når den er merged)

## CSV-format

Forventede kolonner (typisk fra netbank-eksport): dato, beskrivelse/tekst, beløb
(negativ = udgift), evt. saldo. Header-navne kan variere mellem banker —
skill'en forsøger at matche automatisk.

## Kategori-regler

`data/category-rules.csv` opbygges og forfines automatisk over tid, så
kategoriseringen af dit forbrug bliver mere præcis for hver gennemgang.

---

# Porteføljegennemgang (aktier)

Et separat, tilbagevendende spor i samme repo: hver anden uge analyseres hele
aktieporteføljen på tværs af handelsplatforme (i dag Nordnet med 3 depoter, og
Saxo Bank) — samlet værdi, allokering, valuta- og koncentrationsrisiko, udvikling
over tid, og konkrete anbefalinger. Resultatet leveres som et Gmail-udkast.

## Sådan virker det

1. **Eksporter depot/beholdning** fra hver platform som CSV (Nordnet: "Beholdning"
   eller "Aktietabel"-eksport; Saxo: "Portfolio"-eksport) og læg dem i Google
   Drive — skill'en finder selv de nyeste filer.
2. Skill'en `.claude/skills/portfolio-review` indeholder alle instrukser:
   parsing af de forskellige platformformater, normalisering, allokering,
   koncentrations-/FX-flags, trend og rapportgenerering.
3. Alle snapshots normaliseres til `data/portfolio/holdings-history.csv`
   (akkumuleres over tid — slettes aldrig), og rå kildefiler arkiveres under
   `data/portfolio/snapshots/YYYY-MM-DD/`.
4. Rapporter gemmes i `reports/portfolio/` som HTML, så historikken bevares og
   bruges til at vise udvikling over tid i fremtidige gennemgange.
5. Et Gmail-udkast oprettes automatisk med rapporten — du skal selv trykke
   "send".

## Sådan sætter du den tilbagevendende kørsel op

Opsæt en **Trigger** i Claude Code on the web:

- Skema: hver anden uge
- Prompt: "Brug skill'en portfolio-review og gennemfør porteføljegennemgangen"
- Repo: `michaelskouboe-droid/skouboe`

## Datamodel

`data/portfolio/holdings-history.csv` indeholder én række pr. position pr.
snapshot-dato: `dato,platform,depotnavn,depotnummer,navn,isin,valuta,antal,kurs,
vaerdi_dkk,anskaffelse_dkk,ureal_dkk`. `anskaffelse_dkk`/`ureal_dkk` er kun
udfyldt for kilder med kostbasis (i dag Nordnet Beholdning-eksport — Saxo viser
kun øjebliksværdi).

## Fremtidige forbedringer (oplæg)

- **Kostbasis fra Saxo**, hvis platformen kan eksportere det, så afkast kan
  beregnes for hele porteføljen, ikke kun Nordnet-delen.
- **Sektor-/branchefordeling** — kræver enten manuel mapping pr. ISIN eller en
  ekstern datakilde, da CSV-eksportene ikke indeholder sektor.
- **Realiseret afkast/transaktionshistorik** (køb/salg/udbytte) som supplement
  til den nuværende beholdnings-øjebliksvisning, til at se faktisk performance
  over tid i stedet for kun urealiseret gevinst.
- **Benchmark mod et indeks** (f.eks. MSCI World eller C25), så det er muligt
  at se om porteføljen slår markedet — kræver en ekstern kursserie.
