# CO2 Savings Calculator — Changelog

All notable changes to this calculator are documented below. Users can check this file to see what has been updated and when.

## Versioning

The calculator uses semantic versioning: **MAJOR.MINOR.PATCH**
- **MAJOR**: Significant methodology changes, major bug fixes affecting core calculations
- **MINOR**: New features, evidence/tooltip improvements, formula refinements
- **PATCH**: UI fixes, typo corrections, minor adjustments

---

## [1.1.0] — 2026-06-04

### Major Calculation Fixes
- **CRITICAL:** Fixed Module D "openwater" MCF from 0.30 to 0.10 (IPCC 2006 Table 6.3). This corrects a 3× overestimation of sanitation emission reductions for projects upgrading from open defecation near water.
- **CRITICAL:** Fixed FuelEngine charcoal combustion non-CO₂ emissions — now correctly apply fNRB scaling. Previously over-counted by up to 6× in low-fNRB countries (e.g., Kiribati at fNRB=0.15). Fixes Modules C, E, and G charcoal calculations.
- **CRITICAL:** Module D static formula placeholder was missing the `× 365 days/year` multiplier. Corrected display to match actual computation.
- **CRITICAL:** Module C formula display now correctly shows `× fNRB` only for biomass fuels; for fossil fuels (LPG, kerosene) shows `(fossil fuel, fNRB not applicable)` to reflect actual calculation.
- Fixed Module A diesel EF (2.68 kgCO₂/L) to read from `defaults.fuels.diesel` instead of hardcoded constant — ensures consistency if fuel EF is updated.
- Fixed Module F kerosene fuel savings back-calculation to derive EF from `defaults.fuels.kerosene` using density assumption rather than hardcoded 2.5 kgCO₂/L constant.

### Methodology & Evidence Improvements
- **Module D:** Replaced generic method tag with direct link to IPCC 2006 Vol. 5 Ch.6 methodology. Noted that Gold Standard SDG Sanitation methodology is currently "in-development."
- **Module D:** Updated GWP-CH₄ field label and tooltip to clarify AR5 biogenic (34) vs. fossil (28) values. Wastewater methane is biogenic; clarified when to use 28 vs. 34.
- **Household Sanitation (Module D):** Enhanced calculation with **Functionality (F) and Usage (U) factors** per Gold Standard SDG Sanitation methodology. Users can now adjust F (0–1) for system functionality/flood risk and U (0–1) for verified use. Formula updated to: `ER = Population × BOD × 365 × Bo × ΔMCF × F × U × GWP / 1,000,000`. Added expandable guidance on three evidence levels (Level 1 desk-based, Level 2 programme MRV with verification, Level 3 locally calibrated) and conservative crediting rules (do not credit flooded/collapsed/unmanaged systems).
- **defaults.json:** Fixed implausibly low fNRB values for Pacific atoll nations (Kiribati, Tuvalu). Revised to conservative high-fNRB estimates (0.70+) pending country-specific CDM Tool 30 studies.
- **defaults.json:** Added `fnrb_source` fields indicating data provenance (e.g., "Bailis et al. 2015", "CDM Tool 30", "estimate — country-specific study recommended").
- **defaults.json:** Added notes on charcoal `prodRatio` range (traditional kilns 8–12, improved kilns 5–7, retort 3–4). Default 6 reflects improved kiln assumption.

### Tooltip & Documentation Enhancements
- **Module D:** Rewrote all 4 tooltips (BOD, Bo, GWP, ΔMCF) to cite IPCC 2006 Ch.6 specific tables, provide valid ranges, and explain when/why to deviate from defaults.
- **Module A:** Added "Motor Efficiency η" tooltip with typical range 0.85–0.95 (IEC IE2 standard). Fixed "Diesel Engine η" tooltip to show 25–35% typical range with IPCC/IEA source.
- **Module A:** Updated diesel LHV formula tooltip to dynamically display actual user-entered LHV instead of hardcoded "10 kWh/L."
- **Module B:** Added tooltip for "Boiling Stove Efficiency" (8–15% for 3-stone baseline per Gold Standard TPDDTEC). Updated "Energy per Litre" (0.36 MJ/L) tooltip with physics derivation and Gold Standard Methodology 443 v1.0 citation.
- **Module B:** Added validation warning when both Module B (Avoided Boiling) and Module C (Institutional Cooking) are active on the same institutional population — users must verify these represent distinct fuel uses.
- **Module C:** Updated formula box to conditionally display different text for fossil vs. biomass fuels, with explanatory tooltip for fNRB application.
- **Module F:** Updated baseline EF tooltip with worked derivation for diesel genset (5 kWh/day scenario yielding ~1.6–2.5 tCO₂e/yr/facility depending on hours). Clarified that kerosene lamps require different parameters per AMS-III.AR.
- **Module F:** Replaced unstable CDM DB hash-path URL with stable CDM PDF reference for AMS-III.AR v2.
- **Module G:** Added explanatory tooltip on Module G formula: "Project Emissions assumed zero (solar PV covers all electricity). If partial grid or diesel backup is used, subtract PE manually."

### Version & Metadata Updates
- Added version number and last-update date display to calculator header.
- Created CHANGELOG.md to track all changes and provide user transparency on updates.

---

## [1.0.0] — Initial Release

- 7 core calculation modules (A–G) for WASH/energy projects in LDCs
- Gold Standard and CDM methodologies for CO₂ savings estimation
- Comprehensive country defaults (fNRB, Grid EF, fuel prices)
- CSV export of all assumptions and results
- Portfolio-level impact tracking (People Reached, Institutions Reached)
- Overlap detection and validation warnings

---

## How to Use This Changelog

- **For latest updates:** Check the version number at the top of the calculator interface.
- **For detailed changes:** Read the entry for the most recent version below.
- **Backwards compatibility:** Calculations from v1.0.0 may differ slightly from v1.1.0 due to bug fixes (especially Module D sanitation, charcoal fuel handling, and formula displays). If you have published results, note the version used.

---

## Reporting Issues or Suggesting Improvements

If you discover a calculation inconsistency, evidence gap, or have suggestions for methodology updates, please contact the project maintainers. Future versions will incorporate feedback and publish updates here.
