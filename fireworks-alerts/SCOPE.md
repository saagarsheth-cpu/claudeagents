# Fireworks-alert agent — scope notes

This file supplements the scheduled task's stored prompt. It records scope
changes requested by Saagar, applied to the scheduled task's actual prompt
text (via the claude.ai scheduled-task editor) as of 2026-09-19 — Saagar
pasted a full rewritten prompt on that date that already folds in the
2026-09-13 expanded-discovery-scope change below plus the two 2026-09-19
changes (2-day alert window, "Balcony Shows" calendar). This file is kept as
a historical/reference record of what's in that prompt; it is not itself
read by the automated run.

## Expanded discovery scope (as of 2026-09-13)

In addition to fireworks displays, each run should also discover and track:

1. **Drone light shows with high production value** — judged the same way as
   fireworks (vendor/sponsor prestige, scale, media coverage), using the
   Hoboken Italian Festival's 100th-anniversary show (Pixis Drones, sponsored
   by Mutti USA, Sept 12 2026, explicitly billed "look east to the Hudson
   River") as the bar for what counts as good production value.
2. **Special astronomical events**: eclipses (lunar/solar), notable
   supermoons, meteor shower peaks, planet conjunctions/viewings, and similar
   sky events — anything with a specific date/time worth looking outside for.
3. **Aerial and water events/shows** — naval reviews, tall ship parades,
   military flyovers (e.g. Blue Angels/Thunderbirds), air shows, and similar
   large-scale spectacles on or over the Hudson/harbor, using Sail4th 250
   (America's 250th-anniversary NYC Harbor bash: International Parade of
   Sail, tall ships, naval vessels, an aerial review, July 4 2026) as the bar
   for what counts as this category, even though that specific event has
   already passed.

For astronomical events, the `production_score` field is repurposed as a
rough "how worth watching" score, and `score_justification` should explain
that framing (rarity, brightness, whether conditions are favorable, e.g. a
new moon coinciding with a meteor shower peak).

Astronomical viewability notes should account for NYC-area light pollution:
a 32nd-floor Jersey City balcony facing the lit Manhattan skyline is bad for
faint events (most meteor showers) but genuinely good for anything that uses
the east-facing skyline as a backdrop (e.g. a full moon rising over
Manhattan).

The same alert-timing rule applies to these new categories — track
immediately on discovery, email only once (event_date - today) <= the
threshold in effect (2 days as of 2026-09-19; see below).

## Alert-timing rule change (as of 2026-09-19)

Saagar asked to move the email trigger from 7 days out to **2 days out**.
The rule is now: track every discovered show/event immediately regardless of
date, but only send an email once `(show_date - today) <= 2` days AND the
show has not already been emailed (`notified` flag). Replace every "7 days"
/ "7-day window" reference in the scheduled task's stored prompt with
"2 days" / "2-day window" (this appears in the ALERT TIMING RULE section and
in the email subject line template's "(in {N} days)" framing, which should
now only ever show values of 0-2).

## Google Calendar event creation (as of 2026-09-19)

Saagar also asked for a Google Calendar event to be created for each tracked
show/event, not just an email alert. This should happen on **every run**, at
the same time a show is upserted into tracked-shows.json — independent of
the 2-day email-alert threshold (i.e. create the calendar event immediately
on discovery, don't wait for the show to be within 2 days).

- Use the Composio Google Calendar connector (account alias
  `saagarsheth-triage-routine`, connected 2026-09-19) via
  `GOOGLECALENDAR_CREATE_EVENT`. **Do not use the `primary` calendar** --
  Saagar explicitly asked for a separate calendar, not his personal/work one.
  Use the dedicated **"Balcony Shows"** calendar (id
  `3420da5c58857df69f454507278684d5178f0ca539e3dbb121e798f1fecdd951@group.calendar.google.com`,
  created 2026-09-19, timezone `America/New_York`) as `calendar_id` on every
  create/patch call.
- Event summary: `"{type emoji/label} {sponsor_or_name}"`. Description:
  production/worth-watching score + justification, viewability note, and
  source. Start/end: use the show's `time` field to build a start_datetime
  and a reasonable duration (default 2 hours if the time is a single instant
  like a meteor shower peak or moonrise; use the given window when the
  show's `time` field already specifies one).
- To avoid duplicate events on repeat runs, each tracked-shows.json entry now
  also carries `"calendar_event_id"` (the Google Calendar event id, or null
  until created), `"calendar_event_created"` (bool), and `"calendar_id"` (the
  calendar it was created on, or null). Only call `GOOGLECALENDAR_CREATE_EVENT`
  for an entry when `calendar_event_created` is false; set it true and store
  the id + calendar_id immediately after a successful create.
- If a previously-tracked entry's date/time/details change on a later run
  (re-discovered with new info), use `GOOGLECALENDAR_PATCH_EVENT` with the
  stored `calendar_event_id` to update it rather than creating a duplicate.

## Durability status: RESOLVED (2026-09-19)

This session did not have a tool to edit the actual scheduled task prompt
stored by the scheduler (only session-local CronCreate/CronList, which is
not what runs this daily task), so the fix had to go through Saagar
directly. He confirmed on 2026-09-19 that he replaced the scheduled task's
entire stored prompt (via the claude.ai scheduled-task editor) with a
rewritten version that includes:
1. The 2-day alert-timing threshold (was 7 days).
2. A "CALENDAR EVENT CREATION" step targeting the dedicated "Balcony Shows"
   calendar (never `primary`).
3. The 2026-09-13 expanded discovery scope (drone shows, astronomical
   events, aerial/water spectacles) — already folded into that same rewrite.

Future automated runs should reflect all of this without further action.
If a future run is observed still using the old 7-day threshold or writing
to the primary calendar, the prompt swap didn't take — ask Saagar to check
the scheduled task's saved text against the current version of this file.
