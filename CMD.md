# Windows CMD (Command Prompt) - Complete Guide

## What is CMD?

**CMD** (Command Prompt) is the command-line interpreter for Windows operating systems. It's like a text-based interface to interact with your computer.

## How to Open CMD

1. **Windows + R** → Type `cmd` → Press Enter
2. **Search** → Type "Command Prompt" or "cmd"
3. **Right-click Start Menu** → Choose "Command Prompt" or "Terminal"
4. **Shift + Right-click** in a folder → "Open command window here"

---

## Table of Contents (Organized by Frequency of Use)

1. **Most Used Daily Commands** - Commands you'll use every day
2. **Essential Daily Commands** - Detailed explanations of common commands
3. **Docker Commands** - If you work with Docker
4. **Keyboard Shortcuts** - Speed up your work
5. **Common Daily Workflows** - Real-world examples
6. **Advanced Commands** - Use when needed
7. **Less Commonly Used Commands** - Occasional use
8. **Complete Command Reference** - All commands alphabetically
9. **Batch Files** - Automation scripts
10. **Rarely Used but Powerful Commands** - Advanced features

---

## Most Used Daily Commands (Quick Reference)

These are the commands you'll use almost every day:

```cmd
# Navigation
cd folder_name              # Change directory
cd ..                       # Go up one level
cd \                        # Go to root
dir                         # List files and folders
cls                         # Clear screen

# File Operations
type file.txt               # View file content
copy file.txt backup.txt    # Copy file
move file.txt folder\       # Move file
del file.txt                # Delete file
ren old.txt new.txt         # Rename file

# Folder Operations
mkdir foldername            # Create folder
rmdir foldername            # Remove empty folder
rmdir /s foldername         # Remove folder with contents

# System & Network
ipconfig                    # Check IP address
ping google.com             # Test internet connection
python app.py               # Run Python script
exit                        # Close CMD
```

---

## Essential Daily Commands (Detailed)

### 1. Directory/File Navigation

```cmd
# Show current directory
cd

# Change directory
cd folder_name
cd C:\Users\chand\Desktop

# Go up one level
cd ..

# Go to root directory
cd \

# Go to specific drive
D:
E:

# List files and folders
dir

# List with details
dir /a

# List only folders
dir /ad

# List only files
dir /a-d
```

### 2. File Operations

```cmd
# Create a new file
echo. > filename.txt
type nul > filename.txt

# Create file with content
echo Hello World > file.txt

# Append to file
echo New line >> file.txt

# Display file contents
type filename.txt

# Copy file
copy source.txt destination.txt
copy file.txt C:\backup\file.txt

# Move file
move file.txt C:\newfolder\

# Rename file
ren oldname.txt newname.txt
rename oldname.txt newname.txt

# Delete file
del filename.txt

# Delete multiple files
del *.txt

# Delete with confirmation
del /p filename.txt

# Force delete (no confirmation)
del /f filename.txt
```

### 3. Folder Operations

```cmd
# Create folder
mkdir foldername
md foldername

# Create nested folders
mkdir folder1\folder2\folder3

# Remove empty folder
rmdir foldername
rd foldername

# Remove folder with contents
rmdir /s foldername

# Remove without confirmation
rmdir /s /q foldername

# Copy folder and contents
xcopy source destination /E /I

# Move folder
move foldername C:\newfolder\
```

### 4. System Information

```cmd
# System information
systeminfo

# Computer name
hostname

# IP configuration
ipconfig

# Detailed IP info
ipconfig /all

# Check Windows version
ver

# Display date
date

# Display time
time

# List running processes
tasklist

# Detailed task list
tasklist /v

# Find specific process
tasklist | findstr chrome
```

### 5. Network Commands

```cmd
# Ping a website
ping google.com

# Ping with count
ping -n 10 google.com

# Trace route
tracert google.com

# Network statistics
netstat

# Active connections
netstat -an

# DNS lookup
nslookup google.com

# Release IP address
ipconfig /release

# Renew IP address
ipconfig /renew

# Flush DNS cache
ipconfig /flushdns
```

