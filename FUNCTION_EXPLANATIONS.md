# A2 Notebook - Complete Function Explanations

## Overview

This document provides detailed explanations for every function in the A2 notebook. The notebook solves all 6 required tasks for reservoir hydrological analysis.

---

## Task Completion Checklist

### ✅ Task 1: Calculate Inflow/Outflow Indicators (a-e)
- **Functions**: `calculate_indicator_a()`, `calculate_indicator_b()`, `calculate_indicator_c()`, `calculate_indicator_d()`, `calculate_indicator_e()`
- **Status**: Complete

### ✅ Task 2: Bar Plots for Indicators (a), (b), (e)
- **Function**: `plot_indicators()`
- **Status**: Complete - Creates 3 bar plots, one per indicator

### ✅ Task 3: Mass Balance Function and Storage Timeseries
- **Function**: `calculate_reservoir_mass_balance()`
- **Status**: Complete - Returns storage timeseries

### ✅ Task 4: 9-Panel Storage Volume Plot
- **Function**: `plot_storage_comparison()` (within analyze function)
- **Status**: Complete - Creates 3x3 subplot with all reservoirs

### ✅ Task 5: Flood Indicator and Low Storage Exposure
- **Function**: Enhanced `calculate_reservoir_mass_balance()`
- **Status**: Complete - Returns flood and low storage metrics

### ✅ Task 6: Scatter Plot (Flood vs Low Storage)
- **Function**: `plot_flood_vs_low_storage_scatter()`
- **Status**: Complete - Creates scatter plot with all reservoirs

---

## Utility Functions

### `circular_day_difference(day1, day2, year_length=365)`

**Purpose**: Calculate the shortest distance between two days of the water year, accounting for the circular nature of the calendar.

**Why Needed**: When comparing timing of events (like peak flow on day 360 vs day 10), we need to recognize these are only 15 days apart, not 350 days apart.

**How It Works**:
1. Calculate raw difference: `day2 - day1`
2. If difference > 182 days (half a year), take the shorter path around the circle
3. Return the smallest absolute distance

**Example**:
- Day 360 to Day 10: Raw = -350 days, Circular = +15 days
- Day 50 to Day 200: Raw = +150 days, Circular = +150 days (unchanged)

**Used By**: `calculate_indicator_c()`, `calculate_indicator_d()` for timing calculations

---

### `get_day_of_water_year(date)`

**Purpose**: Convert a calendar date to the day number within the water year (1-365).

**Water Year Definition**: Runs from October 1 to September 30
- October 1 = Day 1
- December 31 = Day 92
- January 1 = Day 93
- September 30 = Day 365

**How It Works**:
1. Determine the water year start date based on the month
2. If date is Oct-Dec: water year started Oct 1 of same calendar year
3. If date is Jan-Sep: water year started Oct 1 of previous calendar year
4. Calculate days elapsed since water year start

**Example**:
- March 15, 2017 → Day 166 of Water Year 2017
- November 20, 2017 → Day 51 of Water Year 2018

**Used By**: All timing indicator calculations

---

### `add_water_year(df)`

**Purpose**: Add a 'water_year' column to the dataframe.

**How It Works**:
- For dates in Jan-Sep: water_year = calendar year
- For dates in Oct-Dec: water_year = calendar year + 1

**Example**:
- December 15, 2016 → Water Year 2017
- March 1, 2017 → Water Year 2017

**Used By**: All preprocessing and indicator calculations

---

### `load_and_preprocess(filepath)`

**Purpose**: Load reservoir CSV data and prepare it for analysis.

**Steps**:
1. **Load CSV**: Read the data file
2. **Rename columns**: Standardize to 'date', 'inflow', 'outflow'
3. **Convert dates**: Parse datetime strings to pandas Timestamps
4. **Remove Feb 29**: Eliminate leap day to maintain 365-day water years
5. **Add water year**: Calculate water year for each date
6. **Sort by date**: Ensure chronological order
7. **Reset index**: Clean up row indices

**Why Remove Feb 29**: Ensures all water years have exactly 365 days, making year-to-year comparisons consistent and simplifying day-of-year calculations.

**Returns**: Clean DataFrame ready for analysis

---

## Task 1: Indicator Calculation Functions

### `calculate_indicator_a(df)`

**Purpose**: Calculate percent change in 3-day peak flow between inflow and outflow.

