# 🖥 VMware Automation with PowerShell & PowerCLI

This guide provides PowerShell scripts to automate common VMware vCenter and ESXi operations using **VMware PowerCLI**.

---

## 📌 1. Automate Snapshot Report

### **Description**
Generate a snapshot report in VMware vCenter and export it as a CSV file.

### **Prerequisites**
- Install VMware PowerCLI module:
  ```powershell
  Install-Module -Name VMware.PowerCLI -Scope CurrentUser
  ```
- Connect to your vCenter Server:
  ```powershell
  Connect-VIServer -Server <vcenter-server-name>
  ```

### **Script: Snapshot Report**
```powershell
# Import the PowerCLI module
Import-Module VMware.VimAutomation.Core

# Connect to vCenter Server
$vCenterServer = "<vcenter-server-name>"
Connect-VIServer -Server $vCenterServer

# Output file for the report
$outputFile = "SnapshotReport.csv"

# Retrieve all snapshots in vCenter
$snapshots = Get-VM | Get-Snapshot

# Create a report object
$report = @()

foreach ($snapshot in $snapshots) {
    $report += [PSCustomObject]@{
        VMName        = $snapshot.VM.Name
        SnapshotName  = $snapshot.Name
        CreatedBy     = $snapshot.CreatedBy
        CreatedOn     = $snapshot.Created
        SizeGB        = "{0:N2}" -f ($snapshot.SizeGB)
        Description   = $snapshot.Description
    }
}

# Export the report to CSV
$report | Export-Csv -Path $outputFile -NoTypeInformation -UseCulture

# Disconnect from vCenter
Disconnect-VIServer -Server $vCenterServer -Confirm:$false

Write-Host "Snapshot report generated: $outputFile"
```

### **Explanation**
- `Connect-VIServer` → Connects to vCenter.
- `Get-VM | Get-Snapshot` → Retrieves snapshots for all VMs.
- `Export-Csv` → Saves the snapshot details to `SnapshotReport.csv`.

---

## 📌 2. Automate Creating Snapshots

### **Description**
Automatically create snapshots for specific or all VMs in a vCenter environment.

### **Prerequisites**
- Install VMware PowerCLI:
  ```powershell
  Install-Module -Name VMware.PowerCLI -Scope CurrentUser
  ```
- Connect to vCenter:
  ```powershell
  Connect-VIServer -Server <vcenter-server-name>
  ```

### **Script: Create Snapshots**
```powershell
# Import the PowerCLI module
Import-Module VMware.VimAutomation.Core

# Connect to vCenter Server
$vCenterServer = "<vcenter-server-name>"
Connect-VIServer -Server $vCenterServer

# Parameters
$snapshotName = "AutomatedSnapshot_$(Get-Date -Format 'yyyyMMdd_HHmmss')"
$snapshotDescription = "Automated snapshot created on $(Get-Date)"
$quiesce = $false # Set to $true to quiesce the guest file system

# VM selection
# Option 1: All VMs
# $vms = Get-VM

# Option 2: Specific VMs
$vms = Get-VM -Name "VM1", "VM2"

# Create snapshots
foreach ($vm in $vms) {
    try {
        Write-Host "Creating snapshot for VM: $($vm.Name)" -ForegroundColor Green
        New-Snapshot -VM $vm -Name $snapshotName -Description $snapshotDescription -Quiesce $quiesce
        Write-Host "Snapshot created for VM: $($vm.Name)" -ForegroundColor Cyan
    } catch {
        Write-Host "Failed to create snapshot for VM: $($vm.Name). Error: $_" -ForegroundColor Red
    }
}

# Disconnect from vCenter
Disconnect-VIServer -Server $vCenterServer -Confirm:$false

Write-Host "Snapshot process completed." -ForegroundColor Green
```

### **Notes**
- **Snapshot naming** uses date/time for uniqueness.
- **Quiesce** requires VMware Tools inside the guest.
- Avoid excessive snapshots to prevent performance impact.

---

## 📌 3. Automate ESXi Maintenance Mode

### **Description**
Automatically put an ESXi host into maintenance mode.

### **Prerequisites**
- Install VMware PowerCLI:
  ```powershell
  Install-Module -Name VMware.PowerCLI -Scope CurrentUser
  ```
- Connect to vCenter:
  ```powershell
  Connect-VIServer -Server <vcenter-server-name>
  ```

### **Script: Maintenance Mode**
```powershell
# Import the PowerCLI module
Import-Module VMware.VimAutomation.Core

# Connect to vCenter Server
$vCenterServer = "<vcenter-server-name>"
Connect-VIServer -Server $vCenterServer

# Specify the ESXi host name
$esxiHostName = "<esxi-host-name>"

# Retrieve the ESXi host object
$esxiHost = Get-VMHost -Name $esxiHostName

# Check if the host is already in maintenance mode
if ($esxiHost.ConnectionState -eq "Maintenance") {
    Write-Host "Host $esxiHostName is already in Maintenance Mode." -ForegroundColor Yellow
} else {
    try {
        # Enter Maintenance Mode
        Write-Host "Putting ESXi host $esxiHostName into Maintenance Mode..." -ForegroundColor Green
        Set-VMHost -VMHost $esxiHost -State Maintenance -Confirm:$false
        Write-Host "Host $esxiHostName is now in Maintenance Mode." -ForegroundColor Cyan
    } catch {
        Write-Host "Failed to put host $esxiHostName into Maintenance Mode. Error: $_" -ForegroundColor Red
    }
}

# Disconnect from vCenter
Disconnect-VIServer -Server $vCenterServer -Confirm:$false

Write-Host "Script completed." -ForegroundColor Green
```

### **Optional Enhancements**
- **Multiple Hosts in Cluster**
```powershell
$clusterName = "<cluster-name>"
$hosts = Get-Cluster -Name $clusterName | Get-VMHost

foreach ($host in $hosts) {
    if ($host.ConnectionState -ne "Maintenance") {
        Set-VMHost -VMHost $host -State Maintenance -Confirm:$false
        Write-Host "Host $($host.Name) is now in Maintenance Mode."
    } else {
        Write-Host "Host $($host.Name) is already in Maintenance Mode."
    }
}
```

- **Exit Maintenance Mode**
```powershell
Set-VMHost -VMHost $esxiHost -State Connected -Confirm:$false
```

- **Scheduling**
Use Windows Task Scheduler or cron to run scripts at specific times.

---
