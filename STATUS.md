# Challenge 4 — build status

**This is the working status document, not the challenge card.** It is for ROI and for Google.
It does not ship to teams. The student-facing `README.md` gets written at Stage 3, and it is
deliberately not written yet — see "The open decision" below.

Last updated: 2026-08-09, end of Stage 1.

| | |
|---|---|
| **Challenge** | Adaptive Emergency Evacuation & Vulnerable Community Dispatch *(name under review)* |
| **Differentiator** | Grounding with Google Maps *(under review — see below)* |
| **Stage 1 — data research** | **CLOSED 2026-08-09.** `_shared/research/datasets-challenge-4.md` |
| **Stage 2 — load notebook** | **DONE and verified 2026-08-09.** `notebooks/c4_01_load_explore.ipynb` — 58 cells, **20/20 checks in 78 seconds** in a fresh Skills project |
| **Stage 3 — repo** | Next. `load.sh`, publish notebook, README, `JUDGING.md` section, maintainer differentiator run |
| **Probe runs** | `c4_probe.py` … `c4_probe6.py` — **all six run in Skills, all findings recorded** |

## The headline finding

Of **53,942** usable shelters in FEMA's national file, **11,954** record wheelchair access as YES,
2,157 as NO, and roughly **36,000 — two-thirds — are blank.**

**For two-thirds of America's shelters, nobody has recorded whether a person in a wheelchair can
get in.** And Grounding with Google Maps cannot fill that gap either: 0 of 5 accessibility probes
returned a definite answer.

That gap is the challenge's honest edge. It goes in the README's "questions that need more than we
gave you" table, not in a footnote.

Beside it: **only 1,089 of 53,942 shelters — 2.02% — carry any MEDICAL designation.** Florida has
151, South Carolina has one. Median medical-shelter capacity is 200, identical to the median for
all shelters, so they are not larger, only designated.

---

## Decisions taken 2026-08-09 (Patrick)

**The reframe is approved and the challenge is built around the Stage 1 finding.**

- **Name:** *Adaptive Evacuation Readiness & Vulnerable Community Planning* — one phrase changed
  from Google's title so it stays recognisably descended from the doc they have read.
- **User:** a county emergency manager or resilience planner, working weeks or months ahead of a
  storm. Not a dispatcher, not during an event.
- **Deliverable**, keeping Paul's three-segment shape:
  1. **Who can't leave** — neighbourhoods and institutions holding people who cannot self-evacuate,
     with counts and the reason for each. Paul's pickup roster, de-personalised to block-group and
     facility level.
  2. **Where they'd go, and whether it holds up** — candidate destinations checked live against the
     world, with the gaps named. Paul's safe-haven matching, strengthened.
  3. **What to fix before the season** — the readiness gap list. Replaces turn-by-turn routing,
     which is Private Preview and does not exist for us.
- **The hook (notebook §3): let the differentiator fail in public.** Ask Maps grounding where to
  shelter people in the chosen county — it returns a fluent, cited, plausible list. Then ask the
  three questions a planner actually needs (wheelchair access, generator, surge zone) and it
  answers *"I couldn't find specific information."* Teaches the division of labour in the first ten
  minutes, and smoke-tests the API at minute 10 rather than minute 200.
- **Scope: one state, chosen by the team.** Server-side state filter on FEMA and CMS, county filter
  on emPOWER. `load.sh` takes the state as its argument, the way Challenge 1's takes a city.
- **HHS emPOWER is IN.** Its purpose-of-use condition — *"agreement to use it for the specified
  purposes and to make no attempt to identify any individual"* — is satisfied by preparedness
  planning, which is the stated purpose. Terms go in the notebook markdown.

### Consequences of those choices, to handle in Stage 2

- The hook makes `aiplatform.googleapis.com` a **hard dependency of the data phase**, not just the
  build phase. §3 must fail gracefully and tell the team exactly what to enable.
- One-state scope means the "pick your state" table must carry the **real numbers** from probe 6,
  or someone picks South Carolina and finds a single medical shelter.
- Proposed default: **Florida** — the right hazard profile, 151 medical shelters (second in the
  country), and the worst wheelchair-reporting rate of any coastal state at 88 of 1,533. That
  contrast is the demo.

## Verified Florida numbers, 2026-08-09

`shelters` 2,793 · `vulnerability_tracts` 5,160 · `hazard_tracts` 5,114 · `care_facilities` 694 ·
`power_dependent_counties` 67.

- Wheelchair access: **247 yes, 83 no, 2,463 unrecorded (88%)**
- Medical-needs shelters: **155**. Total evacuation capacity: **848,784**
- Median distance to any shelter **1.9 km**; to one recorded wheelchair accessible **7.1 km**
- **493,461 Floridians** live more than 10 km from any shelter at all
- All 2,793 shelters placed into all 67 counties and joined to emPOWER
- Model is **`gemini-3.6-flash`** — verified with Maps grounding, and half the price per grounded
  prompt of the 2.5 line

