# Linux Commands - Complete Guide

## What is Linux Terminal?

The **Linux Terminal** (also called shell or command line) is a text-based interface to interact with your Linux system. The most common shell is **Bash** (Bourne Again Shell).

## How to Open Terminal

1. **Ctrl + Alt + T** (Most Linux distributions)
2. **Search** → Type "Terminal"
3. **Right-click on desktop** → "Open Terminal Here"
4. **Applications Menu** → System Tools → Terminal

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
11. **Shell Scripting** - Automation scripts
12. **Rarely Used but Powerful Commands** - Advanced features

---

## Most Used Daily Commands (Quick Reference)

These are the commands you'll use almost every day:

```bash
# Navigation
cd folder_name              # Change directory
cd ..                       # Go up one level
cd ~                        # Go to home directory
pwd                         # Print working directory
ls                          # List files and folders
ls -la                      # List all files with details
clear                       # Clear screen

# File Operations
cat file.txt                # View file content
nano file.txt               # Edit file with nano
vim file.txt                # Edit file with vim
cp file.txt backup.txt      # Copy file
mv file.txt folder/         # Move file
rm file.txt                 # Delete file
touch file.txt              # Create empty file

# Folder Operations
mkdir foldername            # Create folder
mkdir -p parent/child       # Create nested folders
rm -r foldername            # Remove folder with contents
rmdir foldername            # Remove empty folder

# File Permissions
chmod +x script.sh          # Make file executable
chmod 755 file.txt          # Set permissions
sudo command                # Run as administrator

# System & Network
ping google.com             # Test internet connection
ifconfig                    # Check network config (or ip a)
python3 app.py              # Run Python script
exit                        # Close terminal
history                     # Show command history
```

---

## Essential Daily Commands (Detailed)

### 1. Navigation and Directory Commands

```bash
# Show current directory
pwd

# Change directory
cd /path/to/directory
cd ~/Desktop
cd /home/username/projects

# Go to home directory
cd
cd ~

# Go up one level
cd ..

# Go up two levels
cd ../..

# Go to previous directory
cd -

# Go to root directory
cd /

# List files and folders
ls

# List with details (permissions, size, date)
ls -l

# List all files including hidden
ls -a

# List all with details
ls -la

# List with human-readable file sizes
ls -lh

# List sorted by modification time
ls -lt

# List sorted by size
ls -lS

# List recursively
ls -R
```

### 2. File Viewing and Editing

```bash
# View entire file
cat file.txt

# View file with pagination
less file.txt
more file.txt

# View first 10 lines
head file.txt

# View first 20 lines
head -n 20 file.txt

# View last 10 lines
tail file.txt

# View last 20 lines
tail -n 20 file.txt

# Follow file in real-time (like logs)
tail -f log.txt

# Edit file with nano (beginner-friendly)
nano file.txt

# Edit file with vim
vim file.txt

# Edit file with vi
vi file.txt

# Create or update file timestamp
touch file.txt

# Count lines, words, characters
wc file.txt
wc -l file.txt    # Just lines
wc -w file.txt    # Just words
```

### 3. File Operations

```bash
# Copy file
cp source.txt destination.txt

# Copy file to directory
cp file.txt /home/user/backup/

# Copy directory recursively
cp -r source_folder/ destination_folder/

# Copy with progress (verbose)
cp -v file.txt backup.txt

# Copy and preserve permissions
cp -p file.txt backup.txt

# Move or rename file
mv oldname.txt newname.txt

# Move file to directory
mv file.txt /home/user/documents/

# Move multiple files
mv file1.txt file2.txt file3.txt /destination/

# Delete file
rm file.txt

# Delete without confirmation
rm -f file.txt

# Delete directory and contents
rm -r foldername

# Delete directory without confirmation
rm -rf foldername

# Interactive delete (asks confirmation)
rm -i file.txt

# Find and delete files
find . -name "*.tmp" -delete
```

### 4. Folder Operations

```bash
# Create directory
mkdir foldername

# Create nested directories
mkdir -p parent/child/grandchild

# Create multiple directories
mkdir folder1 folder2 folder3

# Remove empty directory
rmdir foldername

# Remove directory with contents
rm -r foldername

# Remove directory without confirmation
rm -rf foldername

# Copy directory
cp -r source/ destination/

# Move directory
mv oldfolder/ newfolder/
```

### 5. File Permissions and Ownership

