

# Index

**Theory** : [what is ssh (Secure Shell) ?](OverTheWire%20Bandit%20Notes.md#what-is-ssh-secure-shell-) | [Problem SSH solved](OverTheWire%20Bandit%20Notes.md#problem-ssh-solved) | [The core value of SSH](OverTheWire%20Bandit%20Notes.md#the-core-value-of-ssh) | [SSH notes](OverTheWire%20Bandit%20Notes.md#ssh-notes)

|Level|Topic|Key command(s)|Link|
|---|---|---|---|
|0|First SSH login, reading a file|`ssh`, `cat`|[Bandit level 0](OverTheWire%20Bandit%20Notes.md#bandit-level-0)|
|1|File named `-`|`cat ./-`|[Bandit level 1](OverTheWire%20Bandit%20Notes.md#bandit-level-1)|
|2|File name with spaces|quotes, `\`, `--`|[bandit level 2](OverTheWire%20Bandit%20Notes.md#bandit-level-2)|
|3|Hidden files|`ls -a`, `cat .` + Tab|[Bandit level 3](OverTheWire%20Bandit%20Notes.md#bandit-level-3)|
|4|Find the human-readable file|`file`, `for` loop|[Bandit level 4](OverTheWire%20Bandit%20Notes.md#bandit-level-4)|
|5|Find by size / not executable|`find`|[Bandit level 5](OverTheWire%20Bandit%20Notes.md#bandit-level-5)|
|6|Find by owner / group / size on the whole server|`find`, `2>/dev/null`|[bandit level 6](OverTheWire%20Bandit%20Notes.md#bandit-level-6)|
|7|not started yet||[Bandit level 7](OverTheWire%20Bandit%20Notes.md#bandit-level-7)|

## Concept Map

|Concept|Where|
|---|---|
|SSH connection command (`-p 2220`)|[Bandit level 0](OverTheWire%20Bandit%20Notes.md#bandit-level-0), [SSH notes](OverTheWire%20Bandit%20Notes.md#ssh-notes)|
|Filenames that look like options (`-`, `--`)|[Bandit level 1](OverTheWire%20Bandit%20Notes.md#bandit-level-1), [bandit level 2](OverTheWire%20Bandit%20Notes.md#bandit-level-2), [Bandit level 4](OverTheWire%20Bandit%20Notes.md#bandit-level-4)|
|Quoting / escaping spaces, word splitting|[bandit level 2](OverTheWire%20Bandit%20Notes.md#bandit-level-2)|
|File naming conventions, batch rename|[bandit level 2](OverTheWire%20Bandit%20Notes.md#bandit-level-2)|
|Hidden files|[Bandit level 3](OverTheWire%20Bandit%20Notes.md#bandit-level-3), [Bandit level 5](OverTheWire%20Bandit%20Notes.md#bandit-level-5)|
|Tab completion|[Bandit level 3](OverTheWire%20Bandit%20Notes.md#bandit-level-3)|
|File types, loops in bash|[Bandit level 4](OverTheWire%20Bandit%20Notes.md#bandit-level-4)|
|`find` (syntax, filters, actions, `-size` units)|[Bandit level 5](OverTheWire%20Bandit%20Notes.md#bandit-level-5), [bandit level 6](OverTheWire%20Bandit%20Notes.md#bandit-level-6)|
|Permissions, owner, group|[bandit level 6](OverTheWire%20Bandit%20Notes.md#bandit-level-6)|
|Standard streams, `2>/dev/null`|[bandit level 6](OverTheWire%20Bandit%20Notes.md#bandit-level-6)|
|Host fingerprint, port, SSH keys|[SSH notes](OverTheWire%20Bandit%20Notes.md#ssh-notes)|

## Command Index

|Command|Used for|Level|
|---|---|---|
|`ssh user@host -p 2220`|Connect to the server|[Bandit level 0](OverTheWire%20Bandit%20Notes.md#bandit-level-0)|
|`cat`|Read a file|[Bandit level 0](OverTheWire%20Bandit%20Notes.md#bandit-level-0)|
|`cat ./-`|Read a file named `-`|[Bandit level 1](OverTheWire%20Bandit%20Notes.md#bandit-level-1)|
|`cat "./name with spaces"` / `cat name\ with\ spaces`|Files with spaces|[bandit level 2](OverTheWire%20Bandit%20Notes.md#bandit-level-2)|
|`cat -- file`|Read a file whose name starts with `-`|[bandit level 2](OverTheWire%20Bandit%20Notes.md#bandit-level-2), [Bandit level 4](OverTheWire%20Bandit%20Notes.md#bandit-level-4)|
|`ls -a`|List hidden files|[Bandit level 3](OverTheWire%20Bandit%20Notes.md#bandit-level-3)|
|`for file in ./*; do ... done`|Loop over every file|[Bandit level 4](OverTheWire%20Bandit%20Notes.md#bandit-level-4)|
|`file ./inhere/*`|Detect ASCII text vs data|[Bandit level 4](OverTheWire%20Bandit%20Notes.md#bandit-level-4)|
|`find . -type f -size 1033c ! -executable`|Find by size, not executable|[Bandit level 5](OverTheWire%20Bandit%20Notes.md#bandit-level-5)|
|`find / -type f -size 33c -user bandit7 -group bandit6 2>/dev/null`|Find by owner/group/size, hide errors|[bandit level 6](OverTheWire%20Bandit%20Notes.md#bandit-level-6)|
|`ls -l`, `id`|Read permissions, owner, group|[bandit level 6](OverTheWire%20Bandit%20Notes.md#bandit-level-6)|
|`man find`|Check an option in the manual|[Bandit level 5](OverTheWire%20Bandit%20Notes.md#bandit-level-5)|

---

# what is ssh (Secure Shell) ?

**SSH** is a secure cryptographic network protocol used to connect and control remote computer over an unsecure network (like internet )

# Problem SSH solved

Befor **SSH**, developers used the old system **Telnet** which was dangerous because it transmitted everything in **plain text** .

**logging into a server using Telnet at a coffee shop, an attacker on the same WI-FI could read your pass and see your keystrokes effortlessly.**

**SSH** solves this by using **strong mathematical encryption**. So, when an attacker intercept data mid-flight it looks like absolute **gibberish** .

# The core value of SSH

## Remote Server Administration

Most servers running the cloud like aws or google cloud do not have a monitor, keyboard or graphical user interface attached to them. They sit in massive data centers. Engineers use SSH to log into them remotely to configure software, update websites, or fix errors entirely from their local command line.[OLOID- Everything You Must Know About Security Shell (SSH)](https://www.oloid.com/blog/everything-you-must-know-about-security-shell).

## Encrypted Automation (SSH Keys)

Production setups rely on **SSH Keys** which are **pairs of long, unbreakable cryptographic files**. They allow software scripts and automated servers to log into each other securely without a human ever having to type a password . [SSH Academy - SSH Protocol – Secure Remote Login and File Transfer](https://www.ssh.com/academy/ssh/protocol) .

## Secure File Transfer (SFTP / SCP)

**SSH** includes subsystems like **SFTP (SSH File Transfer Protocol)** and **SCP (Secure Copy)**. This means we can securely move large project folders or sensitive logs back and forth between the local computer and the cloud server using the exact same secure channel.[GoAnywhere - What Is SSH and How Does It Work?](https://www.goanywhere.com/blog/what-is-ssh-and-how-does-it-work).

# SSH notes

**Host fingerprint** : the first time we connect, ssh shows the fingerprint of the server's key and asks if we want to continue (yes/no) , then it saves the key in `~/.ssh/known_hosts` . If the key changes later, ssh shows a warning ( the server was reinstalled or someone is intercepting the connection ) .

**Port** : the default port of SSH is 22 , bandit uses the port 2220 so we must write `-p 2220` .

**SSH Keys** : `ssh-keygen` creates a pair : a **private key** ( stays on our machine , never share it ) and a **public key** ( added to `~/.ssh/authorized_keys` on the server ) . We connect with `ssh -i private_key user@host -p port` and the private key file must be protected ( `chmod 600` ) or ssh refuses it . Later bandit levels use a private key instead of a password .

[Index](OverTheWire%20Bandit%20Notes.md#index) | [Next](OverTheWire%20Bandit%20Notes.md#bandit-level-0)

---

# Bandit level 0

```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
```

**Password**: [REDACTED — retrieve from the live challenge]

**Command** : cat readme

**Output** : Congratulations on your first steps into the bandit game!! Please make sure you have read the rules at https://overthewire.org/rules/ If you are following a course, workshop, walkthrough or other educational activity, please inform the instructor about the rules as well and encourage them to contribute to the OverTheWire community so we can keep these games free!

The password you are looking for is: `[REDACTED in the public copy — retrieve it from the live Bandit challenge during practice]`

**See also** : [what is ssh (Secure Shell) ?](OverTheWire%20Bandit%20Notes.md#what-is-ssh-secure-shell-) , [SSH notes](OverTheWire%20Bandit%20Notes.md#ssh-notes)

[Index](OverTheWire%20Bandit%20Notes.md#index) | [Next](OverTheWire%20Bandit%20Notes.md#bandit-level-1)

---

# Bandit level 1

```bash
ssh bandit1@bandit.labs.overthewire.org -p 2220 
```

**Password**: [REDACTED Bandit challenge password]

**Context** : the password of the next level is stored in a file called `-` located in the `~` directory .

**Problem encountered** : When I execute the command `cat -` it doesn't show the file , it waits for input and seems stuck ( Ctrl+C to exit ) .

**Cause of the problem** : The single dash `-` is treated as a command flag or standard I/O option ( in cat , `-` means read from standard input ) .

**Solution** : _Read the file safely_ , use relative path like `./` or the absolute path ,so the terminal knows that `-` is a file and not an option flag.

**Output** : [REDACTED Bandit challenge password]

**See also** : [bandit level 2](OverTheWire%20Bandit%20Notes.md#bandit-level-2)

[Index](OverTheWire%20Bandit%20Notes.md#index) | [Previous](OverTheWire%20Bandit%20Notes.md#bandit-level-0) | [Next](OverTheWire%20Bandit%20Notes.md#bandit-level-2)

---

# bandit level 2

```bash
ssh bandit2@bandit.labs.overthewire.org -p 2220 
```

**Password** : [REDACTED Bandit challenge password]

**Context** : The password for the next level is stored in a file called `--spaces in this filename--` located in the home directory

**Problem encountered** :

_**First problem**_ :

```bash
cat --spaces in this filename--
error: unexpected argument '--spaces' found

  tip: to pass '--spaces' as a value, use '-- --spaces'

Usage: cat [OPTION]... [FILE]...

For more information, try '--help'.
```

cat command treated `--spaces` as an option . However, it is not an existing argument for the command `cat` .

_**Second problem**_:

```bash
cat ./--spaces in this filename--
cat: ./--spaces: No such file or directory
cat: in: No such file or directory
cat: this: No such file or directory
cat: filename--: No such file or directory
```

The shell split the file name at the spaces before running cat ( word splitting ) , so cat treated the parts as separate filenames: `--spaces`, `in`, `this` and `filename--`. Those files do not exist; the actual filename is `--spaces in this filename--`.

**Solutions** :

_**First Solution**_ : Use Quotation Marks "" with `./` ( quotes fix the spaces , `./` fixes the dashes ) :

```bash
cat "./--spaces in this filename--"
```

_**Second Solution**_ : Escape the Spaces : Place a backslash (`\`) directly before every space character.

```bash
cat ./--spaces\ in\ this\ filename--
```

_**Third Solution**_ : Use `--` ( end of options ) with quotes :

```bash
cat -- "--spaces in this filename--"
```

**Output** : [REDACTED Bandit challenge password]

**NOTES** : _**Best Practice for Naming Files**_: **Snake Case** : Replace spaces with underscores -> my_file_name.txt

**Kebab Case** : Replace spaces with hyphens/dashes -> my-file-name.txt

**Camel Case** : Capitalize the first letter of each word without spaces -> MyFileName.txt

_**Batch Renaming Files**_:

```bash
for file in *\ *; do mv "$file" "${file// /_}"; done
```

**See also** : [Bandit level 1](OverTheWire%20Bandit%20Notes.md#bandit-level-1) ( the `./` trick ) , [Bandit level 4](OverTheWire%20Bandit%20Notes.md#bandit-level-4) ( more about `--` )

[Index](OverTheWire%20Bandit%20Notes.md#index) | [Previous](OverTheWire%20Bandit%20Notes.md#bandit-level-1) | [Next](OverTheWire%20Bandit%20Notes.md#bandit-level-3)

---

# Bandit level 3

```bash
ssh bandit3@bandit.labs.overthewire.org -p 2220
```

**Password** : [REDACTED Bandit challenge password]

**Context** : The password for the next level is stored in a hidden file in the **inhere** directory.

**Problem Encountered** : no problem

**Command used** :

```bash
cat ...Hiding-From-You
```

I just typed `cat .` and then press tab and all the hidden files will appear so I choose the one I want to show its content .

**Output** : [REDACTED Bandit challenge password]

**Notes** : a hidden file is just a file whose name starts with a dot , `ls -a` ( or `ls -la` ) lists them .

**See also** : [Bandit level 5](OverTheWire%20Bandit%20Notes.md#bandit-level-5) ( `.file2` is a hidden file too )

[Index](OverTheWire%20Bandit%20Notes.md#index) | [Previous](OverTheWire%20Bandit%20Notes.md#bandit-level-2) | [Next](OverTheWire%20Bandit%20Notes.md#bandit-level-4)

---

# Bandit level 4

```bash
ssh bandit4@bandit.labs.overthewire.org -p 2220
```

**Password** : [REDACTED Bandit challenge password]

**Context**: The password for the next level is stored in the only human-readable file in the **inhere** directory. Tip: if your terminal is messed up, try the “reset” command.

**No problem Encountered**

**Solutions** :

_**First solution**_ : to cat every single file under `inhere` directory

_**Second solution**_ : Automate it to save time and effort :

```bash
for file in ./*; do
    if [ -f "$file" ]; then
        cat -- "$file"
        echo ""
    fi
done

```

**Output** :

```bash
�F1d�aIڗ�3n̊�����7��@Ԭ��ci^�
���Nm����t���~
��L]y���wæ����b
Yi�u`p�L),�=���7�=動_g3}��^܎O3
a0q�܃��b��H~
            R>����ˡ��b�
����r�ul���|v,�c`�E�JC��H)
�.�6���f���f�ku�	cc4ȅ2��wk�pt�
�?/�[��?
        x��
           �S7Y]��1ܿV���7
[REDACTED Bandit challenge password]

}a�"����d]���:�v�&��}b�m6:�79
�D�w��W%��"S��nAy.�KƩ���"6�
```

the only string which is human-readable is : [REDACTED Bandit challenge password] which is located in the file `-file07`

_**Third Solution ( best )**_ use the `file` command to show if one file is **ASCII text** ( human-readable) or **data** .

```bash
file ./inhere/*
```

**output** :

```
./-file00: data
./-file01: data
./-file02: data
./-file03: data
./-file04: data
./-file05: data
./-file06: data
./-file07: ASCII text
./-file08: data
./-file09: data

```

So we can notice that the file `-file07` is **ASCII text** so it is the readable data .

```bash
cat ./inhere/-file07
```

**Output** :

```
[REDACTED Bandit challenge password]
```

**Notes** : _**The `--` option**_ : it means end of options , everything after it is a file name even if it starts with `-` . That is why `cat -- "$file"` works with files like `-file07` ( same idea as `./` in level 1 and level 2 ) . We also quote `"$file"` so the shell does not split names with spaces .

**See also** : [Bandit level 1](OverTheWire%20Bandit%20Notes.md#bandit-level-1) , [bandit level 2](OverTheWire%20Bandit%20Notes.md#bandit-level-2) , [Bandit level 5](OverTheWire%20Bandit%20Notes.md#bandit-level-5) ( `find` is another way to filter files )

[Index](OverTheWire%20Bandit%20Notes.md#index) | [Previous](OverTheWire%20Bandit%20Notes.md#bandit-level-3) | [Next](OverTheWire%20Bandit%20Notes.md#bandit-level-5)

---

# Bandit level 5

```bash
ssh bandit5@bandit.labs.overthewire.org -p 2220
```

**Password** : [REDACTED Bandit challenge password]

**Context** : The password for the next level is stored in a file somewhere under the **inhere** directory and has all of the following properties:

- human-readable
- 1033 bytes in size
- not executable

**Solution** : Use the `find` command to see which file meets the requirements to get the next password . command :

```bash
find . -type f -size 1033c ! -executable
```

**Output** : `./maybehere07/.file2`

```bash
cat ./maybehere07/.file2
```

**Output** : [REDACTED Bandit challenge password]

**Notes**: **The absolute general syntax for the find command in Unix/linux is**:

```bash
find [starting_directories] [expression_options] [filters/tests] [actions]
```

1. `[starting_directories]`-> where to look
    
    - `.` : Search current directory and **subdirectories** .
    - `/` : Search the entire system.
    - `/var/log` `/tmp` : search multiple specific directories.
2. `[expression_options]` -> how to behave
    
    - `-maxdepth N` : Descend at most N levels of directories .
    - `-mindepth N` : Do not apply tests at levels less than N .
    - `-L` : Follow symbolic links.
3. `[filters/tests]` (What to look for)
    
    - **By Name:** `-name "pattern"` (case-sensitive) or `-iname "pattern"` (case-insensitive).
    - **By Type:** `-type f` (files), `-type d` (directories), `-type l` (symlinks).
    - **By Size:** `-size +10M` (greater than 10MB), `-size -5k` (less than 5KB), `-size 1033c` (exactly 1033 bytes).
    - **By Time:** `-mtime -7` (modified in last 7 days), `-amin +60` (accessed more than 60 minutes ago).
    - **By Ownership/Permissions:** `-user username`, `-group groupname`, `-perm 644`.
4. `[actions]` (What to do with the results) If omitted, it defaults to `-print` (displaying paths on the screen).
    
    - `-print` : Plain text output (default).
    - `-delete` : Delete matched files instantly (use with extreme caution).
    - `-exec command {} +` : Run an external command efficiently over all found files at once.
    - `-exec command {} \;` : Run an external command separately for _each_ found file.

**Example**:

```bash
find /path/to/search -maxdepth 2 -type f -name "*.log" -size +1M -exec chmod 644 {} +
```

_**About `-size`**_ : without a suffix find counts blocks of 512 bytes , so always add a unit : `c` bytes , `k` kibibytes , `M` mebibytes , `G` gibibytes . `+n` greater than , `-n` less than , `n` exactly .

_**Check the manual**_ : `man find` ( search with `/text` , quit with `q` ) or `find --help` .

**See also** : [Bandit level 3](OverTheWire%20Bandit%20Notes.md#bandit-level-3) ( hidden file ) , [bandit level 6](OverTheWire%20Bandit%20Notes.md#bandit-level-6) ( `find` again )

[Index](OverTheWire%20Bandit%20Notes.md#index) | [Previous](OverTheWire%20Bandit%20Notes.md#bandit-level-4) | [Next](OverTheWire%20Bandit%20Notes.md#bandit-level-6)

---

# bandit level 6

```bash
ssh bandit6@bandit.labs.overthewire.org -p 2220
```

**Password** : [REDACTED Bandit challenge password]

**Context** : The password for the next level is stored **somewhere on the server** and has all of the following properties:

- owned by user bandit7
- owned by group bandit6
- 33 bytes in size

**Problem Encountered** : I used the right command which is find with the right params :

```bash
find / -type f -size 33c -user bandit7 -group bandit6
```

The problem is that the _output_ of the command is too messy with permission error files , these files I don't have the permission as a user to read them so it shows the error of _permission denied_ for example `find: ‘/var/lib/amazon’: Permission denied`

**Solution**: To hide the _permission denied_ errors I added the option : `2>/dev/null` , so the command become :

```bash
find / -type f -size 33c -user bandit7 -group bandit6 2>/dev/null
```

**Output** : `/var/lib/dpkg/info/bandit7.password`

```bash
cat /var/lib/dpkg/info/bandit7.password
```

**Output** : [REDACTED Bandit challenge password]

**Notes** : _What is the option 2>/dev/null_ : The option **`2>/dev/null`** is a powerful trick in Linux/Unix used to **hide error messages** from your terminal screen.

1. The Core Meaning
    - **`2`**: Represents **Standard Error (stderr)**. This is the specific stream where programs send error messages.
    - **`>`**: The **redirect operator**. It takes data from the left side and sends it to the destination on the right side.
    - **`/dev/null`**: A special virtual file in Linux called the **"null device"** or "black hole." Anything you send here vanishes forever and cannot be recovered.
2. The Three Standard Streams (Data Channels)

|Number|Name|Description|Default Destination|
|---|---|---|---|
|0|`stdin` (Standard Input)|Keyboard input or data fed into a command|keyboard|
|1|`stdout` (Standard Output)|The normal, successful output of a command.|screen|
|2|`stderr` (Standard Error)|Error messages generated by a command.|screen|

Because both `1` and `2` print to the screen by default, they look identical to us. But Linux sees them as separate channels. `2>/dev/null` targets _only_ channel 2.

3. Variations You Will See in the Wild

- **`> /dev/null` (or `1>/dev/null`)**  
    This hides **normal output** but still shows errors. Use this if you only care if a command fails, but don't want to see its normal spam.
- **`> /dev/null 2>&1`**  
    This hides **everything** (both normal output and errors). It tells Linux: _"Send normal output to the black hole, and send channel 2 (errors) to the exact same place as channel 1."_
- **`&> /dev/null`**  
    A modern, shorter way of writing `> /dev/null 2>&1`. It completely silences the command.

_**Permissions and ownership**_ : in `ls -l` the line starts with the file type and 9 permission characters : owner / group / others , each one with r (read) w (write) x (execute) . r=4 , w=2 , x=1 so `rw-r--r--` = 644 . That is why we search with `-user` and `-group` : what we can read depends on who owns the file . `id` shows our user and groups . In find , `-executable` means executable by the current user and `! -executable` negates it .

**See also** : [Bandit level 5](OverTheWire%20Bandit%20Notes.md#bandit-level-5) ( `find` syntax )

[Index](OverTheWire%20Bandit%20Notes.md#index) | [Previous](OverTheWire%20Bandit%20Notes.md#bandit-level-5) | [Next](OverTheWire%20Bandit%20Notes.md#bandit-level-7)

---

# Bandit level 7

[Index](OverTheWire%20Bandit%20Notes.md#index) | [Previous](OverTheWire%20Bandit%20Notes.md#bandit-level-6)

## Command reference

- [Linux shell command reference](../Commands/01%20-%20Linux%20Commands.md)

