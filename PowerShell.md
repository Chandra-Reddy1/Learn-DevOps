# PowerShell Commands - Complete Guide

## What is PowerShell?

**PowerShell** is a powerful command-line shell and scripting language built on .NET Framework. It's more advanced than CMD and is the default shell in modern Windows systems. PowerShell commands are called **cmdlets** (command-lets).

## How to Open PowerShell

1. **Windows + X** → Choose "Windows PowerShell" or "Terminal"
2. **Windows + R** → Type `powershell` → Press Enter
3. **Search** → Type "PowerShell"
4. **Right-click Start Menu** → "Windows PowerShell"
5. **For Admin:** Right-click → "Run as Administrator"

---

## Table of Contents (Organized by Frequency of Use)

1. **Most Used Daily Commands** - Commands you'll use every day
2. **Essential Daily Commands** - Detailed explanations of common commands
3. **Docker Commands** - If you work with Docker
4. **Git Commands** - Version control basics
5. **Keyboard Shortcuts** - Speed up your work
6. **Common Daily Workflows** - Real-world examples
7. **Package Management** - Installing software
8. **Advanced Commands** - Use when needed
9. **Less Commonly Used Commands** - Occasional use
10. **Complete Command Reference** - All commands alphabetically
11. **PowerShell Scripting** - Automation scripts
12. **Rarely Used but Powerful Commands** - Advanced features

---

## Most Used Daily Commands (Quick Reference)

These are the commands you'll use almost every day:

```powershell
# Navigation
cd folder_name              # Change directory (or Set-Location)
cd ..                       # Go up one level
cd ~                        # Go to user profile
pwd                         # Print working directory (or Get-Location)
ls                          # List files (or Get-ChildItem)
dir                         # List files (alias for Get-ChildItem)
cls                         # Clear screen (or Clear-Host)

# File Operations
cat file.txt                # View file (or Get-Content)
type file.txt               # View file
copy file.txt backup.txt    # Copy file (or Copy-Item)
move file.txt folder\       # Move file (or Move-Item)
del file.txt                # Delete file (or Remove-Item)
ren old.txt new.txt         # Rename file (or Rename-Item)
ni file.txt                 # Create new file (or New-Item)

# Folder Operations
mkdir foldername            # Create folder (or New-Item -ItemType Directory)
rmdir foldername            # Remove folder (or Remove-Item)

# System & Network
Test-Connection google.com  # Test internet (like ping)
ipconfig                    # Show IP configuration
Get-Process                 # List running processes
python app.py               # Run Python script
exit                        # Close PowerShell
Get-History                 # Show command history
```

---

## Essential Daily Commands (Detailed)

### 1. Navigation and Directory Commands

```powershell
# Show current directory
Get-Location
pwd                         # Alias

# Change directory
Set-Location C:\Users\Documents
cd C:\Users\Documents       # Alias

# Go to user profile directory
cd ~
cd $HOME

# Go up one level
cd ..

# Go up two levels
cd ..\..

# Go to previous directory
cd -

# Go to root of current drive
cd \

# Change drive
D:
E:

# List files and folders
Get-ChildItem
ls                          # Alias
dir                         # Alias

# List with details
ls -Force                   # Show hidden files

# List recursively
Get-ChildItem -Recurse

# List only files
Get-ChildItem -File

# List only directories
Get-ChildItem -Directory

# List with specific extension
Get-ChildItem -Filter *.txt
Get-ChildItem *.py

# List sorted by size
Get-ChildItem | Sort-Object Length -Descending

# List sorted by date
Get-ChildItem | Sort-Object LastWriteTime -Descending
```

### 2. File Viewing and Content

```powershell
# View entire file
Get-Content file.txt
cat file.txt                # Alias
type file.txt               # Alias

# View first 10 lines
Get-Content file.txt -Head 10

# View last 10 lines
Get-Content file.txt -Tail 10

# View file in real-time (like tail -f)
Get-Content file.txt -Wait

# Count lines in file
(Get-Content file.txt).Count

# Search text in file
Select-String "search term" file.txt

# Search in multiple files
Get-ChildItem -Filter *.txt | Select-String "pattern"

# Edit file with notepad
notepad file.txt

# Create new file
New-Item file.txt -ItemType File
ni file.txt                 # Alias
"" > file.txt              # Simple way

# Add content to file
Add-Content file.txt "New line"
"Text" >> file.txt          # Append

# Overwrite file content
Set-Content file.txt "New content"
"Text" > file.txt           # Overwrite
```