### 6. Process Management

```cmd
# List running tasks
tasklist

# Kill a process by name
taskkill /IM chrome.exe

# Kill by process ID
taskkill /PID 1234

# Force kill
taskkill /F /IM notepad.exe

# Kill all instances
taskkill /IM chrome.exe /F
```

### 7. Disk Management

```cmd
# Check disk
chkdsk

# Check and fix disk
chkdsk C: /F

# Show disk usage
dir /s

# Format drive (BE CAREFUL!)
format D:

# Show drive information
vol

# List drives
wmic logicaldisk get name
```

### 8. User & Permissions

```cmd
# Show current user
whoami

# List all users
net user

# Add user
net user username password /add

# Delete user
net user username /delete

# Change password
net user username newpassword

# Run as administrator
runas /user:Administrator cmd
```

### 9. Text & Search

```cmd
# Find text in file
find "search text" filename.txt

# Find in multiple files
find "text" *.txt

# Search with case-insensitive
find /i "text" file.txt

# Search in directory
findstr "pattern" *.txt

# Display first few lines
type file.txt | more
```

### 10. Environment Variables

```cmd
# Show all environment variables
set

# Show specific variable
echo %PATH%
echo %USERNAME%
echo %COMPUTERNAME%
echo %USERPROFILE%

# Set temporary variable
set MY_VAR=Hello

# Use the variable
echo %MY_VAR%
```

---

## Docker Commands in CMD (If You Use Docker Daily)

```cmd
# Navigate to docker project
cd C:\Users\chand\OneDrive\Desktop\docker

# Most used Docker commands
docker images                           # List all images
docker ps                               # List running containers
docker ps -a                            # List all containers
docker build -t app-name:latest .       # Build image
docker run -d -p 8080:80 app-name       # Run container
docker logs -f container_name           # Follow logs
docker stop container_name              # Stop container
docker start container_name             # Start container
docker rm container_name                # Remove container
docker rmi image_name                   # Remove image
docker exec -it container_name bash     # Access container
docker system prune                     # Clean up unused data
```

---

## Keyboard Shortcuts (Very Useful!)

| Shortcut | Description |
|----------|-------------|
| `Ctrl + C` | Cancel/Stop current command |
| `Tab` | Auto-complete file/folder names |
| `Up/Down Arrow` | Navigate command history |
| `F7` | Display command history window |
| `cls` | Clear screen |
| `Esc` | Clear current line |

---

## Common Daily Workflows

### Workflow 1: Start Working on a Project
```cmd
C:\Users\chand> cd Desktop\MyProject
C:\Users\chand\Desktop\MyProject> dir
C:\Users\chand\Desktop\MyProject> python app.py
```

### Workflow 2: Check Network Issues
```cmd
C:\Users\chand> ping google.com
C:\Users\chand> ipconfig /all
C:\Users\chand> ipconfig /flushdns
```

### Workflow 3: Find and Kill a Program
```cmd
C:\Users\chand> tasklist | findstr chrome
C:\Users\chand> taskkill /IM chrome.exe /F
```

### Workflow 4: Quick File Management
```cmd
C:\Users\chand> cd Desktop
C:\Users\chand\Desktop> mkdir NewProject
C:\Users\chand\Desktop> cd NewProject
C:\Users\chand\Desktop\NewProject> echo print("Hello") > app.py
C:\Users\chand\Desktop\NewProject> type app.py
```

---

## Advanced Commands (Use When Needed)

### 1. Command Chaining

```cmd
# Run commands sequentially (even if first fails)
command1 & command2

# Run second only if first succeeds
command1 && command2

# Run second only if first fails
command1 || command2

# Pipe output
dir | findstr txt

# Redirect output to file
dir > output.txt

# Append to file
dir >> output.txt
```

### 2. Batch Files

