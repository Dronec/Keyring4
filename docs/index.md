# Keyring4 User Manual

## 1. Overview

This device is a portable password vault with a touchscreen interface.

Main functions:

- Store up to 256 password slots
- Protect access with an 8-digit PIN
- Type stored passwords over USB HID keyboard emulation
- Organize slots with short labels and colours
- Generate new passwords on-device
- Import/export passwords over USB mass storage
- Store WiFi credentials for firmware updates
- Dim and auto-lock after inactivity

Each slot is numbered `001` to `256`.

## 2. First Start and Unlocking

### First boot

If no PIN has been stored yet, the device will ask you to:

1. Create PIN
2. Repeat PIN

If both entries match, the PIN is saved and the device unlocks.

If they do not match, the device returns to PIN creation and you must try again.

### Normal unlock

If a PIN already exists, the lock screen shows `Enter PIN`.

- Enter exactly 8 digits
- The keypad digits are randomized every time the PIN pad appears
- If the PIN is wrong, the device briefly shows `Wrong PIN` and then returns to the PIN entry state

## 3. Screen Lock and Idle Behavior

The device tracks touch activity.

- After the configured dim timeout, the screen dims
- After the configured lock timeout, the device locks and requires the PIN again
- On the slots screen, the `Lock` button locks the device immediately

Default timeouts:

- Dim timeout: 60 seconds
- Lock timeout: 120 seconds

## 4. Main Slots Screen

After unlocking, the device opens the slots screen.

Top bar buttons:

- `Lock`: lock immediately
- Colour filter: filter the list by colour
- Settings: open Settings

Slot list behavior:

- Tap a row to select a slot
- Long-press a row to open the slot action menu
- Empty or unlabeled slots are shown with dotted placeholders
- Coloured slots use a coloured row background

## 5. Slot Actions

Long-press a slot to open its menu.

Available actions:

- `Type`
- `Change label`
- `Set colour`
- `Create new`
- `Delete`
- `Cancel`

If the slot does not contain a password yet, only `Create new` and `Cancel` are enabled.

### Type

`Type` sends the stored password over USB keyboard emulation.

Important:

- The device must be connected to a host that has mounted it as a USB HID keyboard
- The firmware types the password characters only
- It does not automatically press `Enter` or `Tab`

### Change label

Labels are stored per slot.

Current device behavior:

- Maximum length: 15 characters
- The on-device label editor currently accepts only `A-Z` and `_`
- Empty labels are allowed

### Set colour

Available slot colours:

- `No colour`
- `Blue`
- `Green`
- `Red`
- `Violet`
- `Yellow`
- `Orange`

These colours are also used by the filter button on the main slots screen.

### Create new

This opens the password generator for the selected slot.

Generator options:

- Length from 8 to 64 characters
- `Use letters`
- `Use digits`
- `Use special chars`

The screen also shows an entropy estimate.

Notes:

- Default generator length is 16
- Generated passwords are built from the enabled character classes
- The current generator does not guarantee that every enabled class appears at least once
- Creating a new password stores it directly into the selected slot

### Delete

`Delete` clears the selected slot completely:

- password
- label
- colour

## 6. Settings

Open Settings from the gear button on the slots screen.

Available settings:

- `Lock Timeout`
- `Dim Timeout`
- `Flip screen`
- `Change PIN`
- `USB Import / Export`
- `Firmware update`
- `Factory Reset`
- `Security Status`
- `About`

### Lock Timeout

Available presets:

- 30 s
- 60 s
- 120 s
- 300 s
- 600 s

### Dim Timeout

Available presets:

- 5 s
- 15 s
- 30 s
- 60 s
- 90 s

The device always keeps dim timeout earlier than lock timeout.

### Flip screen

`Flip screen` rotates the display by 180 degrees.

The setting is applied immediately.

### Change PIN

To change the PIN:

1. Enter the current PIN
2. Enter the new PIN
3. Repeat the new PIN

Rules:

- PIN length is always 8 digits
- The keypad digits are randomized on every PIN screen
- If the current PIN is wrong, the device shows an error and stays in the change flow
- If the new PIN entries do not match, you must enter the new PIN again

