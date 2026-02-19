## to check uac settings 
```
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System
```
# Scheduled Tasks & Services
  ### Items to evaluate include:
```
.Write Permissions

.Non-Standard Locations

.Unquoted Executable Paths

.Vulnerabilities in Executables

.Permissions to Run As SYSTEM
```
## command vuln schtasks
```
schtasks /query /fo LIST /v

open task schedular and look at triggers to determin what is causing it to open
-you can look at  actions to see what this task is doing
you can take advantage of a pre-existing schtask by naming a maliciouse file to what ever is already scheduled to run ,
    -you must insure you change the name of the clean file first so that you can name your maliciouse file the clean name
```
## vuln serveices
```
wmic service list full
sc query


services app and filter by description
an empty description should be lookd into
when you go on file explorerm click on view and check hidden items
```
## to check system resources
```
wmic
net
netstat
```
## show audit category setings 
```
auditpol /get /category:*
```
## Audit policy command 
```
auditpol /get /category:* | findstr /i "success failure"
- insure you have proper permissions to run this command, must run powershell as admin
```
##  event logs 
```
Get-Eventlog -List

or event viewer app and go to dates( you can go to create costume range on the right side of the page in order to limit the time frame
```
## determine if logging is set 
```
reg query [hklm or hkcu]\software\policies\microsoft\windows\powershell
reg query hklm\software\microsoft\wbem\cimom \| findstr /i logging
# 0 = no | 1 = errors | 2 = verbose
```
## KEYS AND SUBKEYS TO CHECK 
```
-HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\
--Run
--RunOnce

-HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\
--Run
--RunOnce
```
