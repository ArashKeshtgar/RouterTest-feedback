# Router issues, per model

One file per router model: the defects and quirks seen on that model, what causes
them, and what the station does about each. The point is to recognise a known
problem fast on the bench and know whether it is the unit's fault or ours.

- [condor.md](condor.md) — Vantiva **Condor** MGA600BVBV5 (10.0.0.1)
- [vantiva.md](vantiva.md) — **Vantiva** MGA5331VBV5 (192.168.0.1)
- [fastmile.md](fastmile.md) — Nokia **FastMile** Gateway 12 (192.168.1.1)

## Status key

- **fixed** — the station handles it now.
- **open** — still needs work, or not yet confirmed on the bench.
- **by-design** — a real unit fault; the station reports it correctly (a FAIL is right).

## How to add an issue

Add a row to the model's file: a short title, what you see, the cause if known,
and the status. Put the station commit or the saved report file in reports/ when
there is one. Keep the newest at the top.
