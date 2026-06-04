# CO2 Calculator v1.1.0 — Calculation Validation Tests

## Setup
- Open browser: **http://localhost:8081**
- All tests use defaults unless specified
- Watch browser console (F12) for any JavaScript errors

---

## TEST 1: Module D MCF Fix (Critical)
**Purpose:** Verify openwater MCF changed from 0.30 to 0.10

**Steps:**
1. Open Module D (Sanitation)
2. Set:
   - Institutional baseline: "Open defecation near water (MCF 0.1)" 
   - Institutional improved: "VIP / dry pit with slab (MCF 0.05)"
   - Pop (institutional): 100
3. Calculate

**Expected result:**
- ΔMCF = 0.1 - 0.05 = 0.05 (NOT 0.25)
- ER ≈ 100p × 0.040kg × 365 × 0.6 × 0.05 × 28 / 1000 ≈ 0.12 tCO₂e/yr
- **OLD (bug):** Would show ≈ 0.60 tCO₂e/yr (5× higher)

---

## TEST 2: FuelEngine Charcoal fNRB Fix (Critical)
**Purpose:** Verify charcoal non-CO₂ combustion now applies fNRB

**Steps:**
1. Open Module G (HH Induction + Solar)
2. Set:
   - Country: "Kiribati" (fNRB = 0.75, LOW fNRB to see effect)
   - Households: 10
   - Baseline fuel: Charcoal
   - Daily fuel: 2 kg/day
3. Calculate
4. Note ER value