**What It Measures**: How much the reservoir reduces (or increases) the magnitude of peak flow events.

**Calculation Steps**:
1. **For each water year**:
   - Calculate 3-day rolling average of inflow
   - Find maximum value (peak inflow)
   - Calculate 3-day rolling average of outflow
   - Find maximum value (peak outflow)
   
2. **Calculate percent change**:
   ```
   % Change = 100 × (out_peak - in_peak) / in_peak
   ```

3. **Average across all water years**

**Interpretation**:
- **Negative values**: Reservoir reduces peaks (flood control)
- **Positive values**: Reservoir increases peaks (rare)
- Example: -28.7% means outflow peaks are 28.7% lower than inflow peaks

**Physical Meaning**: 
- Large negative values indicate aggressive flood control
- Small negative values suggest run-of-river operations
- This metric reveals the reservoir's flood attenuation effectiveness

**Returns**: DataFrame with columns: `water_year`, `in_3day_peak`, `out_3day_peak`, `pct_change`

---

### `calculate_indicator_b(df)`

**Purpose**: Calculate percent change in 31-day low flow between inflow and outflow.

**What It Measures**: How much the reservoir augments (increases) or reduces low flows during dry periods.

**Calculation Steps**:
1. **For each water year**:
   - Calculate 31-day rolling average of inflow
   - Find minimum value (lowest inflow period)
   - Calculate 31-day rolling average of outflow
   - Find minimum value (lowest outflow period)

2. **Calculate percent change**:
   ```
   % Change = 100 × (out_low - in_low) / in_low
   ```

3. **Average across all water years**

**Interpretation**:
- **Positive values**: Reservoir augments low flows (water supply benefit)
- **Negative values**: Reservoir reduces low flows (water extraction exceeds natural flow)
- Example: +61.8% means reservoir provides 61.8% more flow during dry periods

**Physical Meaning**:
- Positive values show the reservoir storing winter/spring runoff for summer release
- Negative values may indicate high water demands or evaporation losses
- Critical for maintaining downstream ecological flows and water supply

**Returns**: DataFrame with columns: `water_year`, `in_31day_low`, `out_31day_low`, `pct_change`

---

### `calculate_indicator_c(df)`

**Purpose**: Calculate timing shift of 3-day peak flow between inflow and outflow.

**What It Measures**: How many days the reservoir delays (or advances) peak flow events.

**Calculation Steps**:
1. **For each water year**:
   - Calculate 3-day rolling average for inflow and outflow
   - Find date when inflow peaks
   - Find date when outflow peaks
   - Convert both dates to day-of-water-year (1-365)

2. **Calculate circular difference**:
   ```
   Timing Difference = circular_day_difference(inflow_day, outflow_day)
   ```

3. **Average across all water years**

**Interpretation**:
- **Positive values**: Outflow peaks occur later than inflow peaks (delay)
- **Negative values**: Outflow peaks occur earlier than inflow peaks (advance)
- Example: +25.8 days means outflow peaks about 26 days after natural peaks

**Physical Meaning**:
- Reservoirs typically delay peaks (positive values) for flood control
- The delay allows time to gradually release stored water
- Important for fish spawning cues and floodplain ecology

**Returns**: DataFrame with columns: `water_year`, `in_timing`, `out_timing`, `timing_diff`

---

### `calculate_indicator_d(df)`

**Purpose**: Calculate timing shift of 31-day low flow between inflow and outflow.

**What It Measures**: How many days the reservoir shifts the annual low flow period.

**Calculation Steps**:
1. **For each water year**:
   - Calculate 31-day rolling average for inflow and outflow
   - Find date when inflow is lowest
   - Find date when outflow is lowest
   - Convert both to day-of-water-year

2. **Calculate circular difference**:
   ```
   Timing Difference = circular_day_difference(inflow_day, outflow_day)
   ```

3. **Average across all water years**

**Interpretation**:
- **Positive values**: Low flows occur later in outflow than inflow
- **Negative values**: Low flows occur earlier in outflow
- Example: +48.2 days means the driest period is delayed by 48 days

**Physical Meaning**:
- Natural low flows typically occur late summer (Aug-Sep)
- Reservoir operations may shift this to fall (Oct-Nov)
- Important for understanding when downstream water stress occurs

**Returns**: DataFrame with columns: `water_year`, `in_timing`, `out_timing`, `timing_diff`

