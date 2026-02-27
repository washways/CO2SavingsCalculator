# CO2 Savings Calculator

A pure HTML/JavaScript tool that allows users to enter data to calculate CO₂ savings for WASH and Energy projects in Least Developed Countries (LDCs). The calculations are based on Gold Standard and CDM methodologies.

## Purpose

The calculator provides a unified interface to estimate CO₂ emission reductions from multiple project types:
- **A — Solar Water Pumping:** Replacing diesel or grid-powered water pumps with solar systems, while also capturing **Avoided Boiling Emissions** from avoiding the need to boil unsafe drinking water.
- **B — Institutional Induction Cooking:** Replacing biomass (wood/charcoal) cooking with electric induction in schools/hospitals.
- **C — Sanitation Methane Reduction:** Upgrading baseline sanitation (open defecation/pits) to improved systems (VIP, EcoSan, Biogas).
- **D — Improved HH Cookstoves:** Distributing improved cookstoves to replace traditional 3-stone fires.
- **E — Institutional Solar Lighting:** Replacing kerosene or diesel lighting with solar.
- **F — HH Induction Cooking + Solar:** Replacing household biomass cooking with induction/solar cooking.

### Key Features
- **Comprehensive CSV Export:** The tool allows users to export all active assumptions and results into a comprehensive CSV sheet detailing over 60 specific parameters, metrics, units, and methodology references, suitable for dashboard ingestion.
- **Dynamic Population UI:** All 6 modules contain a unified "Total Population Served" readout that dynamically scales up and down based on the overarching Portfolio Input variables (number of clinics, households per cluster, etc.), providing immediate transparency into the scale of interventions.
- **Transparent Guardrails:** Built-in safeguards actively block invalid logic, such as throwing overlapping double-counting warnings if users attempt to register households in both Module D and Module F simultaneously.

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
All cooking and fuel-combustion modules (B, D, F) and Avoided Boiling (A) rely on a single shared methodology engine (`FuelEngine.calculateEmissions`) to prevent double counting and ensure conservative fNRB application. Equations strictly use mass-based units without referencing energetic NCV.
- **Wood:** 
  - `Emission Reduction = Mass_wood_saved_kg × (EF_CO₂ + EF_NC) × fNRB`
- **Charcoal:** 
  - `Combustion emissions = Mass_charcoal × EF_NC` *(Charcoal CO₂ combustion is inherently biogenic/net-zero).*
  - `Upstream Wood Production emissions = Mass_charcoal × Production_Ratio × (EF_wood_CO₂ + EF_wood_NC) × fNRB` *(Default Production_Ratio is 6.0kg wood per 1kg charcoal).*
  - `Total Emission Reduction = Upstream Wood Production emissions + Combustion emissions`

### Module A: Solar Water Pumping & Avoided Boiling
**Methodology:** 
- GS AMS-I.F / CDM AM0020 (Pumping Displacement)
- GS Safe Water Supply Methodology (Avoided Boiling)

Calculates baseline emissions from water pumping that would have occurred without the solar intervention, as well as the emissions avoided by eliminating the need to boil unsafe water with biomass.
- **Total Population Served** = Institutional populations + Total Community Size (default 3,000 per system).
- **Hydraulic Energy (kWh/day)** = `(Volume × Head × 9.81) / 3600`
- **Shaft Energy (kWh/day)** = `Hydraulic Energy / Pump Efficiency`
- **Pumping Baseline (Diesel):** 
  - `Diesel Consumed (L/yr) = (Shaft Energy / (Diesel Efficiency × LHV)) × Operating Days`
  - `Emission Reduction (tCO₂e/yr) = Diesel Consumed × 2.68 kgCO₂/L / 1000`
- **Pumping Baseline (Grid):**
  - `Emissions = (Shaft Energy / Motor Efficiency) × Operating Days × Grid EF / 1000`
- **Avoided Boiling (Wood/Charcoal):**
  - Calculates the energy required to boil the assumed drinking water volume (2L/person/day) from 20°C to 100°C for 5 minutes (`~0.36 MJ/L`).
  - Total required fuel mass = `(Total Energy / Fuel NCV) / Stove Efficiency`.
  - Emission reduction is derived via the `FuelEngine` applying fNRB.

