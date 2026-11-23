# Sewer-Headspace Acoustic Lease Protocol for Fail-Closed Basement Flood Sealing

> **Note:** This README is an ultra-condensed summary of the research reports published by Unpatentable.org. To view the full innovation documentation and explore the complete research, use the links at the bottom of this document.

**Domain:** [Construction & Infrastructure](https://unpatentable.org/innovation/?domain=construction-infrastructure)
**Subdomain:** [Resilient Urban Planning](https://unpatentable.org/innovation/?domain=construction-infrastructure&subdomain=resilient-urban-planning)

---

### Why We Invented This Technology
The false assumption is that urban flash-flood deaths and damage in basement and ground-floor dwellings are an unavoidable consequence of undersized storm sewers, fixable only by multi-decade, multi-billion-dollar pipe enlargement. In reality most pluvial flooding harm comes from uncontrolled sewer backflow and surface water entering through a few predictable building openings, and low-cost sensing, passive hydraulic valves and block-scale flow modelling now make it possible to protect each building and actively steer stormwater to sacrificial surfaces (streets, parks, parking lots) instead of homes. Those who suffer are low-income renters in basement and ground-floor units in dense cities (on the order of tens of millions of people worldwide, my rough estimate, unverified by search); Hurricane Ida in 2021 drowned 11 people in New York City basement apartments, and urban pluvial flooding causes tens of billions of dollars in mostly uninsured losses each year, a figure that will grow with more intense rainfall if nothing changes.

### The Innovation
## Abstract
This concept holds a building's sewer lateral valve and low-opening barriers open only while the building keeps receiving an authenticated acoustic heartbeat. The heartbeat is sent through the air headspace of the sewer itself from a sentinel in a hydraulically lower manhole. Surcharge anywhere on the path removes the air column, so the hazard itself destroys the permission channel and the lease expires. Stored-energy actuators then seal the lateral and raise the barriers before the hydraulic grade line reaches basement fixtures. Radio messages can revoke leases but never extend them. A quorum of expired leases releases gravity kerb gates that route street flow onto designated floodable surfaces. All components are existing products.

## Background & Prior Work
Existing protection falls into four groups, none of which combines sensing, fail-safe default and block-scale coordination:
- **Passive backwater flaps** only react once reverse flow has started, and they foul.
- **Motorised backwater valves** (Kessel and ACO product families) close on a local probe, so sewage is already at the valve and a dead sensor fails open.
- **Buoyancy-raised barriers** need the floodwater to arrive before they lift.
- **Cloud-controlled stormwater assets and street flood sensors** depend on radio and servers, which are weak in basements and during storms.

The closest acoustic prior art is the InfoSense SL-RAT (US 8,220,484; ASTM F3220). It sends sound between adjacent manholes and scores blockage from transmission loss, as a one-off manual diagnostic with no actuation. In-pipe airborne acoustic data links have been studied for inspection robots and sensor networks (Kokossalakis, MIT 2006; IJAV 2023). The first search found no disclosure of loss of a sewer-headspace signal triggering valve closure, only these pieces separately.

The new element is using headspace transmission as a continuously renewed, authenticated permission whose absence actuates sealing.

## Competitive Landscape & Strategic Edge
Valve makers compete on local sensing and flap mechanics. Smart-city vendors compete on telemetry, dashboards and forecasting. Utilities fund pipe replacement. All three treat communication and sensing as separate layers, so every architecture has a state where the sensor or link is dead and the valve stays open.

The blind spot is that a sewer's airspace is both a free, already-installed waveguide into every basement and a physical indicator of remaining capacity. Binding the keep-open permission to that waveguide makes every failure mode resolve toward sealed: flood, blockage, noise, a dead sentinel or a dead battery. Open publication keeps this from becoming a proprietary valve ecosystem.

## Core Concept
**Architecture.** Each block has three device classes.

1. **Sentinel.** A battery-powered loudspeaker unit sits under the cover of the manhole whose pipe crown is the lowest point on the sewer path serving the block. It normally faces upstream toward the protected laterals.
2. **Building node.** This replaces the cap of the street-side cleanout, on the sewer side of any house trap. It contains a MEMS microphone, microcontroller, sub-GHz radio, supercapacitor reserve and a wet probe. It commands three things:
   - a full-bore air-to-close pinch valve (DN100–150, an existing industrial product) spliced into the lateral;
   - gas-spring flip-up barriers, 0.6–0.9 m high, at stairwells, window wells and garage ramps;
   - a sounder and egress light.
3. **Surface gates.** These are gravity-drop kerb gates or hinged kerb sections that open a flow path from the gutter to a designated floodable surface such as a sunken plaza, park or sacrificial street.

**Acoustic lease.** The sentinel emits a beacon in the plane-wave band of the main. The cutoff is f_c ≈ 0.586·c/D, about 670 Hz for D = 300 mm, so the band is 150–600 Hz.

- **Beacon format.** Sixteen symbols of 125 ms each (2 s total). Each symbol is one of 16 near-orthogonal chirp or pseudo-noise waveforms, giving 4 bits per symbol and 64 bits in all.
- **Processing gain.** Time-bandwidth product is about 56, giving roughly 17 dB per symbol.
- **Payload.** A 4-bit state and lease class, the low 20 bits of a monotonic counter, and a 40-bit truncated HMAC-SHA-256 over sentinel ID, full counter and state. The per-block key is provisioned at commissioning.
- **Acceptance rule.** The node accepts a beacon only if the MAC verifies and the counter is strictly increasing. Reverberant echoes of earlier beacons and ambient noise therefore cannot refresh a lease.
- **Lease duration.** Each valid beacon grants a time-to-live: 120 s in dry weather (beat every 40 s) and 20 s in wet weather (beat every 6 s).

**Node state machine.**

- **OPEN.** The lease is valid and in-band received level L is at least L₀ − 6 dB, where L₀ is the commissioning baseline.
- **GUARD.** Entered when L falls below L₀ − 10 dB, when ambient in-band noise rises more than 10 dB above the dry baseline, or when the sentinel flags wet state. The node itself caps the lease at 20 s, pre-alerts occupants and raises barriers.
- **SEALED.** Entered on lease expiry (three missed wet-weather beats), a local wet probe, a signed radio revocation, supercapacitor undervoltage or low reservoir pressure. The valve closes, the barriers latch up and the sounder says "move upstairs".
- **Re-opening.** Needs six consecutive valid beacons at L ≥ L₀ − 6 dB and a dry probe.

The node's own noise measurement shortens leases independently, so a sentinel that fails to speed up cannot leave a long lease running in a storm.

**Asymmetric authority.** This is the secure-protocol core. Only the physical channel can extend permission to stay open. Radio (LoRa-class) carries signed revoke and status messages only. Unauthenticated radio is ignored. A jammed radio changes nothing. A forged message cannot hold a valve open because no "extend" message type exists.

**Lead time.** The building is protected before backflow because the pipe crown on the acoustic path lies below the lowest fixture. The estimate is:

t_lead ≈ (z_fixture − z_crown,min) / (dH/dt)

For an elevation difference of 1–2 m and a hydraulic grade line rising at 0.1–0.5 m/min, this gives about 2–20 minutes. Lease expiry plus valve stroke (under 10 s for a pinch valve fed from a local reservoir) takes about 30 s. Siting rule: choose the sentinel manhole so that z_fixture − z_crown,min is at least 1 m for every enrolled building.

**Fail-closed actuation.** The pinch sleeve is closed by a 2–5 L air reservoir at 4–6 bar through a 3/2 pilot valve. A magnetic latch holds the pilot shut and a capacitor pulse releases it. A hardware comparator fires the release when supercapacitor voltage falls below the level needed for one actuation ("last gasp"). A pressure switch fires it when the reservoir nears the minimum sealing pressure. The device therefore cannot remain open without the means to close. Pinch valves seal around solids, which avoids flap fouling. A small trickle compressor or annual cartridge swap maintains the reservoir.

**Drainage while sealed.** Roof leaders tied to the lateral get an existing downspout diverter, released by the same node, that discharges to grade toward the gutter. Occupants receive a "do not flush" indication. The pipe volume upstream of the valve buffers sanitary flow during the typical 20–90 minute event.

**Block diversion.** Nodes report state by radio to the gate controller. Gates drop when at least k of n nodes (default 3 of 8, on two or more laterals) are SEALED or in GUARD within 120 s, or when the gate's own gutter float trips. A 30 s strobe and siren on the floodable surface precedes release. A spurious opening in dry weather routes no water. A failed gate controller drops the gates on its own last-gasp circuit.

## Physics & Logical Feasibility
Airborne plane waves propagate in any air-continuous duct. The smooth-wall viscothermal loss is α ≈ 3×10⁻⁵·√f / r Np/m, about 0.035 dB/m at 400 Hz for r = 0.15 m. That is roughly 4 dB over 100 m. Real joints, the junction split into the lateral (estimated 10–15 dB) and wall roughness add more. SL-RAT's routine manhole-to-manhole operation in 150–300 mm sewers shows usable transmission at segment scale.

A drive of 1 W electrical at about 1% efficiency yields roughly 100–110 dB SPL in the pipe (estimate). Dry-weather duty draws about 50 mW on average, or roughly 440 Wh per year. That needs a 0.5 kWh primary pack or a smaller rechargeable one.

A water-filled section presents an air–water impedance mismatch of about 3,600:1, so transmission collapses by tens of dB. Extinction is therefore unambiguous. Rising flow noise lowers the signal-to-noise ratio, which also pushes toward sealing.

Assumptions: gravity sewers with normally free headspace, a cleanout on the street side of any trap, and fixtures above the path crown.

## Implementation Readiness & Key Constraints
- **Phase 1 (months 0–9).** Channel-sounding campaign in 150–600 mm combined sewers, measuring loss, reverberation and wet-weather noise. Bench validation of the fail-closed pneumatics.
- **Phase 2 (months 9–24).** Fifty-building pilot on three blocks with shadow-mode logging before live actuation. Tuning of the L₀ thresholds.
- **Phase 3 (months 24–48).** Plumbing-code listing, utility manhole access agreements and a 1,000-building deployment.

Estimated programme cost is $15–25M, under the $50M ceiling.

Constraints:
- sewers with permanent sags or heavy debris have no clean baseline and need cleaning first;
- mains above about 600 mm need lower bands or multiple sentinels;
- intrinsically safe electronics are required in the sewer atmosphere;
- installing the valve means cutting the lateral indoors;
- key provisioning and rotation need a simple installer workflow.

## Utility & Purpose
The system gives basement and ground-floor occupants automatic sewage-backflow sealing, raised barriers and an upstairs-evacuation alarm minutes before water reaches fixtures. It does not depend on mains power, the internet or the occupant being awake. Beneficiaries are tenants of basement dwellings, small landlords, housing authorities and insurers seeking a verifiable mitigation device. Utilities gain a stopgap for the decades before sewer upsizing. Every node also acts as a continuous blockage monitor, because a slowly falling L in dry weather signals grease or root build-up. That gives utilities an operating reason to host sentinels.

## Potential Applications & Impact
Immediate uses are cloudburst-prone combined-sewer districts, basement flats, below-grade shops, transit entrances and hospital basements. The same lease principle extends to storm drains guarding underpasses, to utility vaults, and to industrial sites where backflow carries contaminants.

More broadly, the concept offers resilience planners a design pattern in which permission travels through the hazard's own path. This could apply to culverts, ducts or cable tunnels, where smoke or water filling a conduit extinguishes a keep-open token. It also yields dense, continuous sewer-headspace data for model calibration. Finally, it demonstrates privilege-asymmetric control (physical proof to extend, signature to revoke) as a template for cyber-safe municipal actuation.

## Market/Adoption Feasibility (If Applicable)
All figures here are estimates.
- Node electronics: $300–600.
- Pinch valve and pneumatics: $800–2,000.
- Barriers: $1,000–4,000 per opening.
- Sentinel: $1,500.
- Kerb gates: $5,000–20,000 each.
- Installed cost per building: about $4,000–10,000.

This compares with tens of thousands of dollars per sewage-backup claim and billions for sewer replacement. Adoption levers are insurer premium credits, municipal backwater-valve subsidy programmes that already exist in several North American cities, and utility interest in the blockage data. Barriers are plumbing-code approval of a normally-open powered valve, manhole access rights and installer training.

## Risk & Counterarguments
- **Nuisance sealing.** Debris, sags, sentinel battery failure or heavy flow noise can cause closures. These are safe but annoying, and repeated nuisance invites bypassing. Mitigations are GUARD hysteresis, health telemetry and tamper logging.
- **Inlet-limited flooding.** Surface water can arrive while sewers still have headspace. The gutter float, wet probe and radio revocation cover this case, with less lead time.
- **Fixtures below the path crown.** These get no acoustic lead and are excluded.
- **Barriers and egress.** Barriers must never impede escape, hence the low height and absence of door locking.
- **Displaced water.** Sealing many laterals modestly raises sewer and street levels elsewhere, so block-level hydraulic review is required.
- **Liability.** Responsibility for a failed or spurious actuation must be allocated contractually.

## Open Questions & Future Research Directions
- What are the measured attenuation, reverberation and storm-noise statistics across pipe materials, diameters and junction types? No field data for a live-sewer airborne modem appeared in the search.
- How much graded pre-warning does the L-versus-fill relationship give before extinction?
- What are the best band and waveform for mains above 600 mm, where higher modes propagate?
- Would building-initiated challenge-response with time-of-flight bounding, as in distance-bounding protocols, usefully locate the surcharge front?
- Can sentinels be chained along a catchment to produce a moving map of headspace loss?
- Can gully inlet throttles be coordinated with the gates?
- How do sleeve life and sealing performance hold up over years in sewage?

## Novelty Threshold Justification
**Medium Novelty**, a cross-domain synthesis. The closest prior art is SL-RAT ([US 8,220,484, per InfoSense](https://infosense.com/wp-content/uploads/2018/06/SL-RAT.pdf)), which is manual and diagnostic with no actuation. Next are in-pipe airborne acoustic communication studies ([IJAV 2023](https://iiav.org/ijav/content/volumes/28_2023_777961679948577/vol_1/1930_fullpaper_1796141679948801.pdf)), which cover data links and not safety permissions, and probe-triggered motorised backwater valves. Three elements are new relative to these:
1. a replay-proof authenticated heartbeat through sewer headspace used as an expiring keep-open lease, so that surcharge physically revokes it;
2. extend-only-by-physical-channel, revoke-by-signature authority;
3. lease-expiry quorum releasing fail-open surface diversion gates.

Three searches surfaced no such combination, though that is not an exhaustive patent search.

## References & Prior Art Checks
- InfoSense SL-RAT spec sheet (cites US 8,220,484; ASTM F3220 named in search results): https://infosense.com/wp-content/uploads/2018/06/SL-RAT.pdf
- ENR article on SL-RAT: https://www.enr.com/articles/8599-new-sewer-inspection-technology-makes-itself-heard
- SL-RAT description, acoustic pipe condition paper: https://scholarworks.uark.edu/cgi/viewcontent.cgi?article=2168&context=jaas
- Kokossalakis, MIT thesis 2006: https://www.researchgate.net/publication/279824523_Acoustic_data_communication_system_for_in-pipe_wireless_sensor_networks
- IJAV 2023 air-filled pipe acoustic communication: https://iiav.org/ijav/content/volumes/28_2023_777961679948577/vol_1/1930_fullpaper_1796141679948801.pdf
- Acoustic head-loss monitoring in partially full sewers: https://iwaponline.com/wst/article-abstract/65/9/1639/16002/Local-head-loss-monitoring-using-acoustic
- US 4,624,280, automatic gate valve shut-off: https://image-ppubs.uspto.gov/dirsearch-public/print/downloadPdf/4624280
- From recall: Brands & Chaum, "Distance-Bounding Protocols" (EUROCRYPT 1993); Kessel and ACO motorised backwater valves.

**Prior art searched:**
1. "acoustic signal transmitted through sewer pipe headspace loss of signal triggers automatic backwater valve closure". Used the UArk SL-RAT paper, the IWA paper and US 4,624,280. No combined disclosure found.
2. "SL-RAT acoustic transmission between manholes sewer blockage assessment InfoSense patent". Used the InfoSense spec sheet and the ENR article.
3. "acoustic data communication through sewer pipe air column wireless sensor network in-pipe airborne sound modem". Used the Kokossalakis thesis and the IJAV 2023 paper.

---

### About the Reports
Each innovation on Unpatentable.org is published as a three-part open documentation set:
- **Innovation Report** – the master blueprint describing the first principals concept, mechanism, and rationale behind the invention.
- **Implementation Guide** – a step-by-step, practical manual detailing how to build or prototype the innovation. Includes estimated cost breakdown.
- **Industry Impact Report** – an analytical overview exploring the potential economic, industrial, and societal effects of the innovation.

Together, these documents form a complete public disclosure package — enabling researchers, developers, and policymakers to understand, replicate, and build upon each invention.

---

### Explore this Innovation
- [Innovation Report](https://unpatentable.org/innovation/sewer-headspace-acoustic-lease-protocol-for-fail-closed-basement-flood-sealing)
- [Implementation Guide](https://unpatentable.org/innovation/implementation-guide-sewer-headspace-acoustic-lease-protocol-for-fail-closed-basement-flood-sealing)
- [Industry Impact Report](https://unpatentable.org/innovation/industry-impact-report-sewer-headspace-acoustic-lease-protocol-for-fail-closed-basement-flood-sealing)

### Download PDFs
- [Innovation Report (PDF)](https://unpatentable.org/wp-content/uploads/2026/11/Sewer-Headspace-Acoustic-Lease-Protocol-for-Fail-Closed-Basement-Flood-Sealing.pdf)
- [Implementation Guide (PDF)](https://unpatentable.org/wp-content/uploads/2026/11/Implementation-Guide-Sewer-Headspace-Acoustic-Lease-Protocol-for-Fail-Closed-Basement-Flood.pdf)
- [Industry Impact Report (PDF)](https://unpatentable.org/wp-content/uploads/2026/11/Industry-Impact-Report-Sewer-Headspace-Acoustic-Lease-Protocol-for-Fail-Closed-Basement-Flo.pdf)

---

**Timestamps:**
- [View Main Innovation Report on Arweave](https://arweave.net/oaUOc24iyeIkHFErMngzL8RMq5xy1-XuYL-fBaXVsUc)
- [View Implementation Guide on Arweave](https://arweave.net/nK0Zwg6G_34QLYJEZxuOYiL2LY2oP-OgOYcE0yQDCdk)

**Suggested Citation:**
> Unpatentable.org (2026). *Sewer-Headspace Acoustic Lease Protocol for Fail-Closed Basement Flood Sealing.* Defensive Publication Series.
> https://unpatentable.org/innovation/sewer-headspace-acoustic-lease-protocol-for-fail-closed-basement-flood-sealing

---

```json
{
  "domain": "Construction & Infrastructure",
  "subdomain": "Resilient Urban Planning",
  "published": "2026-11-23",
  "arweave_main_tx": "oaUOc24iyeIkHFErMngzL8RMq5xy1-XuYL-fBaXVsUc",
  "license": "CC BY 4.0",
  "arweave_implementation_tx": "nK0Zwg6G_34QLYJEZxuOYiL2LY2oP-OgOYcE0yQDCdk",
  "value_unlocked": "$38B"
}
```

Unpatentable.org is a public library of timestamped defensive publications —
open innovations released into the commons to prevent monopolization, democratize knowledge, and accelerate human progress.

License: CC BY 4.0
