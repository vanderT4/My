# Linux Concepts Guide

## 1. File Permissions (`chown`, `touch`, `chmod`)

### Creating a New File
The `touch` command creates new, empty files or updates the timestamps of existing ones.
```bash
touch example.txt
ls
```
*Output: `example.txt`*

### Changing Ownership (`chown`)
The `chown` command modifies the user and group ownership of a file or directory.
```bash
ls -l example.txt
```
*Output:*
```
-rw-rw-r-- 1 labex labex 0 Jul 29 15:11 example.txt
```
Changing ownership to `root`:
```bash
sudo chown root:root example.txt
ls -l example.txt
```
*Output:*
```
-rw-rw-r-- 1 root root 0 Jul 29 15:11 example.txt
```

Changing ownership of a directory recursively:
```bash
mkdir -p new-dir/subdir
echo "Hello, world" > new-dir/file1.txt
echo "Another file" > new-dir/subdir/file2.txt
ls -lR new-dir
```
*Output:*
```
new-dir:
total 4
-rw-rw-r-- 1 labex labex 13 Jul 29 09:15 file1.txt
drwxrwxr-x 2 labex labex 23 Jul 29 09:15 subdir
new-dir/subdir:
total 4
-rw-rw-r-- 1 labex labex 13 Jul 29 09:15 file2.txt
```
```bash
sudo chown -R root:root new-dir
ls -lR new-dir
```
*Output:*
```
new-dir:
total 4
-rw-rw-r-- 1 root root 13 Jul 29 09:15 file1.txt
drwxrwxr-x 2 root root 23 Jul 29 09:15 subdir
new-dir/subdir:
total 4
-rw-rw-r-- 1 root root 13 Jul 29 09:15 file2.txt
```

### Changing Permissions (`chmod`)
Permissions are represented by `r` (read, 4), `w` (write, 2), and `x` (execute, 1) for Owner, Group, and Others.

**Numeric Notation:**
```bash
sudo chmod 700 example.txt
ls -l example.txt
```
*Output:*
```
-rwx------ 1 root root 0 Jul 29 15:11 example.txt
```

**Directory Permissions:**
```bash
mkdir ~/test-dir
chmod 700 ~/test-dir
ls -ld ~/test-dir
```
*Output:*
```
drwx------ 2 labex labex 4096 Jul 29 15:45 /home/labex/test-dir
```
```bash
chmod -R 755 ~/test-dir
ls -ld ~/test-dir
```
*Output:*
```
drwxr-xr-x 2 labex labex 4096 Jul 29 15:45 /home/labex/test-dir
```

**Symbolic Notation:**
```bash
echo '#!/bin/bash' > script.sh
echo 'echo "Hello, World"' >> script.sh
chmod u+x script.sh
ls -l script.sh
```
*Output:*
```
-rwxrw-r-- 1 labex labex 32 Jul 29 16:30 script.sh
```
```bash
./script.sh
```
*Output:*
```
Hello, World
```

---

## 2. User Account Management

### Creating Users (`useradd`)
```bash
sudo useradd joker
sudo grep -w 'joker' /etc/passwd
```
*Output:*
```
joker:x:5001:5001::/home/joker:/bin/sh
```
Creating a user with a home directory:
```bash
sudo useradd -m bob
sudo ls -ld /home/bob
```
*Output:*
```
drwxr-x--- 2 bob bob 57 Jan 19 13:33 /home/bob
```

### Setting Passwords (`passwd`)
```bash
sudo passwd joker
```

### Modifying User Properties (`usermod`)
Changing a home directory:
```bash
sudo usermod -d /home/wayne joker
```
Changing default shell:
```bash
sudo usermod -s /bin/bash joker
```
Adding to `sudo` group:
```bash
sudo usermod -aG sudo joker
```

### Locking/Unlocking and Deleting Accounts
Locking: `sudo passwd -l joker`
Unlocking: `sudo passwd -u joker`
Deleting: `sudo userdel -r bob`

### Advanced User Management (`chage` and `usermod`)
Viewing password aging: `sudo chage -l student1`
Modifying password aging (90 max, 7 min, 14 warning): 
```bash
sudo chage -M 90 -m 7 -W 14 student1
```
Modifying secondary groups:
```bash
sudo groupadd developers
sudo usermod -aG developers student1
```

---

## 3. Package Management
*   `apt update`: Update package repositories.
*   `apt install`: Install new software.
*   `apt show`: Verify an installation.
*   `apt remove`: Remove obsolete packages.
*   `apt autoremove`: Clean up unused dependencies.

---

## 4. Wildcards
*   `*`: Matches any number of characters.
*   `?`: Matches any single character.
*   `[abc]`: Matches any one character listed in the brackets.

