# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A repo for publishing iCalendar (`.ics`) calendars that the owner subscribes to from their own devices, to keep track of important dates. The owner updates the dates here and the devices pick the changes up through the subscription. It currently holds one calendar, `mexico_y_uefa_nations_league_2026-27.ics` (football), and more calendars are expected to be added later, each as its own standalone `.ics` at the repo root. There is no code, build, test suite or linter, and no generator script: edits are made directly to the `.ics` files.

The files are published through GitHub raw URLs (`https://raw.githubusercontent.com/oupvroac/Calendars/refs/heads/main/<file>.ics`), so a change only reaches the devices once it is on `main`. Work on a feature branch does not show up on devices until it is merged. Because devices are subscribed, don't rename or move an existing `.ics`, and don't change existing UIDs, without checking with the user first.

Everything below is documented from the football calendar. Mirror its conventions in any new calendar unless the user says otherwise.

## The football calendar

All user-facing text is in Spanish (`PRODID` is `-//Claude//Futbol Mexico + UEFA Nations League//ES`). It holds 162 `VEVENT`s in two kinds, described below: 6 Selección Mexicana matches and 156 UEFA Nations League 2026/27 league-phase matches.

The owner mainly follows **México, España, Inglaterra and Alemania**; the other teams are included for completeness. Keep those four teams' matches verified and current, and don't remove the rest: the owner wants the matches of every country in UEFA national-team competitions in the calendar (interpreted as senior national teams; not clubs). Followed teams are marked in the `SUMMARY` with the team flag (see below).

**Alarms (`VALARM`, 30 minutes before) only for México, España and Inglaterra.** Alemania and every other team get no alarm. Note that the owner's iPhone subscription has "Eliminar alertas" switched on, which strips alarms coming from the file; they only fire if the owner turns that off (or sets default alerts on the device).

`DTSTAMP` is per event: it is the date the event was last created or edited (`20260923T120000Z` for events untouched since the first version, `20261111T120000Z` for the ones changed on 11/11/2026). `X-WR-CALDESC` states which data was verified and when: everything as of 23/09/2026, and México/España/Inglaterra/Alemania as of 11/11/2026. Update both when you refresh data.

## Validating changes

