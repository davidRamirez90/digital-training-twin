# FIT Export Format (Imported Workouts)

## Goals
- Preserve the **imported workout name** in the exported FIT so downstream tools display the planned session label.
- Ensure the exported activity is recognized as an **indoor bike training** session.

## Required fields

### Workout name
- Populate the **`workout` message** with `workout_name` set to the imported planned workout name.
- Mirror the same value into `session.name` when available so platforms that do not read the workout message still surface the label.
- If the imported workout does not have a name, fall back to `"Imported workout"`.

### Indoor bike classification
To ensure the activity is recognized as an indoor bike session, set the sport metadata on both session-level and activity-level records:
- `sport = cycling`
- `sub_sport = indoor_cycling`

## Export structure (minimum set)
Include the following FIT messages in the export, in order, with the required fields populated:

1. `file_id`
   - `type = activity`
   - `time_created`
2. `workout`
   - `workout_name` (imported workout name)
   - `sport = cycling`
   - `sub_sport = indoor_cycling`
3. `session`
   - `sport = cycling`
   - `sub_sport = indoor_cycling`
   - `name` (imported workout name)
   - `start_time`
   - `total_elapsed_time`
   - `total_timer_time`
4. `activity`
   - `sport = cycling`
   - `sub_sport = indoor_cycling`
   - `timestamp`
5. `record` (repeated)
   - `timestamp`
   - power/cadence/heart_rate/speed fields as available

## Validation checklist
- Exported FIT displays the **workout name** in downstream apps (Garmin Connect / TrainingPeaks / Intervals.icu).
- Exported FIT is categorized as **Indoor Cycling** (not Outdoor Cycling).
- When workout name is missing, the fallback label is applied consistently.
