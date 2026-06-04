# CO2 Savings Calculator

A pure HTML/JavaScript tool that allows users to enter data to calculate CO₂ savings for WASH and Energy projects in Least Developed Countries (LDCs). The calculations are based on Gold Standard and CDM methodologies.

**Current Version:** [v1.1.0](CHANGELOG.md) (Updated 2026-06-04)  
**Status:** ✅ Production Ready | [See Release Notes](CHANGELOG.md)

## Purpose

The calculator provides a unified interface to estimate CO₂ emission reductions from multiple project types:
- **A — Solar Water Pumping:** Replacing diesel or grid-powered water pumps with solar systems.
- **B — Avoided Boiling:** Eliminating the need to boil unsafe drinking water with biomass.
- **C — Institutional Induction Cooking:** Replacing biomass (wood/charcoal) cooking with electric induction in schools/hospitals.
- **D — Sanitation Methane Reduction:** Upgrading baseline sanitation (open defecation/pits) to improved systems (VIP, EcoSan, Biogas).
- **E — Improved HH Cookstoves:** Distributing improved cookstoves to replace traditional 3-stone fires.
- **F — Institutional Solar Lighting:** Replacing kerosene or diesel lighting with solar.
- **G — HH Induction Cooking + Solar:** Replacing household biomass cooking with induction/solar cooking.

### Key Features

#### Core Functionality
- **Centralized Module Toggles:** Users can globally activate or deactivate any of the 7 core calculator modules from a master "Active Modules" panel at the top of the interface.
- **Detailed Calculation Tooltips:** All parameters cite IPCC chapters, Gold Standard methodologies, and CDM references. Tooltips explain formulas, ranges, and when to deviate from defaults—no circular definitions.
- **Comprehensive CSV Export:** The tool allows users to export all active assumptions and results into a comprehensive CSV sheet detailing over 60 specific parameters, metrics, units, and methodology references, suitable for dashboard ingestion.
- **Scale of Impact Tracking:** The sidebar actively computes the total "People Reached" and "Institutions Reached" dynamically across all active interventions without double-counting communities and households.
- **Dynamic Population UI:** All 7 modules contain a unified "Total Population Served" readout that dynamically scales up and down based on the overarching Portfolio Input variables (number of clinics, households per cluster, etc.), providing immediate transparency into the scale of interventions.
- **Transparent Guardrails:** Built-in safeguards actively block invalid logic, such as throwing overlapping double-counting warnings if users attempt to register households in both Module E and Module G simultaneously.

#### v1.1.0 Enhancements
- **Conservative HH Sanitation Crediting (NEW):** Household sanitation now includes Functionality (F) and Usage (U) factors (0–1 range) to account for system risks and verified adoption, per Gold Standard SDG Sanitation methodology.
- **Enhanced Charcoal Calculations:** Fixed FuelEngine to correctly apply fNRB scaling to charcoal combustion non-CO₂ emissions (prevents up to 6× overestimation in low-fNRB countries).
- **Evidence-Based Defaults:** All tooltips now cite IPCC 2006 chapter/table numbers, Gold Standard document versions, and provide valid parameter ranges.
- **Improved Module D:** Corrected MCF values per IPCC Table 6.3; updated method tag to stable IPCC 2006 Vol.5 Ch.6 reference.

## Detailed Calculation Methodology

### General Parameters
- **fNRB (Fraction of Non-Renewable Biomass):** Country-specific default values provided by the UNFCCC / Gold Standard.
- **Grid Emission Factor (GEF):** Country-specific default values in `kgCO₂/kWh`.
- **Fuel Emission Factors:** 
  *(Note: All emission factors strictly use **mass-based units** `kgCO₂e/kg fuel` derived from IPCC NCV values for simplicity.)*
  - **Firewood:** `NCV = 15 MJ/kg`, `EF = 1.742 kgCO₂/kg (CO₂)` + `0.135 kgCO₂e/kg (Non-CO₂)`
  - **Charcoal:** `NCV = 29.5 MJ/kg`, `EF = 2.638 kgCO₂/kg (CO₂)` + `0.150 kgCO₂e/kg (Non-CO₂)`
  - **Production Ratio (Charcoal):** Assumes 6 kg of wood is required to produce 1 kg of charcoal.

### Unified FuelEngine
All cooking and fuel-combustion modules (C, E, G) and Avoided Boiling (B) rely on a single shared methodology engine (`FuelEngine.calculateEmissions`) to prevent double counting and ensure conservative fNRB application. Equations strictly use mass-based units without referencing energetic NCV.
- **Wood:** 
  - `Emission Reduction = Mass_wood_saved_kg × (EF_CO₂ + EF_NC) × fNRB`
