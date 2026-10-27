# Lead Magnet Dashboard Table Logic

2026-10-27 · Product Team

Every table on the IK Lead Magnet Funnel dashboard, with the BigQuery source it reads and the exact logic behind its numbers, so anyone can check a figure.

## Sources

All tables live in BigQuery project `ik-marketing-data`; the dashboard only reads them.

The tables below use short names for these nine sources. Each one is described after the key: what it is, what one row holds, why it was picked, and where the dashboard uses it.

| Short name | Stands for | BigQuery table |
|---|---|---|
| CS | Clickstream, lead-magnet view | `Marketing_data_new_logic.lead_magnet_clickstream_master` |
| RAW | Raw click log | `Marketing_data_new_logic.all_clickstream_data` |
| VISITS | Page visit log | `Marketing_data_new_logic.Leads_Click_history` (steps `s0`, `s1`) |
| FORMS | Webinar registration forms | `Marketing_data_new_logic.Leads_Click_history` (step `s4`) |
| LEADS | Lead magnet leads | `Tech_ML_Projects.Resume Analysis - GQL data` |
| RUNS | Analysis runs | `Tech_ML_Projects.Resume Analysis` |
| HS | HubSpot contacts | `Sales_Dashboard.contacts` |
| FIN | Finance sales | `Marketing_data_new_logic.Finance_Net_Revenue` |
| ZOOM | Zoom webinar attendance | `Webinar_analytics.webinar_attendee_data_from_zoom` |

### CS: Clickstream, lead-magnet view

- **What it is:** a view the marketing data team built on top of the raw click log, for the lead-magnet pages.

- **One row is:** one click or page event in a visit.

- **Data it holds:** browser ID, visit (session) ID, time, the click name (`ClickID`), page URL, landing page, device, country, channel and UTM source, HubSpot ID when known, tool name and business unit.

- **Why we use it:** it already groups clicks into visits and adds device, country and channel, and it marks staging traffic. That makes it the ready-made source for a step-by-step page funnel.

- **Limit:** it only covers the Resume Analyzer, Salary Analyzer and AI Quotient pages.

- **Used in:** Overview, Compare tools, Experience, and the Conversions KPIs.

### RAW: Raw click log

- **What it is:** every tracked click on every IK web page.

- **One row is:** one click.

- **Data it holds:** browser ID, click name, page URL, time, device fingerprint. No names or emails.

- **Why we use it:** it is the only source for the ATS Resume Checker, FDE Gap Analyzer and AI Mock Interview pages, and for buttons the CS view leaves out (Book my seat, Unlock 90-day path, Share, Copy link and others).

- **Limit:** clicks only. Someone who opens a page and clicks nothing isn't in it.

- **Used in:** Tool funnels and Conversions (registration types, buttons, registered without using the tool).

### VISITS: Page visit log

- **What it is:** IK's page-view tracking, steps `s0` and `s1` of `Leads_Click_history`.

- **One row is:** one page view.

- **Data it holds:** browser ID, page URL, time, device, country, UTM source and medium.

- **Why we use it:** RAW only has clicks, so people who open a tool page without clicking would be missed. VISITS gives the true count of who opened each tool, plus device and source for the who-registered breakdowns.

- **Used in:** Tool funnels (Opened the tool, reached results), Conversions (device and source of registrants).

### FORMS: Webinar registration forms

- **What it is:** webinar registration form submissions, step `s4` of the same `Leads_Click_history` table.

- **One row is:** one submitted registration form.

- **Data it holds:** browser ID, time, page and CTA page, webinar type, and the email (hashed before it leaves BigQuery).

- **Why we use it:** it is the only place a page visitor's email appears. The email links page visitors to HubSpot profiles, sales and Zoom attendance. That is the only way to measure registrations, attendance and sales for ATS Resume Checker and AI Mock Interview, which create no lead records.

- **Used in:** Tool funnels, Conversions (ranking for ATS and Mock, who registered, attendance).

### LEADS: Lead magnet leads

- **What it is:** the backend's own record of people who used a tool and gave their details.

- **One row is:** one submission of contact details on one tool.

- **Data it holds:** email (hashed), HubSpot ID, tool code (`lead_magnet`), time, experience, job area, country, UTM source and medium.

- **Why we use it:** it is the most complete count of real tool users with contact details, and its HubSpot ID is the link to registrations, IK learner status and sales. It also holds the experience and job area answers used for personas.

- **Limit:** only Resume Analyzer, Salary Analyzer, AI Quotient and FDE Gap Analyzer write here. Records start on 18 Apr 2026 in the dashboard's data.

