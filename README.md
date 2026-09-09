# Zero-Touch Windows Provisioning Lab
Autopilot + Intune + Conditional Access

## Goal
Building an end-to-end enterprise device provisioning and access control lab —
zero-touch deployment through Windows Autopilot, device compliance and app
deployment via Intune, and access enforcement through Conditional Access —
in a Microsoft 365 tenant.

## Environment
- Microsoft 365 E3 (Trial) tenant
- Windows 11 Pro host
- Hyper-V, Generation 2 VM with vTPM enabled
- Windows 11 Enterprise (evaluation) as the target OS

## 🔑 Skills Demonstrated
- Windows Autopilot deployment and hardware hash registration
- Hyper-V virtualization (Generation 2, vTPM configuration)
- Microsoft Intune device enrollment and management
- Microsoft 365 / Entra ID tenant administration and licensing
- Entra ID security group-based policy/profile assignment
- Entra ID Mobility (MDM/MAM) scope configuration
- PowerShell scripting and execution policy management
- BitLocker/disk encryption management via command line (manage-bde)
- Windows servicing and update management (DISM, reserved storage)
- Systematic troubleshooting using Event Viewer, Sysprep logs, dsregcmd, and diagnostic tools
- Enterprise identity and licensing architecture (M365 admin center, Entra ID, Intune relationships)

## 🛠️ Notable Troubleshooting

| Issue | Root Cause | Resolution |
|---|---|---|
| VM creation failed (Event ID 15266) | Permissions restriction on default ProgramData path | Created VM in a custom folder instead |
| PowerShell script blocked | Script execution disabled by default policy | `Set-ExecutionPolicy RemoteSigned -Scope CurrentUser` |
| Duplicate VM confusion | Earlier failed attempt left an empty leftover VM | Verified via disk Inspect (used space), deleted the empty one |
| Autopilot import failed (BadRequest 400) | Admin account had no Intune license assigned | Diagnosed via Entra/Billing, assigned license directly |
| proxyAddresses conflict on license save | UI bug in newer M365 admin center | Assigned license via Billing > Licenses instead of Users panel |
| Deployment profile wouldn't assign to device | Profiles assign to Entra ID groups, not individual devices | Created a security group, added the device, assigned profile to the group |
| Sysprep failed (BitLocker) | GUI showed BitLocker off, but drive was still "Used Space Only Encrypted" | Verified true state via `manage-bde -status`, fully decrypted via `manage-bde -off C:` |
| Sysprep failed again (reserved storage) | Pending Windows Updates were using reserved storage | Installed pending updates, ran `DISM /Online /Cleanup-Image /StartComponentCleanup` |
| Device not appearing in Intune after OOBE | Entra ID MDM user scope was set to "None" | Changed scope to "All", forced fresh sign-in to refresh enrollment token |

## 📋 Documentation Approach
Each stage of this lab documents:
- What I configured and why
- What errors I hit, and how I diagnosed them — not just the fix
- What I'd do differently next time

## Progress Log

### August 30, 2026 — Environment Setup
- Confirmed Windows 11 Pro edition, enabled Hyper-V via Windows Features
- Created GitHub repo to document the build as I go
- Next: create Generation 2 VM with virtual TPM for Autopilot testing

### August 30, 2026 — VM Setup
- Enabled Hyper-V on Windows 11 Pro host
- Created external virtual switch for VM networking
- Downloading Windows 11 Enterprise (eval) ISO for the Autopilot lab VM

### August 30, 2026 — VM Creation & Troubleshooting
- Hit Event ID 15266 "Failed to create the virtual hard disk" when creating the VM using
  the default ProgramData storage path
- Diagnosed the issue using Event Viewer (Microsoft-Windows-Hyper-V-VMMS/Admin log) to
  confirm the exact error rather than guessing
- Ruled out disk space (237GB free) and Windows Defender Controlled Folder Access as causes
- Resolved by creating the VM in a custom folder (C:\HyperV-VMs) instead of the default
  system location — pointed to a likely permissions restriction on the default path
- VM created successfully with Generation 2, vTPM enabled, connected to AutopilotSwitch
- Windows 11 Enterprise (eval) installation in progress inside the VM

### August 30, 2026 — Hardware Hash Capture
- Installed Get-WindowsAutoPilotInfo script via PowerShell
- Hit a PSSecurityException (UnauthorizedAccess) — script execution disabled by default policy
- Resolved with `Set-ExecutionPolicy RemoteSigned -Scope CurrentUser`
- Successfully generated AutopilotHash.csv containing device serial number and hardware hash

### September 1, 2026 — Hash File Transfer
- Discovered two duplicate "AutopilotLab" VMs existed (one broken/empty from an earlier
  failed attempt, one with the actual Windows install)
- Verified which was which using Hyper-V's disk Inspect tool (checked used disk space:
  36MB vs 32.35GB)
