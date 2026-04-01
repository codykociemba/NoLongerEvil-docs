# Firmware Installer Screenshots

Reference guide for LLM agents and documentation writers. Each entry states what the code confirms and what is visible in the screenshot. Nothing is assumed or inferred beyond those two sources.

**Source component directory:** `NoLongerEvil-Thermostat/firmware/installer/src/components/`

---

## Flow Overview

```
System Check → Generation Select → [Custom Firmware toggle] → Flashing → Flash Complete
    → Hosting Mode → SSH Access → [Server Config — self-hosted only] → Finding Your Nest
    → Configuring Device → Setup Complete
```

---

## Screens

---

### `1-systemcheck.png`
**Component:** `SystemCheck.jsx`

**What the screen shows (verified from screenshot):**
- Title: "System Check"
- Subtitle: "Verifying your system meets all requirements"
- Four rows, each showing a label + status badge
- "Back" button (left) and "Continue to Installation" button (right)

**What the code confirms about each row:**

| Row label (from code) | Subtitle (from code) | When shown |
|---|---|---|
| **Operating System** | `getPlatformName()` — e.g. "Windows (x64)" | Always |
| **libusb Library** | "Required for USB communication" | Only when `systemInfo.needsLibusb` is true |
| **USB Driver (WinUSB)** | "Required for DFU device access" | Only when `platform === 'win32'` |
| **Administrator Access** | "Required for USB device access" | Always |
| **Firmware Files** | "Bootloader and kernel images" | Always |

**Status badge values the code can show:**
- Firmware Files: `Ready` (green) or `Missing` (red)
- Administrator Access on Windows: `Running as Admin` (green) or `Not Admin` (yellow)
- Administrator Access on non-Windows (macOS/Linux): `Will prompt` (blue info)
- USB Driver (WinUSB) when admin: `Installed` (green) or `Will install` (blue info)
- USB Driver (WinUSB) when not admin: `Not Admin` (yellow)
- libusb: `Installed` (green) or `Missing` (yellow) with an Install button

**What the screenshot shows specifically:** The screenshot is running on Windows — the USB Driver (WinUSB) row is visible and shows "Installed." Administrator Access shows "Running as Admin." Firmware Files shows "Ready." Operating System shows "Detected."

**The Continue to Installation button is never disabled by the code** — it always calls `onNext(systemInfo)` regardless of check results. There is a separate error banner shown if admin is required but not present, or if firmware files are missing.

---

### `2-nestgenselection.png`
**Component:** `GenerationSelect.jsx`

**What the screen shows (verified from screenshot):**
- Title: "Select Your Nest Generation"
- Two side-by-side clickable cards showing photos of the thermostats, labeled "Generation 1" and "Generation 2"
- A blue info box at the bottom
- "Back" and "Continue" buttons

**What the code confirms:**
- Cards use images imported from `../assets/nest-gen1.png` and `../assets/nest-gen2.png`
- When a card is selected it shows a blue ring, darker background, and a checkmark + "Selected" text
- The blue info box heading is: "Not sure which generation you have?" — it is a styled info banner, not just a link
- The link text inside is "compatibility guide" and it goes to `https://docs.nolongerevil.com/compatibility#how-to-identify-your-nest-thermostat`
- The Continue button is `disabled={!selectedGeneration}` — disabled until a card is clicked
- The Custom Firmware section is hidden until a generation is selected (`{selectedGeneration && (...)`)

---

### `2a-customfirmware.png`
**Component:** `GenerationSelect.jsx`

**What the screen shows (verified from screenshot):**
- Same generation selection screen with Gen 2 selected (blue ring, "Selected" checkmark visible)
- A card below with "Custom Firmware Files" heading, a toggle switch (enabled/blue), and three file picker rows
- "Back" and "Continue" buttons

