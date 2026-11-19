# FastMile: cable drops, Wi-Fi name, fixed PC address (2026-11-19)

> خلاصه: تستِ اولِ امروز (واحد ...0D03) به‌خاطرِ قطع و وصل شدنِ کابلِ LAN خراب شد، نه SIM
> و Wi-Fi. همون واحد ساعت ۱۵:۴۵ کامل PASS شد. استیشن حالا قطعیِ کابل رو می‌فهمه و تست
> رو دوباره می‌گیره، اسمِ Wi-Fi که خونده رو تو لاگ می‌نویسه، و برای هر واحد آدرسِ IP
> ثابت می‌ذاره (آخرِ تست DHCP برمی‌گرده). همه commit و push شده، **ولی هنوز روی بنچ
> امتحان نشده**: اول استیشن رو ری‌استارت کن.

- Repo: https://github.com/ArashKeshtgar/RouterTest, branch `refactor/drivers`,
  commit **49e689e** (pushed).
- Logs: `station-log-2026-11-19-20-30-33.txt`, `station-log-2026-11-19-20-57-32.txt`.

## 1. SIM "none" + Wi-Fi 2.4/5/6 FAIL "not found" (unit ...0D03, 15:23)

**Cause: the station PC lost the cable link to the router**, not the unit.
`/var/log/kern.log` on the bench PC:

| time | station log | PC network |
|---|---|---|
| 15:23:40 | IMEI PASS (last good reading) | `enp0s31f6: NIC Link is Down` |
| 15:23:43-50 | | link down/up twice more, DHCP lost |
| 15:24:04-06 | SIM: modem not readable, all rows "none" | no address |
| 15:24:35 | | address back (192.168.1.149) |
| 15:26-15:30 | Wi-Fi 2.4/5/6 FAIL "not found", 2 min each | network fine |

The Overview page opened while the network was down stayed empty; the station
read that dead page for 3 x 2 min. At 15:22:15 the link came up "Half Duplex":
**check the cable / LAN port on that bench position.** Same unit, same
firmware (1.2504.00.0781), passed everything at 15:45. Code was compared with
the pre-refactor version: same logic, not a regression.

**Station now:**
- The cable is watched from login to the factory reset (`Network.arm_link`;
  Linux: `/sys/class/net/<card>/carrier_changes`, so a drop that is already over
  is still seen; Windows: the PC's address on the router network).
- A drop throws away the readings since, waits for link + PC address + router
  (`Timing.link_back_s` 180 s), signs in again and runs the tests again.
- A second drop stops the run: NOT TESTED "The cable link to the router dropped
  again", next step "Check the network cable and the router's LAN port".
- A Wi-Fi band row missing on Overview reopens Overview once; after that no
  more 2-minute waits.

## 2. "Wi-Fi name after reset" FAIL (unit ...062B, 15:57)

The reset looked real (down after 23 s, ping 1m40s like the passed units), but
the log only said "still not the label's name": no way to tell a unit that was
not wiped from a page that was misread. Same pattern on 16 Nov 22:09 (...05CC).

**Station now:**
- The name read is logged, shortened: `unit ...ABCD, label ...WXYZ`, before and
  after the reset.
- "No Wi-Fi name on the Overview page" is told apart from another name: then
  Overview is reopened once, 30 s waits instead of 2 min.
- On FAIL the Overview page is saved in `reports/`:
  `*-fastmile-overview-wifi-name-after-reset.html` and `.png`.
- **Found in the mock:** the reader takes the first text in the Wi-Fi card; with
  no name there it read "... GHz". If a bench log now shows `unit ... GHz`, that
  is the reader, not the unit.

**On the bench:** for ...062B, look at its Wi-Fi name on a phone: the label's
name = the reset worked (station misread); the customer's name = not wiped.

## 3. Fixed PC address for each unit (asked for today)

- Before each unit (Linux), the router card goes on NetworkManager profile
  `routertest-static`: address = router's /24 `.250` (FastMile 192.168.1.250,
  Vantiva 192.168.0.250, Condor 10.0.0.250), gateway = the router, DNS 8.8.8.8,
  autoconnect priority 100 (it must win when the reset drops the link).
- After the unit (any result) it goes back to the profile it was on
  (`Profile 1`, DHCP), without waiting for DHCP. Log lines: `NETWORK: ...`.
- If nmcli fails the card is left as it was and the test goes on (logged).
- `python3 -m routertest.station --dhcp` turns it off. Off in tests (they never
  touch the PC network). **Windows: not done** (log says so).
- Expected: ping after the reset faster (no ~1 min DHCP wait; 3m 37s cases gone).

## Tests

- New: cable watch (Linux / Windows), drop -> tested again and PASS, two drops
  -> NOT TESTED and no reset, Overview without Wi-Fi rows, Wi-Fi name details +
  saved page, no-name case, fixed address commands / restore / failure / Windows,
  each run switches and switches back.
- Mock router: `wifiRowsMissing`, `noWifiCard`.
- `tests/test_station.py test_pages test_qr test_end_to_end`: **112 pass**.
- Known: `test_core.py::test_only_vantiva_takes_a_typed_password` fails on the
  old code too (Condor also takes a typed password). Three demo tests failed once
  under load, passed alone.

## Next

1. Restart the station; run a few FastMile units. Look for `NETWORK: fixed
   address 192.168.1.250 on enp0s31f6 (was Profile 1)` and a shorter Ping.
2. Check the cable / port at the bench position (Half Duplex at 15:22).
3. Next "Wi-Fi name after reset" FAIL: read the name in the log and the saved
   Overview page.
4. Fixed address on Windows (netsh) if the Windows station is used.
5. Fix or update `test_only_vantiva_takes_a_typed_password` for Condor.