- **Charcoal:** 
  - `Combustion emissions = Mass_charcoal × EF_NC × fNRB` *(Charcoal CO₂ combustion is inherently biogenic/net-zero; non-CO₂ is scaled by fNRB per Gold Standard MECD v1.2).*
  - `Upstream Wood Production emissions = Mass_charcoal × Production_Ratio × (EF_wood_CO₂ + EF_wood_NC) × fNRB` *(Default Production_Ratio is 6.0kg wood per 1kg charcoal; range 5–12 depending on kiln type).*
  - `Total Emission Reduction = Upstream Wood Production emissions + Combustion emissions`

### Module A: Solar Water Pumping
**Methodology:** [Gold Standard AMS-I.F / CDM AM0020 (Pumping Displacement)](https://cdm.unfccc.int/methodologies/DB/9KJWQ1G0WEG6LKHX21MLPS8BQR7242)

Calculates baseline emissions from water pumping that would have occurred without the solar intervention.
- **Total Population Served** = Institutional populations + Total Community Size (default 3,000 per system).
- **Hydraulic Energy (kWh/day)** = `(Volume × Head × 9.81) / 3600`
- **Shaft Energy (kWh/day)** = `Hydraulic Energy / Pump Efficiency`
- **Pumping Baseline (Diesel):** 
  - `Diesel Consumed (L/yr) = (Shaft Energy / (Diesel Efficiency × LHV)) × Operating Days`
  - `Emission Reduction (tCO₂e/yr) = Diesel Consumed × 2.68 kgCO₂/L / 1000`
- **Pumping Baseline (Grid):**
  - `Emissions = (Shaft Energy / Motor Efficiency) × Operating Days × Grid EF / 1000`

### Module B: Avoided Boiling
**Methodology:** [GS Safe Water Supply Methodology (Avoided Boiling)](https://globalgoals.goldstandard.org/standards/443_v1.0_ee_gs-methodology-for-emission-reductions-from-safe-drinking-water-supply/)

Calculates the emissions avoided by eliminating the need to boil unsafe water with biomass.
- **Avoided Boiling (Wood/Charcoal):**
  - Calculates the energy required to boil the assumed drinking water volume (2L/person/day) from 20°C to 100°C for 5 minutes (`~0.36 MJ/L`).
  - Total required fuel mass = `(Total Energy / Fuel NCV) / Stove Efficiency`.
  - Emission reduction is derived via the `FuelEngine` applying fNRB.

### Module C: Institutional Induction Cooking
**Methodology:** [Gold Standard MECD v1.2](https://globalgoals.goldstandard.org/standards/431_V1.2_TC_EE_ICS_Methodology-for-Metered-and-Measured-Energy-Cooking-Devices.pdf)

Calculates savings from displacing baseline institutional cooking fuels (wood, charcoal, fossil fuels) with electric/induction cooking.
- **Baseline Emissions (BE):**
  - Calculated automatically via the unified mass-based logic: `BE = Mass × (EF_CO₂ + EF_NC) × fNRB`.
- **Project Emissions (PE):**
  - If Grid connected: `PE = Project Electricity Consumed (kWh) × Grid EF / 1000`
- **Emission Reduction (ER)** = `BE - PE`

### Module D: Sanitation Methane Reduction
**Methodology:** [IPCC 2006 Vol. 5 Ch.6 (Wastewater)](https://www.ipcc-nggip.iges.or.jp/public/2006gl/vol5.html) / [Gold Standard Integrated SDG Sanitation (In-Development)](https://globalgoals.goldstandard.org/in-development/integrated-sdg-methodology-for-sanitation/)

Calculates methane avoided by upgrading the sanitation system to one with a lower Methane Correction Factor (MCF). Follows IPCC Tier 1 methodology with Gold Standard conservative crediting rules.

#### Institutional Sanitation
- **Calculation:** `CH₄ (t/yr) = Population × BOD × 365 × Bo × ΔMCF / 1000`
- Uses MCF values from IPCC 2006 Table 6.3 (open defecation, pits, septic, etc.)

#### Household Sanitation (v1.1.0+)
Includes Functionality (F) and Usage (U) factors for conservative crediting:
- **Variables:**
  - `BOD` = Biochemical Oxygen Demand (IPCC default: 40 g/person/day)
  - `Bo` = Maximum CH₄ producing capacity (IPCC default: 0.6 kg CH₄/kg BOD)
  - `ΔMCF` = MCF(Baseline) - MCF(Improved) per IPCC Table 6.3
  - `F` = Functionality factor (0–1; accounts for flooding, structural failure, abandonment)
  - `U` = Usage factor (0–1; reflects verified household use)
  - `GWP` = 28 (AR5 100-year; biogenic methane)
- **Calculation:** `ER (tCO₂e/yr) = Population × BOD × 365 × Bo × ΔMCF × F × U × GWP / 1,000,000`
- **Evidence Levels:** Level 1 (desk-based), Level 2 (programme MRV with verification), Level 3 (locally calibrated flux measurement)

### Module E: Improved Household Cookstoves
**Methodology:** [Gold Standard TPDDTEC v4 / RECH · AMS-II.G](https://globalgoals.goldstandard.org/standards/407_V4.0_EE_ICS_Reduced-Emissions-from-Cooking-and-Heating-TPDDTEC.pdf)

Calculates savings from fuel efficiency improvements in cooking.
- **Fuel Saved (kg)** = `Baseline Fuel Consumption × (1 - (Efficiency_baseline / Efficiency_improved))`
- **Stacking Discount:** 20% default discount for parallel old-stove use.
- **Emission Reduction (tCO₂e/yr)** = `Number of Stoves × Fuel Saved (kg) × (EF_CO₂ + EF_NC) × fNRB × (1 - Stacking Discount)`
- **Anti-Double-Counting Guardrail:** The calculator explicitly blocks calculation and throws an active UI error if households are simultaneously populated (>0) in both Module E (ICS) and Module G (Induction). Users must split their portfolio safely.

### Module F: Institutional Solar Lighting
**Methodology:** [Gold Standard AMS-III.AR (Simplified)](https://cdm.unfccc.int/methodologies/DB/S3RZMK6KR289WKK0VB1ETT6K73Y3DR)

Calculates savings from displacing kerosene or diesel lighting in institutions.
- **Calculation:**
  - `Emission Reduction (tCO₂e/yr) = Number of Facilities × Baseline Emissions Factor (tCO₂e/yr/facility)`

### Module G: Household Induction Cooking + Solar
**Methodology:** [Gold Standard MECD v1.2 (Household Scale)](https://globalgoals.goldstandard.org/standards/431_V1.2_TC_EE_ICS_Methodology-for-Metered-and-Measured-Energy-Cooking-Devices.pdf)

Calculates savings by totally replacing household biomass with solar-powered induction setups. **Note:** Project Emissions are assumed zero (PE=0) because electricity is sourced from a dedicated on-site solar PV system. If grid backup or diesel hybrid systems are used, Project Emissions must be subtracted separately.
- **Emission Reduction (tCO₂e/yr)** = `Households × Fuel Consumption (kg/yr) × (EF_CO₂ + EF_NC) × fNRB`
- **Assumption:** All electricity from solar PV; PE=0
- **Anti-Double-Counting Guardrail:** Blocked with a UI warning if households are simultaneously active in Module E (Improved Cookstoves).

## Financial Fuel Cost Savings

The calculator automatically tracks the amount of underlying fossil fuels or biomass displaced by these interventions (e.g. avoided diesel for pumps/lighting, avoided wood/charcoal for cooking) and translates these physical quantities into aggregate financial savings. 
- Local unit fuel prices (`$/kg` for wood/charcoal/LPG, `$/L` for diesel/kerosene) are populated automatically by country from the `defaults.json` database, and can be overridden by users locally via the Advanced Assumptions UI.
- Calculated savings are integrated into both the UI dashboard and CSV portfolio export as "Lifetime Fuel Cost Savings".

## Documentation & Transparency

All major parameters, assumptions, and calculations are evidence-based and traceable:

- **Methodology References:** Every module links to its source methodology (IPCC 2006, Gold Standard, CDM)
- **Evidence-Based Tooltips:** Parameter tooltips cite specific chapter/table numbers, valid ranges, and when to deviate from defaults
- **Conservative Defaults:** All defaults use IPCC and Gold Standard baselines; user can override with local data
- **Changelog:** See [CHANGELOG.md](CHANGELOG.md) for detailed release notes and version history

### Additional Resources

- **[CHANGELOG.md](CHANGELOG.md)** — Release notes for v1.1.0 and prior versions
- **[TEST_CALCULATIONS.md](TEST_CALCULATIONS.md)** — 11 detailed test cases for verification
- **[VERIFICATION_REPORT.md](VERIFICATION_REPORT.md)** — Comprehensive audit and plausibility checks

## Deployment
This project is built using basic HTML, CSS, and JS. It can be hosted on GitHub Pages by pushing the `index.html` file to the `main` or `gh-pages` branch. The live calculator can be accessed at: [https://washways.github.io/CO2SavingsCalculator/](https://washways.github.io/CO2SavingsCalculator/)

### Running Locally

**Option 1: Python HTTP Server**
```bash
cd CO2SavingsCalculator
python -m http.server 8081
# Open http://localhost:8081 in your browser
```

**Option 2: Node.js Server**
```bash
node server.js
# Open http://localhost:8081 in your browser
```

## Version History

| Version | Date | Key Changes |
|---------|------|------------|
| [v1.1.0](CHANGELOG.md) | 2026-06-04 | Critical MCF fix, charcoal fNRB scaling, HH sanitation F/U factors, evidence improvements |
| v1.0.0 | Earlier | Initial release with 7 modules |

## Contributing

For bug reports, feature requests, or methodology suggestions, please open an issue on GitHub.

## License

This calculator is provided as-is for use in carbon project development. All methodologies follow IPCC 2006 Guidelines and Gold Standard/CDM requirements.