No iCalendar tooling is installed (the Python `icalendar` package is absent), so check the format with plain shell. Set `f` to the calendar being edited. On the football calendar these report 162, 0, 0 and 0 (`grep -c` exits with status 1 when its count is 0, so don't chain these lines with `&&`):

```bash
f=mexico_y_uefa_nations_league_2026-27.ics   # or any other calendar in the repo
grep -c '^BEGIN:VEVENT' $f                                              # event count
grep '^UID:' $f | sort | uniq -d | wc -l                                # duplicate UIDs
LC_ALL=C awk '{sub(/\r$/,""); if (length($0)>75) n++} END{print n+0}' $f   # lines over 75 octets
grep -vc $'\r$' $f                                                      # lines missing CRLF
```

## Format conventions (all hold across the whole football calendar)

- **CRLF line endings, folded at 75 octets.** A fold is `CRLF` plus one space. Folds land mid-word, which is normal, but never inside a multibyte UTF-8 character (emoji, `ó`, `·`). Count octets, not characters.
- **Text escaping:** `\,` `\;` and a literal `\n` for line breaks inside `DESCRIPTION`/`LOCATION`/`SUMMARY`.
- **Times are UTC (`Z`), with no `VTIMEZONE`.** The official local kickoff is written only in `DESCRIPTION`, so a time change must update `DTSTART`, `DTEND` and that text together. UEFA matches show `(CEST)` or `(CET)` and Mexico matches show `(CDMX)` (UTC-6 all year, since Mexico has no DST). The same local kickoff maps to different UTC across the European DST change, so convert with the offset in force on that date (e.g. 20:45 CEST = `184500Z`, 20:45 CET = `194500Z`).
- **Duration:** every event is timed and has `DTEND` = `DTSTART` + 2h. For an event whose time is not known yet, an all-day event (`DTSTART;VALUE=DATE`, next day as the exclusive `DTEND`) has been used as a placeholder.
- **Order:** events are sorted ascending by `DTSTART`. Insert new ones in position.
- **Fixed properties on every event:** `STATUS:CONFIRMED`, `TRANSP:TRANSPARENT`, plus `CATEGORIES`. No `SEQUENCE` is used.
- **UIDs** have the form `<uuid5>@futbol-cal`. Their derivation is not documented anywhere and could not be reproduced from the match data, so never regenerate an existing UID. Calendar clients key updates on it. New events get a fresh UUID with the same `@futbol-cal` suffix.

## The two event kinds

**UEFA Nations League** (156 events, `CATEGORIES:UEFA Nations League`):
- `SUMMARY:⚽ <Local> vs <Visitante> · Nations League <grupo>`, with team names in Spanish. When a followed team plays (España 🇪🇸, Inglaterra 🏴󠁧󠁢󠁥󠁮󠁧󠁿, Alemania 🇩🇪), the `⚽` is replaced by that team's flag, or by both flags separated by a space (home team first) when two followed teams meet, as Mexico does with 🇲🇽. 16 events are marked this way.
- `DESCRIPTION:UEFA Nations League 2026/27\nLiga <A-D> · Grupo <A1…D2> · Jornada <1-6>\nHora oficial UEFA: HH:MM (CEST|CET)`.
- No `LOCATION`. No alarm, except España and Inglaterra matches, whose alarm text is `Partido de <equipo> en 30 minutos` (`España vs Inglaterra en 30 minutos` when they meet).
- Shape of the data: 6 matchdays × 26 matches. Leagues A, B and C each have 4 groups of 4 teams (12 matches per group). League D has 2 groups of 3 (6 matches per group).

**Selección Mexicana** (6 events, `CATEGORIES:Selección Mexicana`):
- `SUMMARY:🇲🇽 …`, a `LOCATION` (stadium, city, state, country, with commas escaped), and `DESCRIPTION` ending `Hora oficial: HH:MM (CDMX)`.
- Every event carries a `VALARM` (`ACTION:DISPLAY`, `TRIGGER:-PT30M`, text "Partido de México en 30 minutos").
- Four are friendlies (Fecha FIFA, Sep–Oct 2026).
- Two are the Concacaf Nations League 2026/27 quarterfinal against Jamaica: the ida on 14/11/2026 at the National Stadium in Kingston (19:00 CDMX) and the vuelta on 17/11/2026 at Estadio BBVA (also called Estadio Monterrey) in Monterrey (20:30 CDMX). Confirmed on 11/11/2026 from Concacaf's schedule and press reports. The league-phase matchdays 5 and 6 of all 14 groups (52 matches) were also compared against UEFA's fixture list on that date, with no differences.

## Known pending updates

Re-check these after the dates shown; none of them can be written yet because the matches are not defined:
- **After 17/11/2026** (end of the UEFA league phase and of the Concacaf quarterfinals): the UEFA knockout and play-off draw is on 20/11/2026 (12:00 CET). The UEFA quarterfinals are two-legged, with the top two of each group qualifying: first legs 25–27/03/2027 (CET), second legs 28–30/03/2027 (CEST). Add the quarterfinals for España, Inglaterra and Alemania if they qualify, and for the others as appropriate. The finals are in June 2027 (exact dates and venue not confirmed when this was written).
- **UEFA Euro 2028 qualifying:** reported (not verified from an official source) to have its draw on 06/12/2026 in Belfast and its group stage between March and November 2027. No matches exist until the draw; add them once they are defined.
- If México beats Jamaica, the Concacaf Nations League Final Four is expected on 25–28/03/2027 (venue reported as SoFi Stadium; not verified from an official source).
