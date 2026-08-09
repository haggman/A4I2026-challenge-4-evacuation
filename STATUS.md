# Challenge 4 — build status

**This is the working status document, not the challenge card.** It is for ROI and for Google.
It does not ship to teams. The student-facing `README.md` gets written at Stage 3, and it is
deliberately not written yet — see "The open decision" below.

Last updated: 2026-08-09, end of Stage 1.

| | |
|---|---|
| **Challenge** | Adaptive Emergency Evacuation & Vulnerable Community Dispatch *(name under review)* |
| **Differentiator** | Grounding with Google Maps *(under review — see below)* |
| **Stage 1 — data research** | **Done.** `_shared/research/datasets-challenge-4.md` |
| **Stage 2 — load notebook** | Not started. Blocked on the decision below. |
| **Stage 3 — repo** | Not started. |
| **Probe run** | `_shared/research/c4_probe.py` — written, not yet run |

---

## The open decision, and why nothing downstream has been written

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
