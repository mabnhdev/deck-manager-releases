# Deck Manager — v1.1.0

A Python/Tkinter desktop application for swimming Chief Judges to manage
officials, build session rosters, and generate deck assignment sheets for swim meets.

---

## Installation (Windows)

Download and run `DeckManagerSetup-x.x.x.exe` from the
[Releases](https://github.com/mabnhdev/deck-manager/releases) page.

- No Python installation required — the app is fully self-contained.
- Installs to `Program Files\Deck Manager` with a Start Menu shortcut.
- User data (saved meets, preferences, exports) is stored in `%APPDATA%\DeckManager\`.

## Installation (macOS)

Download and open `DeckManager-x.x.x.dmg` from the
[Releases](https://github.com/mabnhdev/deck-manager/releases) page.

- Drag **Deck Manager.app** to your Applications folder.
- On first launch, Sample-Data is copied to `~/Documents/DeckManager/Sample-Data/`
  so you can open it from the Import dialogs.
- User data is stored in `~/Library/Application Support/DeckManager/`.

> **Note:** macOS Gatekeeper may show an "unidentified developer" warning.
> Right-click the app → **Open** → **Open** to proceed.
- An uninstaller is included via Add/Remove Programs.

> **Note:** Windows SmartScreen may show an "Unknown publisher" warning on first
> run because the executable is not yet code-signed. Click **More info → Run anyway**
> to proceed.

---

## Running from Source

```bash
# 1. Clone the repository
git clone https://github.com/mabnhdev/deck-manager.git
cd deck-manager

# 2. Install dependencies
pip install reportlab pdfplumber pyyaml

# 3. Run
python app.py
```

**Requirements:** Python 3.10 or higher.

---

## Features

| Feature | Details |
|---|---|
| **Meet Setup** | Meet name, host club, dates, sanction number; define sessions and pools per session |
| **Multi-Pool Support** | Each session can contain multiple simultaneous pools with independent lane counts and course types (SCY / SCM / LCM) |
| **Official Roster** | Add, edit, remove officials; track role, email, phone, club, LSC, USAS ID, good standing, and position preferences |
| **Sign-Ups** | Per-session sign-up management with five views: **By Official**, **By Session**, **Grid**, **By Preference** *(shown when any signup has role preferences)*, and **By Cert** *(shown when any signup has certifications)*. By Session shows a role-count summary for all assigned positions (MR · AR · TLCJ · CJ · DR · HR · SR · HS · CO · TO · EV · AO · ST). Grid view color-codes national positions (HR, HS, CO, TO, EV) distinctly from standard roles. By Preference and By Cert show a "Sessions signed up" status line for the selected official and let you assign a role to all of their unlocked sessions in one click |
| **Sign-Ups → Auto-Assign** | Bulk-fills AO, CO, DR, SR, and CJ across the whole meet from a single set of ideal per-pool targets (blank = skip a position; saved and undoable like any other meet setting). Candidates are ranked by role preference, then certification level (N3 > N2 > LSC > App), then most sessions signed up, then earliest signup; CO has no certification field so it is preference-only. For multi-pool sessions, a stated pool preference is honored first, and any remaining shortage is split across pools as evenly as possible rather than emptying out later pools. **Clear Auto-Assign** reverts only the officials Auto-Assign placed, leaving manual assignments untouched; a live "Currently Auto-Assigned" table always reflects actual state, including after a restart |
| **Sign-In** | Day-of arrival tracking with walk-in support, pool preference, and assignment preference capture; the **Assignment Preference** dropdown offers paired positive and negative options (e.g. *Turn End* / *Not Turn End*, *Relief* / *Not Relief*) — positive preferences bias the Deck Builder toward that position; negative ("Not") preferences strongly discourage it (safety valve only); preferences set at sign-in and protected by session lock |
| **Session Roster** | Session roster with inline role and pool assignment editing; ★ prefix on Assigned Role flags officials whose role is still the default (ST) and needs review |
| **CJ Team** | TLCJ checklist across Officials Meeting / Pre-Session / Session / Post-Session phases; CJ corner auto-assignment; **Add Custom Task** dialog includes All Sessions checkbox; required CJ count reflects actual enabled position slots (including CJ Relief when enabled); **Auto-Assign This Session** assigns corners first then distributes tasks using least-loaded-first, respecting four built-in affinity pairs — Assignment + Assignment Sheets Distribution, Radio Distribution/Check + Collect Radios, Prepare + Collect Timer Clipboards (per pool), and Distribute + Collect Bell(s) and Lap Counters (per pool) — all always going to the same CJ; a manually set task anchors its affinity pair and is never overwritten by auto-assign; **Auto-Assign All Sessions** runs the same logic across every unlocked session in one click; task scope (session vs per-pool) can be toggled per session; disabling or reassigning a position clears the displaced CJ from all task assignments; **Unpositioned → ST** button (active only when unpositioned eligible CJs exist); **Rotation eligible: TLCJ** checkbox (unchecked by default) opts the Team Lead CJ into auto-assign like any other CJ; Distribute/Collect Bell(s) and Lap Counters only apply to a pool with a freestyle distance event (≥500y SCY / ≥400m LCM-SCM, current USA Swimming rule) and Create and Distribute RTO Slips only applies to a pool with a relay event — both always shown, defaulting to unchecked rather than hidden when a pool doesn't need them |
| **Deck Team** | DR/Starter pairing panel (optional teams); flat DR and SR lists for unpaired officials; **Unassigned → ST** button reassigns unteamed DR/SR officials to ST (active only when unassigned officials exist); pre-defined and custom tasks (Timer Briefing — per-session, SR only, 1 slot default; Invigilate — per-pool, DR only, 1 slot default) with per-role eligibility (DR-only, SR-only, or Either), per-session or per-pool scope, multi-slot support, and round-robin auto-assign; **Rotation eligible: HR / HS** checkboxes (both unchecked by default) opt Head Deck Referees / Head Starters into filling DR/SR rotation slots; **Auto-Assign This Session** builds rotations and tasks for the selected session — any DR/SR shortage relative to the other role is split evenly across pools rather than front-loading one pool, and each run is a full rebuild so a stale assignment from an earlier run can't linger; **Auto-Assign All Sessions** runs the same logic across every unlocked session in one click; **Add Custom Task** dialog includes All Sessions checkbox to scope a task to one session or all; **Export Deck Team PDF** — landscape at-a-glance matrix + portrait per-session detail pages |
| **Time Trials** | Dedicated *Time Trials* session type: CJ Team tab is hidden (no CJ assignments), Sign-In banner notes the TT context, shirt eligibility and session ordering handle TT sessions correctly |
| **Deck Builder** | Configure special positions (stroke judges 0/1/2/4 per side with automatic sub-position labels — Lead/Lag for 2/side, Start End/Start End Mid/Turn End Mid/Turn End for 4/side; 15m judges, relief, reserve); **Start End ST/lane** — select **1 (standard)**, **2 Chairs**, or **3 Chairs**; **TE Lap Counters** checkbox on same row; **RTO lanes/pair** — Half the lanes, 2, or 1; **Freestyle corners** — 4 (T1, S8, T8, S1), 3 (T1, S8, T8, deck), 2 (S8, T1), 1 (deck, T8), or 0; per-pool deployment variations; **Relief interval** — three modes: *fixed N min*, *every N events*, *every N stroke events* (F+M pair); **Generate Deck Assignments** available for any selected session — warns before generating on an unlocked session and warns if manual Deck Set overrides exist; **👁 Preview Current Session** available for unlocked sessions — generates a what-if deck assuming all signed-up officials are present, without saving; 15m judges default on for SCY/SCM, off when stroke judges > 0; selecting stroke judges automatically unchecks 15m (and vice versa); stroke judge default is 2/side for LCM, 0 for SCY/SCM |
| **ST Assignment Fairness** | Burden-based round-robin rotation for all ST positions across sessions: SE/TE lane slots, Stroke, 15m, Relief, and Reserve rotate evenly so no official holds the same position every session; surplus CJ and DR/SR officials sent to ST are selected by least-prior-ST sessions first, with most-recently-ST officials filling rotation slots first so the queue advances in strict round-robin order; positive and negative ("Not") position preferences bias the selection without overriding the rotation |
| **Deck Set Review** | Human-readable two-column review (Position / Assigned To) organized by pool and section (Session Officials, Deck Teams, CJ Team, Stroke & Turn, Stroke/15m Judges, Relief Teams, Reserve, Freestyle/Medley RTO & Corners, Unassigned Officials); **pool selector** — for multi-pool sessions, radio buttons filter the view to show session officials, the selected pool, and unassigned officials (resets to All on session change); **drag-and-drop overrides** — drag an ST name to another ST row to swap officials, drag to Unassigned to clear a slot, drag an ST or any session official (TLCJ, CJ, reserve, relief, etc.) to an RTO/Corner slot to assign a covering position, drag an RTO/Corner slot to another to swap covering positions; session-official overrides show the official's name alongside their position; off-deck rotation overrides labelled "Off Deck Ref N" / "Off Starter N" without a name (rotation may not be off-deck at event time); duplicate RTO/Corner assignments highlighted red with a warning — detects same official twice, same off-deck label twice, or all deck-team rotations assigned as RTOs; overridden rows shown in blue; double-click an ST row to unassign, double-click an RTO/Corner row to choose Unassigned, Off-Deck Official, or Leave as Is; generating a new deck clears all overrides (with confirmation if overrides exist); RTO and corner slots show the covering position label (not a name) because the actual person changes with relief rotations; in national-meet mode all RTO/Corner slots start as Unassigned; displays preview deck while preview mode is active |
| **Import — EV3 / HYV** | Parse a event file to auto-populate meet info, sessions, pools, and events; Finals sessions automatically seeded from Prelims |
| **Import — Google Forms CSV** | Import official sign-ups from a Google Forms export; session columns auto-detected by day and session type |
| **Import — SignUpGenius CSV** | Export the signup list from SignUpGenius as CSV and import via **Import → Import SignUpGenius CSV…**; Deck Manager reads names, emails, and session sign-ups from the slot column headers; duplicate officials (matched by email) have their sign-ups updated rather than re-added |
| **Shirt Tracking** | Optional per-meet shirt distribution tracker; configurable minimum sessions and bonus session rules; size and cut per official; Given checkbox with color-coded treeview; session selector filters Worked counts to sessions up to and including the selection; **Export Shirts PDF** enabled only when selected session is locked; **Preview Shirts** enabled only when selected session is unlocked |
| **Radio Tracking** | Optional per-meet radio management; configurable radio count with custom labels and unavailable flags; pool-to-channel mapping; auto-assign uses graph coloring to give each official a sticky radio with no sharing unless radios are scarce; checkout grid (radio × session, Out/In cells); compact usage heat map; own-equipment registry for officials bringing their own gear; **Export Checkout PDF** (radio × session grid with handwriting space); **Export Radio Check PDF** (per-session list sorted by channel for pre-meet radio check) |
| **Export — Deck Set PDF** | Print-ready deck assignment sheet, one page per pool |
| **Export — Sign-In Sheet PDF** | Formatted sign-in sheet with walk-in rows |
| **Export — CJ Matrix PDF** | Landscape at-a-glance matrix across all sessions + portrait per-session detail pages (TLCJ, CJ corner positions, phase-grouped task assignments) |
| **Export — Event Order PDF** | Session event order for deck use |
| **Export — Shirts PDF** | Color-coded shirt distribution list with earning rules and summary counts |
| **Meet Report** | Post-meet staffing summary and officials activity report; covers locked sessions only; exported PDF matches the tab exactly |
| **Export — Meet Report PDF** | Two-page PDF: session staffing table (role counts, full-deck %) and per-official activity list; locked sessions only |
| **Persistence** | Auto-save on every change; save/load meets as JSON; full undo (Ctrl+Z) |

---

## Workflow

1. **File → New Meet** — enter meet name and info
2. **Import → Import Event File (.ev3 / .hyv)** — auto-populate sessions, pools, and events; or enter sessions and pools manually on the Meet Setup tab
3. **Complete Meet Setup dialog** — review and adjust warmup times, lane counts, and course type after import
4. **Officials tab** — add officials manually, use **Import → Google Forms CSV** to bulk-import from Google Forms, or **Import → Import SignUpGenius CSV…** to import directly from a SignUpGenius signup export
5. **Sign-Ups tab** — review and adjust per-session sign-ups; assign session roles manually (By Official / By Session / By Preference / By Cert), or set ideal per-pool targets on **Auto-Assign** and click **⚡ Auto-Assign Whole Meet** to bulk-fill AO/CO/DR/SR/CJ across every unlocked session at once
6. **Sign-In tab** — mark officials arrived as they check in; record pool and assignment preferences; **lock the session once it has begun** to protect sign-in data for USA Swimming reporting
7. **Roster tab** — final review of session roster; inline-edit roles and pool assignments
8. **CJ Team tab** — assign CJs to checklist tasks and corner positions; use Auto-Assign This Session (or Auto-Assign All Sessions for the whole meet) to populate from the roster
9. **Deck Team tab** — optionally pair DRs and Starters into named teams; use **Auto-Assign This Session** (or **Auto-Assign All Sessions**) to build rotations automatically; use **Unassigned → ST** to reassign unteamed DR/SR officials to ST; assign pre-defined and custom tasks (Timer Briefing, Invigilate) manually or via Auto-Assign; use **Export Deck Team PDF** for a combined meet-wide matrix and per-session detail pages
10. **Deck Builder tab** — select a session; click **Generate Deck Assignments** to generate the deck (a warning is shown if the session is unlocked); use **👁 Preview Current Session** for unlocked sessions to generate a what-if deck (all signed-up officials assumed present) without saving
11. **Deck Set tab** — review the generated assignments, including Deck Team task assignments
12. **Shirts tab** *(optional)* — enable shirt tracking in Meet Setup; configure earning rules; select a session to filter Worked counts and earning status to that cutoff; use **Preview Shirts** (unlocked session) to project earning if the session were locked now; use **Export Shirts PDF** (locked session) to export the final distribution list; mark shirts as given
13. **Radios tab** *(optional)* — enable radio tracking in Meet Setup; set number of radios; click **Edit Labels / Availability** to add custom labels and mark broken radios unavailable; configure pool channels; click **⚡ Auto-Assign** to assign radios using graph coloring (sticky per official, no idle radios); review the checkout grid and usage heat map; export **Checkout PDF** for the table and **Radio Check PDF** for pre-meet channel-sorted check
14. **Export → Export Deck Set PDFs** — generate print-ready assignment sheets, one page per pool
15. **Meet Report tab** — after locking all sessions, review staffing summary and officials activity; export the Meet Report PDF for USA Swimming reporting

---

## Event File Import

The event file parser (`event_file_parser.py`) supports both EV3 and HYV formats
exported from Meet Manager.

**EV3** (full session assignments): Sessions, pool assignments, start times, and
event order are read directly from the file. Missing Finals sessions are
automatically created and pre-populated with all events from their corresponding
Prelims session.

**HYV** (event catalog only): Events are loaded into the Event Library on the
Meet Setup tab for manual assignment to sessions.

After import a **Complete Meet Setup** dialog allows review and adjustment of
warmup times, lane counts, and course type for each session.

---

## Meet Info/Sanction Import

You can use the power of AI to parse a Meet Info/Sanction **PDF** file into a saved meet **JSON** file.

In the AI of your choice, attach the `Sample-Data/deck_manager_meet_template.json` and `Sample-Data/deck_manager_sanction_parser_prompt.md` files plus the sanction PDF and say: "Parse this sanction into a Deck Manager meet JSON following the prompt in the .md file and the schema in the template .json."

You can then replace the **Import** step in the **Workflow** above with **File → Open...** on the file generated by the AI.

---

## CSV Import

### Google Forms
The column mapper dialog auto-detects session columns by matching day and session
type keywords against the meet's defined sessions. Supported formats include:

| Column format | Matches |
|---|---|
| `Fri Prelims` / `Friday Preliminaries` | Friday Prelims session |
| `Fri Finals` / `Friday Finals` | Friday Finals session |
| `Fri (10U/11-12 Sessions)` | Friday Timed Finals session |
| `Thurs Finals` | Thursday Timed Finals session |
| `Sat Prelims`, `Sun Finals`, etc. | Saturday / Sunday sessions |

Unmatched columns can be manually assigned on the Session Columns tab of the
import dialog before importing.

### SignUpGenius
*(Planned — Issue #77.)* Full session sign-up parsing is not yet implemented.
Attempting to import a SignUpGenius CSV will display a "Not Yet Supported" dialog.

---

## Sample Data

The `Sample-Data/` directory contains real-format test files and is bundled with
the Windows installer. The Import dialogs default to this folder on a fresh install.

---

## Shirt Tracking

Shirt tracking is optional and enabled per-meet via a checkbox in the Meet Setup tab.

**Earning rules** — an official earns a shirt if they meet either condition:

- They worked at least **N sessions** (configurable; default 4), or
- They worked **all** of a designated set of bonus sessions

Only **locked sessions** count toward earning. A session is locked on the Sign-In tab once its sign-in data is final. Unlocked sessions are excluded so that partial or in-progress sign-in data does not prematurely grant shirts.

**Size and cut** are imported automatically from Google Forms CSV if the form
includes "Men's Cut" and "Women's Cut" columns, or can be set manually via
inline editing or the **Edit Size/Cut** bulk dialog.

**Treeview color coding:**

| Row color | Meaning |
|---|---|
| Blue | Shirt already marked as given |
| Green | Official has earned a shirt (not yet given) |
| White | Official does not yet qualify |

---

## Radio Tracking

Radio tracking is optional and enabled per-meet via a checkbox in the Meet Setup tab.

**Setup** — configure the number of radios, optional custom labels (e.g. "DR-1"), and pool-to-channel mapping (e.g. East = Ch 1). Mark any broken or unavailable radios via **Edit Labels / Availability**; unavailable radios are shown in pink and excluded from auto-assign. A **Session key** row in the setup section shows the `#1 = Friday Prelims · #2 = …` mapping used throughout the tab and in exports.

**Auto-Assign** — selects which roles require a radio (default: MR, AR, AO, TLCJ, CJ, DR). Assigns radios using graph coloring: officials who work the same session get different radios (sticky across the meet); officials whose sessions never overlap may share a radio only when there are more officials than radios. Idle radios are always filled before any sharing occurs.

**Sub-tabs:**
- **Checkout Grid** — rows = radios, columns = sessions (`#1`, `#2` …). Click the Radio/Official column to assign; click Out/In cells to record checkout. Cells stay blank until explicitly filled.
- **Own Equipment** — officials bringing their own radio, headset, or clip; excluded from auto-assign but included in Radio Check PDF.
- **Usage Heat Map** — full-size grid with initials in every assigned cell (FL format). Shared radios shown in distinct shades of green. Refreshes live while the sub-tab is active.

**Exports:**
- **Checkout PDF** — landscape grid with handwriting-friendly rows; sessions labelled #1, #2 … with a legend; splits across pages if needed
- **Radio Check PDF** — one page per session; columns: Radio | ✓ | Role | Official | Pool/Channel; rows sorted by pool/channel; own-equipment officials shown in green

---

## Building the Windows Installer

Requires [PyInstaller](https://pyinstaller.org) and
[Inno Setup 6](https://jrsoftware.org/isdl.php).

```bash
pip install pyinstaller
./build_windows.sh
```

Output: `installer/DeckManagerSetup-x.x.x.exe`

To release a new version:
1. Bump `__version__` in `app.py`
2. Bump `AppVersion` in `deck_manager.iss`
3. Run `./build_windows.sh`

---

## Project Structure

```
deck-manager/
├── app.py                  # Main GUI (Tkinter) — all UI and tab logic
├── models.py               # Data classes: Meet, Session, PoolSession, Official, assignments
├── engine.py               # Deck assignment engine (ST split, CJ placement, relief, RTO, corners)
├── logic.py                # Pure business logic (shirt eligibility, CJ pool distribution, DR/SR pool distribution)
├── exporter.py             # PDF exports: deck set, sign-in sheet, CJ matrix, event order, shirts
├── importer.py             # CSV import: Google Forms, SignUpGenius, generic
├── event_file_parser.py    #  EV3 and HYV event file parser
├── sanction_parser.py      # swimming sanction PDF parser
├── deployment.py           # Deployment variation table calculations
├── palette.py              # Single source of truth for all brand colours (hex strings + ReportLab objects)
├── _pool_page.py           # CJ matrix single-pool page builder
├── storage.py              # JSON save / load / autosave; platform-aware data directory
├── Sample-Data/            # Sample event files and CSV for testing (bundled with installer)
├── LICENSES.txt            # Third-party license notices (bundled with installer)
├── tests/                  # pytest test suite (models, engine, logic, storage, importer)
├── deck_manager.spec       # PyInstaller build spec
├── deck_manager.iss        # Inno Setup installer script
├── build_windows.sh        # One-command build script (Git Bash)
├── requirements.txt        # Python dependencies
└── LICENSE                 # Deck Manager Software License
```

---

## Testing

```bash
pip install pytest
pytest tests/ -v
```

The test suite covers models, engine, logic, storage, and importer without
requiring a display or Tkinter. See `tests/README.md` for details.

---

## License

Copyright (c) 2026 Michael Berger.
Licensed under the Deck Manager Software License — non-commercial use only.
Modifications must be published back to the license holder.
See [LICENSE](LICENSE) for full terms.

Third-party licenses are listed in [LICENSES.txt](LICENSES.txt).