---

### `calculate_indicator_e(df)`

**Purpose**: Calculate change in flashiness (Richards-Baker Index) between inflow and outflow.

**What It Measures**: How much the reservoir stabilizes (reduces) or destabilizes (increases) day-to-day flow variability.

**Richards-Baker Index (RBI) Formula**:
```
RBI = Σ|Q_i - Q_{i-1}| / ΣQ_i
```
Where:
- Numerator: Sum of absolute daily flow changes
- Denominator: Total flow volume (normalization)

**What RBI Captures**:
1. **Frequency of changes**: How often flow varies
2. **Magnitude of changes**: How much flow varies
3. **Overall stability**: Low RBI = stable, High RBI = flashy

**RBI Interpretation Scale**:
- 0.00-0.10: Very stable (heavily regulated)
- 0.10-0.20: Moderately stable
- 0.20-0.40: Natural variability
- 0.40+: Highly flashy (hydropeaking or urban runoff)

**Calculation Steps**:
1. **For each water year**:
   - Calculate RBI for inflow stream
   - Calculate RBI for outflow stream
   - Compute difference: `RBI_out - RBI_in`

2. **Average across all water years**

**Interpretation**:
- **Negative difference**: Reservoir reduces flashiness (stabilizes flows)
- **Positive difference**: Reservoir increases flashiness (may indicate hydropeaking)
- Example: -0.1104 means significant flow stabilization

**Physical Meaning**:
- Negative values benefit benthic communities (stable substrate)
- But excessive stabilization removes natural flow variability that native species need
- Positive values may stress aquatic organisms (rapid flow changes)

**Returns**: DataFrame with columns: `water_year`, `in_rbi`, `out_rbi`, `rbi_diff`

---

## Task 2: Bar Plot Function

### `plot_indicators(summary_df, output_dir)`

**Purpose**: Create three bar plots showing indicator values for all reservoirs.

**Creates Three Separate Plots**:
1. **Plot (a)**: 3-day peak flow % change for each reservoir
2. **Plot (b)**: 31-day low flow % change for each reservoir
3. **Plot (e)**: RBI difference for each reservoir

**Visual Features**:
- Each reservoir gets a different color
- Horizontal reference line at y=0 (no change)
- Value labels on each bar
- Grid for easy reading
- Professional formatting

**How It Works**:
1. Create 3-panel figure (1 row, 3 columns)
2. For each indicator:
   - Extract values from summary dataframe
   - Create bar chart with reservoir names on x-axis
   - Add reference line and labels
   - Format axes and title

**Interpretation Guide**:
- Bars below zero (negative) = reduction
- Bars above zero (positive) = increase
- Length of bar = magnitude of change
- Compare bars to see which reservoirs have strongest alterations

**Saves**: One PNG file with all three plots: `reservoir_indicators_analysis.png`

---

## Task 3: Mass Balance Function

### `calculate_reservoir_mass_balance(df, max_storage_tafd, initial_storage_fraction=0.70, ...)`

**Purpose**: Simulate daily reservoir storage using the water balance equation.

**Mass Balance Equation**:
```
Storage(t+1) = Storage(t) + Inflow(t) - Outflow(t) - Evaporation(t) - Seepage(t)
```

**Simplified Version (per assignment)**:
```
Storage(t+1) = Storage(t) + Inflow(t) - Outflow(t)
```

**Unit Conversion**:
- Input flows are in CFS (cubic feet per second)
- Convert to TAFD (thousand acre-feet per day):
  ```
  CFS_TO_TAFD = 2.29568411e-5 × 86400 / 1000 = 0.0019834711
  ```

**Calculation Steps**:

1. **Unit Conversion**:
   - Convert inflow from CFS to TAFD
   - Convert outflow from CFS to TAFD
   - Calculate net daily change = inflow - outflow

2. **Initialize Storage**:
   - Set initial storage = 70% of maximum capacity
   - Create arrays to store daily values

3. **Daily Mass Balance Loop**:
   ```
   For each day i from 1 to N:
       unconstrained_storage[i] = storage[i-1] + net_change[i]
       
       Apply physical constraints:
       If unconstrained_storage < 0:
           storage[i] = 0  (cannot be negative)
       Else if unconstrained_storage > max_capacity:
           storage[i] = max_capacity  (spillway prevents overflow)
       Else:
           storage[i] = unconstrained_storage
   ```