- Deleted the broken VM to avoid confusion going forward
- Successfully copied AutopilotHash.csv from the VM to host machine using Hyper-V
  Enhanced Session Mode clipboard sharing

### September 1, 2026 — Tenant Access Setup
- Attempted Microsoft 365 Developer Program (free sandbox, no card required) — repeatedly
  denied eligibility despite multiple account attempts
- Explored Microsoft 365 E3 trial and Azure free tier as alternatives — both require
  card verification for identity purposes (standard practice, no charge during trial)
- Decision: pause to weigh card-verification trial vs. retrying Developer Program later,
  before provisioning the tenant that will host Intune/Entra ID for this lab

### September 7, 2026 — Tenant Licensing Troubleshooting
- Hit "Request not applicable to target tenant" (BadRequest, 400) when importing Autopilot
  hash — traced to missing Intune license on admin account
- Discovered M365 E3 trial from public signup page never fully attached to tenant;
  re-initiated trial directly from within the admin center (Billing > Marketplace) instead
- Hit a second error: "proxyAddresses already exists" when assigning license via Users panel
  — worked around by assigning the license through Billing > Licenses instead
- License confirmed active and assigned; waiting for propagation before retrying
  Autopilot import

### September 7, 2026 — Autopilot Device Import Success
- Successfully imported AutopilotHash.csv into Intune (Devices > Windows > Windows
  enrollment > Devices)
- Device now shows in the Windows Autopilot devices list with serial number confirmed
- Next: create a deployment profile and assign it to the device, then reset the VM to
  OOBE to test the full Autopilot provisioning experience end-to-end

### September 7, 2026 — Deployment Profile Assignment & OOBE Testing
- Created a Windows Autopilot deployment profile (user-driven, Entra joined, OOBE
  privacy/EULA screens hidden)
- Discovered deployment profiles are assigned via Entra ID security groups, not
  directly to individual devices from the device list
- Created an Entra ID security group, added the registered device, and assigned the
  deployment profile to that group via the profile's Properties > Assignments section
- Attempted to reset the VM to OOBE using Sysprep to test the full Autopilot experience
- Hit "Sysprep was unable to validate your Windows installation" — investigating via
  setupact.log to identify the root cause before retrying

### September 8, 2026 — BitLocker & Reserved Storage Troubleshooting
- Retried Sysprep after clearing BitLocker via the GUI — Control Panel showed "Turn on
  BitLocker" (implying it was off), but Sysprep still failed with the same validation error
- Verified true disk state via command line (`manage-bde -status C:`) — GUI was misleading;
  the drive was still "Used Space Only Encrypted" despite protection showing off
- Fully decrypted the drive via `manage-bde -off C:`, confirmed "Fully Decrypted" status
- Retried Sysprep — hit a new error: "Audit mode cannot be turned on if reserved storage
  is in use"
- Installed pending Windows Updates and ran `DISM /Online /Cleanup-Image
  /StartComponentCleanup` to clear reserved storage conflicts
- Sysprep now proceeding past the cleanup phase — awaiting reboot to test the full
  Autopilot OOBE experience

### September 8, 2026 — Zero-Touch Provisioning Success (Core Objective Complete)
- Sysprep completed successfully after resolving BitLocker and reserved storage blockers
- VM rebooted into a genuine OOBE state, presenting a work/school account sign-in
  instead of local account setup — confirming Autopilot recognized the registered
  hardware hash and pulled the assigned deployment profile
- Signed in with tenant credentials; Enrollment Status Page ran successfully,
  applying Intune configuration automatically
- End-to-end zero-touch provisioning flow confirmed working: hardware hash →
  Autopilot registration → deployment profile → Intune enrollment, entirely
  automated with no manual domain join or local account setup

### September 8, 2026 — MDM Enrollment Verification
- Device completed Autopilot OOBE successfully (work account sign-in, Enrollment Status
  Page ran through to completion) but did not appear in Intune > Devices > All devices
- Confirmed device WAS Entra ID joined (visible in Entra ID > Devices) but Intune
  enrollment specifically hadn't triggered
- Root cause: Entra ID's "MDM user scope" (Mobility (MDM and MAM) settings) was set to
  "None" — meaning Entra-joined devices were never being handed off to Intune for
  management
- Changed MDM user scope to "All"
- Verified via `dsregcmd /status`: MDM Url was initially blank even after manual
  enrollment attempts (deviceenroller.exe /c /AutoEnrollMDM)
- Forced a full sign-out/sign-in to refresh the authentication token — MDM Url then
  populated, confirming the device found its enrollment endpoint
- Device still not yet visible in Intune's device list — likely a backend propagation
  delay following the scope change; core Autopilot/OOBE flow already proven successful,
  Intune dashboard sync to be re-verified next session

## Status
✅ Core Autopilot provisioning flow verified end-to-end. 🔄 Confirming Intune enrollment sync.
