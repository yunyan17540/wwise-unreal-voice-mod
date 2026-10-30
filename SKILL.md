---
name: wwise-unreal-voice-mod
description: Build, validate, install, roll back, and package Unreal Engine/Wwise voice-replacement mods, including WEM media, VO language banks, IoStore PAK/UCAS/UTOC containers, backups, runtime testing, and upload-ready releases. Use when the user asks to replace game voices, make a voice-pack Mod, edit Wwise audio, or publish a UE voice mod.
---

# Unreal/Wwise Voice Mod

Use this workflow for evidence-driven voice replacement in Unreal games using Wwise. Prefer a small reversible override over repacking the original game archive.

## 1. Establish scope before editing

1. Resolve `GAME_ROOT`, `MOD_ROOT`, project/workspace root, target language bank, and the exact game build.
2. Identify whether the game uses loose files, PAK, or IoStore (`.pak`, `.utoc`, `.ucas`).
3. Determine the shipped Wwise version from the game or existing assets; use a matching official encoder when possible.
4. Confirm the source audio may be used. Preserve attribution and do not imply ownership of third-party voices.
5. Read existing project notes and inspect current Mod files, active processes, and unrelated appearance/configuration Mods.
6. Create a small evidence ledger: observation, source path, action, result, confidence, and remaining unknowns.

Do not treat an event name, filename, or similar duration as proof of a runtime mapping. Distinguish:

- Wwise Event ID
- Sound/Action/Container IDs
- Media/source ID
- Bulk offset and slot size
- Runtime playback evidence

## 2. Map the target voice events

1. Extract or inspect the language bank and list the target Events.
2. Resolve Event → Action/Sound/Container → Media → bulk offset relationships.
3. Group references by semantic role, for example `attack`, `hurt`, `charge`, `heavy`, `special`, or `breath`.
4. Prefer groups with direct protagonist referencers. Exclude zero-referencer or ambiguous groups until new evidence exists.
5. Keep continuous breathing loops separate; do not add them merely because short combat grunts are available.
6. Start with one isolated event group, test it in game, then expand. Never restore a previously failed/crashed package without re-auditing it.

## 3. Prepare replacement audio

- Keep original source files immutable and record source URL, attribution, hash, language, and rights status.
- Convert working material to 48 kHz, mono, 16-bit PCM WAV unless the target explicitly requires another format.
- Remove only unwanted leading/trailing silence; add a short fade to prevent clicks.
- Do not change pitch, speed, or timing unless the user explicitly requests it.
- Do not use TTS or voice cloning without explicit authorization and a suitable legal basis.
- Preserve a source manifest containing duration, sample rate, channels, peak/clipping status, processing gain, and SHA-256.

## 4. Encode and validate WEM files

1. Use the matching official Wwise external-source encoder and record its version/build and executable hash.
2. Decode every generated WEM with a format-aware decoder such as `vgmstream`.
3. Verify codec/profile, sample rate, channel count, sample count, Vorbis setup identity, no hard clipping, and successful PCM decode.
4. Compare decoded PCM against the intended source. A successful encoder exit alone is not enough; if the encoder returns non-zero but produces files, continue only when every expected output exists and all validations pass, recording the warning.
5. Do not fabricate WEM headers or copy arbitrary allocation parameters from another file.

## 5. Replace fixed-size media safely

For each selected original slot:

1. Read the original WEM profile and record the exact offset, slot size, cache/hash identity, and original SHA-256.
2. Choose a replacement that fits the original slot. If it does not fit, select a shorter source or stop; never silently resize adjacent media.
3. Preserve the original Wwise `hash` chunk identity when the target format requires it.
4. Append a valid `JUNK` chunk for remaining space, update the RIFF size, and assert the final byte length equals the original slot.
5. Decode the padded replacement again and confirm the PCM matches the unpadded encoded output.
6. Assert every byte outside declared replacement intervals is unchanged.
7. Keep the original `.uasset` and Event/Bank metadata byte-identical unless a complete, format-aware rebuild is required and separately validated.

## 6. Build an Unreal override package

1. Stage files outside the live game directory.
2. Reuse the original container version and dependency/store metadata when safe; never guess a ContainerHeader.
3. Use `retoc` or the target's format-aware packer to build IoStore chunks and the `.utoc`.
4. Validate with container info/list/verify and extract each chunk back for SHA-256 comparison.
5. Add only a minimal harmless PAK marker if a PAK companion is required; validate it with the game's UnrealPak `-Test`.
6. Write a package manifest containing version, target build, overridden asset paths, media count, source/candidate hashes, encoder identity, and runtime status.

Do not call offline container validation runtime proof. Keep `runtime_verified=false` until the game actually loads and plays the replacement.

## 7. Backup and install

Before any live mutation:

- Confirm the game process is stopped.
- Hash every existing voice package and copy the complete previous package to a timestamped rollback directory.
- Verify the backup hashes before removing anything.
- Snapshot appearance Mods, shared configuration, save-related files, and the base game container hash.
- Stage and hash-check all incoming `.pak`, `.ucas`, and `.utoc` files outside the game.
- Remove only the previous package owned by this workflow; refuse unexpected `AzusaVoice` packages.
- Copy all companion files together and hash-check each installed file.
- Re-check protected files and record zero mismatches.

Never delete or overwrite an unrelated Mod, the original game archive, a save, or a shared config as a shortcut.

## 8. Runtime verification

Test with the target language enabled:

1. Launch and load a save.
2. Trigger the isolated target event repeatedly.
3. Test each expanded role: attack, hurt, charge/heavy, special, and scene/load transition.
4. Check for original voice overlap, missing sound, wrong actor, clipping, crashes, and persistence after reload.
5. Record exactly what was observed. User confirmation is valid runtime evidence, but do not claim more coverage than the user tested.
6. If failed, stop the game, roll back only this package, and preserve logs and hashes.

## 9. Make an upload-ready release

Release only the payload and public documentation:

- Three companion game files: `.pak`, `.ucas`, `.utoc`
- `README_发布说明.md` or equivalent install/uninstall guide
- `SOURCE_NOTICES.md` with game/audio attribution and takedown terms
- `CHANGELOG.md`
- `release_manifest.json`
- `SHA256SUMS.txt`

Do not include local absolute paths, credentials, encryption keys, private source dumps, rollback backups, developer caches, or unrelated Mods. Label runtime status accurately. Include the release disclaimer from [release-template.md](release-template.md), then validate the ZIP contents and test its archive integrity.

## Required final report

Report:

- Package version and exact installed/release filenames
- Media/reference counts and target roles
- Encoder and container validation results
- Runtime status and what was actually tested
- Backup/rollback location
- Protected-file mismatch count
- Release ZIP path and SHA-256
- Remaining limitations or rights concerns

## Hard stops

Stop instead of guessing when:

- the target event/media relationship is ambiguous;
- no matching Wwise encoder or decoder is available;
- a replacement cannot fit its fixed-size slot;
- the game is running during installation;
- the current package or backup hash does not match the recorded state;
- an unexpected package would be removed;
- the source audio's redistribution rights are unclear.