---

## 5. Variables (Local vs Environment)
**Local Variable:**
```bash
my_var='value'
```
**Environment Variable:**
```bash
export MY_ENV_VAR="This is an environment variable"
echo $MY_ENV_VAR
```
Removing variables: `unset MY_ENV_VAR`
Updating PATH: `export PATH="$PATH:$HOME/my_scripts"`

---

## 6. Archiving and Compression
**Packaging (Archiving) with `tar`:**
```bash
tar -cvf test_archive.tar test_dir
```
*Packaging* combines multiple files into a single file. *Compression* (gzip, bzip2, xz) reduces the size using algorithms.

---

## 7. Disk Usage and Virtual Disks
### Viewing Disk Usage (`df` and `du`)
`df` checks disk space usage on the system.
`du` estimates file space usage.
```bash
du -h
```
*Output:*
```
10M     ./backups
5.0M    ./logs/application
15M     .
```
Sort by size: `du -h | sort -hr`

Finding the largest files:
```bash
find . -type f -exec du -h {} + | sort -hr | head -n 5
```

### Virtual Disks
Creating a 256MB virtual disk:
```bash
dd if=/dev/zero of=virtual.img bs=1M count=256
```
Formatting with ext4:
```bash
sudo mkfs.ext4 virtual.img
```
Mounting the virtual disk:
```bash
sudo mkdir /mnt/virtualdisk
sudo mount -o loop virtual.img /mnt/virtualdisk
```
Unmounting: `sudo umount /mnt/virtualdisk`

### fdisk
Viewing partition information:
```bash
sudo fdisk -l virtual.img
```

---

## 8. Command Chaining and Redirection
*   Sequential execution: `date && ls -l`
*   Logical OR: `||`
*   Pipe: `|` Connects the output of one command to the input of another.
*   `/dev/null`: The "bit bucket" that discards all data written to it.
*   `tee`: Splits output to both a file and standard output.
*   `<`: Redirects standard input from a file (e.g., `sort < items.txt`).

---

## 9. Text Processing Commands
*   `wc -l`: Word count of lines.
*   `wc -w`: Word count of words.
*   `uniq`: Removes duplicate lines (often used with sort).
    ```bash
    cut -d: -f7 /etc/passwd | sort | uniq
    ```

### The `tr` Command
Delete specific characters:
```bash
echo 'hello labex' | tr -d 'olh'
# Output: e abex
```
Remove duplicate characters:
```bash
echo 'hello' | tr -s 'l'
# Output: helo
```
Convert case:
```bash
echo 'hello labex' | tr '[:lower:]' '[:upper:]'
# Output: HELLO LABEX
```

### `join` and `paste`
*   `join`: Joins lines of two files on a common field.
*   `paste`: Merges lines of files side-by-side.
    ```bash
    paste fruits.txt colors.txt tastes.txt
    ```

---

## 10. `sed` (Stream Editor)
Basic substitution:
```bash
sed 's/Hello/Hi/' sed_test.txt
```
Global substitution:
```bash
sed 's/Hello/Hi/g' sed_test.txt
```
Alternative delimiters: `sed 's#/path/to/file#/new/path#g' filename`
Advanced usage:
*   Delete a line: `sed '2d' sed_test.txt`
*   Insert text: `sed '1i\First line' sed_test.txt`
*   Append text: `sed '$a\Last line' sed_test.txt`
*   Multiple commands: `sed -e 's/Hi/Hello/g' -e 's/labex/LabEx/g' sed_test.txt`
*   Regex: `sed 's/[Ww]orld/Universe/g' sed_test.txt`

---

## 11. `awk`
`awk` processes text by treating each line as a record and words as fields (`$1`, `$2`, etc.).

Printing specific fields:
```bash
awk '{print $3}' server_logs.txt | head -n 5
```
*Output:*
```
192.168.1.100
192.168.1.101
```

Filtering entries:
```bash
awk '$4 == "POST" {print $0}' server_logs.txt | head -n 10
awk '$6 == "404" {print $1, $2, $5}' server_logs.txt
awk '$4 == "POST" && $6 >= 400 {print $0}' server_logs.txt
```

Counting and Summarizing (using Arrays):
Count occurrences of HTTP status codes:
```bash
awk '{count[$6]++} END {for (code in count) print code, count[code]}' server_logs.txt | sort -n
```
*Output:*
```
200 3562
404 89
```

Top 5 requested resources:
```bash
awk '{count[$5]++} END {for (resource in count) print count[resource], resource}' server_logs.txt | sort -rn | head -n 5
```
*Output:*
```
1823 /index.html
956 /about.html
```

---

## 12. The `xargs` Command
`xargs` builds and executes commands from standard input.

