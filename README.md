# AirComPlus Minimal — TEST build

> **Altitude datum caveat (2026-11-11):** follow-up validation showed that ordinary, non-Mock GPS `Location.getAltitude()` is displayed without WGS84-to-MSL conversion. This build is **NO-GO as MSL-correct for ordinary GPS altitude**. The Mock GPS conversion path and BARO behavior were checked separately.

Temporary public download page for the isolated AirComPlus-Minimal test build.

## Download

Download `AirComPlus_Minimal_TEST.apk` from the latest GitHub Release assets.

- Package: `ru.aircom24`
- Version: `2.25A`
- Build: test-signed; certificate SHA-256: `0f7761f59f89b9cc2b40aa9060710cf7f944d611b85527b98d25c6995a16738c`
- APK SHA-256: `4BBB11DC61B126548EC275F06C577A8A1FB7E4277B91B7C43252079A7DD4FDC3`

See `AirComPlus_Minimal_Integration_Report.md` for scope, AVD results, and limitations. The build was validated on a dedicated API 37 AVD. It is for review and non-flight testing only; it is not a production or flight release.

This repository is intentionally temporary and public so the owner can download the APK. Ask the owner before sharing it elsewhere; make the repository private or remove it when the temporary download is no longer needed.
