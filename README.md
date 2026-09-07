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
- PowerShell scripting and execution policy management
- Systematic troubleshooting using Event Viewer, Sysprep logs, and diagnostic tools
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
| Sysprep validation failure | Under investigation via setupact.log | In progress |

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

## Status
🚧 In progress
