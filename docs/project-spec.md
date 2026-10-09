# Off-Peak: Did London Stop Commuting?

**Project specification, v2 (post-audit) · 9 Oct 2026**

This file is the authoritative project contract. If anything here conflicts with conversation history, this file wins. Changes follow the change-control rules below.

v2: one page, branded Off-Peak, time-band data deferred, Phase 0 audit complete.

## Purpose

v1 is a **single-page** Power BI report, branded **Off-Peak**, that asks one question: **did London stop commuting?** It uses TfL's daily station footfall from 2019 onwards, built in about a week, learning included.

It has to prove to a hiring manager what the F1 project doesn't:

- Power Query: importing, cleaning and reshaping real public data inside Power BI
- A proper star-schema data model with sensible relationships
- DAX measures that answer a question, not just sum a column
- Report design that looks bespoke, not default
- Validation: numbers reconciled against an independent check
- Communication: three findings a decision-maker can act on

Findings are always phrased as *consistent with* changed travel patterns. Taps show when people travel, never why.

Off-Peak is the brand, not a measurement. Daily data shows *which days* people travel, not whether they travel in peak hours, so the title stays a question.

## Roles

| Who | Responsibility |
| --- | --- |
| Radi | Power BI development and learning: Power Query, model, DAX, report |
| Claude Code | Supporting code, Git, documentation, validation script |
| Reviewers (Claude chat, ChatGPT) | Methodology scrutiny, review, explaining modelling decisions |

Work happens one meaningful operation at a time. Claude does not build the Power BI report.

## Scope

Scope froze at the end of Phase 0 (9 Oct 2026). Nothing is added to v1 except through change control.

**In v1**

- **One report page.** It opens on All London; pick a station and every visual retells that station's story, with London kept as a reference
- Daily station footfall, pinned snapshot from 1 Jan 2019 to 4 Jul 2026
- The measures listed in Metrics, and no others
- Power BI's built-in tooltips only
- Off-Peak theme file and one designed page background
- GitHub repo `off-peak` with README, three findings, screenshots, demo video, the PBIP project and a .pbix on the release

**Out of v1 (parking lot)**

- Station map. It needs a third source for coordinates and a second round of name matching. First candidate for v1.1.
- Bus journeys, Santander Cycles, weather or events data
- TfL's Journeys files (not needed; Station Footfall is the dataset)
- A public Power BI link. It needs a work or school account, so screenshots and video cover it.
- Preprocessing data in Python. Python is only used for the independent validation check.
- Any metric, visual or page not named in this spec

**Deferred from the v1 plan**

- Annual time-band counts (AM peak share). They need a second fact table and their own audit gate.
- Station-to-station comparison. London is the only comparison in v1.
- Station identity labels (Weekday-led, Balanced, Weekend-led). Numerical comparisons replace them.
- A second report page and a custom tooltip page.

**Change control**

1. A new idea goes in the parking lot, never straight into the build.
2. It only enters v1 if it replaces something of equal or bigger effort.
3. This spec is updated before any work on it starts.
4. Nothing changes mid-phase. Changes are only considered at a gate.

## Data sources

