# Fireworks-alert agent — scope notes

This file supplements the scheduled task's stored prompt. It records a scope
change requested by Saagar on 2026-09-13 that should be folded into the
scheduled task's prompt text itself (see note at the bottom — this repo file
alone won't change what the automated run does, since the run is driven by
the scheduler's stored prompt, not by files in this repo).

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

The same 7-day alert-timing rule applies to these new categories — track
immediately on discovery, email only once (event_date - today) <= 7 days.

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
  `GOOGLECALENDAR_CREATE_EVENT` on the `primary` calendar, timezone
  `America/New_York`.
- Event summary: `"{type emoji/label} {sponsor_or_name}"`. Description:
  production/worth-watching score + justification, viewability note, and
  source. Start/end: use the show's `time` field to build a start_datetime
  and a reasonable duration (default 2 hours if the time is a single instant
  like a meteor shower peak or moonrise; use the given window when the
  show's `time` field already specifies one).
- To avoid duplicate events on repeat runs, each tracked-shows.json entry now
  also carries `"calendar_event_id"` (the Google Calendar event id, or null
  until created) and `"calendar_event_created"` (bool). Only call
  `GOOGLECALENDAR_CREATE_EVENT` for an entry when `calendar_event_created` is
  false; set it true and store the id immediately after a successful create.
- If a previously-tracked entry's date/time/details change on a later run
  (re-discovered with new info), use `GOOGLECALENDAR_PATCH_EVENT` with the
  stored `calendar_event_id` to update it rather than creating a duplicate.

## Action needed to make this durable

This session does not have a tool to edit the actual scheduled task prompt
stored by the scheduler (only session-local CronCreate/CronList, which is
not what runs this daily task). To make both scope changes above stick for
future automated runs, Saagar should fold them into the scheduled task's
prompt text via the scheduled-task editor on claude.ai:
1. Change the ALERT TIMING RULE's "7 days" to "2 days" (and the email
   subject template's day count accordingly).
2. Add a new "CALENDAR EVENT CREATION" step alongside the existing tracking
   step, per the "Google Calendar event creation" section above.

Until that's done, both changes only reflect the one-off pass done in this
live session on 2026-09-19 (the 2-day rule applied to this run's alerting
decisions, and calendar events created for the shows tracked as of this
run).