```bash
# View permissions
ls -l file.txt

# Make file executable
chmod +x script.sh

# Remove execute permission
chmod -x script.sh

# Set specific permissions (rwx)
chmod 755 file.txt    # rwxr-xr-x
chmod 644 file.txt    # rw-r--r--
chmod 777 file.txt    # rwxrwxrwx (not recommended!)

# Recursive permission change
chmod -R 755 folder/

# Change owner
sudo chown user file.txt

# Change owner and group
sudo chown user:group file.txt

# Recursive ownership change
sudo chown -R user:group folder/

# Change only group
sudo chgrp group file.txt
```

### 6. Search and Find

```bash
# Find files by name
find . -name "file.txt"

# Find files (case-insensitive)
find . -iname "file.txt"

# Find all .txt files
find . -name "*.txt"

# Find in specific directory
find /home/user -name "*.log"

# Find and delete
find . -name "*.tmp" -delete

# Find files modified in last 7 days
find . -mtime -7

# Find files larger than 100MB
find . -size +100M

# Find files by type (f=file, d=directory)
find . -type f
find . -type d

# Search text in files
grep "search term" file.txt

# Search recursively in all files
grep -r "search term" .

# Search case-insensitive
grep -i "search term" file.txt

# Search with line numbers
grep -n "search term" file.txt

# Search and show context
grep -C 3 "search term" file.txt

# Search multiple patterns
grep -E "pattern1|pattern2" file.txt

# Locate files (faster than find)
locate filename
```

### 7. System Information

```bash
# Show current user
whoami

# Show hostname
hostname

# Show system information
uname -a

# Show OS information
cat /etc/os-release

# Show disk usage
df -h

# Show directory size
du -sh folder/

# Show disk usage of current directory
du -h --max-depth=1

# Show memory usage
free -h

# Show running processes
ps aux

# Show processes in tree view
ps auxf

# Interactive process viewer
top

# Better process viewer
htop

# Show system uptime
uptime

# Show logged in users
who
w

# Show last logins
last

# Check CPU info
lscpu

# Check memory info
cat /proc/meminfo
```

### 8. Network Commands

```bash
# Show IP address
ifconfig
ip addr show
ip a

# Ping a host
ping google.com

# Ping with count
ping -c 4 google.com

# Trace route
traceroute google.com

# DNS lookup
nslookup google.com
dig google.com

# Show network statistics
netstat -tuln

# Show all connections
netstat -a

# Show listening ports
netstat -tuln | grep LISTEN

# Download file
wget https://example.com/file.zip

# Download with custom name
wget -O newname.zip https://example.com/file.zip

# Download with curl
curl -O https://example.com/file.zip

# Test URL
curl https://example.com

# Show active connections
ss -tuln

# Check if port is open
telnet hostname port
nc -zv hostname port
```

### 9. Process Management

```bash
# List all processes
ps aux

# Find specific process
ps aux | grep python

# Kill process by PID
kill 1234

# Force kill process
kill -9 1234

# Kill process by name
killall python3
pkill python3

# Run process in background
python3 app.py &

# Bring background process to foreground
fg

# List background jobs
jobs

# Run command and detach
nohup python3 app.py &

# Show real-time processes
top

# Better process monitor
htop

# Check if process is running
pgrep python
pidof python3
```

---

## Docker Commands (If You Use Docker Daily)

```bash
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
docker run -v $(pwd):/app app-name

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

# Copy files from container
docker cp container_name:/path/to/file ./local/path

# Copy files to container
docker cp ./local/file container_name:/path/to/destination

# Clean up unused resources
docker system prune

# Clean everything
docker system prune -a --volumes

# Docker compose
docker-compose up -d
docker-compose down
docker-compose logs -f
docker-compose ps
```

---

## Git Commands (Daily Version Control)

```bash
# Initialize repository
git init

# Clone repository
git clone https://github.com/user/repo.git

# Check status
git status

# Add files to staging
git add file.txt
git add .              # Add all files
git add *.py           # Add all Python files

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

# Add remote repository
git remote add origin https://github.com/user/repo.git
```

---

## Keyboard Shortcuts (Very Useful!)