### 3. File Operations

```powershell
# Copy file
Copy-Item source.txt destination.txt
cp source.txt dest.txt      # Alias
copy source.txt dest.txt    # Alias

# Copy file to directory
Copy-Item file.txt C:\Backup\

# Copy directory recursively
Copy-Item source_folder destination_folder -Recurse

# Move or rename file
Move-Item oldname.txt newname.txt
mv oldname.txt newname.txt  # Alias

# Move file to directory
Move-Item file.txt C:\Documents\

# Rename file
Rename-Item old.txt new.txt
ren old.txt new.txt         # Alias

# Delete file
Remove-Item file.txt
del file.txt                # Alias
rm file.txt                 # Alias

# Delete without confirmation
Remove-Item file.txt -Force

# Delete directory recursively
Remove-Item folder -Recurse -Force

# Delete files by pattern
Remove-Item *.tmp

# Delete files older than 30 days
Get-ChildItem -Path C:\Temp -Recurse | Where-Object {$_.LastWriteTime -lt (Get-Date).AddDays(-30)} | Remove-Item -Force
```

### 4. Folder Operations

```powershell
# Create directory
New-Item foldername -ItemType Directory
mkdir foldername            # Alias
md foldername               # Alias

# Create nested directories
New-Item parent\child\grandchild -ItemType Directory -Force

# Create multiple directories
mkdir folder1, folder2, folder3

# Remove empty directory
Remove-Item foldername
rmdir foldername            # Alias

# Remove directory with contents
Remove-Item folder -Recurse -Force

# Copy directory
Copy-Item source_folder destination_folder -Recurse

# Move directory
Move-Item old_folder new_folder

# Get folder size
(Get-ChildItem C:\Folder -Recurse | Measure-Object -Property Length -Sum).Sum / 1GB
```

### 5. File Permissions and Properties

```powershell
# View file properties
Get-Item file.txt | Format-List
Get-ItemProperty file.txt

# Get file attributes
(Get-Item file.txt).Attributes

# Set file as read-only
Set-ItemProperty file.txt -Name IsReadOnly -Value $true

# Remove read-only
Set-ItemProperty file.txt -Name IsReadOnly -Value $false

# Hide file
(Get-Item file.txt).Attributes += 'Hidden'

# Unhide file
(Get-Item file.txt).Attributes -= 'Hidden'

# Get file owner
(Get-Acl file.txt).Owner

# Check if file exists
Test-Path file.txt

# Check if directory exists
Test-Path C:\Folder -PathType Container

# Get file hash (checksum)
Get-FileHash file.txt -Algorithm SHA256
Get-FileHash file.txt -Algorithm MD5
```

### 6. Search and Find

```powershell
# Find files by name
Get-ChildItem -Path C:\ -Filter "file.txt" -Recurse -ErrorAction SilentlyContinue

# Find files with specific extension
Get-ChildItem -Path C:\Projects -Filter *.py -Recurse

# Find files modified in last 7 days
Get-ChildItem | Where-Object {$_.LastWriteTime -gt (Get-Date).AddDays(-7)}

# Find large files (over 100MB)
Get-ChildItem -Recurse | Where-Object {$_.Length -gt 100MB} | Sort-Object Length -Descending

# Search text in files
Select-String "search term" -Path *.txt

# Search recursively in all files
Get-ChildItem -Recurse -Filter *.log | Select-String "ERROR"

# Find command location
Get-Command python
gcm python                  # Alias

# Search in command history
Get-History | Where-Object {$_.CommandLine -like "*docker*"}
```

### 7. System Information

