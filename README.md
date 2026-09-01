# Zero-Touch Windows Provisioning Lab
Autopilot + Intune + Conditional Access

## Goal
Building an end-to-end enterprise device provisioning and access control lab —
zero-touch deployment through Windows Autopilot, device compliance and app
deployment via Intune, and access enforcement through Conditional Access —
in a Microsoft 365 Developer tenant.

## Environment
- Microsoft 365 Developer Tenant (Entra ID P2, Intune)
- Windows 11 Pro host
- Hyper-V, Generation 2 VM with vTPM enabled

## Progress Log

### August 30, 2026 — Environment Setup
- Confirmed Windows 11 Pro edition, enabled Hyper-V via Windows Features
- Created GitHub repo to document the build as I go
- Next: create Generation 2 VM with virtual TPM for Autopilot testing
- ### August 30, 2026 — VM Setup
- Enabled Hyper-V on Windows 11 Pro host
- Created external virtual switch for VM networking
- Downloading Windows 11 Enterprise (eval) ISO for the Autopilot lab VM
- ### August 30, 2026 — VM Creation & Troubleshooting
- Hit Event ID 15266 "Failed to create the virtual hard disk" when creating the VM using
  the default ProgramData storage path
- Diagnosed the issue using Event Viewer (Microsoft-Windows-Hyper-V-VMMS/Admin log) to
  confirm the exact error rather than guessing
- Ruled out disk space (237GB free) and Windows Defender Controlled Folder Access as causes
- Resolved by creating the VM in a custom folder (C:\HyperV-VMs) instead of the default
  system location — pointed to a likely permissions restriction on the default path
- VM created successfully with Generation 2, vTPM enabled, connected to AutopilotSwitch
- Windows 11 Enterprise (eval) installation in progress inside the VM
- ### August 30, 2026 — Hardware Hash Capture
- Installed Get-WindowsAutoPilotInfo script via PowerShell
- Hit a PSSecurityException (UnauthorizedAccess) — script execution disabled by default policy
- Resolved with Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
- Successfully generated AutopilotHash.csv containing device serial number and hardware hash
- ### September 1, 2026 — Hash File Transfer
- Discovered two duplicate "AutopilotLab" VMs existed (one broken/empty from an earlier
  failed attempt, one with the actual Windows install)
- Verified which was which using Hyper-V's disk Inspect tool (checked used disk space:
  36MB vs 32.35GB)
- Deleted the broken VM to avoid confusion going forward
- Successfully copied AutopilotHash.csv from the VM to host machine using Hyper-V
  Enhanced Session Mode clipboard sharing

## Status
🚧 In progress
