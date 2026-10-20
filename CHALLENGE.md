# HistorianRCA: TK-101 Root Cause Challenge

Ninety days of historian data from a steam-heated hold tank. Over that
period, things go wrong. Your job is to find each abnormal event, say when
it happened, and identify its root cause.

The data is simulated, but the simulator models the process physics, the
control system and the instruments' behaviour, so the record looks like a
real plant historian: noisy, compressed, with gaps, stuck readings and
weather effects. Every event has a single, known root cause, and every root
cause can be determined from the files in this folder alone.

There is no list of possible causes. You diagnose each event from first
principles and state its root cause in your own words, as you would in a
real investigation.

## What's in the folder

| File | Content |
|---|---|
| `PROCESS_DESCRIPTION.md` | The unit, its physics, control strategy, instruments and tag definitions. Read this first: the relationships in it are what the diagnosis rests on. |
| `historian.parquet` | The primary record. Long format (`tag`, `timestamp`, `value`, `quality`), compressed per tag, irregular timestamps. |
| `historian_60s.parquet` | The same data on a regular 1-minute grid, for convenience. |
| `tags.csv` | Tag dictionary. |
| `tk-101-hold-tank.svg` / `.png` | Process flow diagram of the unit and its instruments. |
| `submission_template.csv` | The format your answer must be in. |
| `CHALLENGE.md` | This file. |

## The task

Find every **fault event** in the record. For each one, report:

1. **Start time**: when the cause began acting on the unit (UTC).
2. **End time**: when the cause was removed (UTC).
3. **Root cause**: one root cause, stated in your own words (see
   *What counts as a correct root cause*).
4. **Evidence**: the tags that support your diagnosis. Not scored, but
   required (see *Rules*).

### What counts as an event

A **fault event** is a period during which a fault (a failure or abnormal
condition in the equipment, instruments, control system, utilities or
upstream supply) is acting on TK-101.

These are **not** fault events, and reporting them as faults costs points:

- Deliberate operator actions to which the unit responds normally.
- Instrument and historian artifacts: stuck readings, single-sample spikes,
  DCS communication outages. Section 8 of the process description
  describes these.
- Weather and seasonal effects, slow transmitter calibration drift, and
  anything on the neighbouring units (`CW-*`, `N2-*`, `SD-*`, `AIR-*`).

You may list operator actions and artifacts in your submission if that
helps you keep track of them. They are not scored either way.

### What you are told, and what you are not

- The first 14 days of the record are normal operation and can be used as a
  reference.
- Each fault event has exactly one root cause, and fault events do not
  overlap.
- Nothing is stated about how many events there are or which kinds of
  cause occur.
- Some faults develop gradually, so a fault can be acting for a while before
  its effects are visible. Your start time is when you judge the cause
  began. Scoring does not depend on precise timing (see below).

### What counts as a correct root cause

A root cause is judged correct when it names the same physical cause as the
answer key. It must say:

- **what** failed or changed: the specific equipment item, instrument,
  valve, controller, utility or upstream system, by tag where it has one;
  and
- **how**: the failure mode or mechanism.

Where more than one item could be responsible (for example two transmitters
measuring the same variable, or two control valves), name the one. A
description of the symptom is not a root cause: "temperature control is
poor" or "the steam valve is opening" describes what you saw, not why.

| Root cause as written | Judged |
|---|---|
| "Steam problem" | Too vague: neither the item nor the mechanism. |
| "Condensate temperature high" | A symptom, not a cause. |
| "Steam trap ST-101 failed open, passing live steam to condensate return" | Specific: item and mechanism. |

Each fault row must give **one** root cause. A row that offers alternatives
("A or B") is judged incorrect. Put your reasoning, and any alternatives you
ruled out, in `notes`.

Root causes are judged by the organiser against the answer key, and the
organiser's judgment is final.

## Submission

One CSV file in the format of `submission_template.csv`, one row per event:

| Column | Format | Example |
|---|---|---|
| `start_utc` | ISO 8601, UTC | `2024-03-10T14:00:00Z` |
| `end_utc` | ISO 8601, UTC | `2024-03-12T08:30:00Z` |
| `kind` | `fault`, `operator_action` or `artifact` | `fault` |
| `root_cause` | Free text, one root cause. Required for `fault` rows. | `Steam trap ST-101 failed open` |
| `evidence_tags` | Tags separated by semicolons | `TI-201;TI-202;FIC-201.PV` |
| `notes` | Free text, optional | |

Rows are discarded, not scored, if they have a missing or malformed time,
`end_utc` before `start_utc`, a `kind` other than the three above, no
`evidence_tags`, or (for `fault` rows) an empty `root_cause`.

## Scoring

Only rows with `kind` = `fault` are scored. These are **fault claims**.

### Matching claims to events

A claim **overlaps** a true fault event if its interval
[`start_utc`, `end_utc`] overlaps the event's true interval, extended by
6 hours after its true end: [true start, true end + 6 h].

Claims are matched to events one-to-one:

1. For every claim–event pair that overlaps, compute the overlap duration.
2. Repeatedly take the pair with the longest overlap among the claims and
   events not yet matched, and match them.
3. A claim left unmatched is a **false positive**. An event left unmatched
   is **missed**.

So a single claim spanning several events matches only one of them, and two
claims on the same event count as one match and one false positive.

### Points

| Outcome | Points |
|---|---|
| Fault event detected (matched by a claim) | +1 |
| Matched claim has the correct root cause | +2 more |
| False positive | −1 |
| Missed event | 0 |

There is no partial credit for a nearly-right cause: the root cause is
either judged correct or it isn't.

### Ranking

Submissions are ranked by total points. Ties are broken by, in order:

1. fewer false positives;
2. smaller median absolute error in `start_utc` over correctly diagnosed
   events.

## Rules

- Use only the files in this folder. The answer cannot be looked up, and
  attempting to obtain the answer key or the generating configuration is
  disqualifying.
- Any method is allowed: manual analysis, statistics, machine learning,
  LLM agents, or any mix of these. Say which you used in a short write-up.
- Every claim must list its evidence tags. A claim without evidence tags is
  discarded.
- One scored submission per person or team.

## How to submit

There is no deadline. Send your CSV, with a short write-up of your method,
to Scott Duncan on LinkedIn (https://www.linkedin.com/in/scottduncanai/).
Your score comes back to you privately. Please don't post submissions or
answers publicly: the dataset only works as a test while the answers stay
unknown.

## Answer key

The answer key is withheld so the dataset stays usable as a blind test. For
every event, it gives the true times and cause, and the tags that confirm
the cause and rule out the alternatives. You can request it, for example for
teaching or after you have submitted, from the same LinkedIn contact.
