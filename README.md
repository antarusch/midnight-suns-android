# Marvel's Midnight Suns — Android runtime

## Download

**Alpha106 — the current Midnight Suns master runtime.** Download `Midnight_Suns_Android_Alpha106_Branded.apk` below. `SHA256SUMS.txt` contains the verified APK checksum.

Supported game: **Marvel's Midnight Suns, PS4 CUSA26174, APP_VER 01.00**. The SFO's separate VERSION 01.02 field does **not** mean an update is installed. This is a game-specific Android runtime adaptation, not a general-purpose PS4 emulator.

## IMPORTANT — required graphics settings

**Disable FSR / upscaling. Set Upscaler to OFF. Leaving upscaling enabled can cause the scene to flicker.** Use the configuration confirmed by the tester:

| Setting | Required configuration |
| --- | --- |
| Upscaler / FSR | **OFF** |
| Resolution | **Native** |
| Anisotropic filtering | **16x** |
| Sharpening | **OFF** |

The tester reported that flickering finally disappeared with upscaling disabled and anisotropic filtering set to 16x, together with the runtime/patch configuration below. These results establish the working combination, not the isolated contribution of each setting.

## IMPORTANT — enable these three patches

Open **Patches** and enable **all three of these exact entries**:

1. **Disable Skylighting - Experimental Workaround** — addresses the incorrect sky-light contribution / green tint.
2. **Runtime Fix - Synchronous Small Buffer Copies** — uses immediate host copies for small read-only buffers instead of deferred copies; retained in the confirmed flicker-free configuration.
3. **Expand Framebuffer Arena to 256 MiB - Experimental** — expands the game's dedicated framebuffer arena, addressing the observed framebuffer allocation crashes.

Use the names above rather than relying on menu positions. **Leave Disable Dynamic Shadows, Disable Eye Adaptation and Disable Screen Space Ambient Occlusion unchecked.** Disabling dynamic shadows previously made character faces look incorrect and did not resolve the flicker.

**Stop the game completely, change the settings/patch selections, then launch again.** The patches are reversible: stop, uncheck the relevant option and restart. All three options remain user-selectable; installing the APK alone does not automatically enable them.

## Foundation and thanks

This project is a heavily modified adaptation of the **God of War III Android runtime by [zFitness and contributors](https://github.com/zFitness/gow3_android)**. Huge thanks to zFitness and everyone who created, maintains and contributes to that runtime: their work made this adaptation possible.

Credit and thanks also belong to:

- **[shadPS4 and its contributors](https://github.com/shadps4-emu/shadPS4)** for the PS4 emulation and Vulkan/shader-translation foundation used by the derived graphics runtime.
- **[FEX and its contributors](https://github.com/FEX-Emu/FEX)** for x86-64 to ARM64 translation.
- The Vulkan driver developers and all other upstream dependency authors.

This is an independent adaptation. Original copyrights, licenses and attribution remain applicable; upstream `gow3_android` is GPL-2.0-or-later and dependencies retain their respective licenses. Midnight Suns and the launcher artwork belong to their respective rights holders; artwork was sourced from the [official 2K Newsroom](https://newsroom-es.2k.com/noticias/cae-la-oscuridad-alzaos-marvels-midnight-suns-de-firaxis-games-estara-disponible-mundialmente-en-marzo-de-2022). No affiliation with or endorsement by Marvel, 2K or Firaxis is implied.

## What was implemented for Midnight Suns

- Adapted the Android launcher, game preparation and existing FEX/PS4 compatibility runtime to **CUSA26174 APP_VER 01.00**, with touchscreen controls and game-specific launch handling.
- Corrected **Onion memory mapping metadata** and replaced the fixed-size live mapping tracker with a growing tracker, removing a demonstrated metadata-capacity failure while preserving physical allocation bookkeeping.
- Added rendering compatibility workarounds, including conservative image-format / compute-clear handling and the optional **skylighting-off engine path** used to address the green lighting contamination.
- Added a separately toggleable **synchronous small-buffer copy** control using the existing runtime optimization-mask mechanism. This is not a global GPU wait or a replacement renderer.
- Added an atomic, signature-checked **128 to 256 MiB framebuffer arena** patch: backing allocation/mapping, accounting, buddy tracking and three address classifiers are updated together. The extra 128 MiB comes from the existing guest memory budget; total guest memory is not increased.
- Kept the experimental shadow, eye-adaptation and SSAO switches independent and reversible. They are **not enabled in the recommended configuration**.
- Rebranded the launcher with Midnight Suns artwork, text, icon references and export filenames. Alpha106 retains **the exact Alpha105 runtime payload and all six patch definitions**; its changes are launcher branding/version text only.

Signature, archive integrity, package/version and native-library alignment checks passed. The release APK is the exact promoted master, not a rebuild. No added rendering capture or shader-dump diagnostics are enabled; ordinary runtime logs remain.

## Testing and limitations

Developed with **AI assistance (ChatGPT/Codex)**, with human-directed development and on-device testing. The tester reported correct graphics, no remaining flicker with the required configuration, and a period of play without crashes after the framebuffer fix. The complete campaign has **not** been validated; later bugs, crashes, freezes or rendering issues may remain. Alpha106 is the tester-designated master and preserves the working Alpha105 core; there is no separate full-campaign validation for Alpha106.

Test device: **REDMAGIC 10S Pro, Android 16, Snapdragon 8 Elite Leading Version, Adreno 830, 24 GB RAM**, using **Rutabaga 26.10.10**. Results on other devices or drivers are unverified.

Install over an existing Midnight Suns build without uninstalling or clearing app data to retain saves, settings and game-folder selection. Keep the graphics settings and three required patches above. Supply your own legally obtained supported game dump. No game executable or installable game dump is included.

### APK SHA-256

`ca6f9540ddae9f40434001b463132acb895de5ef3ea54865e32b262333094f84`