- **Used in:** Pre vs post LM, Impact, Personas & ranking, Conversions ranking, Tool funnels (tools used before, path to registration).

### RUNS: Analysis runs

- **What it is:** the backend log of every analysis the tools ran.

- **One row is:** one analysis.

- **Data it holds:** tool code, time, `response_time_in_seconds`, device type, email (hashed), and the resume (only a fingerprint of it is taken, never the text).

- **Why we use it:** it is the only source of how long the system takes, and of repeat use. The resume fingerprint tells whether a returning user re-ran the same resume or a different one.

- **Used in:** Overview (processing time), Experience, Tool funnels (time, repeat use), Personas (active days).

### HS: HubSpot contacts

- **What it is:** IK's CRM, copied into BigQuery every day.

- **One row is:** one contact on one day (a daily snapshot).

- **Data it holds:** contact ID, creation date, latest webinar registration date (`latest_gql_date`), IK alum flag (`is_candidate_alum_final`), test-lead flag, experience, job role, country.

- **Why we use it:** HubSpot is the system of record for webinar registrations and IK learner status. Because it keeps a copy every day, every registration date a person ever had can be rebuilt; the first-registration field (`gql_date`) is empty for most contacts. The creation date tells whether a person was new to IK when they used the tool.

- **Used in:** every registration figure, IK learner splits, new vs known, Pre vs post LM groups.

### FIN: Finance sales

- **What it is:** finance's table of program sales and net revenue.

- **One row is:** one sale.

- **Data it holds:** sale date, program name, net revenue, HubSpot ID, email (hashed).

- **Why we use it:** it is current (updated daily) and is finance's figure for revenue. The older `BQ_Sale` table stops in April 2026, and `Hubspot_db.Sales_automated_tech` has no deposits from May to August 2026, so neither was used.

- **Used in:** every sales and revenue figure, and the share of all IK sales.

### ZOOM: Zoom webinar attendance

- **What it is:** attendance records pulled from Zoom for every IK webinar.

- **One row is:** one join by one person (someone who rejoins has several rows).

- **Data it holds:** webinar ID, topic, start time, join and leave times, time in the webinar (seconds), email (hashed), HubSpot ID.

- **Why we use it:** it shows who actually showed up. HubSpot's own attendance field is false for every lead, so it can't be used. Time is added up across rejoins to count who stayed 30+ minutes.

- **Limit:** starts on 23 Apr 2026; someone who joins with a different email than they registered with isn't matched.

- **Used in:** Conversions (did registrants attend), Tool funnels (attended, stayed 30+ minutes).

## Rules that apply everywhere

- **Tools:** only the six career tools count. Backend codes map as `IKIQ` = Resume Analyzer, `SALARY_INSIGHT` = Salary Analyzer, `RESUME_ANALYSIS_SU` = AI Quotient, `FDE_READINESS` = FDE Gap Analyzer. ATS Resume Checker and AI Mock Interview exist only in RAW, VISITS and FORMS.

- **Production pages only:** staging, localhost and proxy mirrors are removed.

- **Test leads removed:** emails containing "test", and HubSpot test leads.

- **A person** = a unique email per tool.

- **Lead date** = the first time they left details on that tool in the period.

- **Registered after the tool** = a HubSpot registration date on or after the lead date. Registration dates are every distinct `latest_gql_date` across the daily HS snapshots.

- **Bought after the tool** = a FIN sale on or after the lead date, matched by HubSpot ID or email.

- **Attended** = joined a ZOOM webinar that started on or after the registration date. Time is summed over rejoins.

- **Period** = the last N days ending on the data date. "Vs prior" compares with the N days before that.

## The six career tools

| Tool | What it does for the user | Where it lives |
|---|---|---|
| Resume Analyzer | Scores a resume against a target role and gives a report with fixes and an interview guide | `ikiq.interviewkickstart.com/resume-analysis` |
| Salary Analyzer | Estimates the user's market value from their resume and shows the salary gap | `interviewkickstart.com/ai-salary-analyzer` |
| AI Quotient | Scores how ready the user's profile is for AI roles | `interviewkickstart.com/ai-quotient-analyzer` |
| FDE Gap Analyzer | Scores readiness for Forward Deployed Engineer roles and offers a 90-day path | `dexity.com/career-tools/fde-readiness-analyser` |
| ATS Resume Checker | Checks whether a resume passes applicant tracking systems | `interviewkickstart.com/ats-resume-checker` |
| AI Mock Interview | Lets the user set up and take an AI mock interview | `interviewkickstart.com/ai-mockinterview` |

## Overview

