# Deck Manager — Sanction Parser Prompt

Use this prompt (plus the attached template) when asking an AI to parse a
swim meet sanction document into a Deck Manager `.json` meet file.

---

## Prompt

You are parsing a USA Swimming swim meet sanction document into a JSON file
for the **Deck Manager** application. The JSON schema is described below.
A fully-annotated example is provided in `deck_manager_meet_template.json`.

### Output requirements

- Output **only** valid JSON — no markdown fences, no commentary outside the JSON.
- Every `id` field must be a unique UUID4 string (use `uuid.uuid4()` or equivalent).
- **`deck_manager_meet_template.json` contains `_comment`-prefixed keys
  (`_fixed_fields_comment`, `_sessions_comment`, `_events_comment`, etc.)
  purely to explain the template to you while you read it. These are
  template scaffolding, not data — remove every one of them from your
  output. Your output should contain no key starting with `_` unless you
  are deliberately adding your own new note (optional, and unrelated to
  the template's own comment keys above).
- Match field order and field presence to `deck_manager_meet_template.json`
  exactly (see "Field order" below) — this mirrors what the app itself
  produces when it saves a file.

---

### Top-level meet fields

| Field | Type | Source in sanction |
|---|---|---|
| `name` | string | Full meet name, exactly as titled in the sanction — do not append the host city/state or venue even if you know them; that belongs in `location` instead |
| `host_club` | string | Hosting team name |
| `location` | string | Facility name + full address |
| `meet_referee` | string | Meet Referee name |
| `admin_referee` | string | Administrative Referee name (or "TBD") |
| `sanction_number` | string | Sanction number |
| `dates` | string | Human-readable meet-wide date range, e.g. "July 16 - 19, 2026" (same-month) or "February 26 - March 1, 2026" (spanning months) — always a spaced hyphen |

**Field order** at the top level: `name`, `host_club`, `location`,
`meet_referee`, `admin_referee`, `sanction_number`, `dates`,
`logo_path`, `sessions`, then the fixed fields (see below) in the order shown
in the template.

---

### Determining the course (SCY / SCM / LCM)

Do **not** default to SCY. Read the meet title/classification and the
qualifying-time-standards table to determine the conforming course:

- "Long Course" in the meet name, or a qualifying-times table showing
  distances/times in meters (LCM), or explicit language like "The conforming
  time for this meet is LCM" → the meet is run in **LCM**. All events use
  `course: "LCM"` and event descriptions use meters (e.g. `"200M Freestyle"`),
  even though the qualifying-standards table may also list SCY equivalent
  times for reference.
- "Short Course Yards"/SCY meet name or yard distances/times → `course: "SCY"`.
- A meet with SCM qualifying standards → `course: "SCM"`.

The "LCM qualifying times still swim SCY" pattern only applies to the
opposite case: a **short course** meet that lists LCM times as an alternate
qualifying standard. Don't apply that exception to a meet whose conforming
course is itself LCM.

---

### Sessions

Create one session object per row in the meet schedule table.

**`name`** — build from the day and what the session actually is, e.g.
`"Thursday Timed Finals"`, `"Friday Preliminaries"`, `"Friday Finals"`. Keep
it brief — a plain `"Friday Preliminaries"` is exactly as clear as `"Friday
Morning Preliminaries"` when Friday only has one Preliminaries session, so
prefer the shorter form. Only add a time-of-day qualifier
(`"Morning"`/`"Afternoon"`/`"Evening"`) when a single day genuinely has
**more than one session of the same `session_type`** and the plain name
would otherwise collide — that's the only case where the qualifier adds
information.

**`session_type`** — must be exactly one of:
- `"Prelims"` — morning preliminary heats (Prelim/Finals format)
- `"Finals"` — evening finals (follows a Prelims session same day)
- `"Timed Finals"` — standalone timed final session (10&Under, Thursday specials, etc.)
- `"Time Trials"` — post-meet or standalone time trials (no CJ assignments; omit from sanction-generated sessions unless explicitly listed in the sanction)

**`day`** — full day name: `"Thursday"`, `"Friday"`, `"Saturday"`, `"Sunday"`

**`date`** — **ISO format `"YYYY-MM-DD"`**, e.g. `"2026-07-16"`. (This is what
the app itself writes to the file — do not use a human-readable date like
"July 16, 2026" for this field. The meet-wide `dates` field at the top level
is the only place that uses a human-readable range.)

**`start_time` / `warmup_time`** — 12-hour format: `"9:00 AM"`, `"4:15 PM"`

**`notes`** — any important notes from the sanction (positive check-in
deadlines, scratch rules, deck-seeding procedures, break locations, etc.)

**`locked`** — always `false` for a new meet file

**Field order** on a session object: `id`, `name`, `session_type`, `date`,
`day`, `start_time`, `warmup_time`, `pool_sessions`, `notes`, `locked`.
(`notes` and `locked` come **after** `pool_sessions`.)

---

### Pool sessions

Each `session` contains a `pool_sessions` list — one entry per simultaneous
**physical** pool running during that session.

**Default assumption: ONE pool per session.** Most facilities — especially
for Long Course (LCM) meets — have a single competition pool. Very few
venues in the world have two long-course pools running simultaneously. Only
create multiple `pool_sessions` for a single session if the sanction's
facility description explicitly says the venue can be split into multiple
concurrent competition courses (e.g. "a 50-meter pool which can be broken
into two short course venues") **and** the schedule/format actually calls
for using that split for the sessions in question. If in doubt, or if the
facility section only describes one pool with one lane count, use a single
`pool_session` for every session in the meet, including Timed Finals
sessions.

**The tell-tale sign of a genuine multi-pool session is two or more
schedule blocks sharing the exact same day, start time, AND warm-up
time.** When the facility supports simultaneous multi-pool competition
and you see this pattern, those blocks are **one `session` with multiple
`pool_sessions`** — never two (or more) separate `session` objects that
merely happen to start at the same time. This applies **regardless of
what the blocks are split by** — the split is most often by age group
(e.g. "11-12 Timed Finals" and "10 & Under Timed Finals" both at 1:00 PM),
but can also be by gender, or another category entirely. Do not assume
"multi-pool" only ever means a gender split; the deciding factor is
always simultaneous timing at a multi-pool facility, not the specific
category the sanction uses to label each block. A common mistake is
treating each same-time block as its own session (e.g. producing
"Friday 11-12 Timed Finals" and "Friday 10 & Under Timed Finals" as two
separate sessions) — if they share identical day/start_time/warmup_time
at a confirmed multi-pool facility, they belong in one session's
`pool_sessions` list instead, each with its own `age_groups` (or
`notes`) describing what actually swims there.

When a genuine multi-pool split is confirmed and it happens to be split
by gender specifically (this is common for SCY dual-pool facilities
running age-group Timed Finals sessions split by gender): alternate
which gender is assigned Pool 1 across successive Timed Finals sessions
(e.g. Girls=Pool1 on session A, Boys=Pool1 on session B, Girls=Pool1 on
session C, ...). When the split is by age group instead, pool numbering
doesn't need to alternate — just assign pool_number consistently and
describe each pool's age group(s) in `age_groups`/`notes`.

**Pool session fields:**

| Field | Value |
|---|---|
| `pool_number` | 1-based integer, unique within the session |
| `name` | e.g. `"Pool 1"` or `"Pool 1 — Girls"` (only add a gender suffix if genuinely split) |
| `age_groups` | list of strings: `["11-12", "13-14"]`, `["10 & Under"]` |
| `num_lanes` | integer, from facility description (typically 8 or 10) |
| `course` | `"SCY"`, `"SCM"`, or `"LCM"` |
| `notes` | `"Girls"` or `"Boys"` for genuinely split-gender pools, `"11-12"` /
  `"10 & Under"` etc. for genuinely split-age-group pools, else `""` |
| `start_area_lane_n` | boolean, always `false` (a sanction never says where the deck officials' start area is; the Chief Judge sets it later in Deck Manager) |
| `events` | list of event dicts, see below |

**Field order** on a pool_session object: `id`, `pool_number`, `name`,
`age_groups`, `events`, `num_lanes`, `course`, `notes`, `start_area_lane_n`.
(`events` comes **before** `num_lanes`/`course`/`notes`.)

---

### Events

Each event dict inside `pool_session.events` must have these exact keys, in
this order: `number`, `description`, `gender`, `age_group`, `course`,
`is_relay`, `relay_type` (only if `is_relay` is `true`), `deck_seeded`,
`fastest_heat_finals`.

| Key | Type | Notes |
|---|---|---|
| `number` | integer | Event number from Order of Events |
| `description` | string | `"{age_group} {Girls\|Boys\|Mixed} {distance}{course} {stroke}"` e.g. `"13-14 Girls 200M Backstroke"`. For a Mixed-gender exhibition event, omit the Girls/Boys word: `"13-14 Para Mixed 100M Event"`. |
| `gender` | string | `"F"` for Girls/Women, `"M"` for Boys/Men, `"Mixed"` for combined-gender exhibition events (see below) |
| `age_group` | string | `"10 & Under"`, `"11-12"`, `"13-14"`, `"11-14"`, `"Open"` etc. |
| `course` | string | `"SCY"`, `"SCM"`, or `"LCM"` — match the meet's conforming course (see above) |
| `is_relay` | boolean | `true` for relay events |
| `relay_type` | string | `"Freestyle"` or `"Medley"` — **only when `is_relay` is `true`**, omit otherwise |
| `deck_seeded` | boolean | `true` **only** for events explicitly listed in the sanction's positive check-in table. Don't infer this from event distance alone — check the actual check-in table. |
| `fastest_heat_finals` | boolean | `true` if only the fastest prelim/checked-in heat(s) advance to a Finals session while the remaining heats swim elsewhere (prelims, afternoon session end, etc.) — see "Events split across heats" below |

**Numbering convention:** Girls events typically have odd numbers, Boys have
even numbers (standard USA Swimming convention). Combined events (e.g.
11-14 1650 Free) may share a number range.

**Finals sessions:** Repeat the same event numbers and descriptions as the
corresponding Prelims session. Only include events that actually have Finals
(not timed finals events that only swim in Prelims).

**Timed Finals split-pool (only when a genuine multi-pool split applies,
see above):** Each pool_session only contains its own gender's events.
Girls go in the Girls pool, Boys go in the Boys pool.

---

### Events numbered in one session block but actually swum in another

Sanctions frequently assign event numbers to an age group's events within
one time block's Order of Events table (often for numbering-scheme
convenience), while the narrative text states the event is **actually
contested during a different session** — commonly, an age group's 200
back/fly/breast event is numbered within an afternoon Timed Finals block but
is actually swum interspersed with that evening's Finals session ("Events
17/18 swim in finals").

**Place the event only in the pool_session where it is actually swum** —
do not duplicate it into the block where it merely appears in the numbering
table. Read the narrative sections of the sanction (not just the Order of
Events tables) to confirm where each event is physically contested.

---

### Events split across heats (same event number, multiple entries)

Some events are seeded/run in separated heat groups that appear more than
once in the Order of Events, sharing the same event number — e.g. an event
that runs "Heats 1-3" early in a session and "Heats 4+" later in the same
session, or an event whose Order of Events entries are literally labeled
`17/1`, `17/2`, `17/3`, `17/4+` interspersed among other events.

**Represent each occurrence as its own event dict, in the exact position it
appears in the Order of Events, all sharing the same `number`.** Distinguish
them by appending the heat label to `description` in parentheses:

- `"11-12 Girls 400M IM (Heats 1-3)"` and `"11-12 Girls 400M IM (Heats 4+)"`
- `"11-12 Girls 200M Backstroke (Heat 1)"`, `"... (Heat 2)"`, `"... (Heat 3)"`, `"... (Heat 4+)"`

Set `deck_seeded` and `fastest_heat_finals` the same on every occurrence of
that event number (they describe the event as a whole, not the specific
heat group).

---

### Breaks

If the Order of Events lists a scheduled break (e.g. "10-minute break")
between events, insert an explicit break entry into the `events` list at
that exact position, using this exact shape (this is what the app itself
writes — do not use a `_comment`-only placeholder, it will not display
correctly):

```json
{
  "number": 0,
  "description": "10-minute break",
  "gender": "",
  "is_relay": false,
  "relay_type": null,
  "is_break": true,
  "break_minutes": 10
}
```

Adjust `break_minutes` and `description` to match the break's stated
duration. Include a break entry for every break shown in the Order of
Events tables, in every session where one appears — don't assume breaks are
symmetric across similar sessions (e.g. Sunday Finals may have no break even
if Friday/Saturday Finals do).

---

### Exhibition / Para / Mixed-gender events

Some sanctions include non-scored, no-fee exhibition events for para
athletes swum in a stroke of the athlete's choosing (e.g. "13-14 Para Mixed
100M Event"), interspersed at a specific point in the Order of Events.
**Include these as real event entries** at their correct position — do not
omit them just because they don't fit the standard Girls/Boys pattern:

- `gender`: `"Mixed"`
- `description`: `"{age_group} Para Mixed {distance}{course} Event"` (or
  whatever label the sanction uses)
- `is_relay`: `false`, `deck_seeded`: `false`, `fastest_heat_finals`: `false`
  unless the sanction states otherwise

---

### Fixed fields

Always include these fields verbatim at the end of the JSON (they represent
an empty meet with no officials or assignments yet), in this order:

```json
"officials": [],
"signups": [],
"deck_assignments": [],
"cj_task_defs": [],
"cj_task_assignments": [],
"cj_scope_overrides": {},
"tlcj_rotation_eligible": {},
"deck_task_defs": [],
"deck_task_assignments": [],
"deck_scope_overrides": {},
"event_library": [],
"shirts_enabled": false,
"shirt_min_sessions": 4,
"shirt_bonus_session_ids": [],
"shirt_count_time_trials": false,
"radios_enabled": false,
"num_radios": 0,
"radio_labels": [],
"radio_pool_channels": {},
"radio_assignments": [],
"own_radio_equipment": [],
"radio_roles": ["MR", "AR", "AO", "TLCJ", "CJ", "DR"],
"radio_unavailable": [],
"radio_spares": [],
"auto_assign_targets": {},
"auto_assign_targets_by_type": {}
```

---

### Common patterns to watch for

- **Positive check-in events** — mark `deck_seeded: true` **only** for
  events explicitly listed in the sanction's check-in table. Note the
  check-in deadline in the session's `notes` field.
- **Fastest heat to Finals** — mark `fastest_heat_finals: true` on both the
  Prelims/afternoon occurrence and the Finals occurrence (relays), or on
  every occurrence of an event that's split across heats where only the
  fastest checked-in heat(s) land in a Finals session (see "Events split
  across heats" above).
- **Combined age group events** — use the combined age group string in both
  `description` and `age_group` (e.g. `"11-14"` for 11-14 1000/1650 Free).
- **A short course meet with LCM qualifying times** — events still swim SCY;
  keep `course: "SCY"` on the event. (Do not confuse this with a Long
  Course meet, which swims LCM — see "Determining the course" above.)
- **10&Under / age-group Timed Finals sessions** — only split into two pools
  if the facility genuinely supports two simultaneous competition pools
  (common for SCY meets carved from one facility); otherwise single pool.
  If it does, and the schedule lists (for example) "11-12 Timed Finals" and
  "10 & Under Timed Finals" at the identical day/start_time/warmup_time,
  that is one session with two pool_sessions (one per age group) — not two
  separate sessions that coincidentally share a start time. See "Pool
  sessions" above.
- **Thursday/mixed-block Timed Finals sessions** — may have a mix of age
  groups (e.g. 13-14 relays, 11-12 individual). Still single pool unless a
  genuine multi-pool split is confirmed.
- **Event numbers in Finals** — the Order of Events table usually just lists
  the event number again without repeating the full description. Use the
  same number and rebuild the description from the Prelims entry.
- **Events numbered in one block, swum in another** — see dedicated section
  above; place the event only where it's actually contested.
- **Qualifying-time substitution notes are not event information — ignore
  them for extraction purposes.** Sanctions often let a swimmer enter one
  event using a qualifying time achieved in a *different* event (e.g. "a
  1650Y Freestyle qualifier may enter the 1000Y Freestyle using their 1650Y
  time," or "a 200Y IM qualifying time also qualifies for the 100Y IM").
  This is purely about which time a swimmer may use to become *eligible*
  to enter an event — it says nothing about which events are actually
  contested, and it never means the substitute event (the 1650Y, the 200Y
  IM) is itself swum at this meet, or that the real event (the 1000Y, the
  100Y IM) should be renamed, merged, or skipped. The Order of Events
  table alone determines which events exist and what they're called;
  never let an eligibility/substitution note override or explain away
  what that table literally shows.
- **Don't infer a break, or any other event, exists in a session just
  because a similar session elsewhere in the meet has one.** Each
  session's Order of Events must be read on its own — a Saturday
  Preliminaries session having a "5-minute break" does not mean Friday
  Preliminaries has one too, even if the two sessions otherwise look
  similar. Only include a break where that specific session's table
  actually shows one, with the exact duration given there.
- **Breaks and exhibition events** — see dedicated sections above; both must
  appear as real entries in `events`, positioned exactly where they occur
  in the Order of Events.

---

### Example

See `deck_manager_meet_template.json` for a fully worked example including:
a single-pool Timed Finals session with a heat-split event (heats 1-3 /
heats 4+), a Prelims session with a scheduled break, a Finals session with
a heat-split event interspersed among other events, a break, and a
Mixed-gender exhibition event, and a genuine **multi-pool** Timed Finals
session (Saturday) split by age group across two simultaneous pools —
study that last one closely, since a same-time multi-pool split being
mistaken for two separate sessions is a common extraction error.

---

### Validation checklist before returning

- [ ] All `id` fields are unique UUID4 strings
- [ ] All `session_type` values are exactly `"Prelims"`, `"Finals"`, `"Timed Finals"`, or `"Time Trials"`
- [ ] Session `date` fields use ISO `"YYYY-MM-DD"` format
- [ ] No session has zero `pool_sessions`
- [ ] No pool_session has zero `events`
- [ ] Multi-pool splits are used **only** where the facility genuinely supports simultaneous multi-pool competition — default is one pool per session
- [ ] Checked for two or more schedule blocks sharing the identical day/start_time/warmup_time at a multi-pool facility (whatever they're split by — age group, gender, etc.) — if found, merged into **one** session with multiple `pool_sessions`, not left as separate sessions
- [ ] `course` reflects the meet's actual conforming course (check the meet title/classification, don't default to SCY)
- [ ] All non-break events have `number`, `description`, `gender`, `age_group`, `course`,
      `is_relay`, `deck_seeded`, `fastest_heat_finals`
- [ ] `relay_type` is present on all relay events and absent on non-relay events
- [ ] `deck_seeded` matches the sanction's positive check-in table exactly — not inferred from distance
- [ ] Events split across heats appear as repeated entries (same number) with heat labels in `description`, in their literal Order-of-Events position
- [ ] Events numbered in one block but swum in another appear only in the session where they're actually contested
- [ ] No event was skipped, renamed, or merged because of a qualifying-time substitution/eligibility note (e.g. "X qualifiers may enter Y") — those describe entry eligibility only, never which events are contested
- [ ] No break (or other event) was added to a session just because a similar session elsewhere in the meet has one — each session's own Order of Events table was checked individually
- [ ] For each session, counted the individual event entries actually listed in the sanction's Order of Events (including breaks and exhibition events, matching however the source counts them) and confirmed your extracted `events` list has the same count — a mismatched count is the most common way an event gets silently dropped, especially when two same-stroke events of different distances appear close together (e.g. a 100 IM and a 400 IM in the same block)
- [ ] Scheduled breaks appear as `is_break: true` entries with the exact shape shown above, at their correct position
- [ ] Mixed-gender/Para exhibition events are included at their correct position
- [ ] Finals sessions contain the same event numbers as their Prelims counterpart
- [ ] Field order on session / pool_session / event objects matches the template
- [ ] Fixed fields are present and empty, in the order shown above — use `""` for empty strings, `[]` for empty lists, `{}` for empty objects; never omit a fixed field or set it to `null`
- [ ] Output is valid JSON with no trailing commas
- [ ] Every `_comment`-prefixed key from the template has been removed from
      the output — they exist only to guide extraction, not to appear in
      the final JSON
