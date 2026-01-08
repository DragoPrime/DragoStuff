Windows Server – Auto-restart application using Task Scheduler + PowerShell
Scenario

We have a Windows Server (2012 R2) VM running an application (asm.exe) that automatically closes after ~24 hours.
The goal is to periodically check if the application is running and start it if it is not, without restarting the server.

Key Issue (Root Cause)

The application depends on its Working Directory.

When started manually (double-click or shortcut) → works

When started by Task Scheduler → opens and closes immediately

Reason:
Task Scheduler launches processes from C:\Windows\System32 by default.
Because the application expects to run from its own folder, it exits immediately.

Solution Overview

Use a PowerShell script to:

Check if the process is running

Start the application if it is not

Force the Working Directory inside the script

Run the script using Task Scheduler under a logged-in user session

Folder Layout
C:\Heroes\asm.exe
C:\Scripts\check_asm.ps1
C:\Scripts\asm_monitor.log

PowerShell Script

File: C:\Scripts\check_asm.ps1

Set-Location "C:\Heroes"

$processName = "asm"
$exePath = "C:\Heroes\asm.exe"
$logFile = "C:\Scripts\asm_monitor.log"

$timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"

$process = Get-Process -Name $processName -ErrorAction SilentlyContinue

if ($process -eq $null) {
    Add-Content $logFile "$timestamp - asm.exe not running, starting it"
    Start-Process -FilePath $exePath -WorkingDirectory "C:\Heroes"
} else {
    Add-Content $logFile "$timestamp - asm.exe already running"
}

Important

Set-Location is critical

-WorkingDirectory is mandatory

Without these, the application will start and immediately close

Task Scheduler Configuration
General

Run only when user is logged on

Run with highest privileges

Run as the same user that normally starts the app (e.g. drago)

Trigger

Daily

Repeat task every: 1 hour

Duration: Indefinitely

Action

Program/script

powershell.exe


Add arguments

-NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\check_asm.ps1"


Start in

C:\Heroes


⚠️ Do NOT leave “Start in” empty.

Operational Notes (Very Important)

Do NOT use Sign out

Use Disconnect / Close RDP instead
(signing out will close the application)

After a server reboot:

User must log in once

After that, monitoring works automatically

Logging

Log file:

C:\Scripts\asm_monitor.log


Example entries:

2026-01-08 10:00:00 - asm.exe not running, starting it
2026-01-08 11:00:00 - asm.exe already running
