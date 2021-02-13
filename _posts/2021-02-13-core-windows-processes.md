# Understanding Core Windows Processes

The following guide will serve as a short introduction to understanding the baseline processes found in a Windows system. This will slowly be expanded as I continue to study. Hopefully it's helpful to you as well. :)

## System Idle Process

You may have noticed this process appears to be using +50% of your CPU -- don't worry there isn't anything going wrong. The *System Idle Process* is exactly what it sounds like; an idling process made by the operating system. The reason for it's operation is to keep the processor occupied with something, else your system might freeze.

Windows runs this process as part of the **SYSTEM** user account, so it’s always active in the background while Windows is running. The **PID** (Process ID) of this should always be **0**.

### A Quick Glance

- **PID**: 0

- **Parent Process**: None
- **Child Processes**: None
- **User**: NT AUTHORITY\SYSTEM (S-1-5-18)
- **Image**: None
- **Number of Instances**: 1

## System 

The **System** “process” is a special kind of process that hosts threads that only run in kernel mode. Modules run under **System** are primarily drivers (.sys files), but also  several important DLLs as well as the kernel executable, `ntoskrnl.exe`.

If you aren't sure what *user mode* and *kernel mode*, follow [this](https://docs.microsoft.com/en-us/windows-hardware/drivers/gettingstarted/user-mode-and-kernel-mode) to get more information.

The **System** process is created by `ntoskrnl.exe` via the process manager function, which creates and terminates processes and threads. There should be no visible parent processes; it's **PID** should always be **4**; and should only be found running in **Session 0**. This process should only have one running instance and one of it's most important jobs is spawning the `smss.exe` process.

### A Quick Glance

- **PID**: 4

- **Parent Process**: None
- **Child Processes**: None
- **User**: NT AUTHORITY\SYSTEM (S-1-5-18)
- **Image**: `%Systemroot%\System32\ntoskrnl.exe` (Task Manager / Process Hacker) and `None` (Process Explorer)
- **Number of Instances**: 1

## Session Manager Subsystem (smss.exe)

As stated in the previous section, this process is spawned from the **System** thread. It is responsible for creating new sessions. This is the first user-mode process started by the kernel and handles creating lists of environment variables, starting user sessions -- creating two instances of itself. The first instance being in **Session 0**, which will handle spawning the `wininit.exe` process, while the latter will be spawned in **Session 1** and will represent the first logged-on user, and create the `winlogon.exe`. Both of these instances will spawn their own `csrss.exe` process.

### A Quick Glance

- **PID**: Random

- **Parent Process**: System
- **Child Processes**: `SMSS.EXE` (**Session 0**), `SMSS.EXE` (**Session 1**), `AUTOCHK.EXE` and a new `SMSS.EXE` instance for each new session
- **User**: NT AUTHORITY\SYSTEM (S-1-5-18)
- **Image**: %Systemroot%\System32\smss.exe
- **Number of Instances**: Multiple during boot-up and only one without arguments after boot-up.

## Client Server Runtime Process (csrss.exe)

This process is the user-mode side of the Windows subsystem. This process is an essential subsystem that must be running at all  times. It is responsible for console windows process/thread creation and thread deletion. As stated before, this process is spawned by `smss.exe` as it manages the start and end of other Windows process and applications.

An instance of `csrss.exe` will run for each session. **Session 0** is for services and **Session 1** for the local console session. Additional  sessions are created through the use of Remote Desktop and/or Fast User  Switching. Each new session results in a new instance of `csrss.exe`.

### A Quick Glance

- **PID**: Random

- **Parent Process**: Orphan process (Parent was the `SMSS.EXE` child process of the master `SMSS.EXE`)
- **Child Processes**: None
- **User**: NT AUTHORITY\SYSTEM (S-1-5-18)
- **Image**: %Systemroot%\System32\csrss.exe
- **Number of Instances**: One for each session. You should find at least two on any running system, one for **Session 0** and another for **Session 1**

## Windows Initialization Process (wininit.exe)

This process is important as it is responsible for launching the Windows Initialization process. This process functions to launch the majority of background applications that are running.  Ontop of setting the default enviroment variables it also handles starting the Service Control Manager (`services.exe`), the Local Security  Authority process (`lsass.exe`), and the Local Session Manager (`lsm.exe`).  It also creates the interactive windows station (`Winsta0`).

### A Quick Glance

- **PID**: Random