```powershell
# Show computer name
$env:COMPUTERNAME
hostname

# Show current user
$env:USERNAME
whoami

# Show Windows version
Get-ComputerInfo | Select-Object WindowsVersion, OsArchitecture

# Detailed system information
Get-ComputerInfo
systeminfo                  # CMD command still works

# Disk space (all drives)
Get-PSDrive -PSProvider FileSystem

# Disk space with details
Get-Volume

# Memory information
Get-CimInstance Win32_PhysicalMemory | Select-Object Capacity, Speed

# CPU information
Get-CimInstance Win32_Processor | Select-Object Name, NumberOfCores, NumberOfLogicalProcessors

# Show environment variables
Get-ChildItem Env:
$env:PATH
$env:USERPROFILE

# Set environment variable (current session)
$env:MY_VAR = "value"

# Get system uptime
(Get-Date) - (Get-CimInstance Win32_OperatingSystem).LastBootUpTime

# Show installed programs
Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\* | Select-Object DisplayName, DisplayVersion

# Show Windows updates
Get-HotFix

# Last 10 updates
Get-HotFix | Sort-Object InstalledOn -Descending | Select-Object -First 10
```

### 8. Network Commands

```powershell
# Show IP configuration
Get-NetIPConfiguration
ipconfig                    # CMD command still works

# Detailed IP info
Get-NetIPAddress

# Test connection (ping)
Test-Connection google.com
Test-Connection google.com -Count 4

# Continuous ping
Test-Connection google.com -Count 1000

# Quiet ping (returns true/false)
Test-Connection google.com -Quiet

# Trace route
Test-NetConnection google.com -TraceRoute

# Test specific port
Test-NetConnection google.com -Port 443

# DNS lookup
Resolve-DnsName google.com
nslookup google.com         # CMD command still works

# Show network adapters
Get-NetAdapter

# Show network connections
Get-NetTCPConnection

# Show listening ports
Get-NetTCPConnection -State Listen

# Find process using specific port
Get-NetTCPConnection -LocalPort 8080 | Select-Object OwningProcess
Get-Process -Id (Get-NetTCPConnection -LocalPort 8080).OwningProcess

# Download file from URL
Invoke-WebRequest -Uri "https://example.com/file.zip" -OutFile "file.zip"
wget "https://example.com/file.zip" -OutFile "file.zip"  # Alias

# Get public IP address
(Invoke-WebRequest -Uri "https://ifconfig.me/ip").Content

# Flush DNS cache
Clear-DnsClientCache

# Show DNS cache
Get-DnsClientCache
```

### 9. Process Management

```powershell
# List all processes
Get-Process
ps                          # Alias

# Find specific process
Get-Process chrome
Get-Process | Where-Object {$_.Name -like "*chrome*"}

# Sort by memory usage
Get-Process | Sort-Object WS -Descending

# Sort by CPU usage
Get-Process | Sort-Object CPU -Descending

# Get top 10 memory consumers
Get-Process | Sort-Object WS -Descending | Select-Object -First 10 Name, @{N='Memory(MB)';E={[math]::Round($_.WS / 1MB, 2)}}

# Kill process by name
Stop-Process -Name notepad
Stop-Process -Name chrome -Force

# Kill process by ID
Stop-Process -Id 1234

# Kill all processes with name
Get-Process chrome | Stop-Process -Force

# Start process
Start-Process notepad
Start-Process "C:\Program Files\app.exe"

# Start process as admin
Start-Process powershell -Verb RunAs

# Start process and wait for it to finish
Start-Process notepad -Wait
```

---

## Docker Commands (If You Use Docker Daily)

```powershell
# List images
docker images

# List running containers
docker ps

# List all containers
docker ps -a

# Build image
docker build -t app-name:latest .

# Run container
docker run -d -p 8080:80 app-name

# Run with volume mount
docker run -v ${PWD}:/app app-name

# Stop container
docker stop container_name

# Start container
docker start container_name

# Remove container
docker rm container_name

# Remove image
docker rmi image_name

# View logs
docker logs container_name

# Follow logs
docker logs -f container_name

# Execute command in container
docker exec -it container_name bash

# Clean up
docker system prune
docker system prune -a --volumes

# Docker compose
docker-compose up -d
docker-compose down
docker-compose logs -f
docker-compose ps
```

---

