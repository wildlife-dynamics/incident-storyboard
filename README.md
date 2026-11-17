# Incident Storyboard

An EarthRanger workflow that downloads **incidents** (ER's collections of related events) with **all of their events**, and shows each incident as a **storyboard**: its events in time order, as event-type icons framed in their event-type colour, with the time between events on the arrows.

It is one output of the incident report, split out on its own so it can be combined with other outputs elsewhere.

## Output

| Output | Shows |
|---|---|
| Incident storyboards (widget) | One row per incident, newest first. Same-type events close in time share one tile (×N). Unlocated events are marked "no location". Long type names show in full on hover. |
| `incident_events.parquet` (file) | One row per (incident, event), with the incident's columns, display names, event details, `icon` and `event_type_color`. GeoParquet, listed in the **Files** tab. |

Every event type has one colour, the same in every view: the most common type takes the first of 16 set colours (`color_map_by_frequency`, then `apply_color_map`). Workflows split out of the incident report use the same list and order, so their colours agree.

## Parameters

| Parameter | Notes |
|---|---|
| Data Source | EarthRanger connection |
| Time Range | Selects incidents by their own date. **All** of an incident's events are included, even those dated outside the range. Times are shown in the range's timezone. |
| Group by | Optional. **Incident** gives one dashboard view per incident. |
| Merge window (advanced) | Same-type events this many hours apart or less share one tile. Default 12; 0 never merges. |

## How it works

```
get_incident_events          incidents in the range → their events by id, display names, details, icons
  → convert_values_to_timezone
  → color_map_by_frequency (event_type_display → {type: colour}, most common first)
  → apply_color_map (→ event_type_color)
  → persist_df (incident_events.parquet)
  → split_groups (Incident, or all)
  → draw_incident_storyboard (color_column: event_type_color) → persist_text → plot widget
  → gather_dashboard
```

Every task is from `ecoscope-platform` (≥ 2.35.0). Nested incidents are followed up to 5 levels deep and event icons are always on (`max_depth: 5`, `include_icons: true`, fixed in `spec.yaml`).

## Local development

```bash
make compile          # compile spec.yaml and install the workflow env
make run-mock         # run on mock-io/ data and preview the dashboard
make run              # run on real data (param.yaml)
make test CASE=wd-by-incident
```

To try unreleased ecoscope changes, compile against the released ecoscope-platform and put a local checkout first on the path (an editable pypi install of ecoscope conflicts with the conda solve's sqlalchemy):

```bash
PYTHONPATH=../ecoscope make run-mock
```

Mock data: `mock-io/get-incident-events.example-return.parquet` (Wildlife Dynamics site, 4 incidents, 8 events), used through `WT_TASK_MOCK_IO__ECOSCOPE_PLATFORM_TASKS_IO_GET_INCIDENT_EVENTS` in the Makefile.