| Table | What it means | Source | Logic |
|---|---|---|---|
| Opened the tool | How many people landed on a tool page | CS | Unique people with any event in a tool session |
| Uploaded a resume | How many of them uploaded a resume | CS | Upload click, or any later step |
| Leads captured | How many people gave us their contact details | LEADS | Unique emails per tool |
| Registered for webinar | How many of those leads signed up for a webinar afterwards | LEADS + HS | Leads with a registration on or after their lead date |
| Registered without using the tool | People who signed up for a webinar straight from the tool page, without using the tool | CS | A registration click before any upload in that visit |
| Median processing time | How long the system typically takes to produce a result | RUNS | Median `response_time_in_seconds`, sandbox runs removed |
| On-page funnel | Where people drop off between landing on the page and registering | CS | Each person counted at their furthest step: opened, uploaded, submitted, saw report, clicked register, registered. A later step implies the earlier ones |
| Leads per day | Daily trend of new leads per tool | LEADS | Unique emails per day per tool |

## Pre vs post LM

| Table | What it means | Source | Logic |
|---|---|---|---|
| Groups | Whether each user had signed up for a webinar before using the tool, signed up after, or never | LEADS + HS | Registered before = any registration before the lead date. First registered after = none before, one on or after. Not registered = none at all. One group per person, checked in that order |
| …registered again after | People who had signed up before and signed up again after using the tool | LEADS + HS | People in "Registered before" who also registered on or after the lead date |
| Sales, revenue, bought before | How many bought a program after the tool, the revenue, and how many were already customers | LEADS + FIN | Sales on or after the lead date vs before it; median days from lead to sale |
| Sales conversion by group | Which of the three groups buys most often | LEADS + HS + FIN | Buyers ÷ people in each group |
| Who each tool reaches | Whether a tool mostly reaches people already in our funnel or new ones | LEADS + HS | Each tool's leads split across the three groups |
| By tool | All of the above, one row per tool | LEADS + HS + FIN | Same figures per tool. With Tool = All, a person counts once, at their first tool |

## Compare tools

