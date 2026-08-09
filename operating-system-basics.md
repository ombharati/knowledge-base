# **What an operating system actually does and why computers need one.**

## 1. What is an operating system?

An **operating system**, usually shortened to **OS**, is the main system software that manages a computer.

Examples include:

* Windows
* macOS
* Linux
* Android
* iOS

The operating system sits between:

1. The computer’s physical hardware
2. The applications you use

```text
┌─────────────────────────────┐
│ Applications                │
│ Browser, editor, games      │
├─────────────────────────────┤
│ Operating system            │
│ Windows, Linux, Android     │
├─────────────────────────────┤
│ Hardware                    │
│ CPU, memory, disk, devices  │
└─────────────────────────────┘
```

Applications normally do not control hardware directly. They ask the operating system to perform operations on their behalf.

---

## 2. Why does an operating system exist?

A computer contains many hardware components:

* A **CPU**, which executes instructions
* **Memory**, which temporarily stores active information
* Storage devices, such as SSDs
* A keyboard and mouse
* A display
* Network hardware
* Speakers, cameras, printers, and other devices

Without an operating system, every application would need to understand and control all these devices itself.

For example, a text editor would need to know:

* How to communicate with every model of keyboard
* How to draw characters using every type of graphics hardware
* How to locate free space on every type of storage device
* How to communicate through every type of network adapter
* How to avoid interfering with other running programs

That would make applications extremely complicated and unsafe.

The operating system provides a common, controlled interface.

Instead of saying:

> “Send these exact electrical commands to this particular SSD controller.”

An application can request:

> “Create a file named `notes.txt` and store this information in it.”

The operating system handles the hardware-specific details.

---

## 3. The problems an operating system solves

An operating system solves several major problems.

### Problem 1: Hardware is difficult to control

Different devices behave differently.

A keyboard, disk, display, and network card each use different communication methods. Applications should not need to understand every device model.

The operating system provides simpler concepts:

| Hardware complexity          | OS-provided concept            |
| ---------------------------- | ------------------------------ |
| Disk sectors and controllers | Files and directories          |
| Physical memory chips        | Process memory                 |
| CPU execution details        | Processes and threads          |
| Network adapter commands     | Network connections            |
| Display hardware             | Windows and graphical surfaces |

This process of hiding complicated details behind a simpler interface is called **abstraction**.

### Abstraction

An abstraction presents a simpler way to use something complicated.

For example, a car driver uses:

* A steering wheel
* An accelerator
* A brake pedal

The driver does not directly control:

* Fuel injection timing
* Individual wheel forces
* Engine valve movement

The controls form an abstraction over the vehicle’s machinery.

Similarly, files are an abstraction over storage hardware.

---

### Problem 2: Multiple programs need the same hardware

Suppose these programs are running together:

* A browser
* A music player
* A messaging application
* A file download
* An antivirus scanner

They may all need:

* CPU time
* Memory
* Disk access
* Network access
* Display access

The operating system decides how these resources are shared.

```text
                ┌───────────────┐
Browser ───────▶│               │
Music player ──▶│ Operating     │────▶ CPU
Chat app ──────▶│ system        │────▶ Memory
Downloader ────▶│ resource      │────▶ Disk
Scanner ───────▶│ management    │────▶ Network
                └───────────────┘
```

The OS prevents one cooperative program from accidentally taking everything.

It also attempts to limit damage from programs that behave incorrectly.

---

### Problem 3: Programs must be isolated

Imagine two programs stored their information in the same memory without restrictions.

A game could accidentally overwrite:

* Your browser’s passwords
* A document being edited
* Part of the operating system
* Another application’s instructions

The operating system gives programs controlled, separated environments.

This is called **isolation**.

A program normally cannot directly read or modify another program’s private memory.

```text
┌──────────────────────────┐
│ Browser memory           │
│ Tabs, history, page data │
└──────────────────────────┘
          separated
┌──────────────────────────┐
│ Music player memory      │
│ Audio data, playlists    │
└──────────────────────────┘
          separated
┌──────────────────────────┐
│ OS-protected memory      │
│ Kernel and system data   │
└──────────────────────────┘
```

The separation is not merely organizational. Hardware and operating-system rules actively enforce it.

---

### Problem 4: Applications need standard services

Applications frequently need to:

* Open files
* Create network connections
* Display information
* Read keyboard input
* Start other programs
* Request memory
* Obtain the current time
* Communicate with other programs

The OS provides these common services.

An application makes a request to the operating system. The operating system checks the request, performs the operation, and returns a result.

```text
Application
    │
    │ Request: "Read this file"
    ▼
Operating system
    │
    │ Checks permissions and controls storage
    ▼
Storage device
    │
    │ Returns requested data
    ▼
Operating system
    │
    │ Gives data to the application
    ▼
Application
```

Later, we will examine these requests as **system calls**.

---

## 4. A simple mental model: the operating system as a manager

A useful beginner mental model is:

> The operating system is the computer’s resource manager and rule enforcer.

It manages resources such as:

* Processor time
* Memory
* Storage
* Input and output devices
* Network access

It enforces rules such as:

* Which program can access a file
* Which program owns a region of memory
* Which user may perform an action
* When a program may use the CPU
* Whether a request is valid

### Office-building analogy

Imagine a large office building.

| Office building       | Computer system   |
| --------------------- | ----------------- |
| Workers               | Applications      |
| Office rooms          | Memory regions    |
| Meeting-room schedule | CPU scheduling    |
| Filing room           | Storage           |
| Reception desk        | OS interface      |
| Security staff        | Permission checks |
| Maintenance staff     | Device management |
| Building manager      | Operating system  |

Workers do not usually repair electrical wiring, alter security systems, or seize another worker’s office.

They make requests through controlled procedures.

The building manager coordinates shared resources and enforces boundaries.

---

## 5. The operating system is not the entire computer

These terms are often incorrectly treated as interchangeable.

| Term                 | Meaning                                         |
| -------------------- | ----------------------------------------------- |
| **Hardware**         | Physical electronic components                  |
| **Operating system** | Software that manages hardware and applications |
| **Application**      | Software that performs tasks for a user         |
| **Computer system**  | Hardware, OS, applications, and data together   |

A laptop is not “the operating system.”

Windows or Linux is not the physical computer.

A browser is not normally part of the core operating system, even when it comes preinstalled.

---

## 6. The operating system’s main responsibilities

### 6.1 CPU management

The **CPU**, or central processing unit, executes instructions.

Many programs may appear to run simultaneously, but a CPU core normally executes one instruction stream at a particular instant.

The operating system repeatedly decides:

* Which program runs
* On which CPU core
* For how long
* Which program waits

This is called **CPU scheduling**.

```text
Time ─────────────────────────────────────────▶

CPU core:
[Browser][Music][OS][Browser][Chat][Music][OS]
```

The switches can happen so quickly that the programs appear to run together.

---

### 6.2 Memory management

Programs need memory for:

* Their instructions
* Documents and images
* Temporary calculations
* Interface elements
* Internal data

The OS tracks:

* Which memory is available
* Which program is using which memory
* Whether a program may access a location
* What happens when physical memory is insufficient

```text
Physical memory
┌──────────────────────────┐
│ Operating system         │
├──────────────────────────┤
│ Browser                  │
├──────────────────────────┤
│ Music player             │
├──────────────────────────┤
│ Available memory         │
└──────────────────────────┘
```

This diagram is simplified. Modern operating systems use virtual memory, paging, caching, and many other mechanisms that we will examine later.

---

### 6.3 Storage and file management

Storage hardware fundamentally deals with blocks of data.

Humans and applications prefer concepts such as:

* Files
* Names
* Folders
* Paths
* Permissions

The operating system organizes storage through a **file system**.

```text
Storage blocks
      │
      ▼
File system organization
      │
      ▼
Documents/
├── report.pdf
├── notes.txt
└── images/
    └── diagram.png
```

The OS also determines who may read, change, create, or delete files.

---

### 6.4 Device management

Hardware devices often require specialized control software called **device drivers**.

A driver translates general operating-system requests into commands understood by a particular device.

```text
Application
    │ "Play this audio"
    ▼
Operating system
    │
    ▼
Audio driver
    │ Device-specific commands
    ▼
Audio hardware
```

Drivers allow the OS to support many device models without every application needing device-specific knowledge.

---

### 6.5 Security and access control

The operating system helps answer questions such as:

* Which user is signed in?
* May this user open this file?
* May this application use the camera?
* May this process change system settings?
* May this program access another process?

Security is not handled by one single component. It depends on permissions, isolation, authentication, hardware protection, and application behavior working together.

---

### 6.6 Providing an environment for applications

An application needs more than a CPU.

It needs an environment containing concepts such as:

* Memory
* Files
* Input and output
* Time
* Communication
* Error reporting
* Access permissions

The OS creates and manages that environment.

---

## 7. How an application uses the operating system

Consider opening a photo in an image viewer.

### Step-by-step behavior

1. You select the photo.
2. The graphical interface detects your input.
3. The OS determines which application should open the file.
4. The OS starts the image-viewer program.
5. The OS gives the new program CPU time.
6. The OS provides memory for the program.
7. The program asks the OS to open the photo file.
8. The OS checks whether access is allowed.
9. The file system locates the file’s data.
10. A storage driver communicates with the storage hardware.
11. The data is placed into memory.
12. The image viewer interprets the image format.
13. The program asks the OS to display the result.
14. The graphics system and driver communicate with display hardware.

```text
User
 │
 ▼
Graphical interface
 │
 ▼
Operating system starts image viewer
 │
 ├──▶ CPU: execute program
 ├──▶ Memory: hold program and image
 ├──▶ File system: locate image
 ├──▶ Storage driver: read data
 └──▶ Graphics system: display image
```

Even this ordinary action involves several operating-system responsibilities.

---

## 8. Operating systems provide both abstractions and policies

Two ideas help explain much of OS design.

### Mechanism

A **mechanism** is a capability that makes something possible.

Examples:

* Switching the CPU from one process to another
* Preventing access to protected memory
* Reading blocks from a disk
* Sending information to a network device

### Policy

A **policy** is a rule for deciding how a mechanism should be used.

Examples:

* Which process should run next?
* Which user may read a file?
* How much memory may an application use?
* Which network traffic should be blocked?

| Question                           | Category  |
| ---------------------------------- | --------- |
| How can the CPU switch programs?   | Mechanism |
| Which program should run next?     | Policy    |
| How can file access be restricted? | Mechanism |
| Which users should have access?    | Policy    |

This distinction will reappear throughout operating-system design.

---

## 9. Simplified model versus technical reality

### Simplified mental model

> Applications ask the operating system to use hardware.

This is useful and mostly correct.

### More exact reality

Several complications exist:

* Some application instructions execute directly on the CPU.
* The OS does not interpret every instruction an application performs.
* Hardware participates in enforcing memory and privilege boundaries.
* Some devices can transfer data without the CPU copying every individual byte.
* Applications may use libraries that make OS requests on their behalf.
* Some graphical and computational work may be sent directly to specialized hardware through controlled interfaces.

A more precise model is:

> Applications execute on the hardware under restrictions established and managed by the operating system.

The OS intervenes when privileged actions, shared resources, protection, scheduling, or hardware coordination are required.

---

## 10. What can go wrong?

### A program consumes too much CPU time

Other programs may become slow or unresponsive.

The operating system’s scheduler attempts to distribute CPU time, but a heavily loaded system can still feel slow.

---

### A program consumes too much memory

Possible results include:

* Reduced performance
* Applications being closed
* Operations failing
* The entire system becoming unresponsive

The exact behavior depends on the operating system and its memory-management policies.

---

### A device driver fails

Because drivers interact closely with hardware and often have elevated privileges, a serious driver failure can affect the whole system.

Possible symptoms include:

* Device malfunction
* Frozen system
* Data corruption
* Complete system crash

---

### Storage becomes corrupted

A sudden power loss, failing storage device, software defect, or unsafe removal can damage file-system information.

The OS may no longer know:

* Where a file’s data is located
* Which storage blocks are free
* Whether a directory entry is valid

---

### Isolation fails

If a program escapes its restrictions, it may access information or resources it should not control.

This is one reason operating-system security vulnerabilities can be serious.

---

### The operating system itself crashes

Applications depend on OS services. If a critical part of the operating system fails, the whole system may stop.

Examples include:

* A Windows stop error
* A Linux kernel panic
* A mobile-device system restart

---

## 11. Common misconceptions

### Misconception: “The operating system runs every application instruction”

Not exactly.

Applications usually execute their ordinary instructions directly on the CPU. The OS controls their environment, schedules them, and handles privileged requests.

---

### Misconception: “The OS is just the graphical desktop”

The desktop, windows, icons, and menus are only the visible interface.

A system can have an operating system without a graphical desktop. Many servers operate primarily through text-based interfaces.

---

### Misconception: “More applications means the CPU literally executes all of them at the same instant”

Not necessarily.

A single CPU core switches rapidly among tasks. Multiple cores can execute multiple instruction streams at the same time, but there may still be more runnable tasks than cores.

---

### Misconception: “An application owns the hardware while it runs”

Normally, it temporarily receives controlled access to resources.

The OS remains responsible for scheduling, isolation, permissions, and device coordination.

---

### Misconception: “The OS prevents every crash”

The OS can limit damage and detect some failures, but applications can still contain errors and crash.

The OS itself may also contain defects.

---

### Misconception: “A file is stored as one continuous object exactly as it appears in a folder”

A file is a logical abstraction.

Its data may be divided into separate storage blocks, cached in memory, duplicated for reliability, compressed, encrypted, or stored remotely.

---

## 12. Core mental model

Keep this model for the sections ahead:

```text
Users express intentions
          │
          ▼
Applications perform tasks
          │
          ▼
Operating system manages and protects resources
          │
          ▼
Hardware performs physical computation and I/O
```

The layers have different responsibilities:

| Layer            | Main responsibility                  |
| ---------------- | ------------------------------------ |
| User             | Chooses goals and actions            |
| Application      | Implements a specific task           |
| Operating system | Coordinates, abstracts, and protects |
| Hardware         | Performs physical operations         |

The OS is not merely another application. It is the controlled environment within which ordinary applications operate.

---

# Learning Check

Do not look for answers yet. Attempt each question using the mental models above.

## Conceptual questions

1. What are the two main roles of an operating system as a resource manager and rule enforcer?
2. What does it mean for the OS to provide an abstraction?
3. Why are files considered an operating-system abstraction rather than raw storage hardware?

## Cause-and-effect questions

4. Why could allowing every application to access all physical memory make the system unreliable?
5. What problems could occur if no component decided how CPU time should be shared?

## Misconception question

6. A student says, “The operating system executes every instruction for every application.” What is misleading about this statement?

## Scenario-based question

7. A browser, music player, and file downloader are running together. The music begins skipping while a large file is being downloaded. Identify at least three shared resources that may be involved, and explain what role the operating system plays.

# 2. Hardware, the Kernel, and User Applications

The previous section described the operating system as a manager between applications and hardware.

We now need a more precise model:

```text
┌──────────────────────────────────────┐
│ User applications                    │
│ Browser, editor, game, music player  │
├──────────────────────────────────────┤
│ Operating-system services            │
├──────────────────────────────────────┤
│ Kernel                               │
│ Scheduling, memory, devices, safety  │
├──────────────────────────────────────┤
│ Hardware                             │
│ CPU, RAM, storage, network, display  │
└──────────────────────────────────────┘
```

The three central parts are:

| Part                  | Main role                                                |
| --------------------- | -------------------------------------------------------- |
| **Hardware**          | Physically performs computation and input/output         |
| **Kernel**            | Controls hardware and manages protected system resources |
| **User applications** | Perform tasks for users within OS-imposed limits         |

---

## 2.1 Hardware

### What it is

**Hardware** means the physical components of a computer.

Important examples include:

* CPU
* Main memory, commonly called RAM
* Storage devices
* Keyboard and mouse
* Display hardware
* Network adapter
* Audio hardware
* Timers and other controllers

Hardware is capable of performing operations, but it does not independently understand concepts such as:

* Users
* Documents
* Applications
* Folders
* Windows
* Permissions

Those are software-created concepts.

---

## 2.2 Major hardware components

### CPU

The **central processing unit**, or CPU, executes machine instructions.

Its work includes:

* Performing arithmetic
* Comparing values
* Moving information
* Following decisions
* Communicating with memory
* Responding to special hardware events

A CPU does not inherently know that it is running a browser or music player. It sees instructions and data.

### Mental model

Think of the CPU as a worker who follows extremely small instructions very quickly.

```text
Instruction 1 → Instruction 2 → Instruction 3 → Instruction 4
```

The CPU performs the work, while the operating system controls which program receives opportunities to use it.

---

### Main memory

**Main memory**, usually called **RAM**, holds information that the CPU needs while programs are running.

It may contain:

* Program instructions
* Application data
* Operating-system data
* Recently accessed file contents
* Temporary results

RAM is much faster to access than long-term storage, but its contents normally disappear when power is removed.

### Simplified mental model

```text
Storage: long-term filing cabinet
RAM:     active workbench
CPU:     worker using the workbench
```

A program stored on an SSD generally must have relevant parts loaded into RAM before the CPU can execute them.

---

### Storage

Storage devices preserve information for longer periods.

Examples include:

* Solid-state drives
* Hard-disk drives
* USB drives
* Memory cards

Storage may contain:

* Operating-system files
* Installed applications
* User documents
* Images and videos
* Configuration information

The CPU usually cannot treat storage exactly like RAM. Data must be transferred through storage controllers and operating-system mechanisms.

---

### Input devices

Input devices provide information to the computer.

Examples:

* Keyboard
* Mouse
* Touchscreen
* Microphone
* Camera
* Sensors

The operating system receives and interprets device events before delivering useful information to applications.

For example, a keyboard produces hardware-level events. The OS helps turn them into concepts such as:

> “The user pressed the letter A while this application had keyboard focus.”

---

### Output devices

Output devices present results.

Examples:

* Display
* Speakers
* Printer
* Haptic motor

Applications usually do not control these devices freely. They use OS-controlled interfaces.

---

### Hardware controllers

Many devices are managed through electronic components called **controllers**.

For example:

* A storage controller manages communication with an SSD.
* A graphics controller manages graphics operations.
* A USB controller manages connected USB devices.

The CPU communicates with these controllers using hardware-defined rules.

The operating system must understand those rules, often through device drivers.

---

## 2.3 Why applications do not directly control everything

Suppose a browser could directly control the physical display, keyboard, memory, disk, and network adapter without restrictions.

Several problems would appear.

### Conflict

Two applications might try to control the same device at once.

```text
Browser ──────┐
              ├──▶ Display hardware
Video player ─┘
```

Without coordination, their commands could interfere.

---

### Security

A malicious application might:

* Read keyboard input intended for another application
* Access another program’s memory
* Read private files
* Turn on the camera
* Modify operating-system data

---

### Reliability

A defective application might send invalid commands and leave a device unusable.

---

### Hardware complexity

Every application would need separate support for thousands of hardware models.

---

### Portability

An application written for one machine might fail on another because the hardware differs.

The operating system solves these problems by controlling access and providing standard interfaces.

---

# 2.4 The Kernel

## What it is

The **kernel** is the privileged core of an operating system.

It performs the most sensitive system-management tasks, including:

* CPU scheduling
* Process management
* Memory protection
* Device access
* Interrupt handling
* Communication between programs
* Permission enforcement
* System-call handling

The kernel normally remains active from early in the boot process until the computer shuts down.

---

## Why it exists

Some component must have the authority to:

* Control hardware
* Enforce isolation
* Decide which program runs
* Protect memory
* Validate application requests
* Recover resources when programs stop

That component is the kernel.

An ordinary application cannot safely be trusted with all these powers.

---

## The problem it solves

Without a protected kernel, every application would need to cooperate perfectly.

One faulty or malicious program could:

* Take all CPU time
* Overwrite another program
* Change security rules
* Read every file
* Disable hardware
* Crash the entire system

The kernel acts as a trusted control layer.

---

## Mental model: the kernel as an air-traffic controller

An airport contains many aircraft that need shared infrastructure:

* Runways
* Airspace
* Gates
* Fuel services
* Navigation systems

Individual pilots cannot safely decide all access themselves.

Air-traffic control coordinates them and enforces rules.

| Airport                | Computer            |
| ---------------------- | ------------------- |
| Aircraft               | Applications        |
| Runway time            | CPU time            |
| Gates                  | Memory and devices  |
| Air-traffic controller | Kernel              |
| Flight request         | System call         |
| Aviation rules         | Protection policies |

The kernel does not perform every task that applications perform. It coordinates access to shared system resources.

---

## The kernel is not the entire operating system

This distinction is important.

### Kernel

The protected core that manages hardware and fundamental resources.

### Operating system

A broader collection that may include:

* The kernel
* System libraries
* Background services
* Command-line tools
* Graphical interface components
* Administrative utilities
* File-management tools

```text
Operating system
├── Kernel
├── System services
├── System libraries
├── Device-management tools
├── User interface
└── Utilities
```

Linux is technically the name of a kernel, although people often use “Linux” to refer to an entire operating-system distribution.

Windows and macOS also include kernels, but their operating systems contain much more than those kernels.

---

# 2.5 User Applications

## What they are

A **user application** is software designed to perform a task for a person or another program.

Examples:

* Web browser
* Text editor
* Game
* Video player
* Calculator
* Development environment
* Messaging application

They are called “user applications” partly because they run outside the kernel’s highly privileged environment.

The term does not mean the application is always visibly controlled by a human.

---

## Why applications are separate from the kernel

Applications vary greatly and may contain defects.

Keeping them outside the kernel provides several benefits:

* A browser crash does not normally crash the OS.
* A game cannot normally inspect another application’s private memory.
* Applications can be started and stopped independently.
* Different permissions can be assigned to different programs.
* Application development does not require modifying the kernel.

---

## What an application receives from the OS

When an application runs, the operating system gives it a controlled environment containing:

* CPU execution time
* A private-looking memory space
* Access to permitted files
* Access to approved devices
* Ways to communicate with other programs
* Ways to request OS services

This environment is powerful but restricted.

---

## What an application can do by itself

An application can perform many ordinary calculations without asking the kernel every time.

For example, it can:

* Add numbers
* Compare text
* Reorganize information already in its memory
* Decode an image already loaded into memory
* Update its internal state

These instructions usually execute directly on the CPU.

---

## What requires kernel involvement

An application generally needs kernel assistance when it wants to:

* Open or read a file
* Request additional memory
* Create a process
* Send network data
* Read keyboard input
* Access a device
* Wait for an event
* Communicate through protected OS mechanisms

These actions affect resources controlled by the operating system.

---

# 2.6 The Boundary Between Applications and the Kernel

Applications and the kernel do not usually interact through arbitrary access.

They use carefully defined request mechanisms.

A simplified flow looks like this:

```text
User application
       │
       │ Request an OS service
       ▼
Kernel checks the request
       │
       ├── Is the request valid?
       ├── Is access permitted?
       ├── Is the resource available?
       └── What hardware action is required?
       │
       ▼
Kernel performs or coordinates the operation
       │
       ▼
Result returned to application
```

Later, we will call these controlled requests **system calls**.

---

## Example: application reads a file

Suppose a photo viewer wants to read `holiday.jpg`.

### Step 1: The application requests the file

The application asks the operating system to open the file.

The application uses a filename, not raw storage-sector commands.

**Responsible component:** Application and OS interface

---

### Step 2: The request enters the kernel

The kernel receives the request through a controlled entry point.

**Responsible component:** Kernel

---

### Step 3: The kernel checks permissions

The kernel checks whether the current process is allowed to access the file.

**Responsible component:** Security and file-system mechanisms

---

### Step 4: The file system locates the data

The OS determines where the file’s data is stored.

**Responsible component:** File-system code

---

### Step 5: A device driver communicates with storage

The appropriate driver sends device-specific instructions to the storage controller.

**Responsible component:** Device driver

---

### Step 6: The hardware retrieves the data

The storage device reads the required data.

**Responsible component:** Storage hardware and controller

---

### Step 7: The data reaches memory

The requested data is transferred into RAM under OS control.

**Responsible components:** Hardware, driver, and kernel memory management

---

### Step 8: The application receives access to the data

The kernel reports success and allows the application to use the retrieved information.

**Responsible component:** Kernel and application

---

### Full path

```text
Photo viewer
     │
     │ “Read holiday.jpg”
     ▼
Kernel
     │
     ├── Check permission
     ├── Find file data
     └── Request storage operation
     ▼
Storage driver
     │
     ▼
Storage controller and SSD
     │
     │ File data
     ▼
RAM
     │
     ▼
Photo viewer
```

---

# 2.7 Device Drivers

## What a driver is

A **device driver** is software that allows the operating system to communicate with a particular type of hardware.

Hardware devices have device-specific rules.

A driver understands details such as:

* Which commands a device accepts
* How to configure it
* How to send or receive data
* How the device reports completion
* How to handle errors

---

## Why drivers exist

The operating system wants to provide general operations such as:

* Read data
* Play sound
* Display an image
* Send a network packet

Different hardware implements these actions differently.

Drivers translate general OS operations into device-specific instructions.

```text
General OS request
“Send this network data”
          │
          ▼
Network driver
“Use these device-specific commands”
          │
          ▼
Network adapter
```

---

## Driver mental model: a translator

Imagine two people who do not speak the same language.

* The kernel speaks in general device operations.
* The device speaks in hardware-specific commands.
* The driver translates between them.

This analogy is useful, although a driver does more than language translation. It also manages device state, timing, errors, and coordination.

---

## Are drivers part of the kernel?

There is no single universal answer.

Depending on the operating system and driver design, a driver may run:

* Inside the kernel
* Partly inside and partly outside the kernel
* In a less-privileged user-space service

Many traditional drivers run with kernel privileges because they need close hardware access.

This improves performance and convenience but increases risk: a faulty privileged driver can damage the entire operating system.

---

# 2.8 System Libraries and Background Services

The application-to-kernel model is useful, but real systems often contain additional layers.

```text
┌─────────────────────────────────┐
│ Application                     │
├─────────────────────────────────┤
│ Libraries and system services   │
├─────────────────────────────────┤
│ Kernel                          │
├─────────────────────────────────┤
│ Drivers and hardware            │
└─────────────────────────────────┘
```

---

## System libraries

A **library** is reusable software that applications can use.

A system library may:

* Provide convenient file operations
* Handle text and graphics
* Prepare system-call requests
* Hide operating-system-specific details

An application may call a library function, and the library may then request kernel assistance.

```text
Application
    │
    ▼
Library
    │
    ▼
Kernel
```

The library is not necessarily part of the kernel.

---

## Background services

A **background service** is a program that runs without being the user’s main visible application.

Examples may manage:

* Printing
* Networking
* Audio
* Login sessions
* Software updates
* Notifications

These services are ordinary processes or specially managed system processes, not necessarily kernel components.

An application may communicate with a background service, which may then communicate with the kernel.

```text
Application
    │
    ▼
Background service
    │
    ▼
Kernel
    │
    ▼
Hardware
```

---

# 2.9 Step-by-Step Example: Playing Music

Consider a music application playing a song.

## Stage 1: The application requests file access

The music player asks the OS to open the song file.

**Handled by:** Application, system library, and kernel

---

## Stage 2: The kernel checks access

The kernel verifies that the process has permission to read the file.

**Handled by:** Kernel security and file system

---

## Stage 3: The file data is retrieved

The kernel coordinates with the storage driver and device.

**Handled by:** File system, storage driver, and storage hardware

---

## Stage 4: Data enters RAM

The song data is placed into memory accessible to the music application.

**Handled by:** Kernel memory management and hardware

---

## Stage 5: The application decodes the audio

The application converts compressed song data into audio samples.

Much of this is ordinary computation performed directly by the CPU.

**Handled by:** Application running on the CPU

---

## Stage 6: The application submits audio

The application sends the prepared audio to an operating-system audio service or interface.

**Handled by:** Application and OS audio system

---

## Stage 7: The driver controls the audio device

The audio driver converts the request into commands appropriate for the device.

**Handled by:** Driver

---

## Stage 8: Hardware produces sound

The audio hardware converts digital audio information into an output signal.

**Handled by:** Audio hardware and speakers

---

## Timeline

```text
Time ─────────────────────────────────────────────────────────▶

Application:  request file | decode audio | provide samples
Kernel:       check access | manage read  | coordinate output
Drivers:                   storage work   | audio-device work
Hardware:                  read SSD       | produce sound
```

These stages overlap in real systems. The application may decode one part of the song while the OS retrieves another part.

---

# 2.10 Step-by-Step Example: Typing into a Text Editor

## 1. A key is physically pressed

The keyboard detects the action.

**Component:** Hardware

## 2. The keyboard reports an event

The keyboard controller sends information to the computer.

**Component:** Hardware controller

## 3. The operating system notices the event

The CPU is informed that the keyboard needs attention.

**Component:** Hardware and kernel interrupt handling

## 4. The driver interprets the device information

The keyboard driver converts low-level information into an OS-understood event.

**Component:** Driver

## 5. The OS identifies the intended application

The graphical system determines which application currently receives keyboard input.

**Component:** OS service or graphical environment

## 6. The text editor receives the event

The application interprets the event according to its current state.

For example, it may insert a character into a document.

**Component:** Application

## 7. The text editor requests a display update

The application asks the graphical system to redraw the text.

**Component:** Application and OS graphics services

## 8. The display hardware updates

Graphics software and hardware produce the visible result.

**Component:** Graphics system, driver, and hardware

```text
Keyboard
   │
   ▼
Keyboard driver
   │
   ▼
Kernel and input system
   │
   ▼
Text editor
   │
   ▼
Graphics system and driver
   │
   ▼
Display
```

---

# 2.11 How This Connects to the Previous Section

Previously, the OS was described as:

* A resource manager
* A rule enforcer
* An abstraction provider

The kernel is the component that implements many of these responsibilities.

| Earlier concept        | Component involved                        |
| ---------------------- | ----------------------------------------- |
| Sharing CPU time       | Kernel scheduler                          |
| Separating memory      | Kernel and CPU memory-protection hardware |
| Opening files          | Kernel file-system mechanisms             |
| Controlling devices    | Kernel, services, and drivers             |
| Checking permissions   | Kernel security mechanisms                |
| Providing abstractions | Kernel plus OS libraries and services     |

The kernel is therefore central, but it does not work alone. Hardware provides enforcement mechanisms, while libraries and services provide higher-level interfaces.

---

# 2.12 What Can Go Wrong?

## An application crashes

A defect may cause the application to perform an invalid operation.

Expected result:

* The kernel stops or restricts the faulty application.
* Other applications continue running.
* Resources owned by the failed application are reclaimed.

Isolation makes this possible.

---

## A privileged driver crashes

A driver running inside the kernel may corrupt protected state.

Possible results:

* Device failure
* Frozen system
* Kernel panic or system stop
* Data loss

A kernel-level failure is usually more serious than an ordinary application failure.

---

## The kernel has a defect

A kernel bug may affect:

* Scheduling
* Memory protection
* File integrity
* Security
* Hardware operation

Because all applications rely on the kernel, its failure can affect the whole system.

---

## Hardware behaves incorrectly

Software cannot fully compensate for all hardware failures.

Examples:

* Faulty RAM changes stored information.
* A failing SSD loses data.
* An overheating CPU becomes unstable.
* A device reports invalid results.

The OS may detect some failures, but not all.

---

## An application receives excessive permission

Even when the kernel functions correctly, an application may cause damage if the user or system grants it too much access.

Isolation is only effective when permission policies are appropriately configured.

---

## A driver does not match the hardware

Possible effects include:

* Device not recognized
* Missing functionality
* Incorrect output
* Instability
* Poor performance

The driver must understand the actual hardware and the OS interface it is using.

---

# 2.13 Common Misconceptions

## Misconception: “The kernel is the graphical desktop”

The kernel is not the desktop, taskbar, file browser, or window manager.

Those are higher-level components.

A kernel can operate without a graphical interface.

---

## Misconception: “Applications never access hardware”

This is too absolute.

Applications generally cannot freely control hardware. They may use OS-approved interfaces, shared memory regions, mapped device resources, or specialized APIs.

The more accurate statement is:

> Applications access hardware under restrictions established by the operating system and hardware.

---

## Misconception: “The OS sits between every CPU instruction and the CPU”

Ordinary application instructions usually execute directly on the CPU.

The kernel does not inspect every arithmetic operation.

The OS regains control when events occur, such as:

* A system-service request
* A timer event
* A hardware event
* A fault or exception

---

## Misconception: “A driver is the physical device”

A driver is software.

The device is hardware.

The driver allows software to communicate with the device.

---

## Misconception: “Every OS component runs inside the kernel”

Many OS components run outside it.

Examples include:

* Graphical desktops
* Login managers
* Printing services
* Network configuration services
* Command-line tools

Keeping components outside the kernel can improve isolation and maintainability.

---

## Misconception: “Kernel access means unlimited physical control”

The kernel has high privilege, but it is still constrained by:

* Hardware capabilities
* Firmware
* Device design
* Physical failures
* Configuration
* Its own implementation

Privilege grants authority, not magical control.

---

# 2.14 Simplified Model Versus Technical Reality

## Simplified model

```text
Application → Kernel → Hardware
```

This is the most useful model for beginning OS study.

---

## More exact model

A real interaction might be:

```text
Application
    │
    ▼
Application library
    │
    ▼
Background system service
    │
    ▼
Kernel
    │
    ▼
Driver
    │
    ▼
Hardware controller
    │
    ▼
Physical device
```

Not every operation uses every layer.

For example:

* An arithmetic operation may require only the application and CPU.
* Reading a file requires kernel involvement.
* Displaying a window may involve graphical services, libraries, the kernel, and a driver.
* Some device work continues independently after the kernel starts it.

The simple model remains useful as long as we remember that it hides intermediate components.

---

# 2.15 Core Mental Model

Keep the following distinction clear:

```text
Applications decide what task to perform.
The kernel decides what access is allowed and coordinates resources.
Hardware physically executes and transfers information.
```

Another useful diagram is:

```text
┌───────────────────────────────────────────┐
│ Applications                              │
│ “What should this program accomplish?”   │
├───────────────────────────────────────────┤
│ Kernel                                    │
│ “How can resources be used safely?”       │
├───────────────────────────────────────────┤
│ Hardware                                  │
│ “Perform the physical operation.”         │
└───────────────────────────────────────────┘
```

This model prepares us for the next distinction:

* Applications usually run with limited privileges.
* The kernel runs with greater privileges.

That separation is called **user mode and kernel mode**.

---

# Learning Check

Do not provide answers yet.

## Conceptual questions

1. What is the difference between hardware, the kernel, and a user application?
2. Why is the kernel considered the privileged core of the operating system?
3. What role does a device driver play between the kernel and hardware?

## Cause-and-effect questions

4. Why can a defective kernel-level driver be more dangerous than a defective text editor?
5. Why would allowing every application to directly control storage hardware make systems less portable and less reliable?

## Misconception question

6. A student says, “Every action performed by an application must first be interpreted by the kernel.” What is inaccurate about this statement?

## Scenario-based question

7. A music player reads a song from an SSD and plays it through speakers. Describe the roles of the application, kernel, drivers, RAM, CPU, storage hardware, and audio hardware.

# 3. User Mode and Kernel Mode

The kernel has greater authority than ordinary applications. That authority must be enforced rather than merely requested.

Modern CPUs therefore support different **execution modes**, also called **privilege levels**.

The two-mode mental model is:

```text
┌──────────────────────────────────────┐
│ User mode                            │
│ Applications and many OS services    │
│ Limited authority                    │
├──────────────────────────────────────┤
│ Protected boundary                   │
├──────────────────────────────────────┤
│ Kernel mode                          │
│ Kernel and some device drivers       │
│ High authority                       │
└──────────────────────────────────────┘
```

The operating system uses these CPU-supported modes to separate ordinary software from trusted system software.

---

## 3.1 What Is User Mode?

**User mode** is the restricted execution environment in which ordinary applications usually run.

Examples include:

* Browsers
* Games
* Text editors
* Music players
* Terminal applications
* Most background services

A program running in user mode can perform normal computation, such as:

* Adding numbers
* Comparing values
* Processing text
* Decoding an image already in memory
* Updating its own data
* Following its own instructions

However, it cannot freely perform sensitive operations.

---

## What user-mode software normally cannot do directly

A user-mode application generally cannot:

* Access arbitrary physical memory
* Modify kernel memory
* Read another process’s private memory
* Directly configure hardware
* Disable interrupts
* Change CPU protection settings
* Control the memory mappings of unrelated processes
* Decide which process receives the CPU next
* Read protected files without permission

These restrictions protect the system from faulty and malicious programs.

---

## Mental model: a tenant in a managed building

A tenant can:

* Use their apartment
* Arrange their furniture
* Invite permitted guests
* Request maintenance

A tenant cannot normally:

* Enter every other apartment
* Rewire the entire building
* Disable the fire alarm
* Change all door locks
* Control the building’s electrical supply

User mode is similar to the tenant’s permitted area.

The application has useful freedom inside its controlled environment, but sensitive building-wide operations belong to the manager.

---

# 3.2 What Is Kernel Mode?

**Kernel mode** is a highly privileged execution mode used by the operating-system kernel.

Software running in kernel mode can generally:

* Configure memory protection
* Access hardware-control interfaces
* Manage processes
* Change scheduling state
* Handle device events
* Access protected kernel data
* Create or alter memory mappings
* Enforce permissions
* Stop misbehaving processes

Kernel mode may also be called:

* Supervisor mode
* Privileged mode
* System mode

The terminology differs across CPU architectures and operating systems.

---

## Why kernel mode exists

The kernel must perform operations that applications are not allowed to perform.

For example, only the kernel should be able to change which memory belongs to a process.

Otherwise, an application could simply declare:

> “The browser’s password memory now belongs to me.”

Kernel mode gives trusted system software the authority needed to enforce rules.

---

## Kernel mode does not mean “the kernel is always running”

At any given instant, a CPU core may be executing:

* Application instructions in user mode
* Kernel instructions in kernel mode

The kernel remains part of the operating system, but the CPU does not continuously execute kernel instructions.

A simplified timeline might be:

```text
Time ─────────────────────────────────────────────────────▶

CPU mode:
[User][User][Kernel][Kernel][User][User][Kernel][User]
```

The CPU changes modes when specific controlled events occur.

---

# 3.3 Why Two Modes Are Necessary

Suppose every program had kernel-level authority.

A defective application could accidentally:

* Overwrite operating-system instructions
* Corrupt the file system
* Stop the CPU scheduler
* Read private information
* Disable security checks
* Misconfigure a device

A malicious application could intentionally do the same.

The two-mode design solves three major problems.

---

## Problem 1: Protecting the kernel

The kernel contains critical information such as:

* Process records
* Memory mappings
* File-system state
* Security information
* Device state
* Scheduling information

User-mode programs cannot freely change this information.

---

## Problem 2: Isolating applications

Each process is normally given its own protected memory environment.

The kernel controls those boundaries.

```text
┌────────────────────┐
│ Browser memory     │
└────────────────────┘
          ✕
┌────────────────────┐
│ Editor memory      │
└────────────────────┘
          ✕
┌────────────────────┐
│ Kernel memory      │
└────────────────────┘
```

The crosses represent access that is normally forbidden.

---

## Problem 3: Controlling shared resources

The kernel coordinates access to:

* CPU time
* Files
* Devices
* Network connections
* Physical memory
* Inter-process communication

Applications must request access rather than seize resources directly.

---

# 3.4 How the CPU Knows Which Mode It Is In

The CPU maintains internal state indicating the current privilege level.

A simplified mental model is a **mode indicator**:

```text
CPU mode = USER
```

or:

```text
CPU mode = KERNEL
```

The exact hardware implementation differs across CPU architectures.

When the CPU is in user mode, it enforces restrictions. Certain operations are rejected.

When the CPU is in kernel mode, privileged operations are permitted.

---

## Privileged instructions

A **privileged instruction** is an instruction that may only be executed at a sufficiently high privilege level.

Examples conceptually include instructions that:

* Change memory-protection configuration
* Control interrupt handling
* Configure certain hardware resources
* Alter important processor-management state

If a user-mode program tries to execute a privileged instruction, the CPU does not simply trust it.

Instead, the CPU reports a protection-related error to the kernel.

```text
User application attempts privileged operation
                    │
                    ▼
CPU detects insufficient privilege
                    │
                    ▼
CPU transfers control to kernel
                    │
                    ▼
Kernel handles the violation
```

The kernel may terminate the program or report an error.

---

# 3.5 Hardware Enforces the Boundary

A common misleading explanation is:

> “Applications promise not to access protected resources.”

That is not sufficient.

The boundary is enforced by hardware mechanisms controlled by the kernel.

The CPU helps enforce:

* Current privilege level
* Allowed instructions
* Memory-access permissions
* Controlled entry points into the kernel

The kernel configures these protections, and the CPU applies them while instructions execute.

---

## Why software-only enforcement would be weak

Suppose an application were told:

> “Do not read memory outside your assigned area.”

A faulty or malicious program could ignore that rule.

Hardware enforcement means the CPU checks memory operations regardless of the application’s intentions.

```text
Application requests memory access
                │
                ▼
CPU checks protection rules
        ┌───────┴────────┐
        │                │
     Allowed          Forbidden
        │                │
        ▼                ▼
Access occurs     Kernel is notified
```

We will examine memory protection more deeply in the virtual-memory sections.

---

# 3.6 How an Application Requests Kernel Work

A user-mode application cannot switch itself into kernel mode merely because it wants greater authority.

Instead, it uses a controlled mechanism called a **system call**.

A system call is a request for the kernel to perform a protected service.

Examples include requests to:

* Open a file
* Read data
* Send network information
* Create another process
* Request memory
* Wait for an event

System calls receive a full section later. For now, focus on the mode transition.

---

## Simplified system-call flow

```text
Application running in user mode
              │
              │ Request kernel service
              ▼
CPU enters kernel through an approved entry point
              │
              ▼
Kernel validates the request
              │
              ▼
Kernel performs or rejects the operation
              │
              ▼
CPU returns to user mode
              │
              ▼
Application continues
```

The application does not choose an arbitrary location inside the kernel.

It enters through a carefully defined entry mechanism.

---

# 3.7 Step-by-Step: Opening a File

Suppose a text editor wants to open `notes.txt`.

## Step 1: The editor runs in user mode

The editor performs ordinary work:

* Handling its interface
* Tracking which document the user selected
* Preparing the filename

**CPU mode:** User mode
**Responsible component:** Application

---

## Step 2: The editor requests an OS service

The application uses an operating-system interface to request that the file be opened.

It provides information such as:

* The file’s path
* The requested access type
* Relevant options

**CPU mode initially:** User mode
**Responsible component:** Application or system library

---

## Step 3: The CPU performs a controlled transition

A special instruction or mechanism transfers control to an approved kernel entry point.

During this transition, the CPU changes to kernel mode.

**CPU mode:** User mode → kernel mode
**Responsible components:** CPU and kernel entry mechanism

---

## Step 4: The kernel validates the request

The kernel checks:

* Is the request correctly formed?
* Does the file exist?
* Does this process have permission?
* Is the provided application memory safe to access?
* Are the requested options valid?

**CPU mode:** Kernel mode
**Responsible component:** Kernel

---

## Step 5: The kernel performs the operation

The kernel may:

* Search the file system
* Locate file information
* Prepare internal records
* Coordinate storage access
* Return a file-related identifier

**CPU mode:** Kernel mode
**Responsible components:** Kernel, file system, and possibly driver

---

## Step 6: The kernel prepares a result

The result may indicate:

* Success
* File not found
* Permission denied
* Invalid request
* Hardware or file-system error

**CPU mode:** Kernel mode
**Responsible component:** Kernel

---

## Step 7: The CPU returns to user mode

The kernel restores the application’s execution state and returns control.

**CPU mode:** Kernel mode → user mode

---

## Step 8: The editor continues

The application receives the result and decides what to do.

It might:

* Display the file
* Show an error message
* Ask for a different filename

**CPU mode:** User mode
**Responsible component:** Application

---

## Timeline

```text
Time ───────────────────────────────────────────────────────▶

Application: prepare request | wait              | continue
CPU mode:    USER            | KERNEL            | USER
Kernel:                      validate → perform
Hardware:                         optional I/O
```

---

# 3.8 Crossing the Boundary Is Controlled

The user/kernel boundary is not a normal function-call boundary.

A normal function call within an application remains in user mode:

```text
Application function A
          │
          ▼
Application function B

Mode remains USER
```

A system call crosses a protection boundary:

```text
Application code
      │
      ▼
Controlled kernel entry
      │
      ▼
Kernel code

Mode changes USER → KERNEL
```

The transition involves additional checks and CPU state changes.

---

## What may happen during entry

Conceptually, the CPU and kernel may:

1. Record where the application was executing.
2. Save enough application state to resume later.
3. Change to kernel mode.
4. Switch to a protected kernel execution area.
5. Identify why control entered the kernel.
6. Run the appropriate kernel handler.

The exact details vary by processor and operating system.

---

## What may happen during return

Conceptually, the kernel may:

1. Finish or reject the requested operation.
2. Prepare a result.
3. Restore the application’s saved state.
4. Restore user-mode privilege.
5. Resume the application.

```text
User execution paused
        │
        ▼
Kernel operation
        │
        ▼
User execution resumed
```

---

# 3.9 The Application Does Not Become the Kernel

When an application makes a system call, the application itself is not granted unrestricted kernel privileges.

Instead:

* The CPU begins executing trusted kernel code.
* The kernel performs a specific requested operation.
* The kernel remains in control.
* The application later resumes in user mode.

This distinction is crucial.

### Incorrect mental model

```text
Application asks for service
          │
          ▼
Application becomes privileged
```

### Better mental model

```text
Application asks for service
          │
          ▼
Kernel temporarily executes on its behalf
          │
          ▼
Application resumes with limited privilege
```

---

# 3.10 What Information Crosses the Boundary?

An application often needs to provide information to the kernel.

For example, a file-read request may include:

* Which file to read
* Where the application wants the data placed
* How much data it wants
* Where reading should begin

This creates a safety challenge.

The kernel must not blindly trust application-provided information.

---

## Why application input is untrusted

A faulty application may provide:

* An invalid memory location
* An impossible length
* A malformed request
* A reference to memory it does not own

A malicious application may deliberately attempt to trick the kernel.

The kernel must validate requests before using them.

---

## Boundary-checking mental model

Think of a secure service desk.

A visitor can submit a form, but the staff checks:

* Is the form valid?
* Is the visitor authorized?
* Does the requested resource exist?
* Are the supplied details safe to use?

The visitor cannot walk behind the desk and perform the operation personally.

---

# 3.11 User Space and Kernel Space

You may encounter the terms **user space** and **kernel space**.

They are related to user mode and kernel mode, but not identical.

## User space

Broadly refers to the environment in which user-mode programs operate, including:

* Applications
* Many background services
* Application libraries
* Process-private memory

## Kernel space

Broadly refers to the protected environment used by kernel code and kernel data.

```text
Virtual address-space model

┌──────────────────────────┐
│ Kernel-space region      │
│ Protected                │
├──────────────────────────┤
│ User-space region        │
│ Application-accessible   │
└──────────────────────────┘
```

This diagram is simplified. Operating systems arrange address spaces differently.

---

## Mode versus space

| Term             | Primarily describes                                      |
| ---------------- | -------------------------------------------------------- |
| **User mode**    | The CPU’s current privilege level                        |
| **Kernel mode**  | A highly privileged CPU execution level                  |
| **User space**   | Memory and software environment for user programs        |
| **Kernel space** | Protected memory and software environment for the kernel |

A CPU mode is an execution privilege state.

A memory space describes where code and data reside and what access is permitted.

The concepts work together but should not be treated as synonyms.

---

# 3.12 Are All Operating-System Services in Kernel Mode?

No.

Many operating-system components run in user mode.

Examples may include:

* Printing services
* Graphical desktop components
* Login services
* Audio services
* Update managers
* Network configuration services

An application may communicate with such a service, and that service may then request kernel operations.

```text
Application: user mode
       │
       ▼
System service: user mode
       │
       ▼
Kernel: kernel mode
       │
       ▼
Hardware
```

Keeping services in user mode can limit damage if they fail.

---

## Why not put everything in kernel mode?

Kernel-mode software has extensive authority.

More kernel-mode code usually means:

* More code capable of crashing the whole system
* A larger security-sensitive area
* Greater consequences from programming errors

Placing suitable components in user mode improves isolation.

The tradeoff is that communication across protection boundaries may add complexity and overhead.

---

# 3.13 Multiple Hardware Privilege Levels

The two-mode model is deliberately simplified.

Some CPUs provide more than two privilege levels.

For example, an architecture may distinguish among:

* Application-level execution
* Kernel-level execution
* Hypervisor-level execution
* Firmware or security-related execution environments

Some architectures historically describe privilege levels as numbered **rings**.

```text
Less privilege
     ↓
┌──────────────────┐
│ Applications     │
├──────────────────┤
│ OS services      │
├──────────────────┤
│ Kernel           │
├──────────────────┤
│ Hypervisor       │
└──────────────────┘
     ↑
More control
```

Real systems do not always use every hardware privilege level.

For foundational study, the most important distinction remains:

> Restricted application execution versus privileged kernel execution.

---

# 3.14 How Control Enters the Kernel

There are three major reasons the CPU may begin executing kernel code.

We will study these in detail later.

## 1. System call

The application deliberately requests an operating-system service.

Example:

> “Please read this file.”

---

## 2. Interrupt

Hardware reports that an external event needs attention.

Example:

> “The storage device completed a read.”

---

## 3. Exception

The CPU detects something during instruction execution.

Examples:

* Invalid instruction
* Division error
* Forbidden memory access
* Missing memory page

---

## Comparison

| Event       | Typical source                | Deliberate application request? |
| ----------- | ----------------------------- | ------------------------------: |
| System call | Running application           |                             Yes |
| Interrupt   | Hardware device or timer      |                              No |
| Exception   | Current instruction execution |                      Usually no |

All three can cause a controlled transfer into kernel mode.

---

# 3.15 Step-by-Step: Illegal Memory Access

Suppose an application attempts to access memory it is not allowed to use.

## Step 1: Application executes in user mode

The CPU is following the application’s instructions.

---

## Step 2: The application attempts a memory access

The instruction tries to read or write a particular memory location.

---

## Step 3: Hardware checks access permissions

The CPU’s memory-management hardware checks rules configured by the kernel.

---

## Step 4: Access is denied

The CPU does not complete the forbidden operation.

Instead, it generates an exception.

---

## Step 5: Control enters the kernel

The CPU switches to kernel mode and runs the appropriate exception handler.

---

## Step 6: The kernel investigates

The kernel considers questions such as:

* Is this a valid memory situation that can be resolved?
* Is a needed page temporarily absent?
* Is the application attempting an illegal access?
* Should the process be terminated?

---

## Step 7: The kernel decides

Possible outcomes include:

* Repair the situation and resume the application
* Report an error to the application
* Terminate the process
* Treat it as a serious kernel failure if the fault occurred in kernel code

```text
Application memory access
            │
            ▼
Hardware permission check
      ┌─────┴─────┐
      │           │
   Allowed      Denied
      │           │
      ▼           ▼
 Continue      Enter kernel
                    │
                    ▼
              Resolve or stop
```

Page faults and invalid accesses will be separated carefully in the memory sections.

---

# 3.16 Step-by-Step: A Timer Gives the Kernel Control

An application may be performing valid computation and never voluntarily ask the kernel for anything.

The OS still needs a way to regain control so that one application cannot use the CPU forever.

A hardware timer helps solve this problem.

## Timeline

1. The kernel configures a timer.
2. An application runs in user mode.
3. The timer reaches its trigger point.
4. The CPU receives a timer interrupt.
5. The CPU switches to kernel mode.
6. The kernel’s scheduler decides what should run next.
7. The same application may resume, or another process may run.

```text
Time ─────────────────────────────────────────────────────▶

App A:       running──────────────paused
CPU mode:    USER                  KERNEL
Timer:                             fires
Kernel:                            scheduling decision
Next task:                          App A or another process
```

This is one foundation of preemptive multitasking.

We will return to it during CPU scheduling and context switching.

---

# 3.17 What Can Go Wrong?

## Invalid system-call request

An application may submit incorrect information.

The kernel should reject the request safely rather than trust it.

Possible result:

* An error is returned to the application.

---

## Kernel fails to validate application memory

A kernel defect might allow a user-mode program to make the kernel read or write an unsafe location.

Possible consequences include:

* Kernel crash
* Information disclosure
* Privilege escalation
* System compromise

---

## Application receives excessive privileges

Some applications are deliberately given elevated permissions.

If such an application is compromised, the damage may be greater.

User mode limits CPU privilege, but operating-system permissions also matter.

---

## Kernel-mode driver contains a defect

A driver may access invalid memory or misconfigure hardware.

Because it operates with high privilege, the failure can affect the whole system.

---

## Protection configuration is incorrect

If the kernel configures memory permissions incorrectly, one process may gain access to another process or the kernel.

---

## Kernel code crashes

A kernel-mode failure is often more serious than a user-mode failure because the kernel manages the entire system.

Typical outcomes may include:

* System stop
* Kernel panic
* Restart
* Data loss

---

# 3.18 Common Misconceptions

## Misconception: “User mode means the human user is controlling the CPU”

No.

“User” refers to a restricted privilege environment, not necessarily direct human interaction.

Background services may also run in user mode.

---

## Misconception: “Kernel mode makes software faster”

Kernel mode primarily grants privilege, not automatic speed.

Kernel-mode code may avoid some boundary transitions, but it also carries greater risk.

High privilege is not a performance feature by itself.

---

## Misconception: “A program enters kernel mode and continues running its own code”

Normally, a controlled transition causes the CPU to execute predefined kernel code.

The application’s arbitrary instructions are not simply granted kernel privilege.

---

## Misconception: “System calls let applications bypass security”

System calls are controlled entry points where security checks occur.

They provide access to kernel services without giving the application unrestricted kernel control.

---

## Misconception: “User-mode programs cannot cause serious harm”

User mode limits direct system authority, but a user-mode program may still:

* Delete files it has permission to modify
* Send private information over a network
* Consume resources
* Exploit a kernel vulnerability
* Mislead the user
* Damage application-owned data

User mode reduces risk; it does not eliminate it.

---

## Misconception: “Kernel mode and administrator access are the same”

They are different.

| Concept                       | Meaning                                                 |
| ----------------------------- | ------------------------------------------------------- |
| Kernel mode                   | CPU execution privilege used by kernel code             |
| Administrator or root account | An operating-system identity with extensive permissions |

An administrator’s ordinary application generally still executes in user mode.

It can request more privileged operations because of its identity, but the kernel performs and validates those operations.

---

## Misconception: “Every OS component runs in kernel mode”

Many operating-system services and graphical components run in user mode.

Only the components needing kernel privilege should ideally receive it.

---

# 3.19 Real-World Analogy: Bank Customer and Bank Staff

Consider a bank.

## Customer area: user mode

A customer can:

* Fill out forms
* Review permitted account information
* Request a transfer
* Deposit funds through approved procedures

The customer cannot:

* Enter the vault
* Edit the bank database directly
* Change another customer’s balance
* Disable security systems

---

## Secure staff area: kernel mode

Authorized staff and systems can perform protected operations.

However, they act according to procedures rather than granting customers direct access.

---

## Service request: system call

The customer submits a transfer request.

The bank then:

1. Validates the request.
2. Checks identity and permission.
3. Checks account state.
4. Performs or rejects the transfer.
5. Returns a result.

```text
Customer
   │ approved request
   ▼
Bank service desk
   │ validate and perform
   ▼
Protected banking system
   │
   ▼
Result returned
```

The customer does not temporarily become a bank administrator.

Similarly, an application does not become the kernel when requesting a system service.

---

# 3.20 Connection to Earlier Concepts

## Connection to hardware

The CPU provides privilege modes and enforces restricted instructions.

Memory-management hardware enforces access rules configured by the kernel.

---

## Connection to the kernel

The kernel runs with the authority required to:

* Configure protections
* Control devices
* Schedule processes
* Validate requests
* Handle faults

---

## Connection to applications

Applications perform ordinary work in user mode and request protected operations through controlled interfaces.

---

## Connection to isolation

User mode prevents an application from freely accessing:

* Kernel data
* Other process memory
* Sensitive hardware functions

---

## Connection to resource management

The kernel can regain control through timer interrupts and decide which process should run.

This prevents applications from controlling the CPU indefinitely.

---

# 3.21 Simplified Model Versus Technical Reality

## Simplified mental model

```text
Application work → user mode
OS management    → kernel mode
```

This is useful, but incomplete.

---

## More exact reality

* Some operating-system services run in user mode.
* Some drivers run outside the kernel.
* CPUs may provide more than two privilege levels.
* Virtual machines may introduce a hypervisor privilege level.
* Firmware may operate in separate execution environments.
* An administrator’s applications still normally execute in user mode.
* Hardware, not just software convention, enforces the boundary.
* Kernel mode grants authority but does not guarantee correct behavior.

The core principle remains:

> Sensitive system operations are performed by trusted code at a higher privilege level, while ordinary programs execute under restrictions.

---

# 3.22 Core Mental Model

Keep this sequence in mind:

```text
Application runs normally in user mode
                 │
                 │ Needs protected service
                 ▼
Application makes a controlled request
                 │
                 ▼
CPU switches to kernel mode
                 │
                 ▼
Kernel validates and handles request
                 │
                 ▼
CPU returns to user mode
                 │
                 ▼
Application continues
```

And remember:

```text
User mode   = restricted execution
Kernel mode = privileged execution
```

The distinction is enforced by the CPU under rules configured by the kernel.

---

# Learning Check

Do not look for answers yet.

## Conceptual questions

1. What is the main difference between user mode and kernel mode?
2. Why must the CPU, rather than application cooperation alone, enforce privilege restrictions?
3. What is the difference between user mode and user space?

## Cause-and-effect questions

4. What happens conceptually when a user-mode application attempts to execute a privileged instruction?
5. Why does moving more software into kernel mode increase the possible consequences of defects?

## Misconception question

6. A student says, “When an application makes a system call, the application temporarily becomes part of the kernel.” What is incorrect about this explanation?

## Scenario-based question

7. A text editor requests access to a protected file. Describe the movement from user mode to kernel mode and back, including the checks the kernel should perform.

# 4. Programs, Processes, and Threads

These three terms describe different things:

| Term        | Basic meaning                              |
| ----------- | ------------------------------------------ |
| **Program** | Stored instructions and data               |
| **Process** | A running instance of a program            |
| **Thread**  | One sequence of execution inside a process |

A useful first model is:

```text
Program file on storage
          │
          │ OS starts it
          ▼
       Process
          │
          ├── Thread 1
          ├── Thread 2
          └── Thread 3
```

A program is mostly passive. A process is an active, managed execution environment. A thread is an execution path within that environment.

---

# 4.1 What Is a Program?

## What it is

A **program** is a collection of instructions and associated information stored in a file or another persistent form.

Examples include:

* A browser application stored on an SSD
* A text editor executable
* A game installed on a computer
* A command-line utility
* A system service program

While sitting on storage, a program is not currently executing.

```text
SSD
┌──────────────────────────────┐
│ Browser program file         │
│ Instructions                 │
│ Initial data                 │
│ Metadata                     │
└──────────────────────────────┘
```

---

## Why programs exist

A CPU needs instructions describing what work to perform.

A program provides those instructions in a reusable form. It can be stored, copied, installed, and started later.

---

## The problem a program solves

Without stored programs, a computer would need to be given all its instructions again each time it started.

Program files preserve:

* Instructions
* Initial data
* Information needed to load the program
* References to required libraries
* Other execution-related metadata

---

## Mental model: a recipe

A program is like a recipe in a cookbook.

The recipe describes:

* Required steps
* Required ingredients
* The order of operations

But a recipe sitting in a book is not currently cooking anything.

Similarly, a program file is not automatically executing merely because it exists.

---

# 4.2 What Is a Process?

## What it is

A **process** is an operating-system-managed instance of a running program.

It includes more than the program’s instructions.

A process commonly has:

* A virtual memory space
* At least one thread
* A process identifier
* Open files and other resources
* Security credentials
* Current execution state
* Information used by the scheduler
* Communication channels
* Accounting and usage information

```text
Process
├── Program instructions
├── Process memory
├── One or more threads
├── Open files
├── Permissions
├── Process identifier
└── Operating-system records
```

---

## Why processes exist

Multiple programs need to run safely and independently.

The process abstraction gives the OS a unit that it can:

* Start
* Stop
* Schedule
* Isolate
* Monitor
* Assign resources to
* Protect from other processes

---

## The problem a process solves

A stored program file does not describe its complete live state.

While running, a program needs changing information such as:

* Which instruction is being executed
* Current calculations
* User-entered data
* Open files
* Network connections
* Allocated memory
* Pending events

A process packages this live state into an object the operating system can manage.

---

## Mental model: recipe versus cooking session

| Cooking concept            | OS concept        |
| -------------------------- | ----------------- |
| Written recipe             | Program           |
| One active cooking session | Process           |
| Cook performing steps      | Thread            |
| Counter and utensils       | Process resources |
| Current step in recipe     | Execution state   |

The same recipe can be used in multiple kitchens at the same time.

Likewise, one program can have multiple processes.

---

# 4.3 One Program Can Create Multiple Processes

Suppose you start the same text editor twice.

The same program file may produce two separate processes:

```text
Text-editor program file
          │
          ├───────────────┐
          ▼               ▼
     Process A        Process B
     document.txt     report.txt
```

Each process can have:

* Separate memory
* Different open files
* Different execution state
* Different process identifiers
* Different permissions or environment settings

Closing Process A does not necessarily close Process B.

---

## Example: multiple browser processes

A browser may use separate processes for:

* Different tabs
* Extensions
* Graphics work
* Network services
* The main browser interface

They may all originate from related program files, but the OS manages them as distinct processes.

This design can improve isolation. If one tab process crashes, the entire browser may not need to stop.

---

# 4.4 Process Identity

The OS needs a way to distinguish processes.

It commonly assigns each process a **process identifier**, often abbreviated as **PID**.

A PID is a number used to refer to a process while it exists.

```text
PID 1042 → Text editor
PID 1088 → Music player
PID 1121 → Browser
```

The exact number is not a permanent identity.

After a process ends, its PID may eventually be reused for another process.

---

## Why process identifiers exist

The OS and administrative tools need to refer to a specific process when performing actions such as:

* Inspecting it
* Sending it a notification
* Stopping it
* Measuring its resource use
* Identifying its parent
* Recording an event

---

# 4.5 Process Memory

Each process normally receives a **virtual address space**.

For now, think of this as the process’s private view of memory.

```text
Process A memory view
┌──────────────────────┐
│ Program instructions │
│ Data                 │
│ Heap                 │
│ Stacks               │
└──────────────────────┘

Process B memory view
┌──────────────────────┐
│ Program instructions │
│ Data                 │
│ Heap                 │
│ Stacks               │
└──────────────────────┘
```

Process A and Process B may use similar-looking memory addresses, but the OS and hardware can map them to different physical memory.

We will study this in the virtual-memory section.

---

## Why private memory matters

Without process memory isolation:

* One program could overwrite another’s data.
* A crash could corrupt unrelated applications.
* Passwords and private information would be exposed.
* Programs would have to coordinate all memory use manually.

Processes create protection boundaries.

---

# 4.6 Process Resources

A process may own or refer to resources such as:

* Open files
* Network connections
* Communication channels
* Timers
* Windows
* Devices
* Memory regions

The kernel keeps records of these resources.

```text
Process record
├── Memory mappings
├── Open file A
├── Open file B
├── Network connection
├── Security identity
└── Threads
```

When a process ends, the OS normally reclaims many of these resources.

---

# 4.7 What Is a Thread?

## What it is

A **thread** is one sequence of instructions being executed within a process.

A process must have at least one thread to perform work.

Each thread has its own execution state, including conceptually:

* Current instruction position
* CPU register values
* A stack
* Scheduling state

Threads inside the same process usually share:

* Process memory
* Program instructions
* Open files
* Security identity
* Many other process resources

---

## Mental model: workers in one workshop

Imagine a workshop representing a process.

The workshop contains:

* Shared tools
* Shared materials
* Shared storage
* Shared instructions

Each worker represents a thread.

```text
Process workshop
┌────────────────────────────────────┐
│ Shared memory and resources        │
│                                    │
│ Thread A       Thread B            │
│ worker         worker              │
│                                    │
│ Thread C                           │
│ worker                             │
└────────────────────────────────────┘
```

Workers can handle different tasks at the same time, but they can also interfere with one another when using shared materials.

---

# 4.8 Why Threads Exist

A process may need to make progress on several activities.

For example, a music application may need to:

* Update its interface
* Decode audio
* Send audio to the operating system
* Read more song data
* Respond to user input

Using multiple threads can allow these activities to be organized separately.

---

## Problems threads help solve

### Responsiveness

An application can keep its interface responsive while another thread performs slower work.

```text
Thread 1: handle user interface
Thread 2: load data
```

Without separation, the interface might freeze while data is loaded.

---

### Parallelism

On a multi-core CPU, multiple threads may execute at the same physical time.

```text
CPU core 1 → Thread A
CPU core 2 → Thread B
```

This can make suitable work complete faster.

---

### Overlapping waiting and computation

One thread may wait for input/output while another continues computing.

```text
Thread A: waiting for network data
Thread B: processing existing data
```

---

### Clear task organization

Threads can represent separate activities within one application.

However, threads also introduce concurrency problems, which we will study later.

---

# 4.9 Process Versus Thread

| Property        | Process                               | Thread                                  |
| --------------- | ------------------------------------- | --------------------------------------- |
| Main purpose    | Resource and isolation container      | Unit of execution                       |
| Memory          | Normally private from other processes | Shared with threads in same process     |
| Open files      | Owned or referenced by process        | Usually shared through process          |
| Execution state | Contains one or more threads          | Has its own current CPU state           |
| Failure effect  | Often isolated from other processes   | May corrupt or crash its entire process |
| Communication   | Usually requires OS mechanisms        | Can directly use shared process memory  |
| Creation cost   | Generally heavier                     | Generally lighter                       |

---

## Core distinction

A process answers:

> “What protected execution environment and resources belong together?”

A thread answers:

> “What sequence of instructions is currently making progress?”

---

# 4.10 What Each Thread Keeps Separately

Threads in one process share much, but not everything.

Each thread needs its own:

* Current instruction location
* Register state
* Stack
* Scheduling status

```text
Process
├── Shared program instructions
├── Shared heap
├── Shared open files
│
├── Thread A
│   ├── Instruction position
│   ├── Registers
│   └── Stack A
│
└── Thread B
    ├── Instruction position
    ├── Registers
    └── Stack B
```

The thread’s separate state allows it to pause and later resume independently.

---

# 4.11 Why Each Thread Needs Its Own Stack

A **stack** holds temporary execution information, such as:

* Active function calls
* Temporary local data
* Information needed to return to an earlier operation

For now, think of a thread’s stack as its private working notebook.

If two threads shared one execution stack without careful separation, their active operations could become mixed together.

```text
Thread A → Stack A
Thread B → Stack B
Thread C → Stack C
```

The process’s heap and other memory may still be shared.

Stacks and heaps will receive a full section later.

---

# 4.12 Process State

A process or thread is not always actively executing on a CPU.

A simplified set of states is:

* **Running:** currently executing on a CPU
* **Ready:** able to run, but waiting for CPU time
* **Waiting:** unable to continue until an event occurs
* **Terminated:** finished or stopped

```text
          CPU selected
   ┌─────────────────────┐
   │                     ▼
 READY ───────────────▶ RUNNING
   ▲                     │
   │                     │ waits for input
   │ event completed     ▼
   └────────────────── WAITING

RUNNING ─────────────▶ TERMINATED
          finishes
```

The exact state names differ across operating systems.

---

## Running

The thread is currently executing instructions on a CPU core.

A four-core CPU can physically run up to four ordinary instruction streams at one instant, ignoring additional hardware features and special cases.

---

## Ready

The thread could run immediately, but no CPU core is currently assigned to it.

It waits in a structure managed by the scheduler.

---

## Waiting

The thread cannot make progress until something happens.

Examples:

* File data arrives
* Network data arrives
* A timer expires
* Another thread releases a resource
* The user provides input

A waiting thread generally should not consume CPU time merely to remain waiting.

---

## Terminated

The process or thread has completed or has been stopped.

The OS must clean up its state.

---

# 4.13 A Process Is Not Necessarily Running

The phrase “running process” is common, but it can be misleading.

A process may exist while none of its threads is currently on a CPU.

For example:

```text
Music process
└── Thread waiting for audio device

Editor process
└── Thread ready for CPU

Browser process
└── Thread currently running
```

All three processes exist, but only one thread may be executing on a particular CPU core at that instant.

---

# 4.14 Step-by-Step: What Happens When a Program Starts

Suppose you open a text editor.

The exact details vary, but the following mental model is useful.

---

## Stage 1: A start request occurs

The request may come from:

* Clicking an icon
* Selecting a file
* Entering a command
* Another process
* An automatic system service

**Handled by:** Existing application or OS interface

---

## Stage 2: The request reaches the kernel

The requesting process asks the kernel to create or start a new process.

**Handled by:** System-call interface and kernel

---

## Stage 3: The kernel validates the request

The kernel checks matters such as:

* Does the program exist?
* Is it a valid executable program?
* Does the user have permission to run it?
* Are required resources available?

**Handled by:** Kernel security, file-system, and process-management components

---

## Stage 4: The kernel creates process records

The kernel creates internal information describing the new process.

This may include:

* Process identifier
* Security identity
* Memory-management records
* Resource tables
* Scheduling information

**Handled by:** Kernel process manager

---

## Stage 5: A virtual memory space is prepared

The OS creates the new process’s memory environment.

It arranges places for:

* Program instructions
* Initial data
* Stacks
* Shared libraries
* Future memory allocations

Not every part must be immediately loaded into physical RAM.

**Handled by:** Kernel memory manager

---

## Stage 6: Program information is mapped or loaded

The operating system uses the program file to prepare executable instructions and initial data.

```text
Program file on SSD
          │
          ▼
Process virtual memory
├── Instructions
├── Initial data
└── Supporting libraries
```

**Handled by:** Program loader, file system, and memory manager

---

## Stage 7: The first thread is created

A new process needs a thread to execute.

The kernel prepares the thread’s initial state:

* Starting instruction location
* Initial stack
* Register state
* Scheduling state

**Handled by:** Kernel process and thread management

---

## Stage 8: The thread becomes ready

The new thread can now be considered for CPU execution.

It may not run immediately because other threads are also competing for CPU time.

**State:** Ready
**Handled by:** Scheduler

---

## Stage 9: The scheduler selects the thread

At some point, the scheduler assigns the thread to a CPU core.

**State:** Ready → running
**Handled by:** Kernel scheduler

---

## Stage 10: The program begins executing in user mode

The CPU starts executing the new application’s instructions.

The process may then:

* Initialize internal data
* Load configuration
* Create windows
* Start additional threads
* Open files
* Request other OS services

**CPU mode:** User mode
**Handled by:** Application, libraries, and OS services

---

## Full flow

```text
User clicks application icon
            │
            ▼
Existing interface requests process creation
            │
            ▼
Kernel validates program and permissions
            │
            ▼
Kernel creates process records
            │
            ▼
Memory space is prepared
            │
            ▼
Program instructions and data are mapped
            │
            ▼
Initial thread is created
            │
            ▼
Thread becomes ready
            │
            ▼
Scheduler assigns CPU time
            │
            ▼
Application begins in user mode
```

---

# 4.15 The Program Is Not Necessarily Loaded All at Once

A simplified explanation often says:

> “The entire program is copied from disk into RAM before execution.”

That is not always accurate.

Modern systems may load or map program information in smaller pieces.

Some portions may enter physical memory only when first needed.

```text
Program file
├── Part A → needed now
├── Part B → needed later
└── Part C → possibly never used
```

This reduces unnecessary memory use and startup work.

Paging and page faults will explain how this works.

---

# 4.16 Parent and Child Processes

Processes are often created by existing processes.

The creating process may be called the **parent**, and the new process may be called the **child**.

```text
Desktop process
      │
      ├── Text-editor process
      ├── Browser process
      └── Music-player process
```

The precise relationship and creation mechanism differ among operating systems.

---

## Why process relationships matter

A parent process may:

* Start a child
* Pass initial information
* Wait for it to finish
* Receive its result
* Communicate with it
* Manage some part of its lifetime

A child process is still normally isolated from its parent’s private memory unless the OS explicitly provides sharing.

---

# 4.17 Creating Additional Threads

After a process starts, it may request more threads.

For example:

```text
Browser process
├── Interface thread
├── Network thread
├── Rendering thread
└── Background-work thread
```

The kernel creates execution state for each thread.

The threads then share the process’s memory and resources.

---

## Simplified thread-creation flow

1. A running thread requests another thread.
2. The kernel validates the request.
3. The kernel creates thread-management information.
4. A new stack and initial execution state are prepared.
5. The new thread becomes ready.
6. The scheduler may assign it CPU time.

---

# 4.18 How Multiple Threads Can Run

Consider a process with three threads.

## On one CPU core

Only one thread executes at a particular instant. The scheduler switches among them.

```text
Time ─────────────────────────────────────▶

Core 1:
[Thread A][Thread B][Thread A][Thread C]
```

This is **concurrency**: multiple activities make progress over overlapping periods.

---

## On multiple CPU cores

Different threads may execute at the same physical instant.

```text
Time ─────────────────────────────────────▶

Core 1: [Thread A────────────]
Core 2: [Thread B────────────]
Core 3: [Thread C────────────]
```

This is **parallelism**: work is physically performed simultaneously.

---

## Concurrency versus parallelism

| Concept         | Meaning                                                     |
| --------------- | ----------------------------------------------------------- |
| **Concurrency** | Multiple activities are in progress during overlapping time |
| **Parallelism** | Multiple activities execute at the same physical instant    |

Concurrency can occur on one core through rapid switching.

Parallelism requires multiple execution resources.

---

# 4.19 Threads Share Memory, Which Is Both Useful and Dangerous

Threads in one process can often communicate by reading and writing shared memory.

This is efficient because they do not need the kernel to transfer every piece of information between them.

```text
Thread A ──┐
           ├──▶ Shared process data
Thread B ──┘
```

However, shared memory creates risks.

If two threads modify the same information at the wrong time:

* Updates may be lost.
* Data may become inconsistent.
* Results may vary unpredictably.
* The application may crash.

This is the foundation of race conditions and synchronization.

---

# 4.20 Processes Are More Isolated Than Threads

Consider two separate processes:

```text
Process A memory  ✕  Process B memory
```

They normally cannot directly access one another’s private memory.

To communicate, they commonly need **inter-process communication**, or IPC.

Examples conceptually include:

* Messages
* Pipes
* Shared memory created through the OS
* Network-style connections
* Signals or notifications

The OS controls these mechanisms.

---

## Threads within one process

```text
Thread A ─┐
          ├── Shared process memory
Thread B ─┘
```

Communication can be easier and faster, but isolation is weaker.

A faulty thread can corrupt information used by all threads in the process.

---

# 4.21 What Happens When a Thread Waits for a File

Suppose one thread asks the OS to read a file, but the data is not immediately ready.

## Step 1: The thread makes a system call

**Mode:** User → kernel

---

## Step 2: The kernel starts or arranges the file operation

The file system and storage driver handle the request.

---

## Step 3: The thread cannot continue yet

The requested data is unavailable.

The kernel marks the thread as waiting.

```text
RUNNING → WAITING
```

---

## Step 4: Another thread receives the CPU

The waiting thread does not need to consume CPU time.

The scheduler runs another ready thread.

---

## Step 5: The storage operation completes

The device informs the kernel.

---

## Step 6: The kernel marks the thread ready

```text
WAITING → READY
```

---

## Step 7: The scheduler eventually runs it

The thread resumes and receives the file-read result.

```text
READY → RUNNING
```

This is how the OS overlaps slow input/output with useful work.

---

# 4.22 Step-by-Step: What Happens When a Program Finishes Normally

## 1. The application completes its work

Its final thread reaches the end of its intended execution or requests termination.

**Handled by:** Application

---

## 2. Control enters the kernel

The process reports completion through an OS mechanism.

**Handled by:** System-call interface

---

## 3. The kernel records the completion result

The process may provide a small status value indicating success or failure.

**Handled by:** Kernel process manager

---

## 4. Threads stop executing

The scheduler will no longer select terminated threads.

---

## 5. Resources are reclaimed

The kernel cleans up resources such as:

* Process memory
* Open file references
* Scheduling records
* Communication endpoints
* Internal process information

Some data may persist if another process still owns or references it.

---

## 6. Interested processes may be notified

A parent or supervising process may receive the completion result.

---

## 7. The process ceases to exist

After required bookkeeping is complete, the operating system removes its remaining process records.

---

# 4.23 Step-by-Step: What Happens When a Program Crashes

A crash means the process cannot continue normally.

Consider an application attempting an invalid memory access.

---

## Stage 1: The application executes in user mode

One of its threads is running.

**Handled by:** CPU executing application instructions

---

## Stage 2: The thread attempts an invalid operation

For example, it tries to access memory not assigned to the process.

---

## Stage 3: Hardware detects the violation

The CPU’s memory-protection mechanism blocks the operation.

**Handled by:** CPU and memory-management hardware

---

## Stage 4: An exception transfers control to the kernel

The CPU changes to kernel mode and enters the appropriate fault handler.

**Handled by:** CPU and kernel

---

## Stage 5: The kernel determines whether the problem is recoverable

Some memory exceptions are normal and can be resolved, such as certain page faults.

An illegal access may not be recoverable.

**Handled by:** Kernel memory manager and exception handler

---

## Stage 6: The kernel reports the failure to the process

The OS may deliver an error notification to the process or invoke a registered failure handler.

The exact mechanism varies.

---

## Stage 7: The process may terminate

If the application cannot recover, the kernel stops its threads.

---

## Stage 8: Resources are reclaimed

The OS releases process-owned resources.

---

## Stage 9: Failure information may be recorded

The OS or another service may save:

* Error information
* A crash report
* Relevant execution state
* Diagnostic logs

---

## Stage 10: Other processes usually continue

Because the failed process was isolated, unrelated applications should remain unaffected.

```text
Application performs invalid access
                 │
                 ▼
Hardware blocks operation
                 │
                 ▼
Kernel receives exception
                 │
                 ▼
Recoverable?
       ┌─────────┴─────────┐
       │                   │
      Yes                  No
       │                   │
Fix and resume      Terminate process
                           │
                           ▼
                    Reclaim resources
```

---

# 4.24 A Thread Crash Can End the Entire Process

Threads share one process environment.

If one thread causes a fatal error, the OS commonly terminates the entire process, including its other threads.

```text
Process
├── Thread A
├── Thread B → fatal invalid access
└── Thread C

Possible result: entire process terminates
```

Why?

Because the faulty thread may already have corrupted shared process memory. Continuing other threads might be unsafe.

Some errors can be caught or handled by the application, but not every failure is recoverable.

---

# 4.25 A Process Crash Versus an OS Crash

## Process crash

Only one process is normally terminated.

```text
Browser tab process: stopped
Music player: continues
Text editor: continues
Kernel: continues
```

---

## Kernel crash

The trusted system core fails.

```text
Kernel: failed
      │
      ├── Browser cannot continue safely
      ├── Music player cannot continue safely
      └── Text editor cannot continue safely
```

Because processes depend on kernel services, a serious kernel failure often stops the entire system.

---

# 4.26 What Can Go Wrong?

## Memory corruption inside a process

One thread may accidentally overwrite data used by another thread.

Possible effects:

* Incorrect output
* Delayed crashes
* Security vulnerabilities
* Unpredictable behavior

---

## Too many processes

Each process consumes resources such as:

* Memory
* Kernel records
* File references
* Scheduling capacity

Excessive process creation can exhaust system resources.

---

## Too many threads

Each thread requires:

* A stack
* Scheduling records
* CPU-management overhead

Thousands of unnecessary threads may reduce performance or exhaust resources.

---

## Process becomes unresponsive

A process may still exist but fail to respond because:

* Its main thread is waiting indefinitely.
* It is trapped in excessive computation.
* Its threads are deadlocked.
* It is waiting for a failed device or service.
* Its interface thread is blocked.

“Not responding” does not necessarily mean the process has crashed.

---

## A process fails to release application-level resources

The OS reclaims many resources when a process ends, but it cannot always undo external effects.

For example:

* Partially written files may remain.
* A remote server may retain incomplete state.
* Unsaved user work may be lost.
* A database operation may need recovery.

---

## Threads interfere with shared data

Two threads may observe or modify shared state in an unsafe order.

This can create race conditions.

---

# 4.27 Common Misconceptions

## Misconception: “A program and a process are the same”

A program is stored instructions and data.

A process is a live execution environment created from a program.

One program can produce multiple processes.

---

## Misconception: “A process is always executing”

A process may be:

* Running
* Ready
* Waiting
* Stopped
* Terminated

It can exist without currently using a CPU.

---

## Misconception: “Each application has exactly one process”

Some applications use one process, while others use many.

Modern browsers commonly use multiple processes.

---

## Misconception: “Each process has exactly one thread”

A process begins with at least one thread but may create many.

---

## Misconception: “Threads have completely separate memory”

Threads in the same process usually share most process memory.

Each thread has separate execution state and usually its own stack.

---

## Misconception: “More threads always make a program faster”

More threads may make a program slower because of:

* Scheduling overhead
* Coordination costs
* Shared-resource contention
* Limited CPU cores
* Poorly parallelizable work

Threads are useful only when their benefits exceed their costs.

---

## Misconception: “A crashed process can never affect anything else”

Isolation limits direct damage, but a crash may still:

* Lose unsaved work
* Leave incomplete files
* Interrupt services used by other processes
* Break a larger multi-process application
* Affect external systems

Isolation is strong, not absolute.

---

## Misconception: “Closing a window always ends the process”

A process may continue in the background after its window closes.

Conversely, one process may manage several windows.

A visible window and an OS process are different concepts.

---

# 4.28 Real-World Analogy: Restaurant, Kitchen, and Workers

Consider a restaurant.

| Restaurant concept           | OS concept             |
| ---------------------------- | ---------------------- |
| Recipe book                  | Program files          |
| One active kitchen operation | Process                |
| Cooks                        | Threads                |
| Kitchen workspace            | Process memory         |
| Shared ingredients           | Shared process data    |
| Each cook’s current task     | Thread execution state |
| Restaurant manager           | Operating system       |
| Kitchen number               | Process identifier     |

Several restaurants may use the same recipe.

Each restaurant has its own kitchen and ingredients, just as separate processes have separate memory environments.

Within one kitchen, multiple cooks share resources. This can improve productivity, but they must coordinate.

Two cooks grabbing and changing the same order at once can cause mistakes. This resembles a race condition.

---

# 4.29 Connection to Earlier Concepts

## Connection to the operating system

The OS treats processes and threads as manageable execution units.

It creates them, schedules them, isolates them, and cleans them up.

---

## Connection to hardware

The CPU executes thread instructions.

Memory hardware helps isolate process address spaces.

Multiple CPU cores can execute multiple threads simultaneously.

---

## Connection to user mode

Application threads normally run in user mode.

They enter the kernel through controlled mechanisms when they need OS services.

---

## Connection to kernel mode

The kernel manages:

* Process records
* Thread state
* Memory mappings
* Scheduling
* Resource ownership
* Process termination

---

## Connection to isolation

Processes usually form protection boundaries.

Threads inside a process share resources and are therefore less isolated from one another.

---

# 4.30 Simplified Model Versus Technical Reality

## Simplified mental model

```text
Program = stored instructions
Process = running program
Thread  = worker inside process
```

This model is useful, but “process equals running program” is slightly incomplete.

---

## More exact reality

A process is an OS-managed execution environment that may be:

* Running
* Ready
* Waiting
* Temporarily stopped

Its program information may be loaded gradually rather than entirely at startup.

A process may also:

* Use several program files and libraries
* Contain multiple threads
* Share selected memory with other processes
* Delegate work to child processes
* Exist without currently executing on a CPU

A more precise statement is:

> A process is an isolated resource and execution environment created and managed by the operating system.

And:

> A thread is a schedulable execution sequence within a process.

---

# 4.31 Core Mental Model

Keep these relationships clear:

```text
Storage
┌─────────────────────┐
│ Program file        │
└─────────────────────┘
          │
          │ started by OS
          ▼
Process
┌────────────────────────────────┐
│ Private virtual memory         │
│ Open resources                 │
│ Security identity              │
│                                │
│ Thread A   Thread B   Thread C │
└────────────────────────────────┘
```

The essential distinctions are:

```text
Program  = passive stored instructions
Process  = protected live environment
Thread   = active execution path
```

The operating system schedules **threads**, while processes provide the memory, resources, and isolation in which those threads operate.

The next section examines how the OS decides which ready thread runs, how long it runs, and how the CPU switches between threads.

# Learning Check

Do not look for answers yet.

## Conceptual questions

1. What is the difference between a program, a process, and a thread?
2. Which resources are usually shared by threads in the same process, and which execution information remains separate?
3. What is the difference between a ready thread and a waiting thread?

## Cause-and-effect questions

4. Why can one faulty thread cause the entire process to terminate?
5. Why can adding more threads make an application slower instead of faster?

## Misconception question

6. A student says, “Because an application appears in the task manager, it must currently be executing on the CPU.” What is wrong with this statement?

## Scenario-based question

7. A browser has separate processes for its main interface and several tabs. One tab process performs an invalid memory access and crashes. Explain which OS protections and process concepts help the other tabs and the main browser continue running.

# 5. CPU Scheduling and Context Switching

A computer may have hundreds of runnable threads but only a few CPU cores.

The operating system must repeatedly answer:

> Which thread should use each CPU core next?

This decision-making is called **CPU scheduling**.

When the CPU stops executing one thread and begins executing another, the operating system performs a **context switch**.

```text
Many runnable threads
        │
        ▼
┌───────────────────────┐
│ Kernel scheduler      │
│ Chooses what runs next│
└───────────────────────┘
        │
        ▼
Limited CPU cores
```

---

# 5.1 Why CPU Scheduling Exists

## The underlying problem

Suppose a system has:

* 4 CPU cores
* 20 applications
* 150 threads

Only a limited number of threads can physically execute at one instant.

The remaining runnable threads must wait.

```text
Ready threads:  T1 T2 T3 T4 T5 T6 T7 T8

CPU cores:
Core 1 → T1
Core 2 → T2
Core 3 → T3
Core 4 → T4

Waiting for CPU:
T5 T6 T7 T8
```

The scheduler determines how CPU access is distributed.

---

## Why applications cannot decide for themselves

If applications controlled CPU access directly:

* One application could refuse to stop.
* Important work might wait indefinitely.
* Background work might make the interface unresponsive.
* Programs could interfere with one another.
* Security and isolation would weaken.

The kernel must retain control over CPU allocation.

---

# 5.2 What Is CPU Scheduling?

## What it is

**CPU scheduling** is the operating system’s process of selecting which ready thread should run on a CPU core.

Modern operating systems usually schedule **threads**, because threads are the actual execution sequences.

Processes remain important as resource and isolation containers.

---

## Why it exists

Scheduling allows the operating system to:

* Share CPU time
* Keep interactive applications responsive
* Make progress on background work
* Use multiple CPU cores
* Respect priorities
* Prevent one thread from controlling the CPU forever
* Balance performance and fairness

---

## Mental model: checkout counters

Imagine a supermarket with:

* Many customers
* A limited number of checkout counters
* Different kinds of customers and purchases

The store must decide:

* Who goes to which counter
* Who waits
* Whether urgent cases receive priority
* How to prevent one customer from blocking everyone

| Supermarket       | Operating system   |
| ----------------- | ------------------ |
| Customers         | Ready threads      |
| Checkout counters | CPU cores          |
| Waiting lines     | Ready queues       |
| Store coordinator | Scheduler          |
| Time at counter   | CPU execution time |

The analogy is imperfect because a CPU can pause and resume a thread more easily than a cashier can pause a purchase.

---

# 5.3 Thread States Revisited

The simplified states from the previous section were:

* Running
* Ready
* Waiting
* Terminated

```text
                    selected by scheduler
           ┌──────────────────────────────┐
           │                              ▼
        READY ───────────────────────▶ RUNNING
           ▲                              │
           │                              │ waits for event
           │ event completes              ▼
           └────────────────────────── WAITING

RUNNING ─────────────────────────────▶ TERMINATED
                 finishes
```

Scheduling mainly concerns movement between **ready** and **running**.

---

## Running

The thread is currently executing on a CPU core.

---

## Ready

The thread can execute but is waiting for CPU access.

It has everything it needs except a core.

---

## Waiting

The thread cannot make progress yet.

It may be waiting for:

* File data
* Network data
* Keyboard input
* A timer
* Another thread
* A device
* A synchronization event

A waiting thread is generally not considered for CPU scheduling until its event occurs.

---

# 5.4 The Ready Queue

The kernel keeps track of threads that are ready to run.

A simplified structure is called a **ready queue**.

```text
Ready queue:

Front                                      Back
  │                                          │
  ▼                                          ▼
[T4] → [T7] → [T2] → [T9] → [T5]
```

The scheduler chooses threads from this collection.

Real operating systems may use:

* Multiple queues
* Priority-based structures
* Per-core queues
* Trees
* Deadlines
* Runtime statistics

The simple queue remains a useful starting model.

---

# 5.5 What Makes a Thread Ready?

A thread may become ready when:

* It is newly created.
* Its requested file data arrives.
* A timer finishes.
* A lock becomes available.
* A message arrives.
* It is resumed after being paused.
* It has been removed from a CPU but can continue later.

```text
New thread ───────────────┐
File read completed ──────┤
Timer expired ────────────┼──▶ READY
Lock became available ────┤
Preempted thread ─────────┘
```

Once ready, the thread waits for the scheduler to assign it a core.

---

# 5.6 Scheduling Goals

There is no single scheduling rule that is best for every workload.

Operating systems balance several goals.

## Responsiveness

Interactive applications should react quickly.

Examples:

* Keyboard input
* Mouse movement
* Window updates
* Audio playback

A user notices delays even when total system throughput is high.

---

## Throughput

**Throughput** means how much work the system completes over a period.

A server may prioritize completing many requests efficiently.

---

## Fairness

Runnable threads should receive reasonable opportunities to execute.

Fairness does not always mean equal CPU time.

A high-priority audio thread and a background file-indexing thread may be treated differently.

---

## CPU utilization

The system should avoid leaving a CPU core idle when useful work is ready.

---

## Predictability

Some tasks need consistent response times.

For example, audio playback may fail visibly if scheduled too irregularly.

---

## Priority support

Some work is more urgent than other work.

The scheduler may favor:

* User-interface work
* Audio processing
* Time-sensitive system tasks
* Explicitly prioritized applications

---

## Energy efficiency

On laptops and mobile devices, scheduling also affects battery use.

The OS may:

* Group work together
* Allow cores to sleep
* Move work among cores
* Reduce unnecessary wakeups

---

# 5.7 Scheduling Policies

A **scheduling policy** is the rule used to decide which thread runs next.

Several simplified policies illustrate important tradeoffs.

---

## First-Come, First-Served

Threads run in the order they become ready.

```text
Arrival order:

T1 → T2 → T3 → T4
```

### Benefit

Simple and predictable.

### Problem

A long-running thread can delay many short tasks.

```text
T1: 100 ms
T2:   2 ms
T3:   1 ms
```

T2 and T3 may wait behind T1 even though they would finish quickly.

This is sometimes called a **convoy effect**.

---

## Round Robin

Each ready thread receives a limited amount of CPU time and then moves behind other ready threads.

```text
T1 → T2 → T3 → T1 → T2 → T3
```

### Benefit

Supports responsiveness and sharing.

### Problem

Very frequent switching creates overhead.

Very long turns reduce responsiveness.

---

## Priority Scheduling

Threads are assigned priorities.

Higher-priority threads are generally selected before lower-priority threads.

```text
High priority:   Audio thread
Medium priority: Browser thread
Low priority:    Background indexing
```

### Benefit

Urgent work can run sooner.

### Problem

Low-priority threads may receive too little CPU time.

This is called **starvation**.

---

## Shortest-Work-First Ideas

A scheduler may favor work expected to complete quickly.

### Benefit

Can improve average completion time.

### Problem

The OS usually cannot know exactly how long a thread will run.

It must estimate from past behavior.

---

## Real-time Scheduling

Some systems support tasks with strict timing requirements.

A **real-time scheduler** focuses on meeting timing constraints, not merely making the system feel fast.

Examples may include:

* Industrial control
* Vehicle systems
* Medical equipment
* Audio processing

Real-time does not necessarily mean “extremely fast.”

It means timing behavior must satisfy defined constraints.

---

# 5.8 Preemptive Scheduling

Most modern general-purpose operating systems use **preemptive scheduling**.

## What preemption means

**Preemption** means the kernel can stop a running thread temporarily and give the CPU to another thread.

The running thread does not need to volunteer.

```text
Thread A running
       │
       │ kernel interrupts execution
       ▼
Thread A paused
       │
       ▼
Thread B runs
```

---

## Why preemption exists

Without preemption, a thread could continue until it:

* Finished
* Waited for something
* Voluntarily yielded the CPU

A defective thread might never do any of those things.

Preemption allows the OS to regain control.

---

# 5.9 Cooperative Scheduling

In **cooperative scheduling**, a running task keeps the CPU until it voluntarily gives control back.

It may give up the CPU when it:

* Finishes
* Waits for input/output
* Calls a yield operation
* Requests an OS service

### Benefit

Fewer forced interruptions and simpler behavior.

### Problem

One uncooperative or defective task can make the whole system unresponsive.

```text
Thread A runs
     │
     │ never yields
     ▼
Other threads wait indefinitely
```

Modern desktop and mobile operating systems generally rely heavily on preemption.

---

# 5.10 Time Slices

A scheduler may allow a thread to run for a limited period called a **time slice** or **time quantum**.

A simplified timeline:

```text
Time ─────────────────────────────────────────────▶

CPU:
[Thread A][Thread B][Thread C][Thread A][Thread B]
```

After Thread A’s time slice ends, the kernel may choose another ready thread.

---

## Time slice too long

Possible effects:

* Slow interface response
* Longer waiting times
* A system that feels less interactive

---

## Time slice too short

Possible effects:

* Too many context switches
* More scheduler overhead
* Less useful work completed
* More disruption to CPU caches

The scheduler must balance responsiveness against switching cost.

---

## Exact technical reality

Modern schedulers do not always give every thread a fixed, identical time slice.

Decisions may depend on:

* Priority
* Recent CPU usage
* Number of ready threads
* Type of workload
* CPU topology
* Power policy
* Real-time requirements

---

# 5.11 How the Kernel Regains Control

Suppose Thread A performs an endless calculation and never makes a system call.

How can the kernel stop it?

A hardware timer provides the answer.

## Step by step

1. The kernel configures a timer.
2. Thread A runs in user mode.
3. The timer reaches a configured point.
4. The CPU receives a timer interrupt.
5. The CPU enters kernel mode.
6. The kernel examines scheduling conditions.
7. The scheduler may continue Thread A or choose another thread.

```text
Thread A executing in user mode
               │
               │ timer interrupt
               ▼
Kernel regains control
               │
               ▼
Scheduling decision
       ┌───────┴────────┐
       │                │
 Continue A         Run Thread B
```

This prevents ordinary user-mode code from holding a CPU core indefinitely.

---

# 5.12 What Is a Context?

A thread’s **execution context** is the information needed to stop it and later resume it correctly.

Conceptually, this includes:

* Current instruction position
* CPU register values
* Stack position
* Processor status information
* Scheduling state
* Sometimes additional processor-specific state

```text
Thread context
├── Next instruction location
├── Register values
├── Stack location
├── CPU status
└── Kernel scheduling records
```

---

## Why context must be saved

Suppose a thread is interrupted while calculating:

```text
Current calculation:
partial result = 47
next instruction = continue calculation
```

If the OS switched to another thread without preserving this information, the first thread would not know how to resume.

Saving context is like placing a bookmark and recording the current working notes.

---

# 5.13 What Is a Context Switch?

A **context switch** occurs when the CPU changes from executing one thread to executing another.

The kernel:

1. Saves the current thread’s execution state.
2. Updates its scheduling state.
3. Chooses another ready thread.
4. Restores that thread’s saved state.
5. Resumes execution.

```text
Thread A running
      │
      ▼
Save Thread A context
      │
      ▼
Choose Thread B
      │
      ▼
Restore Thread B context
      │
      ▼
Thread B running
```

---

# 5.14 Step-by-Step: A Context Switch

Suppose Thread A is running and its time slice ends.

## Step 1: Thread A executes in user mode

The CPU is following Thread A’s instructions.

**Thread state:** Running
**CPU mode:** User mode

---

## Step 2: A timer interrupt occurs

The timer informs the CPU that the kernel needs attention.

**Handled by:** Hardware timer and CPU

---

## Step 3: The CPU enters kernel mode

The CPU pauses Thread A and enters a kernel interrupt handler.

Some minimum execution state is preserved automatically by the hardware.

**CPU mode:** User → kernel

---

## Step 4: The kernel saves Thread A’s remaining context

The kernel records enough information to resume Thread A later.

This may include:

* Register values
* Current instruction location
* Stack information
* Processor status

**Handled by:** Kernel

---

## Step 5: Thread A’s state changes

If Thread A can continue later, it usually becomes ready.

```text
RUNNING → READY
```

If it was waiting for something, it may instead become waiting.

---

## Step 6: The scheduler chooses Thread B

The scheduler considers ready threads and scheduling policy.

Possible factors include:

* Priority
* Recent CPU usage
* Fairness
* Core affinity
* Timing requirements
* How long each thread has waited

---

## Step 7: The kernel prepares Thread B

The kernel changes the selected core’s scheduling ownership to Thread B.

If Thread B belongs to another process, the kernel may also change the active memory mapping.

---

## Step 8: Thread B’s context is restored

The kernel restores:

* Thread B’s registers
* Instruction location
* Stack location
* Necessary processor state

---

## Step 9: The CPU returns to user mode

Thread B continues from the point where it previously stopped.

**CPU mode:** Kernel → user
**Thread B state:** Ready → running

---

## Timeline

```text
Time ───────────────────────────────────────────────────▶

Thread A: running──────────────paused
CPU mode: USER                  KERNEL                 USER
Kernel:                        save A
                               choose B
                               restore B
Thread B:                                             resumes
```

---

# 5.15 Context Switching Is Usually Invisible to Applications

Thread A normally does not observe:

> “I was paused for 4 milliseconds and another program ran.”

From its perspective:

1. It was executing one instruction.
2. Some time passed.
3. It resumed with its state preserved.

```text
Thread A’s logical view:

Instruction 100
Instruction 101
Instruction 102
Instruction 103
```

Actual CPU timeline:

```text
Thread A: Instructions 100–101
Thread B: runs
Kernel:   scheduling work
Thread A: Instructions 102–103
```

The OS preserves the illusion that each thread has its own continuing execution.

---

# 5.16 Switching Between Threads in the Same Process

Two threads in one process usually share the same virtual address space.

```text
Process A
├── Thread 1
└── Thread 2
```

A switch between them may require:

* Saving and restoring thread registers
* Changing stacks
* Updating scheduler information

The process memory environment may remain mostly unchanged.

---

# 5.17 Switching Between Threads in Different Processes

Consider:

```text
Process A → Thread A
Process B → Thread B
```

A switch may require additional work:

* Save Thread A’s context
* Restore Thread B’s context
* Change the active virtual-memory mapping
* Update memory-protection state
* Possibly affect address-translation caches

This can be more expensive than switching between threads in the same process.

The exact cost depends on hardware and operating-system design.

---

# 5.18 Context Switch Versus Mode Switch

These are related but different.

## Mode switch

The CPU changes privilege level.

Example:

```text
User mode → kernel mode → user mode
```

The same thread may continue before and after.

---

## Context switch

The CPU changes from one thread to another.

Example:

```text
Thread A → Thread B
```

---

## Comparison

| Event                                |                   Mode changes? | Running thread changes? |
| ------------------------------------ | ------------------------------: | ----------------------: |
| Simple system call                   |                             Yes |         Not necessarily |
| Timer interrupt, same thread resumes |                             Yes |                      No |
| Scheduler chooses another thread     |                Usually involved |                     Yes |
| Thread switch within kernel activity | Possibly already in kernel mode |                     Yes |

A mode switch does not automatically imply a context switch.

A system call may enter the kernel, complete, and return to the same thread.

---

# 5.19 Context Switching Has a Cost

A context switch does not directly advance the application’s useful task.

It is management work needed to share the CPU.

Costs may include:

* Saving register state
* Restoring another context
* Running scheduler logic
* Changing memory mappings
* Disrupting CPU caches
* Disrupting address-translation caches
* Reducing processor prediction accuracy

---

## Direct cost

The kernel performs bookkeeping rather than application work.

---

## Indirect cost

The new thread may need data that is not currently in the CPU’s fast caches.

The processor may spend time reloading:

* Instructions
* Data
* Memory translations

This can make the effective cost greater than the basic save-and-restore work.

---

# 5.20 Why Context Switching Is Still Worthwhile

Despite its cost, context switching enables:

* Multitasking
* Responsive interfaces
* Overlapping computation with input/output
* Fair sharing
* Isolation between programs
* Use of priorities
* Recovery from stalled tasks

The goal is not to eliminate context switches.

The goal is to perform them when their benefits justify their cost.

---

# 5.21 CPU-Bound and I/O-Bound Work

Scheduling behavior depends partly on what a thread does.

## CPU-bound thread

A **CPU-bound** thread spends most of its time computing.

Examples:

* Video encoding
* Large calculations
* Data compression
* Image rendering

```text
CPU work → CPU work → CPU work → CPU work
```

---

## I/O-bound thread

An **I/O-bound** thread frequently waits for external operations.

Examples:

* Reading files
* Waiting for network messages
* Receiving keyboard input
* Waiting for a database response

```text
short CPU work → wait → short CPU work → wait
```

---

## Why the distinction matters

Interactive and I/O-bound threads often need short bursts of CPU time soon after an event occurs.

CPU-bound threads may use longer periods of computation.

A scheduler may use observed behavior to balance both kinds of work.

---

# 5.22 Step-by-Step: A File Read Frees the CPU

Suppose a text editor requests file data.

## 1. Editor thread runs

The thread is executing on a CPU core.

```text
Editor thread: RUNNING
```

---

## 2. It requests a file read

The thread enters the kernel through a system call.

---

## 3. Data is not immediately available

The kernel starts or arranges the storage operation.

---

## 4. The editor thread waits

The editor cannot continue until the data arrives.

```text
Editor: RUNNING → WAITING
```

---

## 5. Scheduler chooses another ready thread

A music thread may now use the CPU.

```text
CPU: Editor → Music
```

---

## 6. Storage hardware works independently

The SSD controller retrieves the requested data.

The CPU can perform other work meanwhile.

---

## 7. Storage completion is reported

The device generates an interrupt or otherwise informs the kernel.

---

## 8. Editor thread becomes ready

```text
Editor: WAITING → READY
```

---

## 9. Scheduler eventually selects the editor

```text
Editor: READY → RUNNING
```

The file-read request finishes, and the editor resumes.

---

## Timeline

```text
Time ─────────────────────────────────────────────────────▶

Editor: [run][request]────waiting──────────────[resume]
Music:                 [running──────────────]
Storage:               [reading file────────]
Kernel:      arrange I/O                 completion handling
```

Scheduling prevents the CPU from sitting idle while storage is busy.

---

# 5.23 Step-by-Step: Multiple Programs Running Together

Suppose the following are active:

* Browser
* Music player
* File downloader
* Text editor

Assume one CPU core for the simplified walkthrough.

---

## Initial state

```text
RUNNING: Browser
READY:   Music, Downloader, Editor
```

---

## Stage 1: Browser executes

The browser processes a webpage.

```text
CPU → Browser thread
```

---

## Stage 2: Browser’s time slice ends

A timer interrupt gives control to the kernel.

The browser remains able to continue, so it becomes ready.

```text
Browser: RUNNING → READY
```

---

## Stage 3: Scheduler selects music thread

Audio playback is time-sensitive.

```text
Music: READY → RUNNING
```

The music thread prepares the next audio data.

---

## Stage 4: Music thread waits for audio timing

It submits data and waits for a future event.

```text
Music: RUNNING → WAITING
```

---

## Stage 5: Scheduler selects downloader

The downloader processes incoming network data.

```text
Downloader: READY → RUNNING
```

---

## Stage 6: Downloader waits for more network data

```text
Downloader: RUNNING → WAITING
```

---

## Stage 7: Scheduler selects editor

The editor responds to a keypress.

```text
Editor: READY → RUNNING
```

---

## Stage 8: Browser becomes active again

After the editor’s turn or wait, the scheduler returns to the browser.

---

## Simplified timeline

```text
Time ───────────────────────────────────────────────────▶

CPU:
[Browser][Music][Downloader][Editor][Browser][Music]

Browser:    run → ready ───────────────→ run
Music:      ready → run → wait ─────────────→ ready
Downloader: ready ───→ run → wait
Editor:     ready ─────────→ run
```

The programs appear to run together because the switching happens quickly.

---

# 5.24 Scheduling on Multiple CPU Cores

With multiple cores, several threads can truly run simultaneously.

```text
Time ───────────────────────────────────────────────▶

Core 1: [Browser thread──────────────────────────]
Core 2: [Music thread────────][Editor thread─────]
Core 3: [Downloader thread───────────────────────]
Core 4: [OS background thread────────────────────]
```

The scheduler must decide:

* Which thread runs on which core
* Whether a thread should move between cores
* How to balance work
* How to preserve cache efficiency
* How to avoid overloading one core

---

# 5.25 Load Balancing

**Load balancing** means distributing runnable work among CPU cores.

Suppose:

```text
Core 1 queue: T1 T2 T3 T4 T5
Core 2 queue: empty
```

The scheduler may move some work:

```text
Core 1 queue: T1 T2 T3
Core 2 queue: T4 T5
```

This lets both cores perform useful work.

---

## Why moving threads is not always free

A thread may have useful data in one core’s cache.

Moving it to another core may require reloading that data.

Therefore, schedulers balance:

* Even workload distribution
* Keeping threads near cached data

---

# 5.26 CPU Affinity

**CPU affinity** describes a preference or restriction that associates a thread with particular CPU cores.

Possible reasons include:

* Improving cache reuse
* Meeting performance requirements
* Separating workloads
* Hardware-specific constraints

A thread with strong affinity may repeatedly return to the same core.

```text
Thread A prefers Core 2
Thread B allowed on Cores 1 and 3
```

Affinity can improve performance, but overly strict restrictions may leave some cores overloaded while others are idle.

---

# 5.27 Priorities

A thread’s **priority** influences how urgently the scheduler treats it.

A higher-priority thread may:

* Run before lower-priority work
* Interrupt lower-priority execution
* Receive more frequent CPU access

---

## Example

```text
High priority:   Audio playback
Normal priority: Browser rendering
Low priority:    Background file indexing
```

The objective is not necessarily to make the high-priority thread run continuously.

It is to ensure that it runs when needed.

---

## Why not make everything high priority?

If every thread is high priority, priority loses meaning.

Excessive high-priority work may:

* Delay ordinary applications
* Reduce responsiveness elsewhere
* Starve background work
* Make the system unstable

Priority must represent genuine scheduling importance.

---

# 5.28 Starvation

**Starvation** occurs when a ready thread waits for an excessively long time because other threads repeatedly receive the CPU.

```text
High-priority work: H H H H H H H H
Low-priority work:  waits indefinitely
```

A scheduler may reduce starvation by using **aging**.

Aging gradually increases the effective priority of a thread that has waited for a long time.

```text
Long waiting time
       │
       ▼
Effective priority increases
       │
       ▼
Thread eventually runs
```

---

# 5.29 Priority Inversion

A subtle scheduling problem occurs when:

1. A low-priority thread holds a resource.
2. A high-priority thread needs that resource.
3. The high-priority thread must wait.
4. Medium-priority threads keep delaying the low-priority holder.

```text
Low-priority thread: holds needed resource
High-priority thread: waits for resource
Medium-priority threads: keep running
```

The high-priority thread is effectively delayed by lower-priority work.

This is called **priority inversion**.

Operating systems may use techniques such as temporarily raising the resource holder’s priority.

Synchronization will be covered later.

---

# 5.30 Scheduling Latency

**Scheduling latency** is the delay between a thread becoming ready and actually beginning to run.

Example:

```text
Keyboard event occurs
        │
        ▼
Editor thread becomes ready
        │  scheduling delay
        ▼
Editor thread runs
```

Low latency matters for:

* Interactive interfaces
* Audio
* Games
* Real-time control

High throughput alone does not guarantee low latency.

---

# 5.31 What Can Go Wrong?

## Too many context switches

If threads switch too frequently:

* More CPU time is spent in kernel bookkeeping.
* Cache contents are disrupted.
* Overall throughput may decrease.

---

## Time slices are too long

If threads run for too long:

* Interactive response becomes slower.
* Audio or visual updates may be delayed.
* The system may feel frozen.

---

## Starvation

Low-priority work may wait indefinitely if higher-priority work continually arrives.

---

## Incorrect priority choices

A background task given excessive priority may interfere with important interactive work.

---

## Poor load balancing

One CPU core may be overloaded while another is underused.

---

## Excessive migration

Moving threads repeatedly among cores may damage cache performance.

---

## Priority inversion

High-priority work may be indirectly blocked by a lower-priority thread.

---

## Scheduler overhead

Complex scheduling decisions consume CPU time themselves.

A more intelligent policy is useful only when its benefits exceed its management cost.

---

## Too many runnable threads

If many threads compete for a small number of cores:

* Each receives less CPU time.
* Context switching increases.
* Cache efficiency declines.
* Latency becomes less predictable.

Creating more threads does not create more CPU capacity.

---

# 5.32 Common Misconceptions

## Misconception: “The operating system schedules programs”

More precisely, modern operating systems usually schedule threads.

Processes provide resources and isolation; threads are execution units.

---

## Misconception: “A running process always has a CPU core”

A process may exist while all its threads are ready or waiting.

Only running threads currently occupy CPU execution resources.

---

## Misconception: “Multitasking means every program runs simultaneously”

On one core, threads take turns.

True simultaneous execution requires multiple execution resources.

---

## Misconception: “A context switch means switching from user mode to kernel mode”

Not necessarily.

A mode switch changes privilege level.

A context switch changes which thread executes.

They often occur together, but they are distinct events.

---

## Misconception: “Context switching is free because it happens quickly”

Even a fast switch consumes time and may disturb processor caches.

Many unnecessary switches can significantly reduce performance.

---

## Misconception: “The highest-priority thread always runs forever”

A high-priority thread runs only while it is ready and permitted by the scheduling policy.

It may wait for input/output or synchronization.

Some scheduling classes also impose limits.

---

## Misconception: “More CPU cores eliminate scheduling”

Multiple cores increase available execution capacity, but scheduling is still required whenever runnable threads exceed available cores.

The OS must also assign work to appropriate cores.

---

## Misconception: “A waiting thread is waiting for its CPU turn”

That describes a ready thread.

A waiting thread cannot continue until some external condition or event occurs.

---

# 5.33 Real-World Analogy: Teachers and Classrooms

Imagine a school with:

* Many classes needing lessons
* A limited number of classrooms
* A limited number of teachers

The scheduler is like the coordinator assigning classes to classrooms.

## Ready class

The students and teacher are prepared, but no classroom is available.

## Running class

The lesson is currently taking place.

## Waiting class

The class cannot continue because it is waiting for equipment, a guest, or another prerequisite.

## Context switch

A classroom changes from one class to another.

The first class must record:

* Where the lesson stopped
* Which materials were in use
* What should happen next

The next class restores its own materials and continues.

The classroom itself is like the CPU core: it provides execution capacity but does not remember each class’s state automatically.

---

# 5.34 Connection to Earlier Concepts

## Connection to processes and threads

Processes own memory and resources.

Threads are the units the scheduler normally selects for execution.

---

## Connection to user and kernel mode

Application threads run in user mode.

Timer interrupts transfer control to kernel mode so the scheduler can make decisions.

---

## Connection to hardware

CPU cores physically execute threads.

Hardware timers help the OS regain control.

CPU caches influence the cost of moving threads.

---

## Connection to waiting and input/output

A thread that waits for a device stops using the CPU.

The scheduler gives the core to another ready thread.

---

## Connection to isolation

A context switch between processes may also change the active protected memory environment.

This helps ensure that one process sees its own memory rather than another process’s memory.

---

# 5.35 Simplified Model Versus Technical Reality

## Simplified mental model

```text
Each thread gets a turn.
The kernel saves one thread and restores another.
```

This is useful but incomplete.

---

## More exact reality

Modern schedulers may consider:

* Multiple priority classes
* Per-core scheduling queues
* CPU topology
* Cache relationships
* Energy use
* Recent behavior
* Deadlines
* Processor affinity
* Security constraints
* Real-time guarantees
* Hardware multithreading

Time slices are not always fixed.

The scheduler may not examine every thread in one simple queue.

Some kernel activity can also be scheduled or preempted.

The foundational principle remains:

> The scheduler distributes limited CPU execution capacity among runnable threads, and context switching preserves each thread’s ability to resume.

---

# 5.36 Core Mental Model

Keep these state transitions clear:

```text
READY ──scheduler selects──▶ RUNNING
RUNNING ──preempted────────▶ READY
RUNNING ──needs event──────▶ WAITING
WAITING ──event occurs─────▶ READY
RUNNING ──finishes─────────▶ TERMINATED
```

And the context-switch sequence:

```text
Thread A runs
     │
     ▼
Kernel gains control
     │
     ▼
Save A
     │
     ▼
Choose B
     │
     ▼
Restore B
     │
     ▼
Thread B runs
```

The scheduler determines **who runs next**.

The context switch makes that decision physically possible.

The next section separates three different ways control reaches the kernel:

* System calls
* Interrupts
* Exceptions

# Learning Check

Do not provide answers yet.

## Conceptual questions

1. What is the difference between a ready thread, a running thread, and a waiting thread?
2. What information must be preserved so that a paused thread can later resume correctly?
3. What is the difference between a mode switch and a context switch?

## Cause-and-effect questions

4. Why can making time slices extremely short reduce overall system performance?
5. Why does a thread waiting for storage data normally stop receiving CPU time?

## Misconception question

6. A student says, “Four applications on a one-core CPU are all physically executing at the same instant because the OS supports multitasking.” What is incorrect about this statement?

## Scenario-based question

7. A browser thread is running when its time slice ends. A music thread is ready, while a file-downloader thread is waiting for network data. Describe the scheduler’s likely actions, the required context switch, and the state of each thread afterward.

# 6. System Calls, Interrupts, and Exceptions

The kernel must regain control of the CPU in several situations:

1. An application deliberately requests an OS service.
2. A hardware device reports an event.
3. The CPU detects a problem or special condition while executing an instruction.

These correspond to:

| Mechanism       | Typical source          | Basic meaning                                                  |
| --------------- | ----------------------- | -------------------------------------------------------------- |
| **System call** | Application             | “Kernel, perform this service for me.”                         |
| **Interrupt**   | Hardware or timer       | “Kernel, an external event needs attention.”                   |
| **Exception**   | Current CPU instruction | “Something special happened while executing this instruction.” |

All three can transfer execution into kernel mode, but they occur for different reasons.

```text id="35yhnq"
                    Control enters kernel
                            ▲
            ┌───────────────┼───────────────┐
            │               │               │
       System call      Interrupt       Exception
       Application      Hardware        CPU execution
       request          event           condition
```

---

# 6.1 Why These Mechanisms Exist

The kernel cannot continuously examine every application instruction and device.

That would be inefficient.

Instead, the system uses controlled events:

* Applications enter the kernel only when they need protected services.
* Devices notify the CPU when attention is required.
* The CPU reports unusual execution conditions automatically.

This produces an event-driven model:

```text id="1rt5kk"
Normal execution continues
          │
          │ important event occurs
          ▼
CPU transfers control to kernel
          │
          ▼
Kernel handles the event
          │
          ▼
Execution resumes or changes
```

---

# 6.2 System Calls

## What a system call is

A **system call** is a controlled request made by a user-mode program for a service provided by the kernel.

Examples include requesting the kernel to:

* Open a file
* Read or write data
* Create a process
* Request memory
* Send network data
* Wait for an event
* Obtain the current time
* Communicate with another process

---

## Why system calls exist

Applications need access to protected resources, but they cannot safely receive unrestricted kernel privilege.

System calls provide a narrow, checked interface.

```text id="ak7oxa"
User application
      │
      │ controlled request
      ▼
Kernel interface
      │
      ├── validate request
      ├── check permissions
      ├── manage resource
      └── return result
```

The system call lets the application ask for an operation without directly controlling the protected mechanism.

---

## The problem system calls solve

Suppose a text editor wants to read a file.

The editor cannot safely:

* Search raw storage structures itself
* Control the SSD directly
* Decide whether it has permission
* Modify kernel file-system data
* Access arbitrary physical memory

Instead, it asks the kernel:

> Open this file for me.

The kernel performs or rejects the request according to system rules.

---

## Mental model: a service counter

Imagine a secure records office.

A visitor cannot enter the archive room. Instead, the visitor submits a request at a service counter.

The employee:

1. Checks the request.
2. Verifies the visitor’s identity.
3. Checks permission.
4. Retrieves the record.
5. Returns an approved result.

| Records office      | Operating system        |
| ------------------- | ----------------------- |
| Visitor             | Application             |
| Request form        | System-call arguments   |
| Service counter     | System-call entry point |
| Authorized employee | Kernel                  |
| Archive             | Protected resource      |
| Returned document   | System-call result      |

---

# 6.3 Step-by-Step: A System Call

Suppose an application requests data from an already opened file.

## Step 1: Application prepares the request

The application identifies:

* Which open file it means
* How much data it wants
* Where the result should be placed

**CPU mode:** User mode

---

## Step 2: A library may help prepare the request

Applications commonly use a library that provides a convenient interface.

```text id="bg7op8"
Application
    │
    ▼
System library
    │
    ▼
System-call mechanism
```

The library is still normally executing in user mode.

---

## Step 3: The application executes a special entry operation

A processor-defined mechanism transfers control into the kernel.

The CPU does not jump to an arbitrary kernel instruction. It enters through a configured and approved entry point.

**CPU mode:** User mode → kernel mode

---

## Step 4: The CPU preserves essential user state

The CPU and kernel preserve enough information to later resume the application.

This may include:

* Current instruction position
* Processor status
* Stack-related information
* Relevant register values

---

## Step 5: The kernel identifies the requested service

The kernel determines which system operation the application requested.

For example:

```text id="z2yfne"
Requested operation: read file data
```

---

## Step 6: The kernel validates the arguments

The kernel checks questions such as:

* Does the file reference belong to this process?
* Is the requested memory region valid?
* Is the requested length reasonable?
* Is the process allowed to read?
* Is the resource still available?

Application-provided information is treated as untrusted.

---

## Step 7: The kernel performs or begins the operation

If the data is already available in memory, the request may finish quickly.

If storage access is necessary, the thread may need to wait.

---

## Step 8: The kernel prepares a result

Possible results include:

* Number of bytes read
* End of file
* Permission denied
* Invalid file reference
* Device error
* Interrupted operation

---

## Step 9: Control returns to user mode

The kernel restores the application’s execution state.

**CPU mode:** Kernel mode → user mode

---

## Step 10: The application handles the result

The program may:

* Process the data
* Request more data
* Display an error
* Stop the operation

---

## Flow diagram

```text id="tzot8e"
Application prepares request
            │
            ▼
Controlled kernel entry
            │
            ▼
Kernel identifies system call
            │
            ▼
Validate arguments and permissions
            │
       ┌────┴────┐
       │         │
    Reject     Perform
       │         │
       └────┬────┘
            ▼
Return result to application
```

---

# 6.4 System Calls Are Not Ordinary Function Calls

An application function call typically remains inside the same process and privilege level.

```text id="f910uu"
Application function A
          │
          ▼
Application function B

CPU mode remains USER
```

A system call crosses a protection boundary:

```text id="q1y6r9"
Application code
      │
      ▼
Special CPU entry mechanism
      │
      ▼
Kernel code

CPU mode changes USER → KERNEL
```

This requires additional protection and state-management work.

---

## Function calls may hide system calls

An application may call a convenient library function that eventually causes one or more system calls.

```text id="8kukij"
Application operation
       │
       ▼
Library processing
       │
       ├── ordinary user-mode work
       │
       └── system call when kernel help is needed
```

Not every library call is a system call.

Not every application operation requires entering the kernel.

---

# 6.5 System Calls Can Block

A system call is said to **block** when the calling thread cannot continue until some condition is satisfied.

For example, a thread asks to read network data, but no data has arrived.

```text id="z2em8d"
Thread running
     │
     │ system call requests data
     ▼
Data unavailable
     │
     ▼
Thread becomes waiting
```

The scheduler then runs another ready thread.

Later, when the data arrives:

```text id="qq2z8r"
Network data arrives
        │
        ▼
Kernel marks thread ready
        │
        ▼
Scheduler eventually runs it
        │
        ▼
System call completes
```

---

## Blocking does not mean the entire computer stops

Only the calling thread may wait.

Other threads and processes can continue.

```text id="7xw9ln"
Thread A: waiting for file
Thread B: updating interface
Process C: playing music
Process D: downloading data
```

If the blocked thread is an application’s only interface thread, the application may appear frozen even though the OS remains active.

---

# 6.6 Interrupts

## What an interrupt is

An **interrupt** is a signal that causes the CPU to temporarily stop its current execution flow and run a designated handler.

Interrupts commonly come from hardware, such as:

* Keyboard controller
* Storage device
* Network adapter
* Timer
* Audio device
* USB controller

An interrupt typically reports:

> An external event has occurred and requires software attention.

---

## Why interrupts exist

Without interrupts, the CPU would need to repeatedly ask each device:

* Has a key been pressed?
* Has storage finished?
* Has a network packet arrived?
* Has the timer expired?
* Is audio output ready?

This repeated checking is called **polling**.

```text id="s2vcjk"
CPU repeatedly checks:

Keyboard ready? No
Storage ready? No
Network ready? No
Keyboard ready? No
Storage ready? Yes
...
```

Polling can waste CPU time.

Interrupts allow a device to notify the CPU when something important happens.

```text id="hx26cr"
CPU performs useful work
          │
          │ device signals interrupt
          ▼
CPU handles device event
```

---

## Mental model: a doorbell

Without a doorbell, you might repeatedly open the door to check whether someone has arrived.

With a doorbell:

1. You continue other work.
2. A visitor arrives.
3. The bell alerts you.
4. You respond.

| Doorbell example | Computer                      |
| ---------------- | ----------------------------- |
| Visitor arrives  | Hardware event occurs         |
| Doorbell rings   | Interrupt signal              |
| You pause work   | CPU pauses current execution  |
| You answer door  | Kernel interrupt handler runs |
| You resume work  | Previous execution continues  |

---

# 6.7 Step-by-Step: A Hardware Interrupt

Suppose a storage device completes a read operation.

## Step 1: A thread previously requested file data

The kernel instructed the storage device to retrieve data.

The requesting thread may be waiting.

---

## Step 2: The CPU performs other work

Another thread may be running while the storage hardware works independently.

```text id="uvqo0p"
CPU: executes Browser thread
SSD: retrieves Editor file data
```

---

## Step 3: The storage operation completes

The device or its controller records completion.

---

## Step 4: The device raises an interrupt

The hardware sends an interrupt request to the CPU through the system’s interrupt-controller mechanisms.

---

## Step 5: The CPU pauses its current execution

The CPU preserves essential state and transfers control to a kernel interrupt entry point.

**CPU mode:** Usually user mode → kernel mode, or kernel mode remains kernel mode if the CPU was already there

---

## Step 6: The kernel runs an interrupt handler

The kernel determines:

* Which device reported the event
* What operation completed
* Whether an error occurred
* Which waiting request is affected

---

## Step 7: The kernel acknowledges or clears the interrupt

The device and interrupt controller must know that the event is being handled.

Otherwise, the same interrupt might continue to be reported.

---

## Step 8: The kernel updates thread state

The file-reading thread may change from waiting to ready.

```text id="heusbk"
WAITING → READY
```

---

## Step 9: The scheduler decides what runs

The interrupted thread may resume, or the newly ready thread may eventually run.

An interrupt does not guarantee that the awakened thread runs immediately.

---

## Timeline

```text id="uorbea"
Time ─────────────────────────────────────────────────────▶

Editor:   requests read ─────waiting──────────────ready
Browser:                 running────paused────possibly resumes
Storage:                 reading────complete
Interrupt:                              ↑
Kernel:                          handle completion
```

---

# 6.8 Timer Interrupts

A timer interrupt is especially important for multitasking.

The kernel configures hardware to report a timer event.

When the timer interrupt occurs, the kernel can:

* Update timekeeping
* Check expired timers
* Wake sleeping threads
* Review CPU scheduling
* Preempt a running thread

```text id="6scrkr"
Thread A runs in user mode
           │
           │ timer interrupt
           ▼
Kernel scheduler runs
           │
      ┌────┴────┐
      │         │
Resume A     Run B
```

Without a reliable way to regain CPU control, an application could run indefinitely.

---

# 6.9 Interrupt Priority and Masking

Not every interrupt has equal urgency.

Hardware and operating systems may assign different priorities.

For example:

* A critical hardware condition may require rapid attention.
* A routine device event may tolerate a short delay.

---

## Interrupt masking

To **mask** an interrupt means to temporarily prevent or delay delivery of certain interrupt types.

The kernel may briefly mask interrupts while modifying sensitive internal state.

However, keeping interrupts disabled for too long can cause:

* Delayed device handling
* Poor responsiveness
* Lost timing guarantees
* Data loss on some devices

---

## Simplified model

```text id="35kh1d"
Interrupt occurs
       │
       ▼
Is it currently allowed?
    ┌──┴──┐
    │     │
   Yes    No
    │     │
Handle   Delay until allowed
```

Exact interrupt behavior depends on hardware and OS design.

---

# 6.10 Interrupt Handlers Should Usually Be Brief

An interrupt may arrive while important work is running.

If the kernel spends too long handling it immediately, other work is delayed.

Operating systems commonly divide device handling conceptually into:

1. Urgent immediate work
2. Deferred work that can happen later

```text id="c2zgwm"
Interrupt arrives
      │
      ▼
Immediate handler
├── acknowledge device
├── record essential state
└── schedule remaining work
      │
      ▼
Deferred processing later
```

The names and exact mechanisms differ between systems.

The principle is:

> Do only the time-critical work immediately, then defer slower processing when possible.

---

# 6.11 Polling Versus Interrupts

| Property                                | Polling                              | Interrupts                        |
| --------------------------------------- | ------------------------------------ | --------------------------------- |
| How events are discovered               | CPU repeatedly checks                | Device notifies CPU               |
| CPU use while nothing happens           | May waste work                       | CPU can perform other work        |
| Response predictability                 | Controlled by polling frequency      | Depends on interrupt latency      |
| Overhead with extremely frequent events | Can sometimes be efficient           | Too many interrupts can be costly |
| Typical use                             | Some high-speed or simple situations | Many general hardware events      |

Interrupts are not always automatically superior.

For extremely frequent events, processing an interrupt for each event may create excessive overhead. Some systems combine interrupts and polling.

---

# 6.12 Exceptions

## What an exception is

An **exception** is an event caused by the execution of the current CPU instruction or its immediate execution context.

Examples include:

* Division by zero
* Invalid instruction
* Forbidden memory access
* Missing virtual-memory page
* Debugging breakpoint
* Arithmetic overflow on architectures that report it
* Deliberate system-call instruction

The terminology varies: some CPU manuals classify system-call entry as a type of exception. For foundational OS study, it is useful to discuss system calls separately because they are intentional service requests.

---

## Why exceptions exist

The CPU must report conditions that cannot be handled as ordinary instruction completion.

Suppose an instruction tries to access forbidden memory.

The CPU must not simply continue as though the operation succeeded.

Instead:

1. It stops the instruction’s normal progress.
2. It records what happened.
3. It enters the kernel.
4. The kernel decides how to respond.

---

## Mental model: a factory safety stop

A machine performs a sequence of operations.

If a sensor detects:

* An obstruction
* Invalid material
* Unsafe pressure
* Missing component

The machine stops its normal operation and alerts a supervisor.

| Factory              | Computer                    |
| -------------------- | --------------------------- |
| Machine operation    | CPU instruction             |
| Safety sensor        | CPU protection mechanism    |
| Safety stop          | Exception                   |
| Supervisor           | Kernel exception handler    |
| Restart or shut down | Resume or terminate process |

---

# 6.13 Synchronous Versus Asynchronous Events

A useful distinction is whether the event is directly tied to the current instruction.

## Synchronous event

Occurs because of the instruction being executed.

Examples:

* Division by zero
* Invalid memory access
* Page fault
* System call

If the same instruction runs under the same relevant conditions, the event generally occurs again.

---

## Asynchronous event

Occurs independently of the current instruction.

Examples:

* Network packet arrival
* Keyboard input
* Timer interrupt
* Storage completion

The event could arrive between almost any two instructions.

---

## Comparison

| Event              | Relationship to current instruction |
| ------------------ | ----------------------------------- |
| System call        | Deliberately caused by it           |
| Exception          | Directly caused by it               |
| Hardware interrupt | Usually independent of it           |

```text id="p9p6a0"
Current instruction
      │
      ├── deliberately requests kernel → system call
      ├── encounters execution condition → exception
      └── unrelated hardware event arrives → interrupt
```

---

# 6.14 Faults, Traps, and Aborts

Some technical descriptions divide exceptions into categories such as:

* Faults
* Traps
* Aborts

Exact terminology depends on processor architecture, so treat this as a conceptual introduction.

---

## Fault

A fault may be correctable.

The kernel may fix the condition and retry the instruction.

Example:

* A valid page is not currently mapped into physical memory.

```text id="gm9y7u"
Instruction attempts access
          │
          ▼
Page fault
          │
          ▼
Kernel makes page available
          │
          ▼
Instruction is retried
```

---

## Trap

A trap is often reported after or as part of an intentional instruction event.

Examples may include:

* Debugging breakpoints
* Some system-call mechanisms

The exact definition differs by architecture.

---

## Abort

An abort represents a severe condition for which reliable continuation may not be possible.

Example conceptually:

* Serious hardware or processor-state failure

Do not rely too strongly on these labels across all operating systems and CPUs. The general idea—recoverable versus nonrecoverable exceptions—is more important.

---

# 6.15 Step-by-Step: Division by Zero

Suppose a program attempts an integer division by zero on hardware that reports this as an exception.

## Step 1: The application thread is running

**CPU mode:** User mode

---

## Step 2: The CPU begins the division instruction

The CPU examines the operands.

---

## Step 3: The CPU detects an invalid condition

The divisor is zero.

The CPU does not complete a normal division result.

---

## Step 4: An exception occurs

The CPU preserves relevant state and enters a kernel exception handler.

**CPU mode:** User → kernel

---

## Step 5: The kernel identifies the exception

The kernel knows:

* Which process caused it
* Which thread was running
* Which instruction was involved
* What kind of error occurred

---

## Step 6: The kernel applies OS policy

Possible outcomes include:

* Notify the process
* Invoke an application-installed handler
* Terminate the process
* Record diagnostic information

---

## Step 7: Other processes continue

Because the error occurred in a user process, the OS normally isolates the failure.

```text id="921mhf"
Application instruction
        │
        ▼
CPU detects division error
        │
        ▼
Kernel exception handler
        │
        ▼
Notify, recover, or terminate
```

---

# 6.16 Step-by-Step: Invalid Memory Access

Suppose a thread attempts to write to memory it does not own.

## Step 1: The instruction requests a memory write

The CPU calculates the target virtual address.

---

## Step 2: Memory-protection hardware checks the address

It examines rules configured by the kernel.

---

## Step 3: The write is forbidden

Examples:

* The address is unmapped.
* The memory is read-only.
* The page belongs to protected kernel space.
* The process lacks required access.

---

## Step 4: The CPU raises an exception

The invalid write does not complete normally.

---

## Step 5: The kernel examines the cause

The kernel distinguishes between situations such as:

* A valid page that can be created or loaded
* A copy-on-write operation that should be resolved
* A genuinely invalid access
* A stack-growth situation allowed by policy

---

## Step 6: The kernel resolves or rejects it

```text id="6xpqvj"
Memory exception
      │
      ▼
Can kernel validly resolve it?
      ├── Yes → update mapping → retry instruction
      └── No  → report fatal error or terminate process
```

Not every memory exception means a program bug.

This becomes important when studying page faults.

---

# 6.17 Page Faults Are Exceptions

A **page fault** occurs when a memory access cannot proceed using the current virtual-memory mapping.

The name sounds like a failure, but many page faults are normal.

For example:

1. A process accesses a valid part of a program.
2. That part is not currently in physical RAM.
3. The CPU raises a page-fault exception.
4. The kernel loads or maps the required page.
5. The instruction is retried.
6. Execution continues.

```text id="ctaxoh"
Valid memory access
       │
       ▼
Page currently unavailable
       │
       ▼
Page-fault exception
       │
       ▼
Kernel makes page available
       │
       ▼
Instruction retries successfully
```

A page fault is only fatal when the kernel determines that the access is invalid or cannot be satisfied.

---

# 6.18 System Call, Interrupt, and Exception Compared

| Question                     | System call             | Interrupt                           | Exception                          |
| ---------------------------- | ----------------------- | ----------------------------------- | ---------------------------------- |
| Who initiates it?            | Application instruction | Hardware/device/timer               | CPU execution condition            |
| Is it intentional?           | Usually yes             | Usually unrelated to program intent | Sometimes, but often no            |
| Tied to current instruction? | Yes                     | Usually no                          | Yes                                |
| Typical purpose              | Request kernel service  | Report external event               | Report special execution condition |
| Can it enter kernel mode?    | Yes                     | Yes                                 | Yes                                |
| Can it cause scheduling?     | Yes                     | Yes                                 | Yes                                |
| Example                      | Read a file             | Network packet arrives              | Invalid memory access              |

---

# 6.19 One Event Can Lead to Another

These mechanisms often interact.

Consider reading a file from storage.

## Stage 1: System call

The application asks the kernel to read the file.

```text id="ngkcg6"
Application → system call → kernel
```

---

## Stage 2: Thread waits

The data is not ready, so the kernel marks the thread waiting.

---

## Stage 3: Context switch

The scheduler selects another ready thread.

---

## Stage 4: Hardware interrupt

The storage device completes the operation and interrupts the CPU.

---

## Stage 5: Kernel wakes the thread

The kernel changes the original thread from waiting to ready.

---

## Stage 6: Another context switch

The scheduler eventually restores the original thread.

---

## Stage 7: System call returns

The application receives the requested data.

---

## Full timeline

```text id="cc21wr"
Application:
request file ───────waiting────────────────── resumes
     │                                             ▲
     │ system call                                 │ return
     ▼                                             │
Kernel:
start I/O → schedule other work → handle interrupt
                                      ▲
                                      │
Storage:
             read data ───────────── complete
```

This combines:

* A system call
* A waiting-state transition
* A context switch
* A hardware interrupt
* Another scheduling decision

---

# 6.20 Detailed Walkthrough: What Happens When a Process Reads a File

Assume the process already has permission to access a file.

## 1. Application decides it needs data

The application prepares a request containing:

* An OS-issued file reference
* Requested amount
* Destination memory area

**Component:** Application
**Mode:** User mode

---

## 2. A system-call transition occurs

A library or application instruction enters the kernel through an approved entry point.

**Component:** CPU and system-call interface
**Mode:** User → kernel

---

## 3. Kernel validates the request

The kernel checks:

* The file reference
* Requested operation
* Destination memory
* Permissions
* Length and boundaries

**Component:** Kernel file and memory subsystems

---

## 4. File-system layer determines the required data

The file system translates the logical file position into the relevant stored data.

**Component:** File system

---

## 5. Kernel checks whether data is cached

The operating system may already have the required file data in RAM.

### Cache hit

```text id="svvt72"
Required data already in RAM
          │
          ▼
Kernel supplies it without storage access
```

### Cache miss

```text id="6xs5iy"
Required data absent from RAM
          │
          ▼
Storage operation needed
```

---

## 6. If necessary, the storage driver receives a request

The driver prepares device-specific commands.

**Component:** Storage driver

---

## 7. Storage hardware begins the operation

The storage controller and device retrieve the data.

**Component:** Hardware

---

## 8. Calling thread waits

If the operation cannot complete immediately, the kernel marks the thread waiting.

```text id="ygf1rm"
RUNNING → WAITING
```

**Component:** Kernel scheduler

---

## 9. Another ready thread runs

The CPU is not left idle while storage works.

**Component:** Scheduler and context-switch mechanism

---

## 10. Device completes the read

The data is transferred into memory controlled by the kernel and hardware.

**Component:** Device, controller, and memory subsystem

---

## 11. Device generates an interrupt

The CPU transfers control to the kernel.

**Component:** Interrupt controller, CPU, and kernel interrupt handler

---

## 12. Kernel records completion

The kernel checks:

* Whether the operation succeeded
* How much data arrived
* Which request it belongs to

---

## 13. Original thread becomes ready

```text id="8xhn8l"
WAITING → READY
```

---

## 14. Scheduler eventually chooses it

Its saved context is restored.

```text id="uanhj0"
READY → RUNNING
```

---

## 15. System call returns

The application receives:

* The data
* The amount read
* Or an error result

**Mode:** Kernel → user

---

## Complete component flow

```text id="n4yyr7"
Application
    │ system call
    ▼
Kernel system-call handler
    │
    ▼
File system
    │
    ▼
Memory cache check
    │ cache miss
    ▼
Storage driver
    │
    ▼
Storage hardware
    │ interrupt on completion
    ▼
Kernel interrupt handler
    │
    ▼
Scheduler wakes application thread
    │
    ▼
Application resumes
```

---

# 6.21 Exceptions Can Be Recoverable or Fatal

## Recoverable exception

The kernel can repair the condition.

Example:

* A valid memory page needs to be mapped.

```text id="nuk2ih"
Exception → kernel repairs state → instruction retries
```

---

## Application-level failure

The current application cannot safely continue.

Example:

* Illegal memory access

```text id="eawxi0"
Exception → kernel terminates process → others continue
```

---

## Kernel-level failure

The exception occurred in privileged kernel code and cannot be safely handled.

```text id="3uhng5"
Kernel exception → system may stop or restart
```

The same type of hardware exception can have very different consequences depending on where and why it occurred.

---

# 6.22 Nested Events

An event handler may itself be interrupted or encounter an exception.

For example:

1. The kernel handles a storage interrupt.
2. A higher-priority interrupt arrives.
3. The CPU temporarily handles the higher-priority event.
4. The original handler later continues.

```text id="e06h03"
Normal program
      │
      ▼
Storage interrupt handler
      │
      ▼
Higher-priority timer interrupt
      │
      ▼
Return to storage handler
      │
      ▼
Return to normal execution
```

Nested handling requires careful state management.

Operating systems may restrict which events can interrupt particular kernel operations.

---

# 6.23 Interrupt Latency

**Interrupt latency** is the delay between a hardware event occurring and the CPU beginning appropriate handling.

```text id="3pm320"
Hardware event occurs
        │
        │ delay
        ▼
Interrupt handler begins
```

Sources of latency can include:

* Higher-priority work
* Temporarily masked interrupts
* Another handler already running
* Hardware communication delay
* Kernel scheduling design

Low interrupt latency matters for time-sensitive devices and real-time systems.

---

# 6.24 System-Call Overhead

A system call requires more work than an ordinary user-mode function call.

Possible costs include:

* Switching privilege mode
* Saving and validating state
* Checking arguments
* Permission enforcement
* Kernel bookkeeping
* Returning to user mode

This does not mean system calls should be avoided entirely. They are necessary for protected services.

Applications and libraries may reduce overhead by:

* Grouping operations
* Buffering data
* Reusing opened resources
* Avoiding unnecessary tiny requests

---

# 6.25 What Can Go Wrong?

## Invalid system-call arguments

An application may provide:

* Invalid memory addresses
* Impossible sizes
* Expired resource identifiers
* Unsupported options

The kernel must reject these safely.

---

## Vulnerable system-call handler

If the kernel incorrectly validates application input, a malicious process may:

* Read protected memory
* Corrupt kernel state
* Gain additional privilege
* Crash the system

System-call interfaces are security-sensitive boundaries.

---

## Interrupt storm

An **interrupt storm** occurs when interrupts arrive so frequently that the CPU spends excessive time handling them.

Possible causes include:

* Faulty hardware
* Misconfigured device
* Driver defect
* Extremely high event rate

Effects may include:

* Poor responsiveness
* Reduced application performance
* High CPU use
* System instability

---

## Lost or mishandled interrupt

If the OS or device mishandles an interrupt:

* A waiting thread may never wake.
* Data may not be processed.
* A device may appear frozen.
* An operation may time out.

---

## Interrupt handler runs too long

Long handlers delay:

* Other interrupts
* Application execution
* Scheduler activity
* Time-sensitive work

---

## Exception loop

Suppose the kernel resumes a thread without correcting the cause of a recoverable-looking exception.

The same instruction may fail repeatedly:

```text id="fuw47q"
Instruction fails
      │
      ▼
Exception handled incorrectly
      │
      ▼
Instruction retried
      │
      ▼
Same exception again
```

---

## Kernel exception

A serious exception in kernel mode can affect the whole system because privileged state may be corrupted.

---

## Incorrect recovery

Attempting to continue after a fatal application error can cause further corruption.

Sometimes terminating a process is safer than trying to recover.

---

# 6.26 Common Misconceptions

## Misconception: “An interrupt always comes from hardware”

Many foundational explanations use “interrupt” specifically for asynchronous hardware events.

However, terminology varies across systems and processor documentation. Some sources use broader terms.

For this course, use:

* **Interrupt:** asynchronous external event
* **Exception:** synchronous CPU-detected event
* **System call:** intentional application request

---

## Misconception: “An interrupt means something went wrong”

No.

Normal events generate interrupts:

* Keyboard input
* Completed storage read
* Network packet arrival
* Timer expiration

Interrupts are a standard coordination mechanism.

---

## Misconception: “Every exception crashes the process”

No.

Some exceptions are expected and recoverable.

Page faults are a major example.

---

## Misconception: “A page fault always means invalid memory”

No.

A page fault means the current memory translation cannot satisfy the access.

The access may be valid and resolved by the kernel.

---

## Misconception: “A system call lets application code run in kernel mode”

The application triggers a controlled transition, but trusted kernel code handles the request.

The application’s arbitrary code is not granted kernel privilege.

---

## Misconception: “The CPU checks every device continuously”

Devices commonly report events using interrupts, allowing the CPU to perform other work.

Some polling still occurs where appropriate.

---

## Misconception: “An interrupt always causes a context switch”

No.

The kernel may handle the interrupt and return to the same thread.

```text id="cc4fna"
Thread A
   │ interrupt
   ▼
Kernel handler
   │
   ▼
Thread A resumes
```

A context switch occurs only if the scheduler selects a different thread.

---

## Misconception: “A system call always blocks”

Some system calls complete immediately.

Others block only when the requested condition is not ready.

---

# 6.27 Real-World Analogy: Office Work

Imagine an employee working at a desk.

## System call

The employee deliberately submits a request to an authorized department.

> “Please retrieve this protected record.”

---

## Interrupt

A phone rings unexpectedly.

> “A delivery has arrived.”

The event is external to the employee’s current task.

---

## Exception

The employee discovers that the current form contains an impossible value.

> “This calculation cannot continue under the current conditions.”

The event arises from the current work itself.

---

## Comparison

```text id="oprrjy"
Employee requests service       → system call
External phone rings            → interrupt
Current task encounters problem → exception
```

---

# 6.28 Connection to Earlier Concepts

## Connection to user and kernel mode

All three mechanisms can transfer execution into kernel mode.

```text id="ybay9v"
User execution
      │
      ├── system call
      ├── interrupt
      └── exception
      │
      ▼
Kernel execution
```

---

## Connection to scheduling

A timer interrupt gives the scheduler an opportunity to preempt a thread.

A blocking system call may move a thread to waiting.

A device interrupt may move it back to ready.

An exception may terminate it.

---

## Connection to context switching

An event may cause:

* No context switch
* A switch to another thread
* Termination of the current process
* Resumption of the same instruction

The event and the scheduling decision are separate concepts.

---

## Connection to processes

The kernel identifies which process and thread caused a system call or exception.

It also tracks which waiting thread should be awakened by a device interrupt.

---

## Connection to hardware

The CPU implements:

* Controlled kernel entry
* Privilege changes
* Exception detection
* State preservation

Devices and interrupt controllers report asynchronous events.

---

# 6.29 Simplified Model Versus Technical Reality

## Simplified model

```text id="cl1bid"
System call = application asks
Interrupt   = hardware notifies
Exception   = CPU detects a condition
```

This is the model to retain.

---

## More exact reality

Terminology varies across CPU architectures and operating systems.

For example:

* A processor manual may classify system calls as software-generated exceptions.
* Some events called traps in one architecture may have different names elsewhere.
* Interrupt handling can involve multiple hardware controllers.
* Devices may group many events into one interrupt.
* High-performance I/O may combine interrupts and polling.
* Some exception handling occurs first in firmware or a hypervisor.
* Modern kernels defer substantial interrupt work rather than performing it immediately.

The essential distinction is based on cause:

* Intentional protected-service request
* Asynchronous external event
* Synchronous execution condition

---

# 6.30 Core Mental Model

Keep this decision tree:

```text id="y2iwnq"
Why did control enter the kernel?
              │
      ┌───────┼────────┐
      │       │        │
Application  Hardware  Current instruction
asked        signaled  caused condition
      │       │        │
System call Interrupt Exception
```

And remember the full I/O pattern:

```text id="hbz8fx"
Application makes system call
            │
            ▼
Kernel starts device operation
            │
            ▼
Application thread waits
            │
            ▼
Another thread runs
            │
            ▼
Device sends interrupt
            │
            ▼
Kernel wakes original thread
            │
            ▼
System call returns
```

This interaction connects applications, the kernel, devices, scheduling, and context switching.

The next section introduces **physical memory and virtual memory**—how the OS gives each process a protected memory view even though all processes share the same RAM hardware.

# Learning Check

Do not look for answers yet.

## Conceptual questions

1. What is the main difference between a system call, an interrupt, and an exception?
2. Why does a system call need a controlled kernel entry point rather than allowing an application to jump anywhere in kernel code?
3. What is the difference between a synchronous event and an asynchronous event?

## Cause-and-effect questions

4. Why can a blocking file-read system call cause the calling thread to move from running to waiting?
5. Why can an interrupt storm reduce application performance even though interrupts are designed to make device handling efficient?

## Misconception question

6. A student says, “Every page fault means that the application accessed illegal memory and must be terminated.” What is wrong with this statement?

## Scenario-based question

7. A process requests network data that has not arrived yet. Later, the network adapter receives the data. Describe the system call, thread-state changes, interrupt, scheduler involvement, and eventual return to user mode.

# 7. Physical Memory and Virtual Memory

Programs need memory while they run.

The computer has a limited amount of physical RAM, but many processes may need memory simultaneously. Each process must also be prevented from reading or overwriting another process’s private data.

The operating system solves this using **virtual memory**.

```text
Processes see virtual memory
          │
          ▼
Kernel configures mappings
          │
          ▼
Hardware translates addresses
          │
          ▼
Physical RAM is accessed
```

---

# 7.1 What Is Physical Memory?

## What it is

**Physical memory** usually refers to the computer’s actual RAM hardware.

RAM contains many storage locations that can temporarily hold:

* Program instructions
* Application data
* Kernel data
* File-system cache
* Device-related data
* Temporary calculation results

```text
Physical RAM
┌─────────────────────────────┐
│ Location 0                  │
│ Location 1                  │
│ Location 2                  │
│ ...                         │
│ Location N                  │
└─────────────────────────────┘
```

Each physical location can be identified by a **physical address**.

---

## Physical address

A **physical address** identifies a location in actual RAM.

A simplified example:

```text
Physical address 1000 → some bytes in RAM
Physical address 1001 → nearby bytes
Physical address 1002 → nearby bytes
```

Real systems use much larger address values.

Applications normally do not freely choose physical addresses.

The kernel and memory-management hardware control how application memory accesses reach physical RAM.

---

## Why physical memory exists

The CPU needs fast temporary storage for active instructions and data.

Long-term storage such as an SSD is much slower than RAM.

A simplified comparison:

| Property     | RAM                        | SSD               |
| ------------ | -------------------------- | ----------------- |
| Main purpose | Active working memory      | Long-term storage |
| Speed        | Very fast                  | Slower            |
| Power loss   | Contents usually disappear | Contents persist  |
| Capacity     | Usually smaller            | Usually larger    |

RAM acts as the computer’s active workspace.

---

## Mental model: a physical warehouse

Imagine RAM as a warehouse containing numbered shelves.

```text
Warehouse shelves
┌──────────┬──────────┬──────────┬──────────┐
│ Shelf 0  │ Shelf 1  │ Shelf 2  │ Shelf 3  │
└──────────┴──────────┴──────────┴──────────┘
```

The shelves are real physical storage locations.

Many processes need to use this same warehouse, so some manager must decide:

* Which shelves are available
* Which shelves belong to which process
* Which shelves contain kernel data
* Whether a process may access a shelf

That manager is the operating system, assisted by CPU hardware.

---

# 7.2 The Problem With Giving Applications Physical Addresses

Suppose applications directly used physical RAM locations.

Process A might be told:

```text
Use physical addresses 1000–1999
```

Process B might be told:

```text
Use physical addresses 2000–2999
```

This appears simple, but it creates several problems.

---

## Problem 1: Relocation

A program might expect to use one memory location, but that location may already be occupied.

```text
Program expects:  addresses 1000–1999
Actual free RAM:  addresses 7000–7999
```

The program would need to be rewritten or adjusted depending on where it was loaded.

---

## Problem 2: Isolation

A faulty or malicious application might access another process’s physical memory.

```text
Process A owns physical 1000–1999
Process B attempts physical 1500
```

Without hardware protection, Process B could read or overwrite Process A’s data.

---

## Problem 3: Fragmentation

Free memory may be divided into separated regions.

```text
Physical RAM
[Used][Free][Used][Free][Used][Free]
```

There may be enough total free memory, but not one large continuous region.

---

## Problem 4: Moving processes

The OS may need to rearrange memory usage.

If applications directly depend on physical addresses, moving their data would be difficult.

---

## Problem 5: Sharing selected memory

Sometimes two processes should share specific information while keeping everything else private.

Direct physical addressing makes controlled sharing harder to manage safely.

---

## Problem 6: Using more memory than is currently in RAM

Some memory contents can temporarily live on storage rather than RAM.

Applications should not need to understand where every piece of memory currently resides.

Virtual memory helps hide this detail.

---

# 7.3 What Is Virtual Memory?

## What it is

**Virtual memory** is a memory abstraction that gives each process its own view of addresses.

A process uses **virtual addresses**.

The hardware and operating system translate those virtual addresses into physical locations.

```text
Process virtual address
          │
          ▼
Address translation
          │
          ▼
Physical RAM address
```

---

## Virtual address

A **virtual address** is an address used by a process inside its own virtual address space.

For example:

```text
Process A: virtual address 5000
Process B: virtual address 5000
```

These identical-looking virtual addresses can refer to different physical memory.

```text
Process A virtual 5000 → physical 12000
Process B virtual 5000 → physical 47000
```

The two processes do not automatically access the same data.

---

# 7.4 Virtual Address Space

A **virtual address space** is the complete range of virtual addresses available to a process.

Each process normally receives its own address space.

```text
Process A virtual address space
┌────────────────────────────┐
│ Program instructions       │
│ Program data               │
│ Heap                       │
│ Shared libraries           │
│ Thread stacks              │
└────────────────────────────┘

Process B virtual address space
┌────────────────────────────┐
│ Program instructions       │
│ Program data               │
│ Heap                       │
│ Shared libraries           │
│ Thread stacks              │
└────────────────────────────┘
```

The layouts may look similar, but their mappings can differ.

---

## Why virtual address spaces exist

They provide several major benefits:

* Process isolation
* Simpler memory layout
* Flexible physical-memory placement
* Controlled sharing
* Efficient program loading
* Support for memory larger than currently available RAM
* Protection of kernel memory
* Easier process creation and management

---

# 7.5 Mental Model: Private Numbering Systems

Imagine a hotel where every guest receives a private map.

On each guest’s map:

```text
Room 1
Room 2
Room 3
```

However, the hotel manager maps these private room numbers to actual physical rooms.

```text
Guest A room 1 → physical room 204
Guest B room 1 → physical room 518
```

Both guests can refer to “room 1,” but they mean different physical locations.

| Hotel concept               | Memory concept          |
| --------------------------- | ----------------------- |
| Guest’s private room number | Virtual address         |
| Actual hotel room           | Physical address        |
| Guest’s map                 | Process address mapping |
| Hotel manager               | Kernel                  |
| Access-control system       | CPU memory hardware     |

The guest does not need to know the actual building coordinates.

Similarly, a process usually works with virtual addresses rather than physical RAM locations.

---

# 7.6 Address Translation

When a thread accesses memory, the CPU must translate the virtual address.

A simplified path is:

```text
Thread instruction
      │
      │ uses virtual address
      ▼
Memory-management hardware
      │
      │ consults mapping
      ▼
Physical address
      │
      ▼
RAM
```

The CPU component responsible for much of this work is commonly called the **memory management unit**, or **MMU**.

---

# 7.7 The Memory Management Unit

## What it is

The **MMU** is processor hardware that helps translate virtual addresses into physical addresses and enforce memory-access rules.

It helps answer:

* Does this virtual address have a valid mapping?
* Which physical memory location does it refer to?
* May the current process read it?
* May it write to it?
* May it execute instructions from it?
* Is the address currently available?

---

## Why hardware performs translation

Memory accesses happen extremely frequently.

If the kernel had to manually inspect every load and store operation, execution would be far too slow.

Instead:

1. The kernel creates and manages mapping information.
2. The MMU applies those mappings during ordinary execution.
3. The kernel is notified only when special handling is required.

```text
Kernel configures rules occasionally
                │
                ▼
MMU enforces rules on every access
```

This resembles user-mode privilege enforcement:

> The kernel defines the policy, while hardware enforces it efficiently.

---

# 7.8 Page Tables

The operating system needs a way to describe virtual-to-physical mappings.

A commonly used structure is a **page table**.

A page table records how regions of virtual memory correspond to physical memory.

A simplified example:

| Virtual region | Physical region      | Permissions      |
| -------------- | -------------------- | ---------------- |
| 0–999          | 8000–8999            | Read and execute |
| 1000–1999      | 12000–12999          | Read and write   |
| 2000–2999      | Not currently mapped | None             |
| 3000–3999      | 5000–5999            | Read only        |

Real page tables operate on fixed-size units called pages rather than arbitrary ranges.

Paging will be examined fully in the next section.

---

## Mental model

A page table is like a translation directory:

```text
Virtual page 4 → Physical frame 18
Virtual page 5 → Physical frame 72
Virtual page 6 → Not present
```

The kernel controls this directory.

The process cannot normally rewrite its own mapping rules freely.

---

# 7.9 Pages and Frames: Preview

Virtual memory is usually divided into fixed-size blocks called **pages**.

Physical RAM is divided into corresponding blocks commonly called **frames** or **page frames**.

```text
Virtual memory pages       Physical RAM frames

┌───────────────┐          ┌───────────────┐
│ Virtual page 0│─────────▶│ Frame 8       │
├───────────────┤          ├───────────────┤
│ Virtual page 1│─────────▶│ Frame 2       │
├───────────────┤          ├───────────────┤
│ Virtual page 2│─────────▶│ Frame 15      │
└───────────────┘          └───────────────┘
```

Virtual pages do not need to map to physically adjacent frames.

```text
Virtual page 0 → Physical frame 30
Virtual page 1 → Physical frame 4
Virtual page 2 → Physical frame 81
```

To the process, the virtual pages may still appear consecutive.

---

# 7.10 Continuous Virtual Memory, Scattered Physical Memory

Suppose a process needs three consecutive pages.

Its virtual view can be:

```text
Virtual pages:
[Page 10][Page 11][Page 12]
```

But physical RAM may contain them in scattered frames:

```text
Physical RAM:
[Frame 2: Page 11]
[Frame 9: Page 10]
[Frame 27: Page 12]
```

The mapping hides the physical arrangement.

```text
Virtual page 10 ──▶ Physical frame 9
Virtual page 11 ──▶ Physical frame 2
Virtual page 12 ──▶ Physical frame 27
```

This reduces the need for large continuous physical regions.

---

# 7.11 Step-by-Step: A Normal Memory Read

Suppose a thread wants to read data from virtual address `V`.

## Step 1: Application instruction uses a virtual address

The program executes an instruction conceptually meaning:

> Read the value at virtual address V.

**CPU mode:** User mode

---

## Step 2: The MMU examines the address

The MMU determines which virtual page contains `V`.

It also identifies the position inside that page.

---

## Step 3: The MMU finds the mapping

It obtains information describing:

* The corresponding physical frame
* Whether the page is present
* Whether reading is allowed
* Whether user-mode access is allowed

---

## Step 4: Permission is checked

If the page is readable by this process, access continues.

---

## Step 5: A physical address is formed

The MMU combines:

* The mapped physical frame
* The position inside the page

---

## Step 6: RAM is accessed

The physical memory location supplies the requested data.

---

## Step 7: The application continues

The process receives the value without entering the kernel.

```text
Application virtual address
           │
           ▼
MMU finds valid mapping
           │
           ▼
Permission check succeeds
           │
           ▼
Physical RAM accessed
           │
           ▼
Application continues
```

The kernel does not need to intervene in every valid memory access.

---

# 7.12 Step-by-Step: A Forbidden Memory Read

Suppose Process A tries to access an address that is not valid in its address space.

## Step 1: Process A executes a memory instruction

The instruction uses a virtual address.

---

## Step 2: MMU checks Process A’s current mappings

The active page table does not permit this access.

---

## Step 3: Hardware blocks the access

The forbidden read does not complete normally.

---

## Step 4: CPU raises an exception

A memory-related exception transfers control to the kernel.

**CPU mode:** User → kernel

---

## Step 5: Kernel investigates

The kernel determines whether:

* The access is valid but the page must be prepared
* The page should be loaded from storage
* A copy-on-write operation is needed
* The address is genuinely illegal

---

## Step 6: Kernel resolves or rejects

```text
Memory access exception
          │
          ▼
Is the access valid and recoverable?
      ┌───┴───┐
      │       │
     Yes      No
      │       │
Map page   Report failure
      │       │
Retry      Possibly terminate process
```

The exception mechanism protects other processes and the kernel.

---

# 7.13 How Process Isolation Works

Suppose Process A and Process B both use virtual address `4000`.

Their mappings can be:

```text
Process A page table:
Virtual 4000 → Physical 12000

Process B page table:
Virtual 4000 → Physical 35000
```

When Process A runs, the CPU uses Process A’s mappings.

When Process B runs, the CPU uses Process B’s mappings.

```text
Thread A running
      │
      ▼
Use Process A memory mappings

Context switch

Thread B running
      │
      ▼
Use Process B memory mappings
```

This is one reason a context switch between processes may require changing memory-management state.

---

## Isolation is not merely different addresses

The important protection is that Process A cannot normally ask the MMU to use Process B’s mappings.

The kernel controls which address space is active.

Hardware then enforces that choice.

---

# 7.14 Kernel Memory Protection

The kernel’s own instructions and data must also be protected.

A process’s address space may conceptually include:

```text
┌────────────────────────────┐
│ Kernel-controlled region   │
│ User access forbidden      │
├────────────────────────────┤
│ User-process region        │
│ Access depends on mapping  │
└────────────────────────────┘
```

The exact organization differs across operating systems.

When the CPU runs application code in user mode:

* Kernel-only pages are inaccessible.
* User pages are accessible according to their permissions.

When trusted kernel code executes:

* It can access protected kernel memory.
* It may also access process memory carefully when servicing requests.

---

# 7.15 Memory Permissions

Virtual-memory mappings can carry permissions.

Common conceptual permissions include:

* Read
* Write
* Execute
* User-accessible
* Kernel-only

---

## Read permission

The process may retrieve data from the page.

---

## Write permission

The process may modify the page.

---

## Execute permission

The CPU may treat bytes in the page as executable instructions.

---

## Example mapping

| Memory region        |          Read |         Write |       Execute |
| -------------------- | ------------: | ------------: | ------------: |
| Program instructions |           Yes |    Usually no |           Yes |
| Program data         |           Yes |           Yes |    Usually no |
| Read-only constants  |           Yes |            No |    Usually no |
| Thread stack         |           Yes |           Yes |    Usually no |
| Kernel code          | User mode: no | User mode: no | User mode: no |

These policies make accidental and malicious behavior harder.

---

## Why writable and executable are often separated

Suppose arbitrary data can also be executed as instructions.

An attacker might place malicious instructions into writable memory and then run them.

Systems therefore often try to separate:

```text
Instruction memory: executable but not writable
Data memory: writable but not executable
```

The protection is not universal or perfect, but it is an important security mechanism.

---

# 7.16 Controlled Memory Sharing

Processes are normally isolated, but sometimes they need to share data.

The kernel can intentionally map the same physical frame into multiple virtual address spaces.

```text
Process A virtual page 20 ─┐
                           ├──▶ Physical frame 42
Process B virtual page 90 ─┘
```

Both processes can then access the shared physical data.

The mappings may have different permissions.

For example:

```text
Process A: read and write
Process B: read only
```

---

## Why shared memory exists

It can support:

* Fast inter-process communication
* Shared libraries
* Shared file data
* Graphics buffers
* Communication with devices
* Memory-efficient duplication

---

## What problem it introduces

Shared memory weakens isolation for the shared region.

Processes must coordinate access.

If both modify the same data unpredictably, race conditions can occur.

---

# 7.17 Shared Program Code

Two processes may run the same program.

Their private data should remain separate, but unchanged program instructions may be shared.

```text
Process A data ──▶ Private physical frames
Process B data ──▶ Private physical frames

Process A code ─┐
                ├──▶ Shared read-only code frames
Process B code ─┘
```

Because the program instructions are normally not modified, both processes can safely use the same physical copy.

This saves RAM.

---

# 7.18 Shared Libraries

Applications often use common system libraries.

Instead of loading a separate physical copy of unchanged library instructions for every process, the OS may share them.

```text
Browser ───────┐
Editor ────────┼──▶ Shared library code in RAM
Music player ──┘
```

Each process can still have separate private data associated with the library.

---

# 7.19 Virtual Memory Does Not Mean “Fake Memory”

The word “virtual” can be misleading.

Virtual memory is not imaginary.

It is a managed addressing system that maps process-visible addresses to real resources.

Those resources may include:

* Physical RAM
* File-backed data
* Temporarily unavailable pages
* Device memory
* Shared memory
* Reserved address ranges

A virtual address may currently have:

* A valid RAM mapping
* A valid mapping that must be loaded
* No valid mapping
* A protected mapping

---

# 7.20 Virtual Address Space Is Not the Same as RAM Usage

A process may have a large virtual address space without occupying the same amount of physical RAM.

For example:

```text
Process virtual address space: 4 GB
Physical RAM currently used:   300 MB
```

Reasons include:

* Some address ranges are reserved but unused.
* Some pages have not yet been accessed.
* Some pages are shared.
* Some pages correspond to files.
* Some pages may be temporarily stored elsewhere.

Therefore:

> Virtual address-space size does not directly equal physical-memory consumption.

---

# 7.21 Reserved Memory Versus Committed Memory

Operating systems use different terminology, but a useful conceptual distinction is:

## Reserved virtual range

A range of virtual addresses has been set aside for a purpose.

It may not yet have physical RAM behind every page.

---

## Backed or committed memory

The system has accepted responsibility for providing storage for the memory, whether immediately in RAM or through another supported mechanism.

Exact definitions vary between operating systems.

The foundational point is:

```text
Owning virtual addresses
does not always mean
all corresponding physical pages exist now.
```

---

# 7.22 Demand Loading

A process does not necessarily receive all program data in RAM at startup.

Instead, memory can be supplied **on demand**.

## Step-by-step

1. The process receives virtual mappings for program regions.
2. Some pages are not physically loaded yet.
3. The process accesses one of those pages.
4. The CPU raises a page-fault exception.
5. The kernel loads or maps the needed page.
6. The instruction is retried.
7. Execution continues.

```text
Program starts
     │
     ▼
Only immediately needed pages loaded
     │
     ▼
Process accesses another page
     │
     ▼
Page fault
     │
     ▼
Kernel loads page
```

This reduces startup time and avoids loading parts that may never be used.

---

# 7.23 Memory and Files Can Be Connected

A virtual-memory region can be associated with a file.

For example, program instructions may be backed by the program file on storage.

```text
Virtual page
     │
     ▼
Corresponding file data
     │
     ▼
Loaded into physical frame when needed
```

This is one way executable programs and shared libraries can be loaded efficiently.

The process still accesses virtual memory, while the OS handles the connection to the file.

---

# 7.24 The Translation Lookaside Buffer

Virtual-to-physical translation must be fast.

Consulting full page-table structures for every memory access would be expensive.

CPUs therefore commonly use a small, fast cache of recent address translations called the **translation lookaside buffer**, or **TLB**.

```text
Virtual address
      │
      ▼
Check recent translation cache
      │
  ┌───┴───┐
  │       │
Found   Not found
  │       │
Use it  Consult page table
```

---

## TLB hit

The required translation is already cached.

The CPU can proceed quickly.

---

## TLB miss

The translation is not cached.

The processor or operating system must obtain it from the page-table structures.

A TLB miss is not necessarily a page fault.

This distinction is important.

---

## TLB miss versus page fault

| Event          | Meaning                                                            |
| -------------- | ------------------------------------------------------------------ |
| **TLB miss**   | Translation is not in the CPU’s fast translation cache             |
| **Page fault** | Current page-table state cannot satisfy the memory access directly |

A TLB miss may be resolved by finding a perfectly valid mapping in the page table.

No kernel-level page loading may be required.

---

# 7.25 Step-by-Step: Virtual Address Translation

Suppose an application accesses virtual address `V`.

## Step 1: Split the address conceptually

The address contains:

* A virtual page number
* A position inside that page, called an offset

```text
Virtual address
┌──────────────────────┬─────────────┐
│ Virtual page number  │ Page offset │
└──────────────────────┴─────────────┘
```

---

## Step 2: Check the TLB

The CPU looks for a recent mapping for the virtual page.

### If found

The physical frame is immediately known.

### If not found

The page-table mapping must be consulted.

---

## Step 3: Examine the page-table entry

The mapping describes:

* Physical frame number
* Present or absent state
* Read/write permission
* User/kernel permission
* Other status information

---

## Step 4: Validate access

The MMU checks whether the requested operation is allowed.

For example:

* Reading from a readable page
* Writing to a writable page
* Executing from an executable page

---

## Step 5: Form the physical address

```text
Physical address
┌──────────────────────┬─────────────┐
│ Physical frame number│ Same offset │
└──────────────────────┴─────────────┘
```

The position inside the page remains the same.

---

## Step 6: Access RAM

The CPU reads or writes the final physical location.

---

## Translation flow

```text
Virtual address
      │
      ▼
Find virtual page mapping
      │
      ▼
Check permissions
      │
      ▼
Physical frame + page offset
      │
      ▼
Physical RAM
```

---

# 7.26 Context Switching and Virtual Memory

When the scheduler switches between threads in different processes, the active virtual-memory context may also change.

## Before the switch

```text
CPU runs Process A
Active mappings: Process A page tables
```

## After the switch

```text
CPU runs Process B
Active mappings: Process B page tables
```

The kernel changes processor state so that virtual addresses are interpreted using Process B’s mappings.

---

## Why this matters

Suppose both processes use virtual address `5000`.

```text
Process A virtual 5000 → Physical 10000
Process B virtual 5000 → Physical 30000
```

After the switch, the same virtual number leads to a different physical location.

This preserves each process’s private view.

---

# 7.27 Memory Allocation: Foundational Walkthrough

A detailed stack, heap, paging, and page-fault walkthrough comes next. For now, consider what happens when a process asks for more memory.

## Stage 1: Application requests memory

The application usually asks a memory-management library for a region of usable memory.

**Component:** Application and user-space library
**Mode:** Usually user mode initially

---

## Stage 2: Library checks existing process memory

The library may already control an unused portion of the process’s allocated memory.

If enough exists, it can satisfy the request without immediately entering the kernel.

```text
Application requests memory
          │
          ▼
Library has unused region?
      ┌───┴───┐
      │       │
     Yes      No
      │       │
Return it   Request more from OS
```

---

## Stage 3: More virtual memory may be requested

If necessary, the process makes a system call requesting additional address-space capacity.

**Component:** Kernel memory manager
**Mode:** User → kernel

---

## Stage 4: Kernel validates the request

The kernel considers:

* Requested size
* Process limits
* Available address-space region
* System memory policy
* Permissions
* Resource limits

---

## Stage 5: Kernel creates or expands mappings

The kernel reserves virtual addresses and records how they may be used.

Physical frames may be assigned immediately or later.

---

## Stage 6: System call returns

The application receives access to a virtual-memory region.

**Mode:** Kernel → user

---

## Stage 7: Process first accesses the region

If no physical frame has been attached yet, the access causes a page fault.

---

## Stage 8: Kernel provides physical backing

The kernel selects a physical frame, initializes it as required, and updates the page table.

---

## Stage 9: Instruction retries

The original memory access now succeeds.

```text
Request memory
      │
      ▼
Receive virtual address range
      │
      ▼
First access
      │
      ▼
Page fault if page not backed yet
      │
      ▼
Kernel assigns physical frame
      │
      ▼
Access succeeds
```

This demonstrates why successful memory allocation does not always mean all physical RAM was immediately assigned.

---

# 7.28 What Happens When RAM Becomes Scarce?

Physical RAM is limited.

When little free RAM remains, the operating system may try several strategies.

---

## Reclaim cached data

Some RAM contains file data that can be read again from storage later.

The OS may discard clean cached copies to free frames.

```text
Cached file page in RAM
        │
        ▼
Discard from RAM
        │
        ▼
Reload from file later if needed
```

---

## Move inactive memory elsewhere

Some systems can move less-active memory contents to a storage-backed area.

This is commonly associated with:

* Swap
* Paging files
* Page files

Storage is far slower than RAM, so heavy use can severely reduce performance.

---

## Ask applications to release memory

Some operating systems or application environments encourage processes to reduce memory use.

---

## Terminate processes

Under severe pressure, the OS may terminate one or more processes to preserve system operation.

The policy differs across systems.

---

## Reject new allocations

A request for more memory may fail.

Applications must handle this possibility safely.

---

# 7.29 Swapping and Paging: Simplified Preview

These terms are sometimes used loosely.

## Paging

Moving or managing memory in fixed-size pages.

A page may be:

* In RAM
* Backed by a file
* Stored in a swap area
* Not yet allocated
* Discarded because it can be recreated

---

## Swapping

Historically, swapping could mean moving an entire process between RAM and storage.

Modern discussions often use “swap” more generally for storage used to hold memory pages.

Exact terminology varies.

The important mental model is:

```text
Fast but limited RAM
        │
        │ inactive data may move
        ▼
Slower storage-backed memory
```

This can expand flexibility but not physical speed.

---

# 7.30 Virtual Memory Does Not Create Unlimited Memory

Virtual memory can make memory management more flexible, but it does not create infinite resources.

The system is still limited by:

* Physical RAM
* Storage capacity
* Address-space limits
* Kernel data structures
* Performance constraints
* Per-process resource limits

If applications demand too much:

* Allocation may fail.
* The system may become extremely slow.
* Processes may be terminated.
* The system may become unstable.

---

# 7.31 Memory Pressure and Thrashing

If the system repeatedly moves pages between RAM and storage because active processes need more RAM than is available, it may spend most of its time managing pages rather than doing useful work.

This is called **thrashing**.

```text
Process needs page A
      │
Load A, remove B
      │
Process needs page B
      │
Load B, remove A
      │
Repeat continuously
```

Symptoms can include:

* Very slow application response
* Heavy storage activity
* Frequent page faults
* Low useful CPU progress

Virtual memory allows graceful flexibility only up to a point.

---

# 7.32 Copy-on-Write: Conceptual Introduction

Sometimes two processes begin by sharing the same physical memory pages.

If both only read those pages, sharing is safe.

When one tries to modify a shared page, the kernel creates a private copy.

This technique is called **copy-on-write**.

```text
Initially:

Process A ─┐
           ├──▶ Shared read-only page
Process B ─┘
```

When Process A writes:

```text
Process A ──▶ Private copied page
Process B ──▶ Original page
```

---

## Why copy-on-write exists

It avoids copying memory unnecessarily.

The system copies only pages that are actually modified.

This is useful when:

* Creating related processes
* Sharing unchanged data
* Managing memory efficiently
* Supporting snapshots or similar mechanisms

---

## Step by step

1. Two processes map the same physical page.
2. The mappings are marked non-writable or specially protected.
3. One process attempts to write.
4. The CPU raises a memory exception.
5. The kernel recognizes a copy-on-write case.
6. The kernel allocates a new physical frame.
7. The original page’s data is copied.
8. The writing process is mapped to the private copy.
9. The instruction retries.

The exception is intentional and recoverable.

---

# 7.33 Memory-Mapped Devices

Not every virtual address must refer to ordinary RAM.

Some virtual-memory mappings can provide controlled access to device-related memory.

For example, graphics or hardware-control regions may be exposed through special mappings.

```text
Process virtual address
          │
          ▼
Special kernel-approved mapping
          │
          ▼
Device-related memory
```

Access remains controlled by:

* Kernel permissions
* Device drivers
* Hardware rules
* Process privileges

Applications do not gain unrestricted device control merely because memory mapping is used.

---

# 7.34 What Can Go Wrong?

## Invalid memory access

A process reads or writes an address that is not valid.

Possible result:

* Memory exception
* Process termination
* Crash report

---

## Writing to read-only memory

A program attempts to modify:

* Program instructions
* Shared read-only pages
* Protected constants
* Kernel mappings

The MMU blocks the operation.

---

## Executing non-executable memory

A process attempts to run instructions from a data-only region.

The CPU may raise a protection exception.

---

## Memory leak

A process obtains memory but no longer needs it and fails to release or reuse it.

Over time:

* Memory use grows.
* System pressure increases.
* Performance declines.
* Allocation may fail.

The OS can reclaim the process’s memory when the process ends, but not necessarily while it continues running.

---

## Use-after-release concept

A process releases a region and later tries to use it again.

The address may:

* Be unmapped
* Belong to another allocation
* Contain unrelated data

Possible effects include crashes, corrupted data, or security vulnerabilities.

---

## Incorrect kernel mappings

A kernel defect may map:

* Another process’s private memory
* Kernel memory into user access
* A page with incorrect permissions

This can break isolation and cause serious security problems.

---

## Excessive page faults

A process repeatedly accesses pages that are not resident in RAM.

This may create high delay and storage activity.

---

## Out-of-memory condition

The system cannot satisfy additional memory requirements.

Possible outcomes include:

* Allocation failure
* Application termination
* System-wide slowdown
* Forced process termination

---

## Faulty RAM

Physical memory hardware may store incorrect values.

Possible symptoms include:

* Random crashes
* Corrupted files
* Kernel failures
* Unpredictable application behavior

Virtual-memory protection cannot fully solve defective hardware.

---

# 7.35 Common Misconceptions

## Misconception: “Virtual memory is just RAM plus swap”

That is incomplete.

Virtual memory is primarily an addressing and protection abstraction.

Swap is only one possible backing mechanism.

Virtual memory also provides:

* Isolation
* Permission control
* Flexible mappings
* Shared memory
* File-backed memory
* Demand loading

---

## Misconception: “A virtual address is fake”

A virtual address is real within a process’s address space.

It becomes useful through a mapping to a physical or otherwise backed resource.

“Virtual” means indirect and translated, not meaningless.

---

## Misconception: “Every process has its own physical RAM chips”

No.

Processes share the same physical RAM hardware.

Each receives separate virtual mappings.

---

## Misconception: “The same virtual address means the same data”

Not across different processes.

```text
Process A virtual 1000 ≠ Process B virtual 1000
```

They may map to different physical frames.

---

## Misconception: “Virtual memory makes memory unlimited”

No.

It improves management and can use storage as support, but all underlying resources remain finite.

---

## Misconception: “Every memory access requires a system call”

No.

Valid mapped memory accesses happen directly through CPU and MMU hardware.

The kernel is involved when mappings must change or an exception occurs.

---

## Misconception: “Every page fault means RAM is full”

No.

A page fault may occur because:

* A program page has not been loaded yet.
* A valid page must be created.
* Copy-on-write is occurring.
* Permissions are violated.
* The address is invalid.

RAM pressure is only one possible context.

---

## Misconception: “A large virtual-memory size means the process uses the same amount of RAM”

No.

Reserved, shared, file-backed, or untouched pages may consume little or no private physical RAM.

---

## Misconception: “A TLB miss is the same as a page fault”

No.

A TLB miss means the fast translation cache lacks an entry.

The page table may still contain a valid mapping.

---

# 7.36 Real-World Analogy: Library Catalog and Shelves

Imagine a large library.

## Physical memory

The actual shelves holding books.

Each shelf has a real physical location.

---

## Virtual address

A catalog number used by a particular research group.

Different groups may use their own numbering systems.

```text
Group A catalog number 12 → Shelf 8
Group B catalog number 12 → Shelf 41
```

---

## Page table

The catalog that maps each group’s numbering system to physical shelf locations.

---

## MMU

The automated system that checks the catalog and directs each request to the correct shelf.

---

## Permissions

Some catalog entries permit:

* Reading
* Editing
* Restricted staff access only

---

## Page fault

A requested book is valid but currently in remote storage.

The librarian retrieves it and places it on a shelf.

---

## Invalid access

The catalog number does not exist for that research group, or the group lacks permission.

The request is rejected.

---

# 7.37 Connection to Earlier Concepts

## Connection to processes

Each process normally receives its own virtual address space.

This is a major part of process isolation.

---

## Connection to threads

Threads in the same process usually share the same virtual address space.

Each thread has its own stack, but all can normally access shared process memory.

---

## Connection to context switching

Switching between processes may require changing the active address-space mappings.

---

## Connection to user and kernel mode

User-mode memory accesses are limited by mapping permissions.

Kernel mode can manage and change those mappings.

---

## Connection to exceptions

An invalid or unresolved memory access causes a memory-related exception.

The kernel may repair the condition or terminate the process.

---

## Connection to system calls

A process may use system calls to:

* Request more memory
* Create shared memory
* Change memory permissions
* Map files
* Release memory

---

## Connection to security

Virtual-memory permissions help prevent:

* Reading other processes’ data
* Modifying kernel memory
* Executing arbitrary writable data
* Writing to protected code

---

# 7.38 Simplified Model Versus Technical Reality

## Simplified mental model

```text
Each process has private virtual memory.
The MMU translates it into shared physical RAM.
The kernel controls the mappings.
```

This is the key foundational model.

---

## More exact reality

Modern memory systems may include:

* Multiple levels of page tables
* Several page sizes
* Per-core translation caches
* Shared and private mappings
* File-backed pages
* Copy-on-write pages
* Device mappings
* Memory compression
* Non-uniform memory access
* Hypervisor-controlled translation
* Security-specific address spaces
* Kernel mapping differences between operating systems

Not every virtual page is backed by RAM at all times.

Not every mapped address refers to ordinary memory.

The central principle remains:

> A process uses virtual addresses, hardware translates them, and the kernel controls which resources those addresses may access.

---

# 7.39 Core Mental Model

Keep this translation path:

```text
Process uses virtual address
            │
            ▼
MMU checks process mapping
            │
            ├── Is it mapped?
            ├── Is the page present?
            ├── Is access permitted?
            └── Which physical frame?
            │
            ▼
Physical RAM is accessed
```

Keep the isolation model:

```text
Process A virtual address 5000
            │
            ▼
Physical frame 10

Process B virtual address 5000
            │
            ▼
Physical frame 73
```

And remember:

```text
Physical memory = actual RAM hardware
Virtual memory  = process-visible addressing and protection system
Page table      = mapping information
MMU             = hardware that applies the mapping
```

The next section examines how memory is organized inside a process:

* Stack
* Heap
* Pages
* Page faults
* Memory allocation behavior

# Learning Check

Do not look for answers yet.

## Conceptual questions

1. What is the difference between a virtual address and a physical address?
2. What roles do the kernel, page tables, and MMU play in address translation?
3. How can two processes use the same virtual address without accessing the same data?

## Cause-and-effect questions

4. Why can a process have a virtual address space larger than the amount of physical RAM it currently occupies?
5. Why might switching between threads in different processes require more memory-management work than switching between threads in the same process?

## Misconception question

6. A student says, “A page fault proves that the process attempted an illegal memory access.” What is wrong with this statement?

## Scenario-based question

7. Process A and Process B both access virtual address `8000`. Process A’s access succeeds, while Process B’s access causes an exception. Explain how their separate page tables and permissions can produce this result.

# 8. Stack, Heap, Paging, and Page Faults

A process does not use its virtual address space as one undivided block.

Different regions serve different purposes:

```text
Simplified process virtual address space

Higher addresses
┌──────────────────────────────┐
│ Kernel-controlled region     │
├──────────────────────────────┤
│ Thread stacks                │
│                              │
│         unused space         │
│                              │
│ Heap                         │
├──────────────────────────────┤
│ Program data                 │
├──────────────────────────────┤
│ Program instructions         │
└──────────────────────────────┘
Lower addresses
```

This layout is only a mental model. Real layouts vary by operating system, processor architecture, security settings, and program.

The two most important dynamic memory regions are:

| Region    | Main purpose                                           |
| --------- | ------------------------------------------------------ |
| **Stack** | Tracks active operations for one thread                |
| **Heap**  | Holds dynamically managed data shared within a process |

Both are made from virtual-memory pages. They are not separate kinds of physical RAM.

---

# 8.1 Four Related but Different Concepts

Before examining each concept, separate them clearly.

## Stack

A region used mainly for temporary execution state associated with a thread.

---

## Heap

A region used for dynamically allocated data whose lifetime is controlled by the application.

---

## Paging

The mechanism that divides virtual memory and physical RAM into fixed-size blocks.

---

## Page fault

An exception raised when a memory access cannot proceed using the current page mapping.

```text
Process memory organization
        │
        ├── Stack: thread execution state
        ├── Heap: dynamic application data
        │
        └── Both consist of virtual pages
                         │
                         ▼
                  Page faults may occur
                  when pages need attention
```

---

# 8.2 The Stack

## What it is

A **stack** is a memory region used by a thread to track its active sequence of operations.

It commonly stores information such as:

* Which operation called which other operation
* Where execution should return afterward
* Temporary local values
* Saved execution information
* Parameters or intermediate results

Every thread normally has its own stack.

```text
Process
├── Shared program instructions
├── Shared heap
│
├── Thread A
│   └── Stack A
│
├── Thread B
│   └── Stack B
│
└── Thread C
    └── Stack C
```

---

## Why the stack exists

Programs often perform nested operations.

For example:

1. The application handles a button click.
2. That operation asks to load a document.
3. Loading the document asks to read metadata.
4. Reading metadata asks to decode text.
5. Each operation must eventually return to the operation that called it.

The system needs to remember this nesting.

```text
Handle button click
        │
        ▼
Load document
        │
        ▼
Read metadata
        │
        ▼
Decode text
```

When the deepest operation finishes, execution returns in reverse order:

```text
Decode text finishes
        │
        ▼
Return to Read metadata
        │
        ▼
Return to Load document
        │
        ▼
Return to Handle button click
```

The stack records enough information to make this return sequence possible.

---

## The problem the stack solves

Without a stack, every nested operation would need a separate manual system for remembering:

* Who called it
* Where to return
* Which temporary values belong to which active operation
* Which work is still unfinished

The stack provides an orderly last-started, first-finished structure.

---

# 8.3 Stack Mental Model: Trays in a Cafeteria

Imagine a stack of trays.

```text
Top
┌──────────────┐
│ Tray C       │ ← most recently added
├──────────────┤
│ Tray B       │
├──────────────┤
│ Tray A       │ ← added first
└──────────────┘
Bottom
```

You normally remove the top tray first.

This pattern is called **last in, first out**, abbreviated **LIFO**.

For nested program operations:

```text
Operation A begins
Operation B begins
Operation C begins

Operation C finishes first
Operation B finishes next
Operation A finishes last
```

---

# 8.4 Stack Frames

Each active operation usually has a corresponding section of the stack, often called a **stack frame** or **activation record**.

A simplified stack might look like:

```text
Top of stack
┌──────────────────────────┐
│ Decode text frame        │
│ temporary values         │
│ return information       │
├──────────────────────────┤
│ Read metadata frame      │
│ temporary values         │
│ return information       │
├──────────────────────────┤
│ Load document frame      │
│ temporary values         │
│ return information       │
├──────────────────────────┤
│ Handle click frame       │
│ temporary values         │
│ return information       │
└──────────────────────────┘
```

When `Decode text` finishes, its frame is removed conceptually.

```text
Before return                  After return

┌────────────────────┐         ┌────────────────────┐
│ Decode text        │         │ Read metadata      │
├────────────────────┤         ├────────────────────┤
│ Read metadata      │         │ Load document      │
├────────────────────┤         ├────────────────────┤
│ Load document      │         │ Handle click       │
├────────────────────┤         └────────────────────┘
│ Handle click       │
└────────────────────┘
```

---

# 8.5 Step-by-Step: How a Stack Is Used

Suppose an application performs three nested operations.

## Step 1: Thread begins Operation A

The thread places information for Operation A on its stack.

```text
Stack:
[A]
```

---

## Step 2: Operation A starts Operation B

A new frame is added.

```text
Stack:
[B]
[A]
```

---

## Step 3: Operation B starts Operation C

Another frame is added.

```text
Stack:
[C]
[B]
[A]
```

---

## Step 4: Operation C finishes

Its frame is removed.

```text
Stack:
[B]
[A]
```

Execution returns to Operation B.

---

## Step 5: Operation B finishes

```text
Stack:
[A]
```

Execution returns to Operation A.

---

## Step 6: Operation A finishes

```text
Stack:
[empty or previous work]
```

The stack naturally mirrors the nested execution path.

---

# 8.6 Each Thread Needs Its Own Stack

Threads may execute different operations at the same time.

Consider two threads:

```text
Thread A:
Handle user input
  └── Update document

Thread B:
Load file
  └── Decode contents
```

Their active operations must remain separate.

```text
Thread A stack             Thread B stack

┌──────────────────┐       ┌──────────────────┐
│ Update document  │       │ Decode contents  │
├──────────────────┤       ├──────────────────┤
│ Handle input     │       │ Load file        │
└──────────────────┘       └──────────────────┘
```

If both threads used one ordinary shared stack, their execution histories could interfere.

---

# 8.7 Stack Size Is Limited

A thread’s stack occupies a limited virtual-memory region.

It cannot grow indefinitely.

Possible reasons a stack becomes too large include:

* Extremely deep nesting
* An operation repeatedly calling itself without stopping
* Very large temporary values stored on the stack
* Too many active operations

When the stack exceeds its allowed region, a **stack overflow** occurs.

---

## Stack overflow mental model

Imagine placing trays onto a shelf with limited height.

```text
Available stack space
┌─────────────────┐
│ Frame 5         │
├─────────────────┤
│ Frame 4         │
├─────────────────┤
│ Frame 3         │
├─────────────────┤
│ Frame 2         │
├─────────────────┤
│ Frame 1         │
└─────────────────┘
```

Adding another frame exceeds the protected region.

The CPU may detect an invalid memory access, causing an exception.

The operating system will commonly terminate the process unless the runtime can safely handle the condition.

---

# 8.8 Stack Growth

A thread’s stack may begin with only some pages physically available.

As the thread uses more of the stack, additional pages may be created on demand.

```text
Reserved stack region

┌──────────────────────┐
│ Not currently backed │
├──────────────────────┤
│ Not currently backed │
├──────────────────────┤
│ Active stack page    │
├──────────────────────┤
│ Active stack page    │
└──────────────────────┘
```

When valid stack growth reaches an unmapped but permitted page:

1. The thread accesses the new stack area.
2. The CPU raises a page fault.
3. The kernel recognizes legitimate stack growth.
4. The kernel maps a physical frame.
5. The instruction retries.
6. Execution continues.

This page fault is normal and recoverable.

---

## Guard pages

Operating systems may place an inaccessible **guard page** near a stack boundary.

```text
┌────────────────────────┐
│ Valid stack pages      │
├────────────────────────┤
│ Guard page: no access  │
├────────────────────────┤
│ Other memory           │
└────────────────────────┘
```

If the stack crosses into the guard page, the CPU raises an exception.

This helps detect overflow before the stack silently corrupts unrelated memory.

---

# 8.9 The Heap

## What it is

The **heap** is a region used for dynamically managed application data.

“Dynamic” means that the program requests memory while it is running, according to current needs.

Heap data may include:

* A document loaded by an editor
* Browser page structures
* Image data
* A list whose size changes
* Cached application information
* Objects that outlive the operation that created them
* Data shared among several threads

```text
Process
┌─────────────────────────────┐
│ Heap                        │
│                             │
│ Document data               │
│ Image data                  │
│ Application objects         │
│ Dynamic collections         │
└─────────────────────────────┘
```

---

## Why the heap exists

The stack works well for temporary, nested execution state.

However, many data items do not follow a simple last-in, first-out lifetime.

For example:

1. A document is opened.
2. Several operations use it.
3. Other documents are opened and closed.
4. The first document remains active.
5. It is released only when the user closes it.

This lifetime does not fit ordinary stack removal order.

The heap allows memory regions to be created and released in a more flexible order.

---

## The problem the heap solves

At program creation time, the application often does not know:

* How many files the user will open
* How large an image will be
* How much network data will arrive
* How many items a user will create
* How long each data object will remain needed

The heap allows memory use to adjust while the process runs.

---

# 8.10 Heap Mental Model: A Storage Facility

Imagine a storage facility with many spaces of different sizes.

A customer can request:

* A small locker
* A medium room
* A large storage area

The spaces do not need to be released in the order they were assigned.

```text
Heap storage facility

┌──────────┬──────────────┬────────┐
│ Data A   │ Free region  │ Data B │
├──────────┴──────┬───────┴────────┤
│ Data C          │ Free region    │
└─────────────────┴────────────────┘
```

A manager tracks:

* Which regions are free
* Which regions are occupied
* How large each allocation is
* Whether adjacent free regions can be combined

That manager is typically a user-space **memory allocator**, assisted by the operating system.

---

# 8.11 The Memory Allocator

Applications often do not request kernel memory for every small object.

Instead, a **memory allocator** manages a larger heap region within the process.

```text
Application requests 200 bytes
            │
            ▼
User-space allocator
            │
      ┌─────┴─────┐
      │           │
Free block      No suitable block
exists          exists
      │           │
Return block   Request more virtual
              memory from kernel
```

The allocator keeps records of:

* Free regions
* Used regions
* Region sizes
* Alignment requirements
* Reusable blocks

---

## Why the allocator usually runs in user space

Entering the kernel for every small memory request would add unnecessary overhead.

Instead:

1. The allocator obtains larger memory regions from the OS.
2. It divides them into smaller pieces.
3. It gives those pieces to application components.
4. It reuses released pieces when possible.

The kernel manages pages and process mappings.

The allocator manages smaller application-level regions inside those pages.

---

# 8.12 Step-by-Step: Allocating Heap Memory

Suppose an application needs space for a large image.

## Stage 1: The application requests memory

The application asks its memory allocator for a region of a particular size.

**Component:** Application
**Mode:** User mode

---

## Stage 2: The allocator searches existing free heap space

The allocator checks whether a suitable unused block already exists.

```text
Heap:
[Used A][Free block][Used B]
```

### Suitable block found

The allocator marks part or all of it as used and returns a virtual address.

No system call may be needed.

### No suitable block found

The allocator must obtain more virtual memory from the operating system.

---

## Stage 3: Allocator requests more address space

The allocator makes a system call requesting an additional memory region.

**Component:** Kernel memory manager
**Mode:** User → kernel

---

## Stage 4: Kernel validates the request

The kernel checks:

* Requested size
* Process resource limits
* Available virtual address range
* Overall memory policy
* Permission requirements

---

## Stage 5: Kernel creates virtual mappings

The kernel records that a virtual-address range belongs to the process.

The pages may be marked as valid but not yet backed by physical frames.

```text
New heap virtual range

┌──────────────────────┐
│ Virtual page A       │ → no physical frame yet
├──────────────────────┤
│ Virtual page B       │ → no physical frame yet
├──────────────────────┤
│ Virtual page C       │ → no physical frame yet
└──────────────────────┘
```

---

## Stage 6: Kernel returns to the allocator

The allocator receives the new virtual range.

**Mode:** Kernel → user

---

## Stage 7: Allocator returns part of the range

The application receives a virtual address for the requested image data.

At this moment, not every page may occupy physical RAM.

---

## Stage 8: Application writes to the memory

The application first accesses one of the new pages.

---

## Stage 9: A page fault may occur

If that page has no physical frame, the MMU cannot complete the write.

The CPU raises a page-fault exception.

**Mode:** User → kernel

---

## Stage 10: Kernel supplies a physical frame

The kernel:

1. Confirms the access is valid.
2. Selects a physical frame.
3. Initializes it, commonly with zeros for security.
4. Updates the page table.
5. Marks the page writable.
6. Returns to the process.

---

## Stage 11: Instruction retries

The original write now succeeds.

---

## Stage 12: Additional pages are supplied as needed

If the application uses only half the allocated region, only some pages may need physical backing.

```text
Allocated virtual region

Page A → physical frame 12
Page B → physical frame 44
Page C → never accessed, no frame yet
Page D → never accessed, no frame yet
```

This is called **lazy allocation** or **demand allocation**.

---

# 8.13 Allocation Does Not Always Mean Immediate RAM Use

Suppose an application successfully requests 500 MB of virtual memory.

That does not always mean the OS immediately assigns 500 MB of physical RAM.

Possible sequence:

```text
Request 500 MB
      │
      ▼
Reserve virtual range
      │
      ▼
Application touches 20 MB
      │
      ▼
Approximately 20 MB needs physical backing
```

The exact accounting and commitment policies differ between operating systems.

The key principle is:

> Receiving virtual addresses and receiving physical frames can happen at different times.

---

# 8.14 Releasing Heap Memory

When the application no longer needs a heap allocation, it informs its allocator.

The allocator may:

* Mark the region free
* Reuse it for a later allocation
* Combine it with neighboring free regions
* Return some pages to the operating system
* Keep the pages for future requests

```text
Before release:

[Used A][Used B][Used C]

After releasing B:

[Used A][Free B][Used C]
```

---

## Releasing memory does not always reduce process RAM immediately

The allocator may retain released memory for reuse.

```text
Application releases block
          │
          ▼
Allocator marks block free
          │
          ▼
Block remains inside process heap
          │
          ▼
Future allocation reuses it
```

Therefore, a process’s reported memory use may not fall immediately after an object is released.

This does not automatically indicate a memory leak.

---

# 8.15 Stack Versus Heap

| Property         | Stack                                        | Heap                                     |
| ---------------- | -------------------------------------------- | ---------------------------------------- |
| Primary purpose  | Active thread execution state                | Dynamic application data                 |
| Ownership        | Normally one stack per thread                | Shared within the process                |
| Lifetime pattern | Usually nested, last-in-first-out            | Flexible                                 |
| Management       | Mostly automatic as operations begin and end | Managed by allocator/runtime/application |
| Typical contents | Return information, local temporary state    | Documents, images, dynamic structures    |
| Size             | Usually limited per thread                   | Can grow across a larger process region  |
| Common failure   | Stack overflow                               | Leak, fragmentation, invalid reuse       |
| Backed by pages  | Yes                                          | Yes                                      |

---

## Important correction

A common explanation says:

> “Stack memory is physical memory in one place, and heap memory is physical memory somewhere else.”

That is misleading.

Both are virtual-address-space regions.

Their pages may map to physical frames anywhere in RAM.

```text
Stack virtual page  ──▶ Physical frame 80
Heap virtual page   ──▶ Physical frame 3
Another stack page  ──▶ Physical frame 41
```

Their differences concern purpose and management, not a special physical material.

---

# 8.16 Is the Stack Always Faster Than the Heap?

A common statement is:

> “The stack is fast; the heap is slow.”

This is too simplistic.

## Why stack management is often cheap

Adding or removing a stack frame commonly involves adjusting a current stack position.

The order is predictable.

```text
Add frame:
stack position moves

Remove frame:
stack position moves back
```

---

## Why heap management may require more work

A heap allocator may need to:

* Search for a suitable free block
* Split a block
* Combine adjacent blocks
* Maintain bookkeeping
* Synchronize multiple threads
* Request more pages

---

## But memory access speed is not simply stack versus heap

Once data is in mapped RAM, access performance depends heavily on:

* CPU caches
* Access pattern
* Data size
* Page locality
* Contention
* Hardware behavior

Heap data that is already cached can be accessed quickly.

Stack data that causes cache misses or page faults can be slow.

A more accurate statement is:

> Stack allocation is often simpler and cheaper to manage, but actual data-access speed depends on where the data is in the memory hierarchy and how it is used.

---

# 8.17 Heap Fragmentation

As heap blocks are allocated and released in different orders, free space can become divided.

```text
Heap:

[Used][Free 10][Used][Free 5][Used][Free 20]
```

The process may have 35 units of free space in total, but no single free block of 30 contiguous units.

This is **external fragmentation**.

---

## Internal fragmentation

An allocator may provide a block larger than requested.

For example:

```text
Requested: 18 units
Allocated: 24 units
Unused inside block: 6 units
```

The unused portion inside the allocated block is **internal fragmentation**.

---

## Why fragmentation matters

It can cause:

* Higher memory use
* Difficulty satisfying large requests
* More allocator bookkeeping
* Reduced cache efficiency
* More requests to the kernel

Allocators use different strategies to balance speed, memory efficiency, and complexity.

---

# 8.18 Memory Leaks

A **memory leak** occurs when a process retains allocated memory that it no longer needs and can no longer effectively reuse or release.

```text
Time ─────────────────────────────────────▶

Useful memory:  ███ ███ ███
Leaked memory:      ██  ███ ████ █████
```

As leaks accumulate:

* Process memory use grows.
* Physical-memory pressure increases.
* More paging may occur.
* Allocations may fail.
* The process or system may slow down.

---

## The OS cannot always identify a leak

The kernel knows that memory belongs to a process.

It usually does not understand the application’s logical intention.

From the kernel’s perspective:

```text
“This process still owns this mapped memory.”
```

The kernel cannot always determine whether the application still needs it.

When the process terminates, the OS can reclaim the entire address space.

---

# 8.19 Invalid Use of Released Memory

Suppose an application releases a heap block but later tries to use its old address.

```text
1. Block belongs to object A
2. Block is released
3. Allocator reuses it for object B
4. Old code still writes through A’s former address
```

Possible results include:

* Corruption of object B
* Unpredictable behavior
* Security vulnerabilities
* Process crash

The operating system may not detect every such error immediately because the page may still be valid within the process.

Process isolation protects one process from another, but it does not automatically protect every object inside one process from every other object.

---

# 8.20 Paging Revisited

## What paging is

**Paging** divides virtual memory into fixed-size **pages** and physical memory into similarly sized **frames**.

```text
Virtual address space           Physical RAM

┌───────────────┐               ┌───────────────┐
│ Page 0        │──────────────▶│ Frame 7       │
├───────────────┤               ├───────────────┤
│ Page 1        │──────────────▶│ Frame 2       │
├───────────────┤               ├───────────────┤
│ Page 2        │──────────────▶│ Frame 19      │
└───────────────┘               └───────────────┘
```

A virtual page and its physical frame have corresponding sizes.

---

## Why fixed-size pages exist

Fixed-size units simplify:

* Address translation
* Physical-memory allocation
* Protection
* Sharing
* Moving data between RAM and storage
* Loading only needed program portions

Without paging, the OS might need to manage many irregularly sized physical regions.

---

# 8.21 Page and Offset

A virtual address can be conceptually divided into:

```text
┌──────────────────────┬────────────────┐
│ Virtual page number  │ Offset in page │
└──────────────────────┴────────────────┘
```

The page number identifies which virtual page is involved.

The offset identifies the exact byte within that page.

During translation:

```text
Virtual page 12 → Physical frame 60
Offset 200 remains offset 200
```

Therefore:

```text
Virtual address:
Page 12, offset 200

Physical address:
Frame 60, offset 200
```

---

# 8.22 Page-Table Entries

For each mapped virtual page, the page table contains a **page-table entry**.

Conceptually, it may record:

* Physical frame number
* Whether the page is present
* Whether it is writable
* Whether it is executable
* Whether user-mode access is allowed
* Whether the page was recently accessed
* Whether it has been modified
* Other architecture-specific information

```text
Virtual page 8
└── Page-table entry
    ├── Frame: 42
    ├── Present: yes
    ├── Readable: yes
    ├── Writable: no
    ├── Executable: yes
    └── User-accessible: yes
```

---

## Accessed status

Hardware may record that a page has recently been used.

The OS can use this information when deciding which pages are less active.

---

## Dirty status

A page is often called **dirty** when it has been modified since it was loaded or last saved.

```text
Clean page:
RAM copy matches its backing data

Dirty page:
RAM contains changes not yet written back
```

This matters during page eviction.

---

# 8.23 What Is a Page Fault?

A **page fault** is a synchronous exception raised when the current memory access cannot proceed using the present mapping state.

Possible reasons include:

* The page is valid but not yet in RAM.
* A new zero-filled page must be created.
* A copy-on-write page must be copied.
* A stack page must grow.
* The requested permission is not allowed.
* The address is invalid.

The term “fault” does not automatically mean the program has failed.

---

# 8.24 Major Categories of Page Faults

## 1. Demand-zero fault

The process accesses newly allocated memory that should begin filled with zeros.

```text
New virtual page
      │ first access
      ▼
Page fault
      │
      ▼
Kernel provides zeroed physical frame
```

Why zero it?

* Prevent old data from another process being exposed
* Give predictable initial contents
* Enforce isolation

---

## 2. File-backed demand fault

The page corresponds to data in a file, such as:

* Program instructions
* Shared-library instructions
* Memory-mapped file contents

```text
Process accesses program page
          │
          ▼
Page not in RAM
          │
          ▼
Kernel reads corresponding file data
          │
          ▼
Page mapped into process
```

---

## 3. Copy-on-write fault

A process attempts to modify a page currently shared as read-only.

```text
Shared page
    │ write attempt
    ▼
Page fault
    │
    ▼
Kernel creates private copy
```

---

## 4. Valid stack-growth fault

The thread reaches a permitted extension of its stack.

The kernel adds another stack page.

---

## 5. Protection fault

The mapping exists, but the attempted operation violates its permissions.

Examples:

* Writing to a read-only page
* Executing a non-executable page
* User-mode access to a kernel-only page

---

## 6. Invalid-address fault

No valid mapping exists and the OS has no reason to create one.

This commonly leads to process termination.

---

# 8.25 Step-by-Step: What Happens During a Recoverable Page Fault

Suppose a process accesses a valid heap page that has not yet received a physical frame.

## Step 1: Thread executes a memory instruction

**Mode:** User mode

The instruction attempts to read or write a virtual address.

---

## Step 2: MMU checks the mapping

The page-table information says the page is not currently present.

---

## Step 3: CPU raises a page-fault exception

The attempted memory operation does not complete.

**Mode:** User → kernel

The CPU preserves enough state for the instruction to be retried.

---

## Step 4: Kernel identifies the faulting process and address

The kernel determines:

* Which thread caused the fault
* Which virtual address was accessed
* Whether it was a read, write, or execute attempt
* Whether the access came from user or kernel mode

---

## Step 5: Kernel consults the process memory map

The kernel asks:

* Does this address belong to a valid region?
* What permissions does the region have?
* Should it be backed by zeros, a file, or shared memory?
* Is this a copy-on-write case?
* Is stack growth allowed here?

---

## Step 6: Kernel obtains a physical frame

It may:

* Use a free frame
* Reclaim a cache page
* Evict another page
* Wait for storage work
* Fail if resources are unavailable

---

## Step 7: Kernel prepares page contents

Depending on the page type, it may:

* Fill the frame with zeros
* Read data from a file
* Copy another physical page
* Restore data from swap

---

## Step 8: Kernel updates the page table

The mapping now points to the prepared physical frame with appropriate permissions.

---

## Step 9: TLB state is updated or invalidated

The CPU must use the new translation rather than an outdated cached entry.

---

## Step 10: Kernel returns to the thread

**Mode:** Kernel → user

---

## Step 11: The original instruction retries

The application usually does not need to issue the memory operation again manually.

The CPU resumes at the faulting instruction, which now succeeds.

```text
Memory instruction
       │
       ▼
Page fault
       │
       ▼
Kernel validates address
       │
       ▼
Prepare physical frame
       │
       ▼
Update page table
       │
       ▼
Retry same instruction
       │
       ▼
Access succeeds
```

---

# 8.26 Page Faults Can Be Fast or Slow

Not all page faults have the same cost.

## Minor page fault

The required data can be supplied without reading it from slow storage.

Examples:

* Demand-zero page
* Copy-on-write page already present in RAM
* Shared page already loaded by another process

These still require kernel work but may be relatively quick.

---

## Major page fault

The kernel must retrieve data from storage.

Examples:

* Program page not currently in RAM
* Memory contents stored in swap
* File-backed page absent from memory cache

```text
Minor fault:
Kernel bookkeeping + RAM work

Major fault:
Kernel bookkeeping + storage I/O
```

Storage access can make a major page fault far slower.

The exact labels and accounting differ among operating systems.

---

# 8.27 Page Fault Versus Invalid Memory Access

These are not synonyms.

## Recoverable page fault

```text
Address belongs to valid region
Page is currently unavailable
Kernel supplies page
Process continues
```

---

## Fatal invalid access

```text
Address belongs to no valid region
or requested permission is forbidden
Kernel cannot safely satisfy access
Process receives fatal error
```

Both may begin with a page-fault exception at the hardware level.

The kernel decides whether the condition is valid and recoverable.

---

# 8.28 Page Replacement

When RAM is full or under pressure, the OS may need a physical frame for a newly needed page.

It can choose another page to remove from RAM.

This process is called **page replacement** or **page eviction**.

```text
Need a free frame
      │
      ▼
Choose existing page
      │
      ▼
Can it be discarded safely?
      │
      ├── Yes → reuse frame
      └── No  → save page first
```

---

## Clean file-backed page

Suppose a page contains unchanged data from a file.

The OS may discard the RAM copy because it can read the file again later.

```text
Clean file-backed page
          │
          ▼
Remove from RAM
          │
          ▼
Reload from file if needed
```

---

## Dirty file-backed page

If the process modified the page, the changes may need to be written back before the frame is reused.

---

## Anonymous page

An **anonymous page** is memory not directly backed by an ordinary file, such as much heap and stack memory.

If its contents must be preserved, the OS may:

* Keep it in RAM
* Compress it
* Store it in a swap or page file
* Use another operating-system-specific mechanism

---

# 8.29 Step-by-Step: Evicting a Page

Suppose the kernel needs a physical frame.

## Step 1: Kernel selects a candidate page

The OS may consider factors such as:

* Recent use
* Whether the page is dirty
* Whether it is shared
* Whether it can be recreated easily
* Process activity
* System policy

---

## Step 2: Kernel determines whether data must be saved

### Clean and reproducible

The page can be discarded.

### Dirty or anonymous

Its contents may need to be written to storage or preserved elsewhere.

---

## Step 3: Page-table entry is changed

The process mapping records that the page is no longer present in RAM.

---

## Step 4: TLB entries are invalidated as needed

The CPU must not continue using an old translation.

---

## Step 5: Physical frame is reused

The frame can now hold another page.

---

## Step 6: Original process later accesses the evicted page

A page fault occurs.

The kernel reloads or reconstructs the page.

```text
Page in RAM
    │ eviction
    ▼
Page absent
    │ later access
    ▼
Page fault
    │
    ▼
Reload page
```

---

# 8.30 Page-Replacement Goals

The OS would prefer to evict pages that will not be needed soon.

However, it cannot know the future exactly.

It uses approximations based on information such as:

* Recent access
* Frequency of access
* Page type
* Dirty state
* Process priority
* Memory pressure

An ideal but impossible rule would be:

> Remove the page whose next use is farthest in the future.

Real systems estimate instead.

---

# 8.31 Locality

Programs often access memory with **locality**.

## Temporal locality

Recently used data is likely to be used again soon.

Example:

* Repeatedly updating the same document structure

---

## Spatial locality

Data near recently accessed data is likely to be used soon.

Example:

* Reading consecutive parts of an image

```text
Recently accessed:
Page 20

Likely next:
Page 20 again or nearby pages 21–22
```

Caches, page systems, and replacement policies benefit from locality.

Programs with poor locality may cause more cache misses and page faults.

---

# 8.32 Working Set

A process’s **working set** is the collection of memory pages it actively needs during a period.

```text
Process virtual pages:
1 2 3 4 5 6 7 8 9 10

Currently active working set:
3 4 5 8
```

If the process’s working set fits in RAM, it can run efficiently.

If active working sets across processes exceed available RAM, the system may constantly move pages in and out.

This can lead to thrashing.

---

# 8.33 Thrashing Revisited

Suppose Process A needs pages 1–5, but only three frames are available.

```text
Load page 1
Load page 2
Load page 3
Need page 4 → evict page 1
Need page 5 → evict page 2
Need page 1 → evict page 3
Need page 2 → evict page 4
```

The system repeatedly transfers pages instead of doing useful work.

```text
Time use during thrashing

Useful computation:  ██
Page movement:        █████████████████
```

Possible symptoms:

* Applications freeze or respond slowly
* Storage activity remains high
* CPU waits frequently
* Context switches and faults increase
* Overall throughput collapses

---

# 8.34 Detailed Walkthrough: Memory Allocation from Request to First Use

Suppose an image editor opens a very large image and needs memory for decoded pixels.

## Stage 1: Application calculates required size

The image editor determines how much memory it needs.

**Component:** Application
**Mode:** User mode

---

## Stage 2: Application asks its allocator

The allocator searches existing free heap regions.

**Component:** User-space memory allocator

---

## Stage 3: Existing heap is insufficient

The allocator makes a system call requesting additional virtual memory.

**Component:** System-call interface
**Mode:** User → kernel

---

## Stage 4: Kernel checks policy and limits

The kernel examines:

* Process memory limits
* Available address-space range
* System commitment policy
* Requested permissions

**Component:** Kernel memory manager

---

## Stage 5: Kernel reserves or maps virtual pages

The process now has a larger valid heap region.

Physical frames may not yet exist for every page.

**Component:** Page-table and virtual-memory manager

---

## Stage 6: Kernel returns success

The allocator records the new region and gives the application a suitable address.

**Mode:** Kernel → user

---

## Stage 7: Image editor begins filling pixel data

It writes to the first page.

---

## Stage 8: Demand-zero page fault occurs

The page is valid but not physically backed.

**Component:** MMU and CPU
**Mode:** User → kernel

---

## Stage 9: Kernel assigns a zeroed frame

The kernel updates the process’s page table.

**Component:** Kernel memory manager

---

## Stage 10: Write instruction retries

The pixel data is stored.

**Mode:** Kernel → user

---

## Stage 11: More pages fault in as decoding continues

Each newly touched page may receive physical backing.

---

## Stage 12: Memory pressure may increase

The kernel may reclaim file cache or evict less-used pages from other processes.

---

## Stage 13: Image is closed

The application releases its allocation to the allocator.

---

## Stage 14: Allocator may reuse or return memory

Some pages may remain in the process for later allocations.

Others may be returned to the kernel.

---

## Stage 15: Kernel reclaims returned pages

Physical frames can be used elsewhere.

```text
Application requests image memory
             │
             ▼
Allocator searches heap
             │
             ▼
Kernel expands virtual mappings
             │
             ▼
Application receives addresses
             │
             ▼
First access causes page fault
             │
             ▼
Kernel assigns physical frames
             │
             ▼
Application uses memory
             │
             ▼
Memory later released and reused
```

---

# 8.35 Detailed Walkthrough: Stack Overflow Crash

Suppose an application repeatedly starts the same operation without reaching a stopping condition.

## Stage 1: First operation begins

A stack frame is created.

---

## Stage 2: Operation starts another copy of itself

Another frame is added.

---

## Stage 3: This repeats many times

```text
Stack:
[Call 5000]
[Call 4999]
[Call 4998]
...
[Call 2]
[Call 1]
```

---

## Stage 4: Stack reaches its reserved limit

The next frame attempts to enter a guard page or unmapped region.

---

## Stage 5: CPU raises a memory exception

The MMU cannot permit the access.

**Mode:** User → kernel

---

## Stage 6: Kernel examines the address

The address is beyond valid stack growth.

The kernel cannot safely add another page.

---

## Stage 7: Process receives a fatal memory error

A language runtime may attempt to report a stack overflow, but recovery is difficult because the stack itself is exhausted.

---

## Stage 8: Process terminates

The kernel stops its threads and reclaims its address space.

Other processes usually continue.

```text
Repeated nested operations
          │
          ▼
Stack reaches limit
          │
          ▼
Guard page accessed
          │
          ▼
Page-fault exception
          │
          ▼
Kernel rejects access
          │
          ▼
Process terminates
```

---

# 8.36 Detailed Walkthrough: Valid File-Backed Page Fault

Suppose a program starts, but one section of its instructions has not yet been loaded into RAM.

## Step 1: Thread reaches an instruction on an absent page

The virtual address is valid and executable.

---

## Step 2: MMU finds that the page is not present

A page-fault exception occurs.

---

## Step 3: Kernel checks the process mapping

The page corresponds to part of the program file.

---

## Step 4: Kernel requests the file data

The file system and storage driver locate and read the relevant page.

---

## Step 5: Thread waits if storage is required

The scheduler may run another thread.

---

## Step 6: Storage interrupt reports completion

The kernel finishes preparing the page.

---

## Step 7: Page table is updated

The virtual page now maps to a physical frame with read-and-execute permission.

---

## Step 8: Original thread becomes ready

The scheduler eventually restores it.

---

## Step 9: Instruction retries

The program continues as though the page had always been available.

```text
Execute instruction
       │
       ▼
Instruction page absent
       │
       ▼
Page fault
       │
       ▼
Read program page from file
       │
       ▼
Map page as executable
       │
       ▼
Retry instruction
```

---

# 8.37 What Happens When Physical Memory Is Allocated

A physical frame may come from:

* A free-frame list
* A reclaimed clean cache page
* An evicted application page
* A page made available after another process exits
* A page returned by a memory allocator
* An operating-system reserve

The kernel must also ensure that old contents do not leak between processes.

---

## Why newly assigned pages are often cleared

Suppose a physical frame previously contained another process’s password data.

If the frame were handed directly to a new process without clearing:

```text
Old process releases frame
          │
          ▼
New process receives same frame
          │
          ▼
New process reads leftover data
```

This would violate isolation.

The kernel therefore commonly zeroes or otherwise sanitizes newly exposed memory.

---

# 8.38 Page Size

Page size is determined by hardware and operating-system configuration.

A common page size on many systems is several kilobytes, but other sizes also exist.

Larger pages may reduce:

* Page-table size
* TLB pressure
* Translation overhead

But they may increase:

* Wasted unused memory
* Cost of moving a page
* Internal fragmentation

```text
Small pages:
More mapping entries, finer control

Large pages:
Fewer entries, coarser allocation
```

Many systems support more than one page size.

---

# 8.39 Page Tables Consume Memory

Page tables are themselves data structures stored in memory.

A process with a large and heavily mapped address space may require significant page-table storage.

To reduce this cost, modern systems commonly use multi-level page tables.

---

## Simplified multi-level concept

Instead of one enormous flat table:

```text
Virtual address
      │
      ▼
Top-level directory
      │
      ▼
Lower-level table
      │
      ▼
Page-table entry
```

Only the portions needed for mapped address ranges must exist.

The exact structure depends on the CPU architecture.

---

# 8.40 What Can Go Wrong?

## Stack overflow

The thread’s stack exceeds its allowed region.

Possible result:

* Memory exception
* Process termination

---

## Memory leak

The process keeps memory that no longer serves useful work.

Possible result:

* Increasing memory use
* Paging pressure
* Allocation failure

---

## Heap fragmentation

Free space exists but is split into inconvenient pieces.

Possible result:

* Higher memory use
* More allocator work
* Inability to satisfy a large request efficiently

---

## Invalid access after release

The application uses memory after returning it to the allocator.

Possible result:

* Corruption
* Delayed crash
* Security vulnerability

---

## Releasing the same region twice

The allocator’s internal records may become inconsistent.

Possible result:

* Heap corruption
* Reuse conflicts
* Process crash

---

## Writing beyond an allocated block

A component modifies neighboring heap data.

Possible result:

* Corruption of another object
* Allocator metadata damage
* Security vulnerability

The OS may not detect this immediately if the write remains inside a valid mapped page.

---

## Excessive page faults

The working set does not fit comfortably in RAM.

Possible result:

* High storage activity
* Poor responsiveness
* Thrashing

---

## Dirty pages cannot be written

A storage failure or full backing store may prevent memory contents from being preserved.

Possible result:

* I/O errors
* Process failure
* System instability

---

## Allocation failure

The system cannot satisfy a memory request because of:

* Address-space exhaustion
* Physical-memory pressure
* Commitment limits
* Process resource limits
* Fragmentation
* Kernel resource exhaustion

Applications must not assume every allocation succeeds.

---

## Kernel page-table error

Incorrect mappings can expose:

* Another process’s data
* Kernel memory
* Device memory
* Writable executable regions

This can become a serious security failure.

---

# 8.41 Common Misconceptions

## Misconception: “The stack is in RAM, while the heap is in virtual memory”

Both are virtual-memory regions whose active pages may reside in physical RAM.

---

## Misconception: “Each process has one stack”

Each thread normally has its own stack.

A multi-threaded process therefore usually has multiple stacks.

---

## Misconception: “The heap is one large unordered pile”

The heap is carefully managed by allocators.

“Heap” does not mean that its internal organization is random.

---

## Misconception: “Stack variables are always physically adjacent”

Stack frames are logically arranged within a thread’s virtual address space.

Their pages may map to scattered physical frames.

---

## Misconception: “Heap allocation always requires a system call”

A user-space allocator may satisfy the request from memory it already manages.

---

## Misconception: “Releasing heap memory always returns RAM immediately to the OS”

The allocator may retain the region for reuse.

---

## Misconception: “A successful allocation means all requested RAM is physically available now”

Some systems reserve virtual space and provide physical frames only when pages are accessed.

Exact guarantees depend on OS policy.

---

## Misconception: “All page faults are errors”

Many are normal parts of:

* Demand allocation
* Program loading
* Stack growth
* Copy-on-write
* Memory mapping

---

## Misconception: “A page fault means the page must come from swap”

The page may instead be:

* Newly zero-filled
* Loaded from a program file
* Already available in shared RAM
* Copied from another page
* Invalid and rejected

---

## Misconception: “A page fault always causes a context switch”

The kernel may resolve a minor fault and return to the same thread.

A switch becomes likely if slow I/O is required or another thread should run.

---

## Misconception: “Stack memory is always faster than heap memory”

Stack allocation is often simpler, but memory-access speed depends on caches, locality, page state, and access patterns.

---

## Misconception: “Process isolation prevents all memory corruption”

It mainly separates processes.

Threads and objects inside the same process can still corrupt shared process memory.

---

# 8.42 Real-World Analogy: Office Desks, Archive Rooms, and Storage Boxes

Imagine a company office.

## Stack: each employee’s active desk tray

Each employee has a tray for the tasks currently being handled.

```text
Top task: answer current request
Below it: prepare report
Below it: review document
```

The newest nested task is completed first.

Each employee needs a separate tray, just as each thread needs a stack.

---

## Heap: shared storage room

The office has a storage room containing boxes of different sizes.

Boxes can be assigned and returned in any order.

Several employees may use information in the storage room.

This resembles process-wide heap memory.

---

## Pages: standard-sized storage shelves

The building organizes physical storage using equal-sized shelf units.

A large box may occupy several shelf units.

This resembles pages and frames.

---

## Page table: storage catalog

The catalog maps logical box locations to actual shelves.

---

## Page fault: requested material is not currently on-site

The requested box may be:

* In an off-site archive
* Not yet prepared
* Waiting to be copied
* Restricted

The storage manager investigates.

### Valid request

The box is retrieved and work continues.

### Invalid request

The employee is denied access.

---

# 8.43 Connection to Earlier Concepts

## Connection to programs and processes

A process’s virtual address space contains:

* Program instructions
* Data
* Heap
* Thread stacks
* Shared mappings

---

## Connection to threads

Each thread normally owns a separate stack.

Threads generally share the process heap.

---

## Connection to virtual memory

Stacks and heaps are built from virtual pages.

The kernel and MMU map those pages to physical frames.

---

## Connection to exceptions

A missing or forbidden page causes a page-fault exception.

The kernel decides whether to repair or reject it.

---

## Connection to system calls

Applications or allocators may use system calls to:

* Obtain additional virtual memory
* Release regions
* Change permissions
* Map files
* Create shared memory

---

## Connection to scheduling

If a page fault requires storage I/O:

1. The faulting thread becomes waiting.
2. Another thread runs.
3. Storage later generates an interrupt.
4. The original thread becomes ready.

---

## Connection to process isolation

The kernel clears newly assigned frames and maintains separate page tables.

This helps prevent one process from reading another process’s old data.

---

# 8.44 Simplified Model Versus Technical Reality

## Simplified mental model

```text
Stack:
temporary nested thread work

Heap:
dynamic process data

Pages:
fixed-size virtual-memory blocks

Page fault:
kernel attention required for a memory access
```

This is the model to retain.

---

## More exact reality

Real memory systems may include:

* Multiple stacks per thread for different privilege modes
* Language-managed heaps
* Garbage collection
* Several allocators inside one process
* Memory-mapped files
* Shared-memory regions
* Huge pages
* Memory compression
* Copy-on-write
* Guard pages
* Stack growth limits
* Multi-level page tables
* Non-uniform memory hardware
* Hypervisor translation layers
* Process-specific and shared kernel mappings

Not every process organizes memory identically.

Not every language exposes direct heap management.

The essential principles remain:

> A stack tracks one thread’s active execution structure.

> A heap provides flexible process-level dynamic storage.

> Paging maps virtual pages to physical frames.

> A page fault asks the kernel to resolve or reject a memory access that hardware cannot currently complete.

---

# 8.45 Core Mental Model

## Stack model

```text
Thread begins nested work
          │
          ▼
Push stack frame
          │
          ▼
Nested work finishes
          │
          ▼
Pop stack frame
```

---

## Heap model

```text
Application needs long-lived data
             │
             ▼
Ask allocator for block
             │
        ┌────┴────┐
        │         │
Reuse free      Request more
heap block      memory from OS
```

---

## Page-fault model

```text
Process accesses virtual address
             │
             ▼
MMU checks mapping
       ┌─────┴─────┐
       │           │
Access valid    Cannot proceed
and present         │
       │            ▼
Continue       Page-fault exception
                    │
                    ▼
             Kernel investigates
               ┌────┴────┐
               │         │
            Recoverable Invalid
               │         │
         Prepare page   Reject access
               │         │
         Retry access   Process may stop
```

---

## Final distinctions

| Concept            | Essential meaning                                |
| ------------------ | ------------------------------------------------ |
| **Stack**          | Per-thread nested execution memory               |
| **Heap**           | Flexible dynamic process memory                  |
| **Page**           | Fixed-size virtual-memory unit                   |
| **Frame**          | Fixed-size physical-RAM unit                     |
| **Page table**     | Virtual-page-to-frame mapping                    |
| **Page fault**     | Exception requiring kernel memory handling       |
| **Stack overflow** | Stack exceeds its valid region                   |
| **Memory leak**    | Unneeded allocated memory remains retained       |
| **Thrashing**      | Excessive page movement prevents useful progress |

The next section examines persistent data:

* Files
* Directories
* File descriptors
* File systems
* How a process reads and writes stored information

# Learning Check

Do not look for answers yet.

## Conceptual questions

1. What is the main difference between the purpose of a thread’s stack and the purpose of a process heap?
2. What is the relationship among a virtual page, a physical frame, and a page-table entry?
3. Why can a valid memory access cause a page fault?

## Cause-and-effect questions

4. Why might releasing heap memory not immediately reduce the process’s reported physical-memory usage?
5. Why can a page fault requiring storage access lead to a context switch, while a demand-zero page fault may return quickly to the same thread?

## Misconception question

6. A student says, “A page fault means the operating system has detected a programming bug.” What is incomplete or incorrect about this statement?

## Scenario-based question

7. An image editor requests a large heap region, writes to only its first quarter, and later releases the image. Explain the roles of the allocator, virtual-memory mappings, demand paging, physical frames, page faults, and eventual memory reuse.

# 9. Files, Directories, File Descriptors, and File Systems

Memory is temporary. When a process ends or the computer loses power, ordinary process memory usually disappears.

Applications therefore need a way to preserve information across:

* Process termination
* User logouts
* Computer restarts
* Power loss

The operating system provides **files** and **file systems** for persistent data.

```text
Application
    │
    │ reads and writes logical files
    ▼
Operating-system file interface
    │
    ▼
File system
    │
    ▼
Storage device
```

---

# 9.1 What Is a File?

## What it is

A **file** is an operating-system abstraction representing a named collection of data and associated information.

Examples include:

* A text document
* An image
* A video
* An installed program
* A configuration file
* A database
* A system log

A file commonly has:

* Contents
* A name
* A size
* An owner
* Permissions
* Timestamps
* A location within a directory structure
* File-system-specific metadata

---

## Why files exist

Raw storage devices do not naturally present information as:

```text
report.pdf
holiday.jpg
notes.txt
```

At a lower level, storage is organized as addressable blocks or device-specific storage units.

Applications and users need a more meaningful abstraction.

The file abstraction solves questions such as:

* Which data belongs together?
* What is it called?
* How large is it?
* Who may access it?
* Where does its content begin and end?
* How can it survive after a program closes?

---

## Mental model: a labeled document folder

Think of a file as a labeled document held by a records department.

The user sees:

```text
project-report.txt
```

The records department tracks:

* Where its pages are stored
* Who owns it
* Who may read it
* When it was modified
* How many pages it contains

The user does not need to know the exact shelf and box for every page.

---

## Simplified model versus exact reality

### Simplified model

> A file is a named sequence of bytes stored on a disk.

This is a useful foundation.

### More exact reality

Depending on the operating system and file type, a file may represent or interact with:

* Ordinary stored data
* A device
* A communication channel
* A generated information source
* A remote network resource
* A memory-backed object

For now, focus on ordinary persistent files.

---

# 9.2 File Contents and File Format

The operating system generally treats an ordinary file’s contents as raw data.

It may know:

* The file’s size
* Storage location
* Permissions
* Timestamps

But it may not understand the meaning of the data.

For example:

```text
Same bytes
   │
   ├── Image viewer interprets them as an image
   ├── Text editor may display unreadable symbols
   └── File-inspection tool may identify a format
```

The **file format** defines how an application should interpret the contents.

Examples:

| File format       | Interpretation                            |
| ----------------- | ----------------------------------------- |
| Plain text        | Character data                            |
| JPEG              | Compressed image information              |
| PDF               | Structured document information           |
| Executable format | Program instructions and loading metadata |

The file system stores the data. Applications usually interpret its higher-level meaning.

---

# 9.3 File Metadata

**Metadata** is information about a file rather than the file’s main contents.

Common metadata includes:

* File size
* Owner
* Permissions
* Creation or modification times
* File type
* Storage-location information
* Whether the file is a directory
* File-system flags

```text
File: report.txt

Contents:
“Operating systems manage resources...”

Metadata:
Size: 37 bytes
Owner: user A
Permissions: readable and writable by owner
Modified: particular date and time
```

---

## Why metadata exists

The OS needs information for tasks such as:

* Enforcing permissions
* Finding file contents
* Listing directories
* Detecting changes
* Managing storage
* Identifying file types
* Recovering file-system state

Metadata is normally managed by the file system rather than embedded entirely inside the file’s visible contents.

---

# 9.4 What Is a Directory?

## What it is

A **directory** is a file-system object that organizes names and references to other file-system objects.

Directories may contain entries referring to:

* Files
* Other directories
* Links
* Special file-system objects

```text
Documents/
├── notes.txt
├── report.pdf
└── Projects/
    ├── design.txt
    └── diagram.png
```

---

## Why directories exist

Without directories, every file might need one globally unique name.

A system containing thousands or millions of files would become difficult to organize.

Directories solve several problems:

* Grouping related files
* Allowing repeated filenames in different locations
* Providing hierarchical organization
* Applying permissions to collections
* Giving applications predictable locations

---

## Mental model: filing cabinets and folders

A directory is like a folder in a filing cabinet.

A folder may contain:

* Documents
* Other folders
* Labels pointing to stored records

The directory does not necessarily contain all file data physically inside itself.

It mainly organizes names and references.

---

# 9.5 Directory Entries

A directory contains **directory entries**.

A directory entry associates a name with a file-system object.

Conceptually:

```text
Directory: Projects

Entry name       Refers to
--------------------------------
design.txt    →  file object 512
diagram.png   →  file object 847
archive       →  directory object 930
```

The name and the underlying file object are related but distinct.

This distinction becomes important when files are renamed or linked.

---

# 9.6 Paths

A **path** describes how to locate a file or directory through the directory hierarchy.

Example:

```text
Documents/Projects/design.txt
```

The path can be read as:

1. Find `Documents`.
2. Inside it, find `Projects`.
3. Inside that, find `design.txt`.

```text
Documents
    │
    ▼
Projects
    │
    ▼
design.txt
```

---

## Absolute path

An **absolute path** begins from a file system’s defined starting point, commonly called its root.

Conceptually:

```text
/root-start/Documents/Projects/design.txt
```

It identifies a location independently of the process’s current directory.

---

## Relative path

A **relative path** is interpreted starting from some existing directory, often the process’s current working directory.

Suppose the current directory is:

```text
Documents
```

Then:

```text
Projects/design.txt
```

can locate the same file.

---

## Current working directory

A process may have a **current working directory**.

This acts as the starting point for many relative paths.

```text
Process working directory:
Documents/Projects

Relative path:
design.txt
```

The OS combines them conceptually:

```text
Documents/Projects/design.txt
```

---

# 9.7 Special Path Components

Many file systems recognize concepts similar to:

```text
.   current directory
..  parent directory
```

Conceptually:

```text
Documents/Projects/.
```

still refers to `Projects`.

```text
Documents/Projects/..
```

refers to `Documents`.

Exact path syntax differs among operating systems.

---

# 9.8 A Filename Is Not the Whole Identity of a File

A filename is a name within a particular directory.

Consider:

```text
Work/report.txt
School/report.txt
```

Both entries use the name `report.txt`, but they may refer to different files.

```text
Work/report.txt   → File A
School/report.txt → File B
```

Therefore:

> A filename alone is not necessarily enough to uniquely identify a file.

Its directory context matters.

---

# 9.9 What Is a File System?

## What it is

A **file system** is the operating-system mechanism and on-storage data structure used to organize files, directories, metadata, and free storage space.

It defines matters such as:

* How files are represented
* How directories map names to objects
* How storage blocks are assigned
* How free space is tracked
* How metadata is stored
* How damage may be detected or recovered
* How permissions and timestamps are represented

---

## Why file systems exist

A storage device provides capacity, but capacity alone does not provide organization.

Without a file system, the OS would not automatically know:

* Which blocks belong to which file
* Which blocks are free
* Where a file ends
* Which names exist
* How directories are connected
* What permissions apply
* Whether stored structures are consistent

The file system turns raw storage capacity into manageable persistent objects.

---

## Mental model: a warehouse inventory system

Consider a warehouse containing many numbered storage spaces.

The file system acts as the inventory system that records:

* Which spaces are free
* Which spaces hold each item
* Which customer owns an item
* Which label refers to which item
* Whether an item spans several spaces
* When the records were last updated

| Warehouse           | File system         |
| ------------------- | ------------------- |
| Storage spaces      | Storage blocks      |
| Stored item         | File data           |
| Item label          | Filename            |
| Inventory record    | File metadata       |
| Sections and aisles | Directories         |
| Free-space map      | Free-block tracking |

---

# 9.10 Storage Blocks

Storage devices commonly transfer data in blocks rather than as individual conceptual files.

A file may occupy several blocks:

```text
File: image.jpg

Logical file data:
[Part 1][Part 2][Part 3][Part 4]

Possible storage locations:
Block 51  → Part 1
Block 900 → Part 2
Block 14  → Part 3
Block 72  → Part 4
```

The blocks do not always need to be physically adjacent.

The file system records how the pieces belong together.

---

## Why files may become scattered

As files are created, expanded, and deleted:

```text
[Used][Free][Used][Free][Used][Free]
```

A new file may be placed into several available regions.

This resembles memory fragmentation, although storage and memory management use different mechanisms and policies.

---

# 9.11 Logical File Position Versus Physical Storage Position

An application sees a file as a logical sequence:

```text
Byte 0
Byte 1
Byte 2
...
Byte N
```

It may request:

> Read 1,000 bytes beginning at logical position 5,000.

The file system translates that logical position into storage locations.

```text
Application file offset 5000
             │
             ▼
File-system mapping
             │
             ▼
Relevant storage block
```

The application usually does not need to know the physical storage address.

---

# 9.12 File Offset

An open file is often associated with a **file offset**, also called a file position.

The offset indicates where the next sequential read or write will occur.

Example:

```text
Initial offset: 0
Read 100 bytes
New offset: 100
Read 50 bytes
New offset: 150
```

```text
File:
[0────────────────────────────500]

Current offset:
                 ▲
                150
```

Applications may also request access at a specific position without relying entirely on one shared sequential offset.

---

# 9.13 Opening a File

A process normally does not repeatedly supply a complete pathname for every byte it reads.

Instead, it first **opens** the file.

Opening establishes a kernel-managed relationship between the process and the file.

```text
Pathname
   │
   │ open request
   ▼
Kernel resolves path and checks permission
   │
   ▼
Process receives a file descriptor
```

---

## Why opening exists

Opening allows the kernel to:

* Resolve the path once
* Verify permissions
* Record access mode
* Track the current offset
* Prepare internal file-system state
* Associate future requests with the correct object

This makes later reads and writes more efficient and controlled.

---

# 9.14 What Is a File Descriptor?

## What it is

A **file descriptor** is a small process-local identifier representing an open file or another OS-managed input/output resource.

Conceptually:

```text
Process file-descriptor table

Descriptor 0 → input source
Descriptor 1 → output destination
Descriptor 2 → error destination
Descriptor 3 → notes.txt
Descriptor 4 → network connection
```

The exact conventions vary, but small integers are commonly used.

---

## Why file descriptors exist

The process needs a simple way to refer to resources that the kernel controls.

Instead of sending a full path and repeating permission checks for every read, the process can say:

> Read from descriptor 3.

The kernel uses the process’s descriptor table to find the corresponding open resource.

---

## Mental model: coat-check ticket

At a coat-check desk:

1. You hand over your coat.
2. The attendant stores it.
3. You receive ticket `37`.
4. Later, you present ticket `37`.
5. The attendant uses internal records to find your coat.

| Coat check        | File access             |
| ----------------- | ----------------------- |
| Coat              | File or resource        |
| Ticket number     | File descriptor         |
| Attendant’s table | Kernel descriptor table |
| Handing over coat | Opening resource        |
| Returning ticket  | Closing descriptor      |

The ticket is meaningful only within the relevant coat-check system.

Similarly, a file descriptor is generally meaningful only inside its process.

---

# 9.15 File Descriptors Are Process-Local

Process A may have:

```text
Descriptor 3 → report.txt
```

Process B may have:

```text
Descriptor 3 → network connection
```

The number `3` does not globally identify one file.

```text
Process A descriptor 3 ≠ Process B descriptor 3
```

The kernel first identifies the calling process, then looks up the descriptor in that process’s table.

---

# 9.16 A File Descriptor Is Not the File Itself

A descriptor is a reference to an open kernel-managed object.

```text
File descriptor
      │
      ▼
Process descriptor-table entry
      │
      ▼
Kernel open-file state
      │
      ▼
File-system object
      │
      ▼
Stored file data
```

Closing a descriptor removes the process’s reference. It does not normally delete the underlying file.

---

# 9.17 Open-File State

The kernel may maintain information such as:

* Which file is open
* Access mode
* Current offset
* Status flags
* References from processes
* File-system-specific state

```text
Open-file state
├── Target file
├── Current offset
├── Read/write mode
├── Status options
└── Reference count
```

A process descriptor points toward this state.

---

# 9.18 Step-by-Step: Opening a File

Suppose a text editor opens:

```text
Documents/notes.txt
```

## Step 1: Application prepares the path

The editor identifies the requested pathname and desired access, such as reading.

**Component:** Application
**Mode:** User mode

---

## Step 2: Application makes an open system call

Control enters the kernel.

**Mode:** User → kernel

---

## Step 3: Kernel validates application memory

The path text itself resides in process memory.

The kernel verifies that it can safely read the supplied information.

---

## Step 4: Kernel determines the starting directory

If the path is relative, the kernel begins from the process’s current working directory.

If absolute, it begins from the file-system root or equivalent starting point.

---

## Step 5: Kernel resolves each path component

Conceptually:

```text
Documents
    │
    ▼
notes.txt
```

For each component, the kernel:

* Finds the directory entry
* Checks that the current object is a directory when needed
* Checks traversal permissions
* Moves to the referenced next object

---

## Step 6: Kernel finds the final file object

The file system retrieves the file’s metadata.

---

## Step 7: Kernel checks permissions

It determines whether the process may perform the requested operation.

Questions may include:

* Is reading allowed?
* Is writing allowed?
* Is the file protected?
* Does the current user identity have access?

---

## Step 8: Kernel creates open-file state

The kernel records:

* File reference
* Initial offset
* Access mode
* Requested options

---

## Step 9: Kernel allocates a descriptor

A free entry is selected in the process’s descriptor table.

Example:

```text
Descriptor 5 → notes.txt
```

---

## Step 10: Kernel returns the descriptor

**Mode:** Kernel → user

The application now uses descriptor `5` for later file operations.

---

## Flow diagram

```text
Application supplies path
          │
          ▼
System call enters kernel
          │
          ▼
Resolve directories and filename
          │
          ▼
Check permissions
          │
          ▼
Create open-file state
          │
          ▼
Add descriptor-table entry
          │
          ▼
Return descriptor
```

---

# 9.19 Step-by-Step: Reading from a File Descriptor

Suppose the process wants the next 4,096 bytes from descriptor `5`.

## Step 1: Application prepares the request

It provides:

* Descriptor number
* Amount requested
* Destination memory region

**Mode:** User mode

---

## Step 2: Read system call enters the kernel

**Mode:** User → kernel

---

## Step 3: Kernel validates the descriptor

The kernel looks up descriptor `5` in the calling process’s table.

Possible failure:

```text
Descriptor 5 does not exist
```

The kernel returns an error.

---

## Step 4: Kernel validates access mode

A descriptor opened only for writing cannot normally be used for reading.

---

## Step 5: Kernel validates destination memory

The kernel checks that the process supplied a valid writable memory region.

---

## Step 6: Kernel checks current file offset

Suppose the offset is:

```text
8,192
```

The read begins from that logical file position.

---

## Step 7: File system identifies the required data

It translates the logical file range into the relevant cached pages or storage blocks.

---

## Step 8: Kernel checks the file cache

### Data already cached

The kernel can supply it from RAM.

### Data not cached

Storage I/O may be required.

---

## Step 9: If necessary, the thread waits

The storage driver starts the device operation.

The calling thread changes:

```text
RUNNING → WAITING
```

The scheduler runs another thread.

---

## Step 10: Storage completion generates an interrupt

The kernel processes the completed I/O.

The original thread becomes ready.

---

## Step 11: Data is made available to the process

The kernel transfers or maps the requested information into the process’s destination memory.

---

## Step 12: File offset advances

If 4,096 bytes were read:

```text
Old offset: 8,192
New offset: 12,288
```

---

## Step 13: System call returns

The kernel reports:

* Number of bytes read
* End-of-file condition
* Or an error

**Mode:** Kernel → user

---

# 9.20 End of File

A file has a logical end.

Suppose the application asks for 4,096 bytes, but only 500 remain.

The kernel may return:

```text
500 bytes read
```

A later read may indicate:

```text
No more data: end of file
```

End of file is not necessarily an error. It means the current reading position has reached the file’s logical end.

---

# 9.21 Partial Reads and Writes

A request to read or write a certain amount does not always complete the full amount in one operation.

For example, an application may request:

```text
Read 4,096 bytes
```

but receive:

```text
1,500 bytes
```

Possible reasons include:

* Reaching end of file
* Device or communication behavior
* Signal or event interruption
* Resource limitations
* Non-blocking operation
* File-system constraints

Applications must treat the returned amount as authoritative.

---

# 9.22 Writing to a File

Writing generally follows a similar pattern.

```text
Application data
      │
      ▼
Write system call
      │
      ▼
Kernel validates descriptor and memory
      │
      ▼
File-system data and metadata updated
      │
      ▼
Storage eventually updated
```

---

## Step-by-Step: Writing Data

Suppose a text editor saves new document contents.

### Step 1: Application prepares data

The edited document exists in process memory.

### Step 2: Application makes a write request

It provides:

* File descriptor
* Source memory region
* Amount to write

### Step 3: Kernel validates the descriptor

The file must be open with suitable write access.

### Step 4: Kernel validates source memory

The process must own readable memory containing the data.

### Step 5: File system determines placement

The file system may need to:

* Reuse existing blocks
* Allocate additional blocks
* Update file size
* Update modification time
* Update allocation metadata

### Step 6: Data may enter an OS cache

The write may initially update RAM-backed file-system cache.

### Step 7: File offset advances

The next sequential write continues from the new position.

### Step 8: System call returns

The application is told how much data the OS accepted.

### Step 9: Storage may be updated later

The OS may defer physical storage writes for efficiency.

This distinction is critical.

---

# 9.23 Buffered and Cached Writes

A successful write request does not always mean the data has already reached the physical storage medium.

A common simplified sequence is:

```text
Application
    │ write
    ▼
Kernel file cache in RAM
    │ later
    ▼
Storage device
```

The operating system may report success after accepting the data into protected memory buffers.

---

## Why writes are buffered

Buffering allows the OS to:

* Combine many small writes
* Reorder device operations
* Reduce storage overhead
* Let the application continue sooner
* Improve overall performance

---

## The risk

If power fails before dirty cached data reaches persistent storage, recent changes may be lost.

```text
Application writes data
        │
        ▼
Data exists only in RAM cache
        │
        ▼
Power failure
        │
        ▼
Data may be lost
```

Applications requiring stronger persistence guarantees must request them through appropriate OS mechanisms.

---

# 9.24 Cache Versus Buffer

These terms are often used loosely.

## Cache

Keeps copies of data so future access can be faster.

Example:

```text
Previously read file data retained in RAM
```

---

## Buffer

Temporarily holds data while it moves between components that operate differently.

Example:

```text
Application produces small pieces
Buffer collects them
Storage receives a larger combined write
```

One memory region may serve both roles, so the distinction is conceptual rather than always physically separate.

---

# 9.25 The Page Cache or File Cache

Operating systems commonly use unused RAM to cache file contents.

```text
Storage file
    │ first read
    ▼
RAM file cache
    │ later read
    ▼
Application receives data quickly
```

---

## Why use RAM for file caching?

RAM is much faster than persistent storage.

If data has been read recently, keeping it in RAM can avoid another device operation.

---

## Is cached memory wasted?

No.

Cached file pages can often be reclaimed when applications need memory.

```text
RAM used for cache
       │
       │ application needs memory
       ▼
Discard clean cache page
       │
       ▼
Frame reused
```

Using otherwise idle RAM for cache usually improves performance.

---

# 9.26 Memory-Mapped Files

A file can sometimes be connected directly to a process’s virtual address space.

This is called **memory mapping** a file.

```text
Process virtual pages
        │
        ▼
File-backed mapping
        │
        ▼
File data
```

The process accesses the mapped region using ordinary memory operations.

The virtual-memory system and file system cooperate to load pages when needed.

---

## Step-by-step concept

1. Process requests a file mapping.
2. Kernel creates virtual-memory mappings associated with the file.
3. Process accesses a mapped page.
4. If absent, a page fault occurs.
5. Kernel loads the corresponding file data.
6. Page table is updated.
7. Memory access retries.
8. Modified pages may later be written back.

---

## Why memory-mapped files exist

They can provide:

* Convenient random access
* Integration with virtual-memory paging
* Efficient sharing
* Reduced copying in some situations
* Shared file-backed memory among processes

They do not eliminate all file-system or storage work.

---

# 9.27 Closing a File Descriptor

When a process no longer needs an open descriptor, it should close it.

```text
Descriptor 5 → notes.txt
       │ close
       ▼
Descriptor-table entry removed
```

---

## Step-by-step

1. Application requests that descriptor `5` be closed.
2. Control enters the kernel.
3. Kernel verifies the descriptor exists.
4. The descriptor-table entry is removed.
5. Kernel reduces the open object’s reference count.
6. If no references remain, associated open-file state can be released.
7. Pending errors or finalization work may be reported.
8. The descriptor number may later be reused.

---

## Closing is not deleting

Closing means:

> This process no longer needs this open reference.

Deleting means:

> Remove a directory name and possibly allow the file’s storage to be reclaimed.

These are different operations.

---

# 9.28 File-Descriptor Leaks

A **file-descriptor leak** occurs when a process repeatedly opens resources but fails to close descriptors it no longer needs.

```text
Time ─────────────────────────────────────▶

Open descriptors:
10 → 20 → 50 → 200 → 1,000 → limit reached
```

Possible effects:

* New files cannot be opened.
* Network connections fail.
* Pipes cannot be created.
* Kernel resources are exhausted.
* The application becomes unreliable.

The OS normally reclaims descriptors when the process terminates, but not while the leaking process remains alive.

---

# 9.29 Standard Input, Output, and Error

Many operating systems give a newly started process predefined descriptors representing:

* Standard input
* Standard output
* Standard error

A common conceptual arrangement is:

| Descriptor | Conventional role |
| ---------: | ----------------- |
|          0 | Standard input    |
|          1 | Standard output   |
|          2 | Standard error    |

These conventions are especially common in Unix-like systems.

---

## Why they exist

An application can read input and produce output without knowing exactly where those streams lead.

For example:

```text
Standard input  → keyboard, file, or another process
Standard output → terminal, file, or another process
Standard error  → terminal, log, or another destination
```

This allows flexible composition.

---

## Important mental model

A descriptor does not necessarily refer to a regular file.

It may refer to:

* Terminal input
* A network connection
* A pipe
* A device
* A communication endpoint

The descriptor is a general kernel-managed I/O handle.

---

# 9.30 Files and Devices

Some operating systems expose devices through file-like interfaces.

For example, software may use file-style operations such as:

* Open
* Read
* Write
* Close

even when the underlying object is a device rather than persistent storage.

```text
Application
    │ read/write interface
    ▼
File-like kernel object
    │
    ▼
Device driver
    │
    ▼
Hardware
```

This provides a consistent abstraction, although devices do not behave exactly like ordinary files.

---

# 9.31 Renaming a File

Renaming often changes a directory entry rather than rewriting the file’s contents.

Conceptually:

```text
Before:
notes.txt → File object 512

After:
ideas.txt → File object 512
```

The underlying file data may remain unchanged.

This is why renaming can often be much faster than copying all file contents.

---

# 9.32 Deleting a File

Deleting a file usually begins by removing a directory entry.

```text
Directory entry:
notes.txt → File object 512
```

After deletion:

```text
notes.txt entry removed
```

Whether the data is immediately reclaimed depends on whether other references remain.

---

## Open file after deletion

In some file-system designs, a process may continue using an already open file even after its directory name is removed.

Conceptually:

```text
Directory name removed
        │
        ▼
Open descriptor still refers to file object
        │
        ▼
Process can continue using it
```

The storage is reclaimed only when:

* No directory links remain
* No open references remain
* File-system rules permit reclamation

This shows that a filename and an open file object are not identical.

---

# 9.33 Links

A **link** allows one file-system object to be reachable through another name or reference.

Two broad concepts are useful.

## Direct or hard link concept

Multiple directory entries refer to the same underlying file object.

```text
name-A.txt ─┐
            ├──▶ Same file object
name-B.txt ─┘
```

Changing the file through either name changes the same underlying contents.

---

## Symbolic-link concept

One file-system object stores a path referring to another location.

```text
shortcut.txt
      │ contains reference path
      ▼
Documents/original.txt
```

If the destination moves or disappears, the symbolic link may become broken.

Exact rules differ across file systems.

---

# 9.34 Mounting File Systems

A computer may use several storage devices or file systems.

The OS can attach a file system into the overall directory hierarchy. This is commonly called **mounting**.

Conceptually:

```text
Main directory tree
├── Documents
├── Applications
└── ExternalDrive
    └── contents of another file system
```

The user sees one navigable hierarchy, even though different sections may come from different devices or file-system types.

Other operating systems may expose separate drive or volume names instead of one unified tree.

---

# 9.35 Volumes and Partitions

These terms are related but not identical.

## Physical device

The actual SSD, hard disk, or other hardware.

## Partition

A defined region of a physical storage device.

## Volume

A logical storage unit presented to the operating system.

## File system

The organization placed on a volume or equivalent storage object.

A simplified model:

```text
Physical SSD
├── Partition A
│   └── File system A
└── Partition B
    └── File system B
```

Real storage stacks may also include:

* Encryption
* RAID
* Logical-volume management
* Network storage
* Virtual disks

---

# 9.36 Free-Space Management

The file system must know which storage blocks are unused.

A simplified free-space map might look like:

```text
Block 0: used
Block 1: free
Block 2: free
Block 3: used
Block 4: used
Block 5: free
```

When a file grows, the file system selects free blocks.

When storage is reclaimed, those blocks return to the free-space system.

---

## What happens when storage is full?

Possible effects include:

* New files cannot be created.
* Existing files cannot grow.
* Applications cannot save work.
* Logs cannot be updated.
* Databases may fail.
* The OS may become unstable if critical components cannot write data.

A successful file open does not guarantee that all future writes will succeed.

---

# 9.37 File-System Consistency

The file system maintains several related pieces of information:

* Directory entries
* File metadata
* Allocated data blocks
* Free-space records
* Timestamps
* Permission state

A single operation may require several updates.

For example, creating a file may involve:

1. Allocate a file metadata object.
2. Add a directory entry.
3. Allocate data blocks.
4. Update free-space tracking.
5. Update timestamps.

If power fails halfway through, these structures could disagree.

---

# 9.38 Journaling

Some file systems use a **journal** to help recover from interrupted updates.

The simplified idea is:

1. Record the planned metadata change in a journal.
2. Perform the actual file-system updates.
3. Mark the operation complete.
4. After a crash, inspect the journal.
5. Replay or discard incomplete operations safely.

```text
Planned change
      │
      ▼
Journal record
      │
      ▼
Apply file-system updates
      │
      ▼
Mark complete
```

---

## What journaling solves

It reduces the risk that a crash leaves core file-system structures inconsistent.

---

## What journaling does not guarantee

Journaling does not automatically guarantee:

* Every recent application write survives
* No file data is lost
* No application-level corruption occurs
* Storage hardware never fails

Different file systems journal different types of information.

---

# 9.39 Copying Versus Moving

## Copying

Creates new file data or a new independently managed file.

```text
Original file
      │ copy
      ├── Original remains
      └── New file created
```

This may require reading and writing the entire file.

---

## Moving within one file system

May only require changing directory entries.

```text
Old directory entry removed
New directory entry created
Underlying file object remains
```

This can be fast.

---

## Moving between file systems

May require:

1. Copy file contents to the destination.
2. Verify success.
3. Remove the source.

This behaves more like copy followed by deletion.

---

# 9.40 File Permissions: Introductory Model

The kernel checks file-system permissions when a process requests operations.

Common permission categories include:

* Read
* Write
* Execute
* Directory traversal
* File creation or deletion

The decision may depend on:

* Process user identity
* Group membership
* File owner
* Access-control lists
* Elevated privileges
* Application sandbox rules
* File-system policy

Security and permissions receive a fuller section later.

---

# 9.41 Directory Permissions Are Distinct

Permissions on a directory may control operations such as:

* Listing its entries
* Looking up a contained name
* Creating files inside it
* Removing entries
* Traversing through it

A process may have permission to read a file but still be unable to locate it through an inaccessible directory.

```text
Accessible file
      │
      ✕ parent directory cannot be traversed
      │
Process cannot reach file through that path
```

---

# 9.42 Step-by-Step: What Happens When a Process Reads a File

This walkthrough combines paths, descriptors, caching, system calls, scheduling, and device management.

Assume a text editor wants to read:

```text
Documents/report.txt
```

---

## Stage 1: Application chooses the file

The text editor receives the path from the user interface.

**Component:** Application
**Mode:** User mode

---

## Stage 2: Application requests that the file be opened

The editor makes an open system call containing:

* Path
* Desired read access
* Relevant options

**Component:** System-call interface
**Mode:** User → kernel

---

## Stage 3: Kernel resolves the path

The file-system layer walks through:

```text
Documents
    │
    ▼
report.txt
```

For each component, it consults directory entries and permissions.

**Component:** Kernel file-system layer

---

## Stage 4: Kernel finds file metadata

The file system obtains:

* File identity
* Size
* Permissions
* Storage mapping
* Timestamps

---

## Stage 5: Kernel checks access

It verifies that the calling process may read the file.

**Component:** Security and file-system permission mechanisms

---

## Stage 6: Kernel creates open-file state

It records:

* File reference
* Read mode
* Initial offset, commonly zero
* Relevant status flags

---

## Stage 7: Process receives a file descriptor

Example:

```text
Descriptor 6 → report.txt
```

**Mode:** Kernel → user

---

## Stage 8: Application requests file data

The editor asks to read a block of data using descriptor `6`.

**Mode:** User → kernel

---

## Stage 9: Kernel validates descriptor and memory

It checks:

* Descriptor exists
* Descriptor permits reading
* Destination buffer belongs to the process
* Requested length is valid

---

## Stage 10: File system checks the cache

### Cache hit

The requested file pages already exist in RAM.

The kernel can supply them without device I/O.

### Cache miss

The file system identifies the required storage blocks.

---

## Stage 11: Storage driver prepares the operation

The kernel sends a device-level request through the storage driver.

**Component:** Driver

---

## Stage 12: Calling thread waits

Because storage is slower than the CPU:

```text
Editor thread:
RUNNING → WAITING
```

**Component:** Scheduler

---

## Stage 13: Another ready thread runs

The CPU may run:

* A music thread
* A browser thread
* Another editor thread
* Kernel background work

---

## Stage 14: Storage device retrieves data

The SSD or other device reads the relevant blocks.

**Component:** Storage controller and device

---

## Stage 15: Device reports completion

A hardware interrupt transfers control to the kernel.

**Component:** Interrupt controller, CPU, and driver

---

## Stage 16: Kernel completes the request

It verifies completion, updates cache state, and associates the returned data with the pending file read.

---

## Stage 17: Editor thread becomes ready

```text
WAITING → READY
```

---

## Stage 18: Scheduler eventually selects the editor

The thread’s context is restored.

```text
READY → RUNNING
```

---

## Stage 19: Data is delivered to process memory

The kernel makes the read data available in the editor’s valid destination region.

---

## Stage 20: File offset is advanced

The next sequential read begins after the returned data.

---

## Stage 21: System call returns

The application receives:

* The data
* Number of bytes read
* Or an error

**Mode:** Kernel → user

---

## Stage 22: Application interprets the contents

The text editor decodes the bytes as text and updates its document model.

**Component:** Application

---

## Stage 23: Application requests display updates

The graphics system eventually displays the document.

---

## Complete flow

```text
Text editor
    │ open system call
    ▼
Kernel resolves path and permissions
    │
    ▼
File descriptor returned
    │
    │ read system call
    ▼
File-system cache checked
    │ cache miss
    ▼
Storage driver
    │
    ▼
Storage hardware
    │ completion interrupt
    ▼
Kernel wakes editor thread
    │
    ▼
Data returned to editor
    │
    ▼
Editor interprets and displays text
```

---

# 9.43 Step-by-Step: What Happens When a Process Saves a File

Suppose the editor saves modifications.

## Stage 1: Application prepares new contents

The document exists in process memory.

---

## Stage 2: Application opens or reuses a writable descriptor

The kernel checks write permission.

---

## Stage 3: Application requests writes

Data moves conceptually:

```text
Process memory
      │
      ▼
Kernel file cache
```

---

## Stage 4: File system updates logical state

It may modify:

* File size
* File data pages
* Timestamps
* Block allocation records

---

## Stage 5: Kernel reports accepted bytes

The application may receive success before every byte reaches persistent storage.

---

## Stage 6: Kernel schedules storage writes

Dirty cached pages are written later.

```text
Dirty RAM pages
      │
      ▼
Storage driver
      │
      ▼
Storage device
```

---

## Stage 7: Device reports completion or error

The kernel updates its records.

---

## Stage 8: Application closes the descriptor

Kernel references are released.

---

## Failure concern

A crash can occur between:

```text
Application sees write success
```

and:

```text
Physical storage safely contains all changes
```

Applications such as databases and editors may use careful save strategies to reduce corruption risk.

---

# 9.44 Safer Save Pattern

An application should not always overwrite an important file in place.

A simplified safer strategy is:

1. Write new contents to a temporary file.
2. Ensure the new file is complete.
3. Request required persistence guarantees.
4. Replace the old directory entry with the new file.
5. Remove obsolete temporary data.

```text
Old valid file remains
        │
        ▼
Write complete temporary version
        │
        ▼
Replace name atomically where supported
        │
        ▼
New valid file becomes current
```

This reduces the risk of leaving a half-written file, although exact guarantees depend on the file system and OS.

---

# 9.45 What Happens When a Process Crashes with Files Open?

When a process crashes:

1. The kernel stops its threads.
2. Its file descriptors are closed by the OS.
3. Kernel open-reference counts are reduced.
4. File locks may be released according to OS rules.
5. Process memory disappears.
6. Already accepted cached writes may still be written later.
7. Data that existed only in application memory is lost.
8. Partially completed application-level file formats may remain inconsistent.

---

## Important distinction

The OS can clean up kernel resources.

It cannot necessarily repair the logical contents of an application file.

Example:

```text
Database update:
Step 1 completed
Step 2 completed
Process crashes before Step 3
```

The file system may remain structurally valid, while the database’s internal data is logically inconsistent.

Applications needing strong consistency must implement their own recovery methods.

---

# 9.46 File-System Integrity Versus File-Content Integrity

These are different.

## File-system integrity

The file system correctly knows:

* Which files exist
* Which blocks belong to them
* Which blocks are free
* How directories are organized

## File-content integrity

The contents of a particular file represent a complete, meaningful application state.

A file system can be healthy while a document or database file is incomplete.

```text
File system:
File exists and blocks are valid

Application file:
Contents represent half-completed update
```

---

# 9.47 What Can Go Wrong?

## File not found

Possible causes:

* Incorrect path
* File was renamed
* File was deleted
* Wrong current working directory
* Broken symbolic link
* Unmounted storage

---

## Permission denied

The process lacks required permission for:

* Reading
* Writing
* Executing
* Directory traversal
* Creating or removing entries

---

## Descriptor is invalid

The application may:

* Use a descriptor that was never opened
* Use one after closing it
* Confuse descriptors between processes
* Attempt the wrong operation

---

## Storage is full

A write may fail after the file was opened successfully.

---

## Device failure

The storage device may:

* Return errors
* Lose data
* Become disconnected
* Stop responding

---

## Partial write

Only

* Stop responding

---

## Partial write

Only some requested data is accepted.

The application must handle the returned amount correctly.

---

## File-descriptor leak

Too many descriptors remain open.

---

## Lost buffered writes

Power failure occurs before dirty data reaches persistent storage.

---

## File corruption

Possible causes include:

* Interrupted updates
* Application defects
* File-system bugs
* Hardware failure
* Unsafe device removal
* Faulty RAM
* Malicious modification

---

## Race between processes

Two processes may modify the same file without coordination.

Possible result:

```text
Process A reads old file
Process B writes new file
Process A writes based on old data
Process B’s changes are lost
```

Synchronization and locking are needed for some shared-file workflows.

---

## Name changes during access

A file may be renamed or deleted after one process opens it.

An already open descriptor may still refer to the original file object.

The pathname and the open reference can therefore diverge.

---

# 9.48 Common Misconceptions

## Misconception: “A file is stored as one continuous region”

Not necessarily.

Its data may occupy scattered storage blocks.

---

## Misconception: “A directory physically contains all file contents”

A directory mainly maps names to file-system objects.

The file data may reside elsewhere.

---

## Misconception: “A file descriptor is a filename”

A descriptor is a process-local reference to an already opened resource.

---

## Misconception: “Descriptor 3 means the same file in every process”

Descriptor tables are normally process-specific.

---

## Misconception: “Closing a file deletes it”

Closing removes an open reference.

Deletion removes a directory entry and may eventually reclaim storage.

---

## Misconception: “Deleting a filename instantly destroys every open use of the file”

Some systems allow existing open descriptors to continue using the underlying file object.

---

## Misconception: “A successful write means the data is physically safe on storage”

The data may still exist only in RAM-backed caches.

---

## Misconception: “The OS understands every file format”

The file system usually manages bytes and metadata.

Applications interpret formats such as JPEG, PDF, or database structures.

---

## Misconception: “The extension determines the true file type”

A filename extension is mainly a naming convention.

The file contents may not match it.

---

## Misconception: “Available RAM used for file cache is wasted”

Cached data can improve performance and may be reclaimed when memory is needed.

---

## Misconception: “A file-system journal preserves every recent file change”

A journal primarily assists consistency and recovery. Exact data guarantees depend on the file system and write policy.

---

## Misconception: “One process writing a file is automatically coordinated with every other process”

The OS provides mechanisms, but applications often need explicit locking or transaction logic.

---

# 9.49 Real-World Analogy: Library Records System

Consider a large library.

## File

A book or document collection.

## Directory

A catalog section grouping related records.

## Filename

The title used in a particular catalog section.

## Path

Instructions for navigating through catalog sections.

## File system

The complete catalog, shelf-allocation, ownership, and tracking system.

## File descriptor

A temporary claim ticket issued after a reader requests access to a particular book.

## File offset

The reader’s current page or position.

## File cache

A frequently requested book placed at the service desk instead of returned immediately to distant storage.

## Closing

Returning the claim ticket.

## Deleting

Removing the book’s catalog entry and eventually reclaiming its shelf space.

---

# 9.50 Connection to Earlier Concepts

## Connection to system calls

Processes use system calls to:

* Open files
* Read and write data
* Change file position
* Inspect metadata
* Close descriptors
* Create and remove files

---

## Connection to user and kernel mode

Applications prepare requests in user mode.

The kernel validates paths, descriptors, permissions, and memory.

---

## Connection to virtual memory

File data may be:

* Copied into process memory
* Cached in RAM
* Memory mapped into virtual pages
* Loaded through page faults

---

## Connection to scheduling

A thread waiting for storage I/O moves to the waiting state.

Another thread runs until device completion.

---

## Connection to interrupts

Storage devices commonly notify the kernel when an operation finishes.

---

## Connection to processes

Each process has its own:

* Descriptor table
* Current working directory
* Permission identity
* Open-resource references

---

## Connection to memory

The OS often uses RAM as a file cache.

Memory pressure may cause clean cached file pages to be discarded and reloaded later.

---

# 9.51 Simplified Model Versus Technical Reality

## Simplified mental model

```text
File:
named persistent data

Directory:
names organized into a hierarchy

File descriptor:
process-local handle to an open resource

File system:
organization that maps files to storage
```

This is the model to retain.

---

## More exact reality

Real file systems may include:

* Journaling
* Copy-on-write storage
* Snapshots
* Compression
* Encryption
* Deduplication
* Network access
* Distributed storage
* Access-control lists
* Symbolic and hard links
* Multiple caches
* Memory-mapped files
* Sparse files
* Checksums
* Transactional updates

A file may represent a device or communication stream rather than persistent data.

A pathname can change while an open descriptor remains valid.

A successful write can have several possible durability meanings.

The central principle remains:

> Applications use logical names and file descriptors, while the kernel and file system manage permissions, metadata, caching, and physical storage placement.

---

# 9.52 Core Mental Model

## Opening

```text
Path
 │
 ▼
Kernel resolves directories
 │
 ▼
Permission check
 │
 ▼
Open-file state created
 │
 ▼
File descriptor returned
```

## Reading

```text
File descriptor
      │
      ▼
Kernel finds open file
      │
      ▼
File cache checked
   ┌──┴──┐
   │     │
 Hit    Miss
   │     │
Return  Storage I/O
data      │
          ▼
       Interrupt
          │
          ▼
       Thread wakes
```

## Writing

```text
Application data
      │
      ▼
Kernel accepts write
      │
      ▼
File-system cache becomes dirty
      │
      ▼
Storage updated later
```

## Closing

```text
Descriptor
    │
    ▼
Descriptor-table entry removed
    │
    ▼
Kernel reference released
```

## Final distinctions

| Concept             | Essential meaning                                 |
| ------------------- | ------------------------------------------------- |
| **File**            | Named collection of data and metadata             |
| **Directory**       | Mapping from names to file-system objects         |
| **Path**            | Route through the directory hierarchy             |
| **File descriptor** | Process-local identifier for an open resource     |
| **File offset**     | Current logical position in an open file          |
| **File system**     | Structure and rules organizing persistent storage |
| **File cache**      | RAM used to speed file access                     |
| **Close**           | Release an open reference                         |
| **Delete**          | Remove a name and eventually reclaim the file     |

The next section examines **input/output and device management**—how the OS communicates with keyboards, displays, disks, networks, audio devices, and other hardware.

# Learning Check

Do not look for answers yet.

## Conceptual questions

1. What is the difference between a pathname, a file descriptor, and the underlying file object?
2. What responsibilities belong to a file system rather than to the storage hardware itself?
3. Why can two processes both have a file descriptor numbered `3` referring to different resources?

## Cause-and-effect questions

4. Why might a successful file-write system call still be followed by data loss during an immediate power failure?
5. Why can a process continue reading an already opened file even after its directory name has been removed on some systems?

## Misconception question

6. A student says, “Closing a file descriptor deletes the file because the process no longer has access to it.” What is wrong with this explanation?

## Scenario-based question

7. A text editor opens a large document that is not present in the file cache. Describe path resolution, permission checks, descriptor creation, storage I/O, thread-state changes, interrupt handling, caching, and the eventual return of data to the editor.

# 10. Input/Output and Device Management

A computer must communicate with hardware outside the CPU and RAM.

Examples include:

* Keyboard input
* Mouse movement
* Display output
* Audio playback
* Storage reads and writes
* Network communication
* Camera capture
* Printing
* USB devices

These operations are collectively called **input/output**, usually shortened to **I/O**.

```text
Input devices
Keyboard, mouse, camera
          │
          ▼
Operating system and applications
          │
          ▼
Output devices
Display, speakers, printer
```

The operating system coordinates I/O because devices are shared, hardware-specific, slower than the CPU, and capable of failing independently.

---

# 10.1 What Is Input/Output?

## Input

**Input** is information entering the computer from an external source.

Examples:

* A keypress
* Mouse movement
* Network data
* Microphone samples
* Data read from storage
* Camera frames

---

## Output

**Output** is information leaving the CPU and memory system toward another component.

Examples:

* Pixels sent to a display
* Sound sent to speakers
* Data written to storage
* Network packets transmitted
* Pages sent to a printer

---

## I/O is broader than human interaction

Input and output do not always involve a person.

For example:

```text
SSD → RAM
```

is input from storage.

```text
RAM → SSD
```

is output to storage.

Similarly:

```text
Network adapter → application
```

is input, while:

```text
Application → network adapter
```

is output.

---

# 10.2 Why Device Management Exists

Hardware devices vary greatly.

A keyboard and an SSD differ in:

* Speed
* Data format
* Control commands
* Error behavior
* Timing
* Transfer size
* Communication method

Applications should not need to understand every device model.

The OS provides controlled and standardized interfaces.

```text
Application request
“Read some data”
        │
        ▼
Operating system
        │
        ▼
Device-specific driver
        │
        ▼
Particular hardware device
```

---

## Problems device management solves

The operating system must handle:

* Hardware differences
* Sharing among processes
* Device permissions
* Slow device response
* Device failures
* Transfer coordination
* Interrupt handling
* Buffering
* Request ordering
* Device discovery and removal

Without OS coordination, every application would need to solve these problems independently.

---

# 10.3 Mental Model: A Shipping Department

Imagine a company where many employees need to send and receive packages.

Employees do not individually:

* Drive delivery trucks
* Control loading docks
* Negotiate with every courier
* Track warehouse equipment
* Repair conveyor belts

Instead, they submit requests to a shipping department.

| Shipping department   | Computer system      |
| --------------------- | -------------------- |
| Employee              | Application          |
| Shipping request      | I/O request          |
| Shipping department   | OS I/O subsystem     |
| Courier specialist    | Device driver        |
| Truck or conveyor     | Hardware device      |
| Package queue         | Device request queue |
| Delivery notification | Interrupt            |

The shipping department coordinates shared equipment and handles device-specific details.

---

# 10.4 Major Categories of Devices

Devices can be grouped conceptually by how they communicate.

## Character or stream-oriented devices

These provide information as a stream of smaller units.

Examples may include:

* Keyboard
* Serial connection
* Terminal
* Some sensors

```text
Data arrives over time:

A → B → C → D
```

---

## Block-oriented devices

These transfer data in addressable blocks.

Examples include many storage devices.

```text
Block 0
Block 1
Block 2
...
```

Applications normally access files, while the file system and storage stack handle blocks.

---

## Network devices

Network adapters send and receive packets.

```text
Packet 1
Packet 2
Packet 3
```

Packets may arrive:

* At unpredictable times
* Out of order at higher protocol levels
* With errors
* Faster than an application can process them

The OS provides networking abstractions above the raw adapter.

---

## Human-interface devices

These include:

* Keyboard
* Mouse
* Touchscreen
* Game controller

They generate events based on user actions.

---

## Multimedia devices

Examples:

* Audio input and output
* Cameras
* Graphics processors
* Video-capture hardware

These often require continuous streams with timing constraints.

---

# 10.5 Physical Device, Controller, and Driver

These three concepts should be separated.

| Component           | Meaning                                                   |
| ------------------- | --------------------------------------------------------- |
| **Physical device** | The hardware performing the operation                     |
| **Controller**      | Hardware logic that manages communication with the device |
| **Driver**          | Software that controls the device through the controller  |

---

## Example: storage

```text
Kernel
  │
  ▼
Storage driver
  │
  ▼
Storage controller
  │
  ▼
SSD hardware
```

The controller may:

* Accept commands
* Transfer data
* Report errors
* Signal completion
* Manage low-level timing

The driver understands how to communicate with that controller.

---

# 10.6 Device Drivers

## What a driver is

A **device driver** is software that translates operating-system requests into operations understood by particular hardware.

A driver may handle:

* Device initialization
* Data-transfer setup
* Command submission
* Interrupt handling
* Error reporting
* Power management
* Device-specific settings

---

## Why drivers exist

The OS wants to express general operations such as:

* Read
* Write
* Start
* Stop
* Configure
* Send packet
* Display image

But different devices require different command sequences.

```text
OS request:
“Play this audio data”
          │
          ▼
Audio driver:
Prepare device-specific transfer
          │
          ▼
Audio controller:
Send samples to hardware
```

---

## Driver mental model: specialized equipment operator

The operating system is a manager.

The driver is a trained operator who knows one type of machine.

The manager says:

> Print this document.

The operator knows:

* How to start the printer
* Which commands it accepts
* How to detect paper errors
* How to report completion

---

# 10.7 Device-Independent and Device-Specific Layers

The OS often separates general I/O behavior from device-specific behavior.

```text
Application
    │
    ▼
General OS I/O interface
    │
    ▼
Device-independent subsystem
    │
    ▼
Device-specific driver
    │
    ▼
Hardware
```

---

## Device-independent layer

This may handle concepts such as:

* Permissions
* File descriptors or handles
* Buffering
* Request queues
* Error conventions
* Naming
* Process waiting and wake-up
* Common read and write operations

---

## Device-specific layer

The driver handles:

* Hardware command formats
* Controller registers
* Device timing
* Device-specific errors
* Interrupt interpretation
* Transfer setup

---

## Why this separation helps

Applications can use a common interface even when hardware changes.

For example, a music player can request audio output without knowing the exact audio-controller model.

---

# 10.8 Devices Are Much Slower Than CPUs

A CPU can execute enormous numbers of instructions while waiting for many devices.

A simplified relative model:

```text
CPU operation:        extremely fast
RAM access:           slower than CPU registers
SSD access:           much slower than RAM
Network response:     variable and often slower
Human keypress:       extremely slow from CPU perspective
```

The exact speeds vary, but the gap is fundamental.

---

## The problem

If a CPU waited idly for every device operation:

```text
CPU starts storage read
        │
        ▼
CPU does nothing
        │
        ▼
Storage finishes
```

most CPU capacity would be wasted.

The OS instead allows:

```text
Thread A waits for device
        │
        ▼
Scheduler runs Thread B
        │
        ▼
Device later signals completion
```

This connects I/O directly to scheduling.

---

# 10.9 Device Registers: Simplified Concept

A controller often exposes special control and status locations to the CPU.

Conceptually, these might represent:

* Command
* Status
* Data
* Error condition
* Transfer address
* Transfer length

```text
Controller interface
├── Command information
├── Status information
├── Data-transfer information
└── Error information
```

The driver reads and writes these controller-visible locations according to the hardware specification.

Applications normally cannot access them freely.

---

## Why access is restricted

Uncontrolled access could:

* Corrupt device state
* Interfere with another process
* Damage stored data
* Leak private information
* Freeze the device
* Bypass OS permissions

The kernel and drivers mediate access.

---

# 10.10 Three Ways Data Can Be Transferred

At a foundational level, device transfer can be understood through three approaches:

1. Programmed I/O
2. Interrupt-driven I/O
3. Direct memory access

Real systems may combine these techniques.

---

# 10.11 Programmed I/O

## What it is

With **programmed I/O**, the CPU actively moves or checks device data using instructions.

A simplified pattern:

```text
CPU asks device for status
       │
       ▼
Ready?
 ┌─────┴─────┐
 │           │
No          Yes
 │           │
Check       Transfer data
again
```

---

## Polling

The CPU may repeatedly check whether the device is ready.

This repeated checking is called **polling**.

```text
Ready? No
Ready? No
Ready? No
Ready? Yes
```

---

## Why polling can be inefficient

If a device takes a long time, the CPU may waste many cycles repeatedly checking it.

---

## When polling can still be useful

Polling may be useful when:

* The expected wait is extremely short
* Events occur very frequently
* Interrupt overhead would be excessive
* Hardware is simple
* Timing must be tightly controlled

Interrupts are not automatically best in every situation.

---

# 10.12 Interrupt-Driven I/O

## What it is

With interrupt-driven I/O:

1. The CPU starts a device operation.
2. The CPU performs other work.
3. The device later generates an interrupt.
4. The kernel handles completion.

```text
CPU starts device
      │
      ▼
CPU runs other work
      │
      ▼
Device completes
      │
      ▼
Interrupt
      │
      ▼
Kernel handles result
```

---

## Why it exists

The CPU does not need to continuously poll the device.

This is effective when device events are relatively infrequent or unpredictable.

---

## Example: keyboard

The CPU does not need to ask millions of times per second:

> Has the user pressed a key yet?

Instead, the keyboard controller signals the system when input occurs.

---

# 10.13 Direct Memory Access

## What it is

**Direct memory access**, or **DMA**, allows a device controller to transfer data between the device and RAM without requiring the CPU to copy each individual unit.

A simplified transfer:

```text
Without DMA:

Device → CPU → RAM
Device → CPU → RAM
Device → CPU → RAM
```

With DMA:

```text
CPU configures transfer
        │
        ▼
Device controller ↔ RAM
        │
        ▼
Interrupt CPU when complete
```

---

## Why DMA exists

Large transfers would consume substantial CPU time if the CPU had to move every byte itself.

DMA allows the CPU to:

* Start the operation
* Perform other work
* Handle completion later

---

## Step-by-step DMA model

1. The kernel prepares a memory region.
2. The driver tells the controller:

   * Where the data should go
   * How much data to transfer
   * Which direction to transfer
3. The controller moves data between the device and RAM.
4. The CPU runs other instructions meanwhile.
5. The controller reports completion.
6. The kernel checks the result.
7. The waiting process may resume.

---

## DMA does not mean the OS is uninvolved

The OS must still:

* Validate the request
* Prepare safe memory
* Configure the device
* Protect processes
* Handle completion
* Process errors
* Manage caches and mappings where necessary

DMA reduces CPU copying work; it does not remove OS control.

---

# 10.14 Step-by-Step: Reading Data with DMA

Suppose a process reads a large file from storage.

## Step 1: Process requests a read

The thread makes a system call.

**Mode:** User → kernel

---

## Step 2: Kernel validates the request

It checks:

* File descriptor
* Permissions
* Requested size
* Destination memory

---

## Step 3: File system determines required storage blocks

The kernel checks whether the data is already cached.

Assume it is not.

---

## Step 4: Driver prepares the transfer

The storage driver creates a request describing:

* Device operation
* Storage location
* Transfer length
* Destination RAM region

---

## Step 5: Controller begins DMA

The controller transfers data from the storage device into RAM.

```text
SSD → controller → RAM
```

The CPU does not copy every byte.

---

## Step 6: Calling thread waits

```text
RUNNING → WAITING
```

The scheduler chooses another ready thread.

---

## Step 7: Device finishes

The controller generates an interrupt.

---

## Step 8: Kernel handles the interrupt

It checks:

* Whether the transfer succeeded
* How much data arrived
* Whether an error occurred

---

## Step 9: Original thread becomes ready

```text
WAITING → READY
```

---

## Step 10: Scheduler resumes the thread

The read system call returns with data or an error.

---

# 10.15 Buffering

## What buffering is

A **buffer** is temporary storage used while data moves between components.

```text
Producer → Buffer → Consumer
```

Buffers help when the producer and consumer operate:

* At different speeds
* In different-sized units
* At irregular times

---

## Example: keyboard input

The user may press several keys before the application processes them.

```text
Keyboard events
A B C D
   │
   ▼
Input buffer
[A][B][C][D]
   │
   ▼
Application reads events
```

Without buffering, input could be lost whenever the application was temporarily busy.

---

## Example: audio output

An application prepares audio data in advance.

```text
Application → Audio buffer → Audio hardware
```

The hardware consumes samples at a steady rate.

If the buffer becomes empty, sound may skip.

This is called a **buffer underrun**.

---

# 10.16 Buffer Underrun and Buffer Overrun

## Buffer underrun

The consumer needs data, but the buffer is empty.

```text
Audio device requests sample
           │
           ▼
Buffer empty
           │
           ▼
Audio gap or distortion
```

---

## Buffer overrun

The producer supplies data faster than the buffer can be emptied.

```text
Network data arrives rapidly
          │
          ▼
Receive buffer full
          │
          ▼
New data may be dropped or delayed
```

---

## Why buffer size matters

A larger buffer can absorb timing differences but may increase latency.

A smaller buffer reduces latency but is easier to empty or overflow.

| Buffer choice | Benefit                         | Cost                             |
| ------------- | ------------------------------- | -------------------------------- |
| Larger        | Handles irregular delays better | More latency and memory          |
| Smaller       | Lower latency                   | Greater underrun or overrun risk |

---

# 10.17 Caching Versus Buffering

These concepts overlap but have different primary purposes.

| Concept    | Main purpose                                    |
| ---------- | ----------------------------------------------- |
| **Buffer** | Temporarily hold data during transfer           |
| **Cache**  | Keep reusable data to avoid repeating slow work |

---

## Buffer example

Data waits before being written to a printer.

---

## Cache example

Recently read disk data remains in RAM in case it is requested again.

---

## One memory region can serve both purposes

For example, a file-system page may temporarily hold data being transferred and later remain available as cached data.

---

# 10.18 Spooling

## What it is

**Spooling** stores complete or partial jobs in a queue for a device that handles work sequentially.

Printing is a classic example.

```text
Application A print job ─┐
Application B print job ─┼──▶ Print queue ──▶ Printer
Application C print job ─┘
```

---

## Why spooling exists

A printer:

* Is slow
* Usually handles one physical job at a time
* Should not be controlled directly by every application

The OS or a print service stores jobs and sends them to the printer in an orderly way.

---

## Spooling versus buffering

A buffer usually smooths a data transfer.

A spool often holds independent jobs waiting for a shared device.

---

# 10.19 Blocking I/O

## What it is

With **blocking I/O**, a thread requests an operation and waits until the operation can make sufficient progress or return a result.

```text
Thread requests data
        │
        ▼
Data unavailable
        │
        ▼
Thread becomes waiting
        │
        ▼
Data arrives
        │
        ▼
Thread becomes ready
```

---

## Why blocking I/O exists

It provides a simple mental model:

> Ask for data, then continue when it arrives.

The scheduler ensures that the waiting thread does not consume CPU time unnecessarily.

---

## Limitation

If an application has only one important thread, blocking on a slow operation may make the interface appear frozen.

---

# 10.20 Non-Blocking I/O

## What it is

With **non-blocking I/O**, the request returns quickly if it cannot proceed immediately.

Conceptually:

```text
Application asks for data
          │
          ▼
Data ready?
   ┌──────┴──────┐
   │             │
  Yes            No
   │             │
Return data   Return “not ready”
```

The application can perform other work and try again later.

---

## Problem non-blocking I/O solves

One thread can manage many operations without waiting on each one individually.

---

## Possible downside

If the application repeatedly checks too aggressively:

```text
Ready? No
Ready? No
Ready? No
Ready? No
```

it recreates inefficient polling.

Operating systems therefore provide event-notification mechanisms.

---

# 10.21 Asynchronous I/O

## What it is

With **asynchronous I/O**, an application starts an operation and is notified later when it completes.

```text
Application starts operation
           │
           ▼
Application continues other work
           │
           ▼
OS completes operation
           │
           ▼
Application receives completion notification
```

---

## Why asynchronous I/O exists

It is useful when an application manages:

* Many network connections
* Several file operations
* User-interface work
* High-throughput storage
* Multiple devices

The thread does not have to block for every operation.

---

## Simplified comparison

| Model        | What happens when data is unavailable?            |
| ------------ | ------------------------------------------------- |
| Blocking     | Calling thread waits                              |
| Non-blocking | Request returns “not ready”                       |
| Asynchronous | Operation continues; completion is reported later |

Exact APIs and behavior vary across operating systems.

---

# 10.22 Event Notification

An OS may let a process wait for any of several events.

For example:

```text
Wake me when:
- Keyboard input arrives
- Network data arrives
- Timer expires
- File operation completes
```

Instead of repeatedly checking each resource, the process waits efficiently.

```text
Process registers interests
          │
          ▼
Process waits
          │
          ▼
One or more events occur
          │
          ▼
Kernel wakes process
```

---

# 10.23 Device Request Queues

Several processes may request the same device.

The OS or driver may maintain a request queue.

```text
Storage request queue:

Request A → Request B → Request C → Request D
```

The device handles requests according to scheduling and hardware capabilities.

---

## Why queues exist

A device may not be able to complete every request immediately.

Queues allow the OS to:

* Preserve pending work
* Prioritize requests
* Combine operations
* Reorder requests
* Apply fairness
* Control overload

---

# 10.24 Device Scheduling

Device scheduling determines the order in which queued I/O requests are processed.

Possible goals include:

* Low latency
* High throughput
* Fairness
* Priority support
* Reduced mechanical or controller overhead
* Meeting deadlines

---

## Example: storage requests

Suppose requests target different storage locations.

A scheduler might process them:

* In arrival order
* By priority
* In an order that improves device efficiency
* By application fairness

Modern SSDs behave differently from mechanical hard drives, so optimal policies differ by device.

---

## Device scheduling versus CPU scheduling

| CPU scheduling                     | Device scheduling                                          |
| ---------------------------------- | ---------------------------------------------------------- |
| Chooses which thread uses a CPU    | Chooses which request uses a device                        |
| Manages runnable execution         | Manages pending I/O                                        |
| Uses thread priorities and runtime | Uses request type, location, priority, and device behavior |

They interact but are not the same mechanism.

---

# 10.25 Device Sharing

Some devices can serve several processes naturally.

Examples:

* Storage
* Network adapter
* Display system
* Audio system

The OS multiplexes their use.

```text
Browser ──────┐
Music player ─┼──▶ OS device management ──▶ Hardware
Editor ───────┘
```

---

## Exclusive devices

Some resources may need temporary exclusive ownership.

Examples might include:

* A specialized measurement device
* Certain cameras
* Some removable devices
* A device undergoing configuration

The kernel or a system service may allow only one process to control it at a time.

---

# 10.26 Multiplexing

**Multiplexing** means combining several logical users onto one physical resource.

Examples:

* Several applications share one CPU through scheduling.
* Several processes share one network adapter.
* Several windows share one display.
* Several audio streams share one speaker system.

```text
Logical streams:
A ─┐
B ─┼──▶ OS combines and coordinates ──▶ One device
C ─┘
```

The OS separates inputs and combines outputs according to policy.

---

# 10.27 Step-by-Step: What Happens When a Key Is Pressed

Suppose the user presses the letter `A` while a text editor is active.

## Stage 1: Physical keypress

The keyboard detects the key’s movement.

**Component:** Keyboard hardware

---

## Stage 2: Keyboard generates low-level information

The device produces a hardware-specific key event.

This does not necessarily directly mean the character `A`.

It may represent a physical key position or device code.

**Component:** Keyboard controller

---

## Stage 3: Information reaches the computer

The device or controller reports the event, commonly through an interrupt or an existing input-transfer mechanism.

**Component:** Hardware controller

---

## Stage 4: CPU enters the kernel

An interrupt causes the CPU to pause its current execution and run a kernel handler.

**Mode:** User or kernel → kernel

---

## Stage 5: Keyboard driver handles device data

The driver:

* Reads the device information
* Acknowledges the event
* Converts device-specific data into an OS-understood form
* Records it in an input buffer

**Component:** Driver

---

## Stage 6: Input subsystem processes the event

The OS considers:

* Which key changed state
* Modifier keys such as Shift
* Keyboard layout
* Repeated key behavior
* Which session should receive the event

A physical key event and a text character are not always identical.

---

## Stage 7: Graphical system determines the target

The system identifies which window or application currently has keyboard focus.

**Component:** Graphical environment or input service

---

## Stage 8: Text editor receives the event

The application may interpret the event as:

* Insert the character `a`
* Insert `A` if Shift is active
* Activate a shortcut
* Ignore it in the current context

**Component:** Application
**Mode:** User mode

---

## Stage 9: Editor changes document state

The document representation in process memory is updated.

---

## Stage 10: Editor requests a display update

The application asks the graphical system to redraw the changed area.

---

## Full flow

```text
Physical keypress
       │
       ▼
Keyboard controller
       │ interrupt
       ▼
Kernel keyboard driver
       │
       ▼
OS input system
       │
       ▼
Focused application
       │
       ▼
Application updates document
       │
       ▼
Graphics system redraws text
```

---

# 10.28 Physical Key Versus Character

A common misconception is that a keyboard directly sends a letter such as `A`.

More accurately:

1. The keyboard reports a physical key event.
2. The OS and input system interpret modifiers and layout.
3. The application interprets the resulting event in context.

The same physical key may produce:

* `a`
* `A`
* Another language character
* A shortcut command
* No visible character

---

# 10.29 Step-by-Step: Displaying a Window

Suppose an application wants to display a button.

## Stage 1: Application describes the interface

The application determines:

* Button position
* Text
* Size
* Current state

**Component:** Application

---

## Stage 2: Application uses a graphics interface

It sends drawing information to a graphics library, window system, or display service.

---

## Stage 3: Graphical system coordinates windows

The system considers:

* Window positions
* Which windows overlap
* Visibility
* Clipping
* Scaling
* Desktop composition

---

## Stage 4: Graphics commands or pixel data are prepared

The work may be performed by:

* CPU
* Graphics processor
* Both

---

## Stage 5: Graphics driver communicates with hardware

The driver submits approved commands and manages graphics memory.

---

## Stage 6: Display hardware reads the completed image

The display controller continuously reads image information and sends the visual signal to the monitor.

---

## Stage 7: Screen shows the button

```text
Application interface description
             │
             ▼
Graphics and window system
             │
             ▼
Graphics driver
             │
             ▼
Graphics hardware and display
```

---

# 10.30 Display Refresh

A display does not usually change only when an application draws once.

The display hardware repeatedly refreshes the visible image.

```text
Frame 1 → Frame 2 → Frame 3 → Frame 4
```

Applications and the graphical system must prepare new frames in time.

If updates are late or poorly synchronized, possible effects include:

* Stuttering
* Tearing
* Delayed input response
* Dropped frames

---

# 10.31 Step-by-Step: Playing Audio

Suppose a music player plays a song.

## Stage 1: Application obtains song data

The data may come from a file or network.

---

## Stage 2: Application decodes compressed data

The CPU converts the encoded format into audio samples.

---

## Stage 3: Application submits samples

Samples are placed into an OS-controlled audio stream or buffer.

---

## Stage 4: Audio service may mix streams

Several applications may produce audio simultaneously.

```text
Music ───┐
Browser ─┼──▶ Audio mixer ──▶ Output device
Game ────┘
```

The OS or audio service combines them.

---

## Stage 5: Driver configures audio hardware

The driver prepares buffers and transfer timing.

---

## Stage 6: Hardware consumes samples steadily

DMA may transfer audio data from RAM to the audio controller.

---

## Stage 7: Device signals progress

Interrupts or timing events indicate that more data is needed.

---

## Stage 8: OS and application refill buffers

If they are too late, the buffer empties and audio skips.

---

# 10.32 Step-by-Step: Receiving Network Data

Suppose a browser receives part of a webpage.

## Stage 1: Network signal reaches the adapter

The hardware receives data from the network medium.

---

## Stage 2: Adapter validates and stores packet data

The adapter may perform low-level checks and place data into a receive buffer.

---

## Stage 3: Adapter transfers data to RAM

DMA is commonly used.

```text
Network adapter → RAM
```

---

## Stage 4: Adapter generates an interrupt or completion event

The kernel is notified that data is available.

---

## Stage 5: Network driver processes the event

The driver identifies received packet data and reports it to the OS networking subsystem.

---

## Stage 6: Networking subsystem interprets protocol information

It determines:

* Which connection the data belongs to
* Whether data is valid
* Whether ordering or retransmission work is needed
* Which process should receive it

---

## Stage 7: Waiting browser thread becomes ready

If the browser was waiting for network data:

```text
WAITING → READY
```

---

## Stage 8: Browser receives data

A system call completes or an asynchronous notification is delivered.

---

## Stage 9: Browser interprets webpage content

The application updates its internal page and may request display changes.

---

# 10.33 Device Discovery

When a computer starts or a device is connected, the OS must determine:

* What device is present
* Which driver should manage it
* What capabilities it supports
* Whether it is trusted or permitted
* Which resources it needs

This is called **device discovery** or **enumeration**.

```text
Device appears
     │
     ▼
Hardware reports identity
     │
     ▼
OS selects driver
     │
     ▼
Driver initializes device
     │
     ▼
Device becomes available
```

---

# 10.34 Device Initialization

A driver may need to:

1. Reset the device.
2. Read capability information.
3. Allocate memory buffers.
4. Configure interrupts.
5. Set transfer modes.
6. Register the device with the OS.
7. Report readiness.

Initialization can fail if:

* The driver is missing
* The driver is incompatible
* Hardware is defective
* Required resources are unavailable
* Security policy blocks the device

---

# 10.35 Hot-Plugging

**Hot-plugging** means connecting or removing a device while the computer remains powered on.

Examples:

* USB drive
* Keyboard
* External display
* Camera
* Headphones

---

## Device connection

```text
Device connected
      │
      ▼
Controller detects change
      │
      ▼
Kernel notified
      │
      ▼
Device identified
      │
      ▼
Driver loaded or activated
      │
      ▼
Applications may use device
```

---

## Device removal

Removal is more difficult because operations may still be active.

The OS must handle:

* Pending reads and writes
* Open descriptors
* Cached data
* Waiting processes
* Driver shutdown
* Error notification

---

# 10.36 Why Safe Removal Matters

Suppose a removable storage device is disconnected while data remains buffered in RAM.

```text
Application writes
      │
      ▼
Kernel cache contains dirty data
      │
      ▼
Device removed before write completes
```

Possible results:

* Lost data
* Incomplete files
* File-system corruption
* Application errors

Safe removal allows the OS to:

* Finish pending writes
* Stop new requests
* Flush caches
* Unmount the file system
* Confirm that removal is safer

---

# 10.37 Device Permissions

Not every process should access every device.

Examples of sensitive devices include:

* Camera
* Microphone
* Raw storage
* Location sensor
* Input devices
* Network interfaces
* Graphics resources

The OS may check:

* User identity
* Application permissions
* Device ownership
* Sandbox rules
* Security policy
* Whether another process has exclusive control

---

## Why device permissions matter

A malicious application with unrestricted access could:

* Record audio
* Capture video
* Read raw storage
* Monitor keyboard input
* Send arbitrary network traffic
* Interfere with another device user

Device access is therefore part of process isolation and security.

---

# 10.38 Device Errors

Devices can fail independently of applications and the kernel.

Possible errors include:

* Device disconnected
* Storage read failure
* Network packet loss
* Printer out of paper
* Camera unavailable
* Audio device busy
* Controller timeout
* Hardware overheating
* Invalid device response

The driver translates hardware-specific failures into OS-level errors.

```text
Hardware-specific error
          │
          ▼
Driver interprets it
          │
          ▼
Kernel reports standard error
          │
          ▼
Application responds
```

---

# 10.39 Timeouts

A device may fail to respond.

The OS should not necessarily wait forever.

A **timeout** is a limit on how long the system waits for an expected event.

```text
Start device operation
        │
        ▼
Wait for completion
   ┌────┴────┐
   │         │
Completes  Timeout expires
   │         │
Success   Error recovery
```

Possible timeout responses include:

* Retry operation
* Reset device
* Report failure
* Disable device
* Terminate a waiting request

---

# 10.40 Retries

Some I/O failures are temporary.

The OS or driver may retry an operation.

Examples:

* Temporary network loss
* Device not immediately ready
* Recoverable storage error
* Busy controller

However, unlimited retries can cause indefinite delays.

The system must balance:

* Reliability
* Latency
* Resource use
* Risk of repeated failure

---

# 10.41 Cancellation

A process may no longer need an I/O request.

For example:

* User closes a window
* Network request is abandoned
* Process exits
* Timeout occurs

Cancellation is not always immediate.

The device may already be transferring data.

```text
Application requests cancellation
          │
          ▼
Has operation started?
     ┌────┴────┐
     │         │
    No        Yes
     │         │
Remove     Stop if possible
from queue or ignore result later
```

The exact behavior depends on the device and OS.

---

# 10.42 What Happens When a Process Exits During I/O?

Suppose a process has pending device requests when it crashes.

The kernel may:

1. Stop the process’s threads.
2. Mark pending requests as canceled where possible.
3. Allow already active hardware transfers to finish safely.
4. Prevent completed data from being delivered to invalid process memory.
5. Release file descriptors and device handles.
6. Notify drivers that ownership has ended.
7. Reclaim buffers and kernel records.

The kernel must protect itself from completion events belonging to processes that no longer exist.

---

# 10.43 I/O Completion and Process Memory

A device must not write into arbitrary process memory after that memory has been released or reassigned.

The OS therefore carefully manages memory used for device transfers.

Possible strategies include:

* Kernel-owned buffers
* Temporarily protected process pages
* Mapped shared buffers
* Copying between kernel and user memory
* DMA mapping restrictions

The foundational idea is:

> Device access to memory must remain valid for the entire transfer.

---

# 10.44 Copying Data Between Kernel and Process

A simple I/O path may involve copying:

```text
Device
   │
   ▼
Kernel buffer
   │
   ▼
Process buffer
```

Copying provides clear isolation, but it uses CPU time and memory bandwidth.

Some I/O techniques reduce copying by sharing or mapping buffers more directly.

---

## Why not eliminate every copy?

Copies can provide:

* Isolation
* Simpler lifetime management
* Validation
* Format conversion
* Protection from device behavior
* Easier buffering

The best design depends on performance and safety requirements.

---

# 10.45 Zero-Copy Concept

**Zero-copy** refers to techniques that reduce unnecessary copying between memory regions.

A simplified ordinary path:

```text
Storage → kernel buffer → application buffer → network buffer
```

A reduced-copy design might allow data to move more directly:

```text
Storage-backed memory → network device
```

The application still controls the operation, but the kernel may avoid copying the data through several intermediate buffers.

This can improve performance for large transfers.

“Zero-copy” does not always mean literally no data movement anywhere. It usually means fewer CPU-managed copies.

---

# 10.46 Device Power Management

Devices consume energy even when idle.

The OS may place devices into lower-power states.

Examples:

* Turn off an unused display
* Suspend a network adapter
* Stop an inactive storage device
* Reduce graphics-processor activity
* Disable unused sensors

---

## Tradeoff

Waking a device takes time.

```text
Active device:
Higher power, fast response

Sleeping device:
Lower power, wake-up delay
```

The OS balances battery life against responsiveness.

---

# 10.47 What Can Go Wrong?

## Driver crash

A privileged driver defect can:

* Corrupt kernel memory
* Freeze the device
* Crash the whole system
* Damage data

---

## Device hangs

The device stops responding.

The OS may:

* Wait until timeout
* Retry
* Reset it
* Disable it
* Report failure

---

## Buffer overflow

More data arrives than the available buffer can hold.

Possible result:

* Dropped data
* Delayed processing
* Application failure

---

## Buffer underrun

A device consumes output faster than software supplies it.

Possible result:

* Audio skipping
* Video stuttering
* Transmission gaps

---

## Interrupt storm

A device generates excessive interrupts.

Possible result:

* High CPU use
* Poor responsiveness
* Reduced application progress

---

## Lost interrupt or completion event

A waiting thread may never be awakened.

The operation may eventually time out.

---

## Incorrect DMA setup

A device may write to the wrong memory.

Possible consequences:

* Data corruption
* Kernel failure
* Security breach

Hardware and OS protections are used to reduce this risk.

---

## Device removed during use

Pending operations may fail.

Data may be lost if cached writes have not completed.

---

## Request starvation

Low-priority I/O requests may wait too long because higher-priority requests continuously arrive.

---

## Excessive buffering

Large buffers may:

* Consume memory
* Increase latency
* Hide backpressure
* Delay error detection

---

## Insufficient buffering

Small buffers may:

* Overflow
* Underrun
* Cause frequent interrupts
* Reduce throughput

---

## Slow consumer

An application cannot process input as quickly as it arrives.

The OS may:

* Queue data
* Apply backpressure
* Drop data
* Disconnect the source
* Report an error

---

# 10.48 Backpressure

**Backpressure** is a mechanism that slows or stops a producer when the consumer cannot keep up.

```text
Producer sends rapidly
        │
        ▼
Buffer nearly full
        │
        ▼
System tells producer to slow down
```

Examples include:

* Network flow control
* Full output queues
* Storage write throttling
* Pipes that block writers

Backpressure prevents unlimited buffer growth.

---

# 10.49 Common Misconceptions

## Misconception: “The CPU moves every byte between devices and RAM”

Not necessarily.

DMA allows controllers to transfer large amounts of data directly between devices and memory after CPU setup.

---

## Misconception: “An interrupt contains all the device data”

An interrupt commonly signals that an event occurred.

The data itself may already be in RAM, a controller buffer, or a device queue.

---

## Misconception: “A driver is part of the physical device”

A driver is software.

The controller and device are hardware.

---

## Misconception: “Every device has a completely different application interface”

The OS often provides common abstractions such as:

* Read
* Write
* File descriptors
* Event notifications
* Device handles

Drivers hide much of the variation.

---

## Misconception: “Blocking I/O freezes the whole computer”

Only the calling thread normally waits.

Other threads can run.

---

## Misconception: “Non-blocking I/O is always faster”

It can improve concurrency, but it also adds complexity and may waste CPU time if implemented as aggressive polling.

---

## Misconception: “Asynchronous I/O means the operation is instantaneous”

It means the application does not have to wait synchronously.

The device operation still takes time.

---

## Misconception: “A successful write means the physical device has completed it”

The OS or device may have accepted the data into a buffer while physical output remains pending.

---

## Misconception: “Larger buffers are always better”

Larger buffers reduce some overflow risks but increase memory use and latency.

---

## Misconception: “Interrupts completely replace polling”

Many high-performance systems combine both.

Interrupts may signal initial work, after which software polls a queue efficiently.

---

## Misconception: “A process can safely disconnect a device after closing one file”

Other processes, cached writes, or mounted file systems may still be using the device.

---

## Misconception: “Devices operate entirely under kernel control at every instant”

Once configured, many controllers perform substantial work independently and notify the kernel later.

---

# 10.50 Real-World Analogy: Restaurant Kitchen Orders

Imagine a busy restaurant.

## Applications

Customers placing orders.

## OS I/O subsystem

The restaurant’s order-management system.

## Driver

A specialist who knows how to operate a particular machine, such as an industrial oven.

## Device

The oven, mixer, or coffee machine.

## Request queue

Orders waiting for the machine.

## Buffer

A counter holding prepared ingredients or finished dishes temporarily.

## Interrupt

A timer or bell indicating that cooking has finished.

## DMA

The machine moves food through an automated conveyor while staff perform other work.

## Blocking request

A waiter waits at the kitchen until an order is ready.

## Asynchronous request

The waiter submits the order and is notified later.

## Spooling

Print orders are queued and processed one at a time by a single receipt printer.

---

# 10.51 Connection to Earlier Concepts

## Connection to hardware and the kernel

The kernel controls access to devices.

Drivers translate general requests into hardware-specific commands.

---

## Connection to user and kernel mode

Applications request I/O from user mode.

The kernel validates the request and interacts with drivers in a privileged environment.

---

## Connection to system calls

Processes use system calls to:

* Read
* Write
* Configure
* Wait for events
* Open devices
* Close resources

---

## Connection to interrupts

Devices notify the kernel about:

* Completed operations
* New input
* Errors
* State changes

---

## Connection to scheduling

A thread waiting for I/O moves:

```text
RUNNING → WAITING
```

When the device completes:

```text
WAITING → READY
```

---

## Connection to context switching

The scheduler can run another thread while the device works.

A completion interrupt may lead to another scheduling decision.

---

## Connection to memory

I/O uses memory for:

* Buffers
* Caches
* DMA regions
* Device queues
* Application data

The OS must protect this memory during transfers.

---

## Connection to files

File reads and writes eventually become storage-device operations unless the requested data is satisfied from cache.

---

## Connection to process isolation

The OS prevents one process from:

* Controlling another process’s device operations
* Reading another process’s input
* Using unauthorized devices
* Corrupting memory through DMA

---

# 10.52 Simplified Model Versus Technical Reality

## Simplified mental model

```text
Application makes I/O request
          │
          ▼
Kernel validates it
          │
          ▼
Driver controls device
          │
          ▼
Device performs operation
          │
          ▼
Interrupt reports completion
          │
          ▼
Waiting application resumes
```

This is the model to retain.

---

## More exact reality

Modern I/O systems may include:

* Several driver layers
* Firmware
* Multiple request queues
* DMA engines
* Dedicated device processors
* Shared memory
* Memory-mapped device control
* Interrupt batching
* Polling
* Virtual devices
* Hypervisor mediation
* User-space drivers
* Remote devices
* Hardware command queues
* Error-recovery services

Some operations complete entirely from cache.

Some devices generate many completion events together.

Some drivers run partly outside the kernel.

The core principle remains:

> The operating system converts application-level I/O requests into safe, scheduled, device-specific operations while allowing the CPU to continue other work.

---

# 10.53 Core Mental Model

## General I/O path

```text
Application
    │ system call or OS interface
    ▼
Kernel I/O subsystem
    │ validate and queue
    ▼
Device driver
    │ device-specific command
    ▼
Controller and device
    │ perform operation
    ▼
Interrupt or completion event
    │
    ▼
Kernel wakes or notifies application
```

---

## Blocking I/O state flow

```text
Thread RUNNING
      │ requests slow I/O
      ▼
Thread WAITING
      │ device completes
      ▼
Thread READY
      │ scheduler selects it
      ▼
Thread RUNNING
```

---

## DMA model

```text
CPU configures transfer
        │
        ▼
Device controller ↔ RAM
        │
        ▼
Interrupt on completion
```

---

## Final distinctions

| Concept                  | Essential meaning                                        |
| ------------------------ | -------------------------------------------------------- |
| **I/O**                  | Movement of information between the computer and devices |
| **Driver**               | Software controlling a particular hardware type          |
| **Controller**           | Hardware managing device communication                   |
| **Interrupt-driven I/O** | Device notifies CPU when attention is needed             |
| **DMA**                  | Controller transfers data between device and RAM         |
| **Buffer**               | Temporary transfer storage                               |
| **Cache**                | Retained data for faster reuse                           |
| **Spooling**             | Queuing jobs for a sequential shared device              |
| **Blocking I/O**         | Calling thread waits                                     |
| **Non-blocking I/O**     | Request returns if it cannot proceed                     |
| **Asynchronous I/O**     | Completion is reported later                             |
| **Backpressure**         | Slows producers when consumers cannot keep up            |

The next section introduces **concurrency and synchronization**—how multiple threads and processes coordinate when they share data or resources.

# Learning Check

Do not look for answers yet.

## Conceptual questions

1. What are the different roles of a device, hardware controller, and device driver?
2. What problem does DMA solve, and what responsibilities still remain with the CPU and kernel?
3. What is the difference between blocking, non-blocking, and asynchronous I/O?

## Cause-and-effect questions

4. Why can a thread waiting for a storage device stop consuming CPU time without stopping the entire process or system?
5. Why can increasing an audio buffer reduce skipping while also increasing latency?

## Misconception question

6. A student says, “When a device raises an interrupt, the interrupt itself carries all the requested data directly into the application.” What is wrong with this explanation?

## Scenario-based question

7. A music player reads song data from storage and plays it through speakers while a browser is also running. Explain the roles of system calls, file cache, storage driver, DMA, waiting and ready states, interrupts, audio buffers, audio driver, scheduling, and the CPU.

# 11. Concurrency and Synchronization

Modern systems perform many activities during overlapping periods:

* A browser downloads data while updating its interface.
* A music player decodes audio while the audio device consumes earlier samples.
* Multiple processes access the same file.
* Several threads work on different parts of one task.
* The kernel handles device events while applications continue running.

This overlapping progress is called **concurrency**.

When concurrent activities share data or resources, they must coordinate. That coordination is called **synchronization**.

```text
Concurrency:
Multiple activities are in progress.

Synchronization:
Rules and mechanisms coordinate those activities.
```

---

# 11.1 What Is Concurrency?

## What it is

**Concurrency** means that multiple activities make progress during overlapping periods of time.

They do not necessarily execute at the exact same physical instant.

On one CPU core:

```text
Time ─────────────────────────────────────────▶

Core 1:
[Thread A][Thread B][Thread A][Thread C]
```

The threads are concurrent because all three remain active and make progress through switching.

On several CPU cores:

```text
Time ─────────────────────────────────────────▶

Core 1: [Thread A────────────────────────]
Core 2: [Thread B────────────────────────]
Core 3: [Thread C────────────────────────]
```

The threads are both concurrent and parallel.

---

## Concurrency versus parallelism

| Concept         | Meaning                                                    |
| --------------- | ---------------------------------------------------------- |
| **Concurrency** | Multiple activities make progress during overlapping time  |
| **Parallelism** | Multiple activities execute physically at the same instant |

Parallelism requires multiple execution resources.

Concurrency can exist on one core through scheduling and context switching.

---

## Why concurrency exists

Concurrency helps systems:

* Keep applications responsive
* Overlap computation with I/O
* Use multiple CPU cores
* Serve several users or requests
* Separate independent activities
* Handle unpredictable events
* Improve throughput

---

## Example: text editor

A text editor may need to:

* Respond to keyboard input
* Save a file
* Check spelling
* Update the display
* Synchronize with cloud storage

These activities should not necessarily stop one another.

```text
Editor process
├── Interface activity
├── File-saving activity
├── Spell-check activity
└── Synchronization activity
```

Concurrency allows them to make progress during overlapping periods.

---

# 11.2 Why Concurrency Is Difficult

Sequential execution is easier to reason about.

Suppose one thread performs:

```text
Step A
Step B
Step C
Step D
```

The order is clear.

With two concurrent threads, their steps may be interleaved in many possible ways.

```text
Thread A: A1 A2 A3
Thread B: B1 B2 B3
```

Possible execution:

```text
A1 A2 B1 B2 A3 B3
```

Another possible execution:

```text
B1 A1 B2 A2 B3 A3
```

Another:

```text
A1 B1 A2 B2 A3 B3
```

The scheduler, timing, interrupts, CPU cores, and device events can all influence the order.

---

## The central difficulty

If threads are independent, the order may not matter.

If they share data, different execution orders can produce different results.

```text
Thread A ─┐
          ├──▶ Shared data
Thread B ─┘
```

The program must remain correct under all permitted execution orders, not just the order observed during one test.

---

# 11.3 Mental Model: Several Workers Sharing a Whiteboard

Imagine two workers updating one whiteboard.

The board says:

```text
Available tickets: 1
```

Both workers want to reserve the final ticket.

Worker A does:

1. Read the number.
2. Check whether it is greater than zero.
3. Subtract one.
4. Write the result.

Worker B performs the same steps.

A dangerous ordering is:

```text
Worker A reads 1.
Worker B reads 1.
Worker A decides a ticket is available.
Worker B decides a ticket is available.
Worker A writes 0.
Worker B writes 0.
```

Both workers believe they received the final ticket.

The final board still says `0`, so the error is not obvious from the final number alone.

The problem occurred because both workers acted on shared information without coordination.

---

# 11.4 Shared State

## What it is

**Shared state** is information accessible to more than one concurrent activity.

Examples include:

* A variable used by several threads
* A shared queue
* A file modified by several processes
* A database record
* An audio buffer
* A kernel device-request queue
* Shared-memory pages
* A process-wide heap object

```text
Thread A ─────┐
Thread B ─────┼──▶ Shared state
Thread C ─────┘
```

---

## Why shared state exists

Shared state lets concurrent activities cooperate.

For example:

* One thread produces data.
* Another thread consumes it.
* One thread updates a document.
* Another saves that document.
* Several server threads update shared statistics.
* The kernel and a driver coordinate I/O requests.

Sharing can be efficient, but it creates dependency.

---

## Private state versus shared state

| State type             | Accessible by                          | Coordination need |
| ---------------------- | -------------------------------------- | ----------------- |
| Thread-local state     | One thread                             | Usually low       |
| Process-shared state   | Threads in one process                 | Often required    |
| Shared-memory state    | Selected processes                     | Required          |
| File or database state | Potentially many processes or machines | Required          |

The less state is shared, the easier concurrency is to reason about.

---

# 11.5 The Read–Modify–Write Pattern

Many operations that look like one action actually contain several steps.

Consider:

> Increase a shared counter by one.

Conceptually, the CPU may need to:

1. Read the current value.
2. Calculate the new value.
3. Write the new value.

```text
Read value
    │
    ▼
Add one
    │
    ▼
Write result
```

This is a **read–modify–write** sequence.

---

## Interference example

Suppose the counter begins at `10`.

Thread A and Thread B both increase it.

Expected final value:

```text
12
```

Possible interleaving:

```text
Thread A reads 10.
Thread B reads 10.
Thread A calculates 11.
Thread B calculates 11.
Thread A writes 11.
Thread B writes 11.
```

Actual final value:

```text
11
```

One update was lost.

---

## Important correction

A high-level action such as:

> Increase the counter.

is not automatically indivisible.

It may involve multiple CPU and memory operations unless special synchronization is used.

---

# 11.6 Atomicity

## What it is

An operation is **atomic** when it appears to occur as one indivisible action relative to other concurrent activities.

Other threads cannot observe it as half-completed.

```text
Before operation: State A
Atomic change
After operation:  State B
```

No other thread observes an intermediate state.

---

## Why atomicity exists

Atomicity prevents harmful interleaving.

Consider transferring money conceptually:

1. Subtract from Account A.
2. Add to Account B.

If another activity observes the system after step 1 but before step 2, the total money temporarily appears incorrect.

A synchronized operation should often make the transfer appear as one logical change.

---

## Atomic does not mean instantaneous

An atomic operation may take time internally.

The important property is:

> Other concurrent participants cannot observe or interfere with an incomplete intermediate state.

---

## Hardware atomic operations

CPUs provide some special operations that can update small pieces of memory atomically.

Conceptually, hardware may support:

```text
Read value
Check condition
Write new value
```

as one protected operation relative to other CPU cores.

Operating systems and synchronization libraries use these hardware capabilities to build locks and other mechanisms.

---

## Not everything can be one hardware atomic operation

A complex update may involve:

* Several values
* Several memory locations
* File operations
* Network communication
* Multiple data structures

For these, synchronization must create a larger protected region.

---

# 11.7 Critical Sections

## What it is

A **critical section** is a region of execution that accesses shared state and must not be executed concurrently in an unsafe way.

Example:

```text
Critical section:
1. Read available balance.
2. Check requested amount.
3. Update balance.
4. Record transaction.
```

If several threads perform these steps on the same account simultaneously, they may interfere.

---

## Why critical sections exist

The program needs to identify:

> Which steps must behave as one coordinated operation?

Once identified, a synchronization mechanism can protect that region.

---

## Mental model: one-person records room

Imagine a records room containing sensitive documents.

Only one authorized person may enter at a time.

```text
Worker A waits ─┐
Worker B waits ─┼──▶ Locked records room
Worker C inside ┘
```

The person inside performs several related actions without others rearranging the records midway.

The room represents a critical section.

The door lock represents a synchronization mechanism.

---

# 11.8 Mutual Exclusion

## What it is

**Mutual exclusion** means that only one participating thread may execute a protected critical section at a time.

```text
Thread A inside critical section
Thread B must wait
Thread C must wait
```

The abbreviation **mutex** comes from “mutual exclusion.”

---

## The problem mutual exclusion solves

Suppose several threads modify one linked data structure, queue, or account record.

Without mutual exclusion:

* One thread may observe partial changes.
* Updates may be lost.
* Internal structure may become inconsistent.
* The application may crash.

With mutual exclusion:

```text
Thread A enters
Thread A completes shared update
Thread A leaves
Thread B enters
```

The shared operation is serialized.

---

## Serialization

To **serialize** operations means to force them into a safe one-at-a-time order.

Concurrency still exists elsewhere.

```text
Thread A: private work → protected update → private work
Thread B: private work ─────wait───────→ protected update
```

Only the critical section is serialized.

---

# 11.9 Locks and Mutexes

## What a lock is

A **lock** is a synchronization object used to control access to a shared resource or critical section.

A mutex has two main conceptual states:

```text
UNLOCKED
LOCKED
```

---

## Basic lock behavior

Before entering a critical section, a thread tries to acquire the lock.

```text
Acquire lock
     │
     ▼
Enter critical section
     │
     ▼
Modify shared state
     │
     ▼
Release lock
```

If another thread already owns the lock, the requesting thread must wait or retry.

---

## Step-by-step example

Suppose two threads update a shared account balance.

### Thread A

1. Acquires the account lock.
2. Reads the balance.
3. Applies its update.
4. Writes the result.
5. Releases the lock.

### Thread B

If it tries to acquire the lock while A owns it:

```text
Thread B:
RUNNING → WAITING
```

After Thread A releases the lock:

```text
Thread B:
WAITING → READY
```

The scheduler eventually runs Thread B.

---

## State diagram

```text
                 acquire succeeds
        ┌───────────────────────────┐
        │                           ▼
    LOCK AVAILABLE              LOCK OWNED
        ▲                           │
        │                           │ release
        └───────────────────────────┘
```

For an individual thread:

```text
Try to acquire
      │
  ┌───┴────┐
  │        │
Free      Busy
  │        │
Enter    Wait
critical   │
section    │ lock released
           ▼
         Ready
```

---

# 11.10 Lock Ownership

A mutex commonly has an owner: the thread that successfully acquired it.

Only the owner should release it.

```text
Lock owner: Thread A

Thread A may release.
Thread B must not release it.
```

This ownership rule helps preserve clear synchronization behavior.

Other synchronization objects may have different rules.

---

# 11.11 Lock Scope

A lock should correspond clearly to the state it protects.

Example:

```text
Document lock protects:
- Document text
- Modification counter
- Related indexing state
```

A thread accessing that state must follow the same locking rule.

---

## Inconsistent locking

Suppose:

* Thread A uses a lock before updating a shared list.
* Thread B modifies the list without acquiring the lock.

The lock no longer provides protection.

```text
Thread A → lock → shared list
Thread B ─────────▶ shared list
```

Synchronization works only when every participant follows the protocol.

---

# 11.12 Lock Granularity

**Lock granularity** means how much state one lock protects.

---

## Coarse-grained lock

One lock protects a large amount of state.

```text
One application lock protects:
- User table
- Message queue
- Cache
- Statistics
```

### Benefits

* Simpler reasoning
* Fewer locks
* Lower risk of inconsistent locking

### Costs

* Unrelated work may block
* Less concurrency
* One slow critical section delays many threads

---

## Fine-grained locks

Different locks protect smaller pieces.

```text
User-table lock
Message-queue lock
Cache lock
Statistics lock
```

### Benefits

* More threads can work concurrently
* Less unnecessary blocking

### Costs

* More complex rules
* Greater risk of acquiring locks incorrectly
* Greater deadlock risk
* More synchronization overhead

---

## Tradeoff

| Lock style     | Simplicity | Concurrency |
| -------------- | ---------: | ----------: |
| Coarse-grained |     Higher |       Lower |
| Fine-grained   |      Lower |      Higher |

The best design balances correctness, complexity, and performance.

---

# 11.13 Blocking Locks

With a blocking mutex, a thread that cannot acquire the lock is placed into a waiting state.

```text
Thread tries lock
      │
      ▼
Lock busy
      │
      ▼
Thread WAITING
      │
      ▼
Owner releases lock
      │
      ▼
Thread READY
```

The scheduler runs another thread meanwhile.

---

## Why blocking is useful

If the wait may be long, repeatedly checking the lock would waste CPU time.

Blocking lets the kernel use the CPU for other work.

---

# 11.14 Spinlocks

A **spinlock** causes a waiting thread to repeatedly check whether the lock has become available.

```text
Lock free?
No
Lock free?
No
Lock free?
Yes → acquire
```

The thread remains running and consumes CPU time while waiting.

---

## Why would spinning ever be useful?

Blocking and waking a thread has costs:

* Entering the kernel
* Changing thread state
* Context switching
* Later rescheduling

If the lock will be held only for an extremely short time, spinning briefly may cost less than sleeping and waking.

---

## When spinning is harmful

If the lock is held for a long time:

* CPU time is wasted.
* Other runnable work is delayed.
* Power consumption increases.

---

## Simplified comparison

| Blocking mutex                    | Spinlock                                  |
| --------------------------------- | ----------------------------------------- |
| Waiting thread sleeps             | Waiting thread repeatedly checks          |
| Good for potentially longer waits | Useful only for very short expected waits |
| Requires scheduler involvement    | Consumes CPU while waiting                |
| Common in applications            | Common in selected low-level contexts     |

Exact implementations may combine spinning and blocking.

---

# 11.15 Contention

**Contention** occurs when several threads compete for the same resource or lock.

```text
Thread A ─┐
Thread B ─┼──▶ One lock
Thread C ─┘
```

High contention means threads spend substantial time waiting rather than doing useful work.

---

## Effects of contention

* Reduced parallelism
* Increased scheduling activity
* Longer response times
* Poor CPU utilization
* Unpredictable delays
* Lock queues

---

## Example

Suppose eight threads update one global counter under one lock.

```text
Thread 1: acquire → update → release
Thread 2: wait
Thread 3: wait
...
Thread 8: wait
```

Although eight cores exist, only one thread performs the protected update at a time.

The lock becomes a bottleneck.

---

# 11.16 Synchronization Overhead

Synchronization itself consumes resources.

Possible costs include:

* Atomic hardware instructions
* Cache coordination between CPU cores
* Kernel operations
* Context switches
* Waiting
* Wake-up delays
* Lock bookkeeping

A correctly synchronized program may still perform poorly if it protects too much work with heavily contended locks.

---

## Important principle

The goal is not:

> Add as many locks as possible.

The goal is:

> Use enough coordination to guarantee correctness while preserving useful concurrency.

---

# 11.17 Memory Visibility

A thread may update shared memory, but another CPU core may not instantly observe the change in the simple order expected by the programmer.

Modern processors and compilers optimize memory operations using:

* CPU caches
* Write buffers
* Instruction reordering
* Compiler reordering

---

## Simplified problem

Thread A performs:

```text
1. Store new data.
2. Set ready flag.
```

Thread B performs:

```text
1. Check ready flag.
2. Read new data.
```

Without proper synchronization, Thread B might observe the ready flag without reliably observing the associated data update in the expected way.

---

## Why synchronization helps

Synchronization operations establish rules about:

* Which updates become visible
* In what order they are observed
* When another thread may safely proceed

A mutex does more than prevent simultaneous entry. It also establishes memory-ordering guarantees.

---

## Simplified mental model

Releasing a lock says:

> My protected updates are complete.

Acquiring the same lock says:

> I should now observe the completed protected updates.

```text
Thread A:
update shared data
release lock
        │
        ▼
Thread B:
acquire lock
observe updated data
```

Exact memory-order rules are language- and hardware-specific, but the conceptual point is essential.

---

# 11.18 Synchronization Is About Ordering

Synchronization does not only mean one-at-a-time access.

It can also express:

* Do this after another event.
* Wait until data exists.
* Allow at most N simultaneous users.
* Notify another thread that state changed.
* Coordinate several phases of work.

This requires mechanisms beyond simple mutexes.

---

# 11.19 Semaphores

## What a semaphore is

A **semaphore** is a synchronization object containing a counter representing available permits or resources.

Conceptually:

```text
Semaphore count = 3
```

This means up to three participants may proceed.

---

## Basic operations

A thread requests a permit.

### If count is greater than zero

1. Reduce the count.
2. Continue.

### If count is zero

The thread waits.

When a participant returns a permit:

1. Increase the count.
2. Wake a waiting thread if appropriate.

---

## Mental model: parking permits

A parking area has three spaces.

```text
Available permits: 3
```

Each arriving car takes one permit:

```text
3 → 2 → 1 → 0
```

A fourth car must wait.

When a car leaves:

```text
0 → 1
```

One waiting car may enter.

---

## Why semaphores exist

They can limit access to a fixed-capacity resource, such as:

* A pool of database connections
* A set of worker slots
* A limited number of device buffers
* A maximum number of simultaneous tasks

---

## Binary semaphore

A semaphore whose count is limited to zero or one can resemble a mutex.

However, mutex ownership rules and semaphore signaling rules often differ.

Do not treat them as identical in all systems.

---

# 11.20 Mutex Versus Semaphore

| Property      | Mutex                     | Semaphore                                |
| ------------- | ------------------------- | ---------------------------------------- |
| Main purpose  | Exclusive ownership       | Count available permits or signal events |
| Typical value | Locked or unlocked        | Integer count                            |
| Owner         | Usually one owning thread | Often no strict owner                    |
| Release       | Usually by owner          | May be signaled by another thread        |
| Common use    | Protect shared state      | Limit capacity or coordinate events      |

---

# 11.21 Condition Variables

## What they are

A **condition variable** allows threads to wait until some condition involving shared state becomes true.

Examples:

* A queue is no longer empty.
* A buffer has free space.
* A job has completed.
* Configuration has finished loading.

---

## The problem condition variables solve

Suppose a consumer thread needs an item from a shared queue.

It could repeatedly check:

```text
Queue empty?
Yes
Queue empty?
Yes
Queue empty?
Yes
```

This wastes CPU time.

Instead, it can wait.

```text
Queue empty
     │
     ▼
Consumer sleeps
     │
Producer adds item and signals
     ▼
Consumer wakes and checks again
```

---

## Condition variable and mutex work together

The shared condition is normally checked while holding a mutex.

Conceptual pattern:

```text
Acquire queue lock
        │
        ▼
Is queue non-empty?
   ┌────┴────┐
   │         │
  Yes        No
   │         │
Use item   Wait on condition
             │
             ▼
         Release lock and sleep
```

When the thread wakes:

```text
Reacquire lock
      │
      ▼
Check condition again
```

---

## Why release the mutex while waiting?

If the waiting consumer kept the queue lock:

```text
Consumer holds lock while waiting
          │
          ▼
Producer cannot acquire lock
          │
          ▼
Producer cannot add item
```

The condition could never become true.

The waiting operation therefore releases the mutex as part of sleeping and reacquires it before returning.

---

# 11.22 Why the Condition Must Be Rechecked

A wake-up does not always guarantee that the desired state remains true.

Possible reasons:

* Another consumer took the item first.
* Several threads woke together.
* The condition changed again.
* The OS or runtime allows wake-ups without a specific signal.

Therefore:

```text
Wake up
   │
   ▼
Reacquire lock
   │
   ▼
Check condition again
```

The notification means:

> Something may have changed.

It does not always mean:

> Your desired resource is certainly available.

---

# 11.23 Producer–Consumer Problem

The **producer–consumer** pattern is a foundational synchronization example.

* Producers create items.
* Consumers process items.
* A shared buffer or queue connects them.

```text
Producer threads
      │
      ▼
Shared queue
      │
      ▼
Consumer threads
```

---

## Example

A network thread receives requests and places them in a queue.

Worker threads remove requests and process them.

```text
Network input
      │
      ▼
Request queue
      │
      ├──▶ Worker A
      ├──▶ Worker B
      └──▶ Worker C
```

---

## Synchronization requirements

The design must ensure:

* Two producers do not corrupt the queue.
* Two consumers do not remove the same item.
* Consumers wait when the queue is empty.
* Producers may wait when the queue is full.
* Updates become visible across threads.

---

## Components

A typical conceptual solution uses:

* A mutex to protect queue structure
* A condition indicating “queue not empty”
* Possibly a condition indicating “queue not full”

---

# 11.24 Step-by-Step: Producer Adds an Item

Assume the queue has capacity.

## Step 1: Producer creates an item

This happens in private thread memory.

No queue synchronization is required yet.

---

## Step 2: Producer acquires the queue mutex

If another thread is modifying the queue, the producer waits.

---

## Step 3: Producer checks queue capacity

If full, it may wait on a “not full” condition.

Waiting releases the mutex so consumers can remove items.

---

## Step 4: Producer inserts the item

The queue data structure changes while protected.

---

## Step 5: Producer signals “not empty”

A waiting consumer may now be able to proceed.

---

## Step 6: Producer releases the mutex

Other threads may access the queue.

```text
Create item privately
        │
        ▼
Acquire queue lock
        │
        ▼
Wait if full
        │
        ▼
Insert item
        │
        ▼
Signal consumers
        │
        ▼
Release lock
```

---

# 11.25 Step-by-Step: Consumer Removes an Item

## Step 1: Consumer acquires the queue mutex

---

## Step 2: Consumer checks whether the queue is empty

### Not empty

Continue.

### Empty

Wait on the “not empty” condition.

The wait releases the lock.

---

## Step 3: Producer adds an item and signals

The consumer becomes eligible to wake.

---

## Step 4: Consumer reacquires the mutex

---

## Step 5: Consumer checks the queue again

Another consumer may already have taken the item.

---

## Step 6: Consumer removes one item

The queue is modified while protected.

---

## Step 7: Consumer signals “not full”

A waiting producer may proceed.

---

## Step 8: Consumer releases the mutex

---

## Step 9: Consumer processes the item privately

The expensive work should often occur outside the queue lock.

```text
Acquire queue lock
        │
        ▼
Wait while empty
        │
        ▼
Remove item
        │
        ▼
Signal producers
        │
        ▼
Release lock
        │
        ▼
Process item without lock
```

---

# 11.26 Why Critical Sections Should Be Short

Suppose a thread acquires a lock and then performs:

* Slow file I/O
* Network access
* A long calculation
* User interaction

Other threads may wait the entire time.

```text
Thread A holds lock
      │
      ├── waits for network
      ├── performs calculation
      └── eventually releases
               
Thread B: waits
Thread C: waits
Thread D: waits
```

---

## Better principle

Keep only the necessary shared-state work inside the critical section.

```text
Acquire lock
Copy or update shared state
Release lock
Perform slow private work
```

This reduces contention.

---

## But do not release too early

Releasing a lock before the shared state is consistent can expose partial updates.

The correct boundary must preserve invariants.

---

# 11.27 Invariants

An **invariant** is a rule that should always be true when shared state is visible outside a protected update.

Example queue invariants:

* Item count is not negative.
* Item count does not exceed capacity.
* Front and back positions are valid.
* Stored count matches actual items.

A critical section may temporarily break an invariant internally, but it must restore it before releasing the lock.

```text
Acquire lock
      │
Temporarily modify several fields
      │
Restore consistent state
      │
Release lock
```

---

## Why invariants matter

Locks protect code regions, but the real goal is to protect relationships among data.

A good synchronization design identifies:

1. Which state is shared.
2. Which invariants must hold.
3. Which lock protects those invariants.
4. Which operations must occur together.

---

# 11.28 Read–Write Locks

## What they are

A **read–write lock** distinguishes between:

* Readers that only inspect shared state
* Writers that modify shared state

Multiple readers may enter together when no writer is active.

A writer requires exclusive access.

```text
Allowed:
Reader A + Reader B + Reader C

Not allowed:
Writer + any reader
Writer A + Writer B
```

---

## Why read–write locks exist

Some workloads have:

* Many reads
* Few writes

Allowing readers to proceed together can increase concurrency.

---

## Tradeoffs

Read–write locks add complexity.

Possible problems include:

* Writers waiting too long
* Readers waiting behind frequent writers
* Higher synchronization overhead
* More complicated fairness policy

A simple mutex may outperform a read–write lock when critical sections are short or writes are frequent.

---

# 11.29 Barriers

## What a barrier is

A **barrier** makes a group of threads wait until all members reach the same phase boundary.

```text
Thread A reaches barrier ─────waits
Thread B reaches barrier ─────waits
Thread C reaches barrier ─────last arrival
                               │
                               ▼
All continue to next phase
```

---

## Why barriers exist

Some parallel work proceeds in stages.

Example:

1. Each thread processes one section of an image.
2. All sections must finish.
3. Threads then begin a second operation using the complete result.

The barrier ensures that no thread begins phase 2 before all threads finish phase 1.

---

# 11.30 Events and Notifications

An event mechanism lets one thread indicate that something happened.

Examples:

* Initialization completed
* Shutdown requested
* Data became available
* A worker finished
* A device state changed

```text
Thread A performs work
       │
       ▼
Signals event
       │
       ▼
Thread B wakes or proceeds
```

An event may be:

* One-time
* Reusable
* Automatically reset
* Manually reset

Exact behavior depends on the OS or programming environment.

---

# 11.31 Message Passing

Shared memory is not the only way to coordinate.

Processes or threads can exchange messages.

```text
Sender
  │
  │ message
  ▼
OS-managed channel or queue
  │
  ▼
Receiver
```

---

## Why message passing exists

Message passing can reduce direct shared-state access.

The sender prepares a message, and the receiver handles it independently.

Examples include:

* Pipes
* Process mailboxes
* Event queues
* Network connections
* Request-response channels

---

## Benefits

* Clear ownership transfer
* Better process isolation
* Easier distribution across machines
* Less direct shared-memory corruption

---

## Costs

* Data copying or serialization
* Kernel involvement
* Queue management
* Communication delay
* More complex failure handling

---

# 11.32 Shared Memory Versus Message Passing

| Shared memory                     | Message passing                         |
| --------------------------------- | --------------------------------------- |
| Participants access common memory | Participants exchange explicit messages |
| Potentially very fast             | Often more controlled                   |
| Requires careful synchronization  | Queue or channel provides some ordering |
| Easy to corrupt shared state      | Better isolation                        |
| Usually limited to one machine    | Can extend across networks              |

Neither model is always superior.

Many systems use both.

---

# 11.33 Thread-Local Storage

**Thread-local storage** gives each thread a separate instance of certain data.

```text
Thread A → private value A
Thread B → private value B
Thread C → private value C
```

Even though the threads share a process, thread-local values are not shared.

---

## Why it exists

It reduces coordination needs for data such as:

* Per-thread temporary state
* Error information
* Caches
* Formatting buffers
* Runtime bookkeeping

Private data cannot be raced on by another thread unless references are shared deliberately.

---

# 11.34 Immutability

**Immutable data** cannot be changed after it is created.

Several threads can safely read immutable data simultaneously.

```text
Thread A ─┐
Thread B ─┼──▶ Immutable configuration
Thread C ─┘
```

No lock is needed merely to prevent modification because modification is disallowed.

---

## Why immutability helps

It reduces:

* Shared updates
* Locking requirements
* Visibility problems
* Accidental corruption
* Reasoning complexity

A common design strategy is:

> Share immutable data; keep mutable data private or carefully synchronized.

---

# 11.35 Ownership Transfer

Instead of several threads modifying one object, ownership can move from one thread to another.

```text
Thread A owns item
      │ sends item
      ▼
Thread B becomes owner
```

At any moment, only one thread is responsible for modifying it.

This can reduce shared mutable state.

---

## Example

1. Network thread receives a request.
2. It creates a request object.
3. It places the object in a queue.
4. A worker removes it.
5. The worker becomes responsible for processing it.

The queue is shared and synchronized, but the request object may be privately owned after removal.

---

# 11.36 Synchronization and the Scheduler

A synchronization operation can change thread states.

Suppose Thread A owns a mutex and Thread B requests it.

```text
Thread B:
RUNNING → WAITING
```

When Thread A releases the mutex:

```text
Thread B:
WAITING → READY
```

The scheduler eventually selects Thread B:

```text
READY → RUNNING
```

---

## Releasing a lock does not necessarily run the waiter immediately

The awakened thread becomes ready.

Whether it runs immediately depends on:

* Priority
* Available CPU cores
* Scheduling policy
* Other ready threads
* Current kernel state

---

# 11.37 Synchronization and Context Switching

Heavy lock contention can cause many context switches.

```text
Thread A acquires lock
Thread B blocks
Scheduler runs Thread C
Thread A releases lock
Thread B wakes
Scheduler later runs Thread B
```

This costs more than the protected update itself in some workloads.

---

## Lock convoy

A **lock convoy** can occur when many threads repeatedly queue behind one lock.

```text
A owns lock
B waits
C waits
D waits
E waits
```

As each owner releases it, another thread wakes, runs briefly, and hands it onward.

This can create:

* Many wake-ups
* Context switches
* Poor cache behavior
* Low scalability

---

# 11.38 Synchronization Across Processes

Processes normally have separate address spaces, but they may still need coordination.

Examples:

* Multiple processes update one file.
* Several processes use shared memory.
* A server process communicates with clients.
* Processes share a database.
* One process waits for another to finish.

Mechanisms may include:

* OS-managed locks
* Shared-memory synchronization objects
* Pipes
* Message queues
* Signals or events
* File locks
* Sockets
* Database transactions

The OS must ensure that the synchronization object itself is visible and meaningful to all participants.

---

# 11.39 File Locking

When several processes access one file, they may use locks to coordinate.

Possible models include:

* Lock the whole file.
* Lock a region.
* Use advisory coordination.
* Use mandatory enforcement.

Exact behavior varies significantly across operating systems and file systems.

---

## Advisory locking

Participants agree to check and obey the lock.

A process that ignores the convention may still access the file.

---

## Mandatory locking

The OS actively blocks operations that violate the lock.

This is less universally used and has system-specific rules.

---

## Important limitation

File locking alone may not ensure application-level correctness.

A database update may need transactions, logging, and recovery in addition to a file lock.

---

# 11.40 Database Transactions as Synchronization

A database transaction groups operations into a logical unit.

Conceptually:

```text
Begin transaction
    │
Read records
    │
Modify related records
    │
Commit
```

The database coordinates concurrent users so that incomplete intermediate states are not exposed improperly.

Transactions address problems involving:

* Atomicity
* Consistency
* Isolation
* Durability

These ideas are related to OS synchronization but implemented at a higher application or database level.

---

# 11.41 Priority and Synchronization

Suppose a high-priority thread waits for a lock held by a low-priority thread.

```text
Low-priority Thread L owns lock.
High-priority Thread H waits.
```

If medium-priority threads keep running, L may not receive enough CPU time to release the lock.

H is indirectly delayed.

This is **priority inversion**, introduced earlier.

---

## Priority inheritance

One possible response is:

1. High-priority thread waits on a lock.
2. Low-priority owner temporarily receives higher priority.
3. Owner runs and releases the lock.
4. Temporary priority boost is removed.

```text
H waits for L
     │
     ▼
L temporarily inherits H’s priority
     │
     ▼
L releases lock sooner
```

This helps but adds scheduler and lock complexity.

---

# 11.42 Fairness

A synchronization mechanism may need to decide which waiter proceeds next.

Possible policies include:

* First waiter first
* Highest priority first
* Unspecified order
* Approximate fairness
* Writer preference
* Reader preference

---

## Why fairness matters

Without fairness, one thread may repeatedly lose access.

This is a form of starvation.

```text
Thread A acquires repeatedly.
Thread B remains ready but never succeeds.
```

Strict fairness can also reduce performance, so systems balance fairness and throughput.

---

# 11.43 Starvation in Synchronization

A thread is **starved** when it waits indefinitely or for an excessive period despite the system continuing to make progress.

Examples:

* Readers continually prevent a writer from acquiring a read–write lock.
* High-priority threads repeatedly beat a low-priority waiter.
* An unfair mutex repeatedly favors recently running threads.

Starvation differs from deadlock:

* In starvation, other work continues.
* In deadlock, a group of participants may all be unable to continue.

Deadlock will be covered in the next section.

---

# 11.44 Livelock Preview

In a **livelock**, threads continue changing state but still make no useful progress.

Imagine two people meeting in a hallway:

1. Both move left to avoid each other.
2. Both move right.
3. Both move left again.
4. They remain active but never pass.

```text
Thread A reacts to B.
Thread B reacts to A.
Both retry continuously.
No work completes.
```

Livelock differs from waiting-based deadlock, but both prevent useful progress.

---

# 11.45 Lock-Free and Wait-Free Ideas

Some algorithms coordinate without traditional blocking locks.

They may use hardware atomic operations and retry logic.

---

## Lock-free concept

The system guarantees that some participant continues making progress, even if individual threads may retry.

---

## Wait-free concept

Each participating operation is guaranteed to complete within a bounded number of its own steps.

---

## Why these approaches exist

They can reduce:

* Lock contention
* Blocking
* Priority inversion
* Problems caused by suspended lock owners

---

## Why they are advanced

They require careful reasoning about:

* Atomic operations
* Memory ordering
* Retry behavior
* Object lifetime
* Subtle interleavings

They are not automatically faster or simpler.

For foundational study, mutexes and condition variables provide the clearest mental model.

---

# 11.46 Step-by-Step: Two Threads Updating One Document

Suppose an editor has:

* An interface thread
* An automatic-save thread

Both access the current document.

---

## Without synchronization

### Step 1: Interface thread begins editing

It updates the document text.

### Step 2: Scheduler switches threads midway

The document’s text has changed, but related metadata has not yet been updated.

### Step 3: Auto-save thread reads the document

It sees:

* New text
* Old length
* Old modification version

### Step 4: Auto-save writes inconsistent data

The saved file may be incomplete or malformed.

---

## With a document mutex

### Step 1: Interface thread acquires document lock

### Step 2: It updates text and related metadata

### Step 3: It restores all document invariants

### Step 4: It releases the lock

### Step 5: Auto-save thread acquires the lock

### Step 6: It copies a consistent document snapshot

### Step 7: It releases the lock

### Step 8: It performs slow storage I/O outside the lock

```text
Interface thread:
lock → update shared document → unlock

Auto-save:
wait → lock → copy consistent state → unlock → write file
```

Keeping the slow file write outside the lock prevents unnecessary blocking.

---

# 11.47 Step-by-Step: Multiple Threads Using a Connection Pool

Suppose an application has three database connections and ten worker threads.

A semaphore starts with:

```text
Count = 3
```

---

## First three workers

Each acquires one permit:

```text
3 → 2 → 1 → 0
```

They use the three connections.

---

## Fourth worker

The semaphore count is zero.

```text
Worker 4:
RUNNING → WAITING
```

---

## One worker finishes

It returns its connection and releases a permit.

```text
0 → 1
```

A waiting worker becomes ready.

---

## Scheduler resumes a waiter

The worker obtains the permit and uses the available connection.

The semaphore limits concurrency to the real resource capacity.

---

# 11.48 Step-by-Step: Thread Waits for Initialization

Suppose one thread loads configuration while several workers need it.

## Stage 1: Configuration state begins as unavailable

```text
configuration_ready = false
```

---

## Stage 2: Worker acquires the state mutex

It checks whether configuration is ready.

---

## Stage 3: Condition is false

The worker waits on a condition variable.

The wait releases the mutex.

```text
Worker:
RUNNING → WAITING
```

---

## Stage 4: Loader thread acquires the mutex

It stores the completed configuration and sets:

```text
configuration_ready = true
```

---

## Stage 5: Loader signals or broadcasts

Waiting workers become eligible to wake.

---

## Stage 6: Loader releases the mutex

---

## Stage 7: Worker reacquires the mutex

It checks `configuration_ready` again.

---

## Stage 8: Worker proceeds

The mutex and condition variable establish both ordering and visibility.

```text
Loader:
lock → write configuration → mark ready → signal → unlock

Worker:
lock → wait while not ready → recheck → use configuration → unlock
```

---

# 11.49 Step-by-Step: Shared Counter with a Mutex

Suppose the counter begins at `100`.

Thread A and Thread B both increment it.

## Thread A

1. Acquires counter mutex.
2. Reads `100`.
3. Calculates `101`.
4. Writes `101`.
5. Releases mutex.

## Thread B

1. Waits while A holds the mutex.
2. Acquires mutex.
3. Reads `101`.
4. Calculates `102`.
5. Writes `102`.
6. Releases mutex.

Final value:

```text
102
```

The mutex ensures that the read–modify–write sequences do not overlap.

---

# 11.50 What Can Go Wrong?

## Missing synchronization

Threads access shared mutable state without an agreed coordination mechanism.

Possible results:

* Lost updates
* Inconsistent state
* Corrupted structures
* Crashes
* Security defects

This becomes a race condition when the result depends on execution timing.

---

## Lock held too long

A thread performs slow work while owning a lock.

Possible results:

* High contention
* Poor responsiveness
* Reduced parallelism
* Lock convoys

---

## Lock not released

A thread exits a path without releasing its lock.

Possible result:

* Other threads wait forever.

This can contribute to deadlock.

---

## Wrong lock used

Thread A protects data with Lock X.

Thread B protects the same data with Lock Y.

```text
Thread A → Lock X → shared data
Thread B → Lock Y → shared data
```

Because the locks are different, they do not exclude each other.

---

## Some access bypasses the lock

A thread reads or writes protected state without following the protocol.

The synchronization guarantee is broken.

---

## Lock protects too much

Unrelated work becomes serialized.

Performance declines.

---

## Lock protects too little

Related changes can still interleave.

Correctness fails.

---

## Waiting without releasing a lock

A thread waits for a condition while preventing other threads from making the condition true.

Possible result:

* Permanent waiting
* Deadlock

---

## Signaling without shared-state discipline

A thread sends a notification but updates shared state incorrectly or outside the agreed synchronization protocol.

A waiter may wake and observe inconsistent data.

---

## Assuming wake-up means success

A thread wakes and proceeds without rechecking the actual condition.

Another thread may already have consumed the resource.

---

## Excessive spinning

A thread repeatedly checks a busy lock for too long.

Possible results:

* Wasted CPU
* Increased power use
* Lower system throughput

---

## Priority inversion

A high-priority thread waits for a lower-priority lock owner.

---

## Starvation

A participant repeatedly loses access and waits excessively.

---

## Too much shared mutable state

The program requires many complicated synchronization rules.

This increases:

* Defect risk
* Review difficulty
* Test complexity
* Performance uncertainty

---

# 11.51 Common Misconceptions

## Misconception: “Concurrency means simultaneous execution”

Not necessarily.

Concurrency can exist on one core through interleaving.

Parallelism means physical simultaneous execution.

---

## Misconception: “One CPU instruction means one high-level operation”

A high-level action may involve many instructions.

Even one source-level increment may require a read–modify–write sequence.

---

## Misconception: “Threads in the same process automatically coordinate”

They share memory, but sharing does not provide ordering or safety.

They need a synchronization protocol.

---

## Misconception: “A mutex makes the protected data globally safe”

Only participants using the same mutex and protocol receive protection.

A thread bypassing the lock can still corrupt the data.

---

## Misconception: “Locks protect variables”

More precisely, locks protect **invariants and operations** involving shared state.

A single operation may involve several related variables.

---

## Misconception: “Only writes need synchronization”

Reads can also require synchronization.

A reader may otherwise observe:

* Partial updates
* Stale values
* Inconsistent combinations of fields

---

## Misconception: “If the program works repeatedly, synchronization is correct”

Timing defects may appear only under:

* Different CPU counts
* Heavy load
* Different scheduling
* Faster or slower devices
* Rare interrupts
* Different compiler optimizations

Successful tests do not prove all interleavings are safe.

---

## Misconception: “More locks make a program safer”

More locks can create:

* Deadlocks
* Incorrect ordering
* Higher complexity
* More contention

Correct lock design matters more than lock quantity.

---

## Misconception: “A semaphore is just a mutex with a different name”

A mutex usually provides single-owner exclusion.

A semaphore represents permits and may be signaled by a different participant.

---

## Misconception: “When a condition variable wakes a thread, the condition is guaranteed true”

The thread must reacquire the lock and check the condition again.

---

## Misconception: “A thread waiting for a mutex keeps using the CPU”

A blocking mutex normally lets the thread sleep.

A spinlock is the case where the thread repeatedly checks while consuming CPU.

---

## Misconception: “Atomic operations eliminate all synchronization problems”

Atomic operations protect particular low-level actions.

They do not automatically make complex multi-step data structures correct.

---

# 11.52 Real-World Analogy: Shared Office Printer and Records Room

Imagine several employees in one office.

## Concurrency

They work on different tasks during overlapping periods.

## Parallelism

Several employees physically work at the same time.

## Shared state

They use one shared project ledger.

## Critical section

Updating the ledger’s balance and transaction history.

## Mutex

The key to the locked records room.

Only the key holder may update the ledger.

## Semaphore

Three access badges for three available meeting rooms.

## Condition variable

An employee waits until a delivery arrives and is notified by reception.

## Barrier

All team members finish their reports before the review meeting begins.

## Message passing

Employees send sealed task envelopes rather than editing one another’s notebooks.

## Contention

Many employees wait for the same records-room key.

---

# 11.53 Connection to Earlier Concepts

## Connection to threads

Threads are concurrent execution sequences.

Threads in one process usually share heap memory and open resources.

This makes synchronization necessary.

---

## Connection to processes

Processes are more isolated, but they can still coordinate through:

* Shared memory
* Files
* Pipes
* Messages
* Network connections
* OS synchronization objects

---

## Connection to scheduling

A thread blocked on a mutex or condition variable enters a waiting state.

When it can proceed, it becomes ready.

---

## Connection to context switching

Lock contention and wake-ups can cause additional context switches.

---

## Connection to CPU hardware

Hardware atomic operations help implement synchronization.

CPU caches and memory ordering make visibility rules necessary.

---

## Connection to virtual memory

Shared-memory mappings allow processes to access the same physical frames.

Synchronization is needed when those pages contain mutable data.

---

## Connection to files

Multiple processes modifying a file may need:

* File locks
* Transactions
* Temporary-file replacement
* Application-level coordination

---

## Connection to I/O

Producer–consumer queues are common in device management.

For example:

```text
Device interrupt handler
        │ produces completion
        ▼
Kernel work queue
        │ consumed by worker
        ▼
Application notified
```

---

## Connection to user and kernel mode

Synchronization mechanisms may begin in user space using atomic operations.

If waiting is necessary, the kernel scheduler may become involved.

---

# 11.54 Simplified Model Versus Technical Reality

## Simplified mental model

```text
Concurrency:
Several activities progress.

Shared mutable data:
Activities can interfere.

Synchronization:
Locks and signaling impose safe ordering.
```

This is the model to retain.

---

## More exact reality

Real concurrency systems may include:

* Hardware atomic instructions
* Detailed memory models
* Per-CPU caches
* Lock-free structures
* Read-copy-update techniques
* Priority-aware locks
* Adaptive spinning
* Kernel wait queues
* Futex-like mechanisms
* Distributed coordination
* Transactional memory
* Language runtimes
* Garbage collectors
* Actor systems
* Asynchronous event loops

Different programming languages provide different guarantees.

Some synchronization occurs entirely in user mode when uncontended.

Kernel involvement may occur only when a thread must block.

The essential principle remains:

> Concurrent activities must coordinate whenever correctness depends on the order or visibility of shared operations.

---

# 11.55 Core Mental Model

## Shared-state problem

```text
Thread A ─┐
          ├──▶ Shared mutable state
Thread B ─┘

Without coordination:
Result may depend on timing.
```

---

## Mutex model

```text
Acquire lock
      │
      ▼
Read and update shared state
      │
      ▼
Restore invariants
      │
      ▼
Release lock
```

---

## Waiting model

```text
Thread requests busy lock
          │
          ▼
Thread WAITING
          │
          │ lock released
          ▼
Thread READY
          │
          │ scheduler selects
          ▼
Thread RUNNING
```

---

## Condition-variable model

```text
Acquire mutex
      │
      ▼
Condition true?
   ┌──┴───┐
   │      │
  Yes     No
   │      │
Proceed  Release mutex and wait
              │
              ▼
          Notification
              │
              ▼
          Reacquire mutex
              │
              ▼
          Recheck condition
```

---

## Producer–consumer model

```text
Producer
    │
    ▼
Synchronized queue
    │
    ▼
Consumer
```

---

## Final distinctions

| Concept                | Essential meaning                                     |
| ---------------------- | ----------------------------------------------------- |
| **Concurrency**        | Activities make progress during overlapping time      |
| **Parallelism**        | Activities execute physically at the same instant     |
| **Shared state**       | Data accessible to multiple concurrent participants   |
| **Atomic operation**   | Appears indivisible to other participants             |
| **Critical section**   | Code that must access shared state safely             |
| **Mutual exclusion**   | Only one participant enters a protected section       |
| **Mutex**              | Ownership-based exclusive lock                        |
| **Semaphore**          | Counter representing available permits                |
| **Condition variable** | Lets threads wait for shared state to change          |
| **Barrier**            | Holds participants until all reach one phase          |
| **Contention**         | Competition for the same synchronization resource     |
| **Starvation**         | A participant waits excessively while others progress |

The next section examines the two most important concurrency failures:

* Race conditions
* Deadlocks

It will also separate them from starvation and livelock.

# Learning Check

Do not look for answers yet.

## Conceptual questions

1. What is the difference between concurrency and parallelism?
2. What is a critical section, and why is mutual exclusion sometimes required around it?
3. What is the difference between a mutex, a semaphore, and a condition variable?

## Cause-and-effect questions

4. Why can holding a mutex during slow file or network I/O significantly reduce concurrency?
5. Why must a thread recheck a condition after waking from a condition-variable wait?

## Misconception question

6. A student says, “If two threads run on only one CPU core, they cannot interfere with each other because they never execute at the exact same instant.” What is wrong with this statement?

## Scenario-based question

7. A bounded queue connects two producer threads with three consumer threads. Explain how a mutex and the conditions “not empty” and “not full” can coordinate access, including when threads become waiting and ready.

# 12. Race Conditions and Deadlocks

Concurrency allows multiple threads or processes to make progress during overlapping periods.

That creates two major classes of failure:

| Failure            | Core problem                                    |
| ------------------ | ----------------------------------------------- |
| **Race condition** | The result depends on an unsafe execution order |
| **Deadlock**       | Participants wait for one another permanently   |

Related problems include:

* **Starvation:** one participant waits excessively while others continue.
* **Livelock:** participants remain active but repeatedly prevent progress.

```text
Concurrency failures
├── Race condition: wrong result because of timing
├── Deadlock: no progress because of circular waiting
├── Starvation: one participant repeatedly loses access
└── Livelock: activity continues, useful progress does not
```

These problems are different and require different solutions.

---

# 12.1 What Is a Race Condition?

## What it is

A **race condition** occurs when a system’s correctness depends on the relative timing or ordering of concurrent operations, but that ordering is not properly controlled.

The participating activities are effectively “racing” to access or modify shared state.

```text
Thread A ─┐
          ├──▶ Shared mutable data
Thread B ─┘

Result depends on which steps happen first.
```

A race condition does not require true simultaneous execution.

Threads can race on a single CPU core because context switches may occur between their individual steps.

---

## Why race conditions exist

High-level operations are often made from several smaller operations.

For example:

> Increase a shared counter.

may require:

1. Read the current value.
2. Calculate a new value.
3. Write the result.

Another thread can run between any of these steps.

```text
Read ── context switch ── calculate ── context switch ── write
```

If two threads perform overlapping read–modify–write sequences, one update may be lost.

---

## The problem a race condition causes

Race conditions can produce:

* Incorrect calculations
* Lost updates
* Corrupted data structures
* Security failures
* Duplicate resource allocation
* Inconsistent files
* Rare crashes
* Different results across runs

The defect may disappear when debugging because debugging changes timing.

---

# 12.2 Mental Model: Two Clerks Reserving One Seat

Suppose a booking system says:

```text
Seats available: 1
```

Two clerks process requests at the same time.

## Unsafe timeline

```text
Time ─────────────────────────────────────────────▶

Clerk A: Read 1 ───── Decide available ───── Write 0
Clerk B:      Read 1 ───── Decide available ───── Write 0
```

Both clerks believe they successfully reserved the last seat.

The final stored value is still `0`, so merely checking the counter does not reveal that two reservations were accepted.

---

## Why this is a race

Correctness depended on one clerk completing the entire reservation before the other checked availability.

That ordering was not enforced.

---

## Safer sequence

```text
Clerk A locks reservation record.
Clerk A checks and reserves seat.
Clerk A unlocks record.
Clerk B locks reservation record.
Clerk B sees no available seat.
```

The lock makes the check-and-update sequence one protected critical section.

---

# 12.3 Data Race Versus Race Condition

These terms are related but not identical.

## Data race

A **data race** usually means that concurrent activities access the same memory location without suitable synchronization, at least one access modifies it, and the language or memory model considers the behavior unsafe.

```text
Thread A writes shared value
Thread B reads or writes same value
No valid synchronization
```

---

## Race condition

A race condition is broader.

The system’s correctness depends on uncontrolled timing, even when individual memory accesses may be technically protected.

---

## Example without a simple data race

Consider:

1. Process A checks whether a file exists.
2. Process B replaces that file.
3. Process A opens the path, assuming it still refers to the checked file.

Each OS operation may be individually valid and synchronized internally.

The race exists between:

```text
Check
  │ uncontrolled time gap
  ▼
Use
```

This is called a **check-then-act** race.

---

## Relationship

| Concept        | Scope                                                |
| -------------- | ---------------------------------------------------- |
| Data race      | Unsafe concurrent memory access                      |
| Race condition | Any correctness defect caused by uncontrolled timing |

A data race is commonly a kind of race condition, but race conditions can occur through files, processes, devices, networks, or multiple individually safe operations.

---

# 12.4 Common Race Pattern: Read–Modify–Write

Suppose a shared counter begins at `50`.

Thread A and Thread B both increase it.

## Expected result

```text
52
```

## Unsafe interleaving

```text
Thread A reads 50.
Thread B reads 50.
Thread A calculates 51.
Thread B calculates 51.
Thread A writes 51.
Thread B writes 51.
```

## Actual result

```text
51
```

One update disappeared.

This is called a **lost update**.

---

## Why a normal read and write are insufficient

Even if reading one number and writing one number are each individually atomic, the complete sequence is not necessarily atomic.

```text
Atomic read
     +
calculation
     +
atomic write
     ≠
atomic combined operation
```

The entire logical operation needs coordination.

---

# 12.5 Common Race Pattern: Check Then Act

A thread checks a condition and later acts based on that result.

```text
Check condition
       │
       │ another thread changes state
       ▼
Act using outdated assumption
```

---

## Example: queue capacity

A queue has one remaining slot.

Thread A:

1. Checks that space exists.
2. Plans to insert an item.

Thread B:

1. Checks that space exists.
2. Inserts its item.

Thread A then inserts based on its earlier check.

The queue may exceed capacity or become corrupted.

---

## Correct approach

The check and action must be protected together.

```text
Acquire queue lock
      │
Check capacity
      │
Insert item
      │
Release queue lock
```

The lock must not be released between the check and the update.

---

# 12.6 Common Race Pattern: Time of Check to Time of Use

A **time-of-check to time-of-use** race occurs when a resource changes after it is checked but before it is used.

This is often abbreviated as **TOCTOU**.

---

## File example

A privileged service does:

1. Check that a path refers to an allowed file.
2. Later open that path.
3. Assume it still refers to the same object.

An attacker changes the path between steps 1 and 2.

```text
Service checks safe-file.txt
          │
          │ attacker replaces path target
          ▼
Service opens a different file
```

The check was correct at the time it happened, but the checked condition did not remain stable.

---

## Safer principle

Prefer obtaining a stable resource reference and then operating through that reference.

For example:

```text
Open object securely
       │
       ▼
Validate opened object
       │
       ▼
Use same open reference
```

A file descriptor is generally more stable than repeatedly resolving a pathname, although exact security guarantees depend on the OS and operation.

---

# 12.7 Common Race Pattern: Initialization Race

Suppose several threads need one shared resource.

Thread A begins creating it.

Thread B checks whether it exists and sees an incomplete state.

```text
Thread A: create structure ───── initialize fields ───── ready
Thread B:          reads structure too early
```

Thread B may observe:

* Missing fields
* Default values
* Partially loaded data
* Invalid pointers or references
* Incorrect readiness state

---

## Correct coordination

The resource should not be published as ready until initialization is complete.

```text
Build resource privately
        │
        ▼
Complete all initialization
        │
        ▼
Publish under synchronization
        │
        ▼
Other threads may use it
```

Synchronization must provide both:

* Ordering
* Memory visibility

---

# 12.8 Common Race Pattern: Resource Lifetime Race

One thread uses an object while another destroys or releases it.

```text
Thread A obtains reference
Thread B releases object
Thread A continues using old reference
```

Possible results:

* Invalid memory access
* Corruption of a newly reused object
* Device or file operations on a closed resource
* Security vulnerabilities

---

## Example: closing a descriptor

Thread A plans to use descriptor `5`.

Thread B closes descriptor `5`.

The OS later reuses the number `5` for another resource.

Thread A then uses `5`, assuming it still refers to the original file.

```text
Initially:
Descriptor 5 → report.txt

After close and reuse:
Descriptor 5 → network connection
```

The descriptor number is valid again, but it refers to the wrong resource.

Correct ownership and lifetime rules are required.

---

# 12.9 Race Conditions Can Occur Without Shared Process Memory

Races can involve:

* Files
* Databases
* Network requests
* Device state
* Process creation
* Permission changes
* Directory entries
* External services
* User actions

---

## Example: file update race

Two processes perform:

1. Read current document.
2. Modify their local copy.
3. Write the complete document.

```text
Process A reads version 1.
Process B reads version 1.
Process A writes version 2A.
Process B writes version 2B.
```

Process A’s update is lost.

The processes never shared memory, but they raced through shared persistent state.

---

# 12.10 Why Race Conditions Are Difficult to Reproduce

The exact execution order can depend on:

* Scheduler decisions
* Number of CPU cores
* Device timing
* Interrupts
* Cache behavior
* System load
* Logging
* Debugger pauses
* Compiler optimizations
* Network delay

A program may work correctly thousands of times and then fail once.

```text
Run 1: safe ordering
Run 2: safe ordering
Run 3: safe ordering
...
Run 20,000: harmful ordering
```

The rarity does not make the design correct.

---

## Heisenbug

A defect that changes or disappears when observed is sometimes called a **Heisenbug**.

Adding diagnostic output may alter timing enough to hide a race condition.

---

# 12.11 How Race Conditions Are Prevented

Common strategies include:

* Mutual exclusion
* Atomic operations
* Immutable data
* Message passing
* Ownership transfer
* Transactions
* Version checks
* Stable resource references
* Carefully designed lock-free algorithms

The correct mechanism depends on what must remain consistent.

---

## Mutual exclusion

Protect the entire logical operation.

```text
Acquire lock
      │
Check and update shared state
      │
Release lock
```

---

## Atomic operation

Use one hardware-supported atomic action when the logical operation is sufficiently small.

---

## Immutability

Do not modify shared data after publication.

---

## Message passing

Send work or data to one owner instead of allowing several threads to modify it directly.

---

## Transactions

Group related persistent operations so they commit or fail as one logical unit.

---

## Version checking

Before saving an update, verify that the underlying state has not changed since it was read.

```text
Read version 10
Make changes
Before writing, verify version is still 10
```

If it has changed, retry or report a conflict.

---

# 12.12 What Is a Deadlock?

## What it is

A **deadlock** occurs when a group of threads or processes permanently waits because each participant needs a resource or event that another participant in the same group is withholding.

```text
Thread A waits for Thread B.
Thread B waits for Thread A.
Neither can continue.
```

No participant can make the first move required to break the cycle.

---

## Why deadlocks exist

Synchronization allows threads to hold resources while waiting for others.

A dangerous pattern appears when resource dependencies form a cycle.

```text
Thread A holds Lock 1 and waits for Lock 2.
Thread B holds Lock 2 and waits for Lock 1.
```

Each thread prevents the other from proceeding.

---

## The problem deadlocks cause

Possible effects include:

* Frozen application
* Unresponsive user interface
* Stalled kernel subsystem
* Hung database transaction
* Requests that never finish
* Resources permanently unavailable
* System-wide failure in severe cases

The threads may consume little CPU because they are sleeping, but no useful progress occurs.

---

# 12.13 Mental Model: Two People and Two Keys

Two people need both a red key and a blue key to complete their work.

```text
Person A holds red key.
Person B holds blue key.
```

Person A waits for the blue key.

Person B waits for the red key.

```text
Person A:
holds red → waits for blue

Person B:
holds blue → waits for red
```

Neither will release the key already held until the other key is obtained.

They wait forever.

---

# 12.14 Step-by-Step Deadlock With Two Locks

Suppose the system has:

* Lock X
* Lock Y
* Thread A
* Thread B

## Step 1: Thread A acquires Lock X

```text
Thread A holds X.
```

---

## Step 2: Scheduler runs Thread B

Thread B acquires Lock Y.

```text
Thread B holds Y.
```

---

## Step 3: Thread A resumes

It tries to acquire Lock Y.

But Thread B owns Y.

```text
Thread A:
RUNNING → WAITING for Y
```

Thread A continues holding X while waiting.

---

## Step 4: Thread B runs

It tries to acquire Lock X.

But Thread A owns X.

```text
Thread B:
RUNNING → WAITING for X
```

Thread B continues holding Y while waiting.

---

## Final state

```text
Thread A holds X and waits for Y.
Thread B holds Y and waits for X.
```

Neither thread is ready to release its current lock.

```text
        waits for
Thread A ─────────▶ Lock Y
   ▲                   │
   │                   │ held by
held by                ▼
 Lock X ◀───────── Thread B
        waits for
```

This cycle is a deadlock.

---

# 12.15 Resource-Allocation Graph

A **resource-allocation graph** is a mental or diagnostic model showing:

* Which participant holds which resource
* Which resource each participant is waiting for

Example:

```text
Thread A → waits for Lock Y
Lock Y   → held by Thread B
Thread B → waits for Lock X
Lock X   → held by Thread A
```

The cycle is:

```text
Thread A → Lock Y → Thread B → Lock X → Thread A
```

A cycle is a central warning sign.

For single-instance resources, such a cycle indicates deadlock.

With multiple instances of a resource, analysis can be more complicated.

---

# 12.16 Four Conditions Commonly Associated With Deadlock

A classic model identifies four conditions that must all hold for resource deadlock.

## 1. Mutual exclusion

A resource can be used by only one participant at a time.

Example:

```text
Only one thread may own a mutex.
```

---

## 2. Hold and wait

A participant holds one resource while waiting for another.

```text
Thread A holds X while waiting for Y.
```

---

## 3. No forced preemption

A held resource cannot simply be taken away safely.

The owner must release it voluntarily.

---

## 4. Circular wait

A cycle exists:

```text
A waits for B
B waits for C
C waits for A
```

If at least one of these conditions is reliably prevented, this type of deadlock cannot occur.

---

# 12.17 Mutual Exclusion Is Not Always Removable

One possible way to prevent deadlock would be to allow every resource to be shared freely.

However, many resources inherently require exclusive access.

Examples:

* A mutable data structure during an update
* A printer processing one physical page stream
* A unique hardware controller state
* A transaction modifying one record

Therefore, deadlock prevention usually focuses on:

* Hold-and-wait
* Resource acquisition order
* Time limits
* Retry strategies

---

# 12.18 Preventing Hold and Wait

One approach is to acquire all required resources together before beginning.

```text
Need X and Y
     │
     ▼
Acquire both or acquire neither
```

If both are unavailable, the thread waits without holding either.

---

## Benefit

A thread cannot hold X while waiting for Y.

---

## Costs

* The thread may reserve resources before it actually needs them.
* Resource utilization may decrease.
* The full resource set may not be known in advance.
* Large atomic acquisition can be difficult to implement.

---

# 12.19 Global Lock Ordering

A common deadlock-prevention technique is to define one universal order for acquiring locks.

Suppose:

```text
Lock order:
X before Y before Z
```

Every thread must follow that order.

Allowed:

```text
Acquire X
Acquire Y
Acquire Z
```

Not allowed:

```text
Acquire Z
Then acquire X
```

---

## Why ordering works

A circular dependency requires at least one participant to acquire resources in the reverse direction.

If all participants move through the same order, the cycle cannot form.

```text
All dependencies move:
X → Y → Z

No dependency moves:
Z → X
```

---

## Example

Thread A needs X and Y:

```text
Acquire X, then Y.
```

Thread B also needs X and Y:

```text
Acquire X, then Y.
```

If A owns X, B waits before acquiring anything else involved in the pair.

B cannot hold Y while waiting for X.

---

## Limitation

Global ordering becomes difficult when:

* The system has many locks
* Resource relationships are dynamic
* Components are developed independently
* Lock requirements are discovered during execution

Good documentation and design discipline are essential.

---

# 12.20 Try-Lock and Backoff

Instead of waiting indefinitely, a thread can attempt to acquire a lock without blocking permanently.

Conceptually:

1. Acquire Lock X.
2. Try to acquire Lock Y.
3. If Y is unavailable:

   * Release X.
   * Wait or back off.
   * Retry later.

```text
Acquire X
    │
Try Y
 ┌──┴───┐
 │      │
Success Busy
 │      │
Proceed Release X
        wait
        retry
```

---

## Benefit

The thread does not remain stuck holding X while waiting for Y.

---

## New risk

Two threads may repeatedly:

1. Acquire one lock.
2. Fail to get the second.
3. Release.
4. Retry at the same time.

This can create livelock.

Randomized or increasing delays can reduce synchronized retries.

---

# 12.21 Timeouts

A thread may wait for a resource only for a limited time.

```text
Wait for lock
      │
      ├── acquired before deadline → continue
      └── timeout expires → recover or fail
```

---

## Why timeouts help

They prevent indefinite waiting and allow:

* Error reporting
* Retry
* Request cancellation
* Resource cleanup
* Diagnostic logging

---

## Why timeouts do not fully solve deadlock

A timeout detects that progress is taking too long.

It does not necessarily identify:

* Which locks form the cycle
* Which participant should abort
* Whether the system is merely slow
* Whether retrying will repeat the problem

Timeouts are recovery tools, not always proof of correct design.

---

# 12.22 Deadlock Avoidance

**Deadlock avoidance** means deciding whether granting a resource request could move the system into a dangerous state.

The system may delay a request even though the resource is currently available.

---

## Safe-state mental model

A state is considered safe if there remains some order in which all participants could eventually complete.

```text
Can A complete and release resources?
Then can B complete?
Then can C complete?
```

If no completion order remains possible, the system may be unsafe.

---

## Limitation

Avoidance requires knowledge such as:

* Maximum future resource needs
* Current resource ownership
* Available resource quantities

General-purpose operating systems often cannot predict every future lock request.

Therefore, avoidance is more practical in controlled resource-allocation systems than for arbitrary application mutexes.

---

# 12.23 Deadlock Detection

Instead of preventing every deadlock, a system can allow one to form and then detect it.

Detection may involve:

* Tracking resource ownership
* Tracking waiting relationships
* Searching for cycles
* Observing requests that fail to progress

```text
Build wait-for graph
       │
       ▼
Search for cycle
   ┌───┴───┐
   │       │
No cycle  Cycle found
   │       │
Continue  Recovery needed
```

---

## Wait-for graph

A simplified graph contains only participants:

```text
Thread A → waits for Thread B
Thread B → waits for Thread C
Thread C → waits for Thread A
```

The cycle indicates deadlock.

---

## Detection cost

Tracking and analyzing dependencies adds:

* Memory use
* CPU overhead
* Implementation complexity
* Possible delay before detection

Not every lock system performs automatic deadlock detection.

---

# 12.24 Deadlock Recovery

Once a deadlock is found, the system must break the cycle.

Possible strategies include:

* Terminate one participant
* Cancel one operation
* Roll back a transaction
* Force one resource to be released when safe
* Restart a service
* Restart the application
* Restart the system in severe cases

---

## Choosing a victim

A system may consider:

* Work already completed
* Priority
* Resources held
* Cost of restarting
* User impact
* Whether rollback is possible

There is no universally harmless choice.

---

## Why forced lock removal is dangerous

A thread may hold a lock because shared data is temporarily inconsistent.

Taking the lock away without repairing that state can expose corruption.

Therefore, terminating or rolling back a larger unit may be safer than forcibly transferring an ordinary mutex.

---

# 12.25 Deadlock Involving More Than Locks

Deadlocks can involve any dependencies, not only mutexes.

Examples:

* Threads waiting for messages
* Processes waiting for each other to exit
* Full communication buffers
* File locks
* Database records
* Device requests
* User-interface events
* Network protocols

---

# 12.26 Communication Deadlock

Suppose two processes communicate through bounded channels.

Process A:

1. Sends a large message to B.
2. Waits for B’s reply.

Process B:

1. Sends a large message to A.
2. Waits for A’s reply.

If both output buffers fill before either process begins reading:

```text
A waits for B to read.
B waits for A to read.
```

Both processes are blocked.

The deadlock involves communication capacity rather than mutexes.

---

# 12.27 Thread-Join Deadlock

A thread may wait for another thread to finish. This is often called **joining**.

Suppose:

```text
Thread A waits for Thread B to finish.
Thread B waits for Thread A to finish.
```

Neither can finish first.

A thread can also accidentally wait for itself, depending on the available interface.

---

# 12.28 Event Deadlock

Suppose Thread A waits for an event that Thread B must signal.

Thread B waits for a lock held by Thread A.

```text
Thread A:
holds Lock X
waits for Event E

Thread B:
needs Lock X
must acquire it before signaling E
```

A waits for B’s signal.

B cannot signal because A holds the required lock.

This shows why waiting while holding a lock is dangerous.

---

# 12.29 Deadlock During File or Network I/O

Suppose a thread holds an application mutex and performs blocking network I/O.

Another thread needs the mutex to process the network response.

```text
Thread A:
holds mutex
waits for network response

Thread B:
received response
needs mutex to process it
```

Thread A waits for work that Thread B cannot complete.

The slow operation and the lock form a dependency cycle.

---

## Safer pattern

Copy or prepare required shared state while holding the lock, then release it before slow I/O.

```text
Acquire lock
Prepare request state
Release lock
Perform network I/O
Reacquire only if needed
```

This must be designed carefully so shared assumptions remain valid.

---

# 12.30 What Is Starvation?

## What it is

**Starvation** occurs when a participant waits for an excessive or unbounded time because others repeatedly receive access first.

```text
Thread A proceeds.
Thread B proceeds.
Thread C proceeds.
Thread A proceeds again.
Thread D keeps waiting.
```

The system as a whole makes progress, but one participant does not.

---

## Examples

* A low-priority thread never receives enough CPU time.
* Readers continuously prevent a writer from acquiring a lock.
* An unfair mutex repeatedly favors other threads.
* High-priority I/O requests continually delay low-priority requests.

---

## Starvation versus deadlock

| Deadlock                           | Starvation                              |
| ---------------------------------- | --------------------------------------- |
| A group cannot progress            | Other participants continue progressing |
| Usually involves cyclic dependency | Often involves unfair allocation        |
| Resources remain mutually blocked  | Resource keeps being granted elsewhere  |

---

## Prevention

Possible techniques include:

* Fair queues
* Aging
* Priority adjustment
* Bounded waiting guarantees
* Limiting consecutive access by one class
* Fair reader–writer policies

Strict fairness may reduce throughput, so systems often use approximate fairness.

---

# 12.31 What Is Livelock?

## What it is

A **livelock** occurs when participants remain active and keep reacting to one another but make no useful progress.

```text
Thread A changes state to help B.
Thread B changes state to help A.
Both retry.
The same pattern repeats.
```

Unlike deadlocked threads, livelocked threads may consume substantial CPU.

---

## Hallway analogy

Two people approach each other.

1. Both step left.
2. Both step right.
3. Both step left.
4. Neither passes.

They are not waiting motionlessly.

They are active but ineffective.

---

## Computer example

Two threads need Locks X and Y.

Both use:

1. Acquire one lock.
2. Try the second.
3. If unavailable, release and retry immediately.

Timeline:

```text
Thread A acquires X.
Thread B acquires Y.
A cannot get Y, releases X.
B cannot get X, releases Y.
A immediately reacquires X.
B immediately reacquires Y.
Repeat.
```

No deadlock remains because locks are released, but no thread completes.

---

## Reducing livelock

Possible techniques include:

* Randomized delays
* Increasing backoff periods
* Priority rules
* One participant yielding
* Central coordination
* Consistent lock ordering

---

# 12.32 Comparison of Four Concurrency Failures

| Problem        | Are participants active? | Does the system progress? | Main cause                |
| -------------- | -----------------------: | ------------------------: | ------------------------- |
| Race condition |                  Usually | Possibly, but incorrectly | Unsafe ordering           |
| Deadlock       |          Usually waiting |     No for involved group | Circular dependency       |
| Starvation     |          Some are active | Yes, but one is neglected | Unfair allocation         |
| Livelock       |                      Yes |        No useful progress | Repeated reactive retries |

---

# 12.33 Race Condition Versus Deadlock

These are often confused.

## Race condition

Threads progress, but the result may be wrong.

```text
Two updates overlap.
One update is lost.
Program continues with incorrect state.
```

---

## Deadlock

Threads cannot progress.

```text
Each thread waits for a resource held by another.
No result is produced.
```

---

## Can both occur in one program?

Yes.

A program can contain:

* A race in one data structure
* A deadlock in another lock sequence
* Starvation in a scheduler or queue
* Livelock in retry logic

Fixing one concurrency problem does not automatically fix the others.

---

# 12.34 Locks Can Fix Races but Introduce Deadlocks

Suppose shared accounts are updated without locks.

This creates a race.

A developer adds one lock per account.

Now a transfer between two accounts requires two locks.

```text
Transfer A → B:
Acquire Account A lock
Acquire Account B lock
```

Another thread performs the opposite transfer:

```text
Transfer B → A:
Acquire Account B lock
Acquire Account A lock
```

The race is reduced, but inconsistent lock order creates deadlock risk.

---

## Lesson

Synchronization design must consider both:

* Data correctness
* Progress guarantees

A correct shared-state update is not enough if threads can permanently block one another.

---

# 12.35 Step-by-Step: Bank Transfer Race

Suppose Account A contains `100`.

Two threads both withdraw `80`.

## Unsafe timeline

```text
Thread 1 reads balance 100.
Thread 2 reads balance 100.
Thread 1 checks 100 ≥ 80.
Thread 2 checks 100 ≥ 80.
Thread 1 writes balance 20.
Thread 2 writes balance 20.
```

Both withdrawals succeed even though only one should.

The final stored balance is `20`, but `160` was withdrawn.

---

## Protected sequence

```text
Thread 1 acquires account lock.
Thread 1 checks balance.
Thread 1 writes 20.
Thread 1 releases account lock.

Thread 2 acquires account lock.
Thread 2 reads 20.
Thread 2 rejects withdrawal.
Thread 2 releases account lock.
```

The check and update form one critical section.

---

# 12.36 Step-by-Step: Bank Transfer Deadlock

Suppose Thread 1 transfers from A to B.

Thread 2 transfers from B to A.

## Thread 1

1. Acquires Account A lock.
2. Tries to acquire Account B lock.

## Thread 2

1. Acquires Account B lock.
2. Tries to acquire Account A lock.

```text
Thread 1 holds A, waits for B.
Thread 2 holds B, waits for A.
```

Deadlock occurs.

---

## Prevention with global order

Define:

```text
Always lock the lower-numbered account first.
```

Both transfers must acquire:

```text
Account A lock
then Account B lock
```

Even the B-to-A transfer follows the same lock order.

The business direction of the transfer does not determine lock order.

---

# 12.37 Step-by-Step: Queue Race

Suppose two consumer threads remove items from one queue.

Queue initially contains:

```text
[Item X]
```

## Unsafe timeline

```text
Consumer A checks: queue not empty.
Consumer B checks: queue not empty.
Consumer A reads Item X.
Consumer B reads Item X.
Consumer A removes first item.
Consumer B also attempts removal.
```

Possible results:

* Item processed twice
* Queue metadata corrupted
* Invalid removal
* Crash

---

## Protected queue operation

```text
Acquire queue mutex
      │
Check not empty
      │
Remove exactly one item
      │
Update queue metadata
      │
Release mutex
```

The item can then be processed outside the lock.

---

# 12.38 Step-by-Step: Queue Deadlock

Suppose a producer holds the queue lock while waiting for free space.

```text
Producer:
acquires queue lock
finds queue full
waits without releasing lock
```

A consumer needs the same lock to remove an item.

```text
Consumer:
waits for queue lock
```

Final state:

```text
Producer waits for queue space.
Consumer could create space but waits for producer’s lock.
```

This is why condition-variable waiting must release the associated mutex.

---

# 12.39 Detailed Walkthrough: Race During File Saving

Suppose two application instances edit the same file.

## Initial state

```text
File version: 5
Contents: Original
```

---

## Stage 1: Process A reads version 5

It creates local changes A.

---

## Stage 2: Process B reads version 5

It creates local changes B.

---

## Stage 3: Process A saves

The file becomes:

```text
Version 6
Contents: Changes A
```

---

## Stage 4: Process B saves its old base

It writes:

```text
Version 6 or 7
Contents: Changes B
```

Process A’s changes disappear.

---

## Why the file system does not automatically prevent this

Each individual write may be valid.

The file system does not necessarily understand that both processes edited from the same earlier logical version.

This is an application-level race.

---

## Possible solutions

* Lock the document while editing.
* Check a version or modification timestamp before saving.
* Merge changes.
* Save through a transaction-capable service.
* Create separate versions and report a conflict.

The best policy depends on the application.

---

# 12.40 Detailed Walkthrough: Deadlock Detection in a Service

Suppose three worker threads hold locks as follows:

```text
Thread A holds Lock 1, waits for Lock 2.
Thread B holds Lock 2, waits for Lock 3.
Thread C holds Lock 3, waits for Lock 1.
```

---

## Stage 1: System records ownership

```text
Lock 1 → A
Lock 2 → B
Lock 3 → C
```

---

## Stage 2: System records waits

```text
A → Lock 2
B → Lock 3
C → Lock 1
```

---

## Stage 3: Build wait-for relationships

```text
A waits for B.
B waits for C.
C waits for A.
```

---

## Stage 4: Detect cycle

```text
A → B → C → A
```

---

## Stage 5: Choose recovery

The service might:

* Cancel Thread C’s operation
* Roll back its transaction
* Release its resources safely
* Allow A and B to continue

---

## Stage 6: Record diagnostics

Useful information includes:

* Threads involved
* Locks held
* Locks requested
* Stack or operation state
* Time waiting
* Request identifiers

Detection is only useful if recovery preserves data consistency.

---

# 12.41 Deadlock in the Kernel

A deadlock inside kernel code can be especially serious.

Kernel threads or interrupt-related work may control:

* Process scheduling
* File systems
* Device operations
* Memory management

A kernel deadlock may cause:

* One device to stop responding
* One subsystem to freeze
* Many processes to wait
* Entire system unresponsiveness

Kernel synchronization therefore uses strict rules about:

* Lock order
* Whether sleeping is allowed
* Interrupt handling
* Execution context
* Maximum lock-hold time

Detailed kernel-locking internals are beyond the current foundation.

---

# 12.42 Interrupt-Related Deadlock Concept

Suppose ordinary kernel code:

1. Acquires Lock X.
2. Is interrupted.
3. The interrupt handler also attempts to acquire Lock X.

The interrupted code cannot release X until the handler finishes.

The handler cannot finish until X is released.

```text
Kernel code holds X
       │ interrupted
       ▼
Interrupt handler waits for X
       │
       ▼
Original code cannot resume to release X
```

Operating systems prevent this through carefully chosen lock types, interrupt masking rules, and strict context constraints.

This is one reason low-level kernel synchronization differs from ordinary application locking.

---

# 12.43 Deadlock and Resource Leaks

A resource leak does not automatically cause deadlock, but it can create similar symptoms.

Suppose a process opens all available connections and never releases them.

Other processes wait for a connection that will never become available.

```text
Leaking process holds every resource.
Other processes wait indefinitely.
```

There may be no circular dependency, so this is not necessarily a formal deadlock.

It is resource exhaustion caused by leaked ownership.

Correct diagnosis requires identifying the dependency structure.

---

# 12.44 Designing Safe Locking Rules

A practical locking design should document:

* Which shared state each lock protects
* Which invariants must hold
* The global lock-acquisition order
* Whether blocking operations are allowed while holding it
* Whether the lock may be acquired recursively
* Which thread owns and releases it
* How cancellation and errors release it
* Whether condition waits use it
* Maximum expected hold time

A lock without a clear protocol is easy to misuse.

---

# 12.45 Avoid Calling Unknown Work While Holding a Lock

Suppose Thread A holds a document lock and calls another component.

That component may:

* Acquire another lock
* Wait for I/O
* Call back into the original component
* Attempt to acquire the same lock
* Block indefinitely

```text
Hold Lock X
     │
Call unknown component
     │
Unknown dependency chain
```

This makes lock ordering difficult to reason about.

A safer design often:

1. Copies required state while locked.
2. Releases the lock.
3. Calls external or slow work.
4. Reacquires only when necessary.

Correctness must still be preserved when state may change during the unlocked period.

---

# 12.46 Recursive Locks

A **recursive lock** allows its owning thread to acquire the same lock multiple times.

Conceptually:

```text
Thread A acquires Lock X.
Thread A acquires Lock X again.
Thread A must release it twice.
```

---

## Why recursive locks exist

They can support nested operations that reuse the same protected component.

---

## Risk

They may hide poor ownership or call-structure design.

A normal non-recursive mutex would reveal that the thread attempted to acquire a lock it already owns.

Recursive locking does not solve deadlock involving different locks or different threads.

---

# 12.47 Cancellation and Lock Safety

A thread may be canceled, fail, or return early while holding a lock.

If the lock is not released:

```text
Thread exits
Lock remains owned
Other threads wait forever
```

Synchronization interfaces and language runtimes often provide structured cleanup mechanisms so lock release occurs even when operations fail.

Conceptually:

```text
Acquire lock
      │
Perform protected work
      │
Regardless of success or failure
      ▼
Release lock
```

Correct error paths are as important as successful paths.

---

# 12.48 Testing Concurrency Failures

Testing cannot explore every possible schedule, but it can improve detection.

Useful strategies include:

* Repeated stress testing
* Running with many CPU cores
* Running under heavy load
* Inserting controlled scheduling points
* Varying operation timing
* Using synchronization-analysis tools
* Recording lock dependencies
* Testing cancellation and failure paths
* Checking invariants continuously

---

## Why ordinary testing is insufficient

Suppose two threads each have 20 relevant steps.

The number of possible interleavings can be enormous.

The goal is therefore not only to observe failures.

The design must establish why harmful interleavings cannot occur.

---

# 12.49 What Can Go Wrong?

## Lock added around only part of an operation

The check is protected, but the update is not.

```text
Lock
Check condition
Unlock
Update state
```

Another thread may change the state between check and update.

---

## Inconsistent lock ordering

Different threads acquire the same locks in different orders.

Possible result:

* Circular waiting
* Deadlock

---

## Waiting while holding unrelated resources

A thread holds resources that other participants need while it waits for slow work.

---

## Forgotten release on an error path

The ordinary path releases a lock, but an exceptional path does not.

---

## Overly broad lock

One lock protects too much state.

Possible result:

* Contention
* Starvation
* Poor scalability

---

## Overly narrow lock

Related fields are protected separately even though their invariant spans all of them.

Possible result:

* Readers observe inconsistent combinations.

---

## Incorrect retry strategy

Participants repeatedly conflict and retry together.

Possible result:

* Livelock

---

## Unfair scheduling or locking

One participant waits indefinitely.

Possible result:

* Starvation

---

## Timeout treated as proof of deadlock

A slow operation may be incorrectly canceled.

Timeouts indicate delay, not necessarily a dependency cycle.

---

## Deadlock recovery corrupts state

A participant is terminated while holding partially updated shared data.

Recovery must include rollback or state repair.

---

## Race hidden by logging

Adding logging changes timing, causing the defect to disappear during investigation.

---

# 12.50 Common Misconceptions

## Misconception: “A race requires two CPU cores”

No.

One-core interleaving can produce races because context switches may occur between the steps of a logical operation.

---

## Misconception: “If individual reads and writes are atomic, the operation is thread-safe”

A multi-step sequence can still race.

```text
Atomic read + calculation + atomic write
```

does not guarantee atomicity of the entire update.

---

## Misconception: “Race conditions always crash programs”

Many races produce silent incorrect results rather than immediate crashes.

---

## Misconception: “A mutex automatically prevents every race involving the data”

Only participants following the same locking protocol are protected.

---

## Misconception: “A deadlocked program uses 100% CPU”

Deadlocked threads often sleep while waiting and may use almost no CPU.

Livelock or excessive spinning is more likely to consume high CPU.

---

## Misconception: “No CPU activity means deadlock”

The process may instead be:

* Waiting for legitimate I/O
* Waiting for user input
* Sleeping on a timer
* Starved
* Blocked by a failed external service

Deadlock requires a dependency cycle or equivalent permanent waiting structure.

---

## Misconception: “A timeout prevents deadlock”

A timeout can break indefinite waiting, but it does not necessarily prevent the underlying dependency from forming.

---

## Misconception: “Locking everything with one global lock is always the safest design”

It may simplify correctness, but it can cause severe contention, poor responsiveness, and indirect deadlocks when slow operations occur inside the lock.

---

## Misconception: “Deadlock always requires exactly two threads”

A deadlock cycle can involve many participants:

```text
A waits for B
B waits for C
C waits for D
D waits for A
```

---

## Misconception: “Starvation and deadlock are the same”

In starvation, other work continues.

In deadlock, the involved dependency group cannot progress.

---

## Misconception: “Livelock means threads are blocked”

Livelocked threads are active, but their repeated reactions prevent completion.

---

# 12.51 Real-World Analogy: One-Lane Bridge and Tool Workshop

## Race condition

Two gatekeepers separately check that a one-lane bridge is empty and both allow traffic from opposite sides.

The unsafe check-and-act timing causes a collision.

---

## Deadlock

Worker A holds the drill and waits for the saw.

Worker B holds the saw and waits for the drill.

Neither releases the held tool.

---

## Starvation

Worker C repeatedly waits for a popular machine because higher-priority workers always go first.

The workshop continues operating, but C never gets access.

---

## Livelock

Two workers repeatedly step aside in the same direction to let each other pass.

Both remain active but never move through the doorway.

---

# 12.52 Connection to Earlier Concepts

## Connection to threads and processes

Threads in one process commonly race through shared heap memory.

Processes may race through files, databases, devices, or shared memory.

---

## Connection to scheduling

A different scheduling order can expose or hide a race.

Threads waiting on locks move to waiting states.

---

## Connection to context switching

A context switch may occur between the check and update portions of one logical operation.

---

## Connection to synchronization

Mutexes, semaphores, condition variables, transactions, and message passing impose controlled ordering.

Incorrect synchronization can create deadlock.

---

## Connection to memory

A race may corrupt shared process memory.

Memory visibility rules also require proper synchronization.

---

## Connection to files

Two processes can overwrite each other’s changes even when every file-system request succeeds individually.

---

## Connection to I/O

Holding a lock during blocking I/O can form a deadlock or produce severe contention.

---

## Connection to security

TOCTOU races may let an attacker change a resource after validation but before use.

Race conditions can therefore become security vulnerabilities, not merely reliability defects.

---

# 12.53 Simplified Model Versus Technical Reality

## Simplified mental model

```text
Race condition:
Threads run in an unsafe order and produce the wrong result.

Deadlock:
Threads form a waiting cycle and cannot proceed.

Starvation:
One thread repeatedly loses access.

Livelock:
Threads remain active but accomplish nothing.
```

This is the model to retain.

---

## More exact reality

Real concurrency failures may involve:

* Hardware memory ordering
* Distributed services
* Database transactions
* Network timeouts
* Kernel interrupt contexts
* Recursive dependencies
* Dynamic resource graphs
* File-system names
* Process termination
* Device queues
* Lock-free retry loops
* External services that fail permanently

Not every long wait is a deadlock.

Not every unexpected result is a race.

Correct diagnosis requires identifying:

1. What state or resource is involved?
2. Who owns it?
3. Who is waiting?
4. What ordering was assumed?
5. Can any participant still make progress?
6. Is there a dependency cycle?

---

# 12.54 Core Mental Models

## Race-condition model

```text
Thread A checks shared state
             │
             │ Thread B changes it
             ▼
Thread A acts using an outdated assumption
             │
             ▼
Incorrect result
```

---

## Deadlock model

```text
Thread A holds X and waits for Y
                  ▲         │
                  │         ▼
Thread B waits for X and holds Y
```

---

## Starvation model

```text
Resource repeatedly granted to others
              │
              ▼
One ready participant waits indefinitely
```

---

## Livelock model

```text
Participants detect conflict
          │
          ▼
Both change behavior
          │
          ▼
Conflict repeats
          │
          └─────────── loops
```

---

## Prevention summary

| Problem        | Common preventive approach                          |
| -------------- | --------------------------------------------------- |
| Race condition | Protect the complete logical operation              |
| Deadlock       | Consistent resource order and limited hold-and-wait |
| Starvation     | Fairness or aging                                   |
| Livelock       | Backoff, priority, or central coordination          |

---

# Learning Check

Do not look for answers yet.

## Conceptual questions

1. What is the difference between a race condition and a data race?
2. What four conditions are commonly associated with resource deadlock?
3. How do deadlock, starvation, and livelock differ in terms of system progress?

## Cause-and-effect questions

4. Why can adding locks to remove a race condition create a new deadlock risk?
5. Why does requiring every thread to acquire locks in one global order prevent circular waiting?

## Misconception question

6. A student says, “Two threads on a one-core CPU cannot have a race condition because only one thread executes at a time.” What is wrong with this explanation?

## Scenario-based question

7. Thread A holds Lock X and waits for Lock Y. Thread B holds Lock Y and waits for Lock X. Meanwhile, Thread C repeatedly tries and fails to acquire Lock X but remains runnable. Identify the state of each thread, explain the deadlock cycle, and distinguish Thread C’s situation from the deadlocked pair.

# 13. Security, Permissions, and Process Isolation

An operating system runs software that may be:

* Correct
* Defective
* Poorly designed
* Compromised
* Intentionally malicious

The OS cannot safely assume that every process will behave properly.

It therefore applies security rules that answer:

1. **Who is making this request?**
2. **What resource are they requesting?**
3. **What action do they want to perform?**
4. **Is that action permitted?**
5. **How can damage be contained if something goes wrong?**

```text
Process requests an operation
            │
            ▼
Kernel identifies the process
            │
            ▼
Kernel checks permissions and policy
       ┌────┴────┐
       │         │
    Allowed    Denied
       │         │
Perform work   Return error
```

Three foundational ideas are:

| Concept            | Main purpose                              |
| ------------------ | ----------------------------------------- |
| **Authentication** | Establish an identity                     |
| **Authorization**  | Decide what that identity may do          |
| **Isolation**      | Limit which resources a process can reach |

---

# 13.1 What Operating-System Security Tries to Protect

The OS protects several kinds of resources.

## Data

Examples:

* Personal documents
* Password databases
* Browser information
* Application settings
* System configuration
* Encryption keys

---

## Processes and memory

The OS attempts to prevent one process from freely:

* Reading another process’s private memory
* Modifying another process’s instructions
* Controlling another process
* Inspecting protected kernel memory

---

## Devices

Sensitive devices include:

* Camera
* Microphone
* Keyboard
* Location sensors
* Raw storage
* Network adapters

---

## System authority

The OS protects operations such as:

* Installing system software
* Changing user accounts
* Reconfiguring security settings
* Loading privileged drivers
* Shutting down services
* Modifying operating-system files

---

## Availability

Security also includes keeping services usable.

A process should not be able to consume unlimited:

* CPU time
* Memory
* Storage space
* File descriptors
* Processes
* Network capacity

Protection against resource exhaustion is part of system security and reliability.

---

# 13.2 Security Goals

A common security model uses three broad goals.

## Confidentiality

Information should be visible only to authorized users and processes.

Example:

```text
Private document
      │
      ├── Owner may read
      └── Unrelated process denied
```

---

## Integrity

Information and system state should be changed only in authorized ways.

Example:

* A normal application may read an OS library.
* It may not silently replace that library.

---

## Availability

Authorized users and applications should be able to access resources when needed.

Example threats include:

* A process consuming all memory
* A service being permanently locked
* Storage being filled intentionally
* Network requests overwhelming a server

---

## Comparison

| Goal            | Protects against          |
| --------------- | ------------------------- |
| Confidentiality | Unauthorized reading      |
| Integrity       | Unauthorized modification |
| Availability    | Unauthorized disruption   |

Security mechanisms often protect more than one goal at once.

---

# 13.3 Authentication

## What it is

**Authentication** is the process of establishing or verifying an identity.

An identity might represent:

* A human user
* A system service
* An application
* A remote computer
* A device

Examples of authentication evidence include:

* Password
* Security key
* Biometric measurement
* Cryptographic credential
* Login token

---

## Why authentication exists

The OS cannot apply user-specific permissions without knowing which identity is active.

For example:

```text
Who is requesting report.txt?
        │
        ├── Owner
        ├── Another user
        └── System service
```

The authorization result may differ for each identity.

---

## Authentication is not authorization

Authentication answers:

> Who are you?

Authorization answers:

> What are you allowed to do?

A user may authenticate successfully but still lack permission to read a particular file.

---

# 13.4 User Accounts and Security Identities

An operating system commonly assigns each user account an internal identity.

This may be represented using:

* A numeric identifier
* A security identifier
* A set of credentials
* Group memberships
* Security tokens

The visible username is a human-friendly label.

```text
Visible name: Om
        │
        ▼
Internal OS identity
        │
        ▼
Permissions and ownership checks
```

The kernel relies on internal credentials rather than merely trusting text supplied by applications.

---

# 13.5 Process Credentials

A process runs with a security identity and related credentials.

These may include:

* User identity
* Group identities
* Privileges
* Security labels
* Sandbox restrictions
* Application identity
* Session information

```text
Process
├── Memory space
├── Threads
├── Open resources
└── Security credentials
```

When the process makes a system call, the kernel uses these credentials to evaluate the request.

---

## Why credentials belong to processes

The OS needs to know which authority applies to each operation.

Suppose two processes request the same file:

```text
Process A credentials → file read allowed
Process B credentials → file read denied
```

The file path is identical, but the requesting identities differ.

---

# 13.6 Authorization

## What it is

**Authorization** is the decision about whether an identified subject may perform a particular action on a resource.

The three main parts are:

```text
Subject → Action → Object
```

Example:

```text
Subject: Process running as User A
Action:  Write
Object:  report.txt
```

The kernel checks whether this combination is allowed.

---

## Why authorization exists

A user or process may need access to some resources but not others.

For example:

| Subject        | Resource         | Action | Result  |
| -------------- | ---------------- | ------ | ------- |
| Document owner | Own document     | Write  | Allowed |
| Other user     | Private document | Read   | Denied  |
| Music player   | Audio device     | Output | Allowed |
| Music player   | Raw disk         | Modify | Denied  |

Authorization limits authority to appropriate operations.

---

# 13.7 Permissions

## What they are

**Permissions** are rules describing which actions are allowed on a resource.

Common actions include:

* Read
* Write
* Execute
* Create
* Delete
* List
* Traverse
* Configure
* Control

Different resources support different permission types.

---

## File permission example

```text
File: notes.txt

Owner:
- Read: yes
- Write: yes

Other users:
- Read: no
- Write: no
```

---

## Device permission example

```text
Application:
- Camera access: denied
- Microphone access: allowed
```

---

## Process permission example

```text
Process A:
- May inspect itself
- May not inspect protected Process B
```

Permissions are interpreted by the kernel or another trusted system component.

---

# 13.8 Ownership

Resources commonly have an **owner**.

Examples:

* A file owned by a user
* A process associated with a user
* A device session controlled by an application
* A shared object owned by a service

Ownership may grant authority such as:

* Changing permissions
* Reading or writing
* Deleting the resource
* Delegating access

Ownership rules differ by resource and operating system.

---

## Ownership is not always complete control

A file owner may still be restricted by:

* System-wide policy
* Mandatory security labels
* Read-only storage
* Application sandboxing
* Administrative controls
* File-system capabilities

Ownership is one input to authorization, not necessarily the only one.

---

# 13.9 Groups and Roles

Managing permissions individually for every user can become difficult.

Operating systems therefore allow identities to be grouped.

```text
Group: project-team
├── User A
├── User B
└── User C
```

A resource can then grant access to the group.

```text
Project directory:
project-team may read and write
```

---

## Why groups exist

Groups simplify permission management:

* Add a user to the group.
* The user receives the group’s permitted access.
* Remove the user when access is no longer needed.

Roles serve a similar conceptual purpose by associating authority with responsibilities.

---

# 13.10 Access-Control Lists

An **access-control list**, or **ACL**, records which identities may perform which actions on a resource.

Conceptually:

```text
File ACL
├── User A: read, write
├── User B: read
├── Backup service: read
└── Everyone else: no access
```

ACLs provide finer control than simple owner/group/other categories.

---

## Why ACLs exist

Real access requirements may be specific.

For example:

* One user owns a document.
* A colleague may edit it.
* A reviewer may only read it.
* A backup service may copy it.
* Everyone else is denied.

An ACL can express these different rules.

---

# 13.11 Capabilities

A **capability** is an unforgeable reference or token that grants authority to a particular resource or operation.

A file descriptor can be understood partly as a capability-like reference.

```text
Process holds descriptor 5
          │
          ▼
Kernel recognizes authority
to use one already-opened resource
```

The process cannot usually invent an arbitrary descriptor value and gain access. The kernel checks its descriptor table.

---

## Why capabilities are useful

They allow authority to be:

* Narrow
* Specific
* Transferable in controlled ways
* Independent of repeated path lookup
* Revoked by closing or invalidating the reference

---

## Name versus capability

A pathname says:

> Locate whatever object this name currently refers to.

A stable open descriptor says:

> Use this already-authorized object reference.

This can reduce some time-of-check to time-of-use problems.

---

# 13.12 Privileges

Some sensitive operations require special authority beyond ordinary resource permissions.

Examples:

* Change system time
* Configure network interfaces
* Load drivers
* Manage user accounts
* Access raw devices
* Inspect unrelated processes

This authority may be represented as:

* Administrator access
* Root authority
* Specific privileges
* Capabilities
* Entitlements
* Security tokens

---

## Broad authority versus narrow authority

A broad administrator identity may be allowed to perform many sensitive actions.

A more restrictive design grants only the exact needed privilege.

```text
Broad authority:
May control almost every system resource

Narrow authority:
May bind one network service
but may not modify user files
```

Narrow authority reduces the damage possible from compromise.

---

# 13.13 Administrator Access Is Not Kernel Mode

This distinction must remain clear.

## Administrator or root

An operating-system identity with extensive authorization.

## Kernel mode

A CPU privilege level used while executing trusted kernel code.

An administrator’s application normally runs in user mode.

```text
Administrator application
        │ user mode
        ▼
System call
        │
        ▼
Kernel checks elevated credentials
        │ kernel mode
        ▼
Kernel performs approved operation
```

The application does not freely execute its own instructions in kernel mode.

---

# 13.14 Least Privilege

## What it is

The **principle of least privilege** says:

> A user, process, or component should receive only the authority required to perform its task.

---

## Why it exists

Suppose a music player only needs:

* Read access to selected music files
* Audio output access
* Possibly network access

It should not automatically receive authority to:

* Modify system files
* Read every private document
* Access the camera
* Manage user accounts
* Load kernel drivers

---

## Problem it solves

If the music player contains a vulnerability, limited permissions reduce the attacker’s possible actions.

```text
Compromised application
        │
        ▼
Can only use granted permissions
        │
        ▼
Damage is limited
```

Least privilege does not prevent every attack, but it reduces impact.

---

# 13.15 Privilege Separation

A complex application can be divided into components with different authority.

Example:

```text
User interface process
        │ limited permissions
        ▼
Privileged helper
        │ narrow controlled interface
        ▼
Sensitive system operation
```

The large user-facing component remains unprivileged.

A smaller helper performs only selected privileged work.

---

## Why privilege separation exists

Large programs contain more code and therefore more possible defects.

By keeping privileged code small:

* Less code can perform sensitive operations.
* Security review becomes easier.
* A defect in the unprivileged component causes less damage.
* The privileged interface can validate requests carefully.

---

# 13.16 Process Isolation

## What it is

**Process isolation** is the combination of hardware and OS mechanisms that separates one process’s resources and authority from another’s.

A process normally has:

* Its own virtual address space
* Its own descriptor table
* Its own credentials
* Its own thread state
* Controlled communication mechanisms

```text
Process A
├── Memory A
├── Descriptors A
└── Credentials A

Process B
├── Memory B
├── Descriptors B
└── Credentials B
```

The kernel mediates access between them.

---

## Why process isolation exists

Applications are not perfectly reliable.

Without isolation, one application failure could directly corrupt:

* Other applications
* The kernel
* User credentials
* Open documents
* System services

Isolation contains failures and limits unauthorized access.

---

# 13.17 Memory Isolation

Each process normally receives separate virtual-memory mappings.

```text
Process A virtual page 10 → Physical frame 50
Process B virtual page 10 → Physical frame 91
```

Process A cannot normally use its own virtual address to reach Process B’s frame.

The MMU enforces the mappings configured by the kernel.

---

## What happens on an illegal access

1. A process executes a memory instruction.
2. The MMU checks its mappings and permissions.
3. The access is forbidden.
4. The CPU raises an exception.
5. The kernel investigates.
6. The process may be terminated.

```text
Process attempts forbidden memory access
                │
                ▼
MMU blocks access
                │
                ▼
Kernel receives exception
                │
                ▼
Reject, report, or terminate
```

This is isolation enforced by hardware, not merely a convention.

---

# 13.18 File and Resource Isolation

Separate processes also have separate resource tables.

For example:

```text
Process A descriptor 4 → private document
Process B descriptor 4 → network socket
```

Process B cannot use A’s descriptor merely by supplying the same number.

The kernel interprets each number in the calling process’s own descriptor table.

---

## Why this matters

A process should not gain access to another process’s files or connections by guessing small integer identifiers.

The identifier only works when backed by a valid entry in that process’s kernel-managed table.

---

# 13.19 Communication Is Explicit

Processes generally communicate through OS-approved mechanisms, such as:

* Pipes
* Message queues
* Sockets
* Shared memory
* Signals or events
* Files
* Service interfaces

```text
Process A
    │
    │ approved IPC mechanism
    ▼
Kernel or system service
    │
    ▼
Process B
```

The communication channel can carry:

* Data
* Notifications
* Requests
* Resource references

The OS can apply permissions to the channel.

---

# 13.20 Sandboxing

## What it is

A **sandbox** is a restricted execution environment that limits what an application can access, even if the user running it has broader permissions.

A sandbox may restrict:

* Files
* Devices
* Network access
* Process creation
* System calls
* Inter-process communication
* Shared resources
* System configuration

---

## Why sandboxing exists

A user may legitimately have access to many sensitive resources.

That does not mean every application should inherit all of that authority.

Example:

```text
User can read:
- Documents
- Photos
- Password manager data

Downloaded game sandbox can read:
- Its own files
- User-selected files only
```

Sandboxing separates user authority from application authority.

---

## Mental model: supervised workroom

A contractor enters a building.

The contractor is allowed into one workroom containing the necessary tools.

They cannot freely enter:

* Payroll office
* Security room
* Executive offices
* Server room

The workroom is a sandbox.

---

# 13.21 Application Permissions

Modern systems may ask users whether an application may access:

* Camera
* Microphone
* Photos
* Contacts
* Location
* Notifications
* Nearby devices
* Local network

These are application-level authorization decisions.

```text
Application requests camera
          │
          ▼
OS checks stored policy
     ┌────┴────┐
     │         │
 Allowed    Not decided
     │         │
Open device  Ask user
```

---

## Permission prompt limitations

A prompt is only useful when the user understands:

* Which application is asking
* Which resource is involved
* Why it is needed
* Whether access is temporary or ongoing

Users may approve excessive permissions without understanding the consequences.

Permission prompts are one defense, not a complete security system.

---

# 13.22 Mandatory and Discretionary Access Control

Two broad access-control styles are useful conceptually.

## Discretionary access control

A resource owner can decide who receives access.

Example:

```text
Document owner grants a colleague read permission.
```

---

## Mandatory access control

System-wide policy or security labels impose restrictions that ordinary owners cannot freely override.

Example:

```text
A confidential file may only be read
by processes with a matching security label.
```

---

## Why mandatory controls exist

An owner may make a mistake or a compromised application may act with the owner’s authority.

Mandatory controls establish additional boundaries.

Real operating systems may combine several security models.

---

# 13.23 Security Labels

A system may assign labels to:

* Processes
* Files
* Devices
* Communication channels

A policy compares the process label with the resource label.

```text
Process label: restricted-app
File label: system-secret
Policy result: deny access
```

Labels can express rules that are not captured by simple user ownership.

---

# 13.24 Executable Permissions

Operating systems distinguish between:

* Reading a file’s data
* Modifying the file
* Executing it as a program

A file may be readable but not executable.

```text
File permissions:
Read:    yes
Write:   no
Execute: no
```

---

## Why execution is separate

Executing a file causes the CPU to treat its contents as program instructions.

That can have greater security impact than merely reading it.

The OS may also apply rules involving:

* File origin
* Digital signatures
* Application trust
* Sandboxing
* User confirmation
* Security policy

---

# 13.25 Code Signing: Conceptual Introduction

A **digital signature** can provide evidence that software was signed by a particular publisher and has not been altered since signing.

Conceptually:

```text
Software package
      │
      ▼
Cryptographic verification
      │
      ├── Signature valid
      └── Signature invalid
```

---

## What code signing can help establish

* Who signed the software
* Whether signed contents changed
* Whether the signer is trusted by policy

---

## What it does not prove

A valid signature does not guarantee that software:

* Has no bugs
* Is not malicious
* Uses permissions responsibly
* Cannot later be compromised

It establishes identity and integrity evidence, not perfect safety.

---

# 13.26 Secure Boot and Trusted Startup: Preview

During startup, the system may verify software before allowing it to control the machine.

A simplified chain is:

```text
Firmware verifies boot component
          │
          ▼
Boot component verifies kernel
          │
          ▼
Kernel verifies selected drivers or modules
```

This creates a **chain of trust**.

The full boot process appears later.

---

# 13.27 Resource Limits

Isolation includes limiting resource consumption.

A process may have limits on:

* Memory
* CPU time
* Number of threads
* Number of open files
* Storage usage
* Network bandwidth
* Process creation
* Device access

---

## Why resource limits exist

Without limits, one process could accidentally or intentionally exhaust system capacity.

Example:

```text
Process creates threads continuously
        │
        ▼
Kernel records and stacks consumed
        │
        ▼
Other applications cannot create threads
```

Resource limits protect availability.

---

# 13.28 CPU Isolation and Scheduling

The scheduler prevents one ordinary process from permanently controlling a CPU core.

A timer interrupt allows the kernel to regain control.

```text
Process runs
    │
    │ timer interrupt
    ▼
Kernel scheduler
    │
    ▼
Another process may run
```

Priorities and quotas can further control CPU access.

---

# 13.29 Memory Limits

A process may be restricted from consuming excessive memory.

When it exceeds a limit, possible outcomes include:

* Allocation failure
* Throttling
* Process termination
* Application-specific error

Memory limits are commonly used in:

* Sandboxes
* Containers
* Service hosting
* Multi-user systems

---

# 13.30 Storage Quotas

A **storage quota** limits how much persistent storage an identity or process group may consume.

```text
User quota: 10 GB
Used:       9.8 GB
New write:  500 MB
Result:     may fail
```

Quotas prevent one user from filling shared storage.

---

# 13.31 Process Creation Limits

A malicious or defective program might repeatedly create new processes.

```text
Process creates 2 children
Each child creates 2 more
Growth continues rapidly
```

This can exhaust:

* Process identifiers
* Memory
* Scheduler capacity
* Kernel records
* File descriptors

The OS can impose process or task limits.

---

# 13.32 Network Isolation

Processes may be restricted in their network access.

Possible policies include:

* No network access
* Local connections only
* Access only to selected destinations
* Incoming connections denied
* Specific ports permitted
* Traffic filtered by a firewall

Network isolation reduces communication with untrusted or sensitive systems.

---

# 13.33 Firewalls

A **firewall** applies rules to network traffic.

Conceptually:

```text
Network packet
      │
      ▼
Firewall rules
  ┌───┴────┐
  │        │
Allow     Block
```

Rules may consider:

* Source
* Destination
* Protocol
* Port
* Connection state
* Application identity
* Network interface

A firewall controls traffic. It does not automatically make allowed applications secure.

---

# 13.34 Process Inspection and Debugging Permissions

Debugging often requires powerful access, such as:

* Reading another process’s memory
* Pausing its threads
* Changing execution state
* Inspecting registers
* Modifying memory

These capabilities can expose secrets or take control of the process.

The OS therefore restricts debugging access.

```text
Debugger requests target process access
             │
             ▼
Kernel checks identity and policy
        ┌────┴────┐
        │         │
     Allowed    Denied
```

---

# 13.35 Inter-Process Communication Permissions

Communication channels can expose sensitive actions.

Suppose a privileged service accepts messages from applications.

The service must check:

* Who sent the request?
* Is the requested action allowed?
* Are the arguments valid?
* Does the resource belong to the sender?
* Can the response reveal private data?

A secure channel alone does not make every request authorized.

---

# 13.36 The Confused Deputy Problem

A **confused deputy** occurs when a privileged component is tricked into using its authority on behalf of an unauthorized requester.

---

## Example

A service can read protected files for legitimate system purposes.

A malicious process sends:

> Please read this protected file and return its contents.

If the service checks only its own permission, the operation succeeds.

The service had authority, but it used that authority for the wrong requester.

---

## Mental model

```text
Unprivileged requester
        │ deceptive request
        ▼
Privileged service
        │ uses its own authority
        ▼
Protected resource exposed
```

---

## Prevention principles

The service should validate:

* Requester identity
* Requester authority
* Exact requested resource
* Purpose and scope
* Whether authority was explicitly delegated

Privileges should not be applied blindly.

---

# 13.37 Environment and Inherited Resources

When a process starts, it may inherit information and resources from its parent.

Examples include:

* Open descriptors
* Environment settings
* Current directory
* Security context
* Communication channels

Inheritance can be useful, but it can accidentally grant authority.

```text
Parent owns sensitive descriptor
          │
          │ process creation
          ▼
Child inherits descriptor unintentionally
```

The child may now access a resource it was not meant to use.

Secure process creation requires controlling what is inherited.

---

# 13.38 Secrets in Process Memory

Applications may hold sensitive information in memory:

* Passwords
* Authentication tokens
* Encryption keys
* Private messages
* Personal data

Process isolation helps prevent unrelated processes from reading this memory.

However, secrets may still be exposed through:

* Application bugs
* Debugging access
* Crash reports
* Swap or paging storage
* Core dumps
* Logs
* Excessive permissions
* Kernel vulnerabilities

Security requires managing the entire data lifetime.

---

# 13.39 Secure Deletion Is Difficult

Deleting a file usually removes its name and allows storage blocks to be reused.

It does not necessarily guarantee immediate physical erasure.

Reasons include:

* File-system caching
* Journals
* Snapshots
* Backups
* Storage-controller behavior
* Copy-on-write file systems
* Wear-leveling in SSDs

```text
Delete file
    │
    ▼
Directory entry removed
    │
    ▼
Storage marked reusable
```

This is not always the same as:

```text
Every physical copy immediately destroyed
```

Encryption and key management are often more reliable foundations for protecting deleted sensitive data.

---

# 13.40 Encryption: Conceptual Introduction

**Encryption** transforms readable information into a form that requires a secret key to interpret.

```text
Readable data
     │ encryption with key
     ▼
Unreadable encrypted data
     │ decryption with key
     ▼
Readable data
```

Operating systems may support encryption for:

* Entire storage devices
* User directories
* Individual files
* Network traffic
* Credentials

---

## What encryption protects

Encryption can protect confidentiality when:

* A storage device is stolen
* Network traffic is intercepted
* Backups are accessed without authorization

---

## What encryption does not automatically protect

If an authorized process has access to the decryption key and is compromised, it may still read the data.

Encryption does not replace:

* Process isolation
* Permission checks
* Secure applications
* Authentication

---

# 13.41 Trusted Computing Base

The **trusted computing base**, or **TCB**, is the set of components whose correct behavior is necessary for system security.

It may include:

* CPU protection mechanisms
* Kernel
* Security-critical drivers
* Authentication services
* Permission databases
* Cryptographic key services
* Trusted firmware

---

## Why the TCB matters

If a security-critical component is compromised, it may bypass protections applied elsewhere.

A smaller TCB is generally easier to:

* Review
* Test
* Verify
* Protect

This is one reason privilege separation is valuable.

---

# 13.42 Attack Surface

The **attack surface** is the collection of interfaces through which an attacker might interact with or influence a system.

Examples include:

* System calls
* Network services
* File parsers
* Device drivers
* Login interfaces
* Browser content
* Inter-process messages
* Removable devices

---

## Why reducing attack surface helps

Every exposed interface may contain defects.

Reducing unnecessary interfaces lowers the number of possible entry points.

```text
Large exposed interface
├── Feature A
├── Feature B
├── Feature C
└── Feature D

Disable unused features
        │
        ▼
Smaller attack surface
```

---

# 13.43 Security Boundaries

A **security boundary** separates components with different trust or authority.

Examples include:

* User mode versus kernel mode
* One process versus another
* Sandbox versus host system
* User account versus administrator account
* Local machine versus network
* Virtual machine versus hypervisor

A boundary is meaningful only if it is enforced.

---

## Boundary enforcement may involve

* CPU privilege levels
* Page-table permissions
* Kernel authorization checks
* Cryptographic verification
* Network filtering
* Resource limits
* Device access controls

---

# 13.44 Defense in Depth

**Defense in depth** means using several independent protections rather than relying on one mechanism.

Example:

```text
Untrusted application
      │
      ├── Runs in user mode
      ├── Has separate address space
      ├── Has limited file permissions
      ├── Runs in a sandbox
      ├── Has no camera access
      └── Has restricted network access
```

If one layer fails, another may still limit damage.

---

## Why one defense is insufficient

A memory-isolated process could still delete every file it is authorized to modify.

A sandboxed process could still exploit a kernel bug.

An encrypted disk does not stop an authorized malicious application after login.

Security mechanisms address different threats.

---

# 13.45 Step-by-Step: Reading a Protected File

Suppose a process attempts to read:

```text
Private/financial-records.txt
```

## Step 1: Application prepares the request

It supplies a pathname and requests read access.

**Component:** Application
**Mode:** User mode

---

## Step 2: Open system call enters the kernel

The CPU switches into kernel mode through a controlled entry point.

---

## Step 3: Kernel identifies the process

The kernel obtains the process’s credentials:

* User identity
* Groups
* Privileges
* Sandbox state
* Security labels

---

## Step 4: Kernel resolves the path

It traverses each directory component.

Directory traversal permissions may be checked along the way.

---

## Step 5: Kernel finds the file metadata

The file system provides:

* Owner
* Permission entries
* ACLs
* Security labels
* File type

---

## Step 6: Authorization policy is evaluated

Conceptually:

```text
Subject:
Calling process credentials

Action:
Read

Object:
financial-records.txt
```

---

## Step 7A: Access is allowed

The kernel creates open-file state and returns a descriptor.

The process may then request file data.

---

## Step 7B: Access is denied

The kernel does not create a usable descriptor.

It returns an error such as permission denied.

```text
Open request
     │
     ▼
Permission check
  ┌──┴───┐
  │      │
Allow   Deny
  │      │
Return  Return
handle  error
```

The application cannot bypass the result by directly controlling the storage device because raw device access is also protected.

---

# 13.46 Step-by-Step: Application Requests Camera Access

Suppose a video-chat application wants to use the camera.

## Stage 1: Application requests camera service

The request enters an OS device or media interface.

---

## Stage 2: OS identifies the application

The system may consider:

* Application identity
* User identity
* Sandbox
* Stored permission state
* Whether the application is active
* Enterprise or system policy

---

## Stage 3: Permission is checked

Possible states:

```text
Allowed
Denied
Not yet decided
Restricted by policy
```

---

## Stage 4: User may be asked

If policy allows user choice, the OS presents a permission decision.

---

## Stage 5A: Permission granted

The OS creates a controlled camera session.

The application does not receive unrestricted controller access.

```text
Application
    │
    ▼
Camera service or kernel interface
    │
    ▼
Camera driver
    │
    ▼
Camera hardware
```

---

## Stage 5B: Permission denied

The application receives an error or unavailable result.

---

## Stage 6: OS may show an indicator

Some systems indicate that the camera is active.

This improves transparency but does not replace permission enforcement.

---

## Stage 7: Session ends

When the application closes the camera or terminates, the OS releases access and device resources.

---

# 13.47 Step-by-Step: A Process Attempts to Read Another Process’s Memory

Suppose Process A tries to read Process B’s private data using an arbitrary address.

## Step 1: Process A executes a memory-read instruction

The address is interpreted within Process A’s virtual address space.

---

## Step 2: MMU uses Process A’s page tables

It does not automatically use Process B’s mappings.

---

## Step 3: Address is either unmapped or refers to A’s own memory

Process B’s private physical frames are not reachable through ordinary A mappings.

---

## Step 4: Invalid access causes an exception

The CPU enters the kernel.

---

## Step 5: Kernel rejects the access

Process A may receive a fatal memory error.

```text
Process A virtual address
            │
            ▼
Process A page table
            │
            ✕ no mapping to Process B private frame
```

---

## Special debugging case

A debugger may request controlled access to another process.

The kernel then checks explicit debugging permissions before providing that access.

The exception is authorized through an OS interface, not achieved through arbitrary memory reads.

---

# 13.48 Step-by-Step: Starting an Application With Reduced Privileges

Suppose the OS launches an untrusted document viewer in a sandbox.

## Stage 1: Program is selected

The system identifies the executable and requested document.

---

## Stage 2: Security policy is assembled

The OS determines allowed resources:

* Read selected document
* Use display
* Receive user input
* No unrestricted file access
* No raw-device access
* Limited network access

---

## Stage 3: Process is created

The kernel creates:

* Virtual address space
* Descriptor table
* Threads
* Credentials
* Sandbox restrictions

---

## Stage 4: Only approved resources are provided

The process may receive:

* A descriptor for the selected document
* A graphics connection
* A restricted communication channel

It may not receive access to the user’s entire home directory.

---

## Stage 5: Application executes in user mode

The CPU and MMU enforce memory isolation.

The kernel checks system calls against the sandbox policy.

---

## Stage 6: Application is compromised

Suppose the document exploits a viewer defect.

The attacker gains control of the viewer process.

---

## Stage 7: Sandbox limits authority

The compromised process may still be restricted from:

* Reading unrelated files
* Starting arbitrary privileged processes
* Accessing the camera
* Changing system settings
* Reading other process memory

The exploit is serious, but the containment reduces damage.

---

# 13.49 Step-by-Step: What Happens When a Program Crashes

Security and isolation affect crash handling.

## Stage 1: Process performs an invalid operation

Examples:

* Illegal memory access
* Invalid instruction
* Failed security assertion

---

## Stage 2: Hardware or kernel detects the failure

Control enters the kernel.

---

## Stage 3: Kernel identifies the faulting process

The error is associated with one process and thread.

---

## Stage 4: Kernel prevents continued unsafe execution

The faulting thread or process is stopped.

---

## Stage 5: Process resources are reclaimed

The OS releases:

* Virtual memory
* Descriptors
* Device sessions
* Kernel records
* IPC references

---

## Stage 6: Other processes remain isolated

Their address spaces and resource tables remain separate.

---

## Stage 7: Diagnostic data may be recorded

Security policy may limit which memory appears in crash reports because process memory can contain secrets.

---

## Stage 8: External effects may remain

The OS cannot automatically undo:

* A partially saved file
* A network request already sent
* A database operation already committed
* Data already exposed

Process isolation limits direct corruption but does not reverse every external action.

---

# 13.50 Security Checks Occur at Several Layers

A request may pass through several checks.

Example: accessing a cloud-synchronized document.

```text
Application sandbox check
          │
          ▼
OS file permission check
          │
          ▼
File-system encryption access
          │
          ▼
Cloud service authentication
          │
          ▼
Remote authorization policy
```

Passing one layer does not guarantee passing all layers.

Each layer protects a different boundary.

---

# 13.51 What Can Go Wrong?

## Excessive permissions

An application receives more authority than it needs.

If compromised, the attacker gains that excess authority.

---

## Incorrect permission checks

A service checks the wrong identity, wrong resource, or wrong operation.

Possible result:

* Unauthorized data access
* Unauthorized modification
* Privilege escalation

---

## Missing check on one path

Most request paths verify permission, but one unusual path forgets.

Attackers search for inconsistent enforcement.

---

## Check performed too early

A resource changes between authorization and use.

This can create a TOCTOU race.

---

## Confused deputy

A privileged service uses its authority for an unauthorized requester.

---

## Inherited descriptor leak

A child process receives a sensitive open resource accidentally.

---

## Weak sandbox policy

The sandbox permits enough access for an attacker to escape meaningful containment without technically escaping the sandbox.

---

## Sandbox escape

A vulnerability in the kernel, system service, or sandbox mechanism allows the process to gain authority outside its restrictions.

---

## Kernel vulnerability

A user-mode process tricks the kernel into:

* Reading or writing unsafe memory
* Granting additional privilege
* Bypassing permissions
* Crashing the system

Because the kernel is part of the TCB, kernel defects can cross major security boundaries.

---

## Vulnerable driver

A privileged driver mishandles device input or application requests.

Possible consequences include:

* Kernel compromise
* Arbitrary memory access
* System crash
* Device misuse

---

## Resource exhaustion

A process consumes excessive:

* Memory
* CPU
* Storage
* Threads
* Descriptors
* Network connections

Even without reading protected data, it can damage availability.

---

## Permission fatigue

Users receive too many prompts and approve them without careful review.

Prompts lose their value.

---

## Stale authorization

A process retains access after its role or permission should have been removed.

Open capabilities and long-lived sessions may need revocation.

---

## Secret leakage

Sensitive data appears in:

* Logs
* Temporary files
* Crash dumps
* Swap
* Clipboard
* Shared memory
* Error messages

---

## Incorrect cleanup

A process exits, but a shared service retains its credentials, files, or device session longer than intended.

---

# 13.52 Common Misconceptions

## Misconception: “Authentication and authorization are the same”

Authentication establishes identity.

Authorization decides permitted actions.

---

## Misconception: “Administrator programs run in kernel mode”

Administrator applications normally run in user mode.

The kernel performs privileged operations after checking their credentials.

---

## Misconception: “User mode means a process is harmless”

A user-mode process can still:

* Delete accessible files
* Steal data it is permitted to read
* Send data over the network
* Consume resources
* Exploit a kernel vulnerability

User mode limits authority; it does not guarantee safety.

---

## Misconception: “Process isolation means processes cannot communicate”

They can communicate through explicit OS-approved mechanisms.

Isolation prevents unrestricted access, not all communication.

---

## Misconception: “File permissions protect against every application running as the same user”

If several applications inherit the user’s full file authority, they may all access the same files.

Sandboxing and application-specific permissions provide additional separation.

---

## Misconception: “A permission prompt guarantees informed consent”

Users may misunderstand, approve automatically, or lack enough context.

Prompts must be combined with safe defaults and least privilege.

---

## Misconception: “A valid digital signature means software is safe”

It proves evidence about signer identity and integrity, not defect-free or non-malicious behavior.

---

## Misconception: “Encryption replaces permissions”

Encryption protects data without the key.

Once an authorized process has decrypted access, permissions and isolation still matter.

---

## Misconception: “Deleting a file securely destroys every copy”

Copies may remain in:

* Backups
* Journals
* Snapshots
* Storage-controller remapping
* Caches

Ordinary deletion mostly removes names and frees logical storage.

---

## Misconception: “One strong security boundary is enough”

Security uses defense in depth because any single mechanism may fail.

---

## Misconception: “A sandbox prevents all damage”

A sandbox only limits actions covered by its policy and enforcement.

A sandboxed process can still damage resources it is allowed to access.

---

## Misconception: “A process cannot affect another process because their memory is isolated”

Processes may still interact through:

* Files
* Services
* Shared memory
* Network connections
* Signals
* Device resources

Memory isolation is one boundary among several.

---

# 13.53 Real-World Analogy: Hotel Security

Imagine a hotel.

## Authentication

A guest shows identification at reception.

## Authorization

Reception decides which room and facilities the guest may use.

## Room key

A capability granting access to one room.

## Process isolation

Each guest has a separate locked room.

## Kernel

Hotel security and management enforce access rules.

## Administrator

A hotel manager with broad authority.

## Kernel mode

The secure building-control system, not merely the manager’s identity.

## Sandbox

A conference visitor may access only:

* Conference hall
* Assigned restroom
* Reception area

## Least privilege

A cleaning contractor receives access only to assigned rooms during a limited period.

## Privilege separation

Reception staff handle guest requests, while a smaller security team controls master keys.

## Resource limits

Each guest may book only a limited number of rooms or services.

## Defense in depth

The hotel uses:

* Identity checks
* Key cards
* Security cameras
* Locked service areas
* Staff procedures

No single control is trusted alone.

---

# 13.54 Connection to Earlier Concepts

## Connection to hardware

The CPU enforces privilege levels.

The MMU enforces memory-access permissions.

---

## Connection to the kernel

The kernel:

* Stores process credentials
* Checks permissions
* Manages resource tables
* Enforces sandbox rules
* Controls devices
* Handles violations

---

## Connection to user and kernel mode

User-mode restrictions prevent ordinary applications from directly changing security rules.

Kernel mode gives trusted code the authority to enforce them.

---

## Connection to processes

Processes provide:

* Memory isolation
* Resource ownership
* Credential boundaries
* Independent failure containment

---

## Connection to system calls

Every protected operation is requested through a controlled system interface.

The kernel checks the request before acting.

---

## Connection to virtual memory

Page-table permissions prevent one process from freely accessing another process or the kernel.

---

## Connection to files

File ownership, ACLs, groups, and security labels govern file operations.

---

## Connection to I/O

Device permissions protect:

* Cameras
* Microphones
* Storage
* Input hardware
* Network interfaces

---

## Connection to concurrency

Security checks must remain correct under concurrent changes.

TOCTOU races can invalidate a permission check between validation and use.

---

## Connection to crashes

Isolation allows the OS to terminate one faulty process while preserving other processes and kernel state.

---

# 13.55 Simplified Model Versus Technical Reality

## Simplified mental model

```text
Identity:
Who is making the request?

Permission:
May they perform this action?

Isolation:
What can the process reach?

Least privilege:
Give it only what it needs.
```

This is the model to retain.

---

## More exact reality

Modern operating systems may combine:

* User and group permissions
* Access-control lists
* Security labels
* Application identities
* Digital signatures
* Sandboxes
* Capabilities
* Privileges
* Mandatory policy
* Device entitlements
* Firewall rules
* Resource quotas
* Namespace isolation
* Cryptographic protection
* Audit systems

Authorization may occur in:

* Kernel code
* User-space services
* File systems
* Database servers
* Remote services
* Hardware security modules

Security is not one isolated subsystem.

It is a set of boundaries and policies applied throughout the system.

The central principle remains:

> The OS identifies each process, checks every protected request against policy, and limits the process’s memory, resources, devices, and authority.

---

# 13.56 Core Mental Models

## Authorization model

```text
Subject
Process credentials
      │
      ▼
Requests action
      │
      ▼
Object
File, device, process, service
      │
      ▼
Policy check
  ┌───┴────┐
  │        │
Allow     Deny
```

---

## Process-isolation model

```text
Process A                      Process B
┌──────────────────┐          ┌──────────────────┐
│ Memory A         │          │ Memory B         │
│ Descriptors A    │          │ Descriptors B    │
│ Credentials A    │          │ Credentials B    │
└──────────────────┘          └──────────────────┘
          │                            │
          └──────── Kernel ────────────┘
                 mediates access
```

---

## Least-privilege model

```text
Task requires:
- Read one document
- Display output

Granted:
- Read that document
- Use display

Not granted:
- Read all files
- Use camera
- Modify system settings
```

---

## Defense-in-depth model

```text
Untrusted input
      │
      ▼
Application validation
      │
      ▼
Process sandbox
      │
      ▼
Kernel permission check
      │
      ▼
Memory isolation
      │
      ▼
Resource limits
```

---

## Final distinctions

| Concept                    | Essential meaning                                           |
| -------------------------- | ----------------------------------------------------------- |
| **Authentication**         | Establish identity                                          |
| **Authorization**          | Decide permitted actions                                    |
| **Permission**             | Rule allowing or denying an action                          |
| **Credential**             | Security information associated with an identity or process |
| **Privilege**              | Special authority for sensitive operations                  |
| **Isolation**              | Prevent unrestricted access between components              |
| **Sandbox**                | Restricted process environment                              |
| **Least privilege**        | Grant only required authority                               |
| **Capability**             | Controlled reference granting specific authority            |
| **Confidentiality**        | Prevent unauthorized reading                                |
| **Integrity**              | Prevent unauthorized modification                           |
| **Availability**           | Preserve usable access to resources                         |
| **Trusted computing base** | Components whose correctness security depends on            |
| **Defense in depth**       | Use multiple independent protections                        |

The next section explains **virtual machines and containers**—two different ways to create isolated execution environments above the operating-system and hardware layers.

# Learning Check

Do not look for answers yet.

## Conceptual questions

1. What is the difference between authentication, authorization, and process isolation?
2. Why is an administrator identity different from kernel mode?
3. What does the principle of least privilege require?

## Cause-and-effect questions

4. Why can sandboxing an application reduce the damage of a successful application exploit?
5. Why can accidentally inheriting an open file descriptor give a child process unintended authority?

## Misconception question

6. A student says, “Because two processes have separate virtual address spaces, they cannot affect each other in any way.” What is wrong with this statement?

## Scenario-based question

7. A document viewer opens an untrusted file and is later compromised through a parsing defect. Explain how user mode, virtual-memory isolation, file permissions, sandbox restrictions, device permissions, resource limits, and kernel authorization checks can contain the damage.

# 14. Virtual Machines and Containers

Operating systems isolate ordinary processes, but sometimes stronger or more structured separation is needed.

Examples include:

* Running several operating systems on one computer
* Hosting unrelated customers on one server
* Packaging an application with its dependencies
* Testing software without changing the main system
* Restricting a service’s resource use
* Reproducing the same application environment on many machines

Two major approaches are:

| Approach            | Core idea                                                               |
| ------------------- | ----------------------------------------------------------------------- |
| **Virtual machine** | Provide virtual hardware on which a separate operating system runs      |
| **Container**       | Isolate groups of processes that share the host operating-system kernel |

```text
Physical hardware
        │
        ├── Virtual machines
        │   └── Each can run its own guest OS kernel
        │
        └── Containers
            └── Processes share the host kernel
```

The most important distinction is:

> A virtual machine contains a separate operating-system kernel.

> A container normally uses the host’s existing kernel.

---

# 14.1 Why Virtualization and Containers Exist

A physical computer contains finite resources:

* CPU cores
* RAM
* Storage
* Network devices
* Graphics or accelerator hardware

Traditionally, one operating system controlled the entire computer.

However, organizations often want several isolated environments to share the same physical machine.

Without virtualization, this might require:

```text
Service A → physical server A
Service B → physical server B
Service C → physical server C
```

This can waste resources when each server is lightly used.

Virtualization allows:

```text
One physical server
├── Environment A
├── Environment B
└── Environment C
```

Each environment can receive a controlled portion of the machine.

---

## Problems these technologies solve

Virtual machines and containers can help provide:

* Isolation
* Resource limits
* Repeatable environments
* Workload consolidation
* Easier deployment
* Testing environments
* Dependency separation
* Failure containment
* Administrative boundaries

They solve some of the same problems but at different layers.

---

# 14.2 Foundational Layer Model

Start with the ordinary physical system:

```text
Applications
     │
     ▼
Operating-system kernel
     │
     ▼
Physical hardware
```

A virtual machine adds a virtualization layer:

```text
Guest applications
        │
        ▼
Guest OS kernel
        │
        ▼
Virtual hardware
        │
        ▼
Hypervisor
        │
        ▼
Physical hardware
```

A container instead remains above the host kernel:

```text
Containerized applications
          │
          ▼
Host OS kernel
          │
          ▼
Physical hardware
```

---

# 14.3 What Is a Virtual Machine?

## What it is

A **virtual machine**, or **VM**, is an isolated computing environment that behaves sufficiently like a computer for an operating system to run inside it.

The VM may be presented with virtual versions of:

* CPU cores
* RAM
* Storage devices
* Network adapters
* Display hardware
* Firmware
* Interrupt controllers
* Other devices

```text
Virtual machine
┌────────────────────────────┐
│ Guest applications         │
├────────────────────────────┤
│ Guest operating system     │
├────────────────────────────┤
│ Virtual CPU and devices    │
└────────────────────────────┘
```

The operating system running inside the VM is called the **guest operating system**.

---

## Why virtual machines exist

A VM allows one physical machine to host several independent operating-system environments.

```text
Physical server
├── VM 1
│   └── Guest OS A
├── VM 2
│   └── Guest OS B
└── VM 3
    └── Guest OS C
```

The guests may:

* Run different operating systems
* Use different kernel versions
* Have different users
* Use different file systems
* Follow different security policies
* Restart independently

---

## Mental model: apartments in one building

A physical server is like an apartment building.

Each VM is a separate apartment with its own:

* Rooms
* Locks
* Electrical controls
* Residents
* Internal rules

The building still has shared physical foundations, electricity, and plumbing.

| Apartment building              | Virtualized computer |
| ------------------------------- | -------------------- |
| Building                        | Physical machine     |
| Apartment                       | Virtual machine      |
| Apartment utilities             | Virtual hardware     |
| Apartment residents             | Guest applications   |
| Apartment management            | Guest OS             |
| Building infrastructure manager | Hypervisor           |

Residents usually experience their apartment as an independent home even though the building is shared.

---

# 14.4 What Is a Hypervisor?

## What it is

A **hypervisor** is the software layer that creates and manages virtual machines.

It controls how VMs use physical resources such as:

* CPU
* Memory
* Storage
* Network interfaces
* Interrupts
* Devices

```text
VM A ─┐
VM B ─┼──▶ Hypervisor ──▶ Physical hardware
VM C ─┘
```

---

## What problem the hypervisor solves

Each guest OS behaves as though it controls a computer.

But several guests cannot safely control the same physical hardware directly.

The hypervisor must:

* Divide hardware resources
* Prevent one VM from accessing another VM’s memory
* Schedule virtual CPUs
* Present virtual devices
* Handle privileged guest operations
* Maintain isolation

---

# 14.5 Hypervisor Types

A common conceptual distinction is between two arrangements.

## Type 1: Hypervisor directly over hardware

```text
Virtual machines
       │
       ▼
Hypervisor
       │
       ▼
Physical hardware
```

The hypervisor acts as the main low-level system controlling the machine.

This arrangement is common in servers and data centers.

---

## Type 2: Hypervisor hosted by another OS

```text
Virtual machines
       │
       ▼
Virtualization application
       │
       ▼
Host operating system
       │
       ▼
Physical hardware
```

The virtualization software runs as an application or system component on a host OS.

This arrangement is common on personal computers used for testing or development.

---

## Simplified comparison

| Type 1                          | Type 2                                   |
| ------------------------------- | ---------------------------------------- |
| Hypervisor is close to hardware | Hypervisor uses a host OS                |
| Common in servers               | Common on desktops                       |
| Fewer host layers conceptually  | Easier integration with existing desktop |
| Designed for workload hosting   | Often designed for user convenience      |

Real implementations may not fit perfectly into one simple category.

---

# 14.6 Host and Guest

## Host

The **host** is the physical system or controlling environment providing resources to virtual machines.

## Guest

The **guest** is the operating system and software running inside a VM.

```text
Host system
└── Virtual machine
    └── Guest operating system
        └── Guest applications
```

A host may run several guests.

---

# 14.7 Virtual CPUs

A VM receives one or more **virtual CPUs**, often called vCPUs.

A guest scheduler treats these as processors on which guest threads can run.

```text
Guest ready threads
        │
        ▼
Guest scheduler
        │
        ▼
Virtual CPUs
        │
        ▼
Hypervisor scheduling
        │
        ▼
Physical CPU cores
```

This creates two scheduling levels:

1. The guest OS chooses which guest thread runs on a virtual CPU.
2. The hypervisor chooses when that virtual CPU runs on a physical CPU.

---

## Example

Suppose a VM has two virtual CPUs.

The guest may run:

```text
vCPU 1 → Guest Thread A
vCPU 2 → Guest Thread B
```

But the hypervisor may need to share four physical cores among many VMs.

A guest thread can be ready according to the guest scheduler while its virtual CPU is temporarily not scheduled by the hypervisor.

---

# 14.8 Virtual-Machine Memory

A guest OS believes it manages physical memory.

However, what the guest considers “physical RAM” is usually another level of managed mapping.

```text
Guest virtual address
        │
        ▼
Guest page tables
        │
        ▼
Guest physical address
        │
        ▼
Hypervisor-controlled mapping
        │
        ▼
Machine physical RAM
```

There may therefore be two translation stages.

---

## Guest virtual address

Used by an application inside the guest.

## Guest physical address

The address the guest kernel believes refers to RAM assigned to the VM.

## Machine physical address

The location in actual host hardware RAM.

---

## Why this extra translation exists

The guest OS must be able to manage memory normally while the hypervisor still isolates it from other VMs.

```text
VM A guest physical page 10
          │
          ▼
Machine physical frame 500

VM B guest physical page 10
          │
          ▼
Machine physical frame 900
```

Both guests can believe they own “physical page 10” without accessing the same machine frame.

---

# 14.9 Hardware Assistance for Virtualization

Modern CPUs commonly include features designed to support virtualization.

These features help the hypervisor:

* Run guest code efficiently
* Detect privileged guest operations
* Maintain additional memory translations
* Control interrupts
* Isolate virtual machines

---

## Why hardware assistance matters

Without hardware support, the hypervisor may need more complicated techniques to safely run guest kernels.

Hardware virtualization allows much guest code to execute directly on the physical CPU while preserving control.

```text
Ordinary guest instruction
          │
          ▼
Runs efficiently on CPU

Sensitive guest operation
          │
          ▼
Hardware transfers control
to hypervisor
```

The guest does not receive unrestricted control of the real machine.

---

# 14.10 Privileged Instructions Inside a VM

A guest kernel expects to execute privileged operations.

For example, it may try to:

* Change memory mappings
* Configure interrupt handling
* Control a device
* Modify processor state

On a physical system, these operations affect real hardware.

Inside a VM, the hypervisor must ensure they affect only the virtual machine.

```text
Guest kernel requests privileged operation
              │
              ▼
Virtualization hardware or hypervisor
              │
              ▼
Apply safe virtual effect
```

The guest believes it changed its machine state, while the hypervisor maintains the real boundary.

---

# 14.11 Virtual Devices

A VM may be presented with virtual devices such as:

* Virtual disk
* Virtual network adapter
* Virtual display
* Virtual keyboard
* Virtual USB controller

```text
Guest application
       │
       ▼
Guest device driver
       │
       ▼
Virtual device
       │
       ▼
Hypervisor or host I/O system
       │
       ▼
Physical device
```

The guest OS uses drivers for the virtual hardware just as it would use drivers for physical hardware.

---

## Device emulation

The hypervisor can imitate the behavior of a known physical device.

### Benefit

An existing guest driver may work without modification.

### Cost

Imitating detailed hardware behavior may add overhead.

---

## Paravirtualized device

A virtual device may be designed specifically for efficient communication between the guest and hypervisor.

The guest uses a virtualization-aware driver.

```text
Guest OS
   │ efficient virtual-device protocol
   ▼
Hypervisor
```

This can reduce the overhead of imitating older physical hardware.

---

## Direct device assignment

In some configurations, a physical device or part of one may be assigned more directly to a VM.

### Benefit

Potentially high performance.

### Costs

* More complex isolation
* Reduced sharing
* Hardware requirements
* More difficult migration

---

# 14.12 Virtual Disks

A guest OS may see a virtual disk:

```text
Guest sees:
Virtual disk with blocks 0–N
```

The host may store that virtual disk as:

* A large host file
* A logical storage volume
* A network-backed volume
* A dedicated physical device
* A snapshot-based storage object

```text
Guest file
    │
    ▼
Guest file system
    │
    ▼
Virtual disk blocks
    │
    ▼
Hypervisor storage layer
    │
    ▼
Host file, volume, or device
```

This adds another storage-management layer.

---

# 14.13 Step-by-Step: Starting a Virtual Machine

Suppose a user starts a VM.

## Stage 1: VM configuration is selected

The configuration may describe:

* Number of virtual CPUs
* Amount of memory
* Virtual disks
* Network interfaces
* Firmware settings
* Attached devices

**Component:** Management application or service

---

## Stage 2: Hypervisor allocates VM resources

The hypervisor prepares:

* VM identity
* Virtual CPU state
* Memory mappings
* Virtual devices
* Security boundaries
* Scheduling information

---

## Stage 3: Virtual firmware begins

The VM starts as though a computer has powered on.

Virtual firmware performs initial setup and selects a boot device.

---

## Stage 4: Guest bootloader starts

The bootloader reads the guest operating-system kernel from the virtual disk.

---

## Stage 5: Guest kernel initializes

The guest kernel:

* Configures its virtual CPUs
* Sets up guest page tables
* Detects virtual devices
* Loads guest drivers
* Initializes its file systems
* Starts system services

---

## Stage 6: Guest user environment starts

The guest may show:

* Login screen
* Command interface
* Desktop
* Server services

---

## Stage 7: Guest applications run

Applications make system calls to the guest kernel.

The guest kernel controls the guest environment, while the hypervisor controls the underlying physical resources.

---

## Full flow

```text
Start VM request
      │
      ▼
Hypervisor creates virtual hardware
      │
      ▼
Virtual firmware starts
      │
      ▼
Guest bootloader loads guest kernel
      │
      ▼
Guest kernel initializes
      │
      ▼
Guest services and applications start
```

A VM therefore performs an operating-system boot process.

---

# 14.14 Step-by-Step: A File Read Inside a VM

Suppose an application inside a guest reads a file.

## Stage 1: Guest application makes a system call

The system call enters the **guest kernel**.

```text
Guest application
        │
        ▼
Guest kernel
```

It does not directly enter the host kernel’s ordinary application system-call interface.

---

## Stage 2: Guest kernel resolves the guest file

The guest file system checks:

* Guest pathname
* Guest permissions
* Guest descriptor
* Guest file-system metadata

---

## Stage 3: Guest kernel requests virtual-disk data

The guest storage driver sends a request to the virtual storage device.

---

## Stage 4: Hypervisor or host receives the virtual I/O request

The virtualization layer translates it into access to:

* A host file
* A host volume
* A physical disk
* Network storage

---

## Stage 5: Host storage stack performs physical I/O

The host kernel or hypervisor uses:

* Physical storage driver
* DMA
* Interrupts
* Device queues

---

## Stage 6: Completion travels back upward

```text
Physical storage completion
          │
          ▼
Host or hypervisor
          │
          ▼
Virtual storage device completion
          │
          ▼
Guest interrupt
          │
          ▼
Guest kernel
```

---

## Stage 7: Guest thread becomes ready

The guest scheduler eventually resumes the application thread.

---

## Stage 8: Guest system call returns

The application receives the file data.

---

## Layered flow

```text
Guest application
      │ system call
      ▼
Guest kernel and guest file system
      │ virtual-device request
      ▼
Hypervisor or host virtualization layer
      │ physical I/O request
      ▼
Host driver and physical storage
```

Each layer sees a different abstraction.

---

# 14.15 Virtual-Machine Isolation

A VM provides an isolation boundary around:

* Guest memory
* Guest virtual CPUs
* Guest devices
* Guest operating-system state
* Guest processes
* Guest users

```text
VM A                         VM B
┌────────────────────┐      ┌────────────────────┐
│ Guest applications │      │ Guest applications │
│ Guest kernel       │      │ Guest kernel       │
│ Guest memory       │      │ Guest memory       │
└────────────────────┘      └────────────────────┘
          │                         │
          └────── Hypervisor ───────┘
```

One guest should not be able to access another guest’s memory or devices without an explicitly configured channel.

---

## Why this can be stronger than ordinary process isolation

A compromised application inside a VM may compromise its guest kernel.

Even then, the attacker must cross the hypervisor boundary to control the host or another VM.

```text
Application compromise
        │
        ▼
Guest OS compromise
        │
        ✕ hypervisor boundary
        ▼
Host system
```

The hypervisor boundary provides an additional layer.

It is not impossible to break, but it is a separate security boundary.

---

# 14.16 VM Snapshots

A **snapshot** records enough VM state to return to an earlier condition.

It may include:

* Virtual-disk state
* Memory state
* Virtual CPU state
* Device state
* Configuration

```text
VM state at time A
       │ snapshot
       ▼
VM changes over time
       │ restore
       ▼
Return toward state A
```

---

## Why snapshots exist

They are useful for:

* Testing
* Recovery
* Temporary experiments
* Software updates
* Reproducible environments

---

## Snapshot limitations

Snapshots are not automatically complete backups.

Potential issues include:

* External services continue changing.
* Network peers do not return to an earlier state.
* Separate storage may not be included.
* Application consistency may require preparation.
* Snapshot chains consume storage.

Restoring a VM does not reverse actions already sent outside it.

---

# 14.17 Live Migration

Some virtualization systems can move a running VM from one physical host to another.

Conceptually:

1. Prepare VM on destination.
2. Copy memory and device state.
3. Track pages modified during copying.
4. Briefly pause the VM.
5. Transfer final state.
6. Resume on destination.

```text
Physical Host A              Physical Host B
      VM ───── state transfer ─────▶ VM
```

This can support:

* Hardware maintenance
* Load balancing
* Failure prevention
* Data-center management

Exact capabilities depend on shared storage, networking, hardware compatibility, and the hypervisor.

---

# 14.18 What Is a Container?

## What it is

A **container** is an isolated group of processes that runs using the host operating-system kernel.

A container commonly receives a restricted view of:

* Processes
* File systems
* Network interfaces
* Hostnames
* Users
* Devices
* Resource limits

```text
Container
┌───────────────────────────┐
│ Application processes     │
│ Libraries and files       │
│ Restricted resource view  │
└───────────────────────────┘
             │
             ▼
      Host OS kernel
```

A container is not usually a complete virtual computer.

It is an OS-level process-isolation environment.

---

# 14.19 A Container Is Not Simply One Process

A container may contain:

* One application process
* Several worker processes
* Supporting services
* Helper processes

```text
Container
├── Main service process
├── Worker process A
├── Worker process B
└── Logging helper
```

The host kernel still manages these as host processes, but it presents them with container-specific views and restrictions.

---

# 14.20 Container Mental Model: Secured Work Areas

Imagine one large factory floor managed by one central authority.

Different teams receive fenced work areas.

Each area may have:

* Its own tool collection
* Its own visible worker list
* Its own storage shelves
* Its own network address
* A resource quota

```text
One factory
├── Fenced area A
├── Fenced area B
└── Fenced area C
```

The teams share:

* The building
* Central safety systems
* Core utilities
* Factory management

| Factory                 | Container system            |
| ----------------------- | --------------------------- |
| Central management      | Host kernel                 |
| Fenced work area        | Container                   |
| Workers                 | Container processes         |
| Assigned shelves        | Container file-system view  |
| Resource quota          | CPU and memory limits       |
| Shared building failure | Shared-kernel vulnerability |

Unlike separate apartments with separate internal management systems, containers share one central system.

---

# 14.21 The Shared-Kernel Model

Consider two containers:

```text
Container A               Container B
┌────────────────┐        ┌────────────────┐
│ Application A  │        │ Application B  │
│ Libraries A    │        │ Libraries B    │
└────────────────┘        └────────────────┘
          │                       │
          └────── Host kernel ────┘
                       │
                       ▼
                  Physical hardware
```

When Application A makes a system call, it enters the host kernel.

When Application B makes a system call, it enters the same host kernel.

The kernel uses isolation rules to give each process a different view.

---

# 14.22 Container Isolation Mechanisms

Different operating systems use different mechanisms, but the main concepts include:

* Namespaces or isolated resource views
* Resource-control groups
* File-system isolation
* User and permission mapping
* Capability restrictions
* System-call filtering
* Security labels
* Network isolation

The exact names differ, but the goals are stable.

---

# 14.23 Isolated Namespaces

A **namespace** in this context is an isolated view of a class of operating-system resources.

Examples may include separate views of:

* Process identifiers
* File-system mount points
* Network interfaces
* Hostnames
* Inter-process communication objects
* User identities

---

## Process view example

Inside Container A:

```text
Process 1 → Main application
Process 2 → Worker
```

On the host, those same processes may have different identifiers:

```text
Host process 7421 → Container A process 1
Host process 7440 → Container A process 2
```

The container sees its own process namespace.

---

## Why isolated views matter

Without them, containerized processes could easily inspect and interfere with unrelated host or container resources.

Namespaces make the environment appear more self-contained.

---

# 14.24 Resource Controls

Containers commonly use resource controls to limit consumption.

Possible limits include:

* CPU time
* Memory
* Number of processes
* Storage I/O
* Network bandwidth
* Device access

```text
Container A:
CPU limit    → 2 cores worth
Memory limit → 2 GB

Container B:
CPU limit    → 1 core worth
Memory limit → 512 MB
```

---

## Why limits matter

Isolation of names does not prevent resource exhaustion.

Without limits, one container could consume nearly all host memory or CPU capacity.

Resource controls protect availability among workloads.

---

# 14.25 Container File Systems

A container commonly receives its own apparent root file-system view.

Inside the container:

```text
/
├── application
├── libraries
├── configuration
└── temporary data
```

This does not necessarily represent a separate physical disk.

The container file system may be assembled from:

* Read-only image layers
* A writable container layer
* Host directories
* Persistent volumes
* Temporary memory-backed storage

---

# 14.26 Container Images

A **container image** is a packaged file-system and metadata template used to create containers.

It may contain:

* Application executable
* Libraries
* Runtime
* Configuration defaults
* File-system layers
* Startup command
* Metadata

```text
Container image
├── Base file-system layer
├── Runtime layer
├── Application layer
└── Configuration metadata
```

The image is not the running container.

---

## Image versus container

| Image                           | Container                               |
| ------------------------------- | --------------------------------------- |
| Stored template                 | Running environment                     |
| Mostly read-only                | Has active processes and writable state |
| Can create many instances       | One particular runtime instance         |
| Comparable to a program package | Comparable to a running process group   |

This resembles:

```text
Program → Process
Image   → Container
```

The comparison is useful but not exact.

---

# 14.27 Layered Images

Container images often use layers.

```text
Application files
──────────────────
Language runtime
──────────────────
System libraries
──────────────────
Base file-system
```

Several images can share unchanged layers on the host.

This reduces storage use and transfer time.

---

## Writable container layer

When a container modifies its image-based file system, the changes may go into a private writable layer.

```text
Container writable changes
──────────────────────────
Read-only application layer
──────────────────────────
Read-only runtime layer
──────────────────────────
Read-only base layer
```

The original image remains unchanged.

---

# 14.28 Persistent Volumes

Container writable layers are often treated as temporary.

Important data may instead be stored in a **persistent volume**.

```text
Container process
      │
      ▼
Mounted persistent volume
      │
      ▼
Host or network storage
```

A replacement container can attach the same volume.

---

## Why volumes exist

Containers are often created and destroyed frequently.

Application state that must survive should not depend solely on the life of one container instance.

Examples:

* Database data
* Uploaded files
* Long-term logs
* User documents

---

# 14.29 Step-by-Step: Starting a Container

Suppose a container runtime starts an application image.

## Stage 1: Image and configuration are selected

Configuration may include:

* Image
* Startup command
* Environment settings
* CPU and memory limits
* Network configuration
* Mounted volumes
* Device access
* Security restrictions

---

## Stage 2: Runtime asks the host kernel for isolation

The runtime creates or configures:

* Process namespace
* File-system view
* Network namespace
* Resource controls
* Credentials
* Security policy

---

## Stage 3: Container file system is prepared

The runtime combines:

* Image layers
* Writable layer
* Mounted volumes

---

## Stage 4: Container’s initial process is created

The kernel creates an ordinary host process with the configured restrictions.

There is normally no separate guest-kernel boot.

---

## Stage 5: Application begins executing

The application uses the host kernel for:

* System calls
* Scheduling
* Memory management
* File access
* Networking
* Device access

---

## Full flow

```text
Start container request
          │
          ▼
Prepare image file system
          │
          ▼
Configure isolated resource views
          │
          ▼
Apply resource and security limits
          │
          ▼
Create initial host process
          │
          ▼
Application begins
```

Because no guest OS is booted, container startup can be relatively fast.

---

# 14.30 Step-by-Step: A File Read Inside a Container

Suppose an application in a container reads:

```text
/config/settings.txt
```

## Stage 1: Application makes a system call

The system call enters the **host kernel**.

```text
Container process
       │ system call
       ▼
Host kernel
```

---

## Stage 2: Kernel interprets the process’s file-system view

The path `/config/settings.txt` is resolved within the container’s configured mount namespace or equivalent file-system view.

---

## Stage 3: Kernel applies permissions

Checks may include:

* Process credentials
* Container user mapping
* File permissions
* Security labels
* Read-only mount restrictions
* Sandbox rules

---

## Stage 4: File is found in a layer or volume

The data might come from:

* Image layer
* Writable container layer
* Host-mounted directory
* Persistent volume
* Network storage

---

## Stage 5: Host file system and device stack perform I/O

The same host kernel handles:

* Page cache
* Storage driver
* DMA
* Interrupts
* Thread waiting and wake-up

---

## Stage 6: Data returns to the container process

From the application’s perspective, it performed an ordinary file read.

The isolation came from the host kernel’s configured view and policy.

---

# 14.31 Containers and Process Identifiers

A containerized process can have more than one visible identifier.

Example:

```text
Inside container:
PID 1

On host:
PID 8304
```

The container’s PID namespace presents a private numbering system.

The host still tracks the underlying process globally.

---

## Why a container has a process 1

The first process in a container often receives a special container-local PID.

It may be responsible for:

* Starting child processes
* Handling termination signals
* Collecting child exit information
* Coordinating shutdown

It is still managed by the host kernel.

---

# 14.32 Containers and Users

A process may appear to run as an administrator-like user inside a container.

That does not necessarily mean it has unrestricted authority on the host.

A user namespace or similar mapping may translate:

```text
Container user 0
        │
        ▼
Unprivileged host identity
```

However, this protection depends on configuration and platform support.

---

## Privileged containers

A container can be given broad host authority.

A highly privileged container may receive:

* Extensive device access
* Powerful kernel capabilities
* Host file-system mounts
* Host networking
* Ability to change sensitive kernel settings

At that point, the container boundary may provide little security protection.

---

# 14.33 System-Call Filtering

Because container processes share the host kernel, restricting unnecessary system calls can reduce attack surface.

```text
Container process makes system call
              │
              ▼
Filtering policy
        ┌─────┴─────┐
        │           │
     Allowed      Blocked
```

For example, an application that never needs to configure kernel modules should not be permitted to request such operations.

Filtering is one layer of defense.

---

# 14.34 Container Networking

A container may receive:

* Its own virtual network interface
* Its own network address
* Its own routing view
* Selected exposed ports
* Firewall rules

```text
Container process
       │
       ▼
Virtual network interface
       │
       ▼
Host networking
       │
       ▼
Physical network adapter
```

The host kernel still handles the physical network device.

---

## Port mapping

A host network port may be directed to a container service.

```text
Host port 8080
      │
      ▼
Container port 80
      │
      ▼
Web service
```

This creates controlled external access.

---

# 14.35 Container Scheduling

Container processes are ordinary schedulable host threads.

```text
Container A threads ─┐
Container B threads ─┼──▶ Host scheduler ──▶ CPU cores
Host processes ──────┘
```

The host scheduler may apply:

* CPU quotas
* Priorities
* Affinity
* Weighting
* Core restrictions

There is normally no separate container kernel scheduler.

---

# 14.36 Container Memory

Container memory is managed by the host kernel.

```text
Container process virtual address
             │
             ▼
Host-managed page tables
             │
             ▼
Physical RAM
```

Resource controls can track or restrict how much memory a container’s process group uses.

---

## Memory-limit behavior

When a container exceeds its memory limit, possible outcomes include:

* Allocation failure
* Reclaim pressure
* Process termination
* Container workload failure

The precise policy depends on the host and runtime.

---

# 14.37 Containers Are Not Hardware Emulation

A container does not ordinarily pretend to provide a different CPU or a complete virtual motherboard.

Its applications execute as host processes.

This means container compatibility depends on:

* Host kernel interface
* CPU architecture
* Required system calls
* Available kernel features

A container image built for a different CPU architecture may require emulation or translation.

---

# 14.38 Virtualization Versus Emulation

These concepts are related but distinct.

## Virtualization

Allows software designed for the underlying CPU architecture to run with controlled virtual hardware.

Much code may execute directly on the physical CPU.

## Emulation

Imitates another hardware architecture or device through software.

```text
Guest instruction for Architecture A
            │
            ▼
Emulator translates behavior
            │
            ▼
Host Architecture B executes equivalent work
```

---

## Tradeoff

Emulation provides broader compatibility but can add substantial overhead.

A VM may use:

* Hardware virtualization for CPU execution
* Device emulation for selected virtual devices

The two techniques can be combined.

---

# 14.39 Virtual Machine Versus Container

| Property              | Virtual machine                          | Container                                     |
| --------------------- | ---------------------------------------- | --------------------------------------------- |
| Kernel                | Separate guest kernel                    | Shares host kernel                            |
| Hardware view         | Virtual hardware                         | Host OS abstractions                          |
| Startup               | Boots guest OS                           | Starts isolated processes                     |
| Typical startup speed | Slower                                   | Faster                                        |
| Memory overhead       | Usually higher                           | Usually lower                                 |
| OS flexibility        | Can run different guest OS kernels       | Must be compatible with host kernel           |
| Isolation boundary    | Hypervisor and guest boundary            | Host-kernel process isolation                 |
| Packaging             | Entire guest system                      | Application and user-space dependencies       |
| Failure scope         | Guest kernel failure usually stays in VM | Host-kernel failure can affect all containers |
| Density               | Lower                                    | Higher                                        |
| Typical use           | Strong OS separation                     | Application deployment and scaling            |

---

# 14.40 Startup Comparison

## Virtual machine

```text
Create virtual hardware
        │
        ▼
Run virtual firmware
        │
        ▼
Load guest kernel
        │
        ▼
Initialize guest services
        │
        ▼
Start application
```

## Container

```text
Prepare isolated views
        │
        ▼
Apply limits and policy
        │
        ▼
Create host process
        │
        ▼
Start application
```

The absence of a full guest boot is a major reason containers often start faster.

---

# 14.41 Isolation Comparison

## VM boundary

```text
Application
    │
Guest kernel
    │
Hypervisor boundary
    │
Host or hardware
```

An attacker may need to compromise both:

1. Guest environment
2. Hypervisor boundary

---

## Container boundary

```text
Application
    │
Host system-call interface
    │
Host kernel
```

The host kernel directly processes container system calls.

A host-kernel vulnerability may therefore affect all containers.

---

## Important conclusion

Containers can provide strong practical isolation when correctly configured, but their boundary is different from a separate-kernel VM boundary.

Security requirements determine which is appropriate.

---

# 14.42 Resource Efficiency

## VM overhead

Each VM may contain:

* Guest kernel
* Guest services
* Guest file-system cache
* Guest device-management state
* Guest background processes

This consumes additional memory and storage.

---

## Container efficiency

Containers share:

* Host kernel
* Many host services
* Sometimes image layers
* Physical device drivers

This permits more environments on the same machine.

```text
One host:
10 heavier VMs
or
many lighter containers
```

Actual capacity depends on workload and configuration, not only the isolation technology.

---

# 14.43 “Noisy Neighbor” Problem

Several VMs or containers share physical resources.

One workload may create excessive:

* CPU usage
* Memory pressure
* Storage I/O
* Network traffic
* Cache contention

This can slow neighboring workloads.

```text
Workload A consumes heavy storage I/O
             │
             ▼
Workload B experiences higher latency
```

Resource limits and scheduling policies reduce but may not eliminate this interference.

---

# 14.44 Combining VMs and Containers

Virtual machines and containers are often used together.

A common arrangement is:

```text
Physical server
└── Hypervisor
    ├── VM A
    │   └── Containers A1, A2, A3
    └── VM B
        └── Containers B1, B2
```

---

## Why combine them?

VMs provide:

* Stronger tenant or operating-system boundaries
* Separate kernels
* Independent guest administration

Containers provide:

* Fast application deployment
* Efficient scaling
* Dependency packaging
* High workload density

The technologies are complementary rather than mutually exclusive.

---

# 14.45 Cloud Computing Connection

Cloud providers commonly allocate computing environments using virtualization.

A customer may receive:

* Virtual CPUs
* Virtual memory
* Virtual disks
* Virtual networks
* Virtual machines

The provider manages physical hardware underneath.

Containers may then run inside those customer VMs.

```text
Cloud physical hardware
        │
        ▼
Provider hypervisor
        │
        ▼
Customer virtual machine
        │
        ▼
Customer container platform
        │
        ▼
Application containers
```

Each layer creates its own resource and security boundaries.

---

# 14.46 Orchestration

When many containers run across many machines, a management system may coordinate them.

This is called **orchestration**.

An orchestrator may handle:

* Starting containers
* Restarting failed workloads
* Distributing them across hosts
* Updating application versions
* Service discovery
* Networking
* Storage attachment
* Resource allocation
* Scaling

```text
Desired state:
Run five web-service containers
          │
          ▼
Orchestrator
          │
          ├── Host A: two containers
          ├── Host B: two containers
          └── Host C: one container
```

---

## Important distinction

The container runtime starts and manages containers on one host.

An orchestrator coordinates many workloads, often across many hosts.

---

# 14.47 Step-by-Step: Containerized Web Request

Suppose an external client sends a request to a web service running in a container.

## Stage 1: Packet reaches physical network adapter

The host kernel’s network driver handles the hardware event.

---

## Stage 2: Host networking processes the packet

Firewall and routing rules determine where it should go.

---

## Stage 3: Port mapping directs it to the container

```text
Host port
    │
    ▼
Container network interface
    │
    ▼
Web-service socket
```

---

## Stage 4: Container process becomes ready

If the service thread was waiting for network input:

```text
WAITING → READY
```

---

## Stage 5: Host scheduler runs the service thread

The thread executes directly on a physical CPU under host scheduling.

---

## Stage 6: Service processes request

It may:

* Read files
* Query a database
* Allocate memory
* Start worker threads

All system calls enter the host kernel.

---

## Stage 7: Response is written

The host networking stack sends the response through the physical adapter.

The container’s isolation affects its view and permissions, but the host kernel performs the core OS work.

---

# 14.48 Step-by-Step: VM Crash Versus Container Crash

## Application crash inside a VM

1. Guest application performs an invalid operation.
2. Guest CPU state reports an exception.
3. Guest kernel handles it.
4. Guest process is terminated.
5. Other guest processes continue.
6. Other VMs and the host normally continue.

---

## Guest-kernel crash

1. Guest kernel encounters an unrecoverable failure.
2. The entire VM may stop or restart.
3. Other VMs normally continue.
4. The hypervisor remains in control.

---

## Container application crash

1. Container process performs an invalid operation.
2. Host kernel handles the exception.
3. Faulting process or container workload stops.
4. Other containers normally continue.

---

## Host-kernel crash

1. The shared host kernel fails.
2. All containers using that kernel are affected.
3. The physical host may restart.

---

## Comparison

```text
VM guest-kernel crash:
One VM affected

Container host-kernel crash:
All host containers potentially affected
```

This is a major architectural difference.

---

# 14.49 Step-by-Step: Resource Limiting a Container

Suppose a container has a 1 GB memory limit.

## Stage 1: Container processes allocate memory

The host kernel maps pages normally.

---

## Stage 2: Kernel accounts memory to the container group

The processes may collectively use:

```text
800 MB
```

---

## Stage 3: Workload requests more memory

Usage approaches or exceeds:

```text
1 GB
```

---

## Stage 4: Resource-control policy reacts

The kernel may:

* Reclaim pages
* Slow allocations
* Return allocation failure
* Terminate one or more processes

---

## Stage 5: Other containers retain their limits

The goal is to prevent one workload from consuming unrestricted host memory.

Resource limits improve isolation of availability, though shared host pressure can still affect overall performance.

---

# 14.50 Portability

Containers are often described as portable because an image packages application dependencies.

This can reduce differences involving:

* Library versions
* Runtime versions
* Configuration files
* Directory layout

However, portability is not absolute.

A container still depends on:

* Compatible CPU architecture
* Compatible host kernel behavior
* Required kernel features
* External storage
* Network configuration
* Secrets
* Device access

---

## VM portability

A VM packages a broader system environment, including a guest kernel.

This can provide stronger consistency across hosts, but VM images are generally larger and more resource-intensive.

---

# 14.51 Updates and Replacement

Containers are often treated as replaceable rather than modified permanently.

A common pattern is:

```text
Old container runs image version 1
          │
          ▼
Create new image version 2
          │
          ▼
Start new container
          │
          ▼
Move traffic
          │
          ▼
Stop old container
```

This supports reproducible deployment.

Persistent state remains in external volumes or services.

---

## VM updates

A VM may be updated like a traditional computer:

* Install packages
* Update guest kernel
* Restart guest services
* Reboot the VM

VMs can also be replaced from images, but their lifecycle is often more system-oriented than process-oriented.

---

# 14.52 What Can Go Wrong?

## VM escape

A guest exploits a vulnerability in the hypervisor or virtual-device implementation and gains access outside the VM.

Possible impact:

* Host compromise
* Access to other VMs
* Data theft
* Infrastructure control

The hypervisor is part of the trusted computing base.

---

## Container escape

A containerized process exploits:

* Host-kernel vulnerability
* Runtime defect
* Misconfiguration
* Excessive capabilities
* Exposed host resource

It then gains host or cross-container authority.

---

## Privileged container

A container receives excessive host authority.

Possible result:

* Host file-system modification
* Device control
* Kernel reconfiguration
* Escape from practical isolation

---

## Shared-kernel vulnerability

Because containers share one kernel, a kernel security defect may threaten every container on the host.

---

## Vulnerable virtual device

Complex emulated devices increase the hypervisor attack surface.

A malicious guest may send carefully constructed device commands.

---

## Resource exhaustion

A VM or container consumes excessive:

* CPU
* Memory
* Storage
* Network bandwidth
* I/O queue capacity

Resource limits may be missing or incorrectly configured.

---

## Noisy neighbor

One workload reduces another workload’s performance without crossing a formal security boundary.

---

## Image vulnerability

A container or VM image may contain:

* Outdated libraries
* Known vulnerabilities
* Malicious software
* Unnecessary services
* Embedded secrets

Packaging an environment does not make it secure.

---

## Image supply-chain compromise

An attacker modifies:

* Base image
* Package repository
* Build system
* Dependency
* Registry
* Signing key

The compromised image may be deployed widely.

---

## Secret embedded in image

A password or key stored inside an image can be copied by anyone with image access.

Secrets should normally be provided through controlled runtime mechanisms.

---

## Writable-state loss

Important data is stored only in a container’s temporary writable layer.

When the container is replaced, the data disappears.

---

## Snapshot inconsistency

A VM snapshot captures storage while an application is midway through an update.

After restoration, the file system may be intact while application data is inconsistent.

---

## Time confusion after pause or restore

A paused or restored VM may observe an unexpected change in time.

Applications relying on timing must handle this carefully.

---

## Overcommitment

A host promises more virtual CPUs or memory than can be used simultaneously.

This can improve utilization when workloads are mostly idle.

If all workloads become active together:

* Performance may collapse.
* Memory pressure may increase.
* Latency may become unpredictable.

---

## Incorrect network exposure

A service intended only for internal container communication is accidentally exposed to the public network.

---

## Host mount exposure

A container receives a writable mount of a sensitive host directory.

The process can modify host data through that mount.

---

# 14.53 Common Misconceptions

## Misconception: “A container is a lightweight virtual machine”

The comparison is convenient but incomplete.

A VM has virtual hardware and a separate guest kernel.

A container is normally a restricted group of host-kernel processes.

---

## Misconception: “Each container has its own kernel”

Containers normally share the host kernel.

They may contain kernel-related files or utilities, but their system calls still reach the host kernel.

---

## Misconception: “A VM is just an ordinary process”

A hosted VM may appear as one or more host processes, but it represents a complete virtual hardware environment containing a guest OS and many guest processes.

---

## Misconception: “A container provides no isolation”

Properly configured containers can provide substantial process, file-system, network, user, and resource isolation.

The boundary is simply different from a separate-kernel VM.

---

## Misconception: “A container is automatically secure because it is isolated”

Container security depends on:

* Kernel security
* Runtime configuration
* Capabilities
* Mounted resources
* Network exposure
* Image contents
* Resource limits

---

## Misconception: “Root inside a container always equals unrestricted host administrator”

User mappings and capability restrictions may reduce its host authority.

However, privileged configurations can make container root extremely powerful.

---

## Misconception: “VMs cannot affect one another”

They share physical resources and may affect performance.

Hypervisor vulnerabilities and misconfigured shared storage or networking can also cross boundaries.

---

## Misconception: “Virtual CPUs are dedicated physical cores”

A virtual CPU may share physical CPU time with other VMs.

Dedicated assignment is possible but not automatic.

---

## Misconception: “Allocating 8 GB to a VM means 8 GB is continuously active in physical RAM”

Hypervisors may use dynamic allocation, reclamation, sharing, or overcommitment.

Exact guarantees depend on configuration.

---

## Misconception: “Snapshots are backups”

Snapshots may depend on their original storage chain and may not include external data.

They are useful recovery points but do not replace a complete backup strategy.

---

## Misconception: “Container images contain all application state”

Images contain packaged starting contents.

Persistent runtime data commonly lives in volumes, databases, or external services.

---

## Misconception: “A container runs identically on every machine”

It still depends on compatible CPU architecture, kernel features, devices, networking, and external services.

---

## Misconception: “Virtualization and emulation are the same”

Virtualization often lets compatible instructions run directly with controlled privilege.

Emulation imitates another hardware behavior and may translate instructions.

---

# 14.54 Real-World Analogy: Apartments Versus Fenced Offices

Consider one large building.

## Virtual machines: apartments

Each apartment has:

* Its own internal electrical panel
* Its own rooms
* Its own residents
* Its own internal management
* A strong wall separating it from other apartments

The building owner still controls the shared foundation.

This resembles a VM with its own guest operating system.

---

## Containers: fenced offices on one managed floor

Each team receives:

* A separate workspace
* Separate cabinets
* Separate labels
* Resource limits

But all teams share:

* The same building management
* The same safety system
* The same central utilities
* The same structural floor

This resembles containers sharing one host kernel.

---

## Comparison

| Building concept               | VM                       | Container                    |
| ------------------------------ | ------------------------ | ---------------------------- |
| Separate internal management   | Yes                      | No                           |
| Shared building foundation     | Yes                      | Yes                          |
| Strong structural walls        | Hypervisor boundary      | Host-kernel isolation        |
| Independent utility controller | Guest kernel             | Shared host kernel           |
| Startup                        | Prepare entire apartment | Admit workers to fenced area |

---

# 14.55 Connection to Earlier Concepts

## Connection to hardware

A hypervisor divides physical CPU, memory, and devices among virtual machines.

Containers use the host kernel’s existing hardware management.

---

## Connection to kernels

A VM has a guest kernel.

A container shares the host kernel.

This is the central distinction.

---

## Connection to user and kernel mode

Inside a VM:

```text
Guest application user mode
          │
          ▼
Guest kernel mode
          │
          ▼
Hypervisor control beneath it
```

Inside a container:

```text
Container process user mode
          │
          ▼
Host kernel mode
```

---

## Connection to processes

A container is built from host processes with additional isolation and resource controls.

A VM contains its own guest processes managed by its guest kernel.

---

## Connection to CPU scheduling

VMs introduce guest scheduling and hypervisor scheduling.

Containers use host scheduling directly.

---

## Connection to virtual memory

VMs may use two levels of address translation.

Containers use host-managed process page tables.

---

## Connection to files

VMs may use virtual disks containing guest file systems.

Containers commonly use layered host file-system views and mounted volumes.

---

## Connection to devices

VMs receive virtual devices or assigned hardware.

Containers generally access host-managed devices through restricted interfaces.

---

## Connection to security

VMs add a hypervisor boundary.

Containers depend more directly on host-kernel isolation.

Both require least privilege, resource limits, secure configuration, and updates.

---

## Connection to crashes

A guest-kernel crash normally affects one VM.

A host-kernel crash can affect every container using that kernel.

---

# 14.56 Simplified Model Versus Technical Reality

## Simplified mental model

```text
Virtual machine:
A virtual computer with its own operating-system kernel.

Container:
An isolated group of processes sharing the host kernel.
```

This is the model to retain.

---

## More exact reality

Virtualization systems may include:

* Hardware-assisted CPU virtualization
* Nested page tables
* Device emulation
* Paravirtualized drivers
* Direct device assignment
* Live migration
* Memory overcommitment
* Nested virtualization
* Virtual firmware
* Distributed storage

Container systems may include:

* Several kinds of namespaces
* Resource-control groups
* Capability restrictions
* System-call filters
* Security labels
* Layered file systems
* User mappings
* Virtual networks
* Orchestrators
* Runtime shims

Some technologies combine VM and container ideas.

For example, a container may run inside a small dedicated VM to gain a separate kernel boundary.

The central architectural question remains:

> Does the workload have its own kernel, or does it share the host kernel?

---

# 14.57 Core Mental Models

## Virtual-machine model

```text
Guest application
       │ system call
       ▼
Guest kernel
       │ virtual hardware operation
       ▼
Hypervisor
       │
       ▼
Physical hardware
```

---

## Container model

```text
Container application
          │ system call
          ▼
Host kernel
          │
          ▼
Physical hardware
```

---

## Isolation model

```text
VM isolation:
Guest processes
      │
Guest kernel
      │
Hypervisor boundary
      │
Host

Container isolation:
Container processes
      │
Host-kernel isolation rules
      │
Host kernel
```

---

## Startup model

```text
VM:
Create hardware → boot guest OS → start application

Container:
Create isolated process environment → start application
```

---

## Final distinctions

| Concept              | Essential meaning                                |
| -------------------- | ------------------------------------------------ |
| **Host**             | System supplying underlying resources            |
| **Guest**            | OS running inside a virtual machine              |
| **Hypervisor**       | Layer that creates and isolates VMs              |
| **Virtual machine**  | Virtual hardware environment with a guest kernel |
| **Virtual CPU**      | CPU execution resource presented to a VM         |
| **Virtual device**   | Device interface presented to a guest            |
| **Container**        | Isolated process group sharing the host kernel   |
| **Container image**  | Packaged template used to start containers       |
| **Volume**           | Storage intended to persist beyond one container |
| **Namespace**        | Isolated view of an OS resource class            |
| **Resource control** | Limit on CPU, memory, I/O, or other use          |
| **Orchestrator**     | System coordinating many container workloads     |
| **Emulation**        | Software imitation of hardware behavior          |

The next section examines the **boot process**—how firmware, a bootloader, the kernel, drivers, system services, and the user environment start from a powered-off machine.

# Learning Check

Do not look for answers yet.

## Conceptual questions

1. What is the central architectural difference between a virtual machine and a container?
2. What roles do the guest kernel and hypervisor perform when an application runs inside a VM?
3. How do container images, running containers, and persistent volumes differ?

## Cause-and-effect questions

4. Why can containers usually start faster and use less memory than virtual machines?
5. Why can a host-kernel vulnerability threaten several containers while a guest-kernel crash normally remains inside one VM?

## Misconception question

6. A student says, “A container is simply a small VM with its own lightweight kernel.” What is wrong with this explanation?

## Scenario-based question

7. A server runs two VMs. Each VM runs several containers. An application in one container reads a file from persistent storage. Trace the request through the container process, guest kernel, virtual storage device, hypervisor, host storage system, physical device, and back to the application.

# 15. The Boot Process

When a computer is powered on, RAM does not yet contain a running operating-system kernel.

The machine must progress through a carefully ordered startup sequence:

```text
Power on
   │
   ▼
Firmware starts
   │
   ▼
Hardware is initialized
   │
   ▼
Boot program is located
   │
   ▼
Operating-system kernel is loaded
   │
   ▼
Kernel initializes the system
   │
   ▼
System services start
   │
   ▼
Login or desktop appears
```

This sequence is called the **boot process**.

The word “boot” comes from the idea of **bootstrapping**: starting a complex system using a much smaller initial mechanism.

---

# 15.1 Why Booting Is Necessary

A stored operating system is passive data.

Before startup:

* The kernel exists on storage.
* Device drivers exist on storage.
* System services exist on storage.
* Applications exist on storage.

But the CPU can execute only instructions available through a valid execution path.

The system therefore needs a small, trusted starting point that can find and load the larger operating system.

```text
Stored kernel
    │
    │ cannot execute itself automatically
    ▼
Firmware and bootloader
    │
    ▼
Kernel placed into memory and started
```

---

## The fundamental startup problem

To read the kernel from storage, software needs to know how to access storage.

But the normal storage driver is often part of the operating system that has not started yet.

This creates a dependency:

```text
Need OS to use hardware fully
Need hardware access to load OS
```

Booting solves this by using layers:

1. Firmware provides minimal initial hardware support.
2. A bootloader performs enough work to load the kernel.
3. The kernel initializes full device-management systems.
4. System services create the normal user environment.

---

# 15.2 Mental Model: Opening a Theater

Imagine a large theater before a performance.

The audience, actors, lighting systems, ticketing systems, and stage crew are not initially active.

A small startup team arrives first.

1. Building power is enabled.
2. Safety systems are checked.
3. Essential lights are turned on.
4. The stage manager is called.
5. The stage manager organizes the full crew.
6. Ticket desks and doors open.
7. The audience enters.
8. The performance begins.

| Theater                   | Computer                    |
| ------------------------- | --------------------------- |
| Building power            | Electrical power            |
| Initial safety controller | Firmware                    |
| Startup checklist         | Hardware initialization     |
| Stage manager             | Bootloader and kernel       |
| Full crew                 | Drivers and system services |
| Ticket desk               | Login system                |
| Performance               | User applications           |

No single component performs the entire startup.

Each stage prepares the conditions required by the next.

---

# 15.3 Main Boot Components

The foundational boot sequence involves:

| Component                        | Main responsibility                                               |
| -------------------------------- | ----------------------------------------------------------------- |
| **Firmware**                     | Begins execution and initializes enough hardware to continue      |
| **Boot manager or bootloader**   | Selects and loads an operating system                             |
| **Kernel**                       | Takes control of CPU, memory, devices, and processes              |
| **Early user-space environment** | Helps prepare the real system storage when needed                 |
| **Initial system process**       | Starts services and establishes the user environment              |
| **System services**              | Provide networking, logging, graphics, login, and other functions |
| **Login or session manager**     | Authenticates the user and starts a session                       |
| **User applications**            | Run after the operating-system environment is ready               |

---

# 15.4 Power-On and CPU Reset

## What happens first

When power becomes stable, the CPU enters a predefined reset state.

It does not begin by searching the entire storage device for an operating system.

Instead, the hardware defines a known starting instruction location.

```text
Power becomes stable
        │
        ▼
CPU enters reset state
        │
        ▼
CPU begins at predefined address
        │
        ▼
Firmware instructions execute
```

---

## Why a predefined location exists

The CPU needs an unambiguous first instruction.

At this stage:

* No process exists.
* No scheduler exists.
* No virtual memory exists in its normal OS form.
* No file system has been mounted.
* No user application is running.

The initial instructions therefore come from firmware accessible through hardware-defined mechanisms.

---

# 15.5 What Is Firmware?

## What it is

**Firmware** is low-level software stored in nonvolatile memory associated with the computer or a device.

System firmware begins the machine’s startup.

Common firmware environments include:

* Traditional BIOS-style firmware
* UEFI-style firmware

Individual devices may also contain their own firmware.

---

## Why firmware exists

Firmware provides enough initial control to:

* Begin CPU execution
* Initialize essential hardware
* Discover devices
* Perform basic checks
* Find a bootable operating system
* Start the next boot stage

Firmware operates before the normal OS kernel is active.

---

## Firmware is software, not hardware

Firmware is stored and executed as code, but it is closely tied to hardware.

```text
Hardware
   │
   ▼
Firmware controls early initialization
   │
   ▼
Operating system later assumes control
```

---

# 15.6 BIOS and UEFI

Two important firmware models are commonly discussed.

## BIOS-style firmware

Traditional BIOS systems generally:

* Perform initial hardware setup
* Select a boot device
* Read a small initial boot region
* Transfer execution to that code

This model was designed under older hardware constraints.

---

## UEFI-style firmware

UEFI provides a richer firmware environment.

It may support:

* Structured boot entries
* File-system access to a firmware-readable system partition
* Larger disks
* Graphical configuration
* Secure Boot
* Network booting
* Firmware applications
* More standardized boot management

---

## Simplified comparison

| BIOS-style startup                          | UEFI-style startup                                       |
| ------------------------------------------- | -------------------------------------------------------- |
| Loads initial code from a small boot region | Can load a boot application from a file-system partition |
| More constrained early environment          | Richer firmware services                                 |
| Older boot model                            | Modern common boot model                                 |
| Often tied to legacy disk structures        | Uses structured boot entries and firmware-readable files |

The precise behavior depends on the machine and configuration.

---

# 15.7 Power-On Self-Test and Hardware Initialization

Firmware performs early hardware checks and configuration.

This is often described as a **power-on self-test**, or **POST**.

Possible checks include:

* CPU initialization
* Basic memory detection
* Firmware integrity
* Keyboard or basic input availability
* Graphics initialization
* Storage-controller discovery
* Hardware configuration validation

```text
Firmware begins
      │
      ▼
Initialize essential hardware
      │
      ▼
Perform basic checks
      │
  ┌───┴────┐
  │        │
Pass      Serious failure
  │        │
Continue  Report or stop
```

---

## What POST does not mean

It does not necessarily perform an exhaustive test of every RAM location or every device component.

A complete hardware test could take a long time.

POST usually performs enough checking to determine whether startup can continue.

---

# 15.8 Early Memory Initialization

The firmware must make usable RAM available before larger software can be loaded.

Conceptually:

1. Memory hardware is detected.
2. Memory-controller settings are established.
3. Basic tests are performed.
4. Usable memory regions are recorded.
5. Firmware and later boot software begin using RAM.

```text
RAM hardware detected
        │
        ▼
Memory controller configured
        │
        ▼
Usable RAM becomes available
```

At this stage, the OS has not yet built per-process virtual address spaces.

---

# 15.9 Device Discovery During Firmware Startup

Firmware detects enough devices to continue booting.

Examples may include:

* Storage controllers
* Internal drives
* USB devices
* Network boot interfaces
* Graphics hardware
* Keyboard
* Trusted security hardware

Firmware may not fully initialize every advanced device feature.

Its goal is to provide enough functionality to locate and start the operating system.

The kernel later performs more complete device initialization using its drivers.

---

# 15.10 Choosing a Boot Source

The firmware must decide where to find boot software.

Possible boot sources include:

* Internal SSD
* Another storage drive
* Removable USB device
* Network server
* Optical media
* Recovery partition
* Firmware recovery environment

---

## Boot order

Firmware settings may define an order such as:

```text
1. Internal SSD
2. USB storage
3. Network boot
```

The firmware checks for an acceptable boot target.

---

## UEFI boot entries

In a UEFI-style system, firmware may store named entries such as:

```text
Primary operating system
Recovery environment
External boot device
Network installer
```

A boot manager can also present a selection menu.

---

# 15.11 What Is a Bootloader?

## What it is

A **bootloader** is a program whose main task is to load and start an operating-system kernel.

It may also:

* Display an OS selection menu
* Load kernel configuration
* Load an early file-system image
* Pass hardware information
* Pass startup parameters
* Support recovery modes
* Verify signatures
* Load different kernel versions

```text
Firmware
    │
    ▼
Bootloader
    │
    ├── Select kernel
    ├── Load kernel into RAM
    ├── Load startup data
    └── Transfer control
```

---

## Why a bootloader exists

Firmware understands how to start boot software, but it may not understand every operating system’s kernel format or startup requirements.

The bootloader bridges this gap.

---

## Bootloader mental model

The firmware is like a receptionist who can find the stage manager.

The bootloader is the stage manager’s assistant who prepares:

* The correct script
* The stage layout
* The equipment list
* Startup instructions

Then the operating-system kernel takes over.

---

# 15.12 Multi-Stage Bootloading

Some systems use more than one bootloader stage.

A small initial stage may be limited by storage or firmware constraints.

It then loads a larger, more capable stage.

```text
Firmware
   │
   ▼
Small boot stage
   │
   ▼
Larger bootloader
   │
   ▼
Kernel
```

The later stage may understand:

* File systems
* Configuration files
* Multiple operating systems
* Encryption
* Recovery options
* Graphical menus

Modern firmware can sometimes load the main boot application directly, reducing the need for tiny legacy stages.

---

# 15.13 Selecting an Operating System or Kernel

A machine may contain:

* Several operating systems
* Several kernel versions
* Recovery environments
* Diagnostic tools

The bootloader may choose automatically or ask the user.

```text
Boot menu
├── Main OS
├── Previous kernel
├── Recovery mode
└── Diagnostic environment
```

A previous kernel can be valuable when a new kernel or driver prevents normal startup.

---

# 15.14 Loading the Kernel Into Memory

Once selected, the bootloader locates the kernel image on storage.

It then:

1. Reads the kernel data.
2. Places it into appropriate RAM.
3. Loads related startup data.
4. Prepares parameters.
5. Transfers CPU control to the kernel entry point.

```text
Kernel image on storage
          │
          ▼
Bootloader reads it
          │
          ▼
Kernel placed in RAM
          │
          ▼
CPU jumps to kernel entry
```

At the moment of transfer, the bootloader stops being the main controlling program.

The kernel begins taking responsibility for the machine.

---

# 15.15 Kernel Compression

Kernel images may be stored in compressed form to reduce storage size and loading time.

Startup may include:

```text
Compressed kernel image
         │
         ▼
Bootloader or kernel startup code
         │
         ▼
Kernel decompressed into memory
         │
         ▼
Kernel execution continues
```

Compression is a storage format detail. The executing kernel must ultimately have usable instructions and data in memory.

---

# 15.16 Information Passed to the Kernel

The kernel needs information about the machine and boot configuration.

The bootloader or firmware may provide:

* Memory layout
* Startup parameters
* Selected root storage
* Firmware tables
* Display information
* Hardware descriptions
* Early file-system image location
* Security state
* Command-line options

```text
Bootloader
   │
   ├── Kernel image
   ├── Hardware information
   ├── Startup parameters
   └── Early file-system data
   ▼
Kernel
```

The exact format depends on the operating system and firmware environment.

---

# 15.17 Secure Boot

## What it is

**Secure Boot** is a mechanism that verifies selected boot components before allowing them to execute.

A simplified chain is:

```text
Firmware trusts selected keys
          │
          ▼
Firmware verifies bootloader
          │
          ▼
Bootloader verifies kernel
          │
          ▼
Kernel verifies selected modules or drivers
```

---

## Why Secure Boot exists

An attacker who modifies early startup software may gain control before normal security systems are active.

Such malicious startup software could:

* Hide from the OS
* Bypass login protections
* Read secrets
* Modify the kernel
* Persist across restarts

Verification helps detect unauthorized changes.

---

## What Secure Boot does not prove

A valid signature does not prove that software:

* Contains no bugs
* Is harmless
* Uses secure configuration
* Cannot be exploited later

It establishes that the software is approved under a trust policy and has not been altered after signing.

---

# 15.18 Kernel Entry

When the bootloader transfers control, the kernel begins executing in a privileged CPU mode.

At this stage, the kernel must build the environment that later applications rely on.

Early kernel work may include:

* Establishing its own memory mappings
* Initializing CPU state
* Configuring exception handlers
* Configuring interrupt handling
* Detecting processors
* Starting memory management
* Initializing timers
* Preparing the scheduler
* Initializing essential drivers

```text
Bootloader hands off
        │
        ▼
Kernel early initialization
        │
        ├── CPU
        ├── Memory
        ├── Interrupts
        ├── Timers
        └── Drivers
```

---

# 15.19 Kernel Memory Initialization

The kernel receives information about available physical memory.

It must determine which regions are:

* Usable RAM
* Occupied by firmware
* Occupied by the kernel
* Reserved for devices
* Needed for boot data
* Unsafe or unavailable

```text
Physical address space
├── Firmware-reserved region
├── Kernel image
├── Usable RAM
├── Device-related region
└── Additional reserved areas
```

The kernel then establishes its physical-memory management structures.

---

## Kernel memory tasks

The kernel may initialize:

* Free-frame tracking
* Page tables
* Kernel heap
* Memory zones or pools
* Page caches
* Protection permissions
* Per-CPU memory structures

Only after these foundations exist can normal process memory be managed safely.

---

# 15.20 Establishing Virtual Memory

Early startup code may operate with limited or temporary mappings.

The kernel replaces or expands these with its full virtual-memory design.

It configures:

* Kernel virtual addresses
* Kernel code permissions
* Kernel data permissions
* Device mappings
* Page-table structures
* Protection against user access

```text
Early temporary mappings
          │
          ▼
Kernel constructs full mappings
          │
          ▼
Normal virtual-memory system active
```

At this stage, ordinary user processes still may not exist.

---

# 15.21 Exception and Interrupt Initialization

The kernel configures CPU structures that determine where control transfers when events occur.

Examples:

* Invalid instruction
* Page fault
* Division error
* Timer interrupt
* Device interrupt
* System-call entry

```text
CPU event
   │
   ▼
Kernel-configured handler table
   │
   ▼
Appropriate kernel handler
```

Without valid handlers, the kernel could not safely respond to errors or devices.

---

# 15.22 Timer Initialization

The kernel configures timers for:

* Timekeeping
* Scheduling
* Thread sleep
* Timeouts
* Periodic maintenance

The timer is essential because it allows the kernel to regain control from running software.

```text
Kernel configures timer
          │
          ▼
Thread later runs
          │
          ▼
Timer interrupt
          │
          ▼
Scheduler may run
```

---

# 15.23 Scheduler Initialization

Before ordinary processes can run, the kernel prepares scheduling structures.

It may initialize:

* Ready queues
* Per-CPU scheduler state
* Priority systems
* Idle threads
* Timing and accounting
* CPU-affinity information

---

## Idle thread

Each CPU core needs something safe to execute when no ordinary thread is ready.

The kernel uses an idle thread or idle loop.

```text
Ready thread exists?
   ┌────┴────┐
   │         │
  Yes        No
   │         │
Run thread  Run idle activity
```

Idle activity may allow the CPU to enter a low-power state until an interrupt occurs.

---

# 15.24 Multiprocessor Startup

A computer may contain several CPU cores.

Often one processor begins the earliest startup work.

The kernel then starts the additional processors.

```text
Primary CPU starts kernel
          │
          ▼
Kernel initializes shared state
          │
          ▼
Additional CPUs are started
          │
          ▼
Each CPU enters scheduler control
```

The kernel must prepare per-CPU:

* Stacks
* Scheduler state
* Interrupt state
* Memory-management information
* Local timers

Only after initialization can those cores execute ordinary threads.

---

# 15.25 Driver Initialization

The kernel must detect and initialize devices.

Examples include:

* Storage controllers
* Network adapters
* USB controllers
* Input devices
* Graphics hardware
* Audio hardware

```text
Kernel discovers device
        │
        ▼
Select matching driver
        │
        ▼
Driver initializes controller
        │
        ▼
Device registered with OS
```

---

## Built-in drivers versus loadable drivers

Some essential drivers may be built directly into the kernel.

Others may be loaded later as modules or separate driver components.

A storage driver needed to access the operating system’s main file system must be available early enough.

---

# 15.26 The Early User-Space Image

Sometimes the kernel cannot immediately mount the real root file system.

It may first use a small temporary file-system image loaded into RAM.

This is commonly called an:

* Initial RAM file system
* Initial RAM disk
* Early user-space environment

Exact terminology differs.

---

## Why it exists

The real root file system may require:

* A storage driver
* A file-system driver
* Disk decryption
* RAID assembly
* Logical-volume activation
* Network configuration
* Device discovery

The temporary environment contains enough tools and drivers to perform this preparation.

---

## Flow

```text
Kernel starts
    │
    ▼
Temporary RAM-based file system
    │
    ├── Load needed drivers
    ├── Unlock encrypted storage
    ├── Assemble storage volumes
    └── Locate real root file system
    │
    ▼
Switch to real root file system
```

---

# 15.27 Step-by-Step: Encrypted System Storage

Suppose the main operating-system storage is encrypted.

## Step 1: Firmware starts the bootloader

The bootloader and early kernel components are available from a boot-accessible location.

---

## Step 2: Kernel loads early user-space environment

The main encrypted data is not yet readable.

---

## Step 3: Early environment identifies encrypted storage

It loads necessary:

* Storage drivers
* Encryption support
* Keyboard or secure-input support

---

## Step 4: Key material is obtained

Possible sources include:

* User-entered password
* Trusted security hardware
* Recovery key
* Enterprise key service

---

## Step 5: Storage is unlocked

The kernel creates a logical decrypted view.

```text
Encrypted physical storage
          │
          ▼
Decryption layer
          │
          ▼
Readable logical volume
```

---

## Step 6: Real root file system is mounted

The operating system can now access its normal files.

---

## Step 7: Startup continues

The temporary early environment hands control to the normal system initialization process.

---

# 15.28 Mounting the Root File System

The **root file system** is the primary file-system hierarchy from which the running OS accesses system files.

Before it is mounted, paths such as:

```text
/system
/configuration
/applications
/users
```

do not yet refer to the normal persistent system hierarchy.

---

## Step-by-step

1. Kernel identifies the root storage device or logical volume.
2. Required file-system driver is available.
3. Kernel reads file-system metadata.
4. File-system consistency is checked as required.
5. Root file system is mounted.
6. Normal system files become accessible.

```text
Storage device
     │
     ▼
File-system driver
     │
     ▼
Root file system mounted
     │
     ▼
System files available
```

---

# 15.29 Kernel Versus User-Space Startup

The kernel does not directly perform every high-level startup task.

Once the kernel has established core mechanisms, it starts the first user-space process or system initialization process.

```text
Kernel initialization
        │
        ▼
First user-space process
        │
        ▼
System services
        │
        ▼
Login and applications
```

This marks an important transition:

* Kernel mechanisms are active.
* User-space policy and services begin.

---

# 15.30 The Initial User-Space Process

The first user-space process has a special role.

It commonly:

* Starts system services
* Reaps terminated child processes
* Establishes startup targets or run states
* Mounts additional file systems
* Starts login services
* Starts graphical services
* Coordinates shutdown and restart

Different operating systems use different initialization systems and process names.

The general role is stable.

---

## Process hierarchy

Conceptually:

```text
Kernel
  │ creates
  ▼
Initial system process
  ├── Logging service
  ├── Network service
  ├── Device service
  ├── Login service
  └── Graphical environment
```

Many later processes descend directly or indirectly from this initial process.

---

# 15.31 Starting System Services

A **system service** is a long-running process that provides functionality to the rest of the system.

Examples include:

* Logging
* Networking
* Time synchronization
* Device discovery
* Printing
* Audio
* Graphical login
* User-account management
* Background updates
* Remote access

---

## Service dependencies

Services may need to start in a particular order.

Example:

```text
Storage available
      │
      ▼
Configuration loaded
      │
      ▼
Network service starts
      │
      ▼
Remote service starts
```

A service manager may represent these dependencies explicitly.

---

## Parallel startup

Independent services can start concurrently.

```text
After root file system is ready:

├── Start logging
├── Start device management
├── Start networking
└── Start graphical services
```

This can reduce boot time.

Dependencies must still be respected.

---

# 15.32 Device Management After Kernel Startup

Some device initialization continues in user space.

A system device manager may:

* React to newly detected devices
* Load additional drivers
* Create device-access interfaces
* Apply permissions
* Run configuration helpers
* Notify applications

```text
Kernel reports device
        │
        ▼
Device-management service
        │
        ├── Apply policy
        ├── Load support
        ├── Set permissions
        └── Notify system
```

Not every device must be fully initialized before the first user-space process starts.

---

# 15.33 Network Startup

Network initialization may involve:

1. Driver initializes the network adapter.
2. Network interface is created.
3. Security and firewall rules are applied.
4. An address is configured.
5. Routes are established.
6. Name-resolution services start.
7. Network-dependent services begin.

```text
Network hardware
       │
       ▼
Driver
       │
       ▼
Interface configuration
       │
       ▼
Address and routes
       │
       ▼
Network services available
```

Network startup may continue after the login screen appears.

---

# 15.34 Logging During Boot

Logging records startup events such as:

* Driver initialization
* File-system mounting
* Service startup
* Errors
* Timeouts
* Hardware discovery
* Security verification

Boot logs are useful when a system fails to start normally.

---

## Early logging problem

At the earliest stages:

* The normal file system may not be mounted.
* The normal logging service may not exist.
* Display output may be limited.

Early messages may therefore be stored in:

* Kernel memory buffers
* Firmware logs
* A serial interface
* A simple screen console

Later, the logging service can collect and preserve them.

---

# 15.35 Login System

Once core services are available, the system can accept user authentication.

A login system may:

1. Display a prompt or login screen.
2. Collect credentials.
3. Authenticate the user.
4. Load account policy.
5. Create a user session.
6. Start the user’s initial processes.

```text
Login request
     │
     ▼
Authentication
     │
  ┌──┴───┐
  │      │
Pass    Fail
  │      │
Create  Deny or retry
session
```

---

# 15.36 Starting a User Session

After successful authentication, the OS creates a user session.

It may establish:

* User credentials
* Environment settings
* Home directory
* Desktop or shell
* Session communication channels
* Display access
* Audio access
* User-specific services

```text
Authenticated identity
          │
          ▼
Session manager
          │
          ├── Shell or desktop
          ├── User services
          ├── Notification service
          └── Startup applications
```

---

# 15.37 Graphical Startup

A graphical environment may require several layers:

```text
Graphics hardware
      │
      ▼
Graphics driver
      │
      ▼
Display or window system
      │
      ▼
Login manager
      │
      ▼
Desktop environment
      │
      ▼
Applications
```

The desktop is not the kernel.

It is a collection of user-space processes using kernel-provided mechanisms.

---

# 15.38 Step-by-Step: Full Cold Boot

A **cold boot** begins from a powered-off state.

## Stage 1: Power is applied

The power system stabilizes voltage and releases the CPU from reset.

**Primary component:** Hardware

---

## Stage 2: CPU begins firmware execution

The processor starts at a hardware-defined location.

**Primary component:** CPU and firmware

---

## Stage 3: Firmware initializes essential hardware

It prepares:

* CPU state
* Basic memory access
* Required controllers
* Basic display or console
* Boot-device discovery

**Primary component:** Firmware

---

## Stage 4: Firmware performs basic checks

Serious failures may produce:

* Error messages
* Indicator lights
* Sound codes
* Recovery mode
* Halt

---

## Stage 5: Firmware selects a boot target

It follows configured boot policy.

**Primary component:** Firmware boot manager

---

## Stage 6: Bootloader starts

The firmware loads or executes the chosen boot application.

**Primary component:** Bootloader

---

## Stage 7: Bootloader selects kernel and options

It may choose:

* Normal startup
* Previous kernel
* Recovery mode
* Alternative OS

---

## Stage 8: Bootloader loads kernel into RAM

It may also load:

* Early file-system image
* Hardware information
* Startup parameters

---

## Stage 9: Bootloader transfers CPU control

The kernel begins privileged execution.

---

## Stage 10: Kernel establishes memory management

It identifies physical RAM, builds page tables, and initializes allocators.

**Primary component:** Kernel memory manager

---

## Stage 11: Kernel initializes interrupts and exceptions

It configures handlers for:

* Page faults
* Timer interrupts
* Device interrupts
* System calls
* Processor exceptions

---

## Stage 12: Kernel initializes scheduling

It creates idle threads, scheduler queues, and CPU-management structures.

---

## Stage 13: Additional CPU cores start

The kernel brings other processors under scheduler control.

---

## Stage 14: Essential drivers initialize

Storage, console, and other required devices become usable.

---

## Stage 15: Early user-space environment runs if needed

It may:

* Load drivers
* Unlock storage
* Assemble volumes
* Find the root file system

---

## Stage 16: Root file system is mounted

Normal operating-system files become available.

---

## Stage 17: Initial system process starts

The kernel creates the first normal user-space process.

---

## Stage 18: System services start

Examples:

* Logging
* Device management
* Networking
* Time services
* Graphics
* Login

---

## Stage 19: Login becomes available

The user authenticates.

---

## Stage 20: User session starts

The shell, desktop, and startup applications begin.

---

## Complete cold-boot timeline

```text
Time ─────────────────────────────────────────────────────▶

Hardware:
Power and reset
     │

Firmware:
     [Initialize hardware][Select boot target]
                              │

Bootloader:
                              [Load kernel]
                                         │

Kernel:
                                         [Memory][CPU][Drivers][Scheduler]
                                                             │

Early user space:
                                                             [Prepare root]
                                                                      │

System services:
                                                                      [Start]
                                                                             │

User session:
                                                                             [Login][Desktop]
```

---

# 15.39 Warm Boot and Restart

A **warm boot** or restart occurs without removing all electrical power.

The operating system first attempts an orderly shutdown.

```text
Restart requested
       │
       ▼
Stop user applications
       │
       ▼
Stop services
       │
       ▼
Flush file-system data
       │
       ▼
Ask hardware to reset
       │
       ▼
Firmware starts again
```

Some hardware initialization may differ from a cold boot.

The precise behavior depends on firmware and device state.

---

# 15.40 Shutdown Is the Reverse Coordination Problem

A safe shutdown must stop components in an orderly sequence.

Possible steps include:

1. Notify user applications.
2. Stop new sessions.
3. Stop system services.
4. Complete or cancel I/O.
5. Flush dirty file-system data.
6. Unmount file systems.
7. Stop devices.
8. Terminate remaining processes.
9. Request firmware power-off or reset.

```text
Applications stop
      │
      ▼
Services stop
      │
      ▼
Storage is synchronized
      │
      ▼
File systems unmount
      │
      ▼
Hardware powers off
```

Removing power without these steps can lose buffered data or damage file-system consistency.

---

# 15.41 Sleep, Hibernation, and Booting

These are different startup paths.

## Sleep or suspend

The system preserves active state in RAM while most hardware enters a low-power mode.

```text
Running system
      │
      ▼
RAM remains powered
      │
      ▼
Most devices sleep
      │
      ▼
Resume quickly
```

If RAM loses power, unsaved state may be lost.

---

## Hibernation

The system saves memory and device state to persistent storage, then powers down.

```text
RAM state
   │
   ▼
Saved to storage
   │
   ▼
Power off
   │
   ▼
Later restore saved state
```

Resume may bypass some ordinary fresh-start initialization.

---

## Fresh boot

Processes and kernel state are constructed anew rather than restored from an earlier running image.

---

## Comparison

| State             | RAM remains powered? | State saved to storage? | Fresh kernel startup? |
| ----------------- | -------------------: | ----------------------: | --------------------: |
| Sleep             |          Usually yes |         Not necessarily |                    No |
| Hibernation       |                   No |                     Yes |          Restore path |
| Shutdown and boot |                   No |       Only normal files |                   Yes |

---

# 15.42 Recovery Mode

A recovery environment provides limited tools when normal startup fails.

It may allow:

* File-system repair
* Password recovery
* Bootloader repair
* Driver removal
* System restoration
* Log inspection
* Backup recovery

```text
Normal boot fails
       │
       ▼
Recovery environment
       │
       ├── Diagnose
       ├── Repair
       └── Restore
```

Recovery software should be protected because it can perform sensitive system operations.

---

# 15.43 Boot Failure: Firmware Stage

Failures before the bootloader may involve:

* Faulty RAM
* CPU initialization problem
* Firmware corruption
* Power instability
* Missing boot device
* Hardware configuration problem

Symptoms may include:

* No display
* Firmware error screen
* Sound or light codes
* Repeated restart
* Boot-device-not-found message

At this stage, the OS kernel has not started, so OS logs may not exist.

---

# 15.44 Boot Failure: Bootloader Stage

Possible causes include:

* Missing bootloader
* Damaged boot configuration
* Incorrect boot entry
* Corrupted system partition
* Signature verification failure
* Missing kernel file
* Unsupported file system

Symptoms may include:

* Boot menu error
* Recovery prompt
* Kernel-not-found message
* Return to firmware

---

# 15.45 Boot Failure: Kernel Stage

Possible causes include:

* Incompatible driver
* Kernel corruption
* Unsupported hardware
* Invalid memory mapping
* Failure mounting root storage
* Kernel exception
* Damaged early startup image

Symptoms may include:

* Kernel panic or stop screen
* Frozen startup
* Repeated restart
* Detailed diagnostic text
* Failure before login services appear

---

# 15.46 Boot Failure: User-Space Stage

The kernel may start successfully, but normal services fail.

Possible causes include:

* Corrupted configuration
* Failed file-system mount
* Broken login service
* Graphics-service failure
* Network dependency timeout
* Full storage
* Incorrect permissions

Symptoms may include:

* Text console but no desktop
* Desktop without networking
* Login loop
* Service failure messages
* Emergency maintenance shell

This demonstrates that “the OS booted” can mean different stages completed.

---

# 15.47 Kernel Panic or Fatal Kernel Error

A **kernel panic** or equivalent fatal kernel stop occurs when the kernel detects that safe continuation is not possible.

Possible causes include:

* Critical kernel defect
* Corrupted kernel memory
* Unrecoverable driver failure
* Essential file-system failure
* Serious hardware error

```text
Critical kernel failure
          │
          ▼
Safe continuation impossible
          │
          ▼
Stop, report, or restart
```

Because the kernel manages all processes and hardware, a fatal kernel failure affects the whole system.

---

# 15.48 Service Startup Timeouts

A service may depend on:

* Network connection
* Storage volume
* Security service
* Another process
* Device availability

If a dependency never becomes ready, startup may stall.

Service managers often use:

* Dependency graphs
* Timeouts
* Retries
* Failure policies
* Parallel startup

A timeout may allow booting to continue in a reduced state.

---

# 15.49 Boot Performance

Boot time is affected by:

* Firmware initialization
* Storage speed
* Number of drivers
* Service dependencies
* File-system checks
* Network waits
* Encryption setup
* Application startup
* Hardware discovery

---

## Techniques that may improve startup

* Start independent services in parallel.
* Delay nonessential services.
* Cache hardware information.
* Use faster storage.
* Avoid unnecessary startup applications.
* Resume from sleep or hibernation.
* Reduce long dependency timeouts.

Faster startup must not bypass required security or consistency checks.

---

# 15.50 Measured Boot

Secure Boot asks:

> Is this component approved before execution?

A related idea, **measured boot**, records cryptographic measurements of startup components.

```text
Firmware component measured
          │
          ▼
Bootloader measured
          │
          ▼
Kernel measured
          │
          ▼
Measurements stored securely
```

Another trusted system can later inspect these measurements to evaluate the startup state.

Verification and measurement are related but distinct.

---

# 15.51 Chain of Trust

A chain of trust begins with a small component trusted by design.

Each stage verifies the next.

```text
Trusted firmware key
         │ verifies
         ▼
Bootloader
         │ verifies
         ▼
Kernel
         │ verifies
         ▼
Drivers or system components
```

If one stage loads unverified privileged code, the chain may be weakened.

The chain is only as trustworthy as:

* Its initial trust anchor
* Key management
* Verification policy
* Implementation correctness

---

# 15.52 Firmware Updates

Firmware itself may contain defects or security vulnerabilities.

Updating firmware can correct:

* Hardware compatibility
* Security problems
* Stability defects
* Device initialization issues

However, firmware updates are sensitive because a failed or malicious update can prevent startup.

Systems may use:

* Signature verification
* Recovery firmware
* Redundant copies
* Protected update modes

---

# 15.53 What Can Go Wrong?

## No bootable device

Firmware cannot find valid boot software.

Possible causes:

* Storage disconnected
* Boot order changed
* Bootloader damaged
* Drive failed

---

## Bootloader corruption

The kernel exists, but no working program can load it.

---

## Secure Boot rejection

A component is:

* Unsigned
* Signed by an untrusted key
* Modified after signing
* Disallowed by policy

Startup stops or enters recovery.

---

## Missing storage driver

The kernel starts but cannot access the device containing the root file system.

---

## Encrypted storage cannot be unlocked

Possible causes:

* Incorrect password
* Missing key
* Security hardware failure
* Damaged encryption metadata

---

## Root file system cannot be mounted

Possible causes:

* File-system damage
* Wrong device selection
* Missing file-system support
* Storage failure

---

## Driver failure

A driver crashes or hangs during device initialization.

---

## Service dependency cycle

Service A waits for B while B waits for A.

This resembles deadlock at the service-management level.

---

## Full system storage

Services may fail because they cannot create:

* Logs
* Temporary files
* Databases
* User-session files

---

## Incorrect system time

Networking, certificates, and authentication may fail when the clock is far from the correct time.

---

## Damaged configuration

The kernel starts, but services cannot establish the normal environment.

---

## Hardware disappears during boot

A removable or unstable device used during startup becomes unavailable.

---

## Firmware or kernel incompatibility

New software expects hardware behavior not provided by the current firmware, or vice versa.

---

# 15.54 Common Misconceptions

## Misconception: “The operating system is running as soon as power is applied”

Firmware runs first.

The OS kernel is loaded and started later.

---

## Misconception: “The firmware is the operating system”

Firmware provides early startup and low-level configuration.

The operating system kernel later manages normal processes, memory, files, scheduling, and devices.

---

## Misconception: “The bootloader remains in charge after the kernel starts”

The bootloader transfers control to the kernel.

It is no longer the main system manager afterward.

---

## Misconception: “The desktop is the operating system kernel”

The desktop is a collection of user-space programs and services.

The kernel runs underneath it.

---

## Misconception: “All device drivers must be loaded before the kernel begins”

Some essential drivers must be available early, but many drivers can be initialized after the kernel starts.

---

## Misconception: “The kernel directly starts every application and service manually”

The kernel creates the first user-space process.

User-space service managers then start most higher-level services and sessions.

---

## Misconception: “A black screen always means the kernel failed”

The kernel may be running while:

* Graphics failed
* Login service failed
* Display configuration is incorrect
* User-space startup stalled

The failure stage must be identified.

---

## Misconception: “Secure Boot encrypts the operating system”

Secure Boot verifies approved startup software.

Storage encryption protects data confidentiality.

They solve different problems.

---

## Misconception: “Secure Boot guarantees that the operating system contains no malware”

It verifies components according to a trust policy.

Approved software can still contain vulnerabilities or malicious behavior.

---

## Misconception: “A restart always completely removes electrical power”

A warm reboot may reset the system without a full power loss.

---

## Misconception: “Sleep is a shortened boot”

Sleep preserves the existing system state in RAM.

A fresh boot creates a new kernel and process environment.

---

## Misconception: “Closing the laptop always saves all state to storage”

The configured action may be:

* Sleep
* Hibernation
* Lock screen
* Shutdown
* Nothing

The behavior depends on system policy.

---

# 15.55 Real-World Analogy: Starting a City Each Morning

Imagine a city that must become operational each morning.

## Firmware

The electrical and infrastructure control center activates essential systems.

## Hardware checks

Power, communications, and transportation controls are tested.

## Bootloader

A coordinator selects the day’s operating plan and summons the central administration.

## Kernel

The central administration takes authority over:

* Roads
* Utilities
* Public resources
* Scheduling
* Safety systems

## Drivers

Specialized departments operate:

* Trains
* Water systems
* Communications
* Traffic lights

## Initial system process

A city manager starts departments and public services.

## System services

Hospitals, transit, communications, and record offices open.

## Login manager

Security desks verify workers and citizens entering controlled areas.

## User session

An authenticated person begins normal daily activity.

Each stage depends on the earlier infrastructure being ready.

---

# 15.56 Connection to Earlier Concepts

## Connection to hardware, kernel, and applications

The boot process constructs the hierarchy introduced at the beginning:

```text
Applications
     │
System services
     │
Kernel
     │
Hardware
```

Startup begins at the bottom and builds upward.

---

## Connection to user and kernel mode

Firmware and boot software run before normal application user mode exists.

The kernel establishes the protected user/kernel boundary before ordinary applications start.

---

## Connection to processes and threads

No ordinary process exists at power-on.

The kernel initializes scheduling and then creates the first user-space process.

That process starts later services and sessions.

---

## Connection to CPU scheduling

The scheduler must be initialized before multiple user processes can share CPU time.

Idle threads are created for periods with no runnable work.

---

## Connection to interrupts and exceptions

The kernel installs handlers before relying on:

* Timer interrupts
* Device interrupts
* Page faults
* System calls
* Processor exceptions

---

## Connection to memory

The kernel discovers physical RAM, constructs page tables, and initializes allocators before creating process address spaces.

---

## Connection to files and file systems

The kernel or early user space mounts the root file system before system programs and configuration files can be accessed normally.

---

## Connection to I/O and devices

Firmware provides minimal device support.

The kernel and its drivers later establish full device management.

---

## Connection to concurrency

As services begin, startup becomes concurrent.

Independent services may initialize in parallel, while dependencies require synchronization.

---

## Connection to deadlocks

Incorrect service dependencies or driver-lock ordering can stall startup.

---

## Connection to security

Secure Boot verifies early components.

Authentication and user permissions become active once system security services start.

---

## Connection to virtual machines

A virtual machine performs a similar boot sequence using:

* Virtual firmware
* Virtual devices
* Guest bootloader
* Guest kernel

A container normally skips this process because it starts as isolated host processes using an already running host kernel.

---

# 15.57 Simplified Model Versus Technical Reality

## Simplified mental model

```text
Power
  │
  ▼
Firmware
  │
  ▼
Bootloader
  │
  ▼
Kernel
  │
  ▼
System services
  │
  ▼
Login and applications
```

This is the model to retain.

---

## More exact reality

Modern startup may involve:

* Several firmware processors
* Security coprocessors
* Multiple boot managers
* Encrypted storage
* Measured boot
* Early RAM file systems
* Kernel modules
* Parallel service startup
* Hardware hot-plugging
* Virtual firmware
* Recovery partitions
* Network booting
* Hibernation restoration
* Immutable system images
* User-space driver components

Some services start before the root file system is fully writable.

Some drivers initialize asynchronously.

The desktop may appear while background startup continues.

The central principle remains:

> Booting is a staged transfer of control that begins with firmware, loads the kernel, initializes core operating-system mechanisms, starts user-space services, and finally creates user sessions and applications.

---

# 15.58 Core Mental Models

## Control-transfer model

```text
Hardware-defined CPU start
           │
           ▼
Firmware
           │
           ▼
Bootloader
           │
           ▼
Kernel
           │
           ▼
Initial user-space process
           │
           ▼
System services
           │
           ▼
User session
```

---

## Kernel initialization model

```text
Kernel begins
     │
     ├── Initialize memory
     ├── Configure exceptions
     ├── Configure interrupts
     ├── Start timers
     ├── Start scheduler
     ├── Start CPU cores
     ├── Initialize drivers
     └── Mount root file system
     │
     ▼
Create first user-space process
```

---

## Early-storage model

```text
Kernel needs root file system
           │
           ▼
Required drivers or decryption missing
           │
           ▼
Use temporary RAM-based environment
           │
           ▼
Prepare storage
           │
           ▼
Mount real root file system
```

---

## Security-chain model

```text
Firmware trust
      │
      ▼
Verified bootloader
      │
      ▼
Verified kernel
      │
      ▼
Verified privileged components
```

---

## Final distinctions

| Concept              | Essential meaning                                              |
| -------------------- | -------------------------------------------------------------- |
| **Firmware**         | Earliest software that initializes enough hardware to continue |
| **POST**             | Early basic hardware checking and initialization               |
| **Boot source**      | Device or environment containing startup software              |
| **Bootloader**       | Program that locates, loads, and starts the kernel             |
| **Kernel entry**     | Point where the kernel begins taking control                   |
| **Early user space** | Temporary environment used to prepare the real system          |
| **Root file system** | Main mounted file hierarchy of the running OS                  |
| **Initial process**  | First user-space process that starts system services           |
| **System service**   | Background process providing OS functionality                  |
| **Secure Boot**      | Verification of approved startup components                    |
| **Cold boot**        | Startup from a powered-off state                               |
| **Warm boot**        | Restart without a complete power removal                       |
| **Sleep**            | Preserve active state in powered RAM                           |
| **Hibernation**      | Save active state to persistent storage                        |
| **Recovery mode**    | Restricted environment for diagnosis and repair                |

The final foundation section connects all major components into complete end-to-end system behavior.

It will trace:

* Computer startup
* Program startup
* File reading
* Memory allocation
* Context switching
* Several programs running together
* Program crashes
* How every major OS component interacts

# Learning Check

Do not look for answers yet.

## Conceptual questions

1. What different responsibilities belong to firmware, the bootloader, the kernel, and the initial user-space process?
2. Why might a kernel need a temporary RAM-based early user-space environment before mounting the real root file system?
3. What is the difference between a fresh boot, sleep resume, and hibernation resume?

## Cause-and-effect questions

4. Why must memory management, exception handling, interrupts, timers, and scheduling be initialized before ordinary applications can run safely?
5. Why can the kernel start successfully even though the graphical desktop or login screen later fails to appear?

## Misconception question

6. A student says, “Secure Boot encrypts the operating system and proves that every loaded component is free of vulnerabilities.” What is wrong with this statement?

## Scenario-based question

7. A computer uses encrypted storage and a graphical login screen. Trace startup from power-on through firmware, bootloader, kernel initialization, early storage unlocking, root-file-system mounting, service startup, graphics initialization, authentication, and user-session creation.

# 16. How the Major Components Work Together

An operating system is not one isolated mechanism.

It is a coordinated system in which:

* Hardware executes instructions.
* The kernel controls privileged resources.
* Processes provide isolated application environments.
* Threads perform execution.
* The scheduler distributes CPU time.
* Virtual memory protects and organizes RAM.
* File systems organize persistent data.
* Drivers control devices.
* System calls connect applications to the kernel.
* Interrupts and exceptions return control to the kernel.
* Synchronization coordinates concurrent work.
* Security policy limits authority.

```text
Users
  │
  ▼
Applications and system services
  │
  │ system calls and messages
  ▼
Kernel
├── Process and thread management
├── CPU scheduling
├── Virtual memory
├── File systems
├── Device management
├── Networking
├── Security and permissions
└── Inter-process communication
  │
  ▼
Hardware
├── CPU cores
├── RAM
├── Storage
├── Network adapter
├── Display and graphics
└── Input/output devices
```

The components are separated for clarity, but real operations normally involve several of them together.

---

# 16.1 The Complete Layered Mental Model

A useful layered model is:

```text
┌──────────────────────────────────────────────┐
│ Users                                        │
├──────────────────────────────────────────────┤
│ Applications                                 │
│ Browser, editor, music player, server        │
├──────────────────────────────────────────────┤
│ Libraries, runtimes, and system services     │
├──────────────────────────────────────────────┤
│ System-call and communication interfaces     │
├──────────────────────────────────────────────┤
│ Kernel                                       │
│ Scheduling, memory, files, devices, security │
├──────────────────────────────────────────────┤
│ Drivers and hardware-control mechanisms      │
├──────────────────────────────────────────────┤
│ CPU, RAM, storage, network, devices           │
└──────────────────────────────────────────────┘
```

Each layer uses services provided by the layer below it.

An application does not normally control an SSD directly.

It requests a file operation from the kernel.

The kernel uses:

* The file system
* Memory caches
* A storage driver
* A hardware controller
* The physical storage device

---

# 16.2 Mechanisms and Policies

The OS contains both **mechanisms** and **policies**.

## Mechanism

A mechanism makes an action possible.

Examples:

* Context switching
* Page-table translation
* Interrupt handling
* File-descriptor lookup
* Lock waiting
* Device command submission

## Policy

A policy determines what should be done.

Examples:

* Which thread runs next
* Whether file access is permitted
* Which memory page should be evicted
* Which process should be terminated under memory pressure
* Which I/O request should be processed first

```text
Mechanism:
“How can the CPU switch threads?”

Policy:
“Which thread should it switch to?”
```

Many OS components separate these concerns.

---

# 16.3 The Central Control Cycle

Most operating-system activity follows a recurring cycle:

```text
Application or hardware event
            │
            ▼
Kernel gains control
            │
            ▼
Validate and interpret event
            │
            ▼
Update system state
            │
            ▼
Perform or schedule work
            │
            ▼
Choose what executes next
            │
            ▼
Return to user mode
```

The kernel does not necessarily remain active continuously.

Much of the time, CPU cores execute user-mode application instructions.

The kernel becomes active when:

* A system call occurs.
* An interrupt arrives.
* An exception occurs.
* A thread must be scheduled.
* Background kernel work is ready.

---

# 16.4 Three Ways Control Reaches the Kernel

Keep this distinction central:

```text
Why did the CPU enter the kernel?
             │
      ┌──────┼────────┐
      │      │        │
Application Hardware Current instruction
requested   signaled caused condition
      │      │        │
System call Interrupt Exception
```

## System call

An application deliberately requests a protected service.

Example:

> Open this file.

## Interrupt

Hardware reports an asynchronous event.

Example:

> The storage read has completed.

## Exception

The current instruction encounters a condition requiring kernel handling.

Example:

> This virtual-memory page is not currently available.

Any of these events may lead to scheduling, but the event itself and the scheduling decision are distinct.

---

# 16.5 Detailed Walkthrough: From Power-On to a Usable Computer

This is the complete startup path.

## Stage 1: Electrical power becomes stable

The CPU enters a predefined reset state.

At this point:

* No application is running.
* No process exists.
* No file system is mounted.
* No scheduler exists.
* Normal virtual memory is not configured.

**Primary component:** Hardware

---

## Stage 2: Firmware begins

The CPU executes firmware instructions from a hardware-defined starting location.

Firmware:

* Initializes essential memory
* Detects basic devices
* Performs startup checks
* Selects a boot source

**Primary component:** Firmware

---

## Stage 3: Bootloader is located

Firmware finds an approved boot application or initial boot region.

Secure Boot may verify it before execution.

**Primary component:** Firmware security and boot manager

---

## Stage 4: Bootloader loads the kernel

The bootloader:

* Selects an OS or kernel version
* Reads the kernel image from storage
* Places it in RAM
* Loads early startup data
* Passes hardware and configuration information

**Primary components:** Bootloader, firmware storage interfaces, RAM

---

## Stage 5: Kernel gains control

The bootloader jumps to the kernel’s entry point.

The kernel begins privileged execution.

**CPU mode:** Privileged kernel mode

---

## Stage 6: Kernel establishes memory management

The kernel:

* Identifies usable physical RAM
* Reserves its own memory
* Initializes physical-frame tracking
* Constructs page tables
* Establishes kernel virtual memory
* Applies read, write, and execute permissions

**Primary component:** Kernel memory manager
**Hardware support:** MMU

---

## Stage 7: Kernel configures exceptions and interrupts

The kernel installs handlers for:

* Page faults
* Invalid instructions
* Timer interrupts
* Device interrupts
* System calls

**Primary component:** Kernel interrupt and exception subsystem
**Hardware support:** CPU and interrupt controller

---

## Stage 8: Kernel initializes scheduling

It creates:

* Ready queues
* Idle threads
* Per-core scheduler state
* Timer accounting
* Thread-management structures

**Primary component:** Scheduler

---

## Stage 9: Additional CPU cores start

Each core receives:

* Kernel stack
* Interrupt state
* Scheduler state
* Memory-management context

**Primary components:** Kernel CPU-management and scheduler subsystems

---

## Stage 10: Essential drivers initialize

The kernel detects controllers and associates drivers with them.

Examples:

* Storage driver
* Display driver
* Input driver
* Network driver

**Primary component:** Device-management subsystem

---

## Stage 11: Early user-space environment may run

If the main storage requires preparation, an early RAM-based environment may:

* Load additional drivers
* Unlock encrypted storage
* Assemble logical volumes
* Locate the root file system

**Primary components:** Kernel and early user-space tools

---

## Stage 12: Root file system is mounted

The kernel gains normal access to:

* System programs
* Configuration files
* Libraries
* Service definitions
* User account information

**Primary component:** File-system subsystem

---

## Stage 13: First user-space process starts

The kernel creates the initial system process.

This requires:

* A process record
* A virtual address space
* At least one thread
* Credentials
* File descriptors
* Executable mappings

**Primary components:** Process manager, virtual-memory manager, file system, scheduler

---

## Stage 14: System services start

Services may include:

* Logging
* Device management
* Networking
* Graphics
* Audio
* Authentication
* Time synchronization
* Login management

**Primary components:** User-space system services using kernel facilities

---

## Stage 15: User authenticates

The login system verifies credentials.

The kernel and security services establish:

* User identity
* Group memberships
* Session credentials
* Resource access
* Desktop or shell environment

**Primary component:** Authentication and authorization services

---

## Stage 16: User applications start

A desktop, shell, browser, editor, or other application is launched.

The machine is now in normal operation.

---

## Complete startup flow

```text
Power
  │
  ▼
Firmware
  │
  ▼
Bootloader
  │
  ▼
Kernel
  ├── Memory
  ├── Interrupts
  ├── Scheduler
  ├── Drivers
  └── Root file system
  │
  ▼
Initial system process
  │
  ▼
System services
  │
  ▼
Authentication
  │
  ▼
User session
  │
  ▼
Applications
```

---

# 16.6 Detailed Walkthrough: Starting a Program

Suppose the user opens a text editor.

## Stage 1: User action is received

The user clicks an icon or enters a command.

A keyboard or mouse event travels through:

```text
Input hardware
      │
      ▼
Device controller
      │ interrupt
      ▼
Input driver
      │
      ▼
Graphical system or shell
```

**Components:** Hardware, driver, kernel input system, user-space interface

---

## Stage 2: Existing process requests program creation

The desktop or shell asks the kernel to start the editor.

This occurs through system calls or a system service.

**CPU mode:** User → kernel

---

## Stage 3: Kernel validates the request

The kernel checks:

* Does the executable exist?
* May the requesting identity execute it?
* Is the file a valid executable format?
* Do sandbox or policy rules permit it?
* Are required resources available?

**Components:** Security subsystem, file system, process manager

---

## Stage 4: Process record is created

The kernel assigns a process identity and records:

* Credentials
* Parent relationship
* Resource limits
* Security policy
* Process state

**Component:** Process manager

---

## Stage 5: Virtual address space is created

The kernel constructs a new memory environment for the process.

It prepares regions for:

* Program instructions
* Static data
* Heap
* Libraries
* Thread stack
* Shared mappings

**Component:** Virtual-memory manager

---

## Stage 6: Executable and libraries are mapped

The kernel does not necessarily copy the whole program into RAM immediately.

It creates file-backed virtual-memory mappings.

```text
Executable file
      │
      ▼
Virtual instruction pages
      │
      ▼
Loaded into physical frames when needed
```

**Components:** File system and virtual-memory manager

---

## Stage 7: Initial thread is created

The thread receives:

* Initial instruction location
* Stack
* CPU context
* Scheduling information

**Component:** Thread manager

---

## Stage 8: Process descriptors are prepared

The process may inherit or receive:

* Standard input
* Standard output
* Standard error
* Desktop communication channel
* Selected open files
* Environment information

**Components:** Process manager and I/O subsystem

---

## Stage 9: Thread becomes ready

```text
New thread
    │
    ▼
READY
```

It is able to execute but may not yet have a CPU core.

**Component:** Scheduler

---

## Stage 10: Scheduler selects it

The scheduler chooses the editor thread for a core.

A context switch restores its initial CPU state.

```text
READY → RUNNING
```

---

## Stage 11: CPU enters user mode

The editor begins executing its program instructions.

**CPU mode:** Kernel → user

---

## Stage 12: Demand page faults occur

The first instruction page or library page may not yet be in RAM.

1. MMU cannot complete the access.
2. CPU raises a page fault.
3. Kernel locates the executable data.
4. Storage may be read.
5. Page table is updated.
6. Instruction retries.

**Components:** CPU, MMU, virtual-memory manager, file system, storage driver

---

## Stage 13: Application initializes

The editor may:

* Allocate heap memory
* Create more threads
* Load configuration
* Connect to graphics services
* Register input handlers
* Restore a previous session

The program is now fully active.

---

# 16.7 Program, Process, and Thread in the Full System

Keep the distinctions clear:

| Concept     | Role                                       |
| ----------- | ------------------------------------------ |
| **Program** | Stored instructions and data               |
| **Process** | Running resource and isolation environment |
| **Thread**  | Execution sequence scheduled on a CPU      |

```text
Program file
    │ started
    ▼
Process
├── Address space
├── Credentials
├── File descriptors
├── Resources
└── Threads
    ├── Thread A
    └── Thread B
```

The scheduler does not normally schedule a stored program.

It schedules runnable threads belonging to processes.

---

# 16.8 Detailed Walkthrough: Allocating Memory

Suppose the text editor needs memory for a large document.

## Stage 1: Application asks its allocator

The application requests a heap block.

**Component:** User-space memory allocator
**Mode:** User mode

---

## Stage 2: Allocator checks existing free space

If a suitable free block already exists, it returns that block without entering the kernel.

```text
Application request
       │
       ▼
Allocator free block available?
   ┌───┴────┐
   │        │
  Yes       No
   │        │
Return    Request more memory
```

---

## Stage 3: Allocator requests more virtual memory

If necessary, it makes a system call.

**Mode:** User → kernel
**Component:** Kernel virtual-memory manager

---

## Stage 4: Kernel validates the request

It checks:

* Process limits
* Address-space availability
* System memory policy
* Requested permissions
* Sandbox restrictions

---

## Stage 5: Kernel reserves a virtual range

The process receives valid virtual addresses.

Physical RAM may not yet be assigned to every page.

```text
Virtual heap pages
├── Page A: reserved
├── Page B: reserved
└── Page C: reserved
```

---

## Stage 6: System call returns

The allocator divides the region and returns an address to the application.

**Mode:** Kernel → user

---

## Stage 7: Application first writes to the memory

The page may not be physically backed.

The MMU detects this and raises a page fault.

---

## Stage 8: Kernel handles the page fault

It verifies that:

* The address belongs to a valid heap region.
* Writing is permitted.
* Physical backing should be supplied.

---

## Stage 9: Physical frame is selected

The kernel may use:

* A free frame
* A reclaimed cache frame
* A frame freed by another process
* A frame made available after eviction

---

## Stage 10: Frame is cleared

The kernel usually ensures that old data from another process is not exposed.

---

## Stage 11: Page table is updated

```text
Virtual page A → Physical frame 72
Permissions: read and write
```

---

## Stage 12: Instruction retries

The application’s original write succeeds.

The application normally does not know that the page fault occurred.

---

## Full allocation flow

```text
Application
    │
    ▼
User-space allocator
    │ no suitable block
    ▼
Memory system call
    │
    ▼
Reserve virtual pages
    │
    ▼
Return virtual address
    │
    ▼
First memory access
    │
    ▼
Page fault
    │
    ▼
Assign physical frame
    │
    ▼
Retry access
```

---

# 16.9 Stack and Heap in the Complete Process

A process may have several threads.

Each thread normally has its own stack:

```text
Process
├── Shared program code
├── Shared process data
├── Shared heap
│
├── Thread A
│   └── Stack A
│
└── Thread B
    └── Stack B
```

## Stack

Tracks nested active operations for one thread.

## Heap

Stores dynamically managed process data.

## Shared-memory risk

Because threads share the heap, they may access the same objects.

This requires synchronization when data is mutable.

---

# 16.10 Detailed Walkthrough: Reading a File

Suppose the editor opens a document.

## Stage 1: Application supplies a path

Example:

```text
Documents/report.txt
```

The application requests read access.

**Mode:** User mode

---

## Stage 2: Open system call occurs

Control enters the kernel through a controlled system-call entry.

**Mode:** User → kernel

---

## Stage 3: Kernel resolves the path

The kernel begins from:

* Root, for an absolute path
* Current working directory, for a relative path

It follows directory entries one component at a time.

```text
Documents
    │
    ▼
report.txt
```

**Component:** File-system layer

---

## Stage 4: Permission checks occur

The kernel examines:

* Process credentials
* Directory traversal permission
* File ownership
* ACLs
* Sandbox restrictions
* Security labels

**Component:** Security subsystem

---

## Stage 5: File descriptor is created

The kernel creates open-file state containing:

* Target file
* Access mode
* Current offset
* Status information

The process receives a descriptor, such as:

```text
Descriptor 5 → report.txt
```

---

## Stage 6: Application requests a read

The application supplies:

* Descriptor
* Destination memory
* Requested byte count

A second system call enters the kernel.

---

## Stage 7: Kernel validates the request

It checks:

* Descriptor validity
* Read permission
* Destination memory validity
* Requested length
* Current file position

---

## Stage 8: File cache is checked

### Cache hit

Data is already in RAM.

The kernel can supply it quickly.

### Cache miss

The file system identifies required storage blocks.

---

## Stage 9: Driver prepares storage I/O

The storage driver creates a device-specific command.

It may configure DMA so the controller can transfer data into RAM.

**Components:** File system, storage driver, controller

---

## Stage 10: Calling thread waits

Because the result is not ready:

```text
RUNNING → WAITING
```

The thread no longer consumes CPU time.

**Component:** Scheduler

---

## Stage 11: Another thread runs

The CPU may execute:

* Another editor thread
* Browser thread
* Music thread
* Kernel background work

---

## Stage 12: Storage device retrieves data

The controller and SSD work independently.

DMA may transfer the data into RAM.

---

## Stage 13: Device reports completion

The controller generates an interrupt.

**Mode:** User or kernel → kernel

---

## Stage 14: Interrupt handler processes completion

The driver:

* Identifies the completed request
* Checks status
* Acknowledges the interrupt
* Updates kernel I/O state

---

## Stage 15: File data enters the cache

The kernel records the data as available in RAM.

---

## Stage 16: Original thread becomes ready

```text
WAITING → READY
```

It does not necessarily run immediately.

---

## Stage 17: Scheduler selects the thread

A context switch restores the editor thread’s saved CPU context.

```text
READY → RUNNING
```

---

## Stage 18: Read system call completes

The kernel:

* Makes data available in process memory
* Advances the file offset
* Returns the number of bytes read

**Mode:** Kernel → user

---

## Stage 19: Editor interprets the bytes

The application determines:

* Character encoding
* Document structure
* Formatting
* Display representation

The file system did not interpret the document format for the editor.

---

## Full file-read path

```text
Editor
  │ open system call
  ▼
Kernel path resolution
  │
  ▼
Permission check
  │
  ▼
File descriptor
  │ read system call
  ▼
File cache
  │ miss
  ▼
File system
  │
  ▼
Storage driver
  │
  ▼
Controller and SSD
  │ interrupt
  ▼
Kernel completion handling
  │
  ▼
Thread becomes ready
  │
  ▼
Scheduler resumes editor
  │
  ▼
Data returned to application
```

---

# 16.11 Detailed Walkthrough: Saving a File

Suppose the user edits the document and selects Save.

## Stage 1: Application holds updated document data

The updated information exists in the editor’s process memory.

---

## Stage 2: Synchronization protects document state

If an auto-save thread and interface thread share the document, the editor may:

1. Acquire a mutex.
2. Copy a consistent document snapshot.
3. Release the mutex.
4. Perform slow storage work outside the lock.

**Components:** Application threads and synchronization primitives

---

## Stage 3: Write system call occurs

The application supplies:

* File descriptor
* Source memory
* Byte count

---

## Stage 4: Kernel validates access

It checks:

* Descriptor is writable
* Source memory is readable
* Process has permission
* Storage limits permit growth

---

## Stage 5: File-system cache is updated

The data may first enter RAM-backed dirty pages.

```text
Process memory
      │
      ▼
Kernel file cache
      │
      ▼
Persistent storage later
```

---

## Stage 6: File-system metadata changes

The file system may update:

* File size
* Modification time
* Block mappings
* Allocation records

---

## Stage 7: Write system call returns

The application may receive success after the kernel accepts the data into cache.

This does not always mean the storage hardware has physically committed it.

---

## Stage 8: Kernel schedules storage output

Dirty pages are sent through:

* File system
* Storage request queue
* Driver
* Controller
* Device

---

## Stage 9: Device completes the write

An interrupt reports completion or error.

---

## Stage 10: Durability requirements are handled

Applications needing stronger guarantees may explicitly request that relevant cached data and metadata reach persistent storage.

---

# 16.12 Detailed Walkthrough: A Context Switch

Suppose the browser is running when its time slice expires.

## Stage 1: Browser thread executes in user mode

The CPU registers contain the browser thread’s current state.

---

## Stage 2: Timer interrupt occurs

The CPU pauses ordinary execution and enters the kernel.

---

## Stage 3: Kernel saves the browser thread’s context

This includes conceptually:

* Instruction pointer
* Stack pointer
* General registers
* Processor flags
* Architecture-specific state

---

## Stage 4: Browser state changes

If it can continue:

```text
RUNNING → READY
```

---

## Stage 5: Scheduler selects another thread

Suppose it selects a music thread because audio work is ready.

The scheduler considers:

* Priority
* Fairness
* Waiting time
* CPU affinity
* Recent runtime
* Timing requirements

---

## Stage 6: Address-space context is prepared

If the music thread belongs to another process, the kernel activates that process’s memory mappings.

---

## Stage 7: Music thread context is restored

The CPU receives the music thread’s saved:

* Registers
* Stack location
* Instruction position

---

## Stage 8: CPU returns to user mode

The music thread resumes exactly where it stopped.

```text
Browser user code
       │ timer interrupt
       ▼
Kernel
├── Save browser
├── Choose music thread
└── Restore music thread
       │
       ▼
Music user code
```

---

## Context switch versus mode switch

A timer interrupt followed by the browser resuming causes mode switches but no thread context switch.

```text
Browser user mode
      │
Kernel mode
      │
Browser user mode
```

A switch from browser to music changes both privilege mode and thread context.

---

# 16.13 Detailed Walkthrough: Several Programs Running Together

Assume one CPU core for simplicity.

Active programs:

* Browser
* Text editor
* Music player
* File downloader

Their threads may be in these states:

```text
Browser:      READY
Editor:       WAITING for keyboard input
Music player: READY
Downloader:   WAITING for network data
```

---

## Timeline stage 1: Music thread runs

The scheduler selects music work because audio output requires timely data.

```text
Music:
READY → RUNNING
```

It decodes audio and fills an output buffer.

---

## Timeline stage 2: Music thread waits

After supplying enough audio, it waits for the next buffer event.

```text
Music:
RUNNING → WAITING
```

---

## Timeline stage 3: Browser runs

The scheduler selects the browser.

It performs page layout calculations.

```text
Browser:
READY → RUNNING
```

---

## Timeline stage 4: Keyboard interrupt occurs

The user presses a key.

1. Keyboard controller signals the CPU.
2. Kernel input driver handles the event.
3. Graphical system associates it with the editor.
4. Editor thread becomes ready.

```text
Editor:
WAITING → READY
```

---

## Timeline stage 5: Scheduler selects editor

The browser may be preempted.

```text
Browser:
RUNNING → READY

Editor:
READY → RUNNING
```

The editor updates the document.

---

## Timeline stage 6: Editor initiates auto-save

A worker thread requests a file write.

The write may enter the file cache quickly or wait for storage.

---

## Timeline stage 7: Network packet arrives

The network adapter uses DMA to place data in RAM and signals an interrupt.

The kernel networking subsystem identifies the downloader’s connection.

```text
Downloader:
WAITING → READY
```

---

## Timeline stage 8: Downloader runs

The scheduler gives it CPU time.

It processes the received data and may request a file write.

---

## Timeline stage 9: Audio device requires more data

An audio completion event wakes the music thread.

The scheduler ensures it runs soon enough to avoid a buffer underrun.

---

## Simplified CPU timeline

```text
Time ─────────────────────────────────────────────────────▶

CPU:
[Music][Browser][Editor][Downloader][Browser][Music]
```

## Simultaneous device activity

```text
CPU:       Browser calculations
SSD:       Writing downloaded data
Network:   Receiving more packets
Audio:     Playing buffered samples
Keyboard:  Waiting for user input
```

The CPU and devices operate concurrently.

The OS coordinates their interactions through:

* Scheduling
* Interrupts
* Buffers
* Drivers
* Wait queues
* Memory mappings

---

# 16.14 Concurrency Inside One Application

Suppose the editor contains:

* Interface thread
* File-loading thread
* Auto-save thread
* Spell-check thread

They share the process heap.

```text
Editor process
├── Interface thread
├── File-loading thread
├── Auto-save thread
└── Spell-check thread
        │
        ▼
Shared document data
```

Without coordination, they may produce:

* Lost updates
* Inconsistent document state
* Use-after-release defects
* Corrupted internal structures

The application uses synchronization such as:

* Mutexes
* Condition variables
* Queues
* Immutable snapshots
* Ownership transfer

---

## Safe auto-save pattern

```text
Interface thread:
lock document
update text and metadata
restore invariants
unlock document

Auto-save thread:
lock document
copy consistent snapshot
unlock document
write snapshot to file
```

The slow file operation occurs outside the document lock.

This preserves correctness without blocking editing for the entire storage operation.

---

# 16.15 Race and Deadlock in the Integrated System

## Race example

Two threads both perform:

1. Check whether a queue has space.
2. Insert an item.

Without one protected critical section, both may act on the same outdated observation.

---

## Deadlock example

Thread A:

```text
Holds document lock
Waits for file-service response
```

Thread B:

```text
Handles file-service response
Needs document lock
```

Neither can continue.

---

## Prevention principles

* Protect complete logical operations.
* Define what each lock protects.
* Keep critical sections short.
* Avoid blocking I/O while holding locks.
* Acquire multiple locks in a consistent order.
* Release locks on every failure path.
* Recheck conditions after waking.

---

# 16.16 Detailed Walkthrough: A Program Crash

Suppose the editor dereferences an invalid memory address.

## Stage 1: Faulting instruction executes

The editor thread attempts to read or write a virtual address.

**Mode:** User mode

---

## Stage 2: MMU checks the mapping

The page table contains no valid permitted mapping.

The hardware blocks the operation.

---

## Stage 3: CPU raises an exception

Control transfers to the kernel’s memory-exception handler.

**Mode:** User → kernel

---

## Stage 4: Kernel examines the cause

The kernel checks whether the access represents:

* Valid demand allocation
* Stack growth
* Copy-on-write
* A file-backed page needing loading
* A genuinely invalid address

Assume the address is invalid.

---

## Stage 5: Process failure policy is applied

The kernel may:

* Notify an application runtime
* Create diagnostic information
* Terminate the faulting process

---

## Stage 6: Threads are stopped

Other threads in the same process are normally terminated as part of process failure.

---

## Stage 7: Scheduler removes them

They no longer appear in ready or waiting collections.

---

## Stage 8: Virtual memory is reclaimed

The kernel releases:

* Page tables
* Private physical frames
* Stack pages
* Heap pages
* Mapped libraries
* File-backed mappings

Shared pages remain if other processes still use them.

---

## Stage 9: File descriptors are closed

The kernel releases references to:

* Files
* Network connections
* Pipes
* Devices

Pending operations are canceled or safely completed according to OS rules.

---

## Stage 10: Security and device sessions end

Camera, microphone, graphics, or other controlled resources are released.

---

## Stage 11: Parent or system service is notified

The desktop may show:

> The application closed unexpectedly.

A service manager may restart a crashed background service.

---

## Stage 12: Other processes continue

The browser, music player, and kernel normally remain active because process isolation contained the invalid memory access.

---

## Crash flow

```text
Invalid user memory access
          │
          ▼
MMU blocks access
          │
          ▼
CPU raises exception
          │
          ▼
Kernel investigates
          │
          ▼
Access is invalid
          │
          ▼
Terminate process
          │
          ├── Stop threads
          ├── Reclaim memory
          ├── Close descriptors
          ├── Cancel I/O
          └── Report failure
```

---

# 16.17 Application Crash Versus Kernel Crash

## Application crash

The failure occurs in user mode.

The kernel remains trusted and can clean up the process.

```text
Faulty application
       │
       ▼
Kernel contains failure
       │
       ▼
Other processes continue
```

## Kernel crash

The failure occurs in privileged kernel code.

The kernel may no longer trust:

* Memory-management state
* Scheduler state
* File-system state
* Device state
* Security policy

The system may stop or restart.

```text
Critical kernel failure
        │
        ▼
Safe continuation uncertain
        │
        ▼
System stop or restart
```

Process isolation cannot protect the system from every kernel defect because the kernel controls the isolation mechanisms themselves.

---

# 16.18 Security Throughout an Operation

Security is not checked only at login.

It is applied throughout the system.

Suppose a photo application opens a private image.

Checks may include:

1. User authenticated successfully.
2. Application is allowed to request the file.
3. Directory traversal is permitted.
4. File read permission is granted.
5. Sandbox permits access to that location.
6. File descriptor belongs to the process.
7. Destination memory is writable.
8. Device access remains controlled by the kernel.

```text
Application request
       │
       ▼
Identity check
       │
       ▼
Permission and sandbox check
       │
       ▼
Resource-table validation
       │
       ▼
Memory validation
       │
       ▼
Operation performed
```

A process cannot generally bypass a denied file operation by reading raw storage because raw-device access is separately protected.

---

# 16.19 Process Isolation as a Combined System

Process isolation depends on several mechanisms working together.

## CPU privilege isolation

User-mode code cannot execute privileged instructions freely.

## Memory isolation

The MMU applies per-process page tables and permissions.

## Resource isolation

File descriptors are interpreted in the calling process’s table.

## Identity isolation

System calls are evaluated using process credentials.

## Communication isolation

Processes communicate through controlled mechanisms.

## Failure isolation

The kernel can terminate one process and reclaim its resources.

```text
Process boundary
├── CPU privilege restrictions
├── Virtual address space
├── Descriptor table
├── Credentials
├── Sandbox policy
└── Resource limits
```

If one of these mechanisms is misconfigured or defective, the isolation boundary can weaken.

---

# 16.20 Physical Resources and Virtual Abstractions

The operating system repeatedly turns physical resources into virtual abstractions.

| Physical resource     | OS abstraction                        |
| --------------------- | ------------------------------------- |
| CPU cores             | Threads receiving scheduled execution |
| Physical RAM          | Per-process virtual address spaces    |
| Storage blocks        | Files and directories                 |
| Network adapter       | Sockets and connections               |
| Display hardware      | Windows and graphical surfaces        |
| Keyboard hardware     | Input events                          |
| Physical machine      | Virtual machines                      |
| Host kernel resources | Containers                            |

This is one of the OS’s most important general patterns:

> Hide difficult physical details behind controlled logical interfaces.

---

# 16.21 The Illusions Created by the OS

The OS creates several useful illusions.

## CPU illusion

Each runnable process appears to have continuing execution, even though threads take turns.

## Memory illusion

Each process appears to have its own large address space, even though RAM is shared.

## Storage illusion

Files appear as named byte sequences, even though storage uses physical blocks.

## Device illusion

Applications use common interfaces rather than hardware-specific command registers.

## Machine illusion

A VM appears to be an independent computer even though hardware is shared.

These are not deceptions in a harmful sense.

They are carefully enforced abstractions.

---

# 16.22 What Happens During a System Call in the Integrated Model

Consider any system call.

```text
User thread
   │
   ▼
Prepare arguments in process memory
   │
   ▼
Execute controlled entry instruction
   │
   ▼
CPU enters kernel mode
   │
   ▼
Kernel identifies process and requested service
   │
   ▼
Validate memory, descriptor, permission, and state
   │
   ▼
Perform immediate work or start asynchronous operation
   │
   ├── Return immediately
   └── Block thread and schedule another
```

A system call may therefore involve:

* CPU privilege transition
* Memory validation
* Security checks
* File-descriptor lookup
* Device operations
* Thread-state changes
* Context switching
* Interrupt handling

---

# 16.23 What Happens During a Page Fault in the Integrated Model

```text
Thread accesses virtual address
          │
          ▼
MMU checks page table
          │
          ▼
Current mapping cannot satisfy access
          │
          ▼
CPU raises exception
          │
          ▼
Kernel checks process memory map
     ┌────┴─────────────┐
     │                  │
Recoverable          Invalid
     │                  │
Assign/load/copy page   Report or terminate
     │
Update page table
     │
Retry instruction
```

A page fault connects:

* CPU exception hardware
* Page tables
* Physical-memory allocation
* File systems or swap
* Storage I/O
* Scheduling
* Process isolation

---

# 16.24 What Happens During Device I/O in the Integrated Model

```text
Application requests I/O
          │
          ▼
System call
          │
          ▼
Kernel validates request
          │
          ▼
Driver configures controller
          │
          ▼
DMA transfers between device and RAM
          │
          ▼
Calling thread may wait
          │
          ▼
Scheduler runs another thread
          │
          ▼
Device completes and interrupts CPU
          │
          ▼
Kernel handles completion
          │
          ▼
Waiting thread becomes ready
          │
          ▼
Scheduler resumes it
```

This is one of the clearest examples of every major OS mechanism cooperating.

---

# 16.25 Virtual Machines in the Complete Model

Inside a VM, the same operating-system concepts exist again.

```text
Guest application
       │ system call
       ▼
Guest kernel
├── Guest scheduler
├── Guest virtual memory
├── Guest file system
└── Guest drivers
       │ virtual hardware
       ▼
Hypervisor
       │
       ▼
Physical hardware
```

A guest file read may pass through:

1. Guest application
2. Guest kernel
3. Guest file system
4. Guest virtual-storage driver
5. Hypervisor
6. Host storage system
7. Physical device
8. Back through the same layers

The VM contains its own kernel and performs its own boot process.

---

# 16.26 Containers in the Complete Model

A container does not normally contain a separate kernel.

```text
Containerized application
          │ system call
          ▼
Host kernel
├── Host scheduler
├── Host virtual memory
├── Host file system
├── Host networking
└── Host drivers
          │
          ▼
Physical hardware
```

The host kernel applies:

* Isolated process views
* File-system views
* Network views
* Resource limits
* Capability restrictions
* Security labels

Container processes remain host processes.

---

# 16.27 VM and Container Failure Boundaries

## Application crash in a container

The host kernel terminates the process. Other containers normally continue.

## Host-kernel crash

All containers sharing that kernel may be affected.

## Application crash in a VM

The guest kernel normally contains it.

## Guest-kernel crash

The whole VM may stop, while other VMs and the hypervisor continue.

## Hypervisor failure

Several or all VMs on the host may be affected.

```text
Failure boundaries

Application
    │
    ▼
Process boundary
    │
    ▼
Container host-kernel boundary
or VM guest-kernel boundary
    │
    ▼
Hypervisor boundary
    │
    ▼
Physical machine
```

---

# 16.28 Resource Pressure Across the System

Suppose several applications use too much memory.

## Stage 1: Free RAM decreases

Heap pages, stacks, file cache, and kernel data occupy physical frames.

## Stage 2: Kernel reclaims easy pages

It may discard clean file-cache pages.

## Stage 3: Less-active pages are moved or compressed

Depending on the OS, memory may be:

* Compressed
* Written to swap
* Reconstructed later

## Stage 4: Page faults increase

Applications must wait for pages to return.

## Stage 5: Storage activity increases

The CPU may spend more time waiting for memory-related I/O.

## Stage 6: Thrashing may begin

Pages are repeatedly evicted and reloaded.

## Stage 7: Allocation requests fail or processes are terminated

The kernel applies its memory-pressure policy.

This affects:

* Memory manager
* Scheduler
* Storage subsystem
* Process manager
* Security and resource limits

---

# 16.29 CPU Overload Across the System

Suppose hundreds of threads are ready.

The scheduler must distribute limited cores.

Possible effects include:

* Longer response time
* More context switching
* Cache disruption
* Priority delays
* Reduced throughput
* Starvation if policy is unfair

Creating more threads does not create more CPU execution capacity.

```text
Runnable threads: 200
CPU cores:          8
```

At most a limited number can execute physically at one moment.

---

# 16.30 Storage Failure Across the System

Suppose the SSD begins returning errors.

The failure may propagate through several layers:

```text
SSD reports error
      │
      ▼
Controller reports status
      │
      ▼
Driver interprets failure
      │
      ▼
File system receives I/O error
      │
      ▼
Kernel reports operation failure
      │
      ▼
Application receives error
```

Possible system effects include:

* File reads fail
* Writes become unsafe
* File system becomes read-only
* Services fail
* Boot may fail
* Data may be lost

The application sees a logical error, while lower layers handle hardware-specific details.

---

# 16.31 Network Request Across the Complete System

Suppose a browser requests a webpage.

## Outgoing path

```text
Browser
  │
  ▼
Networking system call
  │
  ▼
Kernel socket state
  │
  ▼
Protocol processing
  │
  ▼
Network driver
  │
  ▼
Adapter and physical network
```

## Waiting

The browser thread may wait for a response.

```text
RUNNING → WAITING
```

## Incoming path

```text
Network signal
    │
    ▼
Adapter receives packet
    │ DMA
    ▼
RAM
    │ interrupt
    ▼
Kernel networking subsystem
    │
    ▼
Correct connection identified
    │
    ▼
Browser thread becomes ready
```

## Application processing

The browser:

* Parses content
* Allocates memory
* Reads cached files
* Runs several threads
* Requests graphical updates

A single webpage request may involve nearly every major OS subsystem.

---

# 16.32 User Input to Visible Output

Suppose the user types a letter in the editor.

```text
Physical keypress
      │
      ▼
Keyboard controller
      │ interrupt
      ▼
Kernel input driver
      │
      ▼
Input service or graphical system
      │
      ▼
Editor thread becomes ready
      │
      ▼
Scheduler runs editor
      │
      ▼
Editor updates heap data
      │
      ▼
Synchronization protects document state
      │
      ▼
Graphics request
      │
      ▼
Graphics service and driver
      │
      ▼
Display hardware
      │
      ▼
Letter appears
```

This apparently simple action uses:

* Hardware input
* Interrupts
* Drivers
* Scheduling
* Threads
* Process memory
* Synchronization
* IPC
* Graphics output

---

# 16.33 A Unified State Model

The OS tracks changing states across many resource types.

## Thread state

```text
READY ↔ RUNNING ↔ WAITING → TERMINATED
```

## Lock state

```text
AVAILABLE ↔ OWNED
```

## Memory-page state

```text
Mapped and present
Mapped but absent
Invalid
Dirty
Clean
Shared
```

## File state

```text
Named
Open
Modified in cache
Persisted
Deleted but still referenced
```

## Device-request state

```text
Queued
Active
Completed
Failed
Canceled
```

## Process state

```text
Created
Running
Exiting
Terminated
```

Correct OS behavior requires transitions between these states to remain consistent.

---

# 16.34 The Kernel’s Major Internal Responsibilities

A simplified kernel map is:

```text
Kernel
├── System-call interface
├── Process and thread manager
├── CPU scheduler
├── Virtual-memory manager
├── File-system layer
├── Device and driver framework
├── Networking subsystem
├── Security and credentials
├── IPC and synchronization support
└── Interrupt and exception handling
```

These are conceptual divisions.

Real kernels may organize them differently, and the subsystems call one another extensively.

---

# 16.35 What the Kernel Does Not Normally Do

The kernel usually does not:

* Interpret a word-processing document
* Render every application button itself
* Decide the meaning of an image file
* Write a user’s email
* Implement every network service
* Manage all application data structures

These tasks belong largely to applications and user-space services.

The kernel provides protected mechanisms that make those tasks possible.

---

# 16.36 What Applications Must Still Do Correctly

The OS provides isolation and services, but applications remain responsible for:

* Handling system-call errors
* Synchronizing their threads
* Avoiding memory misuse
* Validating untrusted input
* Preserving file-format consistency
* Releasing resources
* Respecting permission results
* Recovering from network and device failures
* Avoiding deadlocks
* Protecting application-level secrets

The OS can stop an application from directly corrupting another process’s private RAM, but it cannot automatically make the application’s internal logic correct.

---

# 16.37 Failure Containment Hierarchy

A useful hierarchy is:

```text
Defective operation
      │
      ▼
Can one thread recover?
      │
      ▼
Can one process be terminated?
      │
      ▼
Can one container or VM be restarted?
      │
      ▼
Must a service group restart?
      │
      ▼
Must the whole OS restart?
      │
      ▼
Is hardware repair required?
```

Good system design tries to contain failures at the smallest reasonable level.

Examples:

* Invalid user memory access → terminate one process
* Failed service → restart one service
* Guest-kernel failure → restart one VM
* Host-kernel failure → restart host
* Physical RAM failure → replace hardware

---

# 16.38 Simplified Model Versus Technical Reality

## Simplified model

```text
Applications ask.
Kernel decides and coordinates.
Hardware executes and transfers.
```

This model is useful, but real systems contain more layers.

---

## More exact reality

A modern operation may involve:

* Application runtime
* Shared libraries
* User-space service
* System call
* Kernel subsystem
* Driver
* Firmware
* Device controller
* Hypervisor
* Remote server
* Security hardware

For example, a file shown in a container inside a VM may involve:

```text
Container application
      │
Guest host kernel
      │
Virtual disk
      │
Hypervisor
      │
Host storage service
      │
Physical storage controller
      │
SSD firmware
```

The abstractions remain valuable because each layer exposes a manageable interface.

---

# 16.39 The Most Important Connections

## Processes and virtual memory

A process is partly defined by its private virtual address space.

## Threads and scheduling

Threads are the execution units selected by the scheduler.

## System calls and kernel mode

System calls provide controlled access to privileged services.

## Interrupts and devices

Devices use interrupts to report asynchronous events.

## Exceptions and memory

Page faults and invalid accesses enter the kernel through exceptions.

## Files and storage

Files provide logical persistent data over physical storage blocks.

## File descriptors and processes

Descriptors are process-local references to kernel-managed resources.

## I/O and scheduling

Threads wait while devices work, allowing other threads to use the CPU.

## Concurrency and synchronization

Shared mutable state requires coordination.

## Security and isolation

Credentials, page tables, resource tables, and policy limit authority.

## Boot and all other components

Booting constructs every mechanism needed for normal operation.

---

# 16.40 Complete System Example: Opening and Editing a Document

This final example combines the entire foundation.

## 1. Computer starts

Firmware loads the bootloader.

The bootloader loads the kernel.

The kernel initializes:

* RAM
* Scheduling
* Drivers
* File systems
* Security
* System services

---

## 2. User logs in

Authentication establishes an identity.

A session is created with:

* Credentials
* Desktop processes
* Environment
* Access to permitted resources

---

## 3. User starts an editor

The desktop asks the kernel to create a process.

The kernel:

* Validates the executable
* Creates an address space
* Maps program pages
* Creates a thread
* Adds it to the ready queue

---

## 4. Scheduler runs the editor

The thread begins in user mode.

Demand page faults load required code and libraries.

---

## 5. Editor allocates memory

Its allocator obtains heap space.

Newly touched pages receive physical frames through recoverable page faults.

---

## 6. User chooses a document

The editor sends an open system call.

The kernel:

* Resolves the path
* Checks permissions
* Creates open-file state
* Returns a descriptor

---

## 7. Editor reads the document

If data is not cached:

* Storage driver begins I/O
* Editor thread waits
* Another thread runs
* SSD transfers data through DMA
* Interrupt reports completion
* Editor thread becomes ready
* Scheduler resumes it

---

## 8. Editor displays contents

The application interprets file bytes.

It sends graphical commands through system services and drivers.

The display hardware refreshes the screen.

---

## 9. User types

Keyboard interrupt reaches the input driver.

The input system directs the event to the editor.

The editor thread updates document memory.

---

## 10. Auto-save occurs concurrently

An auto-save thread acquires a document mutex.

It copies a consistent snapshot, releases the lock, and writes the file.

---

## 11. Browser and music player remain active

The scheduler switches among ready threads.

The audio device consumes buffered samples while the CPU runs other work.

Network and storage devices continue independently.

---

## 12. Editor crashes

An invalid memory access raises an exception.

The kernel determines that the fault is fatal.

It:

* Stops editor threads
* Reclaims memory
* Closes descriptors
* Cancels pending I/O
* Releases locks and device sessions where possible
* Reports the failure

---

## 13. Other applications continue

The browser and music player retain:

* Separate address spaces
* Separate descriptors
* Separate process state

The kernel and hardware isolation mechanisms contain the crash.

---

## Complete diagram

```text
Power-on
   │
   ▼
Firmware → Bootloader → Kernel → Services → Login
                                            │
                                            ▼
                                      Start editor
                                            │
                          ┌─────────────────┼─────────────────┐
                          │                 │                 │
                          ▼                 ▼                 ▼
                     Virtual memory     Scheduler        Security
                          │                 │                 │
                          └──────────┬──────┴─────────────────┘
                                     ▼
                                Editor runs
                                     │
                                     ▼
                              Open file system call
                                     │
                          ┌──────────┼──────────┐
                          ▼          ▼          ▼
                     Permissions  File system  Descriptor
                                     │
                                     ▼
                               Storage driver
                                     │
                                     ▼
                              Device and DMA
                                     │ interrupt
                                     ▼
                                Kernel wakes
                                editor thread
                                     │
                                     ▼
                              Document displayed
                                     │
                                     ▼
                             User edits and saves
                                     │
                                     ▼
                             Crash may be contained
```

---

# 16.41 Final Mental Model

An operating system is best understood as a **trusted coordinator** between applications and hardware.

It provides four broad functions:

## 1. Abstraction

It turns difficult hardware interfaces into manageable concepts:

* Processes
* Threads
* Virtual memory
* Files
* Sockets
* Windows
* Devices

## 2. Resource management

It distributes:

* CPU time
* RAM
* Storage
* Network capacity
* Device access

## 3. Protection

It enforces:

* User and kernel mode
* Process isolation
* Memory permissions
* File permissions
* Sandboxes
* Resource limits

## 4. Coordination

It connects concurrent activity through:

* Scheduling
* Interrupts
* Waiting and wake-up
* Synchronization
* Inter-process communication
* Device queues

```text
Operating system
├── Abstracts hardware
├── Manages resources
├── Enforces protection
└── Coordinates concurrent activity
```

---

# 16.42 Foundation Summary

| Area             | Central idea                                       |
| ---------------- | -------------------------------------------------- |
| Operating system | Trusted resource manager and abstraction layer     |
| Hardware         | Physical execution and storage resources           |
| Kernel           | Privileged core controlling protected operations   |
| User mode        | Restricted execution for applications              |
| Kernel mode      | Privileged execution for trusted kernel code       |
| Program          | Stored instructions and data                       |
| Process          | Isolated running resource environment              |
| Thread           | Schedulable execution sequence                     |
| Scheduler        | Chooses which ready thread runs                    |
| Context switch   | Saves one thread and restores another              |
| System call      | Intentional request to the kernel                  |
| Interrupt        | Asynchronous hardware notification                 |
| Exception        | Condition caused by current execution              |
| Physical memory  | Actual RAM                                         |
| Virtual memory   | Protected process-specific address abstraction     |
| Stack            | Per-thread nested execution state                  |
| Heap             | Dynamically managed process memory                 |
| Page fault       | Exception requiring memory-manager attention       |
| File             | Named persistent data and metadata                 |
| Directory        | Mapping from names to file-system objects          |
| File descriptor  | Process-local reference to an open resource        |
| File system      | Organization of persistent storage                 |
| Driver           | Software controlling a hardware type               |
| DMA              | Device-controller transfer between device and RAM  |
| Concurrency      | Overlapping progress                               |
| Synchronization  | Coordination of shared operations                  |
| Race condition   | Incorrect result caused by uncontrolled timing     |
| Deadlock         | Permanent waiting caused by dependency             |
| Security         | Enforcement of identity, authority, and protection |
| Virtual machine  | Virtual hardware with a separate guest kernel      |
| Container        | Isolated process group sharing the host kernel     |
| Boot process     | Staged construction of the running OS environment  |

---

# Learning Check

Do not look for answers yet.

## Conceptual questions

1. How do processes, threads, virtual memory, and CPU scheduling work together when an application runs?
2. What distinct roles do system calls, interrupts, and exceptions play in transferring control to the kernel?
3. How do files, file descriptors, file systems, drivers, and storage hardware connect during a file read?

## Cause-and-effect questions

4. Why can a thread waiting for storage stop using the CPU while its process and the rest of the system continue making progress?
5. Why can an invalid memory access terminate one application without normally crashing unrelated applications?

## Misconception question

6. A student says, “The operating system continuously executes every application itself and manually copies every memory access and device byte.” What is incorrect about this model?

## Scenario-based question

7. A containerized text editor running inside a virtual machine opens a file from virtual storage, allocates memory for it, displays it, and later crashes from an invalid memory access. Trace the major components involved from the container process through the guest kernel, hypervisor, host storage, physical hardware, scheduling, virtual memory, graphics, and eventual crash cleanup.

# Operating-System Revision Questions — Part 1

**Tags:**

* **[Moderate]** requires solid understanding of one or two concepts.
* **[Hard]** requires multi-step reasoning or cross-topic synthesis.
* **[Tricky]** targets subtle distinctions or common assumptions.
* **[Creative]** applies the notes to an unfamiliar situation.
* **[Inference]** requires a conclusion supported by the notes rather than stated directly.

## Conceptual Understanding

1. **[Moderate]** Why is describing an operating system merely as “software that runs applications” incomplete? Identify the additional responsibilities needed to explain its role accurately.

2. **[Hard] [Inference]** The scheduler is a mechanism for choosing threads, but “fairness” is a policy goal. Explain why separating mechanism from policy makes operating-system design more adaptable.

3. **[Tricky]** Why is it misleading to imagine the kernel as a separate program that runs continuously beside applications?

4. **[Hard]** A process is described as both a resource container and an isolation boundary. Explain why neither description alone is sufficient.

5. **[Moderate]** Why can two processes use the same numerical virtual address without sharing data, while two threads in one process usually interpret that address through the same mappings?

6. **[Tricky]** Stack and heap memory are often spoken of as if they were physically different kinds of memory. Explain why that mental model is incorrect and what actually distinguishes them.

7. **[Hard]** Why is a file descriptor a more useful reference for repeated operations than repeatedly resolving the original pathname?

8. **[Moderate]** Explain why a physical device, its controller, and its driver must be treated as three separate components when reasoning about I/O failures.

9. **[Tricky]** How can a system have concurrency without parallelism, and parallelism without guaranteeing useful progress?

10. **[Hard]** Why is the question “Which kernel processes this application’s system calls?” an effective way to distinguish containers from virtual machines?

11. **[Moderate]** Why must firmware, a bootloader, a kernel, and user-space services exist as separate startup stages instead of one component performing every boot task?

12. **[Hard] [Inference]** Across CPU scheduling, virtual memory, files, and devices, what recurring pattern does the OS use to turn limited physical resources into easier abstractions?

13. **[Tricky]** A process has one blocked thread. Under what circumstances would the whole application appear frozen, and under what circumstances could it remain responsive?

14. **[Moderate]** Why can a page fault be evidence that virtual memory is working correctly rather than evidence of a programming error?

15. **[Tricky]** Why does an administrator-level process still need system calls rather than executing privileged hardware operations directly?

16. **[Moderate]** Why does closing the final descriptor owned by one process not necessarily delete the corresponding file?

17. **[Hard]** Synchronization is often described as preventing simultaneous access. Why is ordering and visibility a more complete description?

18. **[Tricky]** A memory region can act as both a buffer and a cache. Explain how its purpose can satisfy both descriptions without contradiction.

19. **[Hard]** How can two processes share executable code pages while still preserving separate heaps, stacks, and private data?

20. **[Tricky]** Why is defining virtual memory as “RAM plus swap” insufficient even on a system that actively uses swap?

## Why and How

21. **[Hard]** Why does preemptive scheduling depend on a mechanism such as timer interrupts rather than trusting applications to yield voluntarily?

22. **[Hard]** Why must the kernel treat system-call arguments as untrusted even when the caller is a normal application rather than an attacker?

23. **[Moderate]** Why is MMU hardware needed to enforce memory protection efficiently instead of having kernel code inspect every memory access?

24. **[Hard]** How does demand loading reduce both startup cost and memory use, and what later cost does it introduce?

25. **[Hard]** Explain the full cause-and-effect chain by which an unavailable file block causes one thread to wait, another thread to run, and the original thread eventually to resume.

26. **[Moderate]** Why can an interrupt handler that performs too much work damage system responsiveness even if the work is legitimate?

27. **[Hard]** Why do applications normally use a user-space allocator instead of issuing a system call for every small heap allocation?

28. **[Tricky]** How does a guard page convert uncontrolled stack growth into a detectable failure rather than silent corruption?

29. **[Hard]** Why does evicting a dirty page generally require more work than evicting a clean file-backed page?

30. **[Moderate]** Why can the OS use large amounts of otherwise free RAM for file caching without permanently depriving applications of memory?

31. **[Hard]** DMA reduces CPU copying work, but why must the kernel still control buffer lifetime, permissions, transfer direction, and completion?

32. **[Hard]** Why must waiting on a condition variable release the associated mutex, and why must the mutex be reacquired before returning?

33. **[Hard]** Explain why a universal lock-acquisition order prevents circular waiting even when many threads and many locks are involved.

34. **[Moderate]** Why does least privilege reduce the consequences of a successful exploit without necessarily preventing the exploit itself?

35. **[Moderate]** Why can a container usually start more quickly than a virtual machine even when both eventually run the same application?

36. **[Hard]** Why does a VM need translation from guest virtual addresses to guest physical addresses and then to machine physical addresses?

37. **[Hard]** Why might the kernel need a temporary RAM-based early user-space environment before it can access its normal root file system?

38. **[Hard]** How does a chain of trust during boot differ from simply checking one kernel file immediately before execution?

39. **[Moderate]** Why can the kernel usually recover all private memory and descriptors after an application crash but not automatically reverse every external effect caused by that application?

40. **[Hard] [Inference]** Why should CPU, memory, process-count, descriptor, and storage limits be considered security mechanisms rather than only performance controls?

## Connections Between Topics

41. **[Hard]** Construct the chain linking a file-read system call, thread blocking, a context switch, DMA, a hardware interrupt, a ready-state transition, and system-call return.

42. **[Hard]** How can one file-backed page fault involve the MMU, exception handling, the file system, a driver, storage hardware, the scheduler, and an interrupt?

43. **[Tricky]** Why can switching between threads in different processes require more memory-management work than switching between threads in the same process?

44. **[Hard]** Explain how file-backed virtual-memory pages connect executable loading, shared libraries, the page cache, and recoverable page faults.

45. **[Hard]** In what way can an already-opened file descriptor reduce a pathname-based time-of-check to time-of-use risk, and what risks can still remain?

46. **[Hard]** How do interrupt handling, driver request queues, waiting threads, and scheduler wake-ups cooperate without requiring the device to understand processes?

47. **[Moderate]** Why does sharing one heap among threads make process-local memory faster to exchange but harder to use safely?

48. **[Tricky]** Explain how user/kernel mode, process credentials, and administrator authority interact during a privileged file operation.

49. **[Hard]** The boot process initializes memory, exceptions, interrupts, timers, scheduling, drivers, and file systems. Why would starting ordinary applications before any one of these foundations be unsafe or impossible?

50. **[Hard]** Trace a containerized application’s ordinary file read and identify which parts are container-specific and which are performed by the shared host kernel.

51. **[Hard]** Trace the same file read inside a VM and explain why both guest and host storage layers may participate.

52. **[Hard]** How can reclaiming clean file-cache pages reduce memory pressure without losing persistent information, and why might performance later decline?

53. **[Tricky]** How can a scheduling priority policy interact with a mutex to create priority inversion even when the mutex itself is functioning correctly?

54. **[Hard]** Why do separate virtual address spaces and separate file-descriptor tables protect different kinds of process resources?

55. **[Hard]** How can a write system call succeed, the file-system journal remain structurally consistent, and the user’s latest document changes still be lost after a power failure?

56. **[Hard]** Explain how memory-mapped file access replaces an explicit read path with ordinary memory access while still involving the file system and page-fault handling.

57. **[Creative]** A removable storage device is unplugged immediately after an application reports a successful save. Use buffering, cached writes, interrupts, file-system consistency, and safe removal to explain several possible outcomes.

58. **[Hard]** Why can restoring a VM snapshot produce a locally consistent guest state while leaving communication with external services logically inconsistent?

59. **[Moderate]** Compare the likely failure scope of an application crash, a guest-kernel crash, a host-kernel crash, and a hypervisor failure.

60. **[Hard] [Inference]** Why do drivers, hypervisors, system-call handlers, and privileged services all contribute disproportionately to attack surface?

## Application and Scenarios

61. **[Hard]** A text editor’s interface thread updates a document while its auto-save thread writes to storage. Design a safe sequence that preserves document consistency without holding a mutex during slow I/O.

62. **[Moderate]** Two ticket-booking threads both observe one remaining seat and both approve a booking. Identify the exact logical operation that must be atomic.

63. **[Hard]** Two bank-transfer threads move money in opposite directions between the same accounts. Explain how per-account locks can eliminate a race yet introduce deadlock, and how ordering removes the cycle.

64. **[Hard]** A browser thread waits for network data. Trace the packet from the physical adapter through DMA, interrupt handling, protocol processing, the thread’s state changes, and its eventual execution.

65. **[Creative]** A user presses a key, and a character appears in a graphical editor. Identify every major transition from physical input to visible output, including where interpretation rather than mere transfer occurs.

66. **[Tricky]** An application successfully reserves 500 MB of virtual memory but writes to only 20 MB. Explain why its virtual size, committed backing, and current physical RAM use may differ.

67. **[Hard]** A memory access causes a page fault. List the questions the kernel must answer before deciding whether to allocate a page, load data, copy a page, grow a stack, or terminate the process.

68. **[Hard]** A process crashes while a DMA-based device operation is still active. What must the kernel prevent, cancel, retain, or clean up to keep memory and device state safe?

69. **[Tricky]** Thread A records that descriptor `5` refers to a document. Thread B closes it, and the OS later reuses `5` for a network connection. Explain the resulting lifetime race.

70. **[Hard]** A bounded queue is full. A producer waits while holding the queue mutex, and a consumer needs that mutex to remove an item. Explain why the queue cannot recover and how condition-variable waiting avoids this.

71. **[Moderate]** An audio application reduces skipping by greatly enlarging its output buffer. Why might users then complain that controls feel delayed?

72. **[Hard]** A container has a strict memory limit but runs on a host with plenty of free RAM. Explain why allocation may still fail or processes may be terminated.

73. **[Hard]** A guest thread is ready according to its guest scheduler but receives no physical CPU time. Explain how this can happen through two scheduling levels.

74. **[Hard]** A machine’s system partition is encrypted. Trace startup until the real root file system becomes available, identifying which components must be accessible before decryption.

75. **[Tricky]** A file is deleted from its directory while a process continues reading it through an open descriptor. Explain why the process can continue and when the storage may finally be reclaimed.

76. **[Hard]** An editor must save an important document safely. Justify a temporary-file-and-replacement strategy using buffering, partial writes, metadata updates, and crash risk.

77. **[Creative]** A computer reaches a black screen after boot. Construct three explanations in which the kernel is running successfully but different user-space or graphics components failed.

78. **[Moderate]** Firmware reports “no bootable device.” Which later OS components can be ruled out as the immediate source of failure, and why?

79. **[Tricky]** All database connections are leaked by one process, causing others to wait forever. Why might this resemble deadlock while failing the formal circular-wait test?

80. **[Hard]** A network adapter generates interrupts so frequently that performance collapses. Explain why switching to a mixed interrupt-and-polling approach can improve throughput.

## Comparison and Trade-offs

81. **[Hard]** Compare coarse-grained and fine-grained locking in terms of reasoning complexity, parallelism, contention, and deadlock risk. Under what workload could the simpler design outperform the more parallel one?

82. **[Moderate]** Compare blocking, non-blocking, and asynchronous I/O by describing what happens to the calling thread when data is not ready.

83. **[Hard]** Compare polling and interrupt-driven I/O for a rarely active keyboard and a heavily loaded network adapter. Why might opposite choices be appropriate?

84. **[Moderate]** Compare a mutex and a semaphore using ownership, capacity, release rules, and likely use cases.

85. **[Hard]** Under what access pattern might a read–write lock outperform a mutex, and under what pattern might its added complexity make it worse?

86. **[Hard]** Compare VMs and containers in terms of kernel ownership, startup, compatibility, memory overhead, failure boundaries, and system-call paths.

87. **[Tricky]** Compare using RAM and storage-backed swap for active memory. Why can swap increase flexibility without meaningfully increasing computation speed?

88. **[Moderate]** Compare buffering and caching by identifying the problem each primarily solves and one situation where the same pages serve both purposes.

89. **[Moderate]** Compare processes and threads by explaining which resources are normally private, which are shared, and which unit the scheduler actually runs.

90. **[Tricky]** Compare a program file with a running process. Why can several processes originate from one program while having different data, descriptors, and credentials?

91. **[Moderate]** Compare user mode and kernel mode in terms of authority, code identity, and who decides whether a privileged operation occurs.

92. **[Tricky]** Compare an ordinary function call with a system call. Why can a library function involve zero, one, or several system calls?

93. **[Moderate]** Compare a mode switch and a context switch using an interrupt that returns to the same thread and an interrupt that causes another thread to run.

94. **[Tricky]** Compare a TLB miss and a page fault. Why can one be resolved without kernel loading or allocating any page?

95. **[Moderate]** Compare closing a file, deleting a directory entry, and reclaiming the file’s storage. Why can these occur at different times?

96. **[Hard]** Compare a VM snapshot and a complete backup in terms of dependency on existing storage, inclusion of external state, and recovery purpose.

97. **[Moderate]** Compare virtualization and emulation. Why can a VM use virtualization for CPU execution while still emulating selected devices?

98. **[Creative] [Inference]** Least privilege can reduce functionality and increase permission prompts. How should a system balance minimal authority with usability without abandoning the security principle?

99. **[Hard]** Compare fairness and throughput in CPU scheduling, lock acquisition, and device queues. Why can maximizing one reduce the other?

100. **[Tricky]** Compare small and large I/O buffers across latency, throughput, memory use, underrun risk, and overrun tolerance.

# Operating-System Revision Questions — Part 2

## Errors and Misconceptions

101. **[Tricky]** A student argues that unused RAM should always remain empty so applications can use it later. Explain what this overlooks about file caching and reclaimable memory.

102. **[Hard]** A thread changes from `WAITING` to `READY`. Why does this not imply that its system call returns immediately?

103. **[Tricky]** Why is it incorrect to assume that every hardware interrupt causes the currently running thread to be replaced?

104. **[Moderate]** A process has several threads, but only one thread is running on a one-core system. Why are the other threads still part of active concurrent execution?

105. **[Tricky]** A student says that the kernel translates every virtual address by running software before each memory instruction. Identify the hardware and caching mechanisms this model ignores.

106. **[Hard]** Why can an application corrupt its own heap without violating process isolation?

107. **[Tricky]** A process reports a large virtual-memory size and a much smaller resident-memory size. Why is neither measurement necessarily evidence of a leak?

108. **[Hard]** A student concludes that a page marked “not present” must be invalid. Explain several legitimate states that contradict this conclusion.

109. **[Moderate]** Why is an executable file not a process even while one or more processes are executing instructions originating from it?

110. **[Tricky]** Why does a file’s extension provide weaker evidence of its meaning than the application-level structure of its contents?

111. **[Hard]** A write request returns fewer bytes than requested. Why is retrying the whole original write from the beginning potentially incorrect?

112. **[Tricky]** A user believes that moving a file always reads and rewrites all its contents. Under what file-system circumstances can that assumption be false?

113. **[Moderate]** Why can two directory entries refer to one underlying file without creating two independent copies of its data?

114. **[Hard]** A student claims that journaling makes application transactions unnecessary. Distinguish file-system structural recovery from application-level consistency.

115. **[Tricky]** Why does an interrupt signal not necessarily identify which user process should receive the resulting data?

116. **[Moderate]** Why can a device operation continue after the thread that initiated it stops running?

117. **[Tricky]** A developer assumes that closing an application window guarantees that all device operations have physically finished. Why may this be false?

118. **[Hard]** Why does waking every thread waiting on a condition variable not mean every awakened thread can successfully consume the resource?

119. **[Tricky]** A mutex-protected variable is read without the mutex because “reading cannot damage it.” Explain how this can still violate correctness.

120. **[Hard]** A program contains no data races under its language’s rules. Why might it still contain a higher-level race condition?

121. **[Moderate]** Why can deadlocked threads consume almost no CPU while making the application completely unresponsive?

122. **[Tricky]** A program uses timeouts on every lock. Why does this not prove the program is free from deadlock-related design defects?

123. **[Hard]** Why can repeatedly releasing locks and retrying eliminate deadlock yet create livelock?

124. **[Tricky]** A container’s process list shows only a few processes. Why does that not mean the host kernel is unaware of all the underlying processes?

125. **[Hard]** A VM has one virtual CPU, while the physical host has many cores. Why can guest applications still be concurrent but not execute in guest-level parallelism?

126. **[Tricky]** Why does restoring a hibernated system differ fundamentally from loading a fresh executable image, even though both read substantial data from storage?

127. **[Moderate]** A desktop appears on screen. Why does this not prove that every system service and device has completed initialization?

128. **[Hard]** A successful Secure Boot chain is observed. What security questions remain unanswered about runtime permissions, kernel defects, application safety, and data confidentiality?

129. **[Tricky]** Why is a process running as a highly privileged user still constrained by its active virtual address mappings?

130. **[Hard]** A sandboxed application is prevented from reading unrelated files but is allowed to overwrite one user-selected document. Why can compromise still cause meaningful damage without escaping the sandbox?

## Edge Cases and Exceptions

131. **[Hard]** A process opens a file for reading, another process replaces the pathname with a different file, and the first process continues reading. Which object should its existing descriptor refer to, and why?

132. **[Tricky]** Two descriptors in the same process refer to the same underlying file but maintain different offsets. What kernel-level arrangement could produce this behavior?

133. **[Hard]** Two descriptors share one current file offset. What kinds of duplicated or inherited open-file state could explain this, and what synchronization issue can result?

134. **[Creative]** A file has several names through direct links. One name is deleted while another remains and a process also has the file open. Reason about the independent roles of names, open references, and storage reclamation.

135. **[Hard]** A process maps a file into memory, another process modifies the same file, and the first process reads its existing mapping. What factors determine whether and when it observes the change?

136. **[Tricky] [Inference]** Why can mapping a file into memory simplify random access while making error timing less obvious than an explicit read system call?

137. **[Hard]** A process requests executable permission on a writable memory region. Which security principle does this weaken, and why might the kernel or policy reject the request?

138. **[Creative]** A program’s stack grows normally through several recoverable page faults and then suddenly crashes on the next page. Construct a valid explanation involving reserved stack range, guard pages, and maximum stack size.

139. **[Hard]** A valid anonymous page has been evicted to storage. Trace what changes in the page table, thread state, scheduler, storage system, and eventual instruction retry when it is accessed again.

140. **[Tricky]** A page fault occurs, but no physical storage I/O and no context switch follow. Give several supported explanations.

141. **[Hard]** A TLB contains a stale writable translation after the kernel changes a page to read-only. Why must the kernel invalidate or update translation state before relying on the new permission?

142. **[Creative]** Two processes intentionally share one physical page, but one maps it read-only and the other read–write. Predict which operations each can perform and how the MMU enforces the difference.

143. **[Hard]** A copy-on-write page is shared by three processes. One process writes to it. Explain what should remain shared and what becomes private.

144. **[Tricky]** A process releases a heap allocation, but the corresponding virtual pages remain mapped and resident. Why might later access through an outdated reference corrupt data rather than fault immediately?

145. **[Hard]** Physical RAM is plentiful, but a process cannot reserve another virtual region. What address-space or mapping limitations could explain this?

146. **[Creative]** A 32-bit process runs on a machine with far more physical RAM than its address space can represent. Explain why abundant machine memory does not remove the process’s virtual-address limitation.

147. **[Hard]** A computer has no configured swap area but still uses virtual memory extensively. Identify the virtual-memory features that remain fully relevant.

148. **[Tricky]** A clean file-backed page is discarded from RAM and later reloaded. Why is this not equivalent to losing unsaved application data?

149. **[Hard]** A dirty memory-mapped file page is evicted. What additional work may be required compared with evicting an unchanged executable-code page?

150. **[Creative]** A process is terminated under severe memory pressure even though some RAM is occupied by file cache. Construct reasons the kernel might not be able to solve the situation merely by discarding all cache pages.

151. **[Hard]** A keyboard interrupt arrives while the CPU is already executing kernel code. What possibilities exist for immediate handling, delayed handling, priority decisions, or masking?

152. **[Tricky]** A device completion interrupt arrives after an I/O request was canceled. Why must the driver still interpret the completion rather than simply assume it cannot happen?

153. **[Hard]** A DMA controller is configured to write into process pages that are later unmapped. What protection or lifetime rule must prevent memory corruption?

154. **[Creative]** A network receive buffer fills because an application is paused by a debugger. Explain possible consequences involving buffering, backpressure, packet drops, and sender behavior.

155. **[Hard]** An audio application has enough CPU capacity on average but still experiences underruns. How can scheduling delay and burst timing explain the failure?

156. **[Tricky]** A printer job remains queued after the application that submitted it exits. Why is this compatible with process cleanup?

157. **[Hard]** A removable device has no visibly open files, yet the OS refuses immediate removal. Identify other forms of active state or pending work that may justify the refusal.

158. **[Creative]** A driver times out, resets a device, and later receives completion from the original operation. What bookkeeping problem must it solve to avoid confusing old and new requests?

159. **[Hard]** A low-priority thread owns a mutex needed by a high-priority audio thread. Explain how medium-priority CPU-heavy threads can indirectly cause an audio underrun.

160. **[Tricky]** A thread waiting for a semaphore wakes, but the protected resource is still unavailable when it runs. How can scheduling and competition explain this without implying that the semaphore is broken?

## Critical Reasoning

161. **[Hard] [Inference]** Why is “minimize shared mutable state” a broader strategy than “use locks correctly”?

162. **[Hard]** A synchronization design uses one global mutex and is logically correct. What evidence would you examine before deciding whether to replace it with several locks?

163. **[Creative]** Design a reasoning checklist for deciding whether work should occur inside or outside a critical section.

164. **[Hard]** Why must a lock protect an invariant rather than merely a collection of individual variables?

165. **[Tricky]** If a critical section temporarily violates an invariant but restores it before unlocking, why can that still be unsafe if it calls external code while the invariant is broken?

166. **[Hard]** A condition variable notification is sent before a waiting thread begins waiting. Under what synchronization design could the notification effectively be lost, and how should shared state prevent this?

167. **[Creative]** A producer–consumer queue has no fixed maximum size. Which synchronization requirements disappear, and which remain?

168. **[Hard]** Why can processing a removed queue item outside the lock improve concurrency without allowing another consumer to process the same item?

169. **[Tricky]** A lock is uncontended most of the time. Why might its memory-ordering and cache-coherence costs still matter on multiple cores?

170. **[Hard]** Explain why lock-free does not mean race-free, wait-free, simpler, or automatically faster.

171. **[Creative] [Inference]** Under what workload could one dedicated owner thread plus message passing outperform many threads sharing a finely locked data structure?

172. **[Hard]** Why can strict first-come-first-served lock fairness reduce throughput on a multicore machine?

173. **[Tricky]** A reader–writer lock allows many readers. How can a continuous stream of readers starve a writer even though no reader holds the lock forever?

174. **[Hard]** How can writer preference solve writer starvation while creating a new starvation risk for readers?

175. **[Creative]** A service detects a three-thread deadlock. Develop criteria for choosing which operation to cancel while minimizing lost work and preserving consistency.

176. **[Hard]** Why is forcibly transferring an ordinary mutex from one thread to another generally unsafe?

177. **[Tricky]** A thread is waiting for a resource whose owner has crashed. Why might the OS be able to clean up some resource types automatically but not restore an arbitrary application invariant?

178. **[Hard]** A timeout expires during a slow but valid storage operation. What information is needed to distinguish overload, device failure, deadlock, and merely conservative timeout selection?

179. **[Creative]** Explain how diagnostic logging can both help investigate and accidentally conceal a race condition.

180. **[Hard] [Inference]** Why is demonstrating that “every shared access holds some lock” insufficient to prove concurrency correctness?

181. **[Hard]** A privileged service receives an already-open file descriptor from an unprivileged process. What security questions should the service ask before using its authority on that object?

182. **[Tricky]** Why can validating a pathname and then opening it be less secure than opening an object and validating the opened reference?

183. **[Hard]** How can file permissions, sandbox rules, mandatory labels, and an open capability all participate in one authorization decision without being redundant?

184. **[Creative]** A photo editor has permission to read selected images but no general directory access. Explain how capability-style access can support this policy.

185. **[Hard]** Why should a privileged helper expose a narrow operation such as “print this approved document” rather than a broad operation such as “perform any file action”?

186. **[Tricky]** A process loses permission to a pathname after opening the file. What policy choices could govern whether its existing descriptor remains usable?

187. **[Hard]** Why is revoking already-delegated capabilities more difficult than denying future pathname lookups?

188. **[Creative] [Inference]** A system encrypts storage but keeps decryption keys available to a logged-in compromised process. Which confidentiality threat is solved, and which remains?

189. **[Hard]** Why can a validly signed but outdated driver remain a serious security risk?

190. **[Tricky]** How can reducing attack surface conflict with compatibility or functionality, and what design principle helps manage the trade-off?

## Mixed-Topic Synthesis

191. **[Hard]** A process opens a file, maps it into memory, starts a worker thread, and then closes the original descriptor. Explain why the mapping may remain usable and what references keep the underlying object alive.

192. **[Creative]** A worker thread faults on a file-backed mapped page while holding a mutex. Trace how required storage I/O can indirectly block unrelated threads needing the same mutex.

193. **[Hard]** Why can holding an application lock during a major page fault be nearly as harmful as deliberately performing blocking file I/O while holding it?

194. **[Tricky] [Inference]** An application has no explicit file reads inside a critical section, yet storage latency occurs there. How could demand paging or mapped files explain this?

195. **[Hard]** A context switch activates a different process’s page tables, but both processes share a read-only library page. Explain what changes and what remains physically shared.

196. **[Creative]** A scheduler moves a thread to another CPU core immediately after it acquires shared data. Identify cache, memory-visibility, and synchronization considerations without assuming the data itself moves between virtual addresses.

197. **[Hard]** An interrupt wakes a high-priority thread whose required code page is absent from RAM. Explain why waking the thread does not guarantee immediate useful execution.

198. **[Tricky]** A thread becomes ready after network input arrives but blocks again almost immediately on a mutex. Why can the system show frequent context switches with little useful progress?

199. **[Hard]** A process’s working set fits in RAM when running alone but thrashes when several similar processes run together. Explain the system-wide cause without blaming any single process’s access pattern.

200. **[Creative] [Inference]** Why can reducing the number of runnable threads improve both CPU-cache behavior and memory pressure?

201. **[Hard]** A container’s process is denied access to a host file even though its visible user identifier appears to own a similarly named file inside the container. Explain the possible roles of user mapping, mount views, host ownership, labels, and sandbox policy.

202. **[Tricky]** A container sees its process as PID 1, while the host sees PID 9000. Which identifier does the host scheduler use, and why can both views be valid?

203. **[Hard]** A container process makes a page-faulting access to a file-backed page. Identify which operations use container-specific views and which are ordinary host-kernel memory and storage operations.

204. **[Creative]** A container is given a writable mount of a sensitive host directory but has otherwise strict system-call filtering. Explain why the mount may dominate the security outcome.

205. **[Hard]** A VM’s guest OS reports low CPU utilization, but users experience slow response. How could hypervisor scheduling explain the discrepancy?

206. **[Tricky]** A VM’s guest reports free memory while the host is under severe memory pressure. Why can both observations be accurate?

207. **[Hard]** A guest application causes a page fault. Distinguish the guest kernel’s role from the hypervisor’s role in translating and backing the eventual memory access.

208. **[Creative]** A guest file read is satisfied from the guest page cache. Which host and physical-device operations can be avoided, and which virtualization layers still remain relevant?

209. **[Hard]** A guest cache miss is satisfied by the host’s cache without physical storage access. Explain how two caching layers can both miss and hit differently for the same logical read.

210. **[Tricky]** Why can assigning more virtual CPUs to a VM reduce performance under some host workloads rather than improve it?

211. **[Hard]** A VM snapshot is taken while several guest threads hold locks and one device request is in progress. What state must be captured for restoration to resume coherently?

212. **[Creative]** A restored VM retransmits a network request that an external server already completed before the snapshot. Explain why local machine-state restoration cannot guarantee external exactly-once behavior.

213. **[Hard]** A host runs containers inside VMs. Identify the separate scheduling, memory, file-system, and security boundaries crossed by one container system call.

214. **[Tricky]** Why may a container inside a VM be restricted by both container policy and guest-OS policy even though the host hypervisor does not understand the application-level request?

215. **[Hard]** A guest kernel is compromised, but the attacker cannot access another VM. Which hardware and hypervisor-controlled mechanisms must continue functioning correctly?

216. **[Creative] [Inference]** Under what circumstances could a small dedicated VM be chosen to run one container even though this adds startup and memory overhead?

217. **[Hard]** During boot, a storage driver is needed to load the file containing an additional driver. Explain how built-in drivers or early user-space components break this dependency.

218. **[Tricky]** The kernel has initialized storage but cannot start a login service. Why does this indicate a later boot-stage failure rather than a firmware-stage failure?

219. **[Hard]** A service-manager dependency cycle prevents the graphical login from starting. Compare this with a mutex deadlock and identify both similarities and differences.

220. **[Creative]** A system resumes from hibernation onto hardware whose device configuration changed while powered off. What reconstruction or error-handling challenges may arise?

221. **[Hard]** Why must the kernel establish exception handlers before enabling ordinary user-mode execution, even if all initial applications are trusted?

222. **[Tricky]** Why must timer and scheduler initialization precede running an application that accidentally enters an infinite loop?

223. **[Hard]** A system can boot from network storage. Which ordinary boot assumptions about local disks must be replaced by early network drivers, configuration, and authentication?

224. **[Creative]** Secure Boot approves the bootloader and kernel, but encrypted root storage cannot be unlocked. Identify which security mechanism succeeded, which failed, and why the machine still cannot complete startup.

225. **[Hard]** The kernel mounts the root file system read-only after detecting errors. How can this preserve integrity while reducing availability and functionality?

226. **[Tricky]** A system boots successfully but has the wrong current time. Why might authentication, certificate validation, logs, and network services behave incorrectly even though scheduling timers still function?

227. **[Hard]** Why is shutdown not merely “stop executing instructions,” and which buffered, persistent, process, and device states require coordination?

228. **[Creative]** A forced power loss occurs while the OS is writing a file-system journal record but before updating the main metadata. Explain the kind of recovery journaling is designed to support.

229. **[Hard]** A forced power loss occurs after metadata is consistent but before an application’s latest data reaches storage. Why can the file system recover while the application still loses work?

230. **[Tricky]** A process receives a fatal exception while executing kernel code on its behalf. Why must the kernel distinguish a recoverable bad user pointer from corruption of its own internal state?

# Operating-System Revision Questions — Part 3

## Unusual and Unpredictable Scenarios

231. **[Creative]** A process has no runnable threads but still owns open files, allocated memory, and network connections. Explain why the process still exists and what events could make one of its threads runnable again.

232. **[Tricky]** A process is using almost no CPU but causes heavy storage activity. Construct explanations involving asynchronous writes, page eviction, file caching, and device queues.

233. **[Hard]** An application becomes faster after another memory-intensive application exits, even though the first application’s code and workload do not change. Explain the likely interactions among working sets, page faults, cache availability, and scheduling.

234. **[Creative]** A user launches the same program twice. One instance opens correctly, while the other receives a permission error. Give several OS-level reasons the two processes could receive different results.

235. **[Hard]** A program runs correctly when started from one directory but fails to locate a file when started from another. Explain how relative paths, current working directories, and process startup state produce this result.

236. **[Tricky]** A process inherits a current working directory that it cannot list but can still use to reach a known file. What distinction among listing, traversal, and file access could explain this?

237. **[Creative]** A file has read permission but cannot be reached through its pathname. Construct an explanation involving inaccessible parent directories.

238. **[Hard]** A process can read an existing file in a directory but cannot create a new file beside it. Explain why file permissions and directory permissions can produce this combination.

239. **[Tricky]** A process can modify a file’s contents but cannot rename it. Which object’s permissions are most relevant to each operation, and why?

240. **[Creative]** A process can rename a file it cannot read. Explain how directory-entry authority and file-content authority can be separated.

241. **[Hard]** A user deletes a large open file, but reported free storage does not increase until a service restarts. Explain the reference-lifetime chain that could cause this.

242. **[Tricky]** A file appears under two names. An application modifies it through one name, and the changes appear through the other. Explain why this is expected for one kind of link but not necessarily for another.

243. **[Creative]** A symbolic link points to a relative path. Explain why moving the link itself or changing the surrounding directory structure could change which object it reaches.

244. **[Hard]** A file-system move completes almost instantly within one volume but becomes slow across volumes. Trace the different metadata and data-transfer work likely involved.

245. **[Tricky]** A process opens a file, changes its current working directory, and continues using the existing descriptor. Why does the directory change not redirect that descriptor?

246. **[Creative]** A process changes its current working directory while another thread resolves a relative path. What application-level coordination issue could arise even though the kernel resolves each path correctly?

247. **[Hard]** A file read returns fewer bytes than requested even though the read has not reached the file’s end. Identify supported reasons and explain how robust application logic should interpret the result.

248. **[Tricky]** A process reaches end of file and later successfully reads more data from the same open file. Construct a valid explanation involving another process extending the file.

249. **[Creative]** Two processes share one open-file offset through inherited state. Explain how concurrent sequential reads could divide a file between them in an unpredictable but non-duplicating pattern.

250. **[Hard]** Two processes independently open the same file and both begin reading at offset zero. Why are their offsets normally independent even though the underlying file is shared?

251. **[Tricky]** A successful buffered write is followed by an error reported during close or a later synchronization request. Explain why the error may appear after the original write call.

252. **[Creative]** A process crashes after writing a temporary replacement file but before renaming it over the original. What states might remain, and why can the original file still be valid?

253. **[Hard]** A process crashes immediately after replacing an old file with a new temporary file but before related directory metadata is durably stored. Explain why persistence guarantees must include more than the new file’s contents.

254. **[Tricky]** A file-system cache contains newer data than the physical storage device. Which version should ordinary reads return, and why?

255. **[Creative]** Two processes map the same file and update different pages. Explain how page-level dirty tracking can preserve both changes while still leaving application-level consistency concerns.

256. **[Hard]** A process maps a file read-only but also opens it through a writable descriptor. Why can a write system call succeed while a direct store through the mapping fails?

257. **[Tricky]** A process modifies a memory-mapped file page but closes the original file descriptor first. Why may the modified mapping still need to be written back later?

258. **[Creative]** A process has enough permission to open a device but receives an error because another process has exclusive control. Distinguish authorization failure from resource-availability failure.

259. **[Hard]** A camera works in one application but is unavailable in another running as the same user. Explain the roles of application-specific permissions, exclusive ownership, and sandbox policy.

260. **[Tricky]** A device driver reports completion, but the application receives fewer bytes than requested. Why can hardware completion and full application-request completion be different concepts?

## Diagnostic Reasoning

261. **[Hard]** An application is unresponsive, uses almost no CPU, and shows no storage or network activity. Develop a diagnostic distinction among deadlock, legitimate waiting, starvation, and waiting for user input.

262. **[Creative]** An application uses 100% of one CPU core but makes no visible progress. Construct explanations involving infinite computation, spinlock waiting, livelock, polling, and repeated fault handling.

263. **[Hard]** A multithreaded process performs worse after moving from two CPU cores to sixteen. Identify possible roles for contention, cache coordination, context switching, memory bandwidth, and lock granularity.

264. **[Tricky]** A process shows many voluntary context switches. What kind of behavior does this suggest compared with many involuntary context switches?

265. **[Hard] [Inference]** A workload shows low CPU utilization, high major page-fault counts, and heavy storage traffic. What system bottleneck is most likely, and why might adding CPU cores have little effect?

266. **[Creative]** A system shows high CPU utilization, very few page faults, and low I/O activity, yet response time is poor. Construct scheduler- or contention-based explanations.

267. **[Hard]** A process’s resident memory rises steadily, but its virtual size remains almost constant. What forms of demand paging, page touching, or cache use could produce this?

268. **[Tricky]** A process’s virtual size grows steadily while resident memory remains mostly flat. Why might this represent reservation, sparse access, mapped files, or a leak that has not yet caused physical pressure?

269. **[Hard]** A system becomes slow immediately after launching many identical processes. Explain how executable sharing may reduce some memory costs while private heaps, stacks, and working sets still create pressure.

270. **[Creative]** A process starts quickly on its second launch after being closed. Explain how file cache and shared-library pages can improve startup even though the previous process no longer exists.

271. **[Tricky]** A file read is faster after another unrelated process reads the same file. What shared kernel resource likely explains the improvement?

272. **[Hard]** A process repeatedly faults on pages that were recently used. What evidence would distinguish ordinary demand paging from thrashing?

273. **[Creative]** A machine has free memory according to one display but still reports allocation failures for a process. Construct explanations involving limits, address space, commitment policy, fragmentation, and container quotas.

274. **[Hard]** A service fails only under heavy concurrency and succeeds when diagnostic logging is enabled. Why should a race condition be suspected, and what other timing-sensitive failures remain possible?

275. **[Tricky]** A deadlock report shows no cycle among mutexes. What other resource or communication dependencies should be examined?

276. **[Hard]** A thread waits on a mutex whose recorded owner is sleeping during network I/O. Why is the network wait relevant to diagnosing lock contention even without a formal deadlock?

277. **[Creative]** An audio application skips only when a storage-heavy task runs. Explain possible interactions among I/O queues, interrupt load, scheduler latency, memory pressure, and buffer size.

278. **[Hard]** A network server accepts connections but gradually loses the ability to open files and sockets. What shared resource leak could explain both symptoms?

279. **[Tricky]** A process has closed every visible application file but still cannot open more descriptors. What non-file resources might occupy descriptor-table entries?

280. **[Hard]** A system service restarts repeatedly after startup. Explain how process isolation allows the kernel to survive while service dependencies can still make the whole system appear unusable.

281. **[Creative]** A graphical desktop freezes, but remote login remains available. What does this suggest about the health of the kernel, scheduler, networking, and graphics-related user-space services?

282. **[Hard]** A machine responds to network pings but cannot complete user logins. Construct a boot- or service-stage explanation that preserves networking while authentication or session creation fails.

283. **[Tricky]** A kernel log reports a device timeout, but the device later responds. Why is a timeout a policy judgment rather than proof that the hardware permanently failed?

284. **[Hard]** A storage device reports success to the driver, yet application data is corrupted. Identify layers where corruption could still have occurred.

285. **[Creative]** A process receives permission denied only after a previously successful period of access. Consider credential changes, policy changes, resource replacement, sandbox revocation, and remounting.

286. **[Hard]** A containerized service works until the host reboots, after which its data is gone. Which misunderstanding about image layers, writable state, and persistent volumes is most likely?

287. **[Tricky]** A container’s application can reach the network but cannot resolve hostnames. Which networking components may work, and which service or configuration layer may be missing?

288. **[Hard]** A VM reports that its virtual disk is healthy, while the host reports physical-storage errors. Why can the guest’s view remain temporarily optimistic?

289. **[Creative]** One VM experiences severe latency while neighboring VMs appear normal. Construct explanations involving host scheduling, storage placement, resource limits, and guest-internal contention.

290. **[Hard]** A VM’s clock jumps after pause and resume. What application behaviors involving timeouts, logs, authentication, and scheduling might be affected?

## Architecture and Design Decisions

291. **[Hard]** You are designing a service that handles untrusted documents. Justify separating parsing, privileged file access, and user-interface work into processes with different authority.

292. **[Creative]** A service needs to read exactly one user-selected file. Compare granting directory-wide permission with passing one already-open descriptor.

293. **[Hard]** A program needs high-throughput communication between two trusted local processes. Compare shared memory with message passing in terms of copying, synchronization, isolation, and failure handling.

294. **[Tricky]** Why might message passing still require synchronization inside the sender or receiver even if the communication channel itself is thread-safe?

295. **[Hard]** A server processes independent requests. Compare one thread per request, a fixed worker pool, and an asynchronous event loop using only concepts supported by the notes.

296. **[Creative] [Inference]** Under what workload could a fixed worker pool protect availability better than creating an unbounded number of threads?

297. **[Hard]** A workload performs many tiny heap allocations. What user-space allocator strategies can reduce system-call frequency, and what costs can retained arenas introduce?

298. **[Tricky]** Why might returning every freed heap page immediately to the kernel reduce memory use but hurt performance?

299. **[Hard]** A system designer must choose between small and large pages for a memory-intensive workload. Analyze translation overhead, TLB reach, internal waste, and movement cost.

300. **[Creative]** A workload sequentially scans a huge file once. How might file caching help or harm other applications, and what reclaim behavior should the OS prefer?

301. **[Hard]** A database needs durable multi-record updates. Why are file locks and successful writes insufficient without application-level transaction and recovery logic?

302. **[Tricky]** Why can placing a mutex around every database file operation still fail to protect against power loss?

303. **[Hard]** A service must update one file while readers continue seeing either the old or new complete version. Justify a replacement-based strategy rather than in-place modification.

304. **[Creative]** A logging system produces data faster than storage can persist it. Compare dropping logs, blocking producers, enlarging buffers, and applying backpressure.

305. **[Hard]** A latency-sensitive audio service and a throughput-oriented backup service share one system. Explain how scheduling and I/O policies might reasonably treat them differently.

306. **[Tricky]** Why can giving the audio service absolute priority over all other work damage long-term availability or fairness?

307. **[Hard]** A driver can use either frequent interrupts or periodic polling. Identify workload characteristics that should influence the choice.

308. **[Creative]** A device produces data in large bursts separated by long idle periods. Propose a combined interrupt, polling, and buffering strategy.

309. **[Hard]** A security-sensitive workload can run either in a container or a VM. Identify factors that favor the stronger separate-kernel boundary despite added overhead.

310. **[Tricky]** Why might placing mutually untrusted containers inside separate VMs reduce shared-kernel risk without eliminating all shared-resource interference?

311. **[Hard]** A company wants to run many identical stateless services. Explain why container images, shared layers, resource controls, and orchestration fit this workload.

312. **[Creative]** A stateful database is deployed in a replaceable container. Explain which state belongs in the image, container layer, persistent volume, and external backup.

313. **[Hard]** A VM requires high storage performance. Compare emulated storage, paravirtualized storage, and direct device assignment in terms of compatibility, overhead, sharing, and isolation.

314. **[Tricky]** Why can direct hardware assignment improve performance while making migration and resource sharing harder?

315. **[Hard]** A cloud provider overcommits CPU and memory. Explain why this can improve utilization under typical loads and become dangerous when workloads peak together.

316. **[Creative] [Inference]** What monitoring information would help a provider distinguish harmless overcommitment from emerging thrashing or noisy-neighbor failure?

317. **[Hard]** A boot process can start independent services in parallel. What dependency and synchronization information is needed to do this safely?

318. **[Tricky]** Why can delaying a nonessential service improve boot time but create confusing failures if another component silently assumes it is ready?

319. **[Hard]** A system must recover automatically after a failed update. Explain how multiple boot entries, previous kernels, verified components, and recovery environments can cooperate.

320. **[Creative]** Design a staged failure policy for boot: what should happen after firmware failure, bootloader failure, root-file-system failure, and one nonessential service failure?

## Final Whole-System Synthesis

321. **[Hard]** Beginning with a user clicking an application icon, trace every major boundary crossed until the program executes its first user-mode instruction.

322. **[Creative]** Beginning with a process writing one byte to a previously untouched heap page, explain how that action can involve an allocator, system call history, page tables, the MMU, an exception, physical-frame allocation, security clearing, and instruction retry.

323. **[Hard]** Beginning with a thread reading one byte from an uncached file, trace the operation through virtual memory, file descriptors, file-system mapping, device queues, DMA, interrupts, scheduling, and return to user mode.

324. **[Tricky]** In the previous scenario, identify every point at which the operation could fail even though the original pathname and descriptor were valid.

325. **[Hard]** Explain how one timer interrupt can involve a mode switch without a context switch, or both a mode switch and a context switch, depending on scheduler policy.

326. **[Creative]** A process causes a valid page fault while another process’s storage request completes at nearly the same time. Explain how the kernel may handle both events and how scheduling determines which thread next runs.

327. **[Hard]** A multi-threaded editor crashes because one thread corrupts shared heap data used by another. Explain why thread stacks, process isolation, heap sharing, synchronization failure, and eventual exception handling all matter.

328. **[Tricky]** Why can the kernel cleanly reclaim the editor’s virtual memory after the crash even when it cannot determine which application-level object was corrupted first?

329. **[Hard]** A compromised application attempts to read another process’s password, access the camera, overwrite a system file, consume all memory, and send network data. Match each attempt to the distinct OS mechanisms that may stop or limit it.

330. **[Creative]** A malicious process cannot directly access another process’s memory, so it sends deceptive requests to a privileged service instead. Explain why memory isolation succeeds while a confused-deputy vulnerability may still expose protected data.

331. **[Hard]** A containerized application inside a VM receives network input, allocates memory, writes a persistent file, and returns a response. Trace the guest, container, hypervisor, host, and hardware responsibilities.

332. **[Tricky]** In that scenario, identify which scheduler runs the container thread, which scheduler runs the VM’s virtual CPU, and why both can delay the request.

333. **[Hard]** A host-kernel fault crashes all containers but leaves another physical machine unaffected. Explain the relevant process, container, kernel, and hardware failure boundaries.

334. **[Creative]** A guest-kernel fault crashes one VM while containers in another VM continue. Explain why separate guest kernels and hypervisor isolation produce this result.

335. **[Hard]** A system is slow because of memory pressure, but users describe it as “the CPU being slow.” Build a causal explanation connecting page faults, storage waits, ready and waiting states, context switches, and apparent responsiveness.

336. **[Tricky]** Why can CPU utilization be low during severe thrashing even though the system is extremely busy moving data?

337. **[Hard]** A process writes data, receives success, crashes, and restarts to find the file partly updated. Identify which guarantees belonged to process cleanup, file-system consistency, buffering, and application-level atomicity.

338. **[Creative]** A process crashes while holding a user-space mutex. Explain why process termination can release kernel resources but cannot necessarily make shared external data logically consistent.

339. **[Hard]** Two processes cooperate through shared memory and a file. Explain why they may require both memory synchronization and persistent-data coordination.

340. **[Tricky]** Why can a mutex correctly protect shared memory while failing to coordinate another process that ignores the mutex and updates the same file independently?

341. **[Hard]** A user logs out while background processes remain. Which process credentials, sessions, descriptors, and policies determine whether those processes may continue?

342. **[Creative] [Inference]** Why might an OS intentionally allow selected services to survive user logout while terminating ordinary session applications?

343. **[Hard]** A machine enters sleep while devices and processes are active. What state must remain in RAM, what device work must be paused or completed, and how does resume differ from boot?

344. **[Tricky]** Why does resuming from sleep still require driver and device coordination even though the kernel and processes were not recreated?

345. **[Hard]** A machine hibernates while an external network transaction is pending. Why can restoring local process memory not restore the remote connection to the exact earlier state?

346. **[Creative]** Compare the external-state problem in hibernation restoration with the same problem in VM snapshot restoration.

347. **[Hard]** A system update installs a new kernel and driver. Describe the role of storage persistence, bootloader selection, signature verification, boot fallback, and runtime isolation in making the update recoverable.

348. **[Tricky]** Why can a new kernel pass signature verification yet fail during driver initialization?

349. **[Hard]** A process performs an operation requiring file access, network access, memory allocation, and a device. Explain why authorization may succeed for some components and fail for another, causing the complete operation to fail.

350. **[Creative]** Construct a scenario in which confidentiality is preserved, integrity is violated, and availability remains unaffected. Then construct one in which availability fails without a confidentiality breach.

351. **[Hard]** Explain how the same hardware timer supports timekeeping, sleeping, timeouts, preemption, and scheduling without those being the same function.

352. **[Tricky]** Why is a timer interrupt a mechanism while a time slice is part of scheduling policy?

353. **[Hard]** Explain how the same physical RAM can simultaneously contain kernel data, private process pages, shared libraries, file cache, DMA buffers, and guest-VM memory without those categories being interchangeable.

354. **[Creative]** A physical frame previously held one process’s secret, later becomes file cache, and later backs another process’s heap. Identify where clearing is required and where preserving existing contents is intentional.

355. **[Hard]** Explain how page permissions, file permissions, and device permissions protect three different paths by which a process might attempt to reach sensitive information.

356. **[Tricky]** Why can read permission on a file coexist with a non-executable memory mapping of its contents?

357. **[Hard]** A process maps executable code read-only and maps writable data non-executable. Explain how separating permissions reduces exploitation risk.

358. **[Creative] [Inference]** Why might the OS prefer terminating a corrupted process rather than attempting to continue after an unexplained protection fault?

359. **[Hard]** Across booting, process creation, file reading, memory allocation, and crash cleanup, identify where control changes from one component to another and what state must be handed over safely.

360. **[Creative]** Form one coherent explanation of an operating system using only these four ideas: **abstraction, allocation, protection, and coordination**. Your explanation must account for CPU time, memory, files, devices, concurrency, security, virtualization, and booting.

# Final Revision Challenges

361. **[Hard]** Select any ordinary user action, such as opening a photo or playing a song, and justify why it cannot be explained correctly using only one OS subsystem.

362. **[Creative]** Identify three places where the OS deliberately delays work for efficiency and three places where delay can become dangerous.

363. **[Hard]** Identify three resources for which an identifier is meaningful only within a specific context, and explain the relevant context for each.

364. **[Tricky]** Identify three situations where “success” at one system layer does not guarantee success or durability at the next layer.

365. **[Hard]** Identify three cases where the kernel can recover transparently and retry the original instruction or operation, and three cases where termination or an error is necessary.

366. **[Creative]** Identify three examples where sharing improves efficiency and three where it creates isolation or synchronization risk.

367. **[Hard]** Explain why almost every major OS optimization introduces a trade-off involving latency, memory use, complexity, fairness, or security.

368. **[Tricky]** Identify three examples where an object’s visible name or number is not its underlying identity.

369. **[Hard]** Identify three examples where an operation that appears atomic to a user is internally composed of multiple stages requiring coordination.

370. **[Creative] [Inference]** Which single foundational misunderstanding would cause the greatest confusion across the rest of the notes: confusing programs with processes, virtual memory with physical RAM, user authority with kernel mode, or containers with VMs? Defend one choice using connections across multiple sections.

371. **[Hard]** Explain why the OS must preserve both **safety** and **progress**, and give examples where protecting one without the other produces failure.

372. **[Creative]** Construct one scenario that simultaneously contains a recoverable page fault, a blocked thread, a device interrupt, a context switch, and a permission check.

373. **[Hard]** Construct one scenario that simultaneously contains process isolation, explicit IPC, synchronization, file-system access, and a possible deadlock.

374. **[Tricky]** Identify a situation where more concurrency reduces performance, another where more buffering increases latency, and another where more privilege reduces security.

375. **[Hard]** Explain why abstractions such as files, processes, virtual addresses, descriptors, and virtual machines must be backed by kernel-maintained state rather than existing only as application conventions.

376. **[Creative]** Imagine an OS with scheduling but no process isolation. Predict the reliability and security consequences while keeping all other mechanisms unchanged.

377. **[Hard]** Imagine an OS with process isolation but no system calls. Explain why useful interaction with files, devices, and networking would become impossible or unsafe.

378. **[Creative]** Imagine an OS with virtual memory protection but no paging-on-demand. Which protections remain, and which efficiency benefits disappear?

379. **[Hard]** Imagine an OS with interrupts but no scheduler. Explain what device notification can accomplish and what concurrent progress remains impossible.

380. **[Creative]** Imagine an OS with a file system but no file descriptors or persistent open handles. Explain the correctness, performance, and race problems caused by resolving a path for every operation.

381. **[Hard]** Imagine a multithreaded process with no synchronization mechanisms. Which programs could still be correct, and what restrictions would they need to follow?

382. **[Tricky]** Imagine a system where every operation uses one global kernel lock. What correctness benefits and scalability failures would result?

383. **[Hard]** Imagine containers with resource namespaces but no resource limits. Which isolation properties remain, and which availability risks remain unsolved?

384. **[Creative]** Imagine VMs without hypervisor-enforced memory isolation. Why would separate guest kernels no longer provide a meaningful security boundary?

385. **[Hard]** Imagine Secure Boot without runtime memory protection or permission checks. What startup threat is reduced, and what major runtime threats remain?

386. **[Tricky]** Imagine perfect process isolation with a privileged service that authorizes every request incorrectly. Why does the system remain insecure?

387. **[Hard]** Explain why a complete understanding of OS behavior requires following both the **execution path** and the **resource-ownership path** of an operation.

388. **[Creative]** For a file-read operation, separately trace:

* who executes,
* who owns each resource,
* where data resides,
* and which component may block.

389. **[Hard]** For a process crash, separately trace:

* the hardware detection path,
* the kernel decision path,
* the cleanup path,
* and the external effects that cannot be undone.

390. **[Creative] [Inference]** Based strictly on the notes, formulate the strongest general rule you can for deciding whether a problem belongs primarily to an application, the kernel, a driver, a file system, a hypervisor, or hardware.