## Git Commands (Daily Version Control)

```powershell
# Initialize repository
git init

# Clone repository
git clone https://github.com/user/repo.git

# Check status
git status

# Add files to staging
git add file.txt
git add .
git add *.py

# Commit changes
git commit -m "Commit message"

# Push to remote
git push origin main

# Pull from remote
git pull origin main

# Create new branch
git checkout -b new-branch

# Switch branch
git checkout branch-name

# List branches
git branch

# Merge branch
git merge branch-name

# View commit history
git log
git log --oneline

# View differences
git diff

# Discard changes
git checkout -- file.txt

# Unstage file
git reset file.txt

# View remote repositories
git remote -v

# Add remote
git remote add origin https://github.com/user/repo.git
```

---

## Keyboard Shortcuts (Very Useful!)

| Shortcut | Description |
|----------|-------------|
| `Ctrl + C` | Cancel/Stop current command |
| `Ctrl + L` | Clear screen (or type `cls`) |
| `Ctrl + R` | Search command history |
| `Tab` | Auto-complete commands and paths |
| `Ctrl + Space` | Show available parameters |
| `F7` | Command history window |
| `Up/Down Arrow` | Navigate command history |
| `Ctrl + Left/Right Arrow` | Move word by word |
| `Home` | Move to beginning of line |
| `End` | Move to end of line |
| `Esc` | Clear current line |
| `F8` | Search history backward |
| `Alt + F7` | Clear command history |

---

## Common Daily Workflows

### Workflow 1: Start Working on a Project
```powershell
cd C:\Users\chand\Desktop\MyProject
Get-ChildItem
git status
git pull origin main
python app.py
```

### Workflow 2: Create and Setup New Project
```powershell
New-Item -Path C:\Projects\NewProject -ItemType Directory
cd C:\Projects\NewProject
git init
"# New Project" > README.md
mkdir src, tests, docs
New-Item src\main.py -ItemType File
Get-ChildItem
```

### Workflow 3: Check System Resources
```powershell
# Check disk space
Get-PSDrive -PSProvider FileSystem

# Check memory
Get-CimInstance Win32_OperatingSystem | Select-Object FreePhysicalMemory, TotalVisibleMemorySize

# Check CPU usage
Get-Counter '\Processor(_Total)\% Processor Time'

# Check processes
Get-Process | Sort-Object WS -Descending | Select-Object -First 10
```

### Workflow 4: File Management
```powershell
cd C:\Users\chand\Downloads
Get-ChildItem -File
Move-Item *.pdf C:\Users\chand\Documents\PDFs\
Remove-Item -Recurse temp_folder
Get-ChildItem | Where-Object {$_.LastWriteTime -lt (Get-Date).AddDays(-30)} | Remove-Item
```

### Workflow 5: Network Troubleshooting
```powershell
# Test internet connection
Test-Connection google.com -Count 4

# Check IP configuration
Get-NetIPConfiguration

# Test specific port
Test-NetConnection google.com -Port 443

# Check DNS
Resolve-DnsName google.com

# Flush DNS
Clear-DnsClientCache
```

### Workflow 6: Quick Backup
```powershell
$date = Get-Date -Format "yyyyMMdd"
Compress-Archive -Path C:\Important -DestinationPath "C:\Backups\backup_$date.zip"
Copy-Item -Path C:\Projects -Destination "D:\Backups\projects_$date" -Recurse
```

---

## Package Management (Installing Software)

### Using Winget (Windows Package Manager)

```powershell
# Search for package
winget search python

# Install package
winget install Python.Python.3.11

# List installed packages
winget list

# Upgrade package
winget upgrade Python.Python.3.11

# Upgrade all packages
winget upgrade --all

# Uninstall package
winget uninstall Python.Python.3.11

# Show package info
winget show Python.Python.3.11
```

### Using Chocolatey (If Installed)

```powershell
# Search package
choco search python

# Install package
choco install python

# List installed
choco list --local

# Upgrade package
choco upgrade python

# Upgrade all
choco upgrade all

# Uninstall
choco uninstall python
```

### Using PowerShell Gallery (Modules)

