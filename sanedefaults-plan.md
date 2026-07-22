# Portable macOS Defaults and Restore Plan

## Source and deliverables

- Source: macOS 26.5.2 (build 25F84), Apple M2 MacBook Air (Mac14,2).
- `sanedefaults.txt`: commented zsh script containing the curated preferences and backup-first application workflow.
- `restore.txt`: generated on the target Mac immediately before preferences are changed; never stored as a generic prebuilt file.
- `restore-YYYYMMDD-HHMMSS.txt`: timestamped preservation of an older restore point.
- `Readme.md`: usage, restoration, modification, and safe-testing documentation.

## Preference export

1. Read a curated whitelist of portable, user-facing global, keyboard, mouse, trackpad, Finder, Safari, Dock, Mission Control, and screenshot preferences.
2. Exclude histories, recent items, window positions, analytics values, tokens, device identifiers, opaque binary data, and other volatile state.
3. Emit explicit, typed `defaults write` commands with an explanatory comment immediately before each command.
4. Label built-in trackpad settings as laptop-only and distinguish external Apple trackpad and mouse preferences.
5. Normalize source-home paths to `$HOME`; do not silently embed user-specific absolute paths.
6. Identify requested settings whose source key was absent and only emit an inferred effective value when the macOS 26 behavior is known.

## Safe application and restore generation

1. `--check` prints every planned command without prompting, creating or replacing `restore.txt`, restarting processes, or changing preferences.
2. Normal application requires the user to type `CLOSED`, then independently verifies that Safari is not running. Failure or cancellation exits before backup or modification.
3. Before the first preference write, capture the existence, exact scalar type, and value of every property that will be changed. Deduplicate keys if a future edit writes one more than once.
4. Generate `restore.txt` through a temporary file in the script directory. For an existing property, emit a typed command restoring its former value; for an absent property, emit an idempotent `defaults delete` command.
5. Support boolean, integer, float, and string restoration. Stop before applying anything if a previous value has an unsupported complex type or cannot be read safely.
6. Validate the temporary restore script with `zsh -n`, set permissions to `600`, archive an existing restore point with a timestamp, and atomically install the new `restore.txt`. Any failure before installation leaves preferences unchanged.
7. Only after restore validation succeeds, execute the collected preference commands. Stop on a failed write and retain `restore.txt` so a partial application can be reversed.
8. Restart Finder, Dock, and SystemUIServer after application; never force logout or restart. Explain when a logout may still be required.

## Restore safeguards and behavior

- Bind each generated restore file to the originating Mac hardware UUID, user ID, account name, and macOS major version. Refuse to run when any safeguard differs.
- Require the user to type `CLOSED` and verify that Safari is stopped before restoring any property.
- Restore only the properties managed by `sanedefaults.txt`; do not replace complete preference domains or unrelated settings.
- Make absent-key deletions harmless on repeated restoration.
- Restart Finder, Dock, and SystemUIServer after restoration and report possible logout requirements.
- Warn and exit when applying `sanedefaults.txt` on a different macOS major version unless the reviewed operation uses `--force`.

## Verification and safe testing

1. Require `zsh -n sanedefaults.txt` and a successful `zsh sanedefaults.txt --check`; confirm the latter emits all preference commands and creates no restore file.
2. Audit that every preference write has an immediately preceding comment and an explicit supported type.
3. Exercise restore generation from a temporary script copy that exits before applying settings. Verify syntax, `600` permissions, one restore action per managed property, timestamped rotation, correct directory placement, and the wrong-Mac refusal path.
4. Perform end-to-end testing in a disposable standard macOS user account, without `sudo`: establish recognizable baseline settings, apply, inspect `restore.txt`, verify representative UI changes, restore, log out and back in, and confirm the baseline returns.
5. In the disposable account, test repeated restoration, repeated application and archival, recovery after an interrupted application, and deletion of a property that was originally absent.
6. Keep a current Time Machine backup as a last-resort recovery path before testing on the primary account.

## Acceptance criteria

- No preference write can occur before a complete, validated restore point exists.
- `--check`, failed Safari confirmation, failed process verification, incompatible prior types, and failed restore validation make no preference changes.
- `restore.txt` accurately reverses every managed property to its pre-application value or absence and cannot be used accidentally on another Mac, account, or macOS major version.
- Scripts and documentation explain application, restoration, safeguards, limitations, and the recommended disposable-user test procedure.