## What was watched on the first Skills run (all resolved)

Written against published docs and six probe runs, but never executed end to end. The things most
likely to bite, in order:

1. **Runtime.** Budget is 5-10 minutes. The nearest-shelter query cross-joins every tract against
   every shelter (Florida: ~5,160 × 1,533 = 7.9M pairs), which should be trivial for BigQuery but
   has never been timed. If the total lands past ten minutes this becomes a demonstration artifact
   and `load.sh` becomes the student path.
2. **The CMS nursing-home pull.** It pages the full national file (14,693 rows) and filters
   locally, because the datastore API's server-side filter syntax is unverified. That is ~8
   requests and possibly the slowest step.
3. **`load_table_from_dataframe` on nullable booleans.** `wheelchair_accessible` and friends are
   pandas `boolean` dtype. Should map to BigQuery BOOL, unconfirmed.
4. **The SVI and NRI field lists.** Both cells ask the service what fields it has and intersect
   with what we want, so a renamed field degrades rather than crashes — but check what the
   "not present" line prints.
5. **Section 2's Gemini call** in a project where `aiplatform.googleapis.com` is not yet enabled.
   It should print instructions and continue, not raise.

## Still open — licensing calls, not blockers on structure

- **The framing question with Google.** The runtime-refusal transcript makes it an easy ask.
- **FEMA NSS rights** — no stated licence, American Red Cross co-attribution.
- **FEMA NRI terms** — a revocable terms-of-use grant rather than an open licence.
- **US-only?** Canada works on the differentiator and has none of the data.

## The original blocking decision, kept for the record

Google's own terms for Grounding with Google Maps say, verbatim:

> "You won't use Grounding with Google Maps for high risk activities including emergency response
> services."

Identical wording on two independent Google pages, under "Service usage requirements":
https://ai.google.dev/gemini-api/docs/generate-content/maps-grounding and
https://firebase.google.com/docs/ai-logic/grounding-google-maps. Reinforced by Google Maps Platform
ToS §3.2.1(c)(i), with High Risk Activities defined in §20 as *"activities where the use or failure
of the Services would reasonably be expected to result in death, serious personal injury, or severe
environmental damage or property damage."*

The challenge as Paul framed it — real-time dispatch of rescue vehicles, producing a "Rescue Vehicle
Dispatch Manifest" — is an emergency response service in the plain meaning of the phrase. Maps
grounding is the required differentiator. Both cannot stand.

**Three paths, and this is Patrick's call:**

1. **Re-aim the product, keep the people.** A preparedness planning tool used months ahead by a
   county resilience planner, not a dispatch tool used during a storm. Same data, same agent, same
   differentiator. The challenge name changes with it.
2. **Get Google's answer in writing** before building. This is Google's restriction on Google's own
   product inside a programme Google commissioned. Recommended alongside path 1.
3. **Change the differentiator, keep the framing.** Fallback is **BigQuery geospatial** (`ST_*`,
   spatial joins against evacuation-zone polygons) — non-overlapping with Challenge 1's BQML,
   Challenge 2's vector search and Challenge 3's property graph, and a natural fit for a problem
   that is fundamentally about who is inside which polygon.

**Why the README is not written yet.** Its first line is the human hook, and the hook is the thing
under review. Writing the card now means writing it twice, and worse, it means a student-facing
document asserting a differentiator that may not survive. That is precisely the failure mode
`AUTHORING-GUIDE.md` §2 exists to prevent.

**What is *not* blocked.** The data spine below survives all three paths intact. So does the probe
pack. If path 3 is chosen, only the differentiator sections change.

---

## The data spine, as it stands after Stage 1

Full detail, licences quoted verbatim, and the considered-and-rejected list are in
`_shared/research/datasets-challenge-4.md`. Summary:

| Layer | Source | Why it is here |
|---|---|---|
| **Who** | **HHS emPOWER** (ZIP, monthly, verified live) | Electricity-dependent medical equipment, power wheelchairs, home dialysis, home oxygen. The strongest "cannot self-evacuate" signal that exists in public data |
| | **CDC/ATSDR SVI 2022** (tract) | `EP_DISABL`, `EP_LIMENG`, `EP_MOBILE`, `EP_GROUPQ`, `EP_NOVEH` in one file. Themes separate cleanly, so Theme 3 drops out with no residue |
| | **CDC PLACES 2025** (tract) | `MOBILITY`, `SELFCARE`, `INDEPLIVE`, `LACKTRPT` |
| | **ACS in BigQuery** | Denominators, `no_cars`, `group_quarters`, `mobile_homes`. Note: no disability and no living-alone columns |
| | **Census CRE 2024** (tract) | "N people with 3+ risk factors" — the number an emergency manager would actually quote |
| **Facilities** | **CMS Nursing Homes** `4pq5-n9py` | Coordinates *and* bed counts. 14,693 rows, monthly |
| | **CMS Dialysis** `23ew-n7w9` | Address-only — geocoding needed. `of_dialysis_stations` gives patient volume |
| | **NCES EDGE schools 2024-25** | Coordinates. Daytime concentration, shelter building, bus depot |
| | **FTA National Transit Database** | ADA-accessible vehicle counts by agency — the supply side |
| **Hazard** | **FEMA National Risk Index** (tract) | `HRCN_*` and `CFLD_*` exposure and expected-annual-loss, joined on `TRACTFIPS`. No spatial work |
| | **FEMA NFHL** | Regulatory flood zones. Geometry-only, spatial join |
| | **Metro evacuation zones** | The actual planning unit — but licences are weak across the board |
| | `bigquery-public-data.noaa_hurricanes.hurricanes` | Narrative context, zero load time |
| **Destination** | **FEMA ESF#6 National Shelter System** | 71,710 standing records with capacity, ADA, generator, surge exposure, and `population_code` flagging medical-needs shelters |

