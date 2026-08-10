# Your agent goes here

This folder is empty on purpose.

We built the on-ramp: every registered shelter in your state with capacity and accessibility, the
census tracts around them with disability, vehicle access and age, the counties where people depend
on electricity for medical equipment, hurricane and coastal-flood exposure per tract, and a
validation suite that tells you plainly whether any of it is wrong. We did not build the vehicle.
The design decisions in your agent are what you are judged on.

## What has to be true of what you build here

- **An ADK agent**, in Python.
- **At least one tool you built yourself.** A Python function tool, or one you defined in MCP
  Toolbox—either counts. The obvious candidate wraps a shelter query. The more valuable one holds
  the logic that is not a single query: deciding what "reachable" means for a household with no
  vehicle, weighing an unrecorded accessibility flag against a recorded one, turning a list of
  tracts into a readiness brief with a recommendation attached. Consuming only prebuilt generic
  tools and calling that your design does not count.
- **At least one Google-managed MCP server, consumed.** Do not author your own—use BigQuery's
  built-in server or the [MCP Toolbox for Databases](https://github.com/googleapis/mcp-toolbox).
- **Deployed to Google Cloud**—Agent Runtime or Cloud Run, your choice.
- **Your required differentiator: Grounding with Google Maps, genuinely used.** Your agent must
  call it and act on what comes back. Naming it in a slide does not count, and neither does calling
  it once for decoration in a flow that would give the same answer without it.

## The architecture is not a suggestion

**A built-in tool cannot share an agent with a function tool of your own.** The API is explicit:

```
400 INVALID_ARGUMENT—"Unable to submit request because Multiple tools are supported
only when they are all search tools."
```

`GoogleMapsGroundingTool` takes no constructor arguments and does **not** accept
`bypass_multi_tools_limit`, unlike `GoogleSearchTool`. ADK will not transparently wrap it either.
Since this event requires a tool of your own, you need two agents:

- a **maps agent** holding `google_maps_grounding` and nothing else
- your **root agent** holding your function tools, an MCP server, and the maps agent wrapped in
  `AgentTool`

Verified: `google_maps` and `google_search` *can* live in the same agent, so your maps sub-agent
can hold both grounding tools if you want web results alongside places.

This costs about twenty minutes if you know it and about an hour if you discover it. Now you know it.

## What Grounding with Google Maps will and will not do

| Will | Will not |
|---|---|
| Find places, addresses, ratings, opening hours | Tell you whether a place is wheelchair accessible |
| Tell you whether somewhere is open **right now** | Give you a driving time or a distance |
| Tell you a business has permanently closed | Give you a route or a polyline |
| Say honestly when it does not know | Answer anything but English |

Routing and Search Along Route are Private Preview. **Do not design a deliverable around a route.**
Five to eight seconds per grounded call is normal, not a bug.

Attribution is a requirement, not a nicety: display the Google Maps sources immediately after the
content they support, viewable within one user interaction, with the words "Google Maps" unmodified
and `translate="no"`. You may keep `place_id` and `review_id`; you may not cache, store or export
the rest of what comes back.

## The thing worth remembering while you build

**Our data knows what a place *is*. Maps knows whether it is *still there*. Neither knows whether it
works for the person you are trying to help.**

FEMA's file is a historical registry—a building recorded as a shelter during a 2022 hurricane may
not be a shelter, or even standing, today. That is what a runtime lookup is for. But no amount of
grounding will tell you whether someone in a wheelchair can get through the door, because for
roughly two thirds of American shelters **nobody has written it down.**

An agent that reports "no accessible shelters nearby" when it means "nobody recorded any" has
stated a fact about paperwork as a fact about buildings. An agent that says *"eleven of these I can
speak to, sixty-four nobody has assessed"* is more useful, more honest, and will score better.

See the README for the five data traps and Section 13 of the notebook for the constraints.