| Table | What it means | Source | Logic |
|---|---|---|---|
| Every tool on the same steps | Which tool loses the most people at each step, side by side | CS | Each funnel step ÷ the step before, per tool |
| People at each step | The raw counts behind the step rates | CS | Count at each funnel step |
| Backend funnel | Of people who gave details, how many got a report and how many signed up for a webinar | LEADS + RUNS + HS | Leads → got a report (the lead's `uuid` appears in RUNS) → registered |

## Experience

| Table | What it means | Source | Logic |
|---|---|---|---|
| How long the system takes | How fast each tool returns a result, typically and at its slowest | RUNS, CS | Typical = median, slowest 5% = 95th percentile of processing time. Felt wait = last submit click to the first report event |
| Time on report | How long people spend reading their result | CS | Minutes from the first report event to the last tracked event in that visit |
| Repeat use | How many people use a tool more than once, and whether with the same resume | RUNS | Runs per person; one run = one resume within 10 minutes. Same resume = only one resume hash among their runs |
| Where we lose people | The steps where the most people drop out | CS | People lost between consecutive funnel steps |

## Conversions

| Table | What it means | Source | Logic |
|---|---|---|---|
| KPIs | Headline count of webinar sign-ups that followed a tool | LEADS + HS, CS | Registered after a tool, tracked results-page registrations, direct registrations, lead-to-registration rate |
| Conversions by tool | All six tools ranked by how many purchases and sign-ups they led to | Best source per tool | Resume, Salary, AI Quotient and FDE use LEADS + HS + FIN. ATS and Mock use VISITS + FORMS + FIN, with sales traced through the registration email. Sorted by purchases, then registrations |
| Did registrants attend? | Of people who signed up for a webinar, how many actually showed up, and whether showing up led to buying | The above + ZOOM | Attended and stayed 30+ minutes ÷ registered; buy rate for attendees vs registrants who didn't attend; most-attended webinars |
| How people registered | When the sign-up happened: before using the tool, on the results page, or elsewhere after using it | RAW | Each registration click on a tool page, classed as without using the tool (before any upload or start), on the results page, or after using the tool elsewhere. Attended from ZOOM |
| Which button earns it | Which button people clicked to sign up | RAW | The last registration button clicked before the registration |
| Registered without using the tool | People who signed up straight from the tool page: where they came from and who they are | RAW + FORMS + HS + VISITS | Where from: page, button and webinar type. Who: HubSpot profile via the hashed form email (IK learner, new vs known, experience, job role, country), plus device and source from the first visit |
| Registration rate by user type | Whether IK learners or non-IK users sign up more often | LEADS + HS | IK learner = HS `is_candidate_alum_final` |

## Impact

| Table | What it means | Source | Logic |
|---|---|---|---|
| Share of IK sales | What part of all IK sales came from people who had used a career tool first | FIN + LEADS | Sales in the period where the buyer had a lead on or before the sale date. Period filter only |
| What each tool is worth | Each tool's value: new people, sign-ups, purchases and revenue per user | LEADS + HS + FIN | New to IK, registered, bought, revenue, revenue per person, typical days to buy |
| New vs known | Whether the tools find new people or mostly reach people we already know | LEADS + HS | New = HubSpot contact created on or after the lead time (within 1 hour); otherwise already known |
| More tools, more sales? | Whether people who try several tools buy more | LEADS + RUNS | People grouped by how many distinct tools they have ever used |
| Monthly cohorts | Whether newer months convert better than older ones, compared over the same time window | LEADS + HS + FIN | Each lead month's registrations and sales within 30, 60 and 90 days; blank until that window has passed |
| How fast | How soon after using a tool people sign up and buy | LEADS + HS + FIN | Days from lead to first registration and to first sale |
| Who converts | Which kinds of users (experience, job area, geo and so on) sign up and buy most | LEADS | Rates by experience, job area, geo, source, device and IK learner |

## Personas & ranking

| Table | What it means | Source | Logic |
|---|---|---|---|
| Ranking | Tools ordered from most to fewest webinar sign-ups, with the sales each led to | LEADS + HS + FIN | Tools sorted by registrations, then sales. Sales share = the tool's sales ÷ all sales after a tool |
| How to judge each tool | What each tool is for and the one number that says whether it works | Definitions + current values | Each tool's job, north-star metric, supporting metrics, guardrails, and today's rates |
| Personas | The typical users of each tool and which groups sign up most | LEADS (+ HS, FIN) | Career stage = experience band, used instead of age: 0–4, 5–10, 9–20, 20+ years. Profession = job area. Most common stage × job area × IK combinations, with their rates |
| Do they come back? | Whether users return on other days, and which other tools they use when they do | LEADS + RUNS | Distinct active days on any tool: one day, 2–3 days, 4+ days; and which other tools returning users used |
| Frequent users | Who our most engaged users are and how many sign up and buy | LEADS + RUNS + HS + FIN | People active on 4+ days: their profile and how many registered and bought |

## Tool funnels

Pick a tool at the top of the tab. This tab uses the Period filter only.

| Table | What it means | Source | Logic |
|---|---|---|---|
| Funnel | Step by step, how many people get through the chosen tool | RAW + VISITS | Opened = visited or clicked the tool page; uploaded; clicked analyze; reached results; dropped off = uploaded but never reached results |
| Who | How many users are IK learners vs not | RAW toggle, or LEADS + HS | ATS: IK learner = ticked "I'm an IK learner". Mock: IK learner = clicked Sign in. Others: HubSpot IK flag |
| Time | How long the system takes, how long users wait, and how long they read the result | RUNS, RAW + VISITS | System time from RUNS. Felt wait = submit to results. Time on results needs 2+ tracked moments |
| Repeat use | Whether people come back to use the tool again, and how often | RAW | Analyze or start clicks at least a minute apart; most uses by one person; span between first and last use |
| Registrations | Every way people signed up for a webinar from this tool, and how many attended | RAW + FORMS + ZOOM | First-page register clicks, registered without using, registered on results, registration forms after the first visit, attended, stayed 30+ minutes |
| Conversions | How many people bought a program after the tool | FORMS + FIN | Bought after visiting (traced via the form email), and bought among attendees |
| Buttons | How many people clicked each button, and how many of them signed up afterwards | RAW + FORMS | People who clicked each listed click ID, and how many of them registered afterwards |
| Programs bought | Which programs people bought after the tool | FORMS + FIN | Program names bought after the tool |
| Tools used before | Which other tools people had already used before this one | LEADS | For each tool, earlier tools used before its first use, named, with counts |
| Tools used before registering | Which tools people had used by the time they signed up | LEADS + HS | Other tools used on or before the registration date |
| Path to registration | The order of tools people used before signing up | LEADS + HS | Each registrant's tools in order of first use up to the registration date, grouped by path |
| How often used again | How many people reuse each tool, with the same or a different resume | RUNS | Repeat users, same vs different resume, most resumes by one person, span |

The code behind every table is in `~/lead-magnet-dashboard/lm/`: `metrics.py`, `funnels.py`, `impact.py`, `personas.py`, `attendance.py` and `views.py`. Ready-to-run validation SQL is in the dashboard's Data notes tab.
