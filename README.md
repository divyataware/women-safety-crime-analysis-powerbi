# Women Safety Crime Analysis: India (Power BI)

An end-to-end Power BI project on recorded crimes against women in India, 2014 to 2020. I took messy government data, cleaned it, built a data model, wrote the DAX, and turned it into an interactive report that answers 15 specific business questions.

> **Please read this first.** This project uses *recorded* cases. A rise in recorded cases can mean more crime, but it can also mean better reporting. Nothing here measures actual crime.

---

## Screenshots

| Overview | State analysis | Insights |
|---|---|---|
| ![Overview](screenshots/01_overview.png) | ![State analysis](screenshots/02_state_analysis.png) | ![Insights](screenshots/03_insights.png) |

**Data model**

![Data model](screenshots/04_data_model.png)

---

## Why I built this

I wanted to practise the full analyst workflow on data that is hard to work with: five separate files, text in number columns, total rows mixed with real rows, and different spellings of the same state. I also wanted a topic where getting the numbers wrong has real consequences, because that forces you to check everything twice.

I started from business questions, not charts. The full list is in [`docs/business_questions_and_findings.md`](docs/business_questions_and_findings.md).

---

## The data

- **Source:** [add the real source name and link here]
- **Period:** 2014 to 2020
- **Files used:** missing women, traced/rescued cases, acid attacks, rape cases (by age group), and other crimes (murder, kidnapping & abduction, gang robbery and more)
- **Size:** about 1.7 million recorded cases in total
- **Licence:** [add the licence or terms of use, or note that raw files are not included]

---

## What I did

1. **Cleaned the data in Power Query.** Turned "NR" and "Null" text into real blanks, removed total rows and duplicate rows, fixed spellings, and set correct data types.
2. **Combined the crime tables into one main fact table**, `Fact_Crime`, with State/UT, Years, Case Type and Cases Reported.
3. **Kept two tables separate on purpose.** Rape-by-age data sits in `Fact_Rape_Cases`, and traced cases sit in `Rescued case`. They are a different level of detail, and merging them caused double counting in my first attempt.
4. **Built a multi-fact star schema.** `Dim_State` and `Dim_year` are shared by all three fact tables. Every relationship is one-to-many with a single filter direction, and no fact table connects to another.
5. **Wrote the DAX measures** (in [`docs/measures.dax`](docs/measures.dax)).
6. **Designed a 3-page interactive report** with slicers, page navigation, a state matrix, a map, an age chart, and an insights page.

### Data model

```
Dim_State ─┬─> Fact_Crime        (State/UT, Years, Case Type, Cases Reported)
Dim_year  ─┼─> Fact_Rape_Cases   (State/UT, Years, Age Category, Cases Reported)
           └─> Rescued case      (State/UT, Years, Total traced)
```

### Core measures

```dax
Total Cases = SUM ( Fact_Crime[Cases Reported] )

Missing =
CALCULATE ( SUM ( Fact_Crime[Cases Reported] ), Fact_Crime[Case Type] = "Missing" )

Recovery Rate =
DIVIDE ( SUM ( 'Rescued case'[Total] ), [Missing] )
```

---

## What the data shows

These figures reconcile to the report totals (Missing + Rape + Acid Attack + Other = Total Cases).

| Measure | Value |
|---|---|
| Total reported cases | about 1.71 million |
| Missing women | 479K (28.0%) |
| Kidnapping & abduction | 393K (22.9%) |
| Rape | 227K (13.2%) |
| Murder | about 220K (12.8%) |
| Acid attacks | 1,244 |
| Tracing rate (traced ÷ missing) | 66% |

**Key findings**

- **Missing women and kidnapping & abduction together make up about half of all reported cases.** That is the clearest pattern in the data.
- **Rape accounts for about 13% of reported cases**, close to murder. Acid attacks are rare by comparison.
- **Within rape victims, the 18 to 30 age group is the largest**, followed by 30 to 45. The youngest and oldest groups are the smallest.
- **About two in three missing women were traced**, which leaves roughly one in three unresolved.
- **Reported cases rise to a peak in 2018**, fall in 2019, and recover partly in 2020.

