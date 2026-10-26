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

Midnight Suns is a late PS4 **Unreal Engine 4** release. Adapting the inherited runtime required more than changing the launcher or adding graphics switches: the game's startup and gameplay exercised PS4 kernel, memory-pool, filesystem and NP Toolkit interfaces that needed additional compatibility work. The items below describe the game-specific additions and corrections, not a claim that the entire PS4 SDK or online stack is implemented.

### PS4 services and engine startup

- **Kernel, threads and synchronization:** extended the inherited guest-thread support with current-thread ID queries, POSIX read/write-lock compatibility, and thread-exit / C++ destructor handling. CPU-mode queries use the game's SFO attributes, while the Neo/PS4 Pro query reports the runtime's base-PS4 profile rather than confusing the phone's CPU count with the emulated console mode.
- **Filesystem and content access:** corrected directory enumeration to the compact PS4/FreeBSD directory-entry layout actually consumed by this executable, preserving real directory contents and iteration. Added DiscMap sysmodule/query compatibility for the extracted-file game layout.
- **Asynchronous-I/O startup:** implemented parameter initialization and validation/acceptance of the game's low-, medium- and high-priority AIO configuration. This is initialization compatibility; it should not be read as a claim that every PS4 AIO submission API was newly implemented.
- **Game-provided NP Toolkit2:** adjusted import resolution so the game's own linked toolkit code, constructors and state can execute, instead of replacing toolkit startup with a fabricated successful initialization.
- **Toolkit service dependencies:** added the required NetCtl and UserService callback registration/check/unregistration aliases, plus NP Manager state-callback/check and account-ID compatibility. Registrations retain ownership/state and support cleanup; offline callback checks do not manufacture connection events or PSN authentication.
- **Offline service behavior:** account queries report signed-out status and VoiceQoS initialization reports an unavailable voice backend after argument validation. These paths let the game handle an offline runtime; **online PSN services and voice chat are not implemented**.
- **Supporting initialization/query interfaces:** added the required coredump-handler registration, heap-trace information, lifecycle and sanitizer-query compatibility used during engine startup. Handler registration is not a claim of a complete PS4 crash-dump implementation.
- Adapted the Android launcher, executable preparation and existing FEX/PS4 compatibility runtime to **CUSA26174 APP_VER 01.00**, with touchscreen controls and game-specific launch handling.

### Memory management and allocation stability

- **Real memory-pool backing:** implemented/extended pool expansion, virtual reservation and commit through the inherited direct-memory and guest virtual-memory backends, using the game's 64 KiB pool granularity. Successful commits require actual backing and mappings, rather than merely returning success to the engine.
- **Pool decommit and reuse:** corrected decommit/release bookkeeping and reuse of freed backing, with split/coalescing and ownership handling for fragmented allocations. Direct-memory free-range tracking and available-memory queries were needed so the engine could make allocation decisions against the runtime's real state.
- **Flexible-memory accounting:** supplied the available-flexible-memory query from the runtime's live usage counter rather than a fixed pretend amount.
- **Onion/Garlic metadata:** corrected virtual mapping metadata for Onion allocations while retaining the actual backing ownership. This addressed a mismatch between the memory type requested by the game and the type exposed through guest mapping queries.
- **Growing allocation tracker:** replaced the fixed-size live mapping metadata tracker with a growing tracker after allocation churn exhausted its record capacity. This preserves physical-allocation bookkeeping and removes a demonstrated metadata-limit failure; it is separate from running out of the phone's physical RAM.
- **Dedicated framebuffer arena:** added the optional, atomic, signature-checked **128 to 256 MiB** expansion. Backing allocation/mapping, accounting, buddy tracking and three address classifiers are updated together, including correct recognition and cleanup of allocations in the expanded range. This targets the observed framebuffer allocation failures. The additional 128 MiB comes from the **existing guest direct-memory budget**, so total guest memory is not increased and other allocations have correspondingly less headroom.

### Rendering, controls and packaging

- Added the optional **skylighting-off engine path** used to address the green lighting contamination, alongside the inherited shadPS4-derived Vulkan renderer and retained game-specific rendering compatibility work.
- Added a separately toggleable **synchronous small-buffer copy** control using the existing runtime optimization-mask mechanism. It changes when small read-only host copies occur; it is not a global GPU wait or a replacement renderer.
- Kept the experimental shadow, eye-adaptation and SSAO switches independent and reversible. They are **not enabled in the recommended configuration**.
- Rebranded the launcher with Midnight Suns artwork, text, icon references and export filenames. Alpha106 retains **the exact Alpha105 runtime payload and all six patch definitions**; its changes are launcher branding/version text only.

The working lighting/flicker configuration still requires **FSR/upscaling OFF, Native resolution, 16x anisotropic filtering**, and the three named patches listed above. Memory-service support, the framebuffer patch and the rendering workarounds solve different parts of getting this title running; they should not be treated as one universal UE4 fix.

Signature, archive integrity, package/version and native-library alignment checks passed. The release APK is the exact promoted master, not a rebuild. No added rendering capture or shader-dump diagnostics are enabled; ordinary runtime logs remain.

## Testing and limitations

Developed with **AI assistance (ChatGPT/Codex)**, with human-directed development and on-device testing. The tester reported correct graphics, no remaining flicker with the required configuration, and a period of play without crashes after the framebuffer fix. The complete campaign has **not** been validated; later bugs, crashes, freezes or rendering issues may remain. Alpha106 is the tester-designated master and preserves the working Alpha105 core; there is no separate full-campaign validation for Alpha106.

Test device: **REDMAGIC 10S Pro, Android 16, Snapdragon 8 Elite Leading Version, Adreno 830, 24 GB RAM**, using **Rutabaga 26.10.10**. Results on other devices or drivers are unverified.

**Driver recommendation: for better results, use the latest Rutabaga Turnip Vulkan driver compatible with your device.** The version used for the reported testing was Rutabaga 26.10.10; newer driver versions have not been individually validated here.

Install over an existing Midnight Suns build without uninstalling or clearing app data to retain saves, settings and game-folder selection. Keep the graphics settings and three required patches above. Supply your own legally obtained supported game dump. No game executable or installable game dump is included.

### APK SHA-256

`ca6f9540ddae9f40434001b463132acb895de5ef3ea54865e32b262333094f84`