Basic Usage:
```bash
cat fruits.txt | xargs echo
```
*Output: `apple orange banana`*

Processing Files (Using `-I {}` placeholder):
```bash
cat books.txt | xargs -I {} touch {}.txt
```

Limiting Arguments (`-n`):
```bash
cat more_books.txt | xargs -n 2 echo "Processing books:"
```

Parallel Processing (`-P`):
```bash
cat more_books.txt | xargs -P 3 -I {} ./process_book.sh {}
```

Combining Options with a Subshell:
```bash
cat classic_books.txt | xargs -n 2 -P 3 sh -c 'echo "Processing batch: $@"' _
```

---

## 13. Shell Scripting Basics

### Command Line Arguments and Special Variables
*   `$0`: Script name
*   `$1`, `$2`: First and second arguments
*   `$@`: All arguments
*   `$#`: Number of arguments
*   `$$`: Process ID of current shell
*   `$?`: Exit status of the last command
*   `$!`: Process ID of the last background command

Difference between `$@` and `$*`:
`"$@"` treats each argument as a separate entity. `"$*"` combines all arguments into a single string.

### Arrays
```bash
NUMBERS=()
NUMBERS+=(1 2 3)
NumberOfNames=${#NAMES[@]}
second_name=${NAMES[1]}
```

### Arithmetic
```bash
TOTAL=$((COST_PINEAPPLE + (COST_BANANA * 2)))
```

### String Operations
*   Length: `${#STRING}`
*   Character Position: `$(expr index "$STRING" "$CHAR")` (1-indexed)
*   Substring: `${STRING:START:LENGTH}` (0-indexed)
*   Replacement: 
    *   First occurrence: `${STRING/o/O}`
    *   All occurrences: `${STRING//o/O}`
    *   At beginning: `${STRING/#The quick/The slow}`
    *   At end: `${STRING/%dog/cat}`

### Conditionals
```bash
if [ "$NAME" = "John" ]; then
  echo "The name is John"
elif [ $NUMBER -eq 10 ]; then
  echo "Number is 10"
fi
```
Operators: `-eq` (equal), `-lt` (less than), `-gt` (greater than), `-z` (string is empty), `&&` (AND), `||` (OR).

### Loops
**For Loop:**
```bash
for name in "${NAMES[@]}"; do
  echo "Hello, $name!"
done
```
**While Loop:**
```bash
while [ $count -gt 0 ]; do
  count=$((count - 1))
done
```
**Until Loop:** Continues executing until a condition becomes true.
`break` and `continue` can be used to control flow within loops.

### Functions
```bash
greet() {
  local name=$1
  echo "Hello, $name!"
}
```
Use the `local` keyword to keep variables confined to the function. Functions return values by either echoing them (`result=$(get_square 5)`) or modifying global variables.

### The `trap` Command
Used to catch and handle signals (like Ctrl+C).
```bash
cleanup_and_exit() {
  echo "Cleaning up..."
  exit 0
}
trap cleanup_and_exit SIGINT SIGTERM
```

### File & Directory Existence
Check if file exists: `if [ -e "$filename" ]; then`
Check if directory exists: `if [ -d "$dirname" ]; then`

---

## 14. Text Editors (`vi/vim` vs `nano`)

### `vi/vim`
*   **Modes**: Normal Mode (commands) and Insert Mode (typing).
*   **Enter Insert Mode**: `i`
*   **Enter Normal Mode**: `Esc`
*   **Save & Quit**: `:w` (save), `:wq` (save & quit), `:q!` (quit discarding changes).
*   **Navigation**: `h` (left), `j` (down), `k` (up), `l` (right), `gg` (jump to top).
*   **Search**: `/word` to search forward, `n` to jump to next.
*   **Edit**: `dw` to delete a word, `o` to open a new line below and enter Insert Mode.

### `nano`
A modeless, beginner-friendly text editor.
*   Save: `Ctrl+O` (Write Out)
*   Exit: `Ctrl+X`

---

## 15. Shell Environment and Configuration

### Parent vs Child Shells
Environment variables (`export var_name=value`) are inherited by child shells (`zsh` or script executions). Local variables and aliases (`alias ldetc='ls -ld /etc'`) are not inherited.

### Automatic Export
Enable automatic exporting of all new variables:
```bash
set -o allexport
```

### Persistent Settings
Add permanent aliases, shell options, and variables to the configuration file (`~/.zshrc` or `~/.bashrc`). Apply changes immediately with:
```bash
source ~/.zshrc
```

## 16. `su` vs `su -`
*   `su`: Switches user but keeps the current environment profile.
*   `su -`: Switches user and targets the target user's environment profile (fetching permissions and variables from their home directory).