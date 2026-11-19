# Vantiva — MGA5331VBV5 (192.168.0.1)

A `.lp` web app (web UI 23.1.b, "(c) Vantiva 2025"). From the bench notes of
2026-11-17 and the handoff. The Condor is the same app, newer — see condor.md.

## Issues

### `.lp` pages stuck on 503 after a reset  · by-design (power-cycle)
- **See:** after the reset the `.lp` pages answer 503 for minutes while the static
  files answer.
- **Cause:** many cookie-less checks overload the `.lp` engine.
- **Station:** checks with one session cookie, and asks the operator to power the
  unit off and on after ~4 min of 503.

### Logs the station out now and then  · fixed
- **See:** a step suddenly lands on /login.lp.
- **Station:** each step signs in again and retries (up to 5 tries). A Sign in is
  never repeated blindly — 3 wrong passwords lock the unit for a while.

### A used unit on another LAN  · fixed
- **See:** the unit is not at 192.168.0.1 (one was on 192.168.2.1).
- **Station:** tries the PC's gateway and uses the address that answers.

### QR will not scan — typed admin password  · fixed
- **See:** the label's QR does not scan.
- **Station:** the operator types the admin password (K); serial / MAC / Wi-Fi are
  then INFO and the IMEI is read from the router.

## Not yet confirmed on the bench
- The typed-password path, the gateway search, and the power-cycle question.