4. **Track Events** (for Task 5):
   - **Spill events**: Days where unconstrained storage > max capacity
   - **Low storage events**: Days where storage < 30% capacity
   - Group consecutive days into single events

5. **Calculate Indicators**:
   - Count total spill events
   - Count total low storage events
   - Divide by number of years for annual averages

**Physical Constraints**:

**Lower Bound (Storage ≥ 0)**:
- Cannot have negative storage (physically impossible)
- If outflow exceeds available water, storage = 0
- Indicates water supply shortage

**Upper Bound (Storage ≤ Max Capacity)**:
- Cannot store more than reservoir capacity
- Excess water spills over dam
- Indicates flood conditions

**Key Innovation for Task 5**:
- Tracks **unconstrained storage** separately
- This reveals when storage *would* exceed capacity
- Used to identify actual spill events before constraint is applied
- Critical for calculating flood indicator accurately

**Returns**: Tuple of (DataFrame, indicators_dict)

**DataFrame columns**:
- `inflow_tafd`, `outflow_tafd`: Flows in thousand acre-feet/day
- `net_change_tafd`: Daily change in storage
- `storage_unconstrained`: Storage before applying limits (for flood detection)
- `storage_tafd`: Actual storage after applying constraints
- `storage_percent`: Storage as % of maximum capacity
- `spill_event`: Boolean flag for spill event days
- `low_storage_event`: Boolean flag for low storage days
- `constraint_violation`: Boolean flag when constraints are hit

**Indicators dictionary**:
- `flood_indicator`: Average annual spill events
- `low_storage_exposure`: Average annual low storage events
- `total_spill_events`: Total count over all years
- `total_low_storage_events`: Total count over all years
- `n_years`: Number of water years analyzed
- `spill_events_list`: Details of each spill event
- `low_storage_events_list`: Details of each low storage event

---

## Supporting Functions for Mass Balance

### `plot_storage_timeseries(df, reservoir_name, max_storage_tafd, ...)`

**Purpose**: Create comprehensive 4-panel visualization of reservoir storage dynamics.

**Four Panels**:

**Panel (a): Inflow and Outflow Timeseries**
- Shows daily inflow (blue) and outflow (red) in CFS
- Reveals seasonal patterns and operational responses
- Helps identify when inflows spike or drop

**Panel (b): Storage Volume Timeseries**
- Shows daily storage in thousand acre-feet (TAF)
- Horizontal line at maximum capacity (red dashed)
- Scatter points mark constraint violations
- Shows filling and draining cycles

**Panel (c): Storage as Percentage of Capacity**
- Shows storage as % of maximum (0-100%)
- Reference lines at 100% (full) and 50% (half-full)
- Shaded area under the curve
- Easier to compare across different sized reservoirs

**Panel (d): Daily Storage Change**
- Bar chart of daily net change (inflow - outflow)
- Green bars = gaining storage
- Red bars = losing storage
- Shows operational patterns and inflow events

**Visual Features**:
- Consistent color scheme across panels
- Violation days highlighted in red
- Professional formatting with labels and grids
- High resolution (300 dpi) output

**Saves**: Individual PNG file for each reservoir

---

### `calculate_storage_statistics(df, reservoir_name, max_storage)`

**Purpose**: Compute summary statistics for reservoir storage performance.

**Calculates**:

**Storage Metrics**:
- Initial storage (TAF and %)
- Final storage (TAF and %)
- Mean storage (TAF and %)
- Minimum storage (TAF and %)
- Maximum storage (TAF and %)
- Storage range (max - min)

**Operational Metrics**:
- Days empty (storage = 0)
- Days full (storage = max capacity)
- Constraint violations count
- Mean daily change
- Total inflow volume
- Total outflow volume

**Returns**: Dictionary with all statistics

**Used By**: Analysis function to create summary tables and reports

---

### `analyze_reservoir_mass_balance(data_directory, output_directory, initial_fraction=0.70)`

**Purpose**: Process all reservoirs in the data directory and generate complete analysis.

**Workflow**:

1. **Setup**:
   - Print analysis header with parameters
   - Find all CSV files in data directory
   - Initialize result storage

2. **For Each Reservoir**:
   - Load and preprocess data
   - Run mass balance calculation
   - Store results and indicators
   - Calculate statistics
   - Create individual 4-panel plot
   - Print progress and results

