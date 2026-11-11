# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A repo for publishing iCalendar (`.ics`) calendars that the owner subscribes to from their own devices, to keep track of important dates. The owner updates the dates here and the devices pick the changes up through the subscription. It currently holds one calendar, `mexico_y_uefa_nations_league_2026-27.ics` (football), and more calendars are expected to be added later, each as its own standalone `.ics` at the repo root. There is no code, build, test suite or linter, and no generator script: edits are made directly to the `.ics` files.

Because devices are subscribed to the published files, don't rename or move an existing `.ics`, and don't change existing UIDs, without checking with the user first.

Everything below is documented from the football calendar. Mirror its conventions in any new calendar unless the user says otherwise.

## The football calendar

All user-facing text is in Spanish (`PRODID` is `-//Claude//Futbol Mexico + UEFA Nations League//ES`). It holds 161 `VEVENT`s in two kinds, described below: 5 Selección Mexicana matches and 156 UEFA Nations League 2026/27 league-phase matches. `X-WR-CALDESC` says the data is as of 23/09/2026, and every event's `DTSTAMP` is `20260923T120000Z`. Keep the two consistent if you refresh data.

## Validating changes

No iCalendar tooling is installed (the Python `icalendar` package is absent), so check the format with plain shell. Set `f` to the calendar being edited. On the football calendar these report 161, 0, 0 and 0:

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
- **Times are UTC (`Z`), with no `VTIMEZONE`.** The official local kickoff is written only in `DESCRIPTION`, so a time change must update `DTSTART`, `DTEND` and that text together. UEFA matches show `(CEST)` or `(CET)` and Mexico matches show `(CDMX)`. The same local kickoff maps to different UTC across the European DST change, so convert with the offset in force on that date (e.g. 20:45 CEST = `184500Z`, 20:45 CET = `194500Z`).
- **Duration:** every timed event has `DTEND` = `DTSTART` + 2h. All-day events use `DTSTART;VALUE=DATE` with the next day as the exclusive `DTEND`.
- **Order:** events are sorted ascending by `DTSTART`. Insert new ones in position.
- **Fixed properties on every event:** `STATUS:CONFIRMED`, `TRANSP:TRANSPARENT`, plus `CATEGORIES`. No `SEQUENCE` is used.
- **UIDs** have the form `<uuid5>@futbol-cal`. Their derivation is not documented anywhere and could not be reproduced from the match data, so never regenerate an existing UID. Calendar clients key updates on it. New events get a fresh UUID with the same `@futbol-cal` suffix.

## The two event kinds

**UEFA Nations League** (156 events, `CATEGORIES:UEFA Nations League`):
- `SUMMARY:⚽ <Local> vs <Visitante> · Nations League <grupo>`, with team names in Spanish.
- `DESCRIPTION:UEFA Nations League 2026/27\nLiga <A-D> · Grupo <A1…D2> · Jornada <1-6>\nHora oficial UEFA: HH:MM (CEST|CET)`.
- No `LOCATION` and no alarm.
- Shape of the data: 6 matchdays × 26 matches. Leagues A, B and C each have 4 groups of 4 teams (12 matches per group). League D has 2 groups of 3 (6 matches per group).

**Selección Mexicana** (5 events, `CATEGORIES:Selección Mexicana`):
- `SUMMARY:🇲🇽 …`, a `LOCATION` (stadium, city, state, country, with commas escaped), and `DESCRIPTION` ending `Hora oficial: HH:MM (CDMX)`.
- Timed events carry a `VALARM` (`ACTION:DISPLAY`, `TRIGGER:-PT30M`, text "Partido de México en 30 minutos").
- Four are friendlies (Fecha FIFA, Sep–Oct 2026).
- The fifth is a Concacaf Nations League quarterfinal second leg on 2026-11-17. It is a placeholder: all-day, "rival por definir", no alarm, and `(hora por confirmar)` in the `SUMMARY`. It is the one event expected to change once the opponent and kickoff are known. When updating it, keep its UID, convert it to a timed event, and add the `VALARM` like the other Mexico matches.
