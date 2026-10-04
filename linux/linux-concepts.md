# Linux Concepts Guide: Commands, Outputs, and Use Cases

## 1. File Permissions (`chown`, `touch`, `chmod`)
**When/Why to use:** Crucial for managing access to files and directories, allowing you to secure your system by controlling exactly who can read, write, and execute files.

### Creating a New File
**When to use:** Use `touch` as a quick way to bring an empty file into existence or to update the last accessed/modified timestamps of an existing file.

```bash
touch example.txt
ls
```
*Output: `example.txt`*

### Changing Ownership (`chown`)
**When to use:** Use when you need to hand over control of a file to another user (like an administrator or a specific service account). Using the recursive `-R` flag is particularly useful for managing complex directory structures where you want to ensure all nested files and subdirectories have the same owner.

```bash
ls -l example.txt
```
*Output:* `-rw-rw-r-- 1 labex labex 0 Jul 29 15:11 example.txt`

Changing ownership to `root`:
```bash
sudo chown root:root example.txt
ls -l example.txt
```
*Output:* `-rw-rw-r-- 1 root root 0 Jul 29 15:11 example.txt`

Changing ownership of a directory recursively:
```bash
sudo chown -R root:root new-dir
ls -lR new-dir
```

### Changing Permissions (`chmod`)
**When to use Numeric vs. Symbolic:** Numeric notation is concise for setting absolute permissions (e.g., 700). Symbolic notation is more intuitive when you only want to change a single permission (e.g., adding execute rights to a script) without recalculating the whole numeric string.

**Numeric Notation:**
```bash
sudo chmod 700 example.txt
```
*Output of `ls -l`:* `-rwx------ 1 root root 0 Jul 29 15:11 example.txt`

**Symbolic Notation (Adding Execute Permission to a Script):**
**Why:** Scripts require execute permissions to be run as programs.
```bash
chmod u+x script.sh
./script.sh
```
*Output:* `Hello, World`

---

## 2. User Account Management
**When/Why to use:** Fundamental skills for system administration to control access, set up new employees or services, and enforce security policies.

### Creating Users (`useradd`)
```bash
sudo useradd joker
sudo useradd -m bob  # -m creates a home directory
```

### Setting Passwords (`passwd`)
```bash
sudo passwd joker
```

### Modifying User Properties (`usermod`)
**When to use:** To update an existing user's home directory (`-d`), change their default shell to a more feature-rich one like bash (`-s`), or grant them administrative privileges (`-aG sudo`).

Changing default shell (Bash provides a more intuitive CLI and better scripting capabilities):
```bash
sudo usermod -s /bin/bash joker
```

**Adding to `sudo` group:**
**Why:** Grants administrative privileges without sharing the root password. It provides convenience, granular control, accountability (logging), and better security.
```bash
sudo usermod -aG sudo joker
```

### Locking/Unlocking and Deleting Accounts
**When to use locking:** When you need to temporarily disable a user account (e.g., employee on leave, investigating suspicious activity) without deleting their files.
* Lock: `sudo passwd -l joker`
* Unlock: `sudo passwd -u joker`
* Delete user and home dir: `sudo userdel -r bob`

### Advanced Password Aging & Groups (`chage`)
**When to use:** To enforce password security policies (forcing periodic password changes) and manage group memberships to control precise permissions.
```bash
sudo chage -M 90 -m 7 -W 14 student1
```

---

## 3. Package Management
**When/Why to use:** Fundamental, everyday skills for maintaining a healthy Linux system, installing new tools, and removing bloatware.
* `apt update`: Update package repositories.
* `apt install`: Install new software.
* `apt show`: Verify an installation.
* `apt remove`: Remove obsolete packages.
* `apt autoremove`: Clean up unused dependencies.

---

## 4. Wildcards
**When/Why to use:** Powerful tools for performing actions on large groups of files simultaneously without typing out every single filename.
* `*`: Matches any number of characters.
* `?`: Matches any single character.
* `[abc]`: Matches any one character listed in the brackets.

---

## 5. Variables (Local vs Environment)
**When/Why to use:** To store information in the shell. Use Local variables for temporary data in the current session. Use Environment (`export`) variables when the data needs to be accessed by child processes or scripts launched from that shell.

**Local Variable:**
```bash
my_var='value'
```

**Environment Variable:**
```bash
export MY_ENV_VAR="This is an environment variable"
```

---

## 6. Archiving and Compression
**When to use Packaging vs. Compression:**
* **Packaging (`tar`):** Used to combine multiple files and directories into a single file to make them easier to move or store. Does not significantly reduce size.
* **Compression (`gzip`, `bzip2`, `xz`):** Used specifically to apply algorithms that reduce the file's size to save disk space.

```bash
tar -cvf test_archive.tar test_dir
```

---

## 7. Disk Usage and Virtual Disks

### Viewing Disk Usage (`df` and `du`)
**When to use `df`:** Your go-to tool for checking overall disk space usage on mounted file systems.
**When to use `du`:** Your detective tool for finding out *which* specific directories or files are consuming that space.

```bash
du -h --max-depth=1 | sort -hr
```
*Output (example):*
```
5.0M    ./application
5.0M    .
0       ./system
```

Finding the largest specific files (excellent for freeing up space):
```bash
find . -type f -exec du -h {} + | sort -hr | head -n 5
```

### Virtual Disks & Partitions
**When to use Virtual Disks:** Useful for testing disk operations safely without risking real hardware, creating isolated storage spaces, or learning disk management.
**When to use Partitions (`fdisk`):** Separating system files from user files, applying different filesystems for different purposes, limiting the impact of disk failures, or isolating backups.