Create a file with `.bat` or `.cmd` extension:

**Example: backup.bat**
```batch
@echo off
echo Starting backup...
xcopy C:\important D:\backup /E /I /Y
echo Backup completed!
pause
```

**Run it:**
```cmd
backup.bat
```

### 3. Useful Shortcuts

| Shortcut | Action |
|----------|--------|
| `cls` | Clear screen |
| `exit` | Exit CMD |
| `Ctrl + C` | Cancel current command |
| `pause` | Pause command execution |
| `F7` | View command history |
| `Up Arrow` | Previous command |
| `Tab` | Auto-complete paths |
| `Enter` | Copy selected text |
| `Right-click` | Paste |

---

## Common Use Cases

### Example 1: Navigate and Create Project
```cmd
C:\Users\chand> cd Desktop
C:\Users\chand\Desktop> mkdir MyProject
C:\Users\chand\Desktop> cd MyProject
C:\Users\chand\Desktop\MyProject> echo print("Hello") > app.py
C:\Users\chand\Desktop\MyProject> type app.py
print("Hello")
```

### Example 2: Find Large Files
```cmd
dir /s /o-s C:\Users\chand\Desktop
```

### Example 3: Network Troubleshooting
```cmd
# Check internet connection
ping 8.8.8.8

# Check DNS
nslookup google.com

# Check local network
ipconfig /all
```

### Example 4: Quick Python Setup
```cmd
cd C:\Users\chand\Desktop\docker
python --version
pip list
pip install flask
python app.py
```

### Example 5: Batch Rename Files
```cmd
# Rename all .txt to .bak
for %f in (*.txt) do ren "%f" *.bak
```

---

## CMD vs PowerShell vs Terminal

| Feature | CMD | PowerShell | Windows Terminal |
|---------|-----|------------|------------------|
| **Type** | Legacy shell | Modern shell | Terminal emulator |
| **Power** | Basic | Advanced | Hosts any shell |
| **Syntax** | Simple | Complex | N/A |
| **Use Case** | Quick tasks | Scripting, automation | Better UI for shells |

---

## Helpful Tips

### 1. Get Help for Any Command
```cmd
command /?

# Examples:
dir /?
copy /?
ipconfig /?
```

### 2. Command History
```cmd
# View history
doskey /history

# Search history
F7  # (opens history window)
```

### 3. Create Aliases
```cmd
doskey ls=dir
doskey clear=cls

# Now you can use:
ls
clear
```

### 4. Run Multiple Commands
```cmd
cd Desktop && mkdir test && cd test && echo Done
```

---

## Essential Commands Quick Reference

| Command | Description | Example |
|---------|-------------|---------|
| `cd` | Change directory | `cd Desktop` |
| `dir` | List files | `dir` |
| `mkdir` | Create folder | `mkdir test` |
| `del` | Delete file | `del file.txt` |
| `copy` | Copy file | `copy a.txt b.txt` |
| `move` | Move file | `move file.txt C:\new\` |
| `ren` | Rename | `ren old.txt new.txt` |
| `type` | Show file content | `type file.txt` |
| `cls` | Clear screen | `cls` |
| `exit` | Close CMD | `exit` |
| `ipconfig` | Network info | `ipconfig /all` |
| `ping` | Test connection | `ping google.com` |
| `tasklist` | List processes | `tasklist` |
| `taskkill` | Kill process | `taskkill /IM app.exe` |
| `echo` | Display text | `echo Hello` |
| `find` | Search text | `find "text" file.txt` |
| `xcopy` | Copy folders | `xcopy src dest /E` |
| `systeminfo` | System details | `systeminfo` |
| `hostname` | Computer name | `hostname` |
| `whoami` | Current user | `whoami` |

---

## Less Commonly Used Commands

### User & Permissions (Occasional Use)

```cmd
# Show current user
whoami

# List all users
net user

# Add user
net user username password /add

# Delete user
net user username /delete