```powershell
# Find module
Find-Module -Name PSReadLine

# Install module
Install-Module -Name PSReadLine -Force

# List installed modules
Get-InstalledModule

# Update module
Update-Module -Name PSReadLine

# Uninstall module
Uninstall-Module -Name PSReadLine

# Import module
Import-Module PSReadLine

# List available modules
Get-Module -ListAvailable
```

---

## Advanced Commands (Use When Needed)

### Text Processing and Filtering

```powershell
# Select specific properties
Get-Process | Select-Object Name, CPU, WS

# Where-Object filtering
Get-Process | Where-Object {$_.CPU -gt 100}

# Sort objects
Get-ChildItem | Sort-Object Length -Descending

# Measure objects
Get-ChildItem | Measure-Object -Property Length -Sum -Average

# Group objects
Get-Process | Group-Object ProcessName

# Format output as table
Get-Process | Format-Table Name, CPU, WS -AutoSize

# Format output as list
Get-Process chrome | Format-List *

# Export to CSV
Get-Process | Export-Csv processes.csv -NoTypeInformation

# Import from CSV
Import-Csv processes.csv

# Export to JSON
Get-Process | Select-Object Name, CPU | ConvertTo-Json | Out-File processes.json

# Import from JSON
Get-Content processes.json | ConvertFrom-Json

# Convert to HTML
Get-Process | ConvertTo-Html | Out-File processes.html
```

### Compression and Archives

```powershell
# Create zip archive
Compress-Archive -Path C:\Folder -DestinationPath archive.zip

# Extract zip archive
Expand-Archive -Path archive.zip -DestinationPath C:\Extracted

# Add to existing archive
Compress-Archive -Path C:\NewFiles -Update -DestinationPath archive.zip

# Create archive of multiple items
Compress-Archive -Path file1.txt, file2.txt, folder1 -DestinationPath archive.zip
```

### Working with CSV and JSON

```powershell
# Read CSV file
Import-Csv data.csv

# Filter CSV data
Import-Csv data.csv | Where-Object {$_.Age -gt 18}

# Export to CSV
Get-Process | Export-Csv processes.csv -NoTypeInformation

# Parse JSON
$json = Get-Content config.json | ConvertFrom-Json

# Create JSON
$data = @{
    name = "John"
    age = 30
    city = "Hyderabad"
}
$data | ConvertTo-Json | Out-File data.json
```

---

## Less Commonly Used Commands

### User and Security

```powershell
# List local users
Get-LocalUser

# Create new user
New-LocalUser -Name "username" -Password (ConvertTo-SecureString "Password123!" -AsPlainText -Force)

# Delete user
Remove-LocalUser -Name "username"

# Add user to group
Add-LocalGroupMember -Group "Administrators" -Member "username"

# List local groups
Get-LocalGroup

# Get group members
Get-LocalGroupMember -Group "Administrators"

# Change password
Set-LocalUser -Name "username" -Password (ConvertTo-SecureString "NewPassword123!" -AsPlainText -Force)

# Check if running as admin
([Security.Principal.WindowsPrincipal] [Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)
```

### Service Management

```powershell
# List all services
Get-Service

# List running services
Get-Service | Where-Object {$_.Status -eq "Running"}

# Get specific service
Get-Service -Name "wuauserv"

# Start service
Start-Service -Name "wuauserv"

# Stop service
Stop-Service -Name "wuauserv"

# Restart service
Restart-Service -Name "wuauserv"

# Set service startup type
Set-Service -Name "wuauserv" -StartupType Automatic
Set-Service -Name "wuauserv" -StartupType Manual
Set-Service -Name "wuauserv" -StartupType Disabled
```

### Registry Operations

```powershell
# Navigate to registry
cd HKLM:\
cd HKCU:\

# List registry keys
Get-ChildItem HKLM:\Software

# Get registry value
Get-ItemProperty "HKLM:\Software\Microsoft\Windows\CurrentVersion" -Name "ProgramFilesDir"

# Set registry value
Set-ItemProperty "HKCU:\Software\MyApp" -Name "Setting" -Value "Value"

# Create registry key
New-Item -Path "HKCU:\Software\MyApp"

# Delete registry key
Remove-Item -Path "HKCU:\Software\MyApp"

# Check if registry key exists
Test-Path "HKCU:\Software\MyApp"
```

