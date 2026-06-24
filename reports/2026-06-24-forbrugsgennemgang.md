# Forbrugsgennemgang — 24. juni 2026

**Periode for hovedtal:** maj 2026 (sidste fulde kalendermåned — data findes til 23. juni 2026)
**Datakilde:** `data/transactions/alle-konti-2020-2026.csv` (3 konti, 9.114 transaktioner, 02-06-2020 til 23-06-2026)
**Status:** Dette er den første gennemgang — der findes ingen tidligere rapport at sammenligne med, så udviklingstal er beregnet ud fra de seneste 12 måneders historik i samme datasæt.

> **Datakvalitet, læs dette først:** Filen lå indtil i dag som `Claude_agent_test.csv` i skill-mappen i stedet for i `data/transactions/` — den er flyttet til den rigtige placering som del af denne kørsel. Kategoriseringen er regelbaseret (se `data/category-rules.csv`) og fanger størstedelen af transaktionerne, men en mindre rest (~7 % af maj's posteringer, netto +8.370 kr. pga. én uidentificeret indbetaling på 13.764 kr.) er endt i "Øvrigt/ukategoriseret". Tre konti er lagt sammen som "husholdningens samlede økonomi" — interne flytninger mellem egne opsparings-/budgetkonti er udskilt i egen linje, så de ikke tæller som forbrug.

## 1. Cashflow-overblik (maj 2026)

| Post | Beløb |
|---|---|
| Indtægt (løn, børnepenge, udbytte m.m.) | **119.835 kr.** |
| Fast + variabelt forbrug | **-68.380 kr.** |
| **Netto før opsparing** | **+51.455 kr.** |
| Til opsparing/investering/andre konti | -42.389 kr. |
| **Reel ændring i de 3 konti samlet** | **+9.066 kr.** |

Husholdningen havde et solidt overskud i maj og lagde samtidig en stor portion til side. Bemærk at "fast + variabelt forbrug" ikke inkluderer realkredit/husleje (se note under afvigelser) eller opsparing.

## 2. Udvikling over tid (sidste 7 måneder)

| Måned | Indtægt | Forbrug | Netto før opsparing | Opsparing/overf. | Saldoændring |
|---|---|---|---|---|---|
| 2025-12 | 131.775 | -129.181 | 2.594 | -9.900 | -7.306 |
| 2026-01 | 92.639 | -70.498 | 22.141 | -6.600 | 15.541 |
| 2026-02 | 130.237 | -125.293 | 4.944 | -30.179 | -25.235 |
| 2026-03 | 115.424 | -113.647 | 1.776 | -19.939 | -18.163 |
| 2026-04 | 190.183 | -68.124 | 122.059 | -16.000 | 106.059 |
| **2026-05** | **119.835** | **-68.380** | **51.455** | **-42.389** | **9.066** |
| 2026-06 (delvis, t.o.m. 23/6) | 141.317 | -58.624 | 82.693 | -33.200 | 49.493 |

12-måneders gennemsnit: indtægt ~116.200 kr./md., forbrug ~95.634 kr./md., opsparing/overførsel ~14.902 kr./md. Forbruget svinger meget måned til måned, primært fordi realkredit (Totalkredit) og nogle forsikringer/energiregninger betales kvartårligt/halvårligt og derfor rammer enkelte måneder hårdt (f.eks. dec. 2025 og feb.–mar. 2026).

## 3. Afvigelser der kræver opmærksomhed (maj vs. 12-mdr. gennemsnit)

| Kategori | Maj 2026 | 12-mdr. snit | Afvigelse | Kommentar |
|---|---|---|---|---|
| El/Vand/Varme | -19.470 | -5.328 | -265 % | Novafos (-11.270) og Gentofte Fjernvarme (-8.200) ramte samme måned — sandsynligvis kvartalsafregning, ikke et reelt prisspring. Værd at tjekke næste afregning for at bekræfte. |
| Sundhed | -11.429 | -1.328 | -760 % | Domineret af ét køb hos Profil Optik (-10.650 kr.) — engangsudgift (nye briller), ikke en trend. |
| Fritid/Sport | -3.593 | -870 | -313 % | Spejder Sport (-3.458) — sæsonudstyr, sandsynligt engangskøb. |
| Transport/rejse (dagstur) | -1.749 | -246 | -611 % | Hænger sammen med en dagstur til Sverige (Øresundsbro, duty-free) — se Rejser/Ferie. |
| Børn (tøj/legetøj) | -2.442 | -1.400 | -74 % | Højere end normalt, men ikke ekstremt. |
| Husleje/Realkredit | 0 | -16.332 | — | Ingen Totalkredit-betaling i maj — betales ikke hvert måned (kvartårlig), så 0 kr. er forventet, ikke en besparelse. |
| Opsparing/overførsel | -42.389 | -14.902 | -184 % | Der blev lagt markant mere til side end normalt — positivt, men tjek at det er en bevidst beslutning (bl.a. en overført ydelse på 25.389 kr.). |

**Ingen af disse er reelle alarmklokker** — de er stort set alle forklaret af engangskøb eller kvartalsvis fakturering. Den eneste der er værd at følge er el/vand/varme-niveauet over de næste 1-2 afregninger.

## 4. Benchmark vs. dansk familie med 2 børn

Kilder: [Danmarks Statistik — "Bolig fylder mest i danskernes husholdningsbudget"](https://www.dst.dk/da/Statistik/udgivelser/NytHtml?cid=52886) (Forbrugsundersøgelsen, 2024-tal, baseret på stikprøve af 2.651 husstande) og [Forbrugerrådet Tænk — rådighedsbeløb](https://taenk.dk/privatoekonomi/gode-raad/raadighedsbeloeb).

- **Bolig** udgør ifølge DST 37,1 % af en gennemsnitlig dansk husstands budget (371 kr. ud af hver 1.000 kr.), **transport** 14,2 %. Disse er nationale gennemsnit, ikke specifikt for familier med 2 børn, men giver en pejling.
- **Rådighedsbeløb:** Forbrugerrådet Tænk anbefaler min. 8.000 kr./md. til et par + 2.500 kr. pr. barn, dvs. **min. 13.000 kr./md.** for et par med 2 børn. Denne husstand har i gennemsnit ~20.566 kr./md. tilbage efter forbrug (før frivillig opsparing) — **markant over minimumsanbefalingen**, hvilket er et godt tegn.
- **Madbudget:** Vejledende budget for en familie på 4 ligger på 5.000-6.500 kr./md. til dagligvarer (jf. flere danske budgetguides baseret på Forbrugerrådets principper). Denne husstand brugte **7.359 kr.** på dagligvarer i maj og har et 12-måneders snit på **7.570 kr./md.** — **ca. 15-50 % over den vejledende ramme**.

*Forbehold: Der findes ikke en officiel, gratis tilgængelig DST-tabel opdelt specifikt på "par med 2 børn" med kronebeløb for hver kategori — tallene ovenfor er nationale gennemsnit og vejledende budgetter. Tjek `https://www.statistikbanken.dk` (tabel FU18) for mere detaljerede tal, hvis en mere præcis sammenligning ønskes.*

## 5. Optimeringsforslag

1. **Dagligvarer (7.360-7.570 kr./md.):** Ligger over det vejledende budget for en familie på 4. Et realistisk mål kunne være 6.000-6.500 kr./md. — potentiel besparelse ~1.000-1.500 kr./md.
2. **Forsikringer er spredt på 3 selskaber** (GF Forsikring, Forsia Forsikring, Sønderjysk Forsikring) — værd at indhente et samlet tilbud (pakkerabat findes typisk ved at samle bil/indbo/familieforsikring hos ét selskab).
3. **Restauranter/take-away (~4.044 kr./md. i 12-mdr. snit, kun 1.012 kr. i maj):** Svingende, men i de dyre måneder en oplagt post at sætte et månedligt loft på.
4. **Abonnementer:** Netflix, Apple.com/Bill, LinkedIn, TV2 DK m.fl. er fundet i data — værd at tjekke om alle stadig bruges aktivt, særligt LinkedIn Premium og TV2-abonnement, som ofte glemmes.
5. **El/vand/varme:** Bekræft om majs kvartalsregning (Novafos + Fjernvarme, samlet 19.470 kr.) er normal eller en stigning — sammenlign med samme kvartal sidste år, og overvej om en del af de tidligere renoveringer (isolering, vinduer) allerede har givet effekt.

## Konklusion

Økonomien ser sund ud: indtægterne dækker forbruget komfortabelt, der spares løbende til side, og rådighedsbeløbet ligger godt over den danske minimumsanbefaling for en familie med 2 børn. De største "afvigelser" denne måned er engangsudgifter (briller, spejderudstyr) eller forventet kvartalsfakturering — ikke tegn på et stigende forbrugsniveau. Den eneste post, der konsekvent ligger over et eksternt benchmark, er dagligvarer.