# Change password
net user username newpassword

# Run as administrator
runas /user:Administrator cmd
```

### Disk Management (Occasional Use)

```cmd
# Check disk
chkdsk

# Check and fix disk
chkdsk C: /F

# Show disk usage
dir /s

# Format drive (BE CAREFUL!)
format D:

# Show drive information
vol

# List drives
wmic logicaldisk get name
```

---

## Complete Command Reference (Alphabetical)

### File and Folder Operations Cheatsheet

### Creating Files/Folders
```cmd
mkdir newfolder              # Create folder
echo text > file.txt         # Create file with content
type nul > empty.txt         # Create empty file
```

### Viewing Content
```cmd
dir                          # List current directory
dir /s                       # List with subdirectories
dir /a                       # List including hidden
type file.txt                # Display file content
more file.txt                # Display with pagination
```

### Copying and Moving
```cmd
copy file.txt backup.txt     # Copy file
xcopy folder1 folder2 /E     # Copy folder with subfolders
move file.txt newloc\        # Move file
move folder1 newloc\         # Move folder
```

### Deleting
```cmd
del file.txt                 # Delete file
del *.txt                    # Delete all .txt files
rmdir folder                 # Delete empty folder
rmdir /s folder              # Delete folder with contents
rmdir /s /q folder           # Delete without confirmation
```

### Renaming
```cmd
ren old.txt new.txt          # Rename file
ren oldfolder newfolder      # Rename folder
```

---

## Network Commands Cheatsheet

```cmd
# Check connectivity
ping google.com              # Basic ping
ping -n 10 google.com        # Ping 10 times
ping -t google.com           # Continuous ping

# IP Information
ipconfig                     # Basic IP info
ipconfig /all                # Detailed IP info
ipconfig /release            # Release IP
ipconfig /renew              # Renew IP
ipconfig /flushdns           # Clear DNS cache

# Network diagnostics
tracert google.com           # Trace route
nslookup google.com          # DNS lookup
netstat                      # Network statistics
netstat -an                  # All connections with ports
netstat -b                   # Show executable

# Network shares
net view                     # View shared resources
net use                      # Map network drive
```

---

## Process Management Cheatsheet

```cmd
# View processes
tasklist                     # List all processes
tasklist /v                  # Verbose list
tasklist /svc                # Show services
tasklist | findstr chrome    # Find specific process

# Kill processes
taskkill /IM notepad.exe     # Kill by name
taskkill /PID 1234           # Kill by process ID
taskkill /F /IM chrome.exe   # Force kill
taskkill /IM chrome.exe /T   # Kill process tree

# Start processes
start notepad.exe            # Start application
start "" "C:\Program.exe"    # Start with path
start /min notepad.exe       # Start minimized
```

---

## Docker Commands in CMD

```cmd
# Navigate to docker project
cd C:\Users\chand\OneDrive\Desktop\docker

# Docker image commands
docker images                           # List all images
docker build -t new-app:latest .        # Build image
docker rmi new-app:latest               # Remove image
docker rmi -f image_id                  # Force remove image

# Docker container commands
docker ps                               # List running containers
docker ps -a                            # List all containers
docker run -d -p 8080:80 new-app:latest # Run container
docker start container_name             # Start stopped container
docker stop container_name              # Stop running container
docker rm container_name                # Remove container
docker rm -f container_name             # Force remove container

# Docker logs and debugging
docker logs container_name              # View logs
docker logs -f container_name           # Follow logs
docker exec -it container_name bash     # Access container

