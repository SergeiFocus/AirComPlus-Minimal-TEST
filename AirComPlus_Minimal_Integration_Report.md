# AirComPlus-Minimal Integration Report

Date: 2026-11-11  
Status: **HOLD / NO-GO for MSL correctness in ordinary GPS mode** (Mock conversion and BARO behavior remain separately verified)

## Source and isolation

- Baseline: `input/Aircom_original.apk`, package `ru.aircom24`, version `2.25A`, min SDK 21, target SDK 35.
- Baseline SHA-256: `CFC101D5278623FD9E4A077BD07CA34CF2E84157A6A60F4FF543FFD4BE3F0892`.
- This file matches the documented stock original. The user's `/mnt/data/Aircom(1).apk` was not accessible on Kitchen; the identical-hash original already present at `E:\TrueNav\Aircom\Aircom.apk` was used and copied without modification.
- Work is isolated in `E:\TrueNav\Aircom\AirComPlus-Minimal`, branch name `AirComPlus-Minimal`. No files in the old `AirComPlus` project, TrueNav-GNSS-Android, source APK, or physical devices were modified or used for testing.
- The old AirComPlus project was consulted only for implementation notes and reports.

## Patch summary

- Added `src/main/java/ru/aircom24/AirComMinimalBridge.java` and integrated its compiled smali into the existing `smali_classes3` DEX so the app's existing multidex loader can load it.
- Patched `tracker.smali` to prefer a fresh, valid Android Mock Location for GPS coordinates, speed, and bearing; ordinary GPS is held while the Mock is fresh, and stock AirCom GPS is accepted after it expires or is removed.
- In GPS altitude mode, Mock `getAltitude()` is treated as WGS84 and converted once to MSL using Android's existing `AltitudeConverter` mechanism. A stale asynchronous result is discarded.
- In QNH/QNE BARO modes, the stock barometer and pressure-setting path remains active. INS, fused/network sources, UI, and unrelated location sources were left outside the patch.

## Build and signature

- Artifact: `build/AirComPlus_Minimal_TEST.apk`
- APK SHA-256: `4BBB11DC61B126548EC275F06C577A8A1FB7E4277B91B7C43252079A7DD4FDC3`
- Installed APK hash on the API 37 AVD matched the artifact hash.
- `apksigner verify`: v1, v2, and v3 signatures verified.
- Signer: `CN=AirComPlus Minimal Test, O=Test Build, C=RU`
- Certificate SHA-256: `0f7761f59f89b9cc2b40aa9060710cf7f944d611b85527b98d25c6995a16738c`
- Test build only; signed with a test certificate.

## AVD integration matrix

Tested on the dedicated `AirComPlus_Minimal_API37_Test` AVD, Android API 37. No physical device was targeted.

| Scenario | Result | Evidence |
|---|---|---|
| GPS without Mock | PASS — stock location callbacks continue. | `evidence/final_stock_gps.log` |
| Fresh Mock vs ordinary GPS | PASS — fresh GPS Mock was accepted; ordinary updates were held while it remained fresh; stock GPS resumed after removal/expiry. | Runtime log observations from the AVD run; fallback is also recorded in `evidence/final_stock_gps.log`. |
| Mock WGS84 altitude to MSL | PASS — injected 250 m WGS84 produced one MSL result of approximately 235.1451 m through the Android converter. | AVD log observation: `published Mock GPS MSL meters=235.1450980392157`. |
| BARO with active Mock | PASS — QNH and QNE were selected through the stock panel while a fresh Mock with altitude 250 m was active. The displayed pressure reference remained 1013 hPa and barometric altitude remained 0 m on the AVD; no Mock GPS MSL conversion was logged in BARO mode. | `evidence/baro_mock.png`, `evidence/qne_fresh.png`. |
| Remove Mock and return to stock GPS | PASS — after removing the test provider, ordinary AirCom GPS callbacks resumed with `mock=false`. | `evidence/final_stock_gps.log`, `evidence/final_stock_gps.png`. |

## GO decision and limits

The original five AVD scenarios were observed and the signed APK was verified byte-for-byte on the AVD. A follow-up ordinary-GPS altitude check found that a non-Mock Android `Location` with WGS84 altitude 250 m is displayed as 250 m without conversion. **The prior GO is withdrawn for MSL correctness in ordinary GPS mode.** The build remains available for source-priority and BARO review only; do not treat its ordinary GPS altitude as MSL. This is not a production or flight release.

The emulator's virtual pressure sensor remained at 1013 hPa, so this verifies preservation of the QNH/QNE BARO path when Mock is active, not real-world pressure accuracy. No flight testing was performed.

## Follow-up QNH selection check

After the integration matrix, with no Mock active on the same API 37 AVD, the QNH panel started at 1013 hPa and 0 m. Increasing QNH to 1020 hPa changed the displayed barometric altitude to 58 m. Selecting GPS showed a GPS altitude of 5 m; returning to QNH restored the 1020 hPa setting and 58 m barometric altitude. The selector outline tracked GPS → QNH → GPS → QNH. Evidence: `evidence/qnh_start.png`, `evidence/qnh_selected.png`, `evidence/qnh_1020.png`, `evidence/gps_after_qnh1020.png`, and `evidence/qnh_returned.png`. This confirms the selector and QNH setting persist across the tested mode changes on this AVD.

## Follow-up ordinary GPS altitude datum audit

On the API 37 AVD with no Mock active and GPS altitude selected, an ordinary emulator GPS fix at 37.4287, -122.0882 with `geo fix` altitude 250 m was accepted as `mock=false`; the UI displayed 250 m exactly. Evidence: `evidence/altitude_logic_stock_gps.png`. Static tracing confirms that only marked Mock GPS enters the `AltitudeConverter` path. Ordinary GPS falls through to `LocationWrapper.getAltitude()`, which Android documents as height above the WGS84 reference ellipsoid. Therefore ordinary GPS altitude is currently shown as ellipsoidal height rather than MSL. The earlier GPS-without-Mock scenario verified callback fallback only and did not verify the altitude datum. **MSL correctness for ordinary GPS is NO-GO pending a conversion fix and a new validation build.**
