# Data Dictionary - Production Records

## Dataset Overview
- **Source**: Manufacturing ERP System
- **Format**: CSV (Comma-Separated Values)
- **Update Frequency**: Hourly snapshots
- **Date Range**: January 1, 2026 – March 4, 2026
- **Records**: 500 hourly production snapshots

## Column Specifications

| Column Name | Data Type | Description | Example Values | Constraints |
|---|---|---|---|---|
| `timestamp` | datetime | Date and time of the record (YYYY-MM-DD HH:MM:SS) | `2026-01-15 08:00:00` | Non-null, Primary key |
| `machine_id` | string | Unique identifier for the production machine | `M1`, `M2`, `M3`, `M4`, `M5` | Non-null, 5 machines |
| `shift` | string | Production shift identifier | `A`, `B`, `C` | Non-null, 3 shifts |
| `product_type` | string | Product category being manufactured | `Widget`, `Gadget`, `Component` | Non-null, 3 types |
| `planned_runtime_hours` | float | Scheduled operating hours for the period | `1.0`, `0.5` | Non-null, > 0 |
| `actual_runtime_hours` | float | Actual hours the machine operated | `0.87`, `0.45` | Non-null, ≥ 0 |
| `downtime_hours` | float | Hours of unplanned downtime | `0.13`, `0.05` | Non-null, ≥ 0 |
| `downtime_reason` | string | Primary cause of downtime | `Breakdown`, `Material Shortage`, `Changeover`, `Quality Hold`, `None` | Nullable |
| `units_produced` | int | Number of units completed | `142`, `78` | Non-null, ≥ 0 |
| `target_units` | int | Planned production quantity | `150`, `100` | Non-null, > 0 |
| `defect_count` | int | Number of defective units | `3`, `0` | Non-null, ≥ 0 |
| `ideal_cycle_time_sec` | float | Standard time per unit (seconds) | `24.0`, `36.0` | Non-null, > 0 |
| `oee_percent` | float | Overall Equipment Effectiveness (%) | `67.2`, `82.5` | Calculated, 0-100 |
| `availability_percent` | float | Availability component of OEE (%) | `87.0`, `90.0` | Calculated, 0-100 |
| `performance_percent` | float | Performance component of OEE (%) | `94.7`, `78.0` | Calculated, 0-100 |

## Derived Metrics (Calculated Columns)

### OEE Formula
```
OEE = Availability × Performance × Quality
```

Where:
- **Availability** = `actual_runtime_hours / planned_runtime_hours`
- **Performance** = `(units_produced × ideal_cycle_time_sec / 3600) / actual_runtime_hours`
- **Quality** = `(units_produced - defect_count) / units_produced`

### Key Performance Indicators

| KPI | Calculation | Interpretation |
|---|---|---|
| **Downtime %** | `(downtime_hours / planned_runtime_hours) × 100` | Lower is better; target <10% |
| **Defect Rate %** | `(defect_count / units_produced) × 100` | Lower is better; target <2% |
| **Throughput** | `units_produced / actual_runtime_hours` | Units per hour |
| **Utilization** | `actual_runtime_hours / planned_runtime_hours` | Machine usage efficiency |

## Data Quality Notes

- **Missing Values**: ~71.8% of records initially had null `downtime_reason` — resolved via machine-level mode imputation (M3: Cleaning module)
- **Outliers**: Some records show performance exceeding 105% of standard cycle time
- **Validation**: All numeric columns enforced non-negative; OEE capped at 100%

## Business Context

- **Target OEE**: 85% (World-class manufacturing standard)
- **Current OEE**: ~67% (as of March 2026 analysis)
- **Primary Bottleneck**: 68% of downtime is unplanned equipment breakdowns
- **Shift Performance**: Shift A outperforms B and C by 5-7 OEE points