**What the code confirms:**
- The Custom Firmware card only renders after a generation is selected
- Toggle is a styled checkbox (`<input type="checkbox">`)
- Section heading: "Custom Firmware Files"
- Section subtitle: "Optional: Use your own firmware files instead of the bundled ones"
- The three file labels are exactly: `x-load.bin`, `u-boot.bin`, `uImage`
- Each row has a "Browse" button that calls `window.electronAPI.selectFirmwareFile(fileType)` — opens a native file picker
- When no file is selected, the row shows "No file selected"
- Warning text when toggle is on: "Custom firmware files will be used instead of the bundled ones. Make sure your files are compatible with your device."

---

### `3-flashing.png`
**Component:** `InstallScreen.jsx`

**What the screen shows (verified from screenshot):**
- Title: "Installing Firmware"
- A spinner in the center
- Status text reading "Flashing Linux kernel..."
- A blue progress bar (partially filled)
- A checklist of three items below the bar
- A blue "Important:" banner at the bottom

**What the code confirms:**

The screen has two distinct states depending on whether the device has been detected:

**Before device detected (`deviceDetected === false`):** Shows numbered instructions (Remove from Wall, Connect via USB, Reboot the Device, Enter DFU Mode) and a pulsing dot "Waiting for device connection..." — this is NOT what the screenshot shows.

**After device detected (`deviceDetected === true`):** This is what the screenshot shows:
- Spinner (while stage ≠ COMPLETE) or green checkmark (when COMPLETE)
- Stage message text from `getStageMessage()`:
  - `'Flashing x-load bootloader...'` during xload stage
  - `'Flashing u-boot...'` during uboot stage
  - `'Flashing Linux kernel...'` during kernel stage
- Progress shown as `{progress}% complete`
- Three dot indicators: `x-load bootloader`, `u-boot`, `Linux kernel (uImage)`
  - Dots turn green when progress crosses 25%, 50%, 75% respectively
- Blue "Important" banner text: "Keep your device connected via USB. Do not disconnect or power off during installation."

**Subtitle text:** When device is detected and not complete: "Do not disconnect your device during installation"

---

### `4-flashed.png`
**Component:** `InstallScreen.jsx`

**What the screen shows (verified from screenshot):**
- Title: "Installing Firmware"
- Large green circle with white checkmark (replaces spinner)
- Status text: "Installation complete!"
- Progress bar: full / 100%
- All three checklist items lit up (green)

**What the code confirms:**
- `INSTALL_STAGES.COMPLETE` renders `<div className="w-20 h-20 bg-green-500 rounded-full ...">` with a checkmark SVG
- The Important/warning banner is NOT rendered when stage is COMPLETE (`{stage !== INSTALL_STAGES.COMPLETE && (...)}`)
- After 2000ms the component automatically calls `onSuccess()` via `setTimeout(() => onSuccess(), 2000)` — advances to Hosting Mode with no user action required
- All three checklist dots are green at 100% progress

---

### `5-hostingselection.png`
**Component:** `HostingModeStep.jsx`

**What the screen shows (verified from screenshot):**
- Title: "Setup Your Thermostat"
- Subtitle: "Choose how your thermostat will connect to the internet"
- Two radio-style option cards
- "Continue" button
- A text link below Continue

**What the code confirms — exact text for each option:**

| Option | Heading | Description text |
|---|---|---|
| `hosted` | Cloud Hosted | "Connect to the hosted cloud service. Easiest setup, your thermostat connects automatically." |
| `selfhosted` | Self-Hosted | "Connect to your own server. Requires the No Longer Evil server or Home Assistant add-on running on your local network." |

- The skip link text is exactly: "Skip setup — I'll configure this later" — calls `onSkip()`
- Continue button is `disabled={!selected}` — disabled until an option is clicked
- The internal value for self-hosted is the string `'selfhosted'` (no hyphen/space)
- **Branch logic from `App.jsx`:** After SSH step, `selfhosted` routes to `HADiscoveryStep` (Server Configuration screen); `hosted` routes directly to `NestDiscoveryStep`

---

### `6-sshoption.png`
**Component:** `SSHConfigStep.jsx`

**What the screen shows (verified from screenshot):**
- Title: "SSH Access"
- Subtitle: "Configure remote SSH access to your thermostat"
- Two radio-style option cards, first one highlighted in blue (selected)
- "Back" and "Continue" buttons

**What the code confirms:**

