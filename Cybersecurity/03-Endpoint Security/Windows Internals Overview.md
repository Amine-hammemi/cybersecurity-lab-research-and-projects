# Windows Internals Overview

> My introductory notes on how Windows runs programs and services. This knowledge is important when investigating endpoint activity in a SOC.

## 1. Program vs process

- A **program** is a file containing executable code, for example `notepad.exe`.
- A **process** is a running instance of a program.
- A process has a PID (Process ID), memory space, security context and other resources.
- Running the same program twice can create two separate processes.

Example:

```text
notepad.exe  -> process with PID 4200
notepad.exe  -> another process with PID 5312
```

The PIDs above are examples, not values from my VM.

## 2. Processes and threads

- A process is a container for resources and execution.
- A **thread** is an execution unit inside a process.
- A process normally has at least one thread and can have multiple threads.
- Threads within a process share its address space and many process resources.

For SOC investigations, process identity and behaviour are often more useful than the executable name alone.

## 3. Parent and child processes

A process can start another process.

Example:

```text
explorer.exe
└── notepad.exe
```

- `explorer.exe` is the parent.
- `notepad.exe` is the child.
- Parent-child relationships help reconstruct how an activity started.

A process tree can reveal suspicious execution chains, but an unusual-looking relationship is not proof of malware by itself.

## 4. Windows services

- A **Windows service** is a background component that provides system or application functionality.
- Services are managed by the Service Control Manager (SCM).
- A service may run inside its own process or share a host process with other services.
- Service configuration includes information such as its name, startup type and executable path.

Useful PowerShell commands:

```powershell
Get-Service
Get-Service -Name WazuhSvc
Get-CimInstance Win32_Service
```

Inspect which service is associated with a PID:

```powershell
Get-CimInstance Win32_Service -Filter "ProcessId = 4436" |
    Select-Object Name, DisplayName, State, ProcessId, PathName
```

Replace `4436` with the PID being investigated.

## 5. What is svchost.exe?

- `svchost.exe` is a legitimate Windows executable used to host services implemented in DLLs.
- Multiple `svchost.exe` processes can run at the same time.
- Seeing `svchost.exe` in an alert is not automatically suspicious.
- I should identify the service hosted by the process and inspect its command line, parent process, file path and signature.

Useful commands:

```powershell
tasklist /svc
```

```powershell
Get-CimInstance Win32_Service |
    Where-Object ProcessId -eq 4436 |
    Select-Object Name, DisplayName, State, ProcessId, PathName
```

## 6. DLLs

- A **DLL (Dynamic-Link Library)** contains code or resources that programs can load and use.
- Multiple applications can use shared libraries.
- Loading a DLL is normal Windows behaviour.
- Unexpected DLL paths, suspicious loading relationships or unusual timing can be investigation leads.

Sysmon Event ID 7 can record image/DLL loading when enabled by the Sysmon configuration.

## 7. Windows Registry

- The Registry is a hierarchical database used by Windows and applications to store configuration.
- It contains keys and values.
- Registry changes can be legitimate or security-relevant.
- Persistence mechanisms sometimes use Registry locations to make programs run automatically.

Sysmon Events 12, 13 and 14 can record selected Registry activity.

## 8. Tools I can use

| Tool | Use |
|---|---|
| Task Manager | Basic process and resource inspection |
| Process Explorer | Detailed process tree, handles and loaded DLLs |
| Services (`services.msc`) | Inspect and manage services |
| PowerShell | Query processes, services and system information |
| Event Viewer | Inspect Windows event channels |

## 9. What I have practised

- Used PowerShell to identify the Windows service associated with a process ID.
- Investigated an alert involving `svchost.exe`.
- Learned that a process name alone is insufficient to decide whether activity is malicious.

## 10. What I still need to practise

- Reading process trees in Process Explorer.
- Understanding process command lines and parent processes.
- Inspecting service executable paths and signatures.
- Understanding Windows security contexts and permissions.
- Correlating process activity with Windows Event Logs and Sysmon.

## Command reference

- [PowerShell command reference](../../Commands/04%20-%20PowerShell%20Commands.md)
- [Windows utilities command reference](../../Commands/05%20-%20Windows%20%26%20Sysmon%20Utilities.md)

## Related notes

- [Windows Event Logs](Windows%20Event%20Logs.md)
- [Sysmon](Sysmon.md)
- [SIEM Fundamentals](../04-Security%20Monitoring/SIEM%20Fundamentals.md)
- [Wazuh Architecture](../04-Security%20Monitoring/Wazuh%20Architecture.md)

## Official resources

- [Microsoft Sysinternals](https://learn.microsoft.com/en-us/sysinternals/)
- [Process Explorer](https://learn.microsoft.com/en-us/sysinternals/downloads/process-explorer)
- [Windows Services](https://learn.microsoft.com/en-us/windows/win32/services/services)
- [Windows Registry overview](https://learn.microsoft.com/en-us/windows/win32/sysinfo/registry)
