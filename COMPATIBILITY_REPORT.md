# CNCMaps Renderer × Mental Omega 3.4.0 — Compatibility Report

Date: 2026-11-19. Scope: the 14 uploaded files. **No Mental Omega 3.4.0 files (MIX archives, rules/art INIs, maps) were supplied**, so Phase 2 comparison and Phase 5 render testing could not be performed.

## 1. Verified facts (checked in this session)

| # | Fact | How verified |
|---|------|--------------|
| 1 | The binaries are **CNCMaps 2.4.0.0** (assemblies CNCMaps.Engine/FileFormats/Shared/Renderer.GUI, NLog, OpenTK 3.0.1.0), .NET Framework 4.0 (CLR v4.0.30319), 32-bit. | .NET metadata parsed with `dnfile` |
| 2 | `CNCMaps.Renderer.exe` is a 6 KB stub (1 class `Program`); the work is in `CNCMaps.Engine.dll`. | metadata |
| 3 | The binaries correspond to the open-source project `zzattack/ccmaps-net` (namespaces/types match), so editable source **does exist upstream**, but not inside the uploaded files. | type listing vs. cloned repo |
| 4 | Upstream `dev` is now at **3.1.0**, targets **.NET 10** (GUI: `net10.0-windows`), and has release notes covering depth/shadow/lighting/terrain fidelity work. Your 2.4.0 build predates this. | cloned repo, `Directory.Build.props`, `RELEASE_NOTES.md` |
| 5 | Engine types present in 2.4.0: Rules/Art type collections (Building/Vehicle/Infantry/Aircraft/Overlay/Smudge/Terrain), `Theater`, `TileCollection`, `PaletteCollection`, `ShpRenderer`, `TmpRenderer`, `VxlRenderer`, `ThumbInjector`, `EngineDetector`. FileFormats: MIX (Blowfish/RSA key handling per the standard format), SHP, TMP, VXL, HVA, PAL, VPL, CSF, PKT, INI, map files. | metadata |
| 6 | `mo_config.xml` uses exactly the ModConfig schema of upstream (Name, Engine, Directories, ExtraMixes, CustomRulesIniFiles, CustomArtIniFiles, Theaters, ObjectOverrides, ExtraOptions). It differs from upstream's built-in YR defaults only by: `Directories=..\`, 4 ExtraMixes (`expandmo99/97/96/95.mix`), `rulesmo.ini`, `artmo.ini`, and theater INIs named `*mo.ini` instead of `*md.ini`. Theater mix lists, extensions, palettes and `NewTheaterChar` (84/65/85/78/68/76 = T/A/U/N/D/L) are identical to upstream YR defaults. | line-by-line diff against `ModConfig.DefaultsYR` |
| 7 | Six theaters configured: Temperate, Snow, Urban, NewUrban, Desert, Lunar. | `mo_config.xml` |
| 8 | `gui_settings.xml`: engine = YR, `modconfig=True`, `modconfigfile=mo_config.xml`, `mixdir=..\`, thumbnail 798×368 PNG, hardware voxels on, PNG/JPEG output off by default, update check skipped. | file |
| 9 | **`BattleClient.ini` and `MentalOmegaMaps.ini` are not renderer inputs.** No string in any uploaded binary, nor in the upstream source, references them. They are game-client data (campaign mission list; game modes + 708 skirmish map entries under `MapsMO\Standard`). | string scan of all binaries; `grep` of upstream |
| 10 | Each INI was uploaded twice; the copies are byte-identical (MD5 match). | `md5sum` |
| 11 | The only "3.3.6" strings are the two header comments (line 1 of each INI). Nothing in the configs or binaries hard-codes a game version. | grep |
| 12 | `check_mo_install.py` (delivered) works: reports OK on a complete synthetic install, exits 1 and names missing files otherwise. | run against synthetic folders |

## 2. Not verified / unknown

- Whether 3.4.0 changes any MIX names, rules/art INI names, theater INI names, or adds theaters/tiles/overlays. **Unknown — no 3.4.0 files.** I have not guessed any.
- Whether the 2.4.0 renderer can parse every 3.4.0 map, or whether new voxel/SHP features break it. Unknown.
- Whether `expandmo99/97/96/95.mix` is still the correct set and order in 3.4.0. Unknown.
- Behaviour of the GUI/renderer at runtime at all: **nothing was executed** (see §4).

## 3. Assessment per phase

**Phase 1 (analysis):** done as far as binaries allow. I did not decompile IL (no .NET decompiler available), but upstream source makes that unnecessary.

**Phase 2 (3.4.0 comparison):** blocked on input. Needed: the 3.4.0 game folder (or at least the top-level `*.mix`, `rulesmo.ini`, `artmo.ini`, `*mo.ini` theater files, and 2–3 sample `.map`s), plus which version the supplied maps came from.

**Phase 3 (renderer update):** the realistic route is **not** patching 2.4.0 binaries (no source inside them) but moving to upstream 3.1.0, which already provides: modern .NET, install-path detection via registry (RA2/TS keys) or `--mixdir`, PNG/JPEG + thumbnails, NLog logging, and a large rendering-accuracy overhaul. Your `mo_config.xml` should load unchanged (fact 6); this is schema-compatible but **untested**. Whether upstream handles Mental Omega's specific assets correctly is untested. Any MO 3.4.0-specific fix can only be written after seeing a failing render/log.

**Phase 4 (GUI):** upstream's GUI (WinForms, Windows-only) already has input/output selection, engine/theater options, mod-config editor, log output and thumbnail/PNG/JPEG options (visible in `gui_settings.xml` keys and `MainForm`/`ModConfigEditor` types). The upstream GUI source contains a Batch tab (verified by control names only, not run); I have not verified batch rendering, progress display, or version detection; rewriting the GUI now would be speculative, so I didn't.

**Phase 5:** see below.

## 4. Why there is no executable or test run

- This environment has no .NET SDK/Mono; NuGet and dotnet download hosts are blocked, so upstream cannot be built here.
- The GUI is Windows-only and the binaries are 32-bit .NET Framework; they can't be run on this Linux sandbox.
- No game assets or maps were provided to render.

So **no renderer feature is reported as working**. The only thing I ran and tested is `check_mo_install.py`.

## 5. Deliverables

- `COMPATIBILITY_REPORT.md` — this file.
- `mo_config.xml` — **unchanged** copy. I made no edits because nothing in 3.4.0 could be verified; editing would be guessing.
- `check_mo_install.py` — read-only pre-flight: confirms every file named in `mo_config.xml` exists in a game folder, flags unreferenced `expand*.mix` files (possible additions in a newer version), and prints a heuristic version hint. It checks file presence only, not contents of MIX archives.
- INI files: not modified. Retitling "3.3.6" → "3.4.0" in the comment would be cosmetic and the lists themselves must come from the real 3.4.0 client files.

## 6. Build instructions (upstream 3.1.0, to run on a Windows machine with internet)

```
git clone https://github.com/zzattack/ccmaps-net
cd ccmaps-net
# install .NET 10 SDK first
dotnet build CNCMaps.slnx -c Release
dotnet test  CNCMaps.Tests
```
Copy your `mo_config.xml` next to the built GUI/renderer, set the game folder (or pass `--mixdir`), and select it as the mod config. Check the exact CLI flags with `CNCMaps.Renderer --help`. Upstream's test suite includes golden-render tests, but those use its own assets, not Mental Omega's.

## 7. Missing dependencies / files needed to proceed

1. Mental Omega 3.4.0 game folder (or the files listed under Phase 2 above) — and the legitimate retail Yuri's Revenge base files that Mental Omega depends on (`ra2.mix`, `ra2md.mix`, `expandmd01.mix`, `language/…` etc.) if you want complete renders. I have not checked which are required.
2. 2–3 sample 3.4.0 `.map` files, including one using each theater you care about.
3. A machine with .NET 10 SDK (Windows for the GUI).
4. Renderer log output (console output or the GUI log pane) from any failing render — this is what would make a targeted 3.4.0 patch possible.

## 8. Recommended next step

Install upstream 3.1.0, run `check_mo_install.py` on the 3.4.0 folder, render one map with your existing `mo_config.xml`, and send me the log plus the 3.4.0 files. Then I can compare asset names and write config/code changes that are grounded in evidence.
