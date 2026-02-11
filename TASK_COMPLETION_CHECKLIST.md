# A2 Assignment - Task Completion Verification

## Quick Reference: What Each Task Does and Where to Find It

---

## ✅ TASK 1: Calculate Inflow/Outflow Indicators

**What to calculate**: 5 indicators comparing inflow vs outflow
- (a) 3-day peak flow % change
- (b) 31-day low flow % change  
- (c) 3-day peak timing difference (days)
- (d) 31-day low timing difference (days)
- (e) Flashiness (RBI) difference

**Functions in notebook**:
- `calculate_indicator_a(df)` → Returns % change in peak flows
- `calculate_indicator_b(df)` → Returns % change in low flows
- `calculate_indicator_c(df)` → Returns timing shift of peaks (days)
- `calculate_indicator_d(df)` → Returns timing shift of lows (days)
- `calculate_indicator_e(df)` → Returns RBI difference

**Where to run**: Cells 5-9 define functions, execution happens in `analyze_reservoir()`

**Output**: Dictionary with average values for each indicator per reservoir

---

## ✅ TASK 2: Bar Plots for Indicators (a), (b), (e)

**What to create**: 3 bar plots showing all reservoirs
- Plot 1: Indicator (a) - 3-day peak % change
- Plot 2: Indicator (b) - 31-day low % change
- Plot 3: Indicator (e) - RBI difference

**Function in notebook**:
- `plot_indicators(summary_df, output_dir)`

**Where to run**: Called in `main()` function

**Output**: PNG file with 3-panel figure: `reservoir_indicators_analysis.png`

**What it shows**: 
- X-axis: Reservoir names (SHA, ORO, FOL, etc.)
- Y-axis: Indicator value
- Bars: Color-coded by reservoir
- Reference line at zero

---

## ✅ TASK 3: Mass Balance Function

**What to create**: Function that calculates storage timeseries

**Function in notebook**:
- `calculate_reservoir_mass_balance(df, max_storage_tafd, initial_storage_fraction, ...)`

**Key features**:
- Unit conversion: CFS → TAFD (using factor 0.0019834711)
- Initial storage: 70% of maximum capacity
- Simplifying assumptions: No evaporation, no infiltration
- Physical constraints: 0 ≤ storage ≤ max capacity
- Mass balance: `Storage(t+1) = Storage(t) + Inflow - Outflow`

**Where to run**: Cell with mass balance function, executed in `analyze_reservoir_mass_balance()`

**Returns**: 
- DataFrame with storage timeseries
- Indicators dictionary with flood/low storage metrics

**Output columns**:
- `storage_tafd` → Daily storage in thousand acre-feet
- `storage_percent` → Storage as % of capacity
- `storage_unconstrained` → Storage before applying limits
- `spill_event`, `low_storage_event` → Event flags

---

## ✅ TASK 4: 9-Panel Storage Volume Plot

**What to create**: Single figure with 9 subplots (3×3 grid), one per reservoir

**Where created**: Inside `analyze_reservoir_mass_balance()` function

**Layout**:
```
Row 1: SHA  ORO  FOL
Row 2: NML  DNP  PNF  
Row 3: CLE  EXC  BER
```

**Each panel shows**:
- X-axis: Date/time
- Y-axis: Storage as % of capacity (0-100%)
- Filled curve: Storage level over time
- Reference lines: 100% (red), 50% (orange)
- Title: Reservoir code and max capacity

**Output**: PNG file: `all_reservoirs_comparison.png`

**Purpose**: Compare storage dynamics across all reservoirs at once

---

## ✅ TASK 5: Flood Indicator & Low Storage Exposure

**What to calculate**: Two new metrics
1. **Flood Indicator**: Average annual spill events
2. **Low Storage Exposure**: Average annual low storage events

**Modified function**: `calculate_reservoir_mass_balance()` now returns BOTH:
- DataFrame with storage timeseries
- Dictionary with indicators including flood & low storage

**Flood Indicator Details**:
- **Definition**: Average number of spill events per year
- **Spill event**: Period where unconstrained storage > max capacity
- **Key**: Uses unconstrained storage BEFORE applying physical limits
- **Grouping**: Consecutive spill days = 1 event