- **Parent Process**: Orphan process (Parent was the **Sessions 0** `SMSS.EXE` during boot)
- **Child Processes**: `services.exe`, `lsass.exe`, `fontdrvhost.exe`
- **User**: NT AUTHORITY\SYSTEM (S-1-5-18)
- **Image**: %Systemroot%\System32\wininit.exe
- **Number of Instances**: 1

## Service Control Manager (services.exe)

This process manages the operation of starting and stopping services. It also deals with the automatic starting of services during the computers boot-up and the stopping of services during shut-down. Another important function of this process is that once a user has successfully logged on, the **Service Control Manager** will consider the boot successful and will set the **Last Known Good Control** (or Last Known Good Configuration) and will backup a copy to the registry (`HKLM\SYSTEM\Select\LastKnownGood`).

### A Quick Glance

- **PID**: Random

- **Parent Process**: `wininit.exe`
- **Child Processes**: Multiple (Any services defined in `HKLM/SYSTEM/CurrentControlSet/Services/`. For example: `spoolsv.exe`, `svchost.exe`, `SearchIndexer.exe`, …etc.)
- **User**: NT AUTHORITY\SYSTEM (S-1-5-18)
- **Image**: %Systemroot%\System32\services.exe
- **Number of Instances**: 1

## Service Host Process (svchost.exe)

This process is responsible for hosting and managing Windows services that run from dynamic-link libraries (.DLL files). This process hosts a number of services to lower resource consumption and protect computing resources. 

Look at it like this, if all services ran under a single process, and the process in question crashes, the entire Windows system would go down with it. This is why `svchost.exe`. So not only does this process help with resource consumption, by sharing process with multiple services, it also acts as a guard to make sure one crash doesn't lead to system failures.

### A Quick Glance

- **PID**: Random

- **Parent Process**: `services.exe`
- **Child Processes**: Multiple (Depends on the services being launched.)
- **User**: Multiple (NT AUTHORITY\SYSTEM, NT AUTHORITY\LOCAL SERVICE, NT AUTHORITY\NETWORK SERVICE…etc.)
- **Image**: %Systemroot%\System32\svchost.exe
- **Number of Instances**: Multiple

## Local Security Authority Subsystem Service (lsass.exe)

This process is responsible for enforcing the security policy on the system. It also verifies users logging onto the system, handles password changes, creates access tokens, and manages Active Directory. You can find information regarding any actions performed by this process in the [Windows Security Log](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/view-the-security-event-log).

### A Quick Glance

- **PID**: Random

- **Parent Process**: `wininit.exe`
- **Child Processes**: None (Excluding password filters)
- **User**: NT AUTHORITY\SYSTEM (S-1-5-18)
- **Image**: %Systemroot%\System32\lsass.exe
- **Number of Instances**: 1

## Windows Logon (winlogon.exe)

This process handles many critical task, like, loading the user profile after a user signs in, handling the **Secure Attention Sequence** (SAS), as well as locking the computer and running a screen saver when the system is found to be idle. Handles everything related to user’s logons / logoffs and user initialization. 

**Secure Attention Sequence** (SAS) is a special key combination to be pressed on a keyboard before being presented with a login screen. The secure attention key is designed to make [login spoofing](https://en.wikipedia.org/wiki/Login_spoofing) impossible, as the kernel will suspend any program, including those  masquerading as the computer's login process, before starting a trusted login operation.

This process also monitors for mouse and keyboard activity and will lock the screen or run a screen saver when a previously set amount of inactivity time is met.

### A Quick Glance

- **PID**: Random

- **Parent Process**: `Orphan process (Parent was the `SMSS.EXE` child process with session > 0)`
- **Child Processes**: “LogonUI.exe”, “userinit.exe”, “dwm.exe”, “fontdrvhost.exe” and anything else listed in the “Userinit” value
- **User**: NT AUTHORITY\SYSTEM (S-1-5-18)
- **Image**: %Systemroot%\System32\winlogon.exe
- **Number of Instances**: One for each user session

##  Windows/File Explorer (explorer.exe)

This is the process that gives the user access to their folders and  files. It also provides functionality to other features such as the  Start Menu, Taskbar, etc. It provides a graphical user interface for accessing the file systems.

### A Quick Glance

- **PID**: Random

- **Parent Process**: Orphan process (Parent was the “userinit.exe” process)
- **Child Processes**: Multiple processes
- **User**: Logged on user
- **Image**: %Systemroot%\System32\explorer.exe
- **Number of Instances**: 1 for each user connected on the machine.
