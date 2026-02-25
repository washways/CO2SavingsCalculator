# CO2 Savings Calculator

A pure HTML/JavaScript tool that allows users to enter data to calculate CO₂ savings for UNICEF-style WASH and Energy projects in Least Developed Countries (LDCs). The calculations are based on Gold Standard and CDM methodologies.

## Purpose

The calculator provides a unified interface to estimate CO₂ emission reductions from multiple project types:
- **A — Solar Water Pumping:** Replacing diesel or grid-powered water pumps with solar systems.
- **B — Institutional Induction Cooking:** Replacing biomass (wood/charcoal) cooking with electric induction in schools/hospitals.
- **C — Sanitation Methane Reduction:** Upgrading baseline sanitation (open defecation/pits) to improved systems (VIP, EcoSan, Biogas).
- **D — Improved HH Cookstoves:** Distributing improved cookstoves to replace traditional 3-stone fires.
- **E — Institutional Solar Lighting:** Replacing kerosene or diesel lighting with solar.
- **F — HH Induction Cooking + Solar:** Replacing household biomass cooking with induction/solar cooking.

## Detailed Calculation Methodology

### General Parameters
- **fNRB (Fraction of Non-Renewable Biomass):** Country-specific default values provided by the Gold Standard.
- **Grid Emission Factor (GEF):** Country-specific default values (kgCO₂/kWh).
- **Fuel Emission Factors:** 
  - **Firewood:** NCV = 15 MJ/kg, EF = 1.742 tCO₂/t (CO₂) + 0.135 tCO₂e/t (Non-CO₂)
  - **Charcoal:** NCV = 29.5 MJ/kg, EF = 2.638 tCO₂/t (CO₂) + 0.150 tCO₂e/t (Non-CO₂), Production Ratio = 6 kg wood / kg charcoal

### Module A: Solar Water Pumping
**Methodology:** Gold Standard AMS-I.F / CDM AM0020

Calculates baseline emissions from water pumping that would have occurred without the solar intervention.
- **Hydraulic Energy (kWh/day)** = `(Volume × Head × 9.81) / 3600`
- **Shaft Energy (kWh/day)** = `Hydraulic Energy / Pump Efficiency`
- **Baseline (Diesel):** 
  - `Diesel Consumed (L/yr) = (Shaft Energy / (Diesel Efficiency × LHV)) × Operating Days`
  - `Emission Reduction (tCO₂e/yr) = Diesel Consumed × 2.68 kgCO₂/L / 1000`
- **Baseline (Grid):**
  - `Emissions = (Shaft Energy / Motor Efficiency) × Operating Days × Grid EF / 1000`

### Module B: Institutional Induction Cooking
**Methodology:** Gold Standard MECD v1.2

Calculates savings from displacing baseline institutional cooking fuels (wood, charcoal, fossil fuels) with electric/induction cooking.
- **Baseline Emissions (BE):**
  - For Wood: `BE = Fuel Consumed (t/yr) × (EF_CO₂ + EF_NC) × fNRB`
  - For Charcoal: `BE = Fuel Consumed (t/yr) × (EF_CO₂ (charcoal) + Wood Prod Ratio × EF_CO₂ (wood) × fNRB)`
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
- **Fuel Saved (FS)** = `Baseline Fuel Consumption × (1 - (Efficiency_baseline / Efficiency_improved))`
- **Emission Factor (EF):** Differs based on wood vs charcoal, incorporating fNRB.
- **Stacking Discount:** 20% default discount for parallel old-stove use.
- **Emission Reduction (tCO₂e/yr)** = `Number of Stoves × FS × fNRB × EF × (1 - Stacking Discount)`

### Module E: Institutional Solar Lighting
**Methodology:** Gold Standard AMS-III.AR (Simplified)

Calculates savings from displacing kerosene or diesel lighting in institutions.
- **Calculation:**
  - `Emission Reduction (tCO₂e/yr) = Number of Facilities × Baseline Emissions Factor (tCO₂e/yr/facility)`

### Module F: Household Induction Cooking + Solar
**Methodology:** Gold Standard MECD v1.2 (Household Scale)

Calculates savings by totally replacing household biomass with solar-powered induction setups (Project Emissions assumed negligible/zero).
- **Emission Reduction (tCO₂e/yr)** = `Households × Fuel Consumption (t/yr) × EF × fNRB`

## Deployment
This project is built using basic HTML, CSS, and JS. It can be hosted on GitHub Pages by pushing the `index.html` file to the `main` or `gh-pages` branch.
