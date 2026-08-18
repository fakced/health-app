# AGENTS.md

## Project purpose

This repository is a personal, single-user health and endurance-training application.

The product should combine:

- the polished, mobile-friendly daily-health experience of Project Helix / Whoop;
- the depth of endurance analytics found in Intervals.icu;
- Garmin as the primary health and activity data source;
- a future AI coach capable of understanding long-term training history, recovery, goals and constraints, then proposing and adapting training plans.

This project is personal only. It is not intended to become a commercial SaaS and will not be published publicly on the App Store.

## Mandatory reading before architectural work

Before changing architecture, data models, health calculations, Garmin ingestion, or navigation, read:

1. `docs/PROJECT_CONTEXT.md`
2. `docs/PRODUCT_SPEC.md`
3. `docs/ARCHITECTURE.md`

Treat those documents as the current product source of truth.

If code and documentation disagree, flag the discrepancy before making a large architectural change.

## Core product principles

1. **Mobile-first, but not mobile-only.**
   - The application is a responsive web app / PWA.
   - It must feel excellent when launched from the iPhone Home Screen.
   - Desktop should expose deeper analytics without making mobile unusable.

2. **Simple to read, deep when explored.**
   - Do not imitate the visual density of Intervals.icu.
   - Prefer progressive disclosure: summary first, detailed metrics on tap/click.
   - Advanced data must remain available.

3. **Garmin is the primary source of truth for wearable data.**
   - Use Garmin Connect data through a dedicated ingestion layer.
   - Preserve original FIT files whenever possible.
   - Do not make the rest of the application depend directly on Garmin-specific payload shapes.

4. **Personal project: optimize for simplicity and near-zero recurring cost.**
   - Avoid enterprise infrastructure unless clearly necessary.
   - Prefer free-tier/serverless/self-hostable services.
   - Avoid Kubernetes, always-on VMs, or unnecessary queues for the initial version.

5. **Design for future AI coaching from day one, but do not implement the AI coach in V1.**
   - Store structured training, recovery, goals, subjective feedback and planned workouts.
   - Create clean service boundaries so AI tools can later query structured athlete state.
   - Do not send raw second-by-second data to an LLM by default.

6. **Single user, but clean domain model.**
   - Multi-tenancy, billing, teams and SaaS administration are out of scope.
   - It is acceptable to keep a `user_id` / athlete entity where useful for clean modeling, but do not add SaaS complexity.

## UX direction

Visual references:

- Project Helix / Whoop: daily health, recovery, sleep, polished cards, strong hierarchy.
- Intervals.icu: analytical depth, calendar, training history and endurance metrics.
- Garmin Health Dashboard / FitnessJournal / Coach Watts: candidate open-source projects to audit for ideas and reusable patterns.

Do not copy proprietary visual designs pixel-for-pixel. Use them as product inspiration.

On iPhone, prioritize:

- Today
- Health
- Training
- Calendar
- Coach (placeholder until AI is implemented)

The iPhone experience must still expose meaningful detail. Do not oversimplify it into only three scores.

## Garmin ingestion rules

- Keep Garmin integration in its own adapter/service.
- Candidate Python library: `python-garminconnect`.
- Authentication/session handling must be isolated from domain logic.
- Poll conservatively; Garmin Connect integration is unofficial for this personal project.
- Save normalized data in the main database.
- Preserve original activity FIT files outside the relational database.
- Preserve raw Garmin JSON selectively when it helps future migrations/debugging.
- Never make UI components depend directly on raw Garmin responses.

## Data rules

Relational database should contain normalized and queryable data such as:

- daily health summary;
- HRV, resting HR, sleep, stress, Body Battery, Training Readiness;
- activity summaries;
- laps / intervals;
- derived training load;
- goals;
- planned workouts;
- subjective feedback;
- daily athlete state.

Raw high-frequency activity data should generally remain in FIT files rather than creating millions of relational rows.

Downsampled time-series data may be stored when needed for charts.

## AI-readiness rules

Future AI tools should be able to request focused structured data such as:

- `getAthleteState(date)`
- `getRecentActivities(sport, days)`
- `getTrainingLoad(days)`
- `getHealthTrend(metric, days)`
- `getGoals()`
- `getPlannedWorkouts(range)`
- `getSubjectiveFeedback(days)`
- `getActivityDetails(activityId)`

Prefer deterministic calculations in application code/database over asking the LLM to calculate basic metrics from raw data.

## Coding approach

- Prefer TypeScript for the product/application layer.
- A small Python service is acceptable and expected for Garmin ingestion if it materially simplifies Garmin Connect access and FIT parsing.
- Keep domain calculations testable and framework-independent.
- Use strict typing and schemas at API boundaries.
- Favor incremental implementation over broad scaffolding.
- Avoid speculative abstractions that are not needed by the next 1-2 milestones.

## Initial development sequence

Unless the user asks otherwise, implement in this order:

1. Repository foundation and local development.
2. Garmin authentication proof of concept.
3. Import one day of health data.
4. Import recent activities and preserve original FIT files.
5. Normalize health/activity data.
6. Build a polished mobile-first `Today` screen.
7. Add Health trends.
8. Add Activity detail and Training analytics.
9. Add Calendar and planned workouts.
10. Add subjective daily feedback.
11. Only then design the AI Coach tool layer and conversation experience.

## Do not do yet

Do not add unless explicitly requested:

- App Store publishing;
- React Native or Swift app;
- billing;
- multi-user signup flows;
- enterprise auth;
- public SaaS onboarding;
- social features;
- AI-generated training plans before the underlying data model is stable;
- complex vector databases for numeric training metrics.

## Definition of a good first milestone

A first useful version should allow the owner to:

1. connect/import Garmin data;
2. see today's recovery/health metrics in a polished iPhone-friendly PWA;
3. see recent training activities;
4. retain historical data locally/in the chosen backend;
5. open the same application on desktop for deeper analysis.