### Security Status

This overlay shows:

- Flash encryption state
- Flash encryption mode and key size when available
- Password memory encryption state
- SecureBootV2 state

Tap the overlay to close it.

### About

This overlay shows:

- Application name and version
- Device MAC address
- Build date and time
- Firmware image SHA-256

Tap the overlay to close it.

## 7. USB Import / Export

Open `Settings -> USB Import / Export`.

The device first asks for the PIN again before starting the USB session.

After successful PIN verification:

- The device switches into USB mass storage mode
- The user sees a USB drive containing `password.csv` and `wifi.txt`
- The screen shows an import summary and action buttons

Session buttons:

- `Import keys`
- `Import WiFi`
- `Cancel`

### How the USB session works

- The exported `password.csv` contains currently populated slots
- Host edits are staged only
- Slot changes are not committed until you press `Import keys`
- `Cancel` closes the USB session and discards staged slot changes
- Closing the session returns the device to HID mode

### password.csv format

Each line uses this format:

```text
slot,label,password,colour_id
```

Example:

```text
001,EMAIL_MAIN,MySecret123,1
014,BANK_APP,s7f!2Xq9,3
128,SERVER_ROOT,RootPass!,0
```

Rules:

- `slot` must be `001` to `256`
- `label` maximum length is 15 characters
- `password` maximum length is 128 characters
- `colour_id` must be:
  - `0` = no colour / white
  - `1` = blue
  - `2` = green
  - `3` = red
  - `4` = violet
  - `5` = yellow
  - `6` = orange
- Labels and passwords must not contain commas or line breaks

To clear a slot through CSV, leave the password field empty:

```text
005,,,0
```

Import behavior:

- Valid lines are staged
- Invalid lines are ignored
- The summary on screen shows:
  - slots to update
  - slots to clear
  - invalid lines ignored

### wifi.txt format

`wifi.txt` uses exactly two lines:

```text
YOUR_WIFI_SSID
YOUR_WIFI_PASSWORD
```

Rules:

- Line 1 = WiFi SSID
- Line 2 = WiFi password

Use `Import WiFi` to save the file contents into device settings.

## 8. Firmware Update

Open `Settings -> Firmware update`.

The device shows a warning first. Press `OK` to start the update or `Cancel` to leave.

Update flow:

1. Connect to stored WiFi credentials
2. Synchronize time over NTP
3. Check the update manifest
4. Download and verify the OTA image
5. Flash the new firmware
6. Reboot on success

Notes:

- The update screen shows progress log lines
- If no update is available, the device reports that it is already up to date
- A successful OTA update reboots automatically
- If the update fails or is canceled before reboot, the device returns to normal operation

## 9. Factory Reset

Open `Settings -> Factory Reset`.

The device first shows a confirmation warning.

If you press `Proceed`:

- A 30-second countdown starts
- Press `Cancel` before the countdown ends to abort
- If the countdown reaches zero, the device erases password memory and reboots

Factory reset removes:

- stored PIN
- slot labels
- stored passwords
- colours
- saved settings
- stored WiFi credentials

## 10. Practical Notes

- For production use, flash encryption and secure boot should be enabled
- Current on-device label entry is more restrictive than CSV import; on-device editing accepts only `A-Z` and `_`

## 11. Quick Reference

### Unlock

- Enter 8-digit PIN on randomized keypad

### Open slot menu

- Long-press a slot row

### Generate password

- Long-press slot -> `Create new`

### Type password

- Connect USB to host -> long-press slot -> `Type`

### Import CSV

1. Open `Settings -> USB Import / Export`
2. Enter PIN
3. Edit `password.csv`
4. Safely finish host writes
5. Press `Import keys`

### Import WiFi

1. Open `Settings -> USB Import / Export`
2. Enter PIN
3. Edit `wifi.txt`
4. Press `Import WiFi`

### Change PIN

- `Settings -> Change PIN`

### Factory reset

- `Settings -> Factory Reset`
