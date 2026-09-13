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

## Action needed to make this durable

This session does not have a tool to edit the actual scheduled task prompt
stored by the scheduler (only session-local CronCreate/CronList, which is
not what runs this daily task). To make this scope change stick for future
automated runs, Saagar should add the "Expanded discovery scope" section
above into the scheduled task's prompt text via the scheduled-task editor on
claude.ai. Until that's done, this expansion only reflects the one-off pass
done in this live session on 2026-09-13.