---

## Business questions

I wrote 15 questions across six areas and tested which ones the data could actually answer.

| Area | Answered | Still open |
|---|---|---|
| Missing women | Total, tracing rate | Year-by-year and state-level detail (needs source check) |
| Crime categories | Total and shares | Category trend over time |
| Rape and age | Distribution by age group | Age change over time |
| States and population | Cases by state (preliminary) | Population-adjusted rates |
| Case outcomes | None | Investigated, chargesheeted, pending (not in the data) |
| Data quality | Fully documented | None |

The full answers, evidence and reliability ratings are in [`docs/business_questions_and_findings.md`](docs/business_questions_and_findings.md).

---

## What went wrong, and how I fixed it

I'm including this because it is the most useful part of the project.

- **My first totals were several times too high.** A "Total Victims" row sat inside the age categories, and a state total was repeated on every age row, so each rape case was counted about nine times. I found it by checking one state against the source file, then rebuilt the data so each state and year appears once.
- **Text in number columns.** "NR" and "Null" blocked sums and type conversion. I converted them to blanks before changing types.
- **A model that looked right but filtered wrong.** I began with five connected fact tables and a category table whose values did not match the data, which blanked the whole page. Moving to shared dimensions and slicing by `Case Type` fixed it.
- **Appending created duplicates.** Combining year sheets repeated rows, so I removed duplicates after every append.
- **Formatting traps.** A count measure showed as a percentage on a chart axis, and a rate showed as a raw count. I corrected the formats and checked every card against the others.

---

## Known issues (work in progress)

I would rather list these than hide them.

- **Year-by-year missing counts look wrong.** They swing from about 176K in 2018 to under 3K in 2019 and back to about 177K in 2020. That pattern points to a data structure problem in the source sheets, not a real trend. I'm checking it.
- **State-level tracing rates are unreliable.** Some states show a tracing rate above 100%, and some small territories show implausibly high missing counts. The Missing and Traced tables may not be aligned by year or state name.
- **The Top 5 state ranking is not verified.** The five bars add up to only a fraction of the total, so I'm treating the state chart as preliminary until it reconciles.
- **No population data**, so I cannot yet compare states fairly.
- **No case-outcome data** (investigated, chargesheeted, disposed, pending) in the files I have.

---

## Limitations

- Recorded cases only. Reporting practices differ between states and years.
- Raw counts favour large states. A rate per 100,000 people would be fairer.
- Age data counts victims and the headline numbers count cases, so the two totals can differ slightly.
- The tracing rate is a ratio of two counts. Traced cases in a year can include women reported missing earlier, so a rate can exceed 100%.

---

## What I would do next

1. Fix the year and state issues above, then restate the affected findings.
2. Add population data and a rate per 100,000.
3. Add case-category and age-group trends over time.
4. Look for outcome data to answer the case-disposal questions.

---

## How to open the project

1. Install Power BI Desktop (free).
2. Open `powerbi/crime.pbix`.
3. If Power BI asks for the data location, point it to the files in `data/raw/`.

---

## Repository structure

```
├── README.md
├── data/
│   ├── raw/                 original files as downloaded
│   └── cleaned/             optional cleaned export
├── powerbi/
│   └── crime.pbix
├── screenshots/
└── docs/
    ├── business_questions_and_findings.md
    ├── measures.dax
    └── data_cleaning_log.md
```

---

## Skills shown

Power BI · Power Query (M) · DAX · Data cleaning · Data modelling (star schema) · Data validation and reconciliation · Business-question framing · Documenting limitations

---

## About me

[Your name] · [LinkedIn link] · [Email]

I'm happy to hear feedback, especially on the data model and the limitations section.