### Two rights questions carried into Stage 2

1. **The differentiator's emergency-response clause.** Above.
2. **FEMA's shelter service states no licence at all.** The only rights string is
   `copyrightText: "FEMA, Mass Care, American Red Cross"`. NSS is not an OpenFEMA dataset, so
   OpenFEMA's terms do not cover it, and the American National Red Cross is a federally chartered
   nonprofit — 17 USC 105 does not reach its contributions.

### One non-negotiable ingest rule, whatever else changes

**47,006 of the 71,710 FEMA shelter rows carry `org_poc_name`, `org_poc_phone`,
`org_poc_after_hours_phone` and `org_poc_email`.** Those are named individuals. Under `BRIEF.md`
that is an automatic rejection *as published*. The notebook must use an explicit `outFields` list
and **never `SELECT *`**. This is a written rule, not a convention.

---

## The finding that shapes the architecture

There is no clean, openly-licensed, standing shelter dataset. That was tested rather than assumed —
Miami-Dade's shelter feed returned `count: 0` on 2026-08-09, because it is an event-time feed and
there was no storm.

Meanwhile FEMA's own layer description says, in capitals:

> "THIS LAYER SHOULD NOT BE USED TO DETERMINE THE OPERATIONAL STATUS OF A FACILITY DURING AN ACTIVE
> EMERGENCY."

So the authoritative federal source explicitly disclaims the one thing a runtime lookup is for.
**Our data is the memory; Maps grounding is the live check; neither can do the other's job.** That
division is the challenge's spine, and it is stronger than a clean dataset would have been.

---

## Differentiator facts already settled (see the research file for sources)

- **Setup is one API.** `aiplatform.googleapis.com` plus billing. No Maps Platform key, no opt-in
  form. Confirmed from Google's Next '26 codelab, which pins `gemini-2.5-flash` and
  `GOOGLE_CLOUD_LOCATION="global"`.
- **Cost is not a problem.** ~$100–250 per city worst case; likely $0 if lab projects carry their
  own billing account. Never billed per agent.
- **The widget is optional** — its only implementation was deprecated 2026-06-15. Text source links
  with correct attribution satisfy the requirement, so **`adk web` survives as a face.**
- **ADK forces a sub-agent.** `GoogleMapsGroundingTool.__init__` takes no arguments and does **not**
  accept `bypass_multi_tools_limit` (read from installed `google-adk` 2.6.3;
  `_convert_tool_union_to_tools` has no branch for it). Since every team must build a tool of their
  own, **wrapping a maps-only agent in `AgentTool` is the architecture, not a workaround.** This
  belongs on the card with the real error text next to it.
- **Routing is contradictory.** The Maps Platform page lists Routing and Search Along Route as
  Private Preview, yet Google's own codelab parses encoded polylines out of `chunk.maps.text` with
  no allowlisting. The probe settles it.
- **Latency 5–8 seconds is normal.** Tell teams, or they debug a non-bug.

---

## Housekeeping done in this folder

- `notebooks/README.md` added. The folder was empty, and **git does not track empty directories** —
  it would have silently not appeared in the pushed repo, exactly as happened on the first push of
  Challenge 1.
- `scripts/README.md` corrected: it referenced `notebooks/01_load_explore.ipynb`, missing the
  challenge prefix. Now `notebooks/c4_01_load_explore.ipynb`.
- `agent/README.md` deliberately **not** updated. It still carries the generic text. Its per-challenge
  half restates the differentiator requirement, and that is the thing under review.

## Next actions

1. **Patrick:** decide the framing, and decide whether to put the emergency-response clause to Google.
2. **Patrick:** run `_shared/research/c4_probe.py` in a fresh Skills project and paste the diagnostic
   block back. It settles Toronto coverage, polylines, the multi-tool error text, host reachability
   from a Colab runtime, the ACS column questions, and whether FEMA NSS is one row per facility or
   one row per facility-per-incident.
3. **Then Stage 2:** propose the notebook section structure and discuss before writing a cell.