**Low Storage Exposure Details**:
- **Definition**: Average number of low storage events per year
- **Low storage event**: Period where storage < 30% of capacity
- **Key**: Uses constrained (actual) storage
- **Grouping**: Consecutive low days = 1 event

**Calculation example**:
```
Water Year 2016: 2 spill events (Mar 5-8, Apr 15-20)
Water Year 2017: 1 spill event (Feb 10-12)
Water Year 2018: 0 spill events
---
Flood Indicator = 3 events / 3 years = 1.0 events/year
```

**Where to run**: Automatically calculated in mass balance function

**Returns**: Added to `indicators` dictionary:
- `flood_indicator` → Annual avg spill events
- `low_storage_exposure` → Annual avg low storage events
- `total_spill_events` → Total count
- `total_low_storage_events` → Total count

---

## ✅ TASK 6: Scatter Plot (Flood vs Low Storage)

**What to create**: Scatter plot with all 9 reservoirs

**Axes**:
- X-axis: Flood Indicator (spill events/year)
- Y-axis: Low Storage Exposure (low storage events/year)

**Visual elements**:
- Each dot = one reservoir
- Color-coded by reservoir
- Marker size = proportional to capacity
- Labels next to each point
- Quadrant lines at medians
- Quadrant annotations (Well Balanced, Both Extremes, etc.)
- Legend with capacities

**Function in notebook**:
- `plot_flood_vs_low_storage_scatter(indicators_df, save_path)`

**Where to run**: After mass balance analysis completes

**Output**: PNG file: `flood_vs_low_storage_scatter.png`

**Interpretation**:
- Bottom-left = Best (low flood, low drought risk)
- Top-right = Worst (high flood, high drought risk)
- Bottom-right = Flood-prone
- Top-left = Supply-stressed

---

## Execution Order

To complete all tasks, run cells in this order:

### Part 1: Tasks 1 & 2
```python
# 1. Import and setup (Cell 1)
# 2. Define utility functions (Cells 2-4)
# 3. Define indicator functions (Cells 5-9)
# 4. Define analysis functions (Cells 10-13)
# 5. Run main() to execute Tasks 1 & 2
results, summary = main()
```

**Creates**:
- CSV: `reservoir_summary.csv`
- PNG: `reservoir_indicators_analysis.png` (3 bar plots)

### Part 2: Tasks 3, 4, 5
```python
# 6. Define mass balance constants (Cell in Q2 section)
# 7. Define mass balance function (Cell with calculate_reservoir_mass_balance)
# 8. Run mass balance analysis
results_dict, summary_df, indicators_df = analyze_reservoir_mass_balance(
    data_directory=base_path,
    output_directory=output_path,
    initial_fraction=0.70
)
```

**Creates**:
- 9 individual plots: `{RESERVOIR}_mass_balance.png`
- 9-panel plot: `all_reservoirs_comparison.png` ✓ (Task 4)
- CSV: `reservoir_storage_summary.csv`
- CSV: `reservoir_indicators.csv` (with flood & low storage) ✓ (Task 5)

### Part 3: Task 6
```python
# 9. Create scatter plot
plot_flood_vs_low_storage_scatter(
    indicators_df,
    save_path=output_path + 'flood_vs_low_storage_scatter.png'
)
```

**Creates**:
- PNG: `flood_vs_low_storage_scatter.png` ✓ (Task 6)

---

## Files Generated (Complete List)

### From Tasks 1 & 2:
1. `reservoir_summary.csv` - All 5 indicators for all reservoirs
2. `reservoir_indicators_analysis.png` - 3 bar plots

### From Tasks 3, 4, 5:
3. `SHA_mass_balance.png` - Individual 4-panel plot
4. `ORO_mass_balance.png` - Individual 4-panel plot
5. `FOL_mass_balance.png` - Individual 4-panel plot
6. `NML_mass_balance.png` - Individual 4-panel plot
7. `DNP_mass_balance.png` - Individual 4-panel plot
8. `PNF_mass_balance.png` - Individual 4-panel plot
9. `CLE_mass_balance.png` - Individual 4-panel plot
10. `EXC_mass_balance.png` - Individual 4-panel plot
11. `BER_mass_balance.png` - Individual 4-panel plot
12. `all_reservoirs_comparison.png` - **9-panel subplot (Task 4)** ✓
13. `reservoir_storage_summary.csv` - Storage statistics
14. `reservoir_indicators.csv` - **Flood & low storage (Task 5)** ✓

