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