3. **Create Summary Outputs**:
   - Compile all statistics into summary DataFrame
   - Compile all indicators into indicators DataFrame
   - Save both to CSV files

4. **Generate Comparison Plot** (Task 4):
   - Create 3×3 subplot grid (9 panels)
   - Plot each reservoir's storage % timeseries
   - Reference lines at 50% and 100%
   - Consistent axes for comparison
   - Save as single PNG file

5. **Print Summary Tables**:
   - Storage statistics for all reservoirs
   - Flood and low storage indicators
   - Final completion message

**Returns**: Tuple of (results_dict, summary_df, indicators_df)

- `results_dict`: Dictionary mapping reservoir codes to full DataFrames
- `summary_df`: Summary statistics for all reservoirs
- `indicators_df`: Flood and low storage metrics for all reservoirs

**Files Created**:
- Individual plots: `{RESERVOIR}_mass_balance.png` (9 files)
- Comparison plot: `all_reservoirs_comparison.png` (Task 4 output)
- Summary CSV: `reservoir_storage_summary.csv`
- Indicators CSV: `reservoir_indicators.csv`

---

## Task 4: 9-Panel Comparison Plot

**Created By**: `analyze_reservoir_mass_balance()` function (embedded subplot code)

**Purpose**: Show all 9 reservoirs' storage dynamics in a single figure for easy comparison.

**Layout**:
```
┌─────────┬─────────┬─────────┐
│  SHA    │  ORO    │  FOL    │
├─────────┼─────────┼─────────┤
│  NML    │  DNP    │  PNF    │
├─────────┼─────────┼─────────┤
│  CLE    │  EXC    │  BER    │
└─────────┴─────────┴─────────┘
```

**Each Panel Shows**:
- Storage as % of capacity (y-axis)
- Time (x-axis)
- Filled area under curve (shaded blue)
- Reference lines at 100% (red) and 50% (orange)
- Reservoir name and capacity in title

**Visual Features**:
- Consistent y-axis limits (0-110%)
- Same time period for all panels
- Color-coded reference lines
- Grid for easier reading
- Title with reservoir code and capacity

**Interpretation**:
- Compare filling/draining patterns across reservoirs
- Identify which reservoirs frequently reach capacity
- See which reservoirs struggle with low storage
- Observe seasonal patterns (similar or different?)

**Saves**: `all_reservoirs_comparison.png`

---

## Task 6: Scatter Plot Function

### `plot_flood_vs_low_storage_scatter(indicators_df, save_path)`

**Purpose**: Visualize the relationship between flood risk and low storage risk for all reservoirs.

**Axes**:
- **X-axis**: Flood Indicator (average annual spill events)
- **Y-axis**: Low Storage Exposure (average annual low storage events)

**Visual Features**:

**1. Color Coding**:
- Each reservoir has unique color:
  - SHA: Red, ORO: Orange, FOL: Yellow
  - NML: Teal, DNP: Blue, PNF: Purple
  - CLE: Violet, EXC: Pink, BER: Green

**2. Marker Size**:
- Proportional to reservoir capacity
- Larger reservoirs = bigger dots
- Helps identify capacity effects visually

**3. Labels**:
- Reservoir code displayed next to each point
- Color-matched background for clarity
- Easy to identify all reservoirs

**4. Quadrant Lines**:
- Vertical line at median flood indicator
- Horizontal line at median low storage
- Divides plot into four operational categories

**5. Quadrant Annotations**:
- **Top-Right**: High Flood & High Low Storage (Both Extremes)
- **Top-Left**: Low Flood & High Low Storage (Supply Stressed)
- **Bottom-Right**: High Flood & Low Low Storage (Flood-Prone)
- **Bottom-Left**: Low Flood & Low Low Storage (Well Balanced) ✓

**6. Legend**:
- Lists all reservoirs with capacities
- Shows color coding
- Positioned outside main plot area

**Interpretation Guide**:

**Quadrant Meanings**:

1. **Bottom-Left (Well Balanced)**:
   - Low flood risk AND low drought risk
   - Best operational performance
   - Adequate capacity for variability
   - Effective management

2. **Top-Right (Both Extremes)**:
   - High flood risk AND high drought risk
   - Experiencing both problems
   - May need infrastructure upgrades
   - Operational challenges