### From Task 6:
15. `flood_vs_low_storage_scatter.png` - **Scatter plot (Task 6)** ✓

---

## Verification Checklist

Use this to verify all tasks are complete:

- [ ] **Task 1**: Can you find `calculate_indicator_a` through `calculate_indicator_e` functions?
- [ ] **Task 1**: Do they calculate % change or timing difference correctly?
- [ ] **Task 2**: Does `plot_indicators()` create 3 bar plots (a, b, e)?
- [ ] **Task 2**: Are all 9 reservoirs shown in each bar plot?
- [ ] **Task 3**: Does `calculate_reservoir_mass_balance()` exist?
- [ ] **Task 3**: Does it convert CFS to TAFD using 0.0019834711?
- [ ] **Task 3**: Does it start at 70% capacity?
- [ ] **Task 3**: Does it apply constraints (0 ≤ storage ≤ max)?
- [ ] **Task 4**: Is there code creating a 3×3 subplot grid?
- [ ] **Task 4**: Does it plot all 9 reservoirs' storage %?
- [ ] **Task 4**: Are axes labeled appropriately?
- [ ] **Task 5**: Does mass balance return flood_indicator?
- [ ] **Task 5**: Does it calculate spill events from unconstrained storage?
- [ ] **Task 5**: Does it group consecutive days into single events?
- [ ] **Task 5**: Does it return low_storage_exposure?
- [ ] **Task 5**: Does it check storage < 30% threshold?
- [ ] **Task 6**: Does `plot_flood_vs_low_storage_scatter()` exist?
- [ ] **Task 6**: X-axis = flood indicator, Y-axis = low storage?
- [ ] **Task 6**: Are reservoirs color-coded or labeled?

---

## Expected Results Summary

### Typical Indicator Values:

**Task 1 Indicators**:
- (a) 3-day peak: -10% to -36% (reservoirs reduce peaks)
- (b) 31-day low: -67% to +2228% (wide variation)
- (c) Peak timing: +26 to +76 days (delays)
- (d) Low timing: -26 to +97 days (varies)
- (e) RBI: -0.22 to +0.01 (mostly stabilize)

**Task 5 Indicators**:
- Flood indicator: 0 to 3 events/year (depends on capacity)
- Low storage: 0 to 2 events/year (depends on demand)

### What Makes Sense:
- All reservoirs reduce peak flows (negative %)
- Most reservoirs augment low flows (positive %)
- Most reduce flashiness (negative RBI)
- Timing shifts are typically positive (delays)
- Larger reservoirs → fewer spill events
- High demand reservoirs → more low storage events

---

## Common Issues & Solutions

### Issue: "Tuple unpacking error"
**Problem**: Old code expects single return, now returns tuple
**Solution**: Update to `result_df, indicators = calculate_reservoir_mass_balance(...)`

### Issue: "No module named X"
**Problem**: Missing required packages
**Solution**: Install: `pip install pandas numpy matplotlib`

### Issue: "File not found"
**Problem**: Data path incorrect
**Solution**: Update `base_path` variable to your data directory

### Issue: "KeyError: 'water_year'"
**Problem**: Data not preprocessed
**Solution**: Make sure `load_and_preprocess()` was called first

### Issue: Plot doesn't show all reservoirs
**Problem**: Some CSVs missing or misnamed
**Solution**: Check that all 9 CSV files exist: `{CODE}-daily-flows.csv`

---

## Final Confirmation

✅ All 6 tasks are implemented in the notebook
✅ Code is clean with minimal comments
✅ Separate explanation document provided (FUNCTION_EXPLANATIONS.md)
✅ All functions are properly defined and called
✅ Expected outputs will be generated when run

**Ready to execute!**
