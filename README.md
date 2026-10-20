# HistorianRCA: A Root Cause Analysis Benchmark

By Scott Duncan. Reach me on LinkedIn here: https://www.linkedin.com/in/scottduncanai/. View my website here: https://scottduncanai.com/

HistorianRCA is 90 days of simulated plant historian data from TK-101, a steam-heated agitated hold tank. The record contains abnormal events. Your job is to find each one, say when it started and ended, and identify its root cause.

The data is built for testing root cause analysis: by engineers, statistical methods, machine learning or AI agents.

## Process flow diagram

[![TK-101 process flow diagram: feed pump P-100, agitated tank TK-101 with submerged steam coil E-101, TIC-101 to FIC-201 cascade on TV-101, LIC-101 on the gravity outlet LV-101](tk-101-hold-tank.png)](tk-101-hold-tank.svg)

*Click the image for the full-resolution SVG. Conceptual drawing, not to scale. Full details are in `PROCESS_DESCRIPTION.md`.*

## What makes this dataset useful

- **Realistic historian behaviour.** Tags are noisy, autocorrelated, quantized, compressed with swinging-door compression, and stored at irregular timestamps. The record also has communication outages, stuck readings, single-sample spikes, slow calibration drift, and weather and seasonal effects.
- **Real control behaviour.** The unit runs a temperature-to-steam-flow cascade and a level loop. The controllers hide many faults: temperature and level stay at setpoint while the fault shows up somewhere else.
- **Known answers.** Every fault has one known root cause and known start and end times, held in a private answer key.
- **Distractors.** The record includes deliberate operator actions and data-quality artifacts that are not faults. It also includes tags from four neighbouring units that share only the weather with TK-101.

## Everything you need is in the data

There is no alarm journal, operator log, shift note, maintenance record, work order or lab result. You won't need them.

**Every root cause can be determined from the historian data and the process description alone**, using chemical engineering first principles: mass and energy balances, heat transfer, steam properties, control loop behaviour, and comparison of redundant or related instruments. No outside information, plant experience or guesswork about unrecorded events is required.

Read `PROCESS_DESCRIPTION.md` before the data. It defines the equipment, the physics the unit obeys, the control strategy, every tag and calculation, and how the instruments and historian behave. The diagnosis rests on those relationships.

## Files

| File | Content |
|---|---|
| `PROCESS_DESCRIPTION.md` | Unit, physics, control strategy, instruments, tag definitions, historian behaviour. **Start here.** |
| `CHALLENGE.md` | The task, what counts as an event, submission format, scoring and rules. |
| `historian.parquet` | Primary record (~33 MB). Long format: `tag`, `timestamp` (UTC), `value` (float32), `quality` (0 good, 1 uncertain, 2 bad). Compressed per tag, irregular timestamps. |
| `historian_60s.parquet` | The same data on a regular 1-minute grid (~25 MB), one column per tag. Linear interpolation between stored points; bad-quality samples are `NaN`. Convenient, but it inherits compression error. Check the primary record before trusting a short-lived feature. |
| `tags.csv` | Tag dictionary: description, units, range, instrument class, role, loop, DCS node, transport delay, compression deadband, sensor lag, update period, owning unit. |
| `submission_template.csv` | Required answer format. |
| `tk-101-hold-tank.svg` / `.png` | Process flow diagram (shown above). |
| `LICENSE` / `CITATION.cff` | Licence terms and how to cite the dataset. |

## Data at a glance

| Item | Value |
|---|---|
| Period | 2024-03-01 06:00 to 2024-05-30 06:00 UTC (90 days) |
| Plant local time | America/Chicago; the period includes the spring daylight-saving change |
| Tags | 90 total: 30 on TK-101, 60 on neighbouring units (`CW-*`, `N2-*`, `SD-*`, `AIR-*`) |
| DCS scan | 10 s |
| Reference period | The first 14 days are normal operation and can be used as a baseline |

All timestamps are UTC. Daily cycles in ambient temperature and steam header pressure follow plant local time.

## The task in brief

For each **fault event**, report:

1. Start time (UTC): when the cause began acting on the unit
2. End time (UTC): when the cause was removed
3. Root cause, in your own words: which item failed and how. There is no list of causes to choose from.
4. Evidence: the tags that support the diagnosis

Key rules:

- Each fault has exactly one root cause. Faults do not overlap.
- Some faults develop gradually, so a cause can act for a while before its effects are visible.
- **The following are not faults**, and reporting them as faults costs points:
  - Deliberate operator actions to which the unit responds normally
  - Instrument and historian artifacts: stuck readings, single-sample spikes, DCS communication outages
  - Weather and seasonal effects, and slow transmitter calibration drift
  - Anything on the neighbouring units
- The number of events is not stated.

Scoring: +1 for each fault detected, +2 more if the cause is correct, −1 for each false positive. Timing precision is used only as a tie-breaker. Full details are in `CHALLENGE.md`.

There is no deadline. To have a submission scored, send your CSV and a short note on your method to Scott Duncan on LinkedIn (https://www.linkedin.com/in/scottduncanai/). Please don't post submissions publicly, since that would give the answers away to others.

## Quickstart

```python
import pandas as pd

wide = pd.read_parquet("historian_60s.parquet").set_index("timestamp")
tags = pd.read_csv("tags.csv")

# TK-101 tags only
tk = wide[tags.loc[tags.unit == "TK-101", "tag"]]
tk[["TIC-101.PV", "FIC-201.PV", "TV-101.POS", "LIC-101.PV"]].resample("1h").mean().plot(subplots=True)

# Primary (compressed) record for one tag
raw = pd.read_parquet("historian.parquet")
steam = raw[(raw.tag == "FIC-201.PV") & (raw.quality == 0)]
```

Requires `pandas` and `pyarrow`.

## Answer key

The answer key is withheld so the dataset stays usable as a blind test. For each event it gives the true start and end times, the cause, and the tags that confirm the cause and rule out the alternatives. To request it (for example for teaching, or after you have submitted), contact Scott Duncan on LinkedIn (https://www.linkedin.com/in/scottduncanai/).

## Sharing your approach

Any method is allowed: manual analysis, statistics, machine learning, AI agents or a mix. If you work through the challenge, please post a short write-up of your method in this repository's Discussions tab, without giving away the answers. Approaches that found every event, and approaches that missed some, are both useful to others.

## Limitations

- The data is simulated. The simulator models the process physics, control system and instrument behaviour, but it is one unit with one product and a fixed design.
- Each fault has a single root cause. Real incidents often have several contributing causes.
- There are no alarms, events or human records. This is deliberate: the test is what can be deduced from process data and engineering principles alone.

## License and citation

Licensed under [CC BY 4.0](LICENSE): you may share and adapt the dataset for any purpose, with attribution.

If you use this dataset, please cite it as described in [`CITATION.cff`](CITATION.cff).

Created by Scott Duncan. Version 1.0.