Creating a virtual disk and mounting it:
```bash
dd if=/dev/zero of=virtual.img bs=1M count=256
sudo mkfs.ext4 virtual.img
sudo mkdir /mnt/virtualdisk
sudo mount -o loop virtual.img /mnt/virtualdisk
```

---

## 8. Command Chaining and Redirection
* **Sequential execution (`&&`):** Use when the second command should only run if the first succeeds.
* **Logical OR (`||`):** Use when the second command should run if the first fails.
* **Pipe (`|`):** Use to connect the output of one command to the input of another.
* **/dev/null:** Use to discard unwanted output (the "black hole").
* **`tee`:** Use when you need to save a command's output to a file (like a log) while *also* viewing it on the terminal simultaneously.
* **`<` (Redirect stdin):** Essential when working with commands that can only read from standard input and don't accept a filename as a direct argument (e.g., `sort < items.txt`).

---

## 9. Text Processing Commands (`tr`, `join`, `paste`)

### The `tr` Command
**When to use:** Particularly useful for tasks like converting case, removing specific characters entirely, or replacing one character with another.

```bash
echo 'hello labex' | tr '[:lower:]' '[:upper:]'
# Output: HELLO LABEX
```

### `join` and `paste`
**When to use `join`:** Useful when you have related data split across multiple files and you want to combine them based on a *common key or identifier* (like a database join).
**When to use `paste`:** Useful when you want to combine files side-by-side blindly or create a table-like output without needing a common key.

---

## 10. `sed` (Stream Editor)
**When/Why to use:** A powerful tool for parsing and transforming text. It is often used to make automated, non-interactive edits to files or output streams inside scripts.

Basic vs Global substitution:
```bash
sed 's/Hello/Hi/' sed_test.txt    # Replaces first instance per line
sed 's/Hello/Hi/g' sed_test.txt   # Replaces all instances
```

---

## 11. `awk`
**When/Why to use:** Excellent for dealing with structured data (columns/fields). Perfect for counting occurrences, summarizing data, and analyzing server logs to identify potential security threats and performance issues.

Printing specific fields:
```bash
awk '{print $3}' server_logs.txt | head -n 5
```

Filtering log entries (e.g., finding POST requests):
```bash
awk '$4 == "POST" {print $0}' server_logs.txt | head -n 10
```

Summarizing Data (Counting occurrences of HTTP status codes):
```bash
awk '{count[$6]++} END {for (code in count) print code, count[code]}' server_logs.txt | sort -n
```

---

## 12. The `xargs` Command
**When/Why to use:** Used to take the output of one command and build it into arguments for another command. It bridges the gap between commands that produce lists (like `find` or `cat`) and commands that operate on arguments.

* **Placeholder (`-I {}`):** Crucial when the command you're running needs the input argument in the middle or at the end, rather than just appended to the end.
* **Limiting Arguments (`-n`):** Useful to process items in specific group sizes, especially if the target command has a limit on how many arguments it can accept.
* **Parallel Processing (`-P`):** Used to significantly reduce execution time by running independent tasks simultaneously (great for I/O bound operations).

```bash
cat books.txt | xargs -I {} touch {}.txt
cat more_books.txt | xargs -n 2 -P 3 sh -c 'echo "Processing batch: $@"' _
```

---

## 13. Shell Scripting Basics

### Command Line Arguments
**When to use:** Passing arguments (`$1`, `$2`, `$@`) allows scripts to be flexible and reusable rather than hardcoding values.

### String Operations
Useful for targeted text formatting, data processing, and filename manipulation inside scripts.
* Length: `${#STRING}`
* Substring: `${STRING:START:LENGTH}` (0-indexed)
* Replacement (First): `${STRING/pattern/replacement}`
* Replacement (All): `${STRING//pattern/replacement}`

### Conditionals and Loops
**When to use Loops:**
* `for`: Iterate over a known list of items or arrays.
* `while`: Execute code as long as a condition remains true.
* `until`: Execute code repeatedly until a condition becomes true.

### File & Directory Existence Tests
**When to use:** Crucial safeguards in scripts to verify a file (`-e`) or directory (`-d`) actually exists before attempting to read, modify, or delete it to prevent script errors.

### The `trap` Command
**When to use:** To catch and handle signals (like user interruptions via Ctrl+C). This allows your script to clean up temporary files and gracefully exit instead of terminating unpredictably.

---

## 14. Text Editors (`vi/vim` vs `nano`)

### `nano`
**When to use:**
* You are new to Linux text editing.
* Making quick, simple edits to configuration files.
* You prefer a straightforward interface with on-screen shortcuts and no modes.

### `vi/vim`
**When to use:**
* Doing extensive programming or complex text manipulation.
* Working on remote servers (vi is universally available out-of-the-box).
* You require advanced features (macros, complex search/replace).
* You value extreme efficiency and speed (once the learning curve is overcome).

---

## 15. Shell Environment and Configuration

### Automatic Export (`set -o allexport`)
**When to use:** A convenient shortcut in complex scripts where *all* locally defined variables need to be automatically passed to sub-processes without typing `export` for each one.

### Persistent Settings (`~/.bashrc` or `~/.zshrc`)
**When to use:** Aliases and variables reset when the terminal closes. Add them to your `rc` file to ensure personal customizations load automatically every time you open a terminal. Use `source ~/.zshrc` to apply them immediately.

---

## 16. `su` vs `su -`
**When/Why to use:**
* `su`: Switches the user identity but keeps your current environment profile.
* `su -`: Switches the user identity AND targets the target user's profile. **Why:** This is important when you need the exact permissions, PATH, and variables fetched specifically from that user's home directory.