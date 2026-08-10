# Challenge 4—Adaptive Evacuation Readiness & Vulnerable Community Planning

**Agents for Impact 2026 · Required differentiator: Grounding with Google Maps**

---

## 1. Why this one matters

When a hurricane is forecast, most people leave. They put the dog in the car, they drive to a
relative's house two hundred miles inland, and they come back in a week.

The people who die are the ones for whom none of that is available. No car. No relative inland. An
oxygen concentrator that needs mains power. A third-floor apartment and a hip replacement. A
daughter who calls every day but lives in Ohio.

After Hurricane Ian, the median age of the dead was in the seventies. That is not a coincidence and
it is not bad luck. It is what happens when a plan quietly assumes a level of mobility that a large
minority of the population does not have.

Here is the part that should bother you, and the part that makes this a data problem rather than a
tragedy: **those people are findable in advance.** Not by name, and not by knocking on doors in a
storm, but by census tract, by facility, by county, months ahead, from public federal data that
anybody can download today.

So: nobody is being rescued in real time in this challenge. You are building what a county emergency
manager needs in **August**, so that in **September** the plan already exists.

And you are going to find something uncomfortable while you do it. We will not spoil it here—it
is in Section 9, and it is worth arriving at honestly.

---

## 2. How to read this

This card is long. It is a **reference for your whole afternoon, not something to read end to end
right now**. Find your lane, read the two or three sections that are yours, and come back for the
rest when you hit it.

