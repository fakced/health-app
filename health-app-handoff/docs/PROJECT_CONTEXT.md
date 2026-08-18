# Project Context

## 1. Vision

Build a personal health and endurance-training application that acts as a long-term athlete data platform and, later, an AI-assisted coach.

The application should feel much better than traditional endurance dashboards on mobile while retaining advanced training analysis.

The desired synthesis is:

- **Project Helix / Whoop** for daily-health UX and readability;
- **Intervals.icu** for analytical depth, training calendar and endurance metrics;
- **Garmin** as the primary health/activity source;
- **AI** later for interpreting history, recovery, goals and training constraints.

This is a personal project only. It will not be commercialized and does not need App Store publication.

## 2. Current hardware / data source

Primary wearable: Garmin watch, worn all day and overnight.

Garmin is expected to provide, depending on what is available through Garmin Connect and the device:

- heart rate;
- resting heart rate;
- HRV;
- sleep and sleep stages;
- stress;
- Body Battery;
- Training Readiness;
- Training Status;
- SpO2 / respiration where available;
- daily activity / steps / calories;
- recorded workouts;
- detailed FIT activity files.

The desired behavior is quasi-live rather than truly sensor-live: Garmin watch -> Garmin Connect sync -> ingestion service -> application.

A delay of several minutes is acceptable for health data.

## 3. Why not a native iPhone app

A native iOS application is not required.

The preferred experience is the same model as Intervals.icu:

- responsive website;
- installable as a PWA from Safari;
- icon on iPhone Home Screen;
- full-screen/app-like usage;
- same application and deployment for desktop and mobile.

This avoids:

- Xcode as a product dependency;
- development provisioning renewal;
- duplicated web/mobile code;
- App Store publication.

## 4. Product philosophy

### Desktop

Desktop can be information-rich and analytical:

- training load;
- power/pace curves;
- zone analysis;
- trends;
- activity comparison;
- season/calendar planning;
- longer-term health correlations.

### iPhone

The iPhone must still expose substantial data, but with excellent information hierarchy.

The goal is not to hide advanced metrics. The goal is to progressively reveal them.

Example:

- card: `HRV 62 ms, +8% vs baseline`;
- tap -> 7d/30d/90d charts, baseline, daily values, interpretation.

This is intentionally different from Intervals.icu's dense visual presentation.

## 5. Main product areas

### Today

The primary daily screen.

Expected content:

- Recovery score;
- HRV relative to personal baseline;
- resting HR relative to baseline;
- sleep duration/score;
- stress;
- Body Battery;
- Garmin Training Readiness;
- current training load/form summary;
- today's planned workout;
- later: AI recommendation/summary.

### Health

Longitudinal health trends:

- HRV;
- resting HR;
- sleep;
- stress;
- Body Battery;
- recovery;
- respiration / SpO2 when useful;
- correlations over 7d / 30d / 3m / 6m / 1y.

### Training

Endurance analytics across running, cycling and swimming:

- training load;
- fitness/fatigue/form;
- volume;
- intensity distribution;
- zones;
- power / pace analysis;
- activity detail;
- laps and intervals;
- progression;
- personal bests / curves.

### Calendar

A central planned-vs-completed training calendar:

- planned workouts;
- completed workouts;
- missed/modified sessions;
- drag/reschedule later if useful;
- future AI modifications;
- eventual Garmin workout push.

### Coach

Initially a placeholder / non-AI section.

Long-term goal: conversational coaching based on structured athlete context.

The AI should eventually be able to:

- understand recent health and load;
- understand training history;
- understand race goals;
- understand availability and subjective feedback;
- propose a day/week/plan;
- adapt a plan when constraints or recovery change;
- compare planned vs completed work;
- eventually create structured Garmin workouts.

## 6. Open-source projects worth auditing

These are references, not fixed dependencies.

### FitnessJournal

Interesting because it reportedly combines:

- Garmin health sync;
- training/recovery context;
- AI coaching;
- workout planning;
- Garmin workout/calendar integration.

Audit for architecture and reusable Garmin/AI patterns.

### Garmin Health Dashboard

Interesting for:

- Next.js / TypeScript dashboard patterns;
- Garmin-oriented health presentation;
- PWA/mobile experience.

Audit especially for data mapping and UI inspiration.

### Coach Watts

Interesting for:

- endurance training metrics;
- fitness/fatigue/form;
- adaptive training ideas;
- TypeScript/PostgreSQL-style architecture.

Audit for training-domain logic.

### garmin-ai-coach / Lumen / coachd / similar personal projects

Useful as references for:

- AI context construction;
- athlete-state summaries;
- planning loops;
- Garmin -> analysis -> recommendation workflows.

Do not assume one of these should be forked until code quality, maintenance status and licensing have been inspected.

## 7. Cost philosophy

Target recurring cost for the personal version: as close to zero as practical.

Acceptable approaches include:

- free-tier PostgreSQL;
- SQLite for local/prototype stages;
- object storage / cheap blob storage for FIT files;
- serverless jobs for scheduled Garmin sync;
- Cloudflare/GitHub/free-tier hosting where appropriate.

The exact provider is intentionally not locked yet.

Choose based on:

1. free-tier sustainability;
2. operational simplicity;
3. ability to run scheduled Python ingestion;
4. reliable database backups;
5. easy PWA deployment;
6. no unnecessary platform lock-in.

## 8. Garmin integration constraints

Garmin Connect consumer APIs are not treated as a stable public API for this project.

For this personal project, using community libraries is acceptable.

Primary candidate:

- `python-garminconnect`.

Important implications:

- keep Garmin calls behind an adapter;
- keep authentication isolated;
- tolerate endpoint changes;
- avoid aggressive polling;
- retain raw source data when useful;
- make the rest of the application independent of Garmin payload structure.

## 9. Data ownership philosophy

The application should become the durable athlete history.

For every imported workout:

- keep Garmin activity ID/source metadata;
- preserve the original FIT file when available;
- normalize activity summary metrics;
- keep laps/intervals;
- compute derived training metrics separately.

This means future algorithms can be recomputed without re-downloading the entire Garmin history.

## 10. Future AI context

The AI should not consume giant dumps of raw activity points.

Instead, the system should expose structured context such as:

- daily athlete state;
- recent training summaries;
- health trends;
- training-load trends;
- upcoming events/goals;
- planned workouts;
- subjective feedback;
- detailed activity data only on demand.

Numeric analysis belongs primarily in deterministic application code. The LLM should reason over curated metrics and context.