---

## Complete Command Reference (Alphabetical)

### File System Cmdlets

```powershell
Clear-Content          # Delete content but keep file
Copy-Item              # Copy files/folders (cp, copy)
Get-ChildItem          # List items (ls, dir, gci)
Get-Content            # Read file (cat, type, gc)
Get-Item               # Get file/folder properties
Get-Location           # Get current directory (pwd, gl)
Move-Item              # Move/rename (mv, move, mi)
New-Item               # Create file/folder (ni)
Remove-Item            # Delete file/folder (rm, del, ri)
Rename-Item            # Rename (ren, rni)
Set-Content            # Write to file (sc)
Set-Location           # Change directory (cd, chdir, sl)
Test-Path              # Check if path exists
```

### Process Cmdlets

```powershell
Get-Process            # List processes (ps, gps)
Start-Process          # Start process (start, saps)
Stop-Process           # Kill process (kill, spps)
Wait-Process           # Wait for process to stop
```

### Service Cmdlets

```powershell
Get-Service            # List services (gsv)
Start-Service          # Start service (sasv)
Stop-Service           # Stop service (spsv)
Restart-Service        # Restart service
Set-Service            # Configure service
```

### Network Cmdlets

```powershell
Get-NetAdapter         # Network adapters
Get-NetIPAddress       # IP addresses
Get-NetIPConfiguration # IP configuration
Get-NetTCPConnection   # TCP connections
Test-Connection        # Ping host
Test-NetConnection     # Test network connectivity
Resolve-DnsName        # DNS lookup
Clear-DnsClientCache   # Flush DNS
Invoke-WebRequest      # HTTP request (wget, iwr, curl)
```

---

## PowerShell Scripting (For Automation)

### What are PowerShell Scripts?

PowerShell scripts are files with `.ps1` extension that contain multiple PowerShell commands.

### Enable Script Execution

```powershell
# Check current execution policy
Get-ExecutionPolicy

# Set execution policy (run as admin)
Set-ExecutionPolicy RemoteSigned

# Run script
.\script.ps1
```

### Script Examples

#### Example 1: Simple Backup Script
**backup.ps1**
```powershell
# Backup script
param(
    [string]$Source = "C:\Important",
    [string]$Destination = "D:\Backups"
)

$date = Get-Date -Format "yyyyMMdd_HHmmss"
$backupName = "backup_$date.zip"

Write-Host "Starting backup..."
Compress-Archive -Path $Source -DestinationPath "$Destination\$backupName"
Write-Host "Backup completed: $backupName"
```

#### Example 2: System Information Script
**sysinfo.ps1**
```powershell
# System information script

Write-Host "===== System Information =====" -ForegroundColor Cyan
Write-Host "Computer Name: $env:COMPUTERNAME"
Write-Host "User Name: $env:USERNAME"
Write-Host "OS: $((Get-CimInstance Win32_OperatingSystem).Caption)"

Write-Host "`n===== CPU Information =====" -ForegroundColor Cyan
$cpu = Get-CimInstance Win32_Processor
Write-Host "CPU: $($cpu.Name)"
Write-Host "Cores: $($cpu.NumberOfCores)"

Write-Host "`n===== Memory Information =====" -ForegroundColor Cyan
$os = Get-CimInstance Win32_OperatingSystem
$totalMemory = [math]::Round($os.TotalVisibleMemorySize / 1MB, 2)
$freeMemory = [math]::Round($os.FreePhysicalMemory / 1MB, 2)
Write-Host "Total Memory: $totalMemory GB"
Write-Host "Free Memory: $freeMemory GB"

Write-Host "`n===== Disk Information =====" -ForegroundColor Cyan
Get-PSDrive -PSProvider FileSystem | Where-Object {$_.Used -ne $null} | ForEach-Object {
    $usedGB = [math]::Round($_.Used / 1GB, 2)
    $freeGB = [math]::Round($_.Free / 1GB, 2)
    Write-Host "$($_.Name): Used: $usedGB GB | Free: $freeGB GB"
}
```

#### Example 3: Project Setup Script
**setup-project.ps1**
```powershell
# Project setup script
param(
    [Parameter(Mandatory=$true)]
    [string]$ProjectName
)