# Docker cleanup
docker system prune                     # Remove unused data
docker system prune -a                  # Remove all unused data
```

---

## Batch Files (For Automation)

### What are Batch Files?

Batch files (.bat or .cmd) are scripts that contain multiple CMD commands. They help automate repetitive tasks.

### Batch File Examples

### Example 1: Simple Backup Script
**backup.bat**
```batch
@echo off
echo Starting backup process...
set SOURCE=C:\Important
set DEST=D:\Backup
xcopy %SOURCE% %DEST% /E /I /Y
echo Backup completed successfully!
pause
```

### Example 2: Project Setup Script
**setup.bat**
```batch
@echo off
echo Setting up project...
mkdir myproject
cd myproject
mkdir src tests docs
echo # My Project > README.md
echo print("Hello World") > src\main.py
echo Setup complete!
pause
```

### Example 3: Network Diagnostics
**netcheck.bat**
```batch
@echo off
echo Network Diagnostics
echo ==================
echo.
echo Checking internet connection...
ping -n 3 8.8.8.8
echo.
echo IP Configuration:
ipconfig
echo.
echo DNS Information:
nslookup google.com
pause
```

### Example 4: Clean Temp Files
**cleanup.bat**
```batch
@echo off
echo Cleaning temporary files...
del /q /f /s %TEMP%\*
echo Temp files cleaned!
pause
```

---

## Rarely Used but Powerful Commands

### Advanced Tips and Tricks

### 1. Create System Restore Point
```cmd
wmic.exe /Namespace:\\root\default Path SystemRestore Call CreateRestorePoint "My Restore Point", 100, 7
```

### 2. Find Large Files
```cmd
forfiles /S /M * /C "cmd /c if @fsize GEQ 104857600 echo @path @fsize"
```

### 3. Schedule Tasks
```cmd
# Run a task at specific time
schtasks /create /tn "MyTask" /tr "C:\script.bat" /sc daily /st 09:00
```

### 4. Check System Uptime
```cmd
systeminfo | find "System Boot Time"
```

### 5. List Installed Programs
```cmd
wmic product get name,version
```

### 6. Check Disk Health
```cmd
wmic diskdrive get status
```

---

## Common Error Solutions

### Error: "Access Denied"
**Solution:** Run CMD as Administrator
- Right-click CMD → "Run as administrator"

### Error: "Command not found"
**Solution:** Check if the program is in PATH
```cmd
echo %PATH%
# Add to PATH if needed
```

### Error: "File not found"
**Solution:** Check current directory and file path
```cmd
cd
dir
# Use full path if needed
```

### Error: "Permission denied"
**Solution:** Check file permissions
```cmd
# Take ownership
takeown /f filename.txt
# Grant permissions
icacls filename.txt /grant username:F
```

---

## Keyboard Shortcuts

| Shortcut | Description |
|----------|-------------|
| `Ctrl + C` | Cancel/Stop current command |
| `Ctrl + V` | Paste (modern CMD) |
| `Ctrl + A` | Select all text |
| `Tab` | Auto-complete file/folder names |
| `Up/Down Arrow` | Navigate command history |
| `F1` | Paste last command character by character |
| `F3` | Repeat last command |
| `F7` | Display command history |
| `F9` | Run command by number from history |
| `Esc` | Clear current line |
| `Alt + Enter` | Toggle fullscreen |

---

## Best Practices

1. **Always backup important data** before running destructive commands
2. **Use quotes** for paths with spaces: `cd "C:\Program Files"`
3. **Test batch files** in a safe environment first
4. **Document your scripts** with comments using `REM` or `::`
5. **Use `/Y` flag** for automatic yes to prompts in scripts
6. **Check command help** with `command /?` before using
7. **Run as Administrator** when making system changes
8. **Use `pause`** at the end of batch files to see output

---

## Resources

- Get help for any command: `command /?`
- Windows CMD documentation: [Microsoft Docs](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/)
- Practice commands in a test folder first
- Create batch files for repetitive tasks

---

## Quick Daily Commands

```cmd
# Navigate to project
cd C:\Users\chand\Desktop\docker

# Check files
dir

# Run Python app
python app.py

# Check running processes
tasklist | findstr python

# Check network
ipconfig
ping google.com

# Clear screen
cls

# Exit
exit
```

---

**Happy Command Line Usage! 💻**