| Shortcut | Description |
|----------|-------------|
| `Ctrl + C` | Cancel/Stop current command |
| `Ctrl + D` | Exit terminal / End of input |
| `Ctrl + L` | Clear screen (same as `clear`) |
| `Ctrl + A` | Move to beginning of line |
| `Ctrl + E` | Move to end of line |
| `Ctrl + U` | Delete from cursor to beginning |
| `Ctrl + K` | Delete from cursor to end |
| `Ctrl + W` | Delete word before cursor |
| `Ctrl + R` | Search command history |
| `Tab` | Auto-complete file/command names |
| `Tab Tab` | Show all possible completions |
| `Up/Down Arrow` | Navigate command history |
| `Ctrl + Z` | Suspend current process |
| `Ctrl + Shift + C` | Copy in terminal |
| `Ctrl + Shift + V` | Paste in terminal |
| `!!` | Repeat last command |
| `!$` | Last argument of previous command |

---

## Common Daily Workflows

### Workflow 1: Start Working on a Project
```bash
cd ~/projects/myapp
ls -la
git status
git pull origin main
python3 app.py
```

### Workflow 2: Create and Setup New Project
```bash
mkdir ~/projects/newproject
cd ~/projects/newproject
git init
touch README.md
echo "# New Project" > README.md
mkdir src tests
touch src/main.py
ls -la
```

### Workflow 3: Check System Resources
```bash
df -h                    # Check disk space
free -h                  # Check memory
top                      # Check CPU/processes
ps aux | grep python     # Find Python processes
```

### Workflow 4: File Management
```bash
cd ~/Downloads
ls -lh
mv *.pdf ~/Documents/PDFs/
rm -rf temp_folder/
find . -name "*.tmp" -delete
du -sh *
```

### Workflow 5: Server/Log Monitoring
```bash
cd /var/log
sudo tail -f /var/log/syslog
sudo tail -f application.log
grep ERROR application.log
```

### Workflow 6: Quick Backup
```bash
tar -czf backup-$(date +%Y%m%d).tar.gz /path/to/important/folder
cp -r ~/projects ~/backups/projects-$(date +%Y%m%d)
```

---

## Package Management (Installing Software)

### Ubuntu/Debian (apt)

```bash
# Update package list
sudo apt update

# Upgrade all packages
sudo apt upgrade

# Install package
sudo apt install package-name

# Install multiple packages
sudo apt install python3 git vim

# Remove package
sudo apt remove package-name

# Remove package and config files
sudo apt purge package-name

# Remove unused dependencies
sudo apt autoremove

# Search for package
apt search package-name

# Show package information
apt show package-name

# List installed packages
apt list --installed
```

### Red Hat/CentOS/Fedora (yum/dnf)

```bash
# Update packages
sudo yum update
sudo dnf update

# Install package
sudo yum install package-name
sudo dnf install package-name

# Remove package
sudo yum remove package-name
sudo dnf remove package-name

# Search package
yum search package-name
dnf search package-name

# List installed
yum list installed
dnf list installed
```

### Arch Linux (pacman)

```bash
# Update system
sudo pacman -Syu

# Install package
sudo pacman -S package-name

# Remove package
sudo pacman -R package-name

# Remove package and dependencies
sudo pacman -Rs package-name

# Search package
pacman -Ss package-name

# List installed
pacman -Q
```

---

## Advanced Commands (Use When Needed)

### Text Processing

```bash
# Sort lines
sort file.txt

# Sort and remove duplicates
sort -u file.txt

# Sort numerically
sort -n numbers.txt

# Remove duplicate lines
uniq file.txt

# Count occurrences
uniq -c file.txt

# Cut columns (delimiter: space)
cut -d' ' -f1 file.txt

# Cut columns (delimiter: comma)
cut -d',' -f1,3 file.csv

# Replace text in file
sed 's/old/new/g' file.txt

# Replace and save
sed -i 's/old/new/g' file.txt

# Print specific lines
sed -n '10,20p' file.txt

# AWK - print columns
awk '{print $1}' file.txt

# AWK - sum column
awk '{sum+=$1} END {print sum}' file.txt

# Print line numbers
nl file.txt

# Paste files side by side
paste file1.txt file2.txt

# Join lines
tr '\n' ' ' < file.txt
```

### Compression and Archives

```bash
# Create tar archive
tar -cvf archive.tar folder/

# Create compressed tar.gz
tar -czf archive.tar.gz folder/

# Extract tar archive
tar -xvf archive.tar

# Extract tar.gz
tar -xzf archive.tar.gz

# List contents of tar
tar -tvf archive.tar

# Create zip archive
zip -r archive.zip folder/

# Extract zip
unzip archive.zip

# Extract to specific directory
unzip archive.zip -d /destination/

# List zip contents
unzip -l archive.zip

# Compress file with gzip
gzip file.txt
# Creates file.txt.gz

# Decompress gzip
gunzip file.txt.gz
```

