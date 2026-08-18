# Product Specification

## Status

Initial product specification for the personal Garmin health + endurance training PWA.

The specification is deliberately focused on the useful first product. AI coaching is a future layer and should not block V1.

---

## 1. Primary user

One athlete / owner of the application.

No public registration, teams, billing or social features.

Primary usage contexts:

- quick check on iPhone after waking;
- quick check before training;
- activity review on iPhone;
- detailed analysis on desktop;
- planning the training week on desktop/mobile;
- later: conversation with an AI coach.

---

## 2. Navigation

Mobile-first bottom navigation target:

1. **Today**
2. **Health**
3. **Training**
4. **Calendar**
5. **Coach**

A compact quick-add action can later capture subjective data such as RPE, soreness or notes.

Desktop may use a sidebar/top navigation but should preserve the same conceptual sections.

---

## 3. Today

### Purpose

Answer in under 10 seconds:

- How recovered am I?
- What changed versus my baseline?
- What training is planned today?
- Is there anything unusual I should notice?

### Top section

Display a visually prominent Recovery card with:

- Recovery score (app-defined, later);
- state label such as Low / Moderate / Good / Excellent;
- HRV delta vs baseline;
- resting HR delta vs baseline;
- sleep score/duration.

Until a custom recovery score is validated, the UI may show Garmin Training Readiness and/or Body Battery more prominently, but the data model should support an independent score.

### Health cards

At minimum:

- HRV;
- resting HR;
- sleep;
- Body Battery;
- stress;
- Garmin Training Readiness.

Each card should show:

- latest value;
- comparison to personal baseline or recent average;
- simple qualitative state;
- tap/click to detailed history.

### Training status section

Show readable labels first:

- Fitness;
- Fatigue;
- Form;
- 7-day load;
- 42-day/chronic load.

Technical names (CTL/ATL/TSB/ACWR) may appear in detailed views/tooltips rather than dominating the main mobile page.

### Today's workout

If planned:

- sport;
- workout type;
- duration/distance;
- main targets;
- interval summary;
- status: planned/completed/missed.

If completed, show a brief comparison versus plan.

### Future AI summary

Reserve space for an AI-generated coaching summary, but do not require it in V1.

---

## 4. Health

### Purpose

Understand long-term recovery and wellness patterns without overwhelming the mobile experience.

### Metrics

Initial priority:

- HRV;
- resting HR;
- sleep duration;
- sleep score;
- deep/REM sleep where available;
- stress;
- Body Battery;
- Training Readiness;
- respiration / SpO2 where available and useful.

### Time ranges

Support:

- 7 days;
- 30 days;
- 3 months;
- 6 months;
- 1 year.

### Detail view

A metric detail should include:

- current value;
- personal baseline/recent mean;
- delta;
- chart;
- useful rolling average;
- optional annotations for significant training events.

Later: correlations between health and training load.

---

## 5. Training

### Purpose

Provide the analytical depth of an endurance platform while maintaining a modern UI.

### Overview

Display:

- total volume by sport;
- training load;
- fitness/fatigue/form;
- intensity distribution;
- recent trend;
- sport-specific summaries.

### Running

Possible metrics:

- distance;
- duration;
- pace;
- HR;
- elevation;
- training load;
- zones;
- threshold/critical pace estimates later;
- pace curve / best efforts later.

### Cycling

Possible metrics:

- duration;
- distance;
- elevation;
- average power;
- normalized power;
- IF;
- TSS or equivalent;
- cadence;
- HR;
- power zones;
- power curve / best efforts;
- FTP history.

### Swimming

Possible metrics:

- distance;
- duration;
- pace;
- intervals/laps;
- HR where available;
- load estimate.

### Load model

The exact load model is not frozen yet.

Initial candidates:

- cycling: power-based TSS-like load when power is available;
- running: HR/TRIMP and/or pace/power-based load;
- swimming: duration/intensity-based load;
- normalized cross-sport daily load for fitness/fatigue/form calculations.

Implement calculations as replaceable domain services.

---

## 6. Activity detail

### Summary

Show:

