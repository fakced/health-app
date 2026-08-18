# Architecture

## 1. Architectural goals

The architecture should be:

- inexpensive / ideally free for personal use;
- simple to operate;
- compatible with scheduled Garmin synchronization;
- PWA-first;
- able to preserve raw Garmin activity files;
- able to support rich endurance analytics;
- ready for a future AI tool layer;
- not over-engineered for multiple users.

The infrastructure provider is deliberately not fixed yet.

---

## 2. Proposed logical architecture

```text
Garmin Watch
    |
    v
Garmin Connect Mobile / Garmin Cloud
    |
    v
Garmin Ingestion Service (Python)
    |\
    | \----> Raw storage (FIT + selected raw JSON)
    |
    v
Normalized relational database
    |
    +----> Domain calculations (health / recovery / load)
    |
    v
Application API / server layer
    |
    v
Next.js responsive PWA
    |
    +----> iPhone Home Screen
    +----> Desktop browser

Future:
Application services -> AI tool layer -> LLM Coach
```

---

## 3. Repository shape

Recommended starting structure:

```text
health-app/
├── AGENTS.md
├── docs/
│   ├── PROJECT_CONTEXT.md
│   ├── PRODUCT_SPEC.md
│   └── ARCHITECTURE.md
├── apps/
│   └── web/
├── services/
│   └── garmin/
├── packages/
│   ├── domain/
│   ├── db/
│   ├── schemas/
│   └── ui/
└── storage/              # optional local dev raw data, gitignored
```

Do not add a separate mobile application initially.

---

## 4. Web application

### Preferred

- Next.js
- React
- TypeScript
- PWA manifest/service worker as needed
- responsive/mobile-first UI

The exact component library is not fixed.

Selection criteria:

- polished mobile controls;
- accessible charts;
- dark mode;
- minimal visual overhead;
- easy customization.

Avoid choosing a component library solely because it produces enterprise/admin-dashboard aesthetics.

### PWA requirements

At minimum:

- installable manifest;
- application icons;
- standalone display mode;
- theme/background colors;
- iPhone safe-area support;
- sensible caching strategy;
- no assumption of continuous connectivity for purely local navigation.

---

## 5. Garmin ingestion service

### Language

Python is preferred for the Garmin adapter because the community Garmin tooling is strongest there.

### Candidate dependencies

- `python-garminconnect`
- Garmin FIT SDK and/or a robust FIT parser

Verify current library APIs before implementation.

### Responsibilities

The Garmin service should:

1. authenticate to Garmin Connect;
2. restore/persist session tokens;
3. retrieve daily health metrics;
4. retrieve latest activities;
5. detect newly imported activities;
6. download original FIT files;
7. normalize Garmin payloads into internal DTOs;
8. write normalized records to the database;
9. store raw files through a storage abstraction;
10. handle rate limiting/failures conservatively.

### Non-responsibilities

It should not:

- render UI;
- calculate all application training metrics;
- contain LLM logic;
- expose raw Garmin payload shapes directly to the frontend.

---

## 6. Synchronization strategy

Exact cadence should be configurable.

Suggested initial model:

### Frequent lightweight sync

Every ~10-15 minutes while using a hosted scheduler, or a less frequent cadence if required by free-tier constraints.

Fetch only data that benefits from intraday updates:

- recent daily metrics;
- stress/Body Battery if available;
- latest activities;
- Training Readiness where useful.

### Daily/deep reconciliation

Once or a few times daily:

- sleep;
- HRV;
- complete daily summaries;
- previous-day corrections;
- Training Status;
- other slower-changing metrics.

### Backfill

Provide a manual command/job to backfill a historical date range.

Backfill should be resumable/idempotent where practical.

---

## 7. Storage strategy

### Relational database

Store normalized, queryable entities.

Recommended database families:

- PostgreSQL for hosted/persistent version;
- SQLite is acceptable for an ultra-fast local proof of concept, but do not let SQLite-specific assumptions leak into the domain model if a hosted PWA is expected soon.

### Raw file storage

Use a storage abstraction for:

- original FIT files;
- selected raw Garmin JSON snapshots;
- optional exports/backups.

Provider may be:

- local filesystem during development;
- S3-compatible object storage for hosted use;
- Cloudflare R2 or equivalent free/cheap object storage.

Keep storage paths stable and source-aware.

Suggested pattern:

```text
athlete/
  garmin/
    activities/YYYY/MM/<activity-id>.fit
    raw/health/YYYY-MM-DD.json
```

---

## 8. Proposed data model

This is a starting point, not a final Prisma schema.

### `athlete_profile`

Fields may include:

- id
- timezone
- max_hr
- resting_hr_manual
- ftp
- threshold_hr
- threshold_pace
- body_mass
- zone configuration
- created_at
- updated_at

### `data_connection`

- id
- provider (`GARMIN` initially)
- provider_user_id if available
- encrypted/session metadata or token reference
- last_sync_at
- last_success_at
- status

### `daily_health`

- date
- resting_hr
- hrv_value
- hrv_baseline_low/high or rolling values
- sleep_duration
- sleep_score
- deep_sleep_duration
- rem_sleep_duration
- stress_avg
- body_battery_min/max/current
- training_readiness
- respiration
- spo2
- steps
- raw_source_version/reference

### `health_sample` (optional)

Only when intraday charts require it.

- timestamp
- metric
- value
- source

Do not store every possible sample unless the UI actually needs it.

### `activity`