### Disk and File System

```bash
# Mount drive
sudo mount /dev/sdb1 /mnt/usb

# Unmount drive
sudo umount /mnt/usb

# Show mounted filesystems
mount

# Check disk usage by directory
du -sh */

# Find largest files
du -ah . | sort -rh | head -20

# Check inode usage
df -i

# Sync data to disk
sync

# Create symbolic link
ln -s /path/to/original /path/to/link

# Create hard link
ln /path/to/original /path/to/link

# Find broken symbolic links
find . -type l ! -exec test -e {} \; -print
```

---

## Less Commonly Used Commands

### System Administration

```bash
# Reboot system
sudo reboot

# Shutdown system
sudo shutdown -h now

# Shutdown in 10 minutes
sudo shutdown -h +10

# Schedule shutdown
sudo shutdown -h 22:00

# Cancel shutdown
sudo shutdown -c

# Switch to root user
sudo su -
su -

# Run single command as root
sudo command

# Edit sudoers file
sudo visudo

# Add user
sudo adduser username

# Delete user
sudo deluser username

# Change user password
sudo passwd username

# Change own password
passwd

# Show system logs
sudo journalctl

# Follow system logs
sudo journalctl -f

# Show logs for service
sudo journalctl -u servicename
```

### Service Management (systemd)

```bash
# Start service
sudo systemctl start servicename

# Stop service
sudo systemctl stop servicename

# Restart service
sudo systemctl restart servicename

# Enable service on boot
sudo systemctl enable servicename

# Disable service on boot
sudo systemctl disable servicename

# Check service status
sudo systemctl status servicename

# List all services
systemctl list-units --type=service

# Show failed services
systemctl --failed
```

---

## Complete Command Reference (Alphabetical)

### File and Directory Operations Summary

```bash
# Basic Operations
cd              # Change directory
pwd             # Print working directory
ls              # List files
mkdir           # Make directory
rmdir           # Remove empty directory
rm              # Remove files/directories
cp              # Copy files/directories
mv              # Move/rename files
touch           # Create empty file/update timestamp

# Viewing Files
cat             # Display file content
less            # View file with pagination
more            # View file with pagination
head            # Display first lines
tail            # Display last lines
nano            # Text editor (beginner-friendly)
vim             # Text editor (advanced)
vi              # Text editor

# File Information
ls -l           # Long listing
ls -a           # Show hidden files
ls -h           # Human-readable sizes
stat            # Detailed file information
file            # Determine file type

# Permissions
chmod           # Change permissions
chown           # Change ownership
chgrp           # Change group

# Search
find            # Find files
locate          # Find files (faster)
grep            # Search text in files
which           # Locate command
whereis         # Locate binary/source/man page
```

### System Commands Summary

```bash
# System Information
uname           # System information
hostname        # Show hostname
uptime          # System uptime
date            # Show date and time
cal             # Calendar
whoami          # Current user
who             # Logged in users
w               # Who is logged in and what they're doing

# Resource Monitoring
top             # Process viewer
htop            # Better process viewer
ps              # Process status
free            # Memory usage
df              # Disk space
du              # Disk usage
iostat          # I/O statistics
vmstat          # Virtual memory statistics

# Process Control
kill            # Terminate process
killall         # Kill processes by name
pkill           # Kill processes by pattern
bg              # Send to background
fg              # Bring to foreground
jobs            # List background jobs
nohup           # Run immune to hangups
```

### Network Commands Summary

```bash
# Network Information
ifconfig        # Network interface configuration
ip              # Network configuration (modern)
hostname        # Show hostname
netstat         # Network statistics
ss              # Socket statistics (modern)

# Network Testing
ping            # Test connectivity
traceroute      # Trace route to host
mtr             # Network diagnostic tool
nslookup        # DNS lookup
dig             # DNS lookup (detailed)
host            # DNS lookup

# File Transfer
wget            # Download files
curl            # Transfer data with URLs
scp             # Secure copy over SSH
rsync           # Sync files/directories
ftp             # FTP client
sftp            # Secure FTP
```

---

## Shell Scripting (For Automation)

### What are Shell Scripts?

Shell scripts are files containing a series of commands. They end with `.sh` extension and help automate repetitive tasks.

### Basic Script Structure

```bash
#!/bin/bash
# This is a comment

echo "Hello, World!"
```

Make it executable:
```bash
chmod +x script.sh
./script.sh
```

### Script Examples

