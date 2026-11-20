
# Table of Contents

-  [[#1. Windows User Directories]]
-  [[#2. Important Directories]]
-  [[#3. Security Relevance of AppData]]
-  [[#4. EXE vs DLL]]
-  [[#5. Processes and Modules]]
-  [[#6. Investigation Approach]]
-  [[#7. Practical Validation]]

---
## 1. Windows User Directories

Windows stores user-specific application data inside:

`C:\Users\<username>\AppData\`

The directory is divided into three main locations:

|Directory|Purpose|
|---|---|
|`Local`|Machine-specific application data, caches, databases and temporary files|
|`LocalLow`|Data used by applications operating with lower integrity levels or restricted security contexts|
|`Roaming`|User-specific application settings and data that can follow a roaming profile|

### Useful environment variables

```
$env:APPDATA
$env:LOCALAPPDATA
$env:TEMP
```

## 2. Important Directories

Some directories commonly found under `AppData\Local`:

- `Packages`: Data associated with packaged Windows applications.
    
- `Temp`: Temporary files created by applications and Windows.
    
- `VirtualStore`: Compatibility mechanism that redirects certain legacy application writes.
    
- `ConnectedDevicesPlatform`: Data related to Windows connected-device features.
    

## 3. Security Relevance of AppData

User-writable directories can be abused by attackers to store and execute malicious files.

Examples of suspicious activity worth investigating:

- An unexpected executable launched from AppData.
    
- PowerShell scripts executed from unusual user directories.
    
- Unknown processes creating executable files.
    
- A process creating a script and subsequently executing it.
    

The directory alone does not prove malicious activity. I need to investigate the process, user, file, parent process and execution context.

## 4. EXE vs DLL

|EXE|DLL|
|---|---|
|Executable program|Dynamic Link Library|
|Can start as a process|Normally loaded by another process|
|Has its own process context when launched|Executes within the context of its loading process|

A DLL contains reusable code and resources that applications can load and use.

For example, an application can load a DLL to use functionality without implementing everything itself.

## 5. Processes and Modules

A process is a running instance of a program.

A process can load multiple modules, including DLLs.

Example:

```
explorer.exe (PID 4500)
├── ntdll.dll
├── kernel32.dll
├── user32.dll
└── shell32.dll
```

This is an illustrative representation.

DLLs do not normally appear as independent processes in Task Manager because they execute inside the process that loads them.

### Security relevance

Attackers may abuse DLL loading through techniques such as:

- DLL hijacking
    
- DLL injection
    

If a malicious DLL is loaded by a legitimate process, its code can execute within that process context.

A legitimate process name does not guarantee that everything running inside it is legitimate.

## 6. Investigation Approach

When investigating a suspicious executable or DLL, I should examine:

1. File path and location.
    
2. File name and extension.
    
3. Digital signature and hash.
    
4. Process that loaded or executed it.
    
5. User and process privileges.
    
6. Parent-child process relationship.
    
7. Creation and execution timestamps.
    
8. Whether the behaviour is expected for that application.
    

## 7. Practical Validation

I explored the following directories on my Windows 11 lab VM:

- `%APPDATA%`
    
- `%LOCALAPPDATA%`
    
- `%LOCALAPPDATA%\Packages`
    
- `%LOCALAPPDATA%\Temp`
    
- `%LOCALAPPDATA%\Microsoft\OneDrive`
    

I also inspected the modules loaded by `explorer.exe` using PowerShell:

```
Get-Process -Name explorer | Select-Object Name, Id, Path

(Get-Process -Name explorer).Modules |
Select-Object -First 15 ModuleName, FileName
```

### Current understanding

- I understand the distinction between Local, LocalLow and Roaming.
    
- I understand why user-writable directories can be relevant to security investigations.
    
- I understand the difference between an executable process and its loaded DLL modules.
    
- I have inspected the loaded modules of a Windows process.
    

**Status:** Practically explored and conceptually understood at an introductory level. Further investigation practice is required.