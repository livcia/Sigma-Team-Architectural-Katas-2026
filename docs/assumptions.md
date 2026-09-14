# Assumptions

The competition brief ([architectural-katas-2026.md](../architectural-katas-2026.md))
intentionally leaves many details unspecified. This document lists the assumptions
made across this documentation set so a judge can see exactly where the team
filled a gap, and why. Where a number is an assumption rather than a fact from the
brief, it is labelled as such here and should be treated the same way wherever it
is reused.

## Assumptions arising from gaps in the brief

- A1: The estate does not currently have any digital ticketing, telemetry or
  analytics system; this solution assumes a **greenfield build**, not a migration
  from an existing system.
- A2: "Popularity" and "demand" are assumed to mean visitor counts, dwell time and
  queue length per ride/enclosure/zone, since the brief does not define the metric
  precisely.
- A3: "High animal care costs" are assumed to be driven primarily by **late
  detection** of health and environmental problems, since the brief explicitly
  links cost to prevention ("preventing illness saves money").
- A4: "Reasons to return" are assumed to mean a combination of personalised
  in-visit experience and post-visit engagement (for example, a reason to come back
  next season), since the brief does not specify a mechanism.
- A5: The three-year, 5,000 → 15,000 visitor growth target is taken as given from
  the brief; no assumption is made about the shape of that growth curve (linear,
  seasonal, etc.) beyond what is needed for illustrative capacity planning.
- A6: The brief's mention of "jumping piranhas" and similarly hazardous exotic
  animals is treated as an indicator that **some enclosures carry direct safety
  risk to keepers and visitors**, reinforcing why AI must not make unsupervised
  safety decisions.
- A7: No specific regulatory framework (animal welfare law, data protection regime)
  is named in the brief; this solution assumes the estate must comply with
  generally reasonable animal-welfare and personal-data-protection practice,
  without naming a specific jurisdiction's law.

## Assumptions about data

- D1: Ticketing and entry data is assumed to be available electronically once a
  ticketing system exists (FR1/FR2 in [requirements.md](requirements.md)); no
  assumption is made about whether it originates from online sales, on-site
  kiosks, or both.
- D2: Environmental telemetry (temperature, humidity, water quality, etc.) is
  assumed to be the primary automated signal for animal-health intelligence;
  detailed biometric sensing per animal is out of scope (see below).
- D3: Keeper observations are assumed to be captured as structured or
  semi-structured input (for example a simple logging form), not free-form
  unstructured notes only, so that they can be combined with sensor data.
- D4: Historical data (past attendance, past health incidents) is assumed not to
  exist in digital form today; initial AI models will need to operate with limited
  historical data and improve as data accumulates. This is a design constraint, not
  a fact stated in the brief.
- D5: Data volumes from 5,000–15,000 daily visitors and 55 enclosures are assumed
  to be modest by general industry standards (i.e., not "big data" scale), so
  architectural complexity should be justified by resilience and correctness needs,
  not by raw data volume.

## Assumptions about MQTT devices

- M1: MQTT-capable devices are assumed to be deployed at the **zone and
  enclosure/ride level** (for example, one or a small number of devices per
  enclosure or ride), not per individual animal, given budget constraints implied
  by "budget for MQTT-capable hardware devices."
- M2: Devices are assumed to publish at a modest frequency (periodic telemetry plus
  event-driven messages), not high-frequency streaming, consistent with a
  budget-constrained, intermittently-connected estate.
- M3: Devices are assumed to connect to a **local** MQTT broker/gateway within
  their zone rather than directly to the cloud, since Wi-Fi is patchy; this is the
  basis for the edge-buffering approach described in
  [README.md](../README.md#handling-patchy-wi-fi-mqtt-edge-and-cloud).
- M4: No assumption is made about a specific device vendor or model; devices are
  referred to only by their logical role (sensor, gateway).

## Assumptions about the visitor application

- V1: A visitor-facing application or portal (mobile app or web) is assumed to
  exist as the channel for tickets, personalised recommendations and feedback,
  since the brief expects a "personalised visitor experience" but does not specify
  its delivery mechanism.
- V2: Any visitor preference or behavioural data used for personalisation is
  assumed to be **collected with explicit visitor opt-in**, never inferred
  covertly, consistent with SPA5 in [requirements.md](requirements.md).
- V3: Visitors without the app (for example, walk-up ticket buyers) are assumed to
  still be able to enjoy the estate fully, with a non-personalised but complete
  experience; the app enhances rather than gatekeeps access.

## Out of scope

The following are explicitly out of scope for this submission, to keep the
architecture focused on the three core problems:

- Detailed financial/payment processing design (assumed to exist as a standard
  ticketing capability, not elaborated further).
- Physical ride safety engineering and certification (assumed to be handled by
  existing mechanical/safety inspection processes, outside this AI-focused
  architecture).
- Per-animal biometric monitoring (for example wearable sensors on individual
  animals) — only enclosure/zone-level environmental telemetry is in scope.
- Detailed HR/staff scheduling systems — the solution recommends staffing actions
  but does not replace a staff rostering system.
- Marketing campaign design — the solution supports return-visit recommendations
  but does not design a full CRM/marketing platform.
- Selection of specific cloud vendors, AI model providers, or product names (see
  [README.md](../README.md)); only logical roles and architectural decisions are
  described.