#### Example 1: Simple Backup Script
**backup.sh**
```bash
#!/bin/bash
# Backup script

SOURCE="/home/user/important"
DEST="/home/user/backups"
DATE=$(date +%Y%m%d_%H%M%S)

echo "Starting backup..."
tar -czf "$DEST/backup_$DATE.tar.gz" "$SOURCE"
echo "Backup completed: backup_$DATE.tar.gz"
```

#### Example 2: System Information Script
**sysinfo.sh**
```bash
#!/bin/bash
# System information script

echo "===== System Information ====="
echo "Hostname: $(hostname)"
echo "OS: $(cat /etc/os-release | grep PRETTY_NAME | cut -d'"' -f2)"
echo "Uptime: $(uptime -p)"
echo "CPU: $(lscpu | grep "Model name" | cut -d':' -f2 | xargs)"
echo ""
echo "===== Disk Usage ====="
df -h | grep -E '^/dev/'
echo ""
echo "===== Memory Usage ====="
free -h
echo ""
echo "===== Top 5 Processes ====="
ps aux --sort=-%mem | head -6
```

#### Example 3: Project Setup Script
**setup_project.sh**
```bash
#!/bin/bash
# Project setup script

PROJECT_NAME=$1

if [ -z "$PROJECT_NAME" ]; then
    echo "Usage: ./setup_project.sh <project_name>"
    exit 1
fi

echo "Creating project: $PROJECT_NAME"
mkdir -p "$PROJECT_NAME"/{src,tests,docs,config}
cd "$PROJECT_NAME"

echo "# $PROJECT_NAME" > README.md
touch src/main.py
touch tests/test_main.py
touch .gitignore

cat > .gitignore << EOF
__pycache__/
*.pyc
.env
venv/
EOF

git init
echo "Project $PROJECT_NAME created successfully!"
```

#### Example 4: Log Cleaner Script
**clean_logs.sh**
```bash
#!/bin/bash
# Clean old log files

LOG_DIR="/var/log/myapp"
DAYS=7

echo "Cleaning logs older than $DAYS days in $LOG_DIR"
find "$LOG_DIR" -name "*.log" -type f -mtime +$DAYS -delete
echo "Cleanup complete!"
```

#### Example 5: Service Monitor Script
**monitor.sh**
```bash
#!/bin/bash
# Monitor service and restart if down

SERVICE="nginx"

if ! systemctl is-active --quiet $SERVICE; then
    echo "$(date): $SERVICE is down. Restarting..." >> /var/log/monitor.log
    sudo systemctl restart $SERVICE
else
    echo "$(date): $SERVICE is running" >> /var/log/monitor.log
fi
```

### Script Variables and Conditionals

```bash
#!/bin/bash

# Variables
NAME="John"
AGE=25

# User input
read -p "Enter your name: " USERNAME
echo "Hello, $USERNAME!"

# If statement
if [ $AGE -gt 18 ]; then
    echo "Adult"
else
    echo "Minor"
fi

# Check if file exists
if [ -f "file.txt" ]; then
    echo "File exists"
fi

# Check if directory exists
if [ -d "folder" ]; then
    echo "Directory exists"
fi

# For loop
for i in {1..5}; do
    echo "Number: $i"
done

# While loop
COUNT=0
while [ $COUNT -lt 5 ]; do
    echo "Count: $COUNT"
    COUNT=$((COUNT + 1))
done

# Case statement
case $1 in
    start)
        echo "Starting..."
        ;;
    stop)
        echo "Stopping..."
        ;;
    *)
        echo "Usage: $0 {start|stop}"
        ;;
esac
```

---

## Rarely Used but Powerful Commands

### Advanced System Commands

```bash
# Create system restore point (if supported)
sudo timeshift --create

# Monitor system calls
strace command

# List open files
lsof

# List open files by process
lsof -p PID

# Find which process uses port
lsof -i :8080

# Show file access
watch -n 1 'ls -l file.txt'

# Execute command at specific time
at 14:00
at> command

# Schedule recurring tasks
crontab -e

# Example cron job (run daily at 2am)
# 0 2 * * * /path/to/backup.sh

# Compare files
diff file1.txt file2.txt

# Compare directories
diff -r dir1/ dir2/

# Patch files
patch < patchfile

# Calculate MD5 checksum
md5sum file.txt

# Calculate SHA256 checksum
sha256sum file.txt

# Create bootable USB
sudo dd if=image.iso of=/dev/sdb bs=4M status=progress

# Benchmark disk speed
sudo hdparm -tT /dev/sda

# Monitor disk I/O
sudo iotop
```

