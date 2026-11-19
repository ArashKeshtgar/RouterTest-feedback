# Vantiva Condor — MGA600BVBV5 (10.0.0.1)

The same web app as the Vantiva MGA5331VBV5, a newer build (web UI 26.1.q). Cards,
pop-ups and the login page match the Vantiva. Branch `refactor/drivers` of
RouterTest; see also handoff/2026-11-19-condor.md.

## Issues

### No internet / dead modem — "Airplane Mode", Not Registered  · by-design
- **See:** Mobile page shows `Power Mode: Airplane Mode`, `Network Status: Not
  Registered`, `wwan Disconnected`; IMEI never appears (stuck "Configuring").
- **Cause:** the modem is in airplane / low-power because it cannot initialise —
  SIM or modem fault. The APN is set (`fwach2.apn`), so it is **not** a customer
  setting, and a factory reset does **not** fix it (confirmed: reset several
  times, no change). There is **no radio / airplane / reset-modem control** in the
  Mobile pop-up to turn it back on.
- **Station:** detects "Airplane" fast (no 90 s wait), still runs the factory
  reset so the data is wiped (**DATA-WIPE PASS**), skips the Wi-Fi split and the
  mobile ping, and fails on the IMEI (**ROUTER-TESTING FAIL**). A dead unit now
  finishes in ~33 s (was ~10 min). Commit `f9c7215`.

### IMEI read too soon — modem still "Configuring"  · fixed
- **See:** IMEI FAIL right after login, Mobile shows `Device Status: Configuring`.
- **Cause:** the modem shows "Configuring" with no IMEI for a while after login or
  a reset; read too soon it looks like a missing IMEI.
- **Station:** waits up to 90 s and re-reads; and if it did not read before the
  reset (and it was not Airplane), it is INFO before the reset and re-read after
  it — the reset can bring a good-but-slow modem up, and that reading counts.
  Commit `a17b261`.

### Web page stuck "waiting to come up" — `/` is 503 logged out  · fixed
- **See:** `router web page: no answer / 503, waiting for it to come up`, never
  reaches login.
- **Cause:** Condor's `/` is a `.lp` home that answers **503 while logged out**
  (its landing page is `/login.lp`). The Vantiva's `/` answers logged out, so the
  old readiness probe (which used `/`) never passed on a Condor.
- **Station:** the readiness probe uses `/login.lp` for Condor (`ready_path`).
  Commit `a17b261`.

### Scanned label went to the Vantiva menu  · fixed
- **See:** a Condor label switched the model menu to "Vantiva", tested on
  192.168.0.1.
- **Cause:** the driver claimed Condor by a placeholder model code; the real code
  is `MGA600BVBV5`.
- **Station:** `CONDOR_MODELS = {"MGA600BVBV5"}`; the label now routes to Condor
  (10.0.0.1). Add more codes if a Condor label with a different code turns up.
  Commit `a17b261`.

### `.lp` pages stuck on 503  · by-design (power-cycle)
- **See:** every `.lp` page (`/`, `/login.lp`) answers 503 for minutes, while the
  static files (`/js/*.js`, `/css/gw.css`) answer 200. The SIM page can hang.
- **Cause:** too many cookie-less requests overload the `.lp` (Lua) engine
  (same as the Vantiva). A faulty SIM/modem querying can also hang it.
- **Do:** power-cycle the unit. The station checks with one session cookie to
  avoid making it worse.

### Wi-Fi split — Save button disabled  · open (fix not bench-confirmed)
- **See:** the Split toggles, but Save (`#save-config`) is disabled (`btn-disable`)
  and its click times out; 3 retries, ~3 min wasted, then it takes.
- **Cause:** the Save button stays disabled when the split is done in the session
  the test has been logged in and out of through the reset. On the bench, doing it
  in a **new incognito window** (a fresh session) enables Save.
- **Station:** starts a fresh browser session for the split (like a new incognito
  window), with a force-click fallback on the switch. Commit `f9c7215`. **Needs a
  good unit that reaches the split step to confirm** — the dead units skip it.