### Module B: Institutional Induction Cooking
**Methodology:** Gold Standard MECD v1.2

Calculates savings from displacing baseline institutional cooking fuels (wood, charcoal, fossil fuels) with electric/induction cooking.
- **Baseline Emissions (BE):**
  - Calculated automatically via the unified mass-based logic: `BE = Mass × (EF_CO₂ + EF_NC) × fNRB`.
- **Project Emissions (PE):**
  - If Grid connected: `PE = Project Electricity Consumed (kWh) × Grid EF / 1000`
- **Emission Reduction (ER)** = `BE - PE`

### Module C: Sanitation Methane Reduction
**Methodology:** IPCC 2006 Waste Ch.6 / Gold Standard Integrated SDG Sanitation

Calculates methane avoided by upgrading the sanitation system to one with a lower Methane Correction Factor (MCF).
- **Variables:**
  - `BOD` = Biochemical Oxygen Demand (IPCC default: 40 g/person/day)
  - `Bo` = Maximum CH₄ producing capacity (IPCC default: 0.6 kg CH₄/kg BOD)
  - `ΔMCF` = MCF(Baseline) - MCF(Improved)
- **Calculation:**
  - `CH₄ (t/yr) = Population × BOD × 365 × Bo × ΔMCF / 1000`
  - `Emission Reduction (tCO₂e/yr) = CH₄ × GWP₂₈ (28)`
- Evaluated separately for institutional settings and household levels.

### Module D: Improved Household Cookstoves
**Methodology:** Gold Standard TPDDTEC v4 / RECH · AMS-II.G

Calculates savings from fuel efficiency improvements in cooking.
- **Fuel Saved (kg)** = `Baseline Fuel Consumption × (1 - (Efficiency_baseline / Efficiency_improved))`
- **Stacking Discount:** 20% default discount for parallel old-stove use.
- **Emission Reduction (tCO₂e/yr)** = `Number of Stoves × Fuel Saved (kg) × (EF_CO₂ + EF_NC) × fNRB × (1 - Stacking Discount)`
- **Anti-Double-Counting Guardrail:** The calculator explicitly blocks calculation and throws an active UI error if households are simultaneously populated (>0) in both Module D (ICS) and Module F (Induction). Users must split their portfolio safely.

### Module E: Institutional Solar Lighting
**Methodology:** Gold Standard AMS-III.AR (Simplified)

Calculates savings from displacing kerosene or diesel lighting in institutions.
- **Calculation:**
  - `Emission Reduction (tCO₂e/yr) = Number of Facilities × Baseline Emissions Factor (tCO₂e/yr/facility)`

### Module F: Household Induction Cooking + Solar
**Methodology:** Gold Standard MECD v1.2 (Household Scale)

Calculates savings by totally replacing household biomass with solar-powered induction setups (Project Emissions assumed negligible/zero).
- **Emission Reduction (tCO₂e/yr)** = `Households × Fuel Consumption (kg/yr) × (EF_CO₂ + EF_NC) × fNRB`
- **Anti-Double-Counting Guardrail:** Blocked with a UI warning if households are simultaneously active in Module D.

## Financial Fuel Cost Savings

The calculator automatically tracks the amount of underlying fossil fuels or biomass displaced by these interventions (e.g. avoided diesel for pumps/lighting, avoided wood/charcoal for cooking) and translates these physical quantities into aggregate financial savings. 
- Local unit fuel prices (`$/kg` for wood/charcoal/LPG, `$/L` for diesel/kerosene) are populated automatically by country from the `defaults.json` database, and can be overridden by users locally via the Advanced Assumptions UI.
- Calculated savings are integrated into both the UI dashboard and CSV portfolio export as "Lifetime Fuel Cost Savings".

## Deployment
This project is built using basic HTML, CSS, and JS. It can be hosted on GitHub Pages by pushing the `index.html` file to the `main` or `gh-pages` branch. The live calculator can be accessed at: [https://washways.github.io/CO2SavingsCalculator/](https://washways.github.io/CO2SavingsCalculator/)