3. **Bottom-Right (Flood-Prone)**:
   - Frequent spilling but maintains storage
   - Conservative flood control
   - May have excess capacity
   - Could optimize releases

4. **Top-Left (Supply-Stressed)**:
   - Rarely spills but often runs low
   - High water demand
   - Drought vulnerability
   - May need supplemental supplies

**Saves**: `flood_vs_low_storage_scatter.png`

---

## Additional Analysis Functions

### `analyze_reservoir(reservoir_file)`

**Purpose**: Calculate all 5 indicators (a-e) for a single reservoir.

**Steps**:
1. Load and preprocess data
2. Calculate each indicator sequentially
3. Compute average values across water years
4. Return dictionary with all results

**Returns**: Dictionary with averages for all 5 indicators

---

### `analyze_multiple_reservoirs(data_directory)`

**Purpose**: Apply indicator analysis to all reservoirs in directory.

**Steps**:
1. Find all CSV files
2. For each file:
   - Extract reservoir code from filename
   - Run `analyze_reservoir()`
   - Store results
3. Compile all results

**Returns**: Dictionary mapping reservoir codes to their indicator results

---

### `create_summary_table(all_results)`

**Purpose**: Convert analysis results into clean DataFrame for reporting.

**Creates Columns**:
- Reservoir code
- Average values for indicators (a), (b), (c), (d), (e)
- Properly formatted and labeled

**Returns**: DataFrame suitable for printing and saving to CSV

---

## Constants and Configurations

### `CFS_TO_TAFD = 0.0019834711`

**Purpose**: Unit conversion factor from cubic feet per second to thousand acre-feet per day.

**Derivation**:
```
1 cfs = 2.29568411e-5 acre-feet per second
1 day = 86400 seconds
1 thousand = 1000

CFS_TO_TAFD = 2.29568411e-5 × 86400 / 1000
            = 0.0019834711
```

**Example**:
- 1000 cfs for 1 day = 1.983 thousand acre-feet

---

### `RESERVOIR_CAPACITIES` (Dictionary)

**Purpose**: Store maximum storage capacity for each California reservoir.

**Values** (in thousand acre-feet):
- SHA (Shasta): 4,552 TAF - Largest reservoir in California
- ORO (Oroville): 3,538 TAF - Second largest
- FOL (Folsom): 977 TAF
- NML (New Melones): 2,400 TAF
- DNP (Don Pedro): 2,030 TAF
- PNF (Pine Flat): 1,000 TAF
- CLE (Clear Lake): 525 TAF - Smallest in dataset
- EXC (Exchequer): 1,025 TAF
- BER (Berryessa): 1,602 TAF

**Source**: California Data Exchange Center (CDEC) and Bureau of Reclamation

**Used By**: All mass balance calculations to apply upper constraint

---

## Main Execution Workflow

### Task 1 & 2 Execution:
```python
# Analyze all reservoirs for indicators
all_results = analyze_multiple_reservoirs(data_directory)

# Create summary table
summary_df = create_summary_table(all_results)

# Generate bar plots (Task 2)
plot_indicators(summary_df, output_directory)
```

### Task 3, 4, 5 Execution:
```python
# Run mass balance analysis (Tasks 3, 4, 5)
results_dict, summary_df, indicators_df = analyze_reservoir_mass_balance(
    data_directory=base_path,
    output_directory=output_path,
    initial_fraction=0.70
)
```

This creates:
- Individual 4-panel plots (Task 3)
- 9-panel comparison (Task 4)
- Flood and low storage indicators (Task 5)

### Task 6 Execution:
```python
# Create scatter plot
plot_flood_vs_low_storage_scatter(
    indicators_df,
    save_path=output_path
)
```

---

## Summary: Complete Task Coverage

| Task | Function(s) | Output |
|------|------------|--------|
| 1 | calculate_indicator_a/b/c/d/e | 5 indicators calculated |
| 2 | plot_indicators | 3 bar plots in 1 figure |
| 3 | calculate_reservoir_mass_balance | Storage timeseries |
| 4 | analyze_reservoir_mass_balance | 9-panel subplot |
| 5 | Enhanced mass balance function | Flood & low storage metrics |
| 6 | plot_flood_vs_low_storage_scatter | Scatter plot |

All 6 tasks are fully implemented and functional in the notebook.