| You are on… | Read now | Come back for |
|---|---|---|
| **Data** | [Pick your state](#3-first-pick-your-state) · [Getting started](#8-getting-started) · [The data](#10-the-data-and-why-we-chose-it) | [Questions that need more](#5-questions-that-need-more-than-we-gave-you) · [Bring your own data](#11-bringing-your-own-data) |
| **Agent** | [The technology](#7-the-technology-youll-use) · [What you're building](#4-what-youre-building) | [What will set yours apart](#6-what-will-set-yours-apart) |
| **Front end** | [The technology](#7-the-technology-youll-use)—especially the attribution rules, they shape your UI | [What you'll have](#9-what-the-data-actually-says) |
| **Story** | [Why this matters](#1-why-this-one-matters) · [What the data says](#9-what-the-data-actually-says) | [How you'll be judged](#13-how-youll-be-judged) |

### Three things everyone needs to know by the end of hour one

**1. Grounding with Google Maps cannot answer the question this challenge is about—at least not
the way you will first ask it.** Ask which shelters in a county take a wheelchair and it cannot
help. Section 2 of the notebook shows that in about ninety seconds, and the whole design follows
from it. There is a way to get more out of it than that, and finding it is worth real credit.

**2. Use a Gemini 3.x model.** On 2.5 and older, a built-in tool cannot share an agent with a
function tool of your own—the request is rejected outright, and working around it costs half an
hour. On 3.x it simply works. [Section 7](#7-the-technology-youll-use) has both architectures.

**3. For most shelters, the accessibility field is blank—and blank does not mean "no".** It means
nobody wrote it down. An agent that conflates those two is confidently wrong about the thing that
matters most, and a judge will find it.

### A four-word glossary, before you meet these as column names

- **Census tract**—a Census Bureau area of roughly 4,000 people. Small enough to act on, large
  enough that nobody in it is identifiable. Our unit for everything about people.
- **Grounding**—connecting a model to a live external source so its answer is anchored in
  something real rather than recalled. Your differentiator is one of these.
- **Unrecorded**—a field nobody filled in. Distinct from a field that says "no", and the
  distinction carries most of the meaning in this challenge.
- **EVAC / POST**—a shelter designated for use *before* landfall versus one for *after* impact.
  Different buildings, different job, and the federal file tells you which.

---

## 3. First: pick your state

Evacuation is run by states and counties, not by metro areas, so this challenge is scoped to a
state. Change one line at the top of the notebook and everything follows.

**You do not have to pick the state you are sitting in.** Pick the one that makes your story.

These numbers come from an actual run, so you are choosing with your eyes open:

| State | Shelters | Total capacity | Wheelchair recorded | Medical-needs | Notes |
|---|---:|---:|---:|---:|---|
| **FL** | 2,793 | 848,784 | 247 | **155** | The default. Right hazard, second-most medical shelters in the country, and the worst accessibility reporting of any coastal state |
| TX | 2,651 | 772,337 | 526 | 31 | Biggest coastal exposure after Florida |
| GA | 1,573 | 1,138,056 | 375 | 14 | Huge capacity, inland receiving state as well as coastal |
| NC | 1,207 | 445,627 | 288 | 16 | Inland flooding as much as surge |
| LA | 1,006 | 355,010 | 125 | 14 | The evacuation-planning literature is mostly about here |
| SC | 604 | 328,746 | 159 | **1** | **Cautionary.** One medical-needs shelter in the whole state |
| NY | 3,724 | 1,639,805 | 921 | 10 | Surge risk with almost no medical designation |
| CA | 5,648 | 6,554,178 | 1,105 | 204 | Wildfire rather than hurricane. Works, different story |

Any state works, including ones not listed—you just have not seen their numbers, and neither have
we. Run the notebook and check Section 9 before you build a narrative on one.

**South Carolina is the worked example of getting this wrong.** 604 shelters, 328,746 spaces, and
exactly one with a medical designation. If your entire demo is medical-needs matching, SC will not
carry it, and you will find that out at minute 200.

### One afternoon, one person, one deadline

Everything below is easier to think about if you have somebody specific in mind.

**Dana is the emergency management planner for a Gulf Coast county.** It is mid-August. The seasonal
forecast was revised upward last week, and she has been asked for a readiness brief by the end of the
month. She has a shelter list from the state, a spreadsheet of nursing homes, and a strong suspicion
that the plan on file assumes everybody can drive.

She is not in an emergency. She has three weeks. That is exactly the point—**everything that can
be decided in advance should be**, because the alternative is deciding it at 3am with the power out.

---

## 4. What you're building

**Not a lookup with a chat box. A capability.**

An agent that helps Dana answer the questions she actually has, using data she does not have time to
assemble, and telling her plainly which of its answers are solid and which are guesses.

Here is what she would ask over one afternoon, and what answers each:

| Dana asks | What answers it |
|---|---|
| *"Which neighbourhoods in my county would struggle most to self-evacuate?"* | `vulnerability_tracts`—no-vehicle households, over-65 share, disability prevalence, mobile homes |
| *"How many people here depend on electricity for medical equipment?"* | `power_dependent_counties`—and note it is county-level, so you can say how many but not which block |
| *"Where would I send them, and how far is it?"* | `shelters` plus `vulnerability_tracts`—a spatial query, not a lookup |
| *"Which of those shelters can actually take a wheelchair?"* | `shelters.wheelchair_accessible`—**and the honest answer is mostly "nobody recorded it"** |
| *"Is that shelter still there? Is it open? What is it now?"* | **Grounding with Google Maps.** Nothing in our data can tell you this |
| *"Which of my shelters is itself in the surge zone?"* | `shelters.in_surge_slosh_area` and `hazard_tracts` |
| *"What should I fix before the season?"* | Your agent's judgment. This is the deliverable |

Notice what that table is telling you. **Some of those questions need the differentiator and some
of them cannot use it at all.** Working out which is which *is* the design problem—an agent that
calls Maps grounding for everything gives confident nonsense about accessibility, and one that never
calls it is quoting a federal record that may describe a building demolished in 2023.

### The deliverable: a Readiness Brief

Three segments. Show them in a demo, not a document.

**Segment 1—Who cannot leave.** The tracts and facilities in Dana's county holding people who
cannot self-evacuate, ranked, with the reason for each. Not a heat map. A short list she could act
on, with numbers attached.

**Segment 2—Where they would go, and whether that holds up.** For each candidate destination:
capacity, what the record says about accessibility and generators, whether it sits in a surge zone
—and **verified live** against Google Maps that it exists, is what it claims to be, and is open.

**Segment 3—What to fix before the season.** The gaps. Zones with no accessible destination
within a reasonable distance. Facilities with no generator near a power-dependent population.
Shelters that are themselves in the flood zone. **This is the segment that makes the agent worth
building**, and it is the one nobody will have time to do well, which is exactly why it separates
teams.

---

## 5. Questions that need more than we gave you

Say these out loud in your demo. Naming a limitation you cannot fix reads as competence; having a
judge find it reads as the opposite.

| Dana asks | Why we cannot answer it | What would close the gap |
|---|---|---|
| *"How long does it take to drive there?"* | Routing through Grounding with Google Maps is Private Preview. We have straight-line distance only | Maps Grounding Lite's `compute_routes` via its MCP server—needs an API key, see [bonus](#6-what-will-set-yours-apart) |
| *"Which assisted living facilities are in my county?"* | **No national public list exists.** Assisted living is state-regulated, so there is no federal roster—a real gap affecting exactly this population | Your state's licensing database, if it publishes one |
| *"Where are the mobile home parks?"* | The national layer was decommissioned in 2025 | ACS mobile-home counts per tract, which we do load—density, not locations |
| *"Which specific people need help?"* | Individual-level data is prohibited at this event, and rightly | Nothing. This is a line, not an obstacle |
| *"Is this shelter open right now, during the storm?"* | FEMA's own file says in capitals that it must not be used for operational status | Live county feeds—which are empty in October, when you are planning |
| *"How many people in this tract use a wheelchair?"* | CDC PLACES gives modelled prevalence, not counts | Nothing better exists publicly. Use the prevalence and say it is modelled |

---

## 6. What will set yours apart

Every team in this room starts from the identical five tables. Nobody wins on the data. Here is
where the distance actually opens up.

**Treat "unrecorded" as its own category, visibly.** The single highest-value thing you can do, and
most teams will not. Your agent should be able to say *"eleven of these I can speak to; sixty-four
nobody has assessed"* rather than silently filtering to the eleven.

**Say which claims are fresh and which are historical.** A shelter record from a 2022 hurricane and
a Maps lookup from nine seconds ago are different kinds of fact. An agent that labels them is more
useful than one that blends them into confident prose.

**Audit your own output.** Produce your ranked list, then join it back to the demographics you were
forbidden to use as an input and ask whether the recommendation lands disproportionately on any
group. Report what you find, including if the answer is "no". See [Section 10](#10-the-data-and-why-we-chose-it).

**Make distance mean something.** Twelve kilometres is not far. It is impassable for a household
with no vehicle, and merely inconvenient for one with two. An agent that knows the difference is
doing the actual job.

**Bonus—get real travel times back.** Maps Grounding Lite is a Google-hosted MCP server at
`https://mapstools.googleapis.com/mcp` with a `compute_routes` tool that returns distance and
duration. It needs an API key, and the [Maps Demo Key](https://developers.google.com/maps/demo-key)
covers it with no billing account and no credit card. It also satisfies your Google-managed MCP
requirement. Optional, and it upgrades Segment 3 considerably.

**Bonus—add the elderly-living-alone layer.** ACS table `B11007_003E` is exactly
*"households with one or more people 65 years and over: 1-person household"*, and it is the best
single proxy for "nobody will notice if they don't leave". It needs a
[free Census API key](https://api.census.gov/data/key_signup.html), which is why it is a bonus
rather than core.

---

## 7. The technology you'll use

### The common stack, same for every challenge

| Component | What it is for |
|---|---|
| **BigQuery** | Where your five tables live. Free tier covers everything here |
| **ADK** (Agent Development Kit) | The agent framework. Python |
| **Gemini** | The model. Use the **3.x line**—`gemini-3.6-flash` is verified working here |
| **A Google-managed MCP server** | Consume one; do not author your own. BigQuery's built-in server, MCP Toolbox, or Maps Grounding Lite |
| **Deployed to Google Cloud** | Agent Runtime or Cloud Run, your choice. Deploy at roughly the halfway mark so the front-end lane has something to build against |
| **Antigravity CLI (`agy`)** | Pre-installed in Cloud Shell. Zero setup. Encouraged |

### Your differentiator: Grounding with Google Maps

**Why it belongs in this problem**, rather than being bolted on:

FEMA's shelter file is a **historical registry**. Every facility ever registered as a shelter, one
row each, some entered during a hurricane years ago. FEMA's own layer description says in capital
letters that it must not be used to determine whether a facility is operational.

We measured how stale it actually is, so you do not have to guess: **we took fifteen Florida
shelters at random and asked Google Maps about each. Fourteen still exist. One had been renamed**—
First Presbyterian Church of Maitland is now Maitland Presbyterian Church, same building, different
name, and a lookup matching on name alone would have missed it.

So the honest case for the differentiator is not "the file is full of demolished buildings". It is
that **a static file cannot tell you which ones moved, renamed or closed, and it cannot tell you
what a place is *now*.** Note what the sample is made of, too: mostly public schools, which persist.
A registry of churches and community centres would age faster.

Our data is the memory. Maps is the eyes. Neither does the other's job.

**Setup is one API.** `aiplatform.googleapis.com` plus billing. No Maps Platform key, no separate
Maps Grounding API, no allowlist form.

```python
from google.adk.tools import google_maps_grounding
```

with these environment variables:

```
GOOGLE_CLOUD_PROJECT      = <your project>
GOOGLE_CLOUD_LOCATION     = global
GOOGLE_GENAI_USE_VERTEXAI = True
```

### Your model choice decides your architecture—read this before you write code

**On Gemini 2.5 and older**, a built-in tool cannot sit in the same agent as a function tool of
your own. The request is rejected:

```
400 INVALID_ARGUMENT—"Unable to submit request because Multiple tools are supported
only when they are all search tools."
```

**On Gemini 3.x it works.** We put `google_maps_grounding` and a BigQuery function tool in one ADK
agent, ran it, and both fired in a single turn—it queried the shelters table *and* grounded against
Maps. Verified 2026-08-09 on `gemini-3.6-flash`; the same request on `gemini-2.5-flash` still
returns the 400.

**Use a 3.x model and keep one agent.** If you pin an older one, use this, which is a perfectly good
architecture regardless:

- a **maps agent** holding `google_maps_grounding` and nothing else
- your **root agent** holding your own tools, an MCP server, and the maps agent wrapped in
  `AgentTool`

`GoogleMapsGroundingTool` takes no constructor arguments and does not accept
`bypass_multi_tools_limit`, so on an older model the sub-agent is the only route.

**A warning you will see and should ignore:** `Tools at indices [0] are not compatible with
automatic function calling (AFC). AFC is disabled.` That is the client declining to run the
function-calling loop for you. ADK runs its own. It is noise.

### What it will and will not do

| Will | Will not |
|---|---|
| Find places, addresses, ratings, hours | Give you a driving time or a distance |
| Tell you whether somewhere is open **right now** | Give you a route or a polyline |
| Tell you a place has closed, or been renamed | Answer anything but English |
| Say honestly when it does not know | Answer an *area* question about accessibility |

**On accessibility, the shape of the question decides the answer.** *"Which shelters in this county
take a wheelchair"* gets you nowhere. **One named building at one address** got a definite answer 5
times out of 8 in our testing, agreeing with FEMA's record 4 times out of 5 where both had a view.
That is worth building around.

Five to eight seconds per grounded call is normal. Routing is Private Preview—**do not design a
deliverable around a route.**

### Attribution is a requirement

Display the Google Maps sources **immediately after** the content they support, viewable within one
user interaction. The words "Google Maps" must not be re-capitalised, restyled, wrapped across
lines or translated—set `translate="no"`.

The response gives you `groundingChunks[].maps` with `place_id`, `title` and `uri`, and
`groundingSupports` mapping each sentence to its sources. Two practical notes: `segment.start_index`
is `None` on the first segment, so treat it as 0, and `confidence_scores` is never populated.

**What you may keep:** `place_id` and `review_id`. Not the rest. Which points at the right design
anyway—store the pointer against your shelter row, re-ground at runtime, and you have a system
that is both compliant and current.

---

## 8. Getting started

### Step 0—split into lanes, now

Four lanes: **data**, **agent**, **front end**, **story**. Ten people all watching one notebook run
is ten people not building anything. Agree who is doing what in the first five minutes.

### Step 1—get the repository

Click the green **Use this template** button at the top of this page, then **Create a new
repository**. That gives your team its own copy.

*Never used GitHub?* You need an account (free, one minute) and that is all. You will not need git
on the command line for any of this.

### Step 2—open your Google Cloud project

You have been given a project. Open the [Cloud Console](https://console.cloud.google.com) and check
the project name in the top bar matches the one on your lab card.

### Step 3—enable two APIs

Open **Cloud Shell**—the `>_` icon in the top right of the console, not a URL—and run:

```bash
gcloud services enable aiplatform.googleapis.com bigquery.googleapis.com
```

*Why this and not clicking:* the console will also prompt you to enable APIs, **twice**—once when
Colab Enterprise first opens, and again from a separate button on its homepage. That second prompt
is easy to miss and it is the most common way to lose ten minutes this morning. One command avoids
both.

### Step 4—open the notebook

In the console, go to **Colab Enterprise → My Notebooks → Import → source: URL**, and paste:

```
https://raw.githubusercontent.com/haggman/A4I2026-challenge-4-evacuation/main/notebooks/c4_01_load_explore.ipynb
```

No clone, no git, no authentication dance.

### Step 5—run it

Top to bottom. Change `STATE` in the first code cell if you are not doing Florida. It takes about
**80 seconds** to run and rather longer to read, and the reading is the point.

### If the notebook will not run

```bash
bash scripts/load.sh FL
```

Same tables, no teaching. Invoke it with `bash`, not `./scripts/load.sh`—that way it works
regardless of whether the executable bit survived. It is safe to run repeatedly; every load replaces
the whole table.

---

## 9. What the data actually says

Run the notebook and you will get these for your own state. Here is Florida, so the story lane can
start writing before the data lane finishes.

**2,793 shelters. 848,784 spaces. And this:**

| Wheelchair access | Shelters |
|---|---:|
| Recorded as **yes** | 247 |
| Recorded as **no** | 83 |
| **Nobody recorded either way** | **2,463—88%** |

Nationally it is roughly two thirds. **For most of America's shelters, nobody has written down
whether a person in a wheelchair can get in.**

And Grounding with Google Maps cannot fill that gap either. Five places, five different types, zero
definite answers—you will watch it happen in Section 2 of the notebook.

**The second number is scarcity.** Nationally only **2%** of shelters carry any medical-needs
designation. Their median capacity is 200—identical to the median for all shelters, so they are
not larger, merely designated. Florida has 155. South Carolina has one.

**The third is distance.** Median distance from a Florida census tract to any shelter: **1.9 km**.
To one recorded as wheelchair accessible: **7.1 km**. Nearly four times further. And **493,461
Floridians** live more than ten kilometres from any shelter at all—which for a household with no
vehicle is not a distance, it is a wall.

### The warning that comes with those numbers

Oregon reports 106 medical-needs shelters. Hawaii reports 88. Texas reports 31.

Oregon is not five times better prepared than Texas. **Oregon fills in the form.**

This file measures what states recorded, not what exists. **Any team that ranks states, counties or
neighbourhoods on these counts has measured bureaucracy and called it risk.** A judge will ask you
about this, and the good answer is that you treated unrecorded as its own visible category rather
than folding it into "no".

---

## 10. The data, and why we chose it

Five tables, in `<your-project>.evacuation_readiness`.

| Table | Rows (FL) | Source | Licence |
|---|---:|---|---|
| `shelters` | 2,793 | FEMA ESF#6 National Shelter System | US Government work. FEMA and American Red Cross co-attributed |
| `vulnerability_tracts` | 5,160 | Census ACS + CDC SVI 2022 + CDC PLACES 2025 | ACS public domain · SVI *"no constraints or limitations"* · PLACES *"Public Domain"* |
| `hazard_tracts` | 5,114 | FEMA National Risk Index | FEMA terms of use, attribution required, *"planning purposes only"* |
| `care_facilities` | 694 | CMS Nursing Home Provider Information | US Government work |
| `power_dependent_counties` | 67 | HHS emPOWER | US Government work, with a purpose-of-use condition |

**On emPOWER specifically**, because it is the sharpest signal here and it comes with a string
attached: *"Use of this tool and data signifies your agreement to use it for the specified purposes
and to make no attempt to identify any individual in this data."* The specified purpose is emergency
preparedness. That is what you are doing. Do not use it for anything else.

### Six defects you will meet, and why we show them to you

The notebook does not hand you a cleaned table, because these are the teaching:

1. **2,205 Wisconsin shelters have latitude and longitude swapped.** Every value is a well-formed
   float in a plausible range. A national range check passes because 97% of the file is fine.
2. **A blank string is not a null.** `ada_compliant IS NOT NULL` matches 49,356 rows nationally;
   46,338 of them contain a single space.
3. **`ada_compliant` is a decoy.** It is the obvious name, sits beside the real field, and has 63
   `YES` rows in the entire United States. `wheelchair_accessible` has 14,092.
4. **The flags are `YES`/`NO`, not `Y`/`N`**—and the same table uses whole words elsewhere. A
   filter returning zero looks exactly like a feature that does not exist.
5. **Census tract boundaries changed in 2020.** The BigQuery geometry table is 2010 vintage;
   everything else is 2020. Anchoring on the wrong one silently discards a fifth of the state.
6. **A handful of shelters record impossible capacities.** Georgia publishes two above 100,000
   spaces and California one—more than the largest stadium in the country. They are four rows in
   nine thousand, and they will quietly inflate any state total you put on a slide. The notebook
   flags them and prints them rather than dropping them, because the number is FEMA's and it is
   not our place to silently overwrite it.

**This is why the notebook's validation section has three verdicts, not two.** A **FAIL** means our
load is broken and you should stop. A **WARN** means the source data is untidy right there, and the
offending rows are printed underneath so you can look at them. Do look at them—defect 6 shows up as
a WARN, and it is the kind of thing that ends up on a slide.

### What we deliberately left out

| Left out | Why |
|---|---|
| `org_poc_name` and the other contact columns | 47,006 rows nationally contain a named individual. Individual-level personal data is an automatic rejection here |
| CDC SVI **Theme 3** and `RPL_THEMES` | Theme 3 is racial and ethnic minority status. See below |
| FEMA NRI `RISK_SCORE` and every `*_RISKS` column | Composites that multiply hazard by social vulnerability. Using them while also ranking on vulnerability double-counts |
| Real-time storm feeds | Empty in October. This is a planning tool |
| Google geocoding for the address-only files | Maps Platform forbids storing geocoded coordinates beyond 30 days, so they cannot underpin a table you keep |

### Race is not a model input here. It is an audit of your output.

This is a rule for the whole event, and the reasoning matters more than the rule:

- Race genuinely **does** correlate with these outcomes. Do not claim otherwise—anyone who knows
  the literature will correct you.
- But it is a **proxy**. The causal variables are physical and structural—vehicle access, building
  type, medical dependency, distance—and we can measure those directly.
- **Removing the column does not remove the bias.** Correlated proxies survive. This is "fairness
  through unawareness" and it does not work.

The remedy is auditing what your agent recommends, not deleting an input. Concretely, and it takes
about fifteen minutes: (1) produce your ranked list of tracts; (2) join it back to the demographic
columns; (3) compare the distribution against the county as a whole and report the difference,
whichever way it goes. This is consistent with Google's own published responsible-AI guidance.

---

## 11. Bringing your own data

**You may, and thoughtful sourcing earns credit.** Two conditions.

**Get the core working first.** "Let's find better data" is the classic way to lose ninety minutes.

**The licence test applies to anything you bring:**

| ✅ Usable | ❌ Not usable |
|---|---|
| US Government work / public domain | **NonCommercial (NC)**—winners are promoted publicly |
| CC0 | **NoDerivatives (ND)**—building on it is the point |
| CC BY (attribution is fine) | **Share-alike (CC BY-SA, ODbL)**—encumbers what you build |
| Aggregate statistics | **Any individual-level personal data** |
| | **No stated licence**—absence of a licence grants no rights |

**The trap most likely to catch this challenge by name:** an ArcGIS Online item called
*"Open Shelters in Harris County, Texas"* looks perfect and is **fabricated Esri training data**,
licensed for demonstration only. Its own description says so. Read the licence on anything you find
on ArcGIS Online, because plenty of it is coursework.

---

## 12. What's in this repository

```
README.md              this file
notebooks/
  c4_01_load_explore.ipynb   start here. The main teaching artifact
  c4_90_publish_snapshot.ipynb  ROI maintainers only—you do not need it
scripts/
  load.sh              headless fallback if the notebook will not run
data/                  empty. Everything lives in BigQuery, in your project
agent/                 EMPTY, on purpose. Your agent goes here
```

**`agent/` is empty deliberately.** We built the on-ramp—data, licensing, load process,
validation. We did not build the vehicle, because the design decisions in your agent are what you
are judged on and what you will demo. `agent/README.md` restates the requirements.

---

## 13. How you'll be judged

| Dimension | Weight |
|---|---:|
| Impact & insight | 30 |
| Technical execution | 30 |
| Rigor & judgment | 25 |
| Craft & communication | 15 |
| **Bonus—technology range and ambition** | **up to +10** |

The bonus is **additive and capped**, deliberately. A team that nails the fundamentals and adds
nothing can still win. A team that wires up five services with no coherent recommendation cannot win
on breadth alone.

**The dimension teams under-invest in is Rigor & judgment**, and on this challenge it is worth an
unusual amount. It covers where your data came from, whether you checked its licence, how you
handled protected attributes, whether you validated anything, and **whether you were honest about
what you do not know**.

That last one is the whole challenge. You are working with a dataset that is blank in the place it
most matters. A team that reports "no accessible shelters in this area" when the truth is "nobody
assessed them" has produced a confident, plausible, wrong answer about vulnerable people—which is
the worst possible output in this domain. A team that says so out loud, and designs around it, is
doing the job.

**Presentation time is short.** You get a brief pitch deck and a quick demo. **Rehearse to the
clock.** The numbers in Section 9 are your opening, and they are more striking than anything you
will build today.

---

## 14. Getting help

**Ask a coach.** They are in the room and they have seen these traps before.

**Run the diagnostic cell** at the very end of the notebook. It prints one compact block—table
names, row counts, validation results, step timings—and pasting that to a coach beats twenty
screenshots.

**If your agent throws `400 INVALID_ARGUMENT` about multiple tools**, you have hit the built-in tool
limit. Go back to [Section 7](#7-the-technology-youll-use).

**If a query returns zero rows**, check whether you are filtering on a field that is blank rather
than false. That is defect 2 and 4 above, and it accounts for most of the confusion this challenge
generates.
