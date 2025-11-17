# Windows System Architecture

![image.png](Image/image.png)

Think of Windows as a **two-floor building**:

- **Top floor → User Mode** (apps, tests, services- safe zone)
- **Bottom floor → Kernel Mode** (OS core- powerful but dangerous)

Windows separates user-mode into different categories:

**1. User Applications**

These are normal apps you run:

- Chrome
- Notepad
- VS Code
- Games

They **cannot talk to hardware directly**.

If they need something (file read, memory allocation, network access), they call **Windows APIs → subsystem DLLs → kernel mode**.

**2. Service Processes**

Services are background processes that start with Windows, e.g.:

- Windows Update Service
- DHCP Client Service
- Print Spooler
- Security Center

These services are user-mode processes but **perform OS-level work**.

**3. System Support Processes**

These are important system processes that Windows uses internally:

Examples:

- **winlogon.exe** → handles login
- **csrss.exe** → Client/Server Runtime Subsystem
- **lsass.exe** → authentication
- **services.exe** → manages all services

They run in user mode but are **critical for Windows to operate**.

**4. Environment Subsystems**

This is SUPER important for Windows architecture.

Windows supports multiple “environments”:

- **Win32 subsystem** → the main one (all your Windows apps)
- **POSIX subsystem** (older versions)
- **WOW64** (x86 apps running on 64-bit Windows)

Environment subsystems **translate app requests into Windows system calls**.

**5. Subsystem DLLs**

These are things like:

- `kernel32.dll`
- `user32.dll`
- `advapi32.dll`
- `gdi32.dll`

They are **the bridge** between user-mode apps and kernel functions.

Example:

When a program calls `CreateFile()`, it actually goes to `kernel32.dll → ntdll.dll → kernel mode`.

So these DLLs act like:

“Hey kernel, the app wants something. Can you handle this?”

### Below the line is where Windows does the real work.

**1. Executive**

This is the bulk of the OS.

It contains different managers:

- **Memory Manager**
- **Process Manager**
- **I/O Manager**
- **Object Manager**
- **Security Reference Monitor**
- **Cache Manager**
- **Plug and Play Manager**

All the high-level OS logic lives here.

You can think of this as:

The brain of the OS.

**2. Kernel (Scheduler, Sync, Low-level CPU work)**

This is the *core of the core*.

The kernel does:

- Thread scheduling
- Interrupt handling
- Context switching
- Low-level synchronization
- Exception dispatching

Think of it as:

The part that directly manages the CPU.

**3. Device Drivers**

These control hardware:

- GPU drivers
- Disk drivers
- Keyboard/mouse drivers
- USB controller drivers

Drivers run in kernel mode because hardware access is dangerous — apps cannot touch them directly.

**4. Hardware Abstraction Layer (HAL)**

The HAL hides hardware differences.

It makes Windows portable across:

- Intel CPUs
- AMD CPUs
- Different motherboard chipsets

Without HAL, Windows would need separate code for every hardware variation.

Think of HAL as:

A translation layer between hardware and the kernel.

**5. Windowing & Graphics**

This is part of the kernel mode in modern Windows:

- GDI
- DirectX kernel parts
- Display drivers

Earlier Windows (XP) had graphics in user mode, but now it’s partially in kernel mode for performance.

![window-system-architecture.png.png](Images\window-system-architecture.png.png)

> The *User mode* is the “safe zone” where apps live,
> 

> The *Kernel mode* is the “power zone” where the OS controls hardware and system resources.
> 

They work together like two halves of a machine.

