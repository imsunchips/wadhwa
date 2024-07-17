---
layout: post
title: Automating PostgreSQL ODBC Driver Installation and Configuration on Windows with PowerShell
subtitle: 
cover-img: /assets/imgblog/markus-spiske-unsplash.jpg
tags: [script, powershell, postgres, odbc, dowload, configure, install, automation]
author: Sanchit Wadhwa
---

Automating routine tasks is a cornerstone of efficient IT management. This guide details a PowerShell script designed to streamline the installation and configuration of the PostgreSQL ODBC driver on Windows machines. By automating these steps, administrators can save time and reduce the risk of manual errors.

## Script Overview

The provided PowerShell script accomplishes the following tasks:

1. **Elevates to Administrative Privileges:** Checks and requests administrative privileges if not already elevated.
2. **Gathers User Input:** Prompts the user for PostgreSQL server details and credentials.
3. **Downloads the ODBC Driver:** Downloads a specified version of the PostgreSQL ODBC driver.
4. **Installs the Driver:** Installs the ODBC driver if it is not already installed.
5. **Configures the ODBC Data Source Name (DSN):** Configures the ODBC DSN with the provided PostgreSQL server details and credentials.

## Script Breakdown

### 1. Elevating to Administrative Privileges

The script begins by ensuring it runs with administrative privileges, which are necessary for installing and configuring ODBC drivers.

```powershell
param([switch]$Elevated)

function Test-Admin {
    $currentUser = New-Object Security.Principal.WindowsPrincipal $([Security.Principal.WindowsIdentity]::GetCurrent())
    $currentUser.IsInRole([Security.Principal.WindowsBuiltinRole]::Administrator)
}

if ((Test-Admin) -eq $false)  {
    if ($elevated) {
        # tried to elevate, did not work, aborting
    } else {
        Start-Process powershell.exe -Verb RunAs -ArgumentList ('-noprofile -noexit -file "{0}" -elevated' -f ($myinvocation.MyCommand.Definition))
    }
    exit
}
```

### 2. Gathering User Input

The script then prompts the user for necessary details such as the PostgreSQL server name, database name, and user credentials.

```powershell
$pgServer = Read-Host "Input Postgres Server Name "
$pgDatabase = Read-Host "Input Postgres Database Name "
$creds = Get-Credential
$pgusername = $creds.UserName  

$BSTR = [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($creds.Password)
$pgpassword = [System.Runtime.InteropServices.Marshal]::PtrToStringAuto($BSTR)
```

### 3. Downloading the ODBC Driver

The script checks if the specified version of the PostgreSQL ODBC driver is already installed. If not, it downloads the driver from the PostgreSQL official website.

```powershell
$downloadVersion = 'psqlodbc_16_00_0000-x64.zip'
$downloadDirectory ='c:\temp\' + $downloadVersion
$uri = 'https://ftp.postgresql.org/pub/odbc/versions/msi/' + $downloadVersion

$drivers = Get-OdbcDriver -Platform '64-bit' | Where-Object Name -Like "Postgres*"

$dExists = 0 
foreach ($d in $drivers) {
    IF ($d.Name -eq $driverName) {
        $dExists = 1
    }
}

IF ($dExists -eq 0) {
    IF (Test-Path $downloadDirectory) {
        Remove-Item $downloadDirectory
    }

    Invoke-WebRequest -Uri $uri -Outfile $downloadDirectory

    $expandDirectory = 'c:\temp\pgodbc\'
    IF (Test-Path $expandDirectory) {
        Remove-Item $expandDirectory
    }

    New-Item -Path 'c:\temp\' -Name "pgodbc" -ItemType "directory"

    Expand-Archive -Path $downloadDirectory -DestinationPath $expandDirectory

    IF (Test-Path $expandDirectory) {
        $msiFile = (Get-ChildItem -Path $expandDirectory | Where-Object { $_.Extension -eq '.msi' }).Name
        $installFile = $expandDirectory + $msiFile
        Start-Process msiexec.exe -Wait -ArgumentList '/I $installFile /quiet ACCEPT_EULA=TRUE'
    }
} else {
    Write-Host "Driver already installed" -BackgroundColor Red
}
```

### 4. Configuring the ODBC DSN

Finally, the script configures the ODBC DSN with the user-provided PostgreSQL server details and credentials.

```powershell
$listODBCDsn = Get-OdbcDsn
$configure = 1

foreach ($l in $listODBCDsn) {
    $attribute = $l.Attribute
    $lServer = $attribute.Servername
    $lData = $attribute.Database

    IF (($lServer -eq $pgServer) -and ($lData -eq $pgDatabase)) {
        Write-Host ''
        Write-Host 'Connector already exists. Exiting configuration.' -BackgroundColor RED
        Write-Host ''
        $configure = 0
        break
    }
}

IF ($configure -eq 1) {
    Write-Host "Configuring Postgres ODBC Connector" -BackgroundColor Green
    Add-OdbcDsn -Name $odbcName -DriverName $driverName -DsnType "System" -SetPropertyValue @("Server=$pgServer", "Trusted_Connection=Yes", "Database=$pgDatabase", "UserName=$pgusername", "Password=$pgpassword")
}
```

## Conclusion

This PowerShell script effectively automates the process of downloading, installing, and configuring the PostgreSQL ODBC driver on Windows machines. By incorporating administrative privilege checks, user input prompts, driver installation, and ODBC DSN configuration, it ensures a streamlined and error-free setup process. This automation not only saves time but also minimizes the potential for configuration errors, thereby enhancing system reliability and administrator efficiency.

## Github 
You can find the complete script on GitHub: [Postgres ODBC Installer for Windows](https://github.com/imsunchips/chip-scripts/blob/main/powershell/odbc/postgres-odbc-installer-win.ps1).