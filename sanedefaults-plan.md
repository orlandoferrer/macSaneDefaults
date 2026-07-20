# Portable macOS Defaults Export Plan

## Source machine

- macOS 26.5.2 (build 25F84)
- Apple M2 MacBook Air (Mac14,2)
- Export file: `sanedefaults.txt`, runnable with zsh

## Procedure

1. Verify that Safari's privacy-protected preference container is readable. Stop rather than guess if Full Disk Access is unavailable.
2. Read a curated whitelist of portable, user-facing preferences from global, keyboard, mouse, trackpad, Finder, Safari, Dock, Mission Control, and screenshot domains.
3. Exclude histories, recent items, window positions, analytics values, device identifiers, opaque binary data, and other volatile state.
4. Generate explicit, typed `defaults write` commands. Put a descriptive comment immediately before each command and label hardware-specific settings.
5. Normalize paths inside the source home directory to `$HOME`; warn about non-portable external paths.
6. Add a macOS-major-version compatibility guard, a `--force` override, and a non-mutating `--check` mode.
7. Validate with `zsh -n`, exercise `--check`, and compare exported values with fresh preference reads without applying the script to the source Mac.

## Application behavior

- Require Safari to be closed before changing Safari preferences.
- Skip built-in trackpad settings on computers without a built-in trackpad and clearly label external pointing-device settings.
- Restart Finder, Dock, and SystemUIServer where needed; never force a logout or restart.
- For requested keys that are unset, only write an effective value when the macOS 26 behavior is known, and identify that inference in the comment.
- Warn and exit on a different macOS major version unless `--force` is supplied.

## Acceptance checks

- `sanedefaults.txt` passes `zsh -n`.
- `zsh sanedefaults.txt --check` performs no writes and prints the commands it would run.
- Every `defaults write` command has an immediately preceding explanatory comment and an explicit type.
- The file contains no source username in paths and no volatile or unreadable Safari values.