| **Aspect** | **User Mode** | **Kernel Mode** |
| --- | --- | --- |
| **Purpose** | Runs user applications and system-level processes that do not need direct hardware access. | Runs the core components of the operating system that manage hardware and system resources. |
| **Access Level** | Limited, cannot directly access hardware or kernel memory; must request kernel services via system calls. | Full, has unrestricted access to system memory, devices, and hardware. |
| **Protection** | Isolated and protected, if an app crashes, the rest of the system remains stable. | Not isolated, a crash can cause the entire system to fail (Blue Screen of Death). |
| **Main Components** | **System processes:** e.g., `explorer.exe`, `services.exe` **Environment subsystems:** Win32, POSIX - **User libraries:** `kernel32.dll`, `user32.dll`, `gdi32.dll` | **Executive:** Manages memory, processes, security, and I/O - **Kernel proper:** Handles low-level CPU operations and scheduling - **Device drivers:** Interface between OS and hardware - **HAL (Hardware Abstraction Layer):** Translates hardware differences for the OS |
| **How It Works** | Applications call functions in user libraries (like `kernel32.dll`), which act as messengers to the kernel. | Executes privileged instructions and manages direct communication with hardware components. |
| **Key File / Core Component** | User libraries and processes. | `ntoskrnl.exe` - Windows NT Operating System Kernel (the “core brain” of Windows). |
| **Stability Impact** | Application-level failures are contained; OS remains stable. | Critical, errors here usually cause a system crash (BSOD). |

### **EXPERIMENT: Kernel Mode vs. User Mode**

1. Press **Win + R**, type:

*perfmon*

Press **Enter**.

ii. On the left side you will see a tree.

1. Expand **Monitoring Tools**.
2. Click **Performance Monitor.**

You should now see a live graph.

iii. Add **% Privileged Time** and **% User Time**.

1. On the Performance Monitor window, look at the toolbar at the top.
    
    Click the **green “+” icon** (Add Counters).
    
2. A new window opens.
3. Scroll down and find **Processor** (not Processor Information — just Processor).
4. Click **Processor** to expand it.
5. Under **Counters**, scroll inside the list and:

Click **% Privileged Time and** Click **% User Time**

iv. Select **_Total** (so it measures total CPU usage).

1. Click **Add and then** Click **OK**.

The graph now shows two new lines: privileged time and user time.

v.  Open **Command Prompt and r**un this command: 

*dir \\%computername%\c$*

Here:

`%computername%` is an environment variable that returns your machine’s name.

`c$` is the **administrative hidden share** for the C drive.

Switch back to *Performance Monitor*.

**% Privileged Time spike** → kernel-mode operations (SMB, file system driver, etc.)

**% User Time spike** → any user-mode work (command shell, etc.)

![Screenshot 2025-11-17 124724.png](Images\Screenshot_2025-11-17_124724.png)

### **Subsystems (Win32, POSIX, etc)**

In easy words Subsystems can be referred to as “The personalities of Windows.”

Originally, Windows NT supported *multiple environments*, not just Windows GUI apps.

Each subsystem allows programs of a certain type to run:

- **Win32 subsystem** – for normal Windows apps (most common).
- **POSIX subsystem** – for UNIX-style applications (used in older versions, now replaced by WSL).
- **OS/2 subsystem** – supported legacy OS/2 apps (no longer used).

These subsystems interpret the app’s API calls and translate them into **system calls** that the kernel can understand.

### **System Calls (User → Kernel Transition)**

“How apps talk to the kernel safely.”

Here’s what happens when your app does something like read a file:

1. Your program calls a Win32 API → e.g., `ReadFile()`
2. That call goes to a system DLL (like `kernel32.dll` → `ntdll.dll`).
3. `ntdll.dll` performs a **system call** (like `NtReadFile`) to switch into **kernel mode**.
4. The **kernel (ntoskrnl.exe)** executes the actual file read operation.
5. The result travels back up to your program.

It’s like going through airport security you cross from the public zone (user mode) to the restricted zone (kernel mode), get your work done, and return.

### **What ntoskrnl.exe and smss.exe do?**

### **ntoskrnl.exe (Windows NT Operating System Kernel)**

“The heart of Windows.”

It’s the **main kernel executable** the *brain* managing:

Memory (paging, virtual memory), Processes and threads, Scheduling, I/O and Security checks.

Without it, the OS simply **cannot run**.

It loads during boot and stays running forever.

### **smss.exe (Session Manager Subsystem)**

“The startup conductor of Windows.”

`smss.exe` is **one of the first user-mode processes** started by the kernel.

It creates *system sessions*, starts *critical system processes* like:

`csrss.exe` (Client/Server Runtime Subsystem handles console, GUI threads)

`wininit.exe` (starts services, `lsass.exe`, etc.)

It also sets up environment variables and paging files.

You can think of `smss.exe` as the *organizer* that sets the stage for Windows to actually start working after the kernel is up.