### Network Advanced

```bash
# Capture network packets
sudo tcpdump -i eth0

# Network packet analyzer
sudo wireshark

# Port scanning
nmap hostname

# SSH tunneling
ssh -L 8080:localhost:80 user@remote

# Transfer files with rsync
rsync -avz source/ user@remote:/destination/

# Create VPN connection
sudo openvpn config.ovpn

# Show routing table
route -n
ip route show

# Add static route
sudo ip route add 192.168.1.0/24 via 192.168.0.1
```

---

## Command Chaining and Redirection

```bash
# Run commands sequentially
command1 ; command2

# Run second only if first succeeds
command1 && command2

# Run second only if first fails
command1 || command2

# Pipe output to another command
ls -l | grep ".txt"

# Redirect output to file (overwrite)
echo "Hello" > file.txt

# Redirect output to file (append)
echo "World" >> file.txt

# Redirect error output
command 2> error.log

# Redirect both output and errors
command > output.log 2>&1

# Redirect output to nowhere
command > /dev/null

# Redirect input from file
command < input.txt

# Here document
cat << EOF > file.txt
Line 1
Line 2
EOF

# Tee (write to file and stdout)
command | tee output.txt

# Multiple pipes
cat file.txt | grep "error" | sort | uniq
```

---

## Useful Aliases (Add to ~/.bashrc)

```bash
# Navigation
alias ..='cd ..'
alias ...='cd ../..'
alias ll='ls -alh'
alias la='ls -A'
alias l='ls -CF'

# Git shortcuts
alias gs='git status'
alias ga='git add'
alias gc='git commit -m'
alias gp='git push'
alias gl='git log --oneline'

# Docker shortcuts
alias dps='docker ps'
alias dpa='docker ps -a'
alias di='docker images'
alias dex='docker exec -it'
alias dl='docker logs -f'

# System shortcuts
alias update='sudo apt update && sudo apt upgrade'
alias install='sudo apt install'
alias remove='sudo apt remove'
alias search='apt search'

# Safety
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'

# Utilities
alias h='history'
alias j='jobs'
alias c='clear'
alias ports='netstat -tuln'
alias myip='curl ifconfig.me'
```

After adding aliases:
```bash
source ~/.bashrc
```

---

## Common Error Solutions

### Error: Permission Denied
**Solution:** Use sudo or check file permissions
```bash
sudo command
chmod +x script.sh
```

### Error: Command Not Found
**Solution:** Install the package or check PATH
```bash
which command
echo $PATH
sudo apt install package-name
```

### Error: No Space Left on Device
**Solution:** Free up disk space
```bash
df -h
du -sh /* | sort -rh | head -10
sudo apt autoremove
sudo apt clean
```

### Error: Port Already in Use
**Solution:** Find and kill the process
```bash
sudo lsof -i :8080
sudo kill -9 PID
```

### Error: SSH Connection Refused
**Solution:** Check SSH service
```bash
sudo systemctl status ssh
sudo systemctl start ssh
```

---

## Tips and Best Practices

1. **Always backup before making system changes**
2. **Use Tab for auto-completion** - saves time and prevents typos
3. **Use Ctrl+R to search history** - faster than typing commands again
4. **Test commands in safe directory first**
5. **Use `man command` to read manual** - e.g., `man ls`
6. **Check command help with `--help`** - e.g., `ls --help`
7. **Use absolute paths for scripts** - more reliable
8. **Comment your shell scripts** - helps future you
9. **Never run `rm -rf /` or `chmod -R 777 /`** - will destroy your system
10. **Learn vim or nano basics** - essential for editing on servers

---

## Getting Help

```bash
# View command manual
man command

# Short help
command --help

# Search in manual pages
man -k keyword

# Info pages (more detailed)
info command

# Which manual page
whatis command

# Where is command located
which command
whereis command

# Describe command
type command
```

---

## Quick Daily Commands Summary

```bash
# Navigation
cd, pwd, ls, cd .., cd ~

# File operations
cat, nano, cp, mv, rm, touch

# Directory operations
mkdir, rm -r, ls -la

# Search
find, grep, locate

# System
df -h, free -h, top, ps aux

# Network
ping, ifconfig, ip a

# Processes
ps aux, kill, killall

# Package management
sudo apt update, sudo apt install

# Permissions
chmod, chown, sudo

# Help
man, --help, which
```

---

**Happy Linux Command Line Usage! 🐧**