| Source | Grain (confirmed) | Used for |
| --- | --- | --- |
| [TfL station footfall](https://tfl.gov.uk/corporate/publications-and-reports/network-demand-data): seven CSVs (2019–2024 yearly, one 2025–2026 file) | Station × travel day, entry and exit taps | Every visual and measure |
| [UK bank holidays](https://www.gov.uk/bank-holidays.json) (England and Wales) | One row per holiday date | Excluding bank holidays from weekday and weekend measures |

Columns: `TravelDate` (text, yyyymmdd), `DayOfWeek` (`DayOFWeek` in 2019–2022), `Station`, `EntryTapCount`, `ExitTapCount`.

File names, sizes and SHA-256 checksums are in `data/README.md`. The seven pinned CSVs live in `data/raw/` and are versioned with the repo. No other raw files are committed.

## Phase 0 audit results

**Gate A passed on 9 Oct 2026.** The footfall data is clean, complete and usable. The audit forced four changes to the plan, all reflected in this spec: the denominator, the cohort, the Elizabeth line corridor and the station name map.

**Pinned snapshot:** downloaded 9 Oct 2026, data from 1 Jan 2019 to Sat 4 Jul 2026. TfL's file hadn't refreshed since July despite the page saying weekly. We build on this snapshot and don't re-download mid-build. The latest full year is 2025.

| File | Dates | Rows | Stations |
| --- | --- | --- | --- |
| StationFootfall_2019.csv | 1 Jan – 31 Dec 2019 | 151,639 | 421 |
| StationFootfall_2020.csv | 1 Jan – 31 Dec 2020 | 150,089 | 428 |
| StationFootfall_2021.csv | 1 Jan – 31 Dec 2021 | 154,167 | 433 |
| StationFootfall_2022.csv | 1 Jan – 31 Dec 2022 | 155,130 | 435 |
| StationFootfall_2023.csv | 1 Jan – 31 Dec 2023 | 156,491 | 434 |
| StationFootfall_2024.csv | 1 Jan – 31 Dec 2024 | 156,995 | 434 |
| stationfootfall-2025-2026.csv | 1 Jan 2025 – 4 Jul 2026 | 235,636 | 437 |

**Checks passed:** no overlapping dates between files, no duplicate station-dates, no missing calendar dates, no nulls or negative counts, and every DayOfWeek label matches its date.

**Power Query fixes needed**

- Header changes from `DayOFWeek` (2019–2022) to `DayOfWeek` (2023 on). Rename before appending.
- 2019–2023 files carry a byte-order mark; import as UTF-8.
- `TravelDate` is text in yyyymmdd form; convert to a date.
- Trim station names: "Cannon Street " with a trailing space (2019–2022) and "Cannon Street" are the same station.
- Elizabeth line renames on 28 May 2025: "Canary Wharf EL", "Custom House EL" and "Woolwich EL" become "Canary Wharf Elizabeth Line", "Custom House Elizabeth Line" and "Woolwich Elizabeth Line". Map each pair to one name.
- Drop the "Unknown" station (Feb 2020 to Dec 2022).

**Missing rows: days without a reported observation.** Only 47 of 421 stations have a row for every day of 2019. Of 1,320 missing station-days that year, 401 fall on 25–26 December and most others on weekends: a pattern consistent with planned closures, but an absent row doesn't prove one. Avg Daily Taps divides by the days a station has a row (average recorded activity) and never zero-fills. That's an assumption: if absence patterns differ between 2019 and 2025 it biases recovery, so Phase 4 tests it.

**Odd rows kept as reported:** 6,339 rows have zero entries but some exits, 2,581 the reverse, 26 both zero. TfL doesn't correct for open gates, and neither do we; the README says so.

**The Elizabeth line corridor breaks a naive 2019 comparison.** 30 stations that existed in 2019, then as TfL Rail, National Rail or Tube-only stations, now carry Elizabeth line traffic: up to 9× their 2019 level (Acton Main Line, Heathrow, Hanwell, Abbey Wood). Their change is network change, not behaviour change. They stay explorable but are left out of London figures and the ranking. The list is in Appendix A. It includes big central interchanges (Bond Street, Tottenham Court Road, Farringdon, Liverpool Street, Paddington, Whitechapel, Stratford) because their counts now include Elizabeth line gatelines.

**Cohort, final:** 389 stations with a row on at least 300 days of 2019, still reporting in 2025, outside the Elizabeth line corridor, excluding Unknown. Post-2019 stations (Battersea Power Station, Nine Elms, Barking Riverside, Paddington EL, the western Elizabeth line stops, Reading, Slough, the renamed Elizabeth line entries) can be explored but never count towards London.

**Early signal (unvalidated; Phase 4 confirms):** for the cohort, 2025 traffic is 82% of 2019 overall, 79% on weekdays and 95% at weekends, bank holidays excluded. By day: Mon 75%, Tue 81%, Wed 80%, Thu 81%, Fri 78%, Sat 95%, Sun 94%. Including the corridor would lift these to 87%, 83% and 100%, which is why it's excluded.

**Ranking caveat:** with the corridor excluded, the biggest weekend-over-weekday gaps include regeneration and stadium effects (Pudding Mill Lane, White Hart Lane) as well as the City (Monument, Liverpool St NR). Those are real changes but not all commuting. Hover text and the README say so; the list isn't filtered to fit the story.

**Data Guide notes that become README limitations:** a tap is an entry or exit, not a passenger or journey. Travel days run 04:30 to 04:30, and `TravelDate` already follows that convention, so dates are used as given and never shifted to midnight. TfL does not scale taps for missing taps or open gates. Interchange stations count all taps across lines. Pink-validator taps aren't counted. Paper-ticket coverage varies by station, and some shared stations such as Reading only include contactless taps. National Rail-only stations are excluded by TfL.

**Licence:** attribution reads "Powered by TfL Open Data". TfL's terms are based on Open Government Licence v2.0 and forbid implying official status or TfL endorsement ([TfL transport data terms](https://tfl.gov.uk/corporate/terms-and-conditions/transport-data-service)).

## Data model and Power Query

A simple star schema: one fact table and two dimensions. The station picker filters through DimStation, so every visual on the page follows it.

```
DimDate                FactFootfall               DimStation
-----------            ------------------         ------------------
Date (key)    1 ──> *  Date                * <── 1 StationKey (key)
Year, Month            StationKey                  StationName
DayOfWeek              EntryTaps, ExitTaps         HasBaseline2019
IsWeekend              one row per station         ELCorridor
IsBankHoliday          per travel day              InCohort
```

Both relationships run one way, dimension to fact.

**Power Query steps (Phase 1, done by Radi in Power BI)**

1. Import the footfall files with **Get Data → Folder**, rename `DayOFWeek` to `DayOfWeek` in the 2019–2022 files, and append them into one staging query.
2. Set data types, convert `TravelDate` to a date, trim station names, and remove blank rows.
3. Create a small **StationMap** table with Enter Data: raw name → clean name. It merges the three Elizabeth line renames of 28 May 2025, drops Unknown, and flags the 30 corridor stations in Appendix A.
4. Merge StationMap into the footfall query to make **FactFootfall**, then build **DimStation** from the distinct clean names.
5. Flag each station with **HasBaseline2019** if it has a row on at least 300 days of 2019, then **InCohort** if it also has 2025 rows and isn't in the corridor. Expect 389.
6. Build **DimDate** covering 1 Jan 2019 to 4 Jul 2026: Year, Month, DayOfWeek, IsWeekend.
7. Import the bank-holiday JSON and merge it into DimDate as **IsBankHoliday**.
8. Turn off load for every staging query so only the three model tables reach the report.

## Metrics and DAX measures

Ten measures, capped. Every comparison uses average daily figures, never raw totals, so years of different lengths compare fairly.

| Measure | Definition | Used on |
| --- | --- | --- |
| Total Taps | Entry taps + exit taps | Base for everything |
| Avg Daily Taps | Total taps ÷ days in the period on which the station has a row. A missing row is a day without a reported observation; never zero-filled | Week chart, every ratio |
| Recovery vs 2019 | Avg Daily Taps in the latest full year ÷ same in 2019, bank holidays excluded | KPI card |
| Weekday Recovery | Mon–Fri Avg Daily Taps, latest full year ÷ 2019 | KPI card, station list |
| Weekend Recovery | Sat–Sun Avg Daily Taps, latest full year ÷ 2019 | KPI card, station list |
| Weekend–Weekday Gap | Weekend Recovery − Weekday Recovery, in percentage points | Ranks "changed most" by size, sign kept: + means weekends recovered more, − means weekdays did |
| Monthly Index | A month's Avg Daily Taps ÷ the same month in 2019 × 100 | Trend chart |
| London Reference | Each headline measure with the station filter removed (REMOVEFILTERS on DimStation), restricted to InCohort | Dashed lines, "London" under each card |
| Recovery Rank | RANKX of Recovery vs 2019 across cohort stations | Selected station's card |
| Selection Label | "All London" or the picked station's name | Dynamic titles |

Every measure answers for the current selection: London by default, the picked station otherwise. "London" always means the 389-station cohort, so neither new stations nor the Elizabeth line corridor inflate recovery. London Reference is one pattern applied to the headline measures, so it counts once against the cap.

Dropped from v1: Weekend Share, Midweek Index, AM Peak Share, Station Identity, Station Verdict. The week chart shows the midweek hump without a measure of its own.

## The page

One page, 1920 × 1080. It opens on **All London**. Pick a station and every visual retells that station's story, with London kept as a dashed reference. One question, one picker, three cards, three visuals.

1. **Header:** the Off-Peak mark and wordmark, the question, a station search box (default All London), a "Back to All London" button once a station is picked, and a "data to [date]" label.
2. **KPI row (3 cards):** Recovery vs 2019, Weekday Recovery, Weekend Recovery. With a station picked, each card adds "London [value]" underneath. The selected station's card also shows its Recovery Rank.
3. **How has demand evolved?** Monthly Index from 2019 to the latest month, with the current year labelled as partial. The selection is a solid teal line; London is dashed when a station is picked. Annotated with the first lockdown (23 Mar 2020) and the Elizabeth line opening (24 May 2022).
4. **How has the week changed?** Avg Daily Taps for Mon–Sun, indexed so the 2019 weekday average = 100 (a station and London then share one scale), 2019 vs the latest full year, with London markers when a station is picked. The midweek hump, if there is one, shows here.
5. **Which stations changed most?** The top 10 by the size of the Weekend–Weekday Gap, in either direction. Each row is a dumbbell from weekday to weekend recovery, built natively: a line chart with markers only, plus error bars spanning the two values (fallback: a clustered bar chart; no paid visuals). The list ignores the station search (Edit interactions set to None), so it always shows the full top 10.
6. **Three findings:** a short text panel, written after validation, labelled as London-wide and static whatever station is picked.

Stations without a 2019 baseline can still be picked. Their recovery cards show "No 2019 baseline" instead of a number, both indexed charts show an empty state saying the same, and they never appear in the ranked list. Elizabeth line corridor stations keep their numbers, carry an "Elizabeth line changed this station" note, and are also left out of the ranking.

The station search is the only way to pick a station. Row-click selection in the list is a stretch goal, added only if it behaves consistently with the search when tested.

## Brand and visual standard

**Off-Peak** · *Did London stop commuting?* · an independent look at how London's station travel changed since 2019. Deep teal and warm neutrals, deliberately not a TfL blue, so it never reads as an official TfL product.

**The mark** is two dots joined by a line: a warm grey dot for 2019, a teal dot for now. It's the before-and-after comparison the whole report is built on. It appears in the report header, the README banner, the GitHub social preview and the LinkedIn thumbnail.

| Token | Value | Use |
| --- | --- | --- |
| Canvas | #F5F3EE | Page background (warm paper tone) |
| Panel | #FFFFFF | Cards, with a very faint shadow |
| Ink | #1D1C1A | Titles, big numbers |
| Muted | #6B675F | Labels, axes, footnotes |
| Grid | #E4E0D7 | Gridlines, dividers |
| Baseline 2019 | #ABA59B | Every 2019 or "before" mark |
| Teal | #0F6B6B | The current selection and every "now" mark |
| London reference | #3B3936, dashed | London, shown only when a station is picked |
| Decline | #B5462E | Falls only, used sparingly |

**Rules**

- Fonts: **DIN** for big numbers, **Segoe UI** for everything else. Both ship with Power BI. The Off-Peak wordmark is part of the background image, so its font is free to choose.
- Line charts use thick strokes (3 px) with rounded ends.
- No visual borders. Cards have 12 px corners and sit on a 24 px grid.
- Every chart title states the finding, not the topic, once findings exist.
- Numbers formatted for reading: 1.2M, 87%, +4 pp.
- No TfL roundel, logo, Johnston typeface or TfL blues. It's an independent project.
- Never claim to measure off-peak travel. Off-Peak is the name, the question is the subject.
- One theme JSON file holds all of this, so every visual starts on-brand.

## Analytical rules

These rules apply to every visual, title and finding. Breaking one is a bug, not a style choice.

1. **Correlation, not cause.** Write "consistent with hybrid working", never "caused by". Taps show when people travel, not why or who they are.
2. **Shares always with volumes.** A station can look more weekend-led just because its weekdays collapsed. Any share is shown next to its volume.
3. **Average daily figures, never raw totals,** whenever periods of different lengths are compared.
4. **Like for like.** Weekdays compare with weekdays, Tuesdays with Tuesdays. Bank holidays are left out of day-of-week measures.
5. **2019 is the baseline.** Stations outside the cohort are left out of London figures and the ranking. Post-2019 stations are labelled "No 2019 baseline"; Elizabeth line corridor stations show their own numbers with an "Elizabeth line changed this station" note. Network expansion is never mistaken for recovery.
6. **The latest full calendar year (2025) is the "now" comparison.** The current year to date appears only on the trend chart, labelled as partial.
7. **Anomalies are annotated, not deleted.** Strike days, closures and events stay in the data. Only confirmed ones get a label.
8. **Every number carries its as-of date.** The header shows the latest date in the model.
9. **Travel days are TfL's 04:30 to 04:30 days.** Dates are used as given, never shifted.

## Validation and definition of done

Every headline number must be reproduced by an independent Python script, outside Power BI, before anything is published.

**Validation checks (Phase 4)**

- Row counts and total taps per year match between the raw files and the Power BI model.
- 5 stations × 3 dates spot-checked in the report against the raw CSVs.
- Recovery vs 2019, Weekday Recovery, Weekend Recovery and the Weekend–Weekday Gap recomputed for London and 3 stations in Python. They must match Power BI to within 0.1%.
- Every raw station name is mapped; no unmapped names left over. The cohort holds 389 stations and the corridor 30.
- Missing-row assumption tested: compare the share of station-days without a row in 2019 vs 2025 for the cohort, and recompute headline recovery with a calendar-day denominator. If the two differ by more than 1 percentage point, the README reports both.
- Changes in station count between years are explained in the README.
- Edge cases tested in the report: a station with no 2019 baseline, a corridor station, the busiest station, the quietest station.

**Definition of done**

- [ ] All validation checks pass and the results are saved in the repo
- [ ] The page matches the visual standard at 100% zoom
- [ ] Every chart title states a finding
- [ ] Three findings written, each backed by a number on the report
- [ ] README covers the question, data sources and licence, method, rules, findings and limitations
- [ ] Screenshots and a 60–90 second demo video recorded
- [ ] Repo public, linked from LinkedIn Featured and Projects

## Phases and gates

About a week in total. The phase times are build time; learning Power BI comes on top, and understanding why a measure works beats saving a day. No phase starts until the gate before it is passed. If a gate fails, fix it inside that phase.

| Phase | Time | Who does the work | Gate to pass | Claude model and effort |
| --- | --- | --- | --- | --- |
| 0 · Data audit | done | Claude, in chat | A: footfall usable (passed 9 Oct 2026) | Opus 5.5, high |
| 1 · Power Query and model | 1 day | Radi in Power BI; guidance in chat | No unmapped stations; row counts match raw files; cohort = 389 | Sonnet 5.5, high |
| 2 · DAX measures | half to 1 day | Radi in Power BI; explanations in chat | Measures agree with hand checks on London and 3 stations | Opus 5.5, high: filter-context bugs are silent |
| 3 · Page and design | half to 1 day | Claude makes the theme, background and mark; Radi builds the page | The page meets the visual standard at 100% zoom | Sonnet 5.5, high |
| 4 · Validation | half a day | Claude Code writes validate.py; Radi runs it | Every validation check passes | Sonnet 5.5, high: it's the accuracy proof |
| 5 · Findings and ship | half a day | Radi writes findings; Claude edits the README | Definition of done | Sonnet 5.5, medium for writing; Haiku 5.5, medium for git |

## Deliverables and repo

One public GitHub repo, **off-peak** (private until v1 ships). The seven pinned TfL CSVs are versioned in `data/raw/`; `data/README.md` documents their checksums, attribution and how to verify them.

**Report format:** the report is developed as a Power BI Project (`.pbip`), so the semantic model (TMDL) and report definition (PBIR, where the installed Desktop version supports it) are text files that Git and Claude Code can diff and review. The recruiter-friendly `.pbix` is produced at release with **File → Save as**, attached to a GitHub Release, and never tracked in history. Local PBIP caches (`.pbi/localSettings.json`, `.pbi/cache.abf`) are ignored; `.pbi/editorSettings.json` is tracked.

```
off-peak/
├── README.md              question, sources + licence, method, rules, 3 findings, limitations
├── report/
│   ├── off-peak.pbip                project entry point (open this in Power BI Desktop)
│   ├── off-peak.Report/             report definition (PBIR)
│   └── off-peak.SemanticModel/      model, Power Query and DAX (TMDL)
├── theme/
│   └── off-peak.json      the Power BI theme
├── assets/
│   ├── mark.svg           the two-dot mark
│   ├── bg-off-peak.png    page background with wordmark
│   └── social-preview.png GitHub and LinkedIn thumbnail
├── validation/
│   ├── validate.py        independent recompute of headline numbers
│   ├── requirements.txt
│   └── results.md         pass/fail output, committed
├── docs/
│   ├── project-spec.md    this file
│   ├── screenshots/
│   └── demo.mp4
└── data/
    ├── README.md          sources, snapshot date, checksums
    └── raw/               the seven pinned CSVs, versioned
```

## Appendix A: Elizabeth line corridor (30 stations)

Stations present in 2019 whose traffic is now shaped by the Elizabeth line. Flagged `ELCorridor` in StationMap; explorable, but excluded from London figures and the ranking.

Abbey Wood; Acton Main Line; Bond Street; Brentwood; Chadwell Heath; Ealing Broadway; Farringdon; Forest Gate; Gidea Park; Goodmayes; Hanwell; Harold Wood; Hayes & Harlington; Heathrow T2&3 TfL Rail/HEx; Heathrow T4 TfL Rail/HEx; Heathrow T5 TfL Rail/HEx; Ilford; Liverpool Street; Manor Park; Maryland; Paddington; Romford; Seven Kings; Shenfield; Southall; Stratford; Tottenham Court Road; West Drayton; West Ealing; Whitechapel

Elizabeth line stations outside the cohort entirely (no 2019 baseline): Reading; Twyford; Maidenhead; Taplow; Burnham Bucks; Slough; Langley Berks; Iver; Paddington EL; Canary Wharf EL / Canary Wharf Elizabeth Line; Custom House EL / Custom House Elizabeth Line; Woolwich EL / Woolwich Elizabeth Line.