$projectPath = "C:\Projects\$ProjectName"

if (Test-Path $projectPath) {
    Write-Host "Project already exists!" -ForegroundColor Red
    exit
}

Write-Host "Creating project: $ProjectName" -ForegroundColor Green

# Create project structure
New-Item -Path $projectPath -ItemType Directory
New-Item -Path "$projectPath\src" -ItemType Directory
New-Item -Path "$projectPath\tests" -ItemType Directory
New-Item -Path "$projectPath\docs" -ItemType Directory

# Create files
"# $ProjectName" | Out-File "$projectPath\README.md"
"" | Out-File "$projectPath\src\main.py"

# Initialize git
Set-Location $projectPath
git init

Write-Host "Project $ProjectName created successfully!" -ForegroundColor Green
```

### Script Variables and Control Flow

```powershell
# Variables
$name = "John"
$age = 25
$items = @("apple", "banana", "orange")

# User input
$username = Read-Host "Enter your name"
Write-Host "Hello, $username!"

# If statement
if ($age -gt 18) {
    Write-Host "Adult"
} else {
    Write-Host "Minor"
}

# For loop
for ($i = 1; $i -le 5; $i++) {
    Write-Host "Number: $i"
}

# ForEach loop
foreach ($item in $items) {
    Write-Host "Item: $item"
}

# While loop
$count = 0
while ($count -lt 5) {
    Write-Host "Count: $count"
    $count++
}

# Try-Catch (error handling)
try {
    Get-Content "nonexistent.txt"
} catch {
    Write-Host "Error: $_" -ForegroundColor Red
}

# Functions
function Get-Greeting {
    param([string]$Name = "World")
    return "Hello, $Name!"
}

Get-Greeting
Get-Greeting -Name "PowerShell"
```

---

## Useful Aliases (Add to Profile)

### PowerShell Profile

```powershell
# Check if profile exists
Test-Path $PROFILE

# Create profile
New-Item -Path $PROFILE -ItemType File -Force

# Edit profile
notepad $PROFILE

# Reload profile
. $PROFILE
```

### Profile Content Examples

```powershell
# Custom functions
function .. { Set-Location .. }
function ... { Set-Location ..\.. }
function ~ { Set-Location ~ }

# Git shortcuts
function gs { git status }
function ga { git add $args }
function gc { git commit -m $args }
function gp { git push }

# Docker shortcuts
function dps { docker ps }
function di { docker images }

# System shortcuts
function Update-System {
    winget upgrade --all
}
```

---

## Tips and Best Practices

1. **Use Tab completion** - Press Tab to auto-complete
2. **Use `Get-Help`** - Learn commands with `Get-Help Get-Process`
3. **Update help files** - Run `Update-Help` as admin
4. **Leverage the pipeline** - PowerShell's strength
5. **Use `-WhatIf`** - Test without executing
6. **Use `-Verbose`** - See details
7. **Comment your scripts** - Use `#` for comments
8. **Error handling** - Use try-catch
9. **Learn object properties** - Use `Get-Member`

---

## Getting Help

```powershell
# Get help for command
Get-Help Get-Process
help Get-Process

# Get examples
Get-Help Get-Process -Examples

# Get online help
Get-Help Get-Process -Online

# Update help files (run as admin)
Update-Help

# Search for commands
Get-Command *process*

# Explore object properties
Get-Process | Get-Member
```

---

## Quick Daily Commands Summary

```powershell
# Navigation
cd, pwd, ls, dir, cd .., cd ~, cls

# File operations
cat, copy, cp, move, mv, del, rm, ren

# Directory operations
mkdir, rmdir, Get-ChildItem

# Search
Select-String, Where-Object

# System
Get-Process, Get-Service

# Network
Test-Connection, Get-NetIPConfiguration

# Help
Get-Help, Get-Command, Get-Member
```

---

**Happy PowerShell Usage! ⚡**