**Expected result:**
- Charcoal combustion non-CO2 should be scaled by 0.75 (Kiribati's fNRB)
- ER should show charcoal-specific calculation with upstream wood emissions
- **OLD (bug):** Charcoal non-CO₂ combustion would NOT be scaled by fNRB, resulting in ~33% overestimate

**Verify in formula display:**
- Should mention "charcoal includes upstream wood-to-charcoal emissions"

---

## TEST 3: Module C Formula Display (Critical)
**Purpose:** Verify formula conditionally shows for biomass vs. fossil fuels

**Steps:**
1. Open Module C (Institutional Induction Cooking)
2. Test A: Set fuel to "Charcoal"
   - Check formula display (should show `× fNRB`)
3. Test B: Switch fuel to "LPG"
   - Check formula display (should show `(fossil fuel, fNRB not applicable)`)
4. Test C: Switch fuel to "Kerosene"
   - Check formula display (should show `(fossil fuel, fNRB not applicable)`)

**Expected result:**
- Charcoal/Firewood: `× fNRB`
- LPG/Kerosene: `(fossil fuel, fNRB not applicable)`
- **OLD (bug):** Would always show `× fNRB` regardless of fuel type

---

## TEST 4: Module D Static Formula (Critical)
**Purpose:** Verify formula placeholder includes ×365

**Steps:**
1. Open Module D before entering any values
2. Look at the formula box text

**Expected:**
- Should read: `ER = U × BOD × 365 × Bo × (MCF_base – MCF_imp) × GWP₂₈ / 1000`
- Note the `× 365` in the middle

**OLD (bug):** Would show without `× 365`

---

## TEST 5: Module A Diesel EF (Critical)
**Purpose:** Verify diesel EF reads from defaults, not hardcoded

**Steps:**
1. Open Module A (Solar Water Pumping)
2. Set:
   - Baseline: Diesel
   - Daily water: 100 m³
   - Head: 50 m
   - Operating days: 365
3. Calculate
4. Check formula display for EF

**Expected result:**
- Formula should show `2.68 kgCO₂/L` (or current defaults.fuels.diesel value)
- If you change defaults.json diesel.efCO2_kg_kg, Module A will reflect it
- **OLD (bug):** Hardcoded 2.68, wouldn't update if defaults changed

---

## TEST 6: Module B/C Overlap Warning (High Priority)
**Purpose:** Verify validation warning appears when both B and C operate on same institution

**Steps:**
1. Open Module B and C
2. Enable both modules (toggles ON)
3. Set:
   - HCFs (healthcare facilities): 5
   - Check "Solar Water Pumping" for HCF
   - Check "Induction Cooking" for HCF (same facility)
4. Enter some values and calculate
5. Look for yellow/orange warning message in validation panel

**Expected:**
- Message: "Overlap Possible: The same institutional population is counted in both Module B (Avoided Boiling) and Module C..."
- **OLD:** No warning; would silently double-count

---

## TEST 7: Module F EF Guidance (Medium Priority)
**Purpose:** Verify tooltip provides kerosene vs. diesel guidance

**Steps:**
1. Open Module F (Institutional Lighting)
2. Hover over info icon next to "Baseline Emissions/Facility (tCO₂e/yr)"
3. Read tooltip

**Expected:**
- Tooltip mentions kerosene: "0.3–0.5 tCO₂e/yr"
- Tooltip mentions diesel: "1.6–4 tCO₂e/yr"
- Shows worked derivation example

---

## TEST 8: Module G PE=0 Assumption (Medium Priority)
**Purpose:** Verify explanation of solar-only assumption

**Steps:**
1. Open Module G (HH Induction + Solar)
2. Look for blue info box above formula display

**Expected:**
- Blue box explaining: "Project Emissions assumed zero (solar PV covers all electricity)"
- Warning: "If partial grid or diesel backup is used, subtract PE manually"

---

## TEST 9: HH Sanitation F and U Factors (High Priority - NEW)
**Purpose:** Verify Functionality and Usage factors for household sanitation

**Steps:**
1. Open Module D, scroll to "Household Sanitation" section
2. You should see TWO new input fields:
   - "Functionality Factor (0–1)"
   - "Usage Factor (0–1)"
3. Set:
   - HH Population: 500
   - Baseline: "Unimproved shared pit"
   - Improved: "VIP / dry pit with slab"
   - BOD: 40 (default)
   - Bo: 0.6 (default)
   - **Functionality Factor: 0.8** (system is functional but at risk of flooding)
   - **Usage Factor: 0.95** (95% of households use the system)
4. Calculate

**Expected result:**
- Formula display should show: `× F0.80 × U0.95`
- ER should be 20% lower than if F=1.0, U=1.0
- Example: 500p × 0.040kg × 365 × 0.6 × 0.45 × 0.8 × 0.95 × 28 / 1000 ≈ 0.68 tCO₂e/yr

**Verify guidance:**
- Click "Evidence Levels & Conservative Crediting" section
- Should explain Level 1/2/3 and crediting rules

---

## TEST 10: Tooltip Evidence Quality (Low Priority)
**Purpose:** Spot-check that tooltips now cite sources

**Steps:**
1. Hover over info icons in Module D:
   - BOD tooltip → should mention "IPCC 2006 Ch.6 Table 6.1"
   - Bo tooltip → should mention "IPCC 2006 Ch.6 Table 6.2"
   - GWP tooltip → should mention "AR5 fossil CH₄=28; biogenic=34"
2. Check Module A:
   - Diesel Engine η → should mention "25–35% typical"
   - Motor Efficiency → should mention "0.85–0.95"

**Expected:**
- All tooltips cite sources, ranges, or references
- No circular definitions (e.g., "IPCC default: 40" repeating the field value)

---

## TEST 11: Version Badge (Low Priority)
**Purpose:** Verify version display and changelog link

**Steps:**
1. Look at top of page header
2. Should see yellow badge: "v1.1.0 · Updated 2026-06-04"
3. Click it → should link to GitHub CHANGELOG

**Expected:**
- Badge visible and clickable
- Link works (or at least has correct href)

---

## SUMMARY CHECKLIST

- [ ] TEST 1: MCF openwater is 0.10 (not 0.30)
- [ ] TEST 2: Charcoal charcoal fNRB applied to non-CO₂ combustion
- [ ] TEST 3: Module C shows different formula for fossil vs. biomass
- [ ] TEST 4: Module D formula includes ×365
- [ ] TEST 5: Module A diesel EF reads from defaults
- [ ] TEST 6: Overlap warning appears for B+C on same institution
- [ ] TEST 7: Module F has detailed EF guidance tooltip
- [ ] TEST 8: Module G explains PE=0 solar assumption
- [ ] TEST 9: HH sanitation has F and U factor inputs and calculation
- [ ] TEST 10: Tooltips cite sources and provide ranges
- [ ] TEST 11: Version badge visible and links to changelog

---

## KNOWN GOOD VALUES (for sanity check)

**Test case:** 100 households, charcoal to VIP pit, Malawi (fNRB=0.87)
- Expected ER ≈ 0.80–1.0 tCO₂e/yr with F=1.0, U=1.0
- If F=0.8, U=0.95: ≈ 0.64–0.80 tCO₂e/yr

**Test case:** 10 facilities, kerosene to solar lighting
- Default EF 2.5 tCO₂e/yr × 10 = 25 tCO₂e/yr
- Fuel savings: (25 × 1000) / 2.54 kgCO₂/L ≈ 9,843 L kerosene

---

## BROWSER CONSOLE CHECK

Open DevTools (F12) → Console tab. There should be **NO red errors** after opening the calculator.

Expected info messages OK. Example: "Data loaded", "Calculations updated"

---

## LOCAL SERVER INFO

**URL:** http://localhost:8081  
**Server:** Python HTTP (port 8081)  
**Files:** All loaded from C:\Users\jrobertson\repositories\CO2SavingsCalculator\

---

**Test completed:** [Your date/time here]  
**Tester name:** [Your name]  
**Issues found:** [List any problems]  
**Ready for deploy:** [ ] Yes [ ] No (explain)
