# macSaneDefaults
Fixing all (some) of the bad macOS default settings

macOS has made some questionable choices in recent years with default settings. Those settings have also moved around a lot in the System Settings, so this is my attempt to preserve my settings and move them around different machines or VMs. 

# Portable macOS Defaults

This directory contains a commented zsh script that reproduces a curated set of macOS preferences. The settings were exported from macOS 26.5.2 on an Apple M2 MacBook Air and cover Finder, Safari, Dock, screenshots, text input, scrolling, and Apple mouse and trackpad gestures.

The script creates a machine-specific restore point before changing any preference. It does not back up histories, recent items, window positions, analytics data, device identifiers, or other volatile application state.

## Files

- `sanedefaults.txt`: the executable preference-application script.
- `restore.txt`: created on the target Mac immediately before settings are applied. It restores the values that existed on that Mac and deletes keys that were previously absent.
- `restore-YYYYMMDD-HHMMSS.txt`: an older restore point preserved when a new one is created.
- `sanedefaults-plan.md`: the original implementation and validation plan.

Although the scripts use a `.txt` extension, they are ordinary zsh scripts and can be inspected in any text editor.

## Preview changes

Quit Safari, open Terminal in this directory, and run:

```zsh
zsh sanedefaults.txt --check
```

Check mode prints the commands that would run. It does not change preferences and does not create or replace `restore.txt`.

## Apply the preferences

Quit Safari before applying the settings. Safari can overwrite its preferences if it remains open. The script pauses and requires you to type `CLOSED`, then independently checks that the Safari process is no longer running. If confirmation or verification fails, no restore file is created and no preferences are changed.

```zsh
zsh sanedefaults.txt
```

Before the first `defaults write`, the script:

1. Reads the existence, type, and value of every property it will change.
2. Creates `restore.txt` in the same directory as `sanedefaults.txt`.
3. Validates the restore script with `zsh -n`.
4. Gives the restore file owner-only permissions (`chmod 600`).
5. Archives an existing restore file with a timestamp.

If any property cannot be backed up safely or the restore script is invalid, the operation stops before changing preferences.

The script refuses to run on a different macOS major version by default because preference keys can change. After reviewing the commands, override that guard with:

```zsh
zsh sanedefaults.txt --force
```

## Undo the changes

Use the `restore.txt` generated on the Mac where the settings were applied:

```zsh
zsh restore.txt
```

Quit Safari first. Like the application script, `restore.txt` requires you to type `CLOSED` and verifies that Safari is no longer running before restoring anything.

The restore script checks the Mac's hardware UUID, user ID, account name, and macOS major version before changing anything. It refuses to run if those values do not match the environment where it was generated.

For each property, restoration does one of two things:

- If the property existed before applying `sanedefaults.txt`, it writes back the old value using its original type.
- If the property did not exist, it deletes the property so macOS can resume using its built-in default.

Finder, Dock, and SystemUIServer are restarted after applying or restoring preferences. Some Safari and input-device settings may require logging out and back in.

## Safest testing procedure

The recommended test environment is a temporary standard macOS user account on the same Mac. It isolates your main account's preferences while still exercising the real trackpad, mouse, Finder, Dock, Safari, and screenshot behavior. Do not use `sudo`; log into the test account normally and run the scripts as that user.

1. Make a current Time Machine backup of the Mac as a last-resort recovery option.
2. In **System Settings → Users & Groups**, create a temporary standard user.
3. Log into the temporary account.
4. Open Safari once, choose a few recognizable test settings, and then quit Safari completely. This ensures that the test account has Safari preference files and gives you known values to restore.
5. If macOS blocks access to Safari's preferences, grant Full Disk Access to Terminal or the application hosting the script in the test account, then restart that application.
6. Copy this directory somewhere owned by the test account and open Terminal there.
7. Validate and preview the script:

   ```zsh
   zsh -n sanedefaults.txt
   zsh sanedefaults.txt --check
   ```

8. Confirm that check mode lists the expected commands, creates no `restore.txt`, and changes no visible settings.
9. Quit Safari and apply the settings:

   ```zsh
   zsh sanedefaults.txt
   ```

   Type `CLOSED` when prompted. The script also checks the running processes before continuing.

10. Before relying on the restore point, inspect it and verify its syntax and permissions:

    ```zsh
    zsh -n restore.txt
    ls -l restore.txt
    ```

    The permissions should begin with `-rw-------`, indicating that only the owning user can read and modify it.

11. Check representative settings in System Settings, Finder, Safari, and the Screenshot interface:

    - Natural scrolling, tap-to-click, and gestures.
    - Finder filename extensions, hidden files, path bar, and desktop disks.
    - Dock size and magnification.
    - Screenshot destination.
    - Safari's download prompt and automatic opening behavior.

12. Quit Safari and restore the original test-account settings:

    ```zsh
    zsh restore.txt
    ```

    Type `CLOSED` when prompted.

13. Log out and back into the test account, then verify that the original settings returned.

### Optional stress tests

Perform these only in the disposable test account:

- Run `restore.txt` twice and confirm that the second run is harmless.
- Apply `sanedefaults.txt` twice and confirm that the older restore point is renamed to `restore-YYYYMMDD-HHMMSS.txt`.
- Interrupt application with **Control-C**, then use the already-created `restore.txt` to recover from the partial application.
- Confirm that an originally absent Finder property becomes absent again after restoration:

  ```zsh
  defaults read-type com.apple.finder AppleShowAllFiles
  ```

  If it was absent before application, this command should report that the property does not exist after restoration.

A macOS virtual machine offers stronger isolation but may not reproduce built-in trackpad or Magic Mouse behavior. The temporary-user test is therefore the best first test for this hardware-oriented script. If the test account becomes confused, it can be removed without changing the main account's preferences.

## Syntax checking

To check either script for shell syntax errors without executing it:

```zsh
zsh -n sanedefaults.txt
zsh -n restore.txt
```

Successful syntax checks normally produce no output. They verify shell structure but do not prove that a preference key is supported by every macOS version.

## Modifying the settings

Each preference command has a comment immediately above it. Add settings using the same form:

```zsh
# Explain what the preference changes and any hardware limitations.
apply_command defaults write DOMAIN KEY -bool true
```

Use an explicit type flag such as `-bool`, `-int`, `-float`, or `-string`. Keep all preference writes behind `apply_command`; direct `defaults write` commands would bypass automatic restoration.

The backup system currently supports previous values whose types are boolean, integer, float, or string. If an affected key contains an array, dictionary, date, or binary data, the script stops before making changes rather than generating an unreliable restore command.

When adding a laptop-only setting, place it inside the existing MacBook hardware check. Do not add volatile or private data such as browsing history, recent files, tokens, window positions, or opaque preference blobs.
