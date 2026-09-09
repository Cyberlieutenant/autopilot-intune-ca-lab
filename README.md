# 🚀 Zero-Touch Windows Provisioning Lab
### Windows Autopilot · Microsoft Intune · Conditional Access

> One-line takeaway: Built and troubleshot a full zero-touch Windows deployment pipeline from scratch, including real-world licensing and enrollment issues most tutorials skip.

![Autopilot](https://img.shields.io/badge/Windows%20Autopilot-0078D4?style=for-the-badge&logo=windows&logoColor=white)
![Intune](https://img.shields.io/badge/Microsoft%20Intune-0078D4?style=for-the-badge&logo=microsoft&logoColor=white)
![Entra ID](https://img.shields.io/badge/Entra%20ID-0078D4?style=for-the-badge&logo=microsoftazure&logoColor=white)
![PowerShell](https://img.shields.io/badge/PowerShell-5391FE?style=for-the-badge&logo=powershell&logoColor=white)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen?style=for-the-badge)

## 🎯 Goal
Building an end-to-end enterprise device provisioning and access control lab —
zero-touch deployment through Windows Autopilot, device compliance and app
deployment via Intune, and access enforcement through Conditional Access —
in a Microsoft 365 tenant.

## 🧱 Environment
| Component | Details |
|---|---|
| Tenant | Microsoft 365 E3 (Trial) |
| Host | Windows 11 Pro |
| Virtualization | Hyper-V, Generation 2 VM with vTPM enabled |
| Target OS | Windows 11 Enterprise (evaluation) |

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

## 📦 What This Lab Covers

### 1. VM & Environment Setup
*Status: ✅ Complete*
- Hyper-V Generation 2 VM with vTPM enabled
- Windows 11 Enterprise (evaluation) installed

### 2. Hardware Hash & Autopilot Registration
*Status: ✅ Complete*
- Captured hardware hash via Get-WindowsAutoPilotInfo
- Imported into Intune, registered with Windows Autopilot

### 3. Deployment Profile & Assignment
*Status: ✅ Complete*
- User-driven, Entra-joined deployment profile
- Assigned via Entra ID security group

### 4. Zero-Touch OOBE & Intune Enrollment
*Status: ✅ Complete*
- Full OOBE flow completed with work account sign-in
- Enrollment Status Page completed successfully
- Device confirmed enrolled and managed in Intune

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
| Device not appearing in Intune after OOBE | Entra ID MDM user scope was set to "None" during original enrollment | Changed scope to "All"; scope changes don't retroactively apply, so re-ran full Sysprep/OOBE cycle with correct scope already in place |

> 💡 **Biggest lesson:** Configuration changes in Entra ID/Intune don't always apply retroactively to already-provisioned devices — a fresh OOBE cycle is often the more reliable fix than patching after the fact.

## 📋 Documentation Approach
Each stage of this lab documents:
- What I configured and why
- What errors I hit, and how I diagnosed them — not just the fix
- What I'd do differently next time

## 📅 Progress Log

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

### September 8, 2026 — Zero-Touch Provisioning Success
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
- Device completed Autopilot OOBE successfully but did not appear in
  Intune > Devices > All devices
- Confirmed device WAS Entra ID joined but Intune enrollment specifically hadn't triggered
- Root cause: Entra ID's MDM user scope was set to "None" during the original enrollment,
  meaning Entra-joined devices were never being handed off to Intune for management
- Changed MDM user scope to "All"; confirmed via `dsregcmd /status` and Event Viewer
  diagnostics that enrollment still hadn't triggered on the already-joined device

### September 9, 2026 — Full Enrollment Verified (Project Complete)
- Confirmed the EnterpriseMgmt scheduled task did not exist on the device — meaning
  MDM enrollment had never actually been attempted
- Key learning: changing MDM user scope after a device is already Entra-joined does not
  reliably trigger retroactive enrollment — a fresh OOBE cycle with the correct scope
  already in place is the reliable fix
- Re-ran Sysprep — completed cleanly with no BitLocker or reserved storage errors
- Went through OOBE again: work account sign-in, Enrollment Status Page completed
- Verified EnterpriseMgmt scheduled task now exists on the device
- Confirmed device now shows in Intune > Devices > All devices as enrolled
- Full pipeline verified end-to-end: hardware hash → Autopilot registration →
  deployment profile → Entra ID join → MDM auto-enrollment → Intune management,
  fully automated with zero manual configuration on the device itself

## 🚧 Status
✅ **Complete** — zero-touch provisioning fully verified end-to-end, device confirmed enrolled and managed in Intune.