Default selected option on mount: `'disable'` (`useState('disable')`)

| Option value | Label | Badge | Description text |
|---|---|---|---|
| `'disable'` | Disable SSH | "Recommended" (green) | "Removes SSH access from the device. More secure — you won't need it after setup." |
| `'password'` | Enable SSH with custom password | none | "Keep SSH enabled and change the default password." |

- The "Recommended" badge is a separate `<span>` next to the label, styled green
- Continue button: `disabled={!passwordValid}` — for `'disable'` mode, `passwordValid` is always `true`

---

### `6a-sshpassword.png`
**Component:** `SSHConfigStep.jsx`

**What the screen shows (verified from screenshot):**
- Same SSH Access screen with "Enable SSH with custom password" selected (highlighted blue)
- "New SSH Password" and "Confirm Password" input fields (showing masked input)
- "Back" and "Continue" buttons

**What the code confirms:**
- Label text: "New SSH Password" and "Confirm Password"
- Both fields use `type={showPassword ? 'text' : 'password'}` — there is a show/hide toggle button (eye icon) on the password field
- A password strength bar (4 segments) and label appear below the password field once typing begins: `['Very weak', 'Weak', 'Fair', 'Strong', 'Very strong']`
- Confirm field border turns red if `confirm.length > 0 && !passwordsMatch`
- "Passwords do not match" error text appears in red under Confirm if they don't match
- Continue is enabled only when: `password.length >= 6 && passwordsMatch && strength >= 2` (strength ≥ "Fair")
- The password strength meter is not visible in the screenshot (fields appear empty or newly typed)

---

### `6b-selfhostingconfig.png`
**Component:** `HADiscoveryStep.jsx`

**What the screen shows (verified from screenshot):**
- Title: "Server Configuration"
- Subtitle: "Where should your thermostat send its data?"
- Two tabs: "Home Assistant" (active, underlined) and "NLE Server"
- A text field labeled "Home Assistant IP Address" — field is empty
- "Auto-detect" link (top-right of the field)
- "Back" and "Continue" buttons

**What the code confirms:**
- Tab values: `'ha'` (Home Assistant) and `'server'` (NLE Server)
- Auto-discovery (`runDiscovery()`) runs automatically on mount via `useEffect`
- The "Auto-detect" button only renders when `haStatus !== 'discovering'` — its presence in the screenshot means discovery has already finished (or been cancelled)
- IP field placeholder: `"192.168.1.100"`
- When an IP is typed, a preview line appears below the field showing the full URL: `http://{haIp}:9543/entry` — the port is hardcoded as `9543` via `const NLE_PORT = '9543'`
- NLE Server tab has separate "Server IP Address" and "Port" fields; the Port field defaults to `9543`

**Discovery status banners (from code, one shown at a time):**
- `discovering`: spinning indicator + message text (no banner box)
- `found`: green banner — "Add-on found at [IP]"
- `addon_missing`: yellow banner — "Home Assistant at [IP] — NLE add-on not running"
- `not_found`: red banner — error message or "Home Assistant not found on your network"

---

### `6c-haautodetect.png`
**Component:** `HADiscoveryStep.jsx`

**What the screen shows (verified from screenshot):**
- Same Server Configuration screen
- A green banner is visible above the IP field
- The IP field is populated (shows an IP address, e.g. 192.168.1.100)
- "Auto-detect" link still visible

**What the code confirms:**
- The green banner renders when `haStatus === 'found'` — it contains a checkmark SVG and the text `haMessage`, which is set to `\`Add-on found at ${result.haIp}\``
- `setHaIp(result.haIp)` is called on success — this populates the IP input field
- The URL constructed on Continue: `http://${haIp.trim()}:9543/entry`

---

### `7-networkscan.png`
**Component:** `NestDiscoveryStep.jsx`

**What the screen shows (verified from screenshot):**
- Title: "Finding Your Nest"
- Subtitle: "Plug your Nest into its wall mount and power it on"
- An instruction box with a "!" badge
- A green "Nest found" result card showing an IP address
- A "Use this device" button
- A manual IP entry section at the bottom
- "Back" button