- id
- provider
- provider_activity_id
- sport
- start_time
- duration
- distance
- elevation_gain
- avg_hr
- max_hr
- avg_power
- normalized_power
- cadence
- calories
- training_effect fields if useful
- raw_fit_path
- source metadata

### `activity_lap`

- activity_id
- lap_index
- start_time
- duration
- distance
- avg_hr
- max_hr
- avg_power
- normalized_power
- avg_pace/speed
- cadence
- elevation

### `activity_series` (optional/downsampled)

Only for chart rendering if reading FIT on demand is too slow.

- activity_id
- timestamp/offset
- hr
- power
- cadence
- speed/pace
- altitude
- lat/lng when needed

Downsample by default.

### `planned_workout`

- id
- date
- sport
- type
- duration_target
- distance_target
- structured_steps JSON/normalized structure
- notes
- origin (`MANUAL`, later `AI`)
- status
- linked_activity_id

### `goal`

- id
- name
- event_date
- sport/event_type
- target
- priority
- notes

### `subjective_feedback`

- date/timestamp
- session/activity reference optional
- rpe
- fatigue
- soreness
- pain_flag/location
- perceived_sleep
- life_stress
- available_training_minutes
- notes

### `athlete_daily_state`

A derived/snapshot table intended to simplify UI and future AI access.

Potential fields:

- date
- recovery_score
- hrv_zscore
- resting_hr_zscore
- sleep_debt
- acute_load
- chronic_load
- fitness
- fatigue
- form
- run_load_7d
- bike_load_7d
- swim_load_7d
- days_since_hard_run
- days_since_hard_bike
- days_since_long_run
- days_since_long_bike
- derived_version

This table should be reproducible from source data where possible.

---

## 9. Domain calculations

Keep calculations in a framework-independent package such as `packages/domain`.

Candidate modules:

```text
health/
  baselines.ts
  recovery.ts
training/
  load.ts
  fitnessFatigue.ts
  zones.ts
  bestEfforts.ts
  intervalAnalysis.ts
```

### Important

Do not commit too early to one proprietary-style recovery or strain formula.

Store the raw components and version derived metrics.

Example:

```text
recovery_score = 81
recovery_model_version = "v0.1"
```

This allows future recalculation.

---

## 10. API/service layer

The frontend should consume internal application services, not Garmin directly.

Candidate API style:

- Next.js server actions/API routes;
- tRPC if it materially improves typed data access;
- another small typed API layer if simpler.

Do not choose an API framework just for symmetry with another project.

The future AI layer should call the same domain/application services where possible.

---

## 11. Future AI architecture

Do not implement the AI coach until the underlying data is reliable.

When implemented:

```text
User question
    |
    v
AI orchestrator
    |
    +--> getAthleteState()
    +--> getRecentActivities()
    +--> getTrainingLoad()
    +--> getGoals()
    +--> getPlannedWorkouts()
    +--> getSubjectiveFeedback()
    |
    v
LLM response / proposed plan
```

The LLM should not query the database with arbitrary unrestricted SQL by default.

Expose narrow, typed tools/services.

Potential write tools later:

- `createPlannedWorkout()`
- `rescheduleWorkout()`
- `replaceWorkout()`
- `createTrainingWeek()`
- `pushWorkoutToGarmin()`

Writes should be explicit and reviewable.

---

## 12. Hosting options

No provider is mandated yet.

### Preferred decision process

For each candidate setup, compare:

- monthly cost at personal usage;
- scheduler support;
- Python job support;
- PostgreSQL availability;
- object storage;
- deployment friction;
- backups;
- free-tier sleep/limits.

A plausible setup is:

```text
PWA hosting: Cloudflare / Vercel / Netlify free tier
Database: Supabase / Neon free tier
Raw files: Cloudflare R2 or equivalent
Garmin scheduled worker: Cloud Run job / GitHub Actions / another serverless scheduler
```

But the first proof of concept may run fully locally.

### Important

Do not create cloud resources until the local Garmin ingestion proof of concept works.

---

## 13. Security

Even for a personal project:

- do not commit Garmin credentials;
- keep Garmin auth/session tokens out of Git;
- encrypt secrets at rest where hosted;
- use environment variables/secret storage;
- keep object storage private;
- avoid exposing raw FIT files via public URLs;
- maintain simple backups of database + raw FIT data.

No enterprise security architecture is required.

---

## 14. Initial milestones

### Milestone 0 — audit

Before coding heavily:

- inspect candidate open-source projects;
- confirm licenses;
- confirm Garmin integration approach;
- decide whether to reuse code or only ideas.

### Milestone 1 — Garmin ingestion POC

Success criteria:

- authenticate to Garmin;
- fetch today's health metrics;
- list recent activities;
- download one original FIT;
- inspect payloads from the actual Garmin account/device.

No polished UI required yet.

### Milestone 2 — normalized persistence

- create DB schema;
- persist daily health;
- persist activity summaries/laps;
- preserve FIT files;
- make sync idempotent.

### Milestone 3 — PWA Today screen

- polished mobile-first design;
- latest HRV, sleep, resting HR, Body Battery, Training Readiness;
- recent training summary;
- installable PWA.

### Milestone 4 — Health + activity analysis

- trend charts;
- activity detail;
- zones/laps;
- basic training load.

### Milestone 5 — Training + Calendar

- fitness/fatigue/form;
- planned workouts;
- planned-vs-completed links.

### Milestone 6 — subjective data

- quick mobile input;
- integrate with athlete state.

### Milestone 7 — AI Coach design

Only after the above data is trustworthy.
