# Bash Scripting: Zero to Hero

A comprehensive guide to mastering Bash scripting from absolute beginner to advanced user.

---

## Table of Contents

1. [Introduction to Bash](#introduction-to-bash)
2. [Getting Started](#getting-started)
3. [Basic Concepts](#basic-concepts)
4. [Variables](#variables)
5. [Input and Output](#input-and-output)
6. [Operators](#operators)
7. [Conditional Statements](#conditional-statements)
8. [Loops](#loops)
9. [Functions](#functions)
10. [Arrays](#arrays)
11. [String Manipulation](#string-manipulation)
12. [File Operations](#file-operations)
13. [Process Management](#process-management)
14. [Advanced Topics](#advanced-topics)
15. [Best Practices](#best-practices)
16. [Common Patterns and Examples](#common-patterns-and-examples)

---

## Introduction to Bash

### What is Bash?

Bash (Bourne Again SHell) is a command-line interpreter and scripting language for Unix-based systems. It's the default shell on most Linux distributions and macOS.

### Why Learn Bash?

- Automate repetitive tasks
- System administration and maintenance
- Process management and monitoring
- File manipulation and data processing
- DevOps and CI/CD pipelines
- Quick prototyping and testing

---

## Getting Started

### Your First Script

Create a file called `hello.sh`:

```bash
#!/bin/bash
# This is a comment
echo "Hello, World!"
```

### Making Scripts Executable

```bash
chmod +x hello.sh
./hello.sh
```

### The Shebang Line

The `#!/bin/bash` at the top tells the system which interpreter to use:

- `#!/bin/bash` - Standard Bash
- `#!/usr/bin/env bash` - More portable, finds bash in PATH
- `#!/bin/sh` - POSIX-compatible shell

---

## Basic Concepts

### Running Commands

```bash
#!/bin/bash

# Run a command
ls -la

# Command substitution
current_date=$(date)
echo "Today is: $current_date"

# Alternative syntax
current_user=`whoami`
echo "Current user: $current_user"
```

### Exit Codes

Every command returns an exit code (0 = success, non-zero = error):

```bash
#!/bin/bash

ls /nonexistent
echo "Exit code: $?"  # $? contains the last exit code

# Explicitly set exit code
exit 0  # Success
exit 1  # General error
```

---

## Variables

### Declaring Variables

```bash
#!/bin/bash

# Simple assignment (no spaces around =)
name="John"
age=30
is_active=true

# Using variables
echo "Name: $name"
echo "Age: ${age}"  # Curly braces are optional but safer
```

### Variable Types

```bash
#!/bin/bash

# String variables
greeting="Hello"
message='Single quotes preserve literal values'

# Numeric variables
count=10
price=19.99

# Environment variables (uppercase by convention)
export PATH="/usr/local/bin:$PATH"
export DATABASE_URL="postgresql://localhost/mydb"

# Read-only variables
readonly MAX_USERS=100
# MAX_USERS=200  # This would cause an error
```

### Special Variables

```bash
#!/bin/bash

echo "Script name: $0"
echo "First argument: $1"
echo "Second argument: $2"
echo "All arguments: $@"
echo "Number of arguments: $#"
echo "Process ID: $$"
echo "Last background process: $!"
echo "Exit code of last command: $?"
```

### Variable Scope

```bash
#!/bin/bash

# Global variable
global_var="I'm global"

function my_function() {
    # Local variable
    local local_var="I'm local"
    echo "Inside function: $local_var"
    echo "Inside function: $global_var"
}

my_function
echo "Outside function: $global_var"
# echo "Outside function: $local_var"  # This would be empty
```

---

## Input and Output

### Reading User Input

```bash
#!/bin/bash

# Simple read
echo "What's your name?"
read name
echo "Hello, $name!"

# Read with prompt
read -p "Enter your age: " age
echo "You are $age years old"

# Read password (hidden input)
read -sp "Enter password: " password
echo
echo "Password saved"

# Read multiple variables
read -p "Enter first and last name: " first last
echo "First: $first, Last: $last"

# Read with timeout (5 seconds)
if read -t 5 -p "Quick! Enter something: " response; then
    echo "You entered: $response"
else
    echo "Too slow!"
fi
```

### Output Formatting

```bash
#!/bin/bash

# Basic echo
echo "Simple text"

# Echo without newline
echo -n "No newline"
echo " continues here"

# Echo with escape sequences
echo -e "Line 1\nLine 2\tTabbed"

# Printf for formatted output
printf "Name: %-10s Age: %3d\n" "John" 25
printf "%.2f\n" 3.14159
```

### Redirections

```bash
#!/bin/bash

# Redirect stdout to file (overwrite)
echo "Hello" > output.txt

# Redirect stdout to file (append)
echo "World" >> output.txt

# Redirect stderr to file
ls /nonexistent 2> error.log

# Redirect both stdout and stderr
command > output.log 2>&1

# Modern syntax for redirecting both
command &> output.log

# Redirect to /dev/null (discard output)
command > /dev/null 2>&1

# Here document
cat << EOF
This is a multi-line
text block that will be
passed to the cat command
EOF

# Here string
grep "pattern" <<< "search in this string"
```

---

## Operators

### Arithmetic Operators

```bash
#!/bin/bash

# Using (( )) for arithmetic
a=10
b=5

((sum = a + b))
((diff = a - b))
((product = a * b))
((quotient = a / b))
((remainder = a % b))

echo "Sum: $sum"
echo "Difference: $diff"
echo "Product: $product"
echo "Quotient: $quotient"
echo "Remainder: $remainder"

# Using let
let result=10+5
echo "Result: $result"

# Using expr (older method)
result=$(expr 10 + 5)
echo "Result: $result"

# Increment and decrement
count=10
((count++))
echo "After increment: $count"
((count--))
echo "After decrement: $count"

# Compound operators
value=5
((value += 3))  # value = value + 3
((value *= 2))  # value = value * 2
echo "Value: $value"
```

### Comparison Operators

```bash
#!/bin/bash

# Numeric comparisons (use (( )) or [ ])
a=10
b=20

# Using [[ ]] (recommended)
if [[ $a -eq $b ]]; then echo "Equal"; fi
if [[ $a -ne $b ]]; then echo "Not equal"; fi
if [[ $a -lt $b ]]; then echo "$a less than $b"; fi
if [[ $a -le $b ]]; then echo "$a less or equal $b"; fi
if [[ $a -gt $b ]]; then echo "$a greater than $b"; fi
if [[ $a -ge $b ]]; then echo "$a greater or equal $b"; fi

# Using (( )) for arithmetic (more intuitive)
if ((a == b)); then echo "Equal"; fi
if ((a != b)); then echo "Not equal"; fi
if ((a < b)); then echo "$a less than $b"; fi
if ((a <= b)); then echo "$a less or equal $b"; fi
if ((a > b)); then echo "$a greater than $b"; fi
if ((a >= b)); then echo "$a greater or equal $b"; fi
```

### String Operators

```bash
#!/bin/bash

str1="hello"
str2="world"
empty=""

# String comparisons
[[ $str1 == $str2 ]] && echo "Equal" || echo "Not equal"
[[ $str1 != $str2 ]] && echo "Not equal"
[[ $str1 < $str2 ]] && echo "$str1 comes before $str2"
[[ $str1 > $str2 ]] && echo "$str1 comes after $str2"

# String tests
[[ -z $empty ]] && echo "String is empty"
[[ -n $str1 ]] && echo "String is not empty"

# Pattern matching
[[ $str1 == h* ]] && echo "Starts with h"
[[ $str1 == *lo ]] && echo "Ends with lo"
[[ $str1 =~ ^h.* ]] && echo "Regex match"
```

### Logical Operators

```bash
#!/bin/bash

a=10
b=20

# AND operator
if [[ $a -lt 15 && $b -gt 15 ]]; then
    echo "Both conditions are true"
fi

# OR operator
if [[ $a -lt 5 || $b -gt 15 ]]; then
    echo "At least one condition is true"
fi

# NOT operator
if [[ ! $a -eq $b ]]; then
    echo "Not equal"
fi

# Command chaining
command1 && command2  # Run command2 only if command1 succeeds
command1 || command2  # Run command2 only if command1 fails
command1 ; command2   # Run both regardless of exit status
```

### File Test Operators

```bash
#!/bin/bash

file="test.txt"

# File existence and type
[[ -e $file ]] && echo "File exists"
[[ -f $file ]] && echo "Is a regular file"
[[ -d $file ]] && echo "Is a directory"
[[ -L $file ]] && echo "Is a symbolic link"
[[ -p $file ]] && echo "Is a named pipe"
[[ -S $file ]] && echo "Is a socket"

# File permissions
[[ -r $file ]] && echo "File is readable"
[[ -w $file ]] && echo "File is writable"
[[ -x $file ]] && echo "File is executable"

# File properties
[[ -s $file ]] && echo "File is not empty"
[[ -h $file ]] && echo "File is a symbolic link"

# File comparison
file1="file1.txt"
file2="file2.txt"
[[ $file1 -nt $file2 ]] && echo "file1 is newer than file2"
[[ $file1 -ot $file2 ]] && echo "file1 is older than file2"
```

---

## Conditional Statements

### If-Else Statements

```bash
#!/bin/bash

# Simple if
age=18
if [[ $age -ge 18 ]]; then
    echo "You are an adult"
fi

# If-else
score=75
if [[ $score -ge 60 ]]; then
    echo "Pass"
else
    echo "Fail"
fi

# If-elif-else
score=85
if [[ $score -ge 90 ]]; then
    echo "Grade: A"
elif [[ $score -ge 80 ]]; then
    echo "Grade: B"
elif [[ $score -ge 70 ]]; then
    echo "Grade: C"
elif [[ $score -ge 60 ]]; then
    echo "Grade: D"
else
    echo "Grade: F"
fi

# Nested if
num=15
if [[ $num -gt 10 ]]; then
    if [[ $num -lt 20 ]]; then
        echo "Number is between 10 and 20"
    fi
fi

# One-liner if
[[ $age -ge 18 ]] && echo "Adult" || echo "Minor"
```

### Case Statements

```bash
#!/bin/bash

# Basic case statement
read -p "Enter a fruit: " fruit

case $fruit in
    apple)
        echo "Apple is red"
        ;;
    banana)
        echo "Banana is yellow"
        ;;
    orange)
        echo "Orange is orange"
        ;;
    *)
        echo "Unknown fruit"
        ;;
esac

# Multiple patterns
read -p "Enter a character: " char

case $char in
    [a-z])
        echo "Lowercase letter"
        ;;
    [A-Z])
        echo "Uppercase letter"
        ;;
    [0-9])
        echo "Digit"
        ;;
    *)
        echo "Special character"
        ;;
esac

# Multiple values per pattern
read -p "Enter yes or no: " answer

case $answer in
    yes|y|YES|Y)
        echo "You said yes"
        ;;
    no|n|NO|N)
        echo "You said no"
        ;;
    *)
        echo "Invalid input"
        ;;
esac
```

---

## Loops

### For Loops

```bash
#!/bin/bash

# Loop over a list
for color in red green blue; do
    echo "Color: $color"
done

# Loop over command output
for file in $(ls); do
    echo "File: $file"
done

# C-style for loop
for ((i=1; i<=5; i++)); do
    echo "Number: $i"
done

# Loop over array
fruits=("apple" "banana" "orange")
for fruit in "${fruits[@]}"; do
    echo "Fruit: $fruit"
done

# Loop over range (using brace expansion)
for i in {1..10}; do
    echo "Count: $i"
done

# Loop with step
for i in {0..20..2}; do
    echo "Even number: $i"
done

# Loop over files with glob pattern
for file in *.txt; do
    echo "Processing: $file"
done
```

### While Loops

```bash
#!/bin/bash

# Basic while loop
count=1
while [[ $count -le 5 ]]; do
    echo "Count: $count"
    ((count++))
done

# Reading file line by line
while IFS= read -r line; do
    echo "Line: $line"
done < input.txt

# Infinite loop
while true; do
    echo "This runs forever"
    sleep 1
    # Use Ctrl+C to exit
done

# While with condition
num=10
while [[ $num -gt 0 ]]; do
    echo "Countdown: $num"
    ((num--))
    sleep 1
done
echo "Liftoff!"
```

### Until Loops

```bash
#!/bin/bash

# Until loop (runs until condition is true)
count=1
until [[ $count -gt 5 ]]; do
    echo "Count: $count"
    ((count++))
done

# Waiting for a condition
until ping -c 1 google.com &> /dev/null; do
    echo "Waiting for network connection..."
    sleep 2
done
echo "Network is up!"
```

### Loop Control

```bash
#!/bin/bash

# Break (exit loop)
for i in {1..10}; do
    if [[ $i -eq 5 ]]; then
        break
    fi
    echo "Number: $i"
done

# Continue (skip iteration)
for i in {1..10}; do
    if [[ $i -eq 5 ]]; then
        continue
    fi
    echo "Number: $i"
done

# Break outer loop
for i in {1..3}; do
    for j in {1..3}; do
        echo "i=$i, j=$j"
        if [[ $i -eq 2 && $j -eq 2 ]]; then
            break 2  # Break out of both loops
        fi
    done
done
```

---

## Functions

### Defining Functions

```bash
#!/bin/bash

# Method 1: Using 'function' keyword
function greet() {
    echo "Hello, World!"
}

# Method 2: Without 'function' keyword (more common)
say_hello() {
    echo "Hello!"
}

# Calling functions
greet
say_hello
```

### Function Arguments

```bash
#!/bin/bash

greet_person() {
    echo "Hello, $1!"
    echo "You are $2 years old"
}

greet_person "Alice" 30

# Function with multiple arguments
calculate_sum() {
    local sum=0
    for num in "$@"; do
        ((sum += num))
    done
    echo "Sum: $sum"
}

calculate_sum 10 20 30 40 50
```

### Return Values

```bash
#!/bin/bash

# Return exit code (0-255)
is_even() {
    if (( $1 % 2 == 0 )); then
        return 0  # Success (true)
    else
        return 1  # Failure (false)
    fi
}

if is_even 10; then
    echo "10 is even"
fi

# Return string using echo and command substitution
get_full_name() {
    local first=$1
    local last=$2
    echo "$first $last"
}

name=$(get_full_name "John" "Doe")
echo "Full name: $name"

# Return multiple values
get_user_info() {
    echo "John"
    echo "30"
    echo "Developer"
}

# Read multiple return values
IFS=$'\n' read -d '' -r name age occupation < <(get_user_info)
echo "Name: $name, Age: $age, Occupation: $occupation"
```

### Function Scope

```bash
#!/bin/bash

global_var="I'm global"

my_function() {
    local local_var="I'm local"
    global_var="Modified global"
    
    echo "Inside function:"
    echo "  Local: $local_var"
    echo "  Global: $global_var"
}

echo "Before function call: $global_var"
my_function
echo "After function call: $global_var"
# echo "Local var: $local_var"  # This would be empty
```

---

## Arrays

### Indexed Arrays

```bash
#!/bin/bash

# Declaration
fruits=("apple" "banana" "orange")

# Alternative declaration
declare -a colors
colors[0]="red"
colors[1]="green"
colors[2]="blue"

# Accessing elements
echo "First fruit: ${fruits[0]}"
echo "Second fruit: ${fruits[1]}"

# All elements
echo "All fruits: ${fruits[@]}"
echo "All fruits: ${fruits[*]}"

# Array length
echo "Number of fruits: ${#fruits[@]}"

# Length of element
echo "Length of first fruit: ${#fruits[0]}"

# Adding elements
fruits+=("grape")
fruits[4]="mango"

# Loop through array
for fruit in "${fruits[@]}"; do
    echo "Fruit: $fruit"
done

# Loop with index
for i in "${!fruits[@]}"; do
    echo "Index $i: ${fruits[$i]}"
done

# Slicing arrays
echo "Slice [1:2]: ${fruits[@]:1:2}"

# Delete element
unset fruits[1]

# Delete entire array
unset fruits
```

### Associative Arrays (Bash 4+)

```bash
#!/bin/bash

# Declaration
declare -A person

# Assignment
person[name]="John"
person[age]=30
person[city]="New York"

# Accessing values
echo "Name: ${person[name]}"
echo "Age: ${person[age]}"

# All keys
echo "Keys: ${!person[@]}"

# All values
echo "Values: ${person[@]}"

# Check if key exists
if [[ -v person[name] ]]; then
    echo "Name exists"
fi

# Loop through associative array
for key in "${!person[@]}"; do
    echo "$key: ${person[$key]}"
done

# Delete key
unset person[age]
```

### Array Operations

```bash
#!/bin/bash

# Sorting array
nums=(5 2 8 1 9 3)
IFS=$'\n' sorted=($(sort -n <<<"${nums[*]}"))
echo "Sorted: ${sorted[@]}"

# Reversing array
reversed=($(printf '%s\n' "${nums[@]}" | tac))
echo "Reversed: ${reversed[@]}"

# Searching array
search="banana"
fruits=("apple" "banana" "orange")
for i in "${!fruits[@]}"; do
    if [[ "${fruits[$i]}" == "$search" ]]; then
        echo "Found $search at index $i"
        break
    fi
done

# Remove duplicates
arr=(1 2 2 3 3 3 4 5 5)
unique=($(printf '%s\n' "${arr[@]}" | sort -u))
echo "Unique: ${unique[@]}"

# Merging arrays
arr1=(1 2 3)
arr2=(4 5 6)
merged=("${arr1[@]}" "${arr2[@]}")
echo "Merged: ${merged[@]}"
```

---

## String Manipulation

### String Length

```bash
#!/bin/bash

str="Hello, World!"
echo "Length: ${#str}"
```

### Substring Extraction

```bash
#!/bin/bash

str="Hello, World!"

# Extract substring (position:length)
echo "${str:0:5}"    # Hello
echo "${str:7:5}"    # World
echo "${str:7}"      # World! (from position to end)
echo "${str: -6}"    # World! (last 6 characters)
echo "${str: -6:5}"  # World (last 6, take 5)
```

### String Replacement

```bash
#!/bin/bash

str="Hello World World"

# Replace first occurrence
echo "${str/World/Universe}"  # Hello Universe World

# Replace all occurrences
echo "${str//World/Universe}"  # Hello Universe Universe

# Replace at beginning
echo "${str/#Hello/Hi}"  # Hi World World

# Replace at end
echo "${str/%World/Universe}"  # Hello World Universe

# Delete pattern
echo "${str//World/}"  # Hello  
```

### Case Conversion

```bash
#!/bin/bash

str="Hello World"

# To uppercase
echo "${str^^}"  # HELLO WORLD

# To lowercase
echo "${str,,}"  # hello world

# First character uppercase
echo "${str^}"   # Hello world

# First character lowercase
echo "${str,}"   # hello World
```

### String Trimming

```bash
#!/bin/bash

str="   Hello World   "

# Trim leading/trailing whitespace
trimmed=$(echo "$str" | xargs)
echo "Trimmed: '$trimmed'"

# Remove prefix
path="/home/user/documents/file.txt"
echo "${path#/home/user/}"  # documents/file.txt

# Remove longest prefix
echo "${path##*/}"  # file.txt

# Remove suffix
filename="document.txt"
echo "${filename%.txt}"  # document

# Remove longest suffix
path="/home/user/documents/file.txt"
echo "${path%%/*}"  # (empty - removes everything)
```

### String Splitting

```bash
#!/bin/bash

# Split string into array
str="apple,banana,orange"
IFS=',' read -ra fruits <<< "$str"
for fruit in "${fruits[@]}"; do
    echo "Fruit: $fruit"
done

# Split by multiple delimiters
str="apple:banana;orange,grape"
IFS=':;,' read -ra fruits <<< "$str"
```

### String Comparison

```bash
#!/bin/bash

str1="hello"
str2="world"

# Equality
[[ $str1 == $str2 ]] && echo "Equal" || echo "Not equal"

# Inequality
[[ $str1 != $str2 ]] && echo "Not equal"

# Lexicographic comparison
[[ $str1 < $str2 ]] && echo "$str1 comes before $str2"

# Contains substring
[[ $str1 == *ell* ]] && echo "Contains 'ell'"

# Starts with
[[ $str1 == hel* ]] && echo "Starts with 'hel'"

# Ends with
[[ $str1 == *lo ]] && echo "Ends with 'lo'"

# Regex matching
[[ $str1 =~ ^h.* ]] && echo "Matches regex"
```

---

## File Operations

### Reading Files

```bash
#!/bin/bash

# Read entire file
content=$(<filename.txt)
echo "$content"

# Read file line by line
while IFS= read -r line; do
    echo "Line: $line"
done < filename.txt

# Read with line numbers
line_num=1
while IFS= read -r line; do
    echo "$line_num: $line"
    ((line_num++))
done < filename.txt

# Read specific lines
sed -n '5,10p' filename.txt  # Lines 5-10
head -n 5 filename.txt       # First 5 lines
tail -n 5 filename.txt       # Last 5 lines
```

### Writing Files

```bash
#!/bin/bash

# Write to file (overwrite)
echo "Hello, World!" > output.txt

# Append to file
echo "Another line" >> output.txt

# Write multiple lines
cat > output.txt << EOF
Line 1
Line 2
Line 3
EOF

# Write with printf
printf "Name: %s\nAge: %d\n" "John" 30 > output.txt
```

### File Testing

```bash
#!/bin/bash

file="test.txt"

if [[ -f $file ]]; then
    echo "File exists"
    
    if [[ -r $file ]]; then
        echo "File is readable"
    fi
    
    if [[ -w $file ]]; then
        echo "File is writable"
    fi
    
    if [[ -x $file ]]; then
        echo "File is executable"
    fi
    
    if [[ -s $file ]]; then
        echo "File is not empty"
    fi
fi
```

### File Operations

```bash
#!/bin/bash

# Copy file
cp source.txt destination.txt

# Move/Rename file
mv oldname.txt newname.txt

# Delete file
rm filename.txt

# Create directory
mkdir -p path/to/directory

# Delete directory
rm -rf directory_name

# Find files
find . -name "*.txt"
find . -type f -mtime -7  # Modified in last 7 days
find . -type f -size +1M  # Larger than 1MB

# File permissions
chmod 755 script.sh
chmod u+x script.sh
chown user:group file.txt
```

### Working with CSV Files

```bash
#!/bin/bash

# Read CSV file
while IFS=',' read -r col1 col2 col3; do
    echo "Column 1: $col1, Column 2: $col2, Column 3: $col3"
done < data.csv

# Write CSV file
echo "Name,Age,City" > output.csv
echo "John,30,New York" >> output.csv
echo "Jane,25,London" >> output.csv
```

---

## Process Management

### Running Commands

```bash
#!/bin/bash

# Run command in foreground
command

# Run command in background
command &

# Get PID of last background process
echo "PID: $!"

# Wait for background process to complete
command &
pid=$!
wait $pid
echo "Process completed"
```

### Process Control

```bash
#!/bin/bash

# List running processes
ps aux
ps -ef

# Find specific process
ps aux | grep "process_name"
pgrep "process_name"

# Kill process
kill PID
kill -9 PID  # Force kill
killall process_name

# Process priority
nice -n 10 command      # Run with lower priority
renice -n 5 -p PID      # Change priority of running process
```

### Job Control

```bash
#!/bin/bash

# Background/Foreground jobs
command &               # Start in background
jobs                    # List jobs
fg %1                   # Bring job 1 to foreground
bg %1                   # Resume job 1 in background
Ctrl+Z                  # Suspend current job
kill %1                 # Kill job 1
```

### Parallel Execution

```bash
#!/bin/bash

# Run multiple commands in parallel
command1 &
command2 &
command3 &
wait  # Wait for all background jobs

# Parallel processing of array
urls=("url1" "url2" "url3")
for url in "${urls[@]}"; do
    curl "$url" &
done
wait

# Using xargs for parallel execution
echo -e "file1\nfile2\nfile3" | xargs -P 3 -I {} process_file {}

# Using GNU parallel (if available)
parallel command ::: arg1 arg2 arg3
```

### Timeout and Sleep

```bash
#!/bin/bash

# Sleep
sleep 5         # Sleep for 5 seconds
sleep 0.5       # Sleep for 0.5 seconds

# Timeout command
timeout 10s command  # Kill after 10 seconds

# Custom timeout function
run_with_timeout() {
    local timeout=$1
    shift
    
    "$@" &
    local pid=$!
    
    ( sleep $timeout; kill -9 $pid 2>/dev/null ) &
    local timeout_pid=$!
    
    wait $pid 2>/dev/null
    local exit_code=$?
    
    kill -9 $timeout_pid 2>/dev/null
    wait $timeout_pid 2>/dev/null
    
    return $exit_code
}
```

---

## Advanced Topics

### Command-Line Argument Parsing

```bash
#!/bin/bash

# Simple argument parsing
while [[ $# -gt 0 ]]; do
    case $1 in
        -h|--help)
            echo "Usage: $0 [options]"
            exit 0
            ;;
        -v|--verbose)
            VERBOSE=1
            shift
            ;;
        -f|--file)
            FILE="$2"
            shift 2
            ;;
        *)
            echo "Unknown option: $1"
            exit 1
            ;;
    esac
done

# Using getopts (for short options)
while getopts "hvf:" opt; do
    case $opt in
        h)
            echo "Usage: $0 [-h] [-v] [-f file]"
            exit 0
            ;;
        v)
            VERBOSE=1
            ;;
        f)
            FILE="$OPTARG"
            ;;
        \?)
            echo "Invalid option: -$OPTARG"
            exit 1
            ;;
    esac
done
```

### Error Handling

```bash
#!/bin/bash

# Exit on error
set -e  # Exit if any command fails
set -u  # Exit if undefined variable is used
set -o pipefail  # Exit if any command in pipeline fails

# Combine them
set -euo pipefail

# Custom error handling
error_exit() {
    echo "Error: $1" >&2
    exit 1
}

command || error_exit "Command failed"

# Trap errors
trap 'echo "Error on line $LINENO"' ERR

# Cleanup on exit
cleanup() {
    echo "Cleaning up..."
    rm -f /tmp/tempfile
}
trap cleanup EXIT

# Ignore errors for specific command
command || true
```

### Debugging

```bash
#!/bin/bash

# Enable debugging
set -x  # Print commands before executing
set -v  # Print shell input lines as they are read

# Debug specific section
set -x
# commands to debug
set +x

# Custom debug function
DEBUG=1
debug() {
    if [[ $DEBUG -eq 1 ]]; then
        echo "DEBUG: $*" >&2
    fi
}

debug "This is a debug message"

# Shell options
shopt -s extglob  # Enable extended pattern matching
shopt -s nullglob # Empty glob returns empty, not literal
```

### Signal Handling

```bash
#!/bin/bash

# Trap signals
cleanup() {
    echo "Caught signal, cleaning up..."
    # Cleanup code here
    exit 0
}

trap cleanup SIGINT SIGTERM

# Ignore signal
trap '' SIGINT

# Reset trap
trap - SIGINT

# Multiple signals
trap 'echo "Script interrupted"' INT TERM QUIT
```

### Regular Expressions

```bash
#!/bin/bash

text="Email: john@example.com"

# Basic regex matching
if [[ $text =~ [a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,} ]]; then
    echo "Valid email found"
    echo "Email: ${BASH_REMATCH[0]}"
fi

# Extract with groups
if [[ $text =~ ([a-z]+)@([a-z]+\.[a-z]+) ]]; then
    username="${BASH_REMATCH[1]}"
    domain="${BASH_REMATCH[2]}"
    echo "Username: $username"
    echo "Domain: $domain"
fi

# Using grep with regex
echo "$text" | grep -oP '\w+@\w+\.\w+'

# Using sed with regex
echo "$text" | sed -E 's/.*([a-z]+@[a-z]+\.[a-z]+).*/\1/'
```

### Command Substitution and Subshells

```bash
#!/bin/bash

# Command substitution
current_date=$(date)
echo "Date: $current_date"

# Old-style (backticks)
current_user=`whoami`
echo "User: $current_user"

# Subshell
(
    cd /tmp
    echo "In subshell: $(pwd)"
)
echo "Main shell: $(pwd)"

# Process substitution
diff <(ls dir1) <(ls dir2)

# Command grouping
{ command1; command2; } > output.txt
```

### Here Documents and Here Strings

```bash
#!/bin/bash

# Here document
cat << EOF
This is a multi-line
text block.
Variables like $USER are expanded.
EOF

# Here document without expansion
cat << 'EOF'
Variables like $USER are NOT expanded.
EOF

# Here document with indentation
cat <<- EOF
	This text is indented
	But leading tabs are removed
EOF

# Here string
grep "pattern" <<< "search in this string"

# Multi-line variable
config=$(cat << EOF
server {
    listen 80;
    server_name example.com;
}
EOF
)
```

### Working with JSON

```bash
#!/bin/bash

# Using jq (JSON processor)
json='{"name":"John","age":30,"city":"New York"}'

# Extract value
name=$(echo "$json" | jq -r '.name')
echo "Name: $name"

# Extract nested value
data='{"user":{"name":"John","email":"john@example.com"}}'
email=$(echo "$data" | jq -r '.user.email')

# Create JSON
jq -n \
    --arg name "John" \
    --arg email "john@example.com" \
    '{name: $name, email: $email}'

# Array processing
json='[{"name":"John"},{"name":"Jane"}]'
echo "$json" | jq -r '.[].name'
```

### Working with YAML

```bash
#!/bin/bash

# Using yq (YAML processor)
yaml='
name: John
age: 30
address:
  city: New York
  zip: 10001
'

# Extract value
name=$(echo "$yaml" | yq -r '.name')
city=$(echo "$yaml" | yq -r '.address.city')

# Convert YAML to JSON
echo "$yaml" | yq -o json

# Convert JSON to YAML
json='{"name":"John","age":30}'
echo "$json" | yq -P
```

### Networking

```bash
#!/bin/bash

# Download file
curl -O https://example.com/file.txt
wget https://example.com/file.txt

# HTTP GET request
response=$(curl -s https://api.example.com/data)
echo "$response"

# HTTP POST request
curl -X POST \
    -H "Content-Type: application/json" \
    -d '{"key":"value"}' \
    https://api.example.com/endpoint

# Check if port is open
nc -zv localhost 8080
timeout 1 bash -c "</dev/tcp/localhost/8080" && echo "Port open" || echo "Port closed"

# Get public IP
curl -s ifconfig.me
curl -s api.ipify.org

# DNS lookup
nslookup example.com
dig example.com
host example.com
```

### Date and Time

```bash
#!/bin/bash

# Current date/time
date
date "+%Y-%m-%d"
date "+%Y-%m-%d %H:%M:%S"

# Custom format
date "+%A, %B %d, %Y"  # Monday, January 15, 2024

# Unix timestamp
date +%s
timestamp=$(date +%s)

# Convert timestamp to date
date -d @$timestamp

# Date arithmetic
date -d "now + 5 days"
date -d "now - 1 week"
date -d "2024-01-01 + 30 days"

# Get day of week
dow=$(date +%u)  # 1-7 (Monday-Sunday)

# Compare dates
date1="2024-01-01"
date2="2024-12-31"
if [[ $(date -d "$date1" +%s) -lt $(date -d "$date2" +%s) ]]; then
    echo "$date1 is before $date2"
fi
```

---

## Best Practices

### Script Structure

```bash
#!/bin/bash

#############################################
# Script Name: example.sh
# Description: Brief description of what the script does
# Author: Your Name
# Date: YYYY-MM-DD
# Version: 1.0
#############################################

# Strict error handling
set -euo pipefail

# Constants (UPPERCASE)
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly SCRIPT_NAME="$(basename "$0")"
readonly MAX_RETRIES=3

# Global variables (lowercase)
verbose=0
dry_run=0

# Functions
show_usage() {
    cat << EOF
Usage: $SCRIPT_NAME [OPTIONS]

Options:
    -h, --help      Show this help message
    -v, --verbose   Enable verbose output
    -d, --dry-run   Perform a dry run
EOF
}

main() {
    # Parse arguments
    while [[ $# -gt 0 ]]; do
        case $1 in
            -h|--help)
                show_usage
                exit 0
                ;;
            -v|--verbose)
                verbose=1
                shift
                ;;
            -d|--dry-run)
                dry_run=1
                shift
                ;;
            *)
                echo "Unknown option: $1"
                show_usage
                exit 1
                ;;
        esac
    done
    
    # Main script logic here
    echo "Script executed successfully"
}

# Run main function
main "$@"
```

### Best Practices Summary

```bash
#!/bin/bash

# 1. Always use shebang
#!/bin/bash

# 2. Use strict mode
set -euo pipefail

# 3. Quote variables
echo "$variable"
echo "${array[@]}"

# 4. Use [[ ]] instead of [ ]
if [[ $var == "value" ]]; then
    # code
fi

# 5. Use functions
my_function() {
    local local_var="value"
    # code
}

# 6. Check command success
if command; then
    echo "Success"
else
    echo "Failed"
fi

# 7. Use meaningful variable names
user_count=10  # Good
uc=10          # Bad

# 8. Add comments
# This function calculates the sum of two numbers
calculate_sum() {
    local a=$1
    local b=$2
    echo $((a + b))
}

# 9. Use readonly for constants
readonly MAX_USERS=100

# 10. Validate inputs
if [[ $# -lt 2 ]]; then
    echo "Usage: $0 arg1 arg2"
    exit 1
fi

# 11. Use proper error messages
error_exit() {
    echo "Error: $1" >&2
    exit 1
}

# 12. Clean up temporary files
trap 'rm -f /tmp/tempfile' EXIT

# 13. Use descriptive function names
get_user_input() {
    # code
}

# 14. Avoid global variables when possible
# Use local variables in functions

# 15. Use logging
log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*"
}

log "Script started"
```

---

## Common Patterns and Examples

### Menu System

```bash
#!/bin/bash

show_menu() {
    clear
    echo "========================"
    echo "   Main Menu"
    echo "========================"
    echo "1. Option 1"
    echo "2. Option 2"
    echo "3. Option 3"
    echo "4. Exit"
    echo "========================"
}

while true; do
    show_menu
    read -p "Enter choice [1-4]: " choice
    
    case $choice in
        1)
            echo "Option 1 selected"
            read -p "Press enter to continue"
            ;;
        2)
            echo "Option 2 selected"
            read -p "Press enter to continue"
            ;;
        3)
            echo "Option 3 selected"
            read -p "Press enter to continue"
            ;;
        4)
            echo "Goodbye!"
            exit 0
            ;;
        *)
            echo "Invalid choice"
            read -p "Press enter to continue"
            ;;
    esac
done
```

### Backup Script

```bash
#!/bin/bash

set -euo pipefail

# Configuration
SOURCE_DIR="/home/user/documents"
BACKUP_DIR="/backup"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="backup_${DATE}.tar.gz"

# Create backup directory if it doesn't exist
mkdir -p "$BACKUP_DIR"

# Create backup
echo "Creating backup..."
tar -czf "${BACKUP_DIR}/${BACKUP_FILE}" "$SOURCE_DIR"

# Verify backup
if [[ -f "${BACKUP_DIR}/${BACKUP_FILE}" ]]; then
    echo "Backup created successfully: ${BACKUP_FILE}"
    echo "Size: $(du -h "${BACKUP_DIR}/${BACKUP_FILE}" | cut -f1)"
else
    echo "Error: Backup failed"
    exit 1
fi

# Delete old backups (keep last 7 days)
find "$BACKUP_DIR" -name "backup_*.tar.gz" -mtime +7 -delete
echo "Old backups cleaned up"
```

### Log Analyzer

```bash
#!/bin/bash

LOG_FILE="/var/log/app.log"

if [[ ! -f "$LOG_FILE" ]]; then
    echo "Error: Log file not found"
    exit 1
fi

echo "=== Log Analysis Report ==="
echo

# Count total lines
total_lines=$(wc -l < "$LOG_FILE")
echo "Total log entries: $total_lines"

# Count errors
error_count=$(grep -c "ERROR" "$LOG_FILE" || true)
echo "Error count: $error_count"

# Count warnings
warning_count=$(grep -c "WARNING" "$LOG_FILE" || true)
echo "Warning count: $warning_count"

# Top 10 most common errors
echo
echo "Top 10 most common errors:"
grep "ERROR" "$LOG_FILE" | sort | uniq -c | sort -rn | head -10

# Activity by hour
echo
echo "Activity by hour:"
grep -oP '\d{2}:\d{2}:\d{2}' "$LOG_FILE" | cut -d: -f1 | sort | uniq -c | sort -rn
```

### System Monitoring

```bash
#!/bin/bash

check_disk_space() {
    echo "=== Disk Space Usage ==="
    df -h | grep -v "tmpfs"
    echo
}

check_memory() {
    echo "=== Memory Usage ==="
    free -h
    echo
}

check_cpu() {
    echo "=== CPU Usage ==="
    top -bn1 | head -5
    echo
}

check_services() {
    echo "=== Service Status ==="
    for service in ssh nginx mysql; do
        if systemctl is-active --quiet $service; then
            echo "$service: Running"
        else
            echo "$service: Stopped"
        fi
    done
    echo
}

# Main execution
check_disk_space
check_memory
check_cpu
check_services
```

### File Processor

```bash
#!/bin/bash

set -euo pipefail

process_file() {
    local file=$1
    local line_count=$(wc -l < "$file")
    local word_count=$(wc -w < "$file")
    local char_count=$(wc -c < "$file")
    
    echo "File: $file"
    echo "  Lines: $line_count"
    echo "  Words: $word_count"
    echo "  Characters: $char_count"
    echo
}

# Process all .txt files in directory
for file in *.txt; do
    if [[ -f "$file" ]]; then
        process_file "$file"
    fi
done
```

### API Client

```bash
#!/bin/bash

set -euo pipefail

API_URL="https://api.example.com"
API_KEY="your_api_key_here"

# GET request
api_get() {
    local endpoint=$1
    curl -s -H "Authorization: Bearer $API_KEY" \
         "${API_URL}${endpoint}"
}

# POST request
api_post() {
    local endpoint=$1
    local data=$2
    curl -s -X POST \
         -H "Authorization: Bearer $API_KEY" \
         -H "Content-Type: application/json" \
         -d "$data" \
         "${API_URL}${endpoint}"
}

# Example usage
response=$(api_get "/users")
echo "Users: $response"

data='{"name":"John","email":"john@example.com"}'
response=$(api_post "/users" "$data")
echo "Created user: $response"
```

### Database Backup Script

```bash
#!/bin/bash

set -euo pipefail

# Configuration
DB_HOST="localhost"
DB_USER="root"
DB_PASS="password"
DB_NAME="mydb"
BACKUP_DIR="/backup/mysql"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="${DB_NAME}_${DATE}.sql.gz"

# Create backup directory
mkdir -p "$BACKUP_DIR"

# Backup database
echo "Backing up database: $DB_NAME"
mysqldump -h "$DB_HOST" -u "$DB_USER" -p"$DB_PASS" "$DB_NAME" | \
    gzip > "${BACKUP_DIR}/${BACKUP_FILE}"

# Verify backup
if [[ -f "${BACKUP_DIR}/${BACKUP_FILE}" ]]; then
    size=$(du -h "${BACKUP_DIR}/${BACKUP_FILE}" | cut -f1)
    echo "Backup successful: ${BACKUP_FILE} ($size)"
else
    echo "Backup failed!"
    exit 1
fi

# Keep only last 30 days of backups
find "$BACKUP_DIR" -name "${DB_NAME}_*.sql.gz" -mtime +30 -delete
echo "Old backups cleaned up"
```

---

## Conclusion

This guide covers Bash scripting from basic to advanced concepts. Practice is key to mastering Bash scripting. Start with simple scripts and gradually work your way up to more complex automation tasks.

### Additional Resources

- Official Bash Documentation: https://www.gnu.org/software/bash/manual/
- Advanced Bash-Scripting Guide: https://tldp.org/LDP/abs/html/
- ShellCheck (linting tool): https://www.shellcheck.net/
- Bash Reference Manual: https://www.gnu.org/savannah-checkouts/gnu/bash/manual/

### Practice Projects

1. Create a backup automation script
2. Build a system monitoring dashboard
3. Develop a log analyzer
4. Create an automated deployment script
5. Build a file organizer
6. Create a batch image processor
7. Develop a network monitoring tool
8. Build a password generator
9. Create a task scheduler
10. Develop a website uptime monitor

Happy scripting!