**What the code confirms:**
- Instruction text (verbatim from code): "Remove the Nest from USB and attach it to your wall mount. It will take about 30–60 seconds to boot and connect to your WiFi network."
- Discovery runs automatically on mount via `useEffect`
- When one device found: green banner with "Nest found" (no exclamation mark in code) and the IP in monospace, then "Use this device" button (`btn-primary w-full`)
- When multiple devices found: lists all IPs as selectable buttons
- When not found: yellow warning banner + "Scan again" button
- Manual entry section is always visible (not conditional), with placeholder `"192.168.1.251"` and a "Use" button
- The "Use" button is `disabled={!manualIp.trim()}`
- There is only a "Back" button at the bottom — no Continue button in this component

---

### `8-provisioningcomplete.png`
**Component:** `ConfiguringStep.jsx`

**What the screen shows (verified from screenshot):**
- Title: "Configuring Device"
- Subtitle: "All done!" (only shown when `done === true`)
- A list of completed steps with green checkmarks
- A green "Configuration complete!" banner at the bottom of the list

**What the code confirms:**

The step rows shown are determined by `getVisibleSteps(cloudregisterurl, sshMode)`:

| Step key | Default label (from STEPS object) | Shown when |
|---|---|---|
| `ssh_connect` | "Connecting via SSH" | Always |
| `serial` | "Reading device serial" | Always |
| `url` | "Updating server URL" | Only when `cloudregisterurl` is set (self-hosted path) |
| `ssh_config` | "Configuring SSH access" | Only when `sshMode !== 'keep'` |

- Each row's displayed text is `stepMessages[stepKey] || STEPS[stepKey]` — the backend can override labels by sending `progress.message`
- Row status values: `'pending'`, `'active'` (spinner), `'done'` (green checkmark), `'skipped'` (grey dot), `'error'` (red X)
- The green "Configuration complete!" banner is a separate element rendered when `done === true`, not a step row
- Subtitle changes from `"Setting up {nestIp}..."` to `"All done!"` when done
- On error: shows a red error banner + Back and Retry buttons + link to `https://docs.nolongerevil.com/hosted/troubleshooting`
- On success: auto-advances via `setTimeout(() => onSuccess({serial, cloudregisterurl, sshMode}), 1500)`

---

### `9-setupcomplete.png`
**Component:** `SetupCompleteStep.jsx`

**What the screen shows (verified from screenshot):**
- Large green circle with checkmark at top
- Title: "Setup Complete!"
- Subtitle: "Your Nest Thermostat is configured and ready"
- "Device Details" section with a serial number field and an SSH status line
- "Next Steps" section with two numbered items
- Footer text at the bottom

**What the code confirms:**

**Device Details section:**
- Serial number renders as a `CopyField` component (label "SERIAL NUMBER", monospace value, copy-to-clipboard button) — only shown when `result?.serial` is truthy
- A "Server URL" `CopyField` also renders when `result?.cloudregisterurl` is truthy (self-hosted path)
- SSH status line shows one of three strings based on `result?.sshMode`:
  - `'disable'` → "SSH disabled" (green dot)
  - `'password'` → "SSH enabled (password changed)" (yellow dot)
  - anything else → "SSH unchanged (default password)" (yellow dot)

**Next Steps — Step 1 (always shown):**
- Heading: "Re-attach to wall plate"
- Text (verbatim from code): "The thermostat is booting. Re-attach after 3–5 minutes."

**Next Steps — Step 2 (conditional on `hostingMode`):**
- When `hostingMode === 'hosted'`:
  - Heading: "Create your account"
  - Text: "Visit the No Longer Evil dashboard to register and link your device."
  - Button text: "Open Dashboard" — calls `window.electronAPI.openExternal('https://nolongerevil.com/dashboard')`
- When `hostingMode === 'selfhosted'`:
  - Heading: "Check Home Assistant"
  - Text: "Your thermostat will appear in Home Assistant once it connects (within 1–2 minutes)."
  - No button

**Footer (verbatim from code):** "Made with ❤️ by Hack House" — "Hack House" is a clickable link to `https://hackhouse.io`