- activity type;
- date/time;
- duration;
- distance;
- elevation;
- HR;
- sport-specific metrics;
- training load;
- Garmin source metadata.

### Mobile information architecture

Use tabs or stacked disclosure sections such as:

- Overview;
- Performance;
- Heart Rate;
- Power / Pace;
- Laps;
- Zones.

Do not render every metric at once.

### Laps/intervals

Preserve enough detail to compare repeated intervals and future planned-vs-actual workouts.

Example future use:

Planned: 6 x 1 km @ 3:55/km

Actual:

- 3:54
- 3:53
- 3:55
- 3:58
- 4:01
- 4:04

The application and future AI should be able to detect the fade without parsing raw FIT data every time.

---

## 7. Calendar

### Purpose

Become the central training plan/history view.

### Required concepts

A calendar item can be:

- planned workout;
- completed activity;
- linked planned + completed pair;
- missed workout;
- rest day;
- event/race.

### Planned workout schema should support

- sport;
- workout type;
- target duration/distance;
- warm-up;
- interval blocks;
- repetitions;
- recoveries;
- cool-down;
- target pace/HR/power;
- rationale/notes;
- origin: manual / AI / imported.

Later:

- push structured workout to Garmin;
- reschedule workouts;
- AI modifications with explicit confirmation.

---

## 8. Subjective feedback

Garmin cannot capture all relevant coaching context.

The product should eventually support quick entries such as:

- session RPE;
- general fatigue;
- leg freshness;
- soreness;
- pain flag/location;
- perceived sleep quality;
- work/life stress;
- notes;
- available training time.

Mobile entry must be extremely fast.

These signals will later be important AI inputs.

---

## 9. Athlete profile and goals

Store manually editable values such as:

- max HR;
- resting HR;
- FTP;
- threshold HR;
- pace/threshold values;
- power/HR zones;
- body mass if desired;
- current sport priorities.

Goals/events should support:

- name;
- date;
- sport/event type;
- target performance;
- priority;
- notes.

---

## 10. Recovery score

Do not hard-code a final formula prematurely.

The custom recovery model will likely use personal baselines and may include:

- HRV vs baseline;
- resting HR vs baseline;
- sleep duration/quality;
- sleep debt;
- stress;
- recent training load;
- subjective feedback.

Garmin Training Readiness and Body Battery should be retained as comparison signals rather than blindly treated as the application's own recovery score.

All underlying components must remain available for auditability.

---

## 11. Future AI Coach

### Long-term interaction examples

- "What should I train today?"
- "Build the next 4 weeks toward my 10 km race."
- "I cannot train Thursday anymore; adapt the week."
- "Why is my recovery low today?"
- "Compare yesterday's threshold session with the previous three."
- "Am I accumulating too much fatigue?"

### AI tool layer

The AI should request data through focused tools, for example:

- `getAthleteState(date)`
- `getHealthTrend(metric, days)`
- `getRecentActivities(sport, days)`
- `getActivityDetails(id)`
- `getTrainingLoad(days)`
- `getGoals()`
- `getPlannedWorkouts(range)`
- `getSubjectiveFeedback(days)`

### Principle

The LLM interprets and plans.

Deterministic application code calculates:

- rolling averages;
- z-scores;
- TSS/TRIMP-like load;
- fitness/fatigue/form;
- zone time;
- best efforts;
- planned-vs-actual interval comparisons.

---

## 12. UX quality bar

The app must not look like a dense admin dashboard.

Use:

- strong typography;
- generous spacing;
- large primary values;
- readable cards;
- restrained color with semantic meaning;
- polished dark mode;
- clear charts;
- touch-friendly controls;
- progressive disclosure;
- excellent iPhone safe-area behavior.

The desired product feeling is closer to a premium consumer health app than to a spreadsheet/dashboard tool.

---

## 13. V1 acceptance criteria

V1 is successful when the owner can:

1. open the PWA from an iPhone Home Screen;
2. see the latest Garmin health/recovery data;
3. see recent activities;
4. open an activity detail page;
5. see basic trends for HRV, resting HR and sleep;
6. use the same app on desktop;
7. keep imported data persistently;
8. preserve original FIT files;
9. do all of this without an AI dependency.
