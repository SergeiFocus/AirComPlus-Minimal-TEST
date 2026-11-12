# AirComPlus Minimal — TEST build

Updated 2026-11-13: Mock course retention.

[Download AirComPlus_Minimal_TEST.apk](https://github.com/SergeiFocus/AirComPlus-Minimal-TEST/releases/download/v2.25A-test/AirComPlus_Minimal_TEST.apk)

- Package: ru.aircom24
- Version: 2.25A
- APK SHA-256: 86193149C00FECACCEFB3BBC715613F441030AD58480D19780E1BF6E46157212
- Test certificate SHA-256: 0f7761f59f89b9cc2b40aa9060710cf7f944d611b85527b98d25c6995a16738c

A Mock fix with positive speed but no bearing previously overwrote a valid course with zero. The latest build retains the last accepted course during slowdown, a stop, and GPS loss. A supplied zero-degree course during motion remains valid.

The previous C741 build reproduced the fault. The exact new APK passed ten controlled callback assertions in the installed app on a dedicated API 37 AVD, including stock GPS-loss detection and course retention. Automatic INS entry was disabled in that run and is not claimed as tested. The fixture did not exercise external UBX/Bluetooth or Android provider delivery.

Full navigation release remains **NO-GO**. Ordinary non-Mock GPS altitude still has the documented WGS84/MSL limitation. Background/resume recovery and complete source handover require separate validation. Source selection, the existing ten-second Mock freshness window, altitude conversion, and the INS algorithm were not changed in this update.

This temporary public repository provides a test-signed APK for ground review. It is not a production or flight release. The integration report in the repository records earlier work; the release notes describe the current binary.
