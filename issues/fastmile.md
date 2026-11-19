# Nokia FastMile Gateway 12 — 5G31-03W-B (192.168.1.1)

Label N `5G31-03W-B`, web app "(c) Nokia 2021-2024/2025". From the bench notes and
the handoff.

## Issues

### IMEI "N/A" — Cellular Status read too soon  · fixed
- **See:** IMEI N/A right after login.
- **Cause:** the Cellular Status was read before the modem filled it in.
- **Station:** waits for a real IMEI. Commit 391d37f. (Not yet re-seen on a real
  unit after the fix.)

### Old builds 1.2402 / 1.2404 — IMEI N/A, SIM None  · open
- **See:** on these firmware builds the IMEI reads N/A and the SIM shows None.
- **Status:** not yet worked through.

### Ping is slow after the reset — router boot  · by-design
- **See:** the ping to 8.8.8.8 takes ~1m40s to answer after the reset.
- **Cause:** that is the router booting. It builds its Overview data ~35-40 s after
  login. A fixed PC address (routertest-static) avoids ~3.5 min of DHCP pings.

### DATA-WIPE doubtful — unit answered the ping too soon  · open
- **See:** 4 units on 2026-11-13 answered the ping 1-31 s after "went down".
- **Cause:** the unit may not have fully reset/rebooted, so DATA-WIPE PASS is
  doubtful for those. Prove the reset by UpTime (log in after the ping, read the
  device info; UpTime under a few minutes = real reset). Not built yet.

### Gateway 3.2 5G (5G16-12W-A) — old app, no role attributes  · open
- **See:** its elements have no role attributes or button ids.
- **Cause:** an older build of the app; the Gateway 12's login and Network-map
  selectors do not carry over. Its login page and Network map are not captured yet.
