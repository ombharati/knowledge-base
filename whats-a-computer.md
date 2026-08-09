# Computer Foundations

## 1. What a Computer Is

### Core idea

A **computer** is an electronic device that:

1. receives data;
2. processes or manipulates it;
3. produces or stores useful information.

### Mental model

Think of a computer as an **input–process–output system**:

| Stage      | Purpose                        | Example                     |
| ---------- | ------------------------------ | --------------------------- |
| Input      | Provides data or commands      | Keyboard press              |
| Processing | Interprets and transforms data | CPU calculates a result     |
| Output     | Presents the result            | Text appears on the monitor |
| Storage    | Preserves data for later       | File saved to an SSD        |

### How it works

At the lowest level, computers represent data using **binary digits**:

* `1`
* `0`

Combinations of binary values can represent:

* text;
* images;
* audio;
* video;
* websites;
* games;
* program instructions.

### Why it matters

The same basic mechanism can support many different tasks because computers are **programmable**. Changing the instructions changes what the machine does.

### Key relationship

> Data becomes useful only when hardware processes it according to software instructions.

### Common mistakes

* Thinking computers understand information the same way humans do.
* Treating a computer as only a desktop or laptop.
* Confusing raw data with the processed result.

### Quick example

Typing a letter:

* keyboard produces input data;
* software interprets the key press;
* CPU processes the instruction;
* monitor displays the letter.

### Recall questions

1. Why can binary represent text, images, and video?
2. Where does software fit into the input–process–output model?
3. What would happen if a computer had hardware but no instructions?
4. Identify the input, processing, output, and storage in taking a digital photo.

---

## 2. Hardware and Software

### Core idea

Every computer system depends on two connected parts:

| Category | Meaning                           | Examples                                |
| -------- | --------------------------------- | --------------------------------------- |
| Hardware | Physical components               | CPU, monitor, keyboard                  |
| Software | Instructions executed by hardware | Browser, operating system, media player |

### Mental model

* **Hardware is the machine.**
* **Software tells the machine what to do.**

Neither is useful alone.

### How they work together

1. A user interacts with software.
2. Software sends instructions to hardware.
3. Hardware performs the requested operations.
4. Software presents the result to the user.

### Key relationships

```text
User ↔ Software ↔ Hardware
```

Examples:

* A word processor tells the CPU how to process text.
* A browser tells network hardware how to request web data.
* A media player tells audio and video hardware how to present media.

### Rule

A physical component is hardware even when it is inside the computer.

Examples:

* motherboard;
* RAM;
* storage drive;
* processor.

### Common mistakes

* Calling the monitor “the computer.”
* Assuming software is physical because it occupies storage space.
* Assuming powerful hardware automatically makes every program fast.
* Forgetting that software must be compatible with the system running it.

### Quick example

When playing a video:

* the media player is software;
* the CPU processes instructions;
* the storage drive supplies the file;
* the monitor and speakers provide output.

### Recall questions

1. Why can software not perform work without hardware?
2. How could the same hardware perform completely different tasks?
3. Which parts of opening a website involve hardware?
4. Is an operating system hardware or software? Explain its role.

---

## 3. Types of Computers

### Core idea

A computer is defined by its ability to process data, not by its shape.

### Main categories

| Type         | Main characteristic                          | Typical use                |
| ------------ | -------------------------------------------- | -------------------------- |
| Desktop      | Fixed workspace and separate components      | Home or office work        |
| Laptop       | Portable, integrated components              | Mobile work                |
| Smartphone   | Small touchscreen mobile computer            | Communication and apps     |
| Tablet       | Touch-oriented portable computer             | Media and lightweight work |
| Smart device | Computer built into another product          | Watches, TVs, appliances   |
| Server       | Provides data or services to other computers | Websites, file sharing     |

### Mental model

Computers exist on a spectrum:

```text
General-purpose ←──────────────→ Specialized
Desktop / Laptop                  Smart appliance
```

* General-purpose computers can run many kinds of software.
* Specialized computers are designed for narrower tasks.

### Personal computers

A **personal computer** is mainly intended for one person at a time.

Common forms:

* desktop;
* laptop.

They usually have similar general capabilities, but differ in portability, size, cooling, and upgrade options.

### Embedded computers

Some devices contain built-in computers:

* televisions;
* game consoles;
* refrigerators;
* smartwatches.

These may have limited functions compared with a desktop or laptop.

### Important assumption

A device does not need a traditional keyboard and monitor to be a computer.

### Common mistakes

* Assuming smartphones are not computers.
* Assuming every computer can run desktop applications.
* Treating device categories as completely separate rather than variations of the same principles.

### Quick example

A smart refrigerator may:

* process temperature data;
* run control software;
* display information;
* connect to a network.

It is therefore performing computer-like operations even though refrigeration is its primary purpose.

### Recall questions

1. What separates a general-purpose computer from a specialized one?
2. Why is a smartphone considered a computer?
3. What capabilities might an embedded computer intentionally lack?
4. When would a laptop be a better personal computer than a desktop?

---

## 4. Servers and Clients

### Core idea

A **server** provides information, files, or services to other computers over a network.

The receiving computer or application acts as a **client**.

### Mental model

Think of a server as a service counter:

* client sends a request;
* server processes the request;
* server returns the requested resource.

```text
Client → Request → Server
Client ← Response ← Server
```

### How it works

When opening a website:

1. the browser requests a web page;
2. the request travels across a network;
3. a web server receives it;
4. the server returns the page data;
5. the browser displays the page.

### Common server uses

* delivering websites;
* storing shared office files;
* handling email;
* hosting applications;
* managing user accounts.

### Why it matters

Servers allow many users and devices to access centralized resources.

### Key relationship

A machine is acting as a server because of the **service it provides**, not only because of its physical appearance.

### Trade-off: centralized storage

| Advantage                   | Risk or cost                         |
| --------------------------- | ------------------------------------ |
| Easier sharing              | Server failure can affect many users |
| Centralized management      | Requires security controls           |
| Access from several devices | Depends on network availability      |
| Consistent file versions    | May create a single point of failure |

### Common mistakes

* Thinking a server must be a huge machine.
* Confusing the internet with one server.
* Assuming a client never provides services of its own.
* Believing website files come directly from another user’s browser.

### Quick example

An office file server stores a shared report. Several employees can access the same central file rather than keeping unrelated copies on every computer.

### Recall questions

1. During web browsing, which component acts as the client?
2. Why can centralized file storage simplify collaboration?
3. What happens if a client cannot reach the server?
4. Could a normal personal computer act as a server? Explain.

---

# Physical Computer Interface

## 5. Buttons, Sockets, and Ports

### Core idea

Ports create physical or wired connections between a computer and other devices.

### Mental model

A port is an **interface with a specific communication or power role**.

Before connecting a device, identify:

1. what the device needs;
2. what connector it uses;
3. whether the computer supports that connector;
4. whether an adapter is required.

### Common connections

| Connection                    | Main purpose                           | Typical devices                  |
| ----------------------------- | -------------------------------------- | -------------------------------- |
| Power socket or charging port | Supplies electrical power              | Power cable, charger             |
| USB                           | Data, peripherals, sometimes power     | Keyboard, printer, flash drive   |
| USB-C                         | Data, charging, video, other functions | Charger, monitor, storage        |
| HDMI                          | Digital video and audio                | Monitor, television              |
| DisplayPort                   | Digital display connection             | Monitor                          |
| Ethernet                      | Wired network connection               | Router, modem, network switch    |
| Audio jack                    | Analog audio input or output           | Headphones, speakers, microphone |
| Optical disc drive            | Reads or writes discs                  | CD, DVD, Blu-ray                 |
| Legacy peripheral port        | Supports older hardware                | Older keyboard, mouse, printer   |

### Important distinction

**Connector shape does not always reveal full capability.**

For example, USB-C may support:

* charging;
* data transfer;
* display output;

but a specific computer port may support only some of these.

### Rule

Never force a connector.

If it does not fit:

* check its orientation;
* confirm the port type;
* verify that it is the correct cable.

### USB

USB is widely used because one standard can connect many peripherals.

Examples:

* keyboards;
* mice;
* printers;
* storage drives.

### USB-C

USB-C describes a small reversible connector format.

Possible uses include:

* charging;
* transferring data;
* connecting a display;
* attaching accessories.

### Monitor connections

Desktop computers normally require a cable between:

* computer case;
* monitor.

Laptops already contain a screen but may support an external monitor.

### Ethernet

Ethernet provides a wired network connection.

```text
Computer → Ethernet cable → Router or modem
```

Typical trade-off:

| Ethernet                  | Wi-Fi                                      |
| ------------------------- | ------------------------------------------ |
| Physical cable required   | No cable between device and router         |
| Usually stable            | Easier mobility                            |
| Limited by cable location | Affected more by distance and interference |

### Audio connections

An audio jack may connect:

* headphones;
* speakers;
* microphones.

Some audio devices instead use:

* USB;
* Bluetooth or another wireless method.

### Legacy ports

Older computers may contain ports for earlier peripherals.

Modern replacements commonly use:

* USB;
* wireless connections.

### Common mistakes

* Assuming every USB-C port has identical capabilities.
* Plugging a monitor cable into the wrong port.
* Confusing an Ethernet port with a telephone-line port.
* Forcing connectors that are incorrectly aligned.
* Assuming a laptop cannot use external peripherals.
* Assuming every computer includes an optical disc drive.

### Quick example

To connect a laptop to an external display:

1. identify the laptop’s display-capable port;
2. identify the monitor’s input;
3. select a compatible cable or adapter;
4. connect both ends;
5. choose the correct display input or system setting.

### Recall questions

1. Why is connector shape alone sometimes insufficient?
2. When would Ethernet be preferable to Wi-Fi?
3. What should you check before buying a display adapter?
4. Why might a USB-C charging cable fail to provide video output?
5. Which connection would you choose for a wired printer, and why?

---

## 6. External Computer Components

### Core idea

A desktop setup separates major functions into different physical components.

### Main components

| Component              | Function                                |
| ---------------------- | --------------------------------------- |
| Computer case          | Contains the main processing components |
| Monitor                | Displays visual output                  |
| Keyboard               | Provides text and command input         |
| Mouse                  | Controls the on-screen pointer          |
| Speakers or headphones | Provide audio output                    |

### Mental model

Each component specializes in one part of interaction:

```text
Keyboard / Mouse → Input
Computer case → Processing and storage
Monitor / Speakers → Output
```

### Computer case

The case contains components responsible for:

* processing;
* memory;
* storage;
* power distribution;
* hardware expansion.

Many desktop cases use a vertical **tower** design, but other shapes exist.

### Monitor

The monitor displays:

* text;
* images;
* video;
* interface elements.

The computer’s graphics hardware generates the display information, while the monitor presents it.

### Display technologies

The source discusses:

* LCD displays;
* LED displays.

Both allow relatively thin monitor designs.

### All-in-one computers

An all-in-one combines:

* monitor;
* main computer components;

into one unit.

#### Trade-off

| Benefit              | Limitation                             |
| -------------------- | -------------------------------------- |
| Compact workspace    | Often harder to repair or upgrade      |
| Fewer visible cables | Display and computer are tied together |
| Simple setup         | Component choices may be limited       |

### Keyboard

A keyboard provides:

* letters;
* numbers;
* shortcuts;
* commands.

Common variations include:

* wired;
* wireless;
* ergonomic.

### Mouse

A mouse controls the pointer through physical movement.

Most modern mice use an **optical sensor** to detect movement.

### Laptop integration

A laptop combines:

* internal computer components;
* screen;
* keyboard;
* pointing device;
* battery.

The built-in pointing surface is called a:

* touchpad;
* trackpad.

### Why it matters

Understanding the function of each component makes setup and troubleshooting easier.

### Common mistakes

* Thinking the monitor performs the main processing.
* Calling the computer case the “CPU.”
* Confusing the pointer shown on-screen with the physical mouse.
* Assuming an all-in-one is only a monitor.
* Assuming laptop components cannot be supplemented with external devices.

### Quick example

If the screen is blank but the computer appears powered on, possible causes include:

* monitor power is off;
* display cable is disconnected;
* wrong monitor input is selected;
* graphics output is not reaching the monitor.

The problem is not automatically the CPU.

### Recall questions

1. Which component produces visual information, and which displays it?
2. Why might an all-in-one be easier to set up but harder to upgrade?
3. How does a touchpad replace a mouse?
4. What should you inspect first when a desktop monitor shows no image?

---

# Internal Components

## 7. Motherboard

### Core idea

The **motherboard** is the main circuit board that connects and supports major computer components.

### Mental model

Think of the motherboard as the computer’s **communication and connection platform**.

It provides pathways between:

* processor;
* RAM;
* storage;
* expansion hardware;
* power connections;
* external ports.

### How it works

Components attach directly or indirectly to the motherboard. Electrical signals travel through its circuits so components can exchange data and commands.

### Why it matters

Component compatibility often depends on the motherboard.

Examples include support for:

* processor type;
* RAM type;
* expansion cards;
* storage connections;
* external ports.

### Common mistake

The motherboard does not itself perform every task. It mainly connects and coordinates components that perform specialized functions.

### Quick example

When opening a file:

1. storage supplies the data;
2. the motherboard provides communication paths;
3. RAM temporarily holds active data;
4. the CPU processes instructions;
5. graphics hardware prepares the display output.

### Recall questions

1. Why is the motherboard compared to a connection platform?
2. How can the motherboard limit future upgrades?
3. Which components communicate when a program opens?
4. Why would incompatible RAM fail even if it physically appears similar?

---

## 8. CPU

### Core idea

The **central processing unit**, or CPU, executes instructions and processes data.

It is often called the computer’s “brain,” although this is only a simplified analogy.

### Mental model

The CPU repeatedly performs a cycle similar to:

```text
Fetch instruction → Interpret instruction → Execute instruction
```

### Main responsibilities

* arithmetic calculations;
* logical comparisons;
* instruction execution;
* coordination of computer operations.

### Cause and effect

More active computation produces heat.

Therefore:

```text
CPU activity → Heat generation → Cooling required
```

### Heat sink

A **heat sink** draws heat away from the processor.

Cooling is necessary because excessive heat can cause:

* reduced performance;
* instability;
* shutdown;
* hardware damage.

### Important distinction

The CPU is not:

* the entire computer case;
* permanent file storage;
* the monitor;
* the operating system.

### Common mistakes

* Referring to the whole desktop tower as the CPU.
* Assuming CPU speed alone determines total computer performance.
* Ignoring cooling and airflow.
* Thinking the CPU stores files permanently.

### Quick example

When calculating a spreadsheet formula, the CPU executes the calculation instructions, but RAM holds active data and storage preserves the spreadsheet file.

### Recall questions

1. Why does CPU activity create a need for cooling?
2. Why is “the CPU is the brain” an incomplete analogy?
3. Which other components affect performance besides the CPU?
4. What symptoms might appear if CPU cooling fails?

---

## 9. RAM

### Core idea

**Random access memory**, or RAM, is short-term working memory used for active tasks.

### Mental model

RAM is like a temporary workspace:

* programs currently running occupy the workspace;
* active data is kept nearby for quick use;
* clearing power clears the workspace.

### Volatility rule

RAM is **volatile**:

> Its contents are normally lost when the computer loses power.

Therefore, RAM should not be used as permanent storage.

### How it works

When a program is opened:

1. program data is read from long-term storage;
2. required data is loaded into RAM;
3. the CPU works with the active data;
4. saved results are written back to storage.

### Why more RAM helps

More RAM allows the computer to keep more active data available at once.

This can improve:

* multitasking;
* use of large applications;
* handling of large files;
* browser tab capacity.

### Limitation

More RAM does not automatically make every operation faster. Performance may still be limited by:

* CPU;
* storage;
* graphics hardware;
* software design.

### RAM vs storage

| RAM                      | Storage                  |
| ------------------------ | ------------------------ |
| Temporary                | Long-term                |
| Active working data      | Saved files and programs |
| Volatile                 | Persistent               |
| Usually faster to access | Usually larger capacity  |

### Common mistakes

* Calling storage “memory” without distinguishing it from RAM.
* Assuming files are permanently saved because they are visible on-screen.
* Believing more RAM can compensate for every slow component.
* Shutting down without saving active work.

### Quick example

A document currently being edited is held partly in RAM. Saving it writes the changes to persistent storage.

### Recall questions

1. Why does unsaved work disappear after a power failure?
2. How does RAM support multitasking?
3. Why might adding RAM fail to improve a CPU-heavy task?
4. What happens between opening and saving a document?

---

## 10. Long-Term Storage

### Core idea

Storage preserves programs and files even when the computer is turned off.

### Main types in the material

| Type              | Storage method             | General characteristics                    |
| ----------------- | -------------------------- | ------------------------------------------ |
| Hard disk drive   | Magnetic spinning platters | Usually cheaper, mechanical                |
| Solid-state drive | Electronic flash storage   | Faster, more durable, often more expensive |

### Mental model

Storage is the computer’s **filing cabinet**.

RAM is the temporary desk where current work is placed.

### Hard disk drive

An HDD uses moving mechanical parts and magnetic platters.

Consequences:

* physical motion takes time;
* shocks may damage the mechanism;
* high capacity may be available at lower cost.

### Solid-state drive

An SSD has no spinning platter.

Consequences:

* faster access;
* greater resistance to physical movement;
* quieter operation;
* typically higher cost per unit of storage.

### Key trade-off

| Priority                               | Likely preference   |
| -------------------------------------- | ------------------- |
| Lower cost for large capacity          | HDD                 |
| Faster startup and application loading | SSD                 |
| Better durability in portable devices  | SSD                 |
| Large secondary archive                | HDD may be suitable |

### Important distinction

Storage capacity and processing speed are different properties.

A computer can have:

* large storage but slow processing;
* fast processing but little storage.

### Common mistakes

* Treating RAM and storage as interchangeable.
* Assuming a larger drive is always a faster drive.
* Thinking files are safe merely because they are stored locally.
* Assuming SSDs make every computation faster.

### Quick example

Installing the operating system on an SSD can reduce startup and loading times because data can be retrieved faster, but it does not directly increase the CPU’s calculation ability.

### Recall questions

1. Why are SSDs generally more resistant to physical shock?
2. When could an HDD still be a reasonable choice?
3. Why does replacing an HDD with an SSD improve loading but not every task?
4. What is the difference between storage capacity and RAM capacity?

---

## 11. Expansion Cards and Upgradeability

### Core idea

Expansion slots allow additional hardware capabilities to be added to some computers.

### Examples

| Expansion card        | Added capability                |
| --------------------- | ------------------------------- |
| Graphics card         | Improved graphics processing    |
| Wireless network card | Wireless network connectivity   |
| Sound card            | Additional audio capabilities   |
| Other interface card  | Additional ports or connections |

### Mental model

An expansion slot is a standardized attachment point for extending the system.

### Desktop advantage

Desktop computers often provide:

* more internal space;
* expansion slots;
* replaceable components;
* better cooling capacity.

### Laptop limitation

Many laptops have limited internal expansion because portability requires:

* smaller dimensions;
* lower weight;
* tightly integrated parts;
* lower power consumption.

### Trade-off

```text
Portability and compactness ↔ Upgrade flexibility
```

Greater integration can reduce:

* size;
* weight;
* setup complexity;

but may also reduce:

* repairability;
* replaceability;
* hardware expansion.

### Exception

A laptop can still be extended externally using:

* USB devices;
* monitors;
* keyboards;
* mice;
* storage drives;
* docking equipment.

### Common mistakes

* Assuming every desktop supports every expansion card.
* Buying a card without checking motherboard compatibility.
* Assuming laptops cannot use any additional hardware.
* Confusing external accessories with internal expansion cards.

### Quick example

A desktop without built-in Wi-Fi may gain wireless networking through an expansion card. A laptop may use an external USB wireless adapter instead.

### Recall questions

1. Why are desktops generally easier to upgrade?
2. What compatibility checks are needed before buying an expansion card?
3. How can external devices reduce a laptop’s expansion limitations?
4. What trade-off does tightly integrated hardware create?

---

## 12. Power Supply and Battery

### Core idea

Computer components require controlled electrical power.

### Desktop power supply

A desktop power supply unit:

1. receives electricity from the wall;
2. converts it into forms required by computer components;
3. distributes power throughout the system.

### Laptop battery

A laptop battery stores electrical energy so the computer can operate without continuous wall power.

### Mental model

* The wall outlet is the energy source.
* The power supply is the converter and distributor.
* The battery is temporary stored energy.

### Cause and effect

```text
No suitable power → Components cannot operate
Unstable power → Risk of malfunction or damage
Battery present → Temporary operation during an outage
```

### Why it matters

Power requirements influence:

* portability;
* performance;
* component selection;
* cooling;
* battery life.

### Common mistakes

* Assuming the desktop power supply only provides a cable connection.
* Using an incompatible laptop charger.
* Expecting a battery to retain full capacity forever.
* Forgetting that high-performance components generally require more power.

### Quick example

During a short power outage:

* a desktop without backup power shuts down;
* a charged laptop can continue operating from its battery.

### Recall questions

1. Why must a desktop power supply convert electricity?
2. How does a battery change a laptop’s behavior during an outage?
3. Why can higher-performance components affect power requirements?
4. What risks arise from using an incompatible charger?

---

# Laptop and Desktop Comparison

## 13. Portability and Integration

### Core idea

Laptops and desktops provide similar general computing functions but optimize for different priorities.

### Mental model

```text
Laptop: integration + portability
Desktop: flexibility + workspace performance
```

### Main comparison

| Factor                       | Laptop                        | Desktop                       |
| ---------------------------- | ----------------------------- | ----------------------------- |
| Portability                  | High                          | Low                           |
| Setup                        | Components already integrated | Separate components connected |
| Built-in battery             | Yes                           | Usually no                    |
| Screen size                  | Usually smaller               | Flexible and often larger     |
| Keyboard and pointing device | Built in                      | Usually separate              |
| Upgradeability               | Often limited                 | Usually better                |
| Peripheral choice            | Built-ins fixed               | Easy to mix and match         |
| Workspace use                | Compact                       | Requires more space           |

### Laptop advantages

* easy to carry;
* quick initial setup;
* built-in display and input devices;
* continues working temporarily during a power outage;
* can operate away from a wall outlet.

### Laptop limitations

* smaller screen;
* less internal expansion;
* fixed built-in keyboard and display;
* battery requires charging;
* compact cooling can limit performance.

### Desktop advantages

* greater choice of monitor, keyboard, and mouse;
* easier repair and upgrading;
* potentially larger displays;
* more internal space;
* less emphasis on battery efficiency.

### Desktop limitations

* not designed for regular transport;
* more cables and separate parts;
* requires a fixed workspace;
* normally stops immediately during a power outage.

### Best-of-both-worlds setup

A laptop can connect to:

* external monitor;
* keyboard;
* mouse;
* speakers;
* network cable.

This creates a desktop-like workspace while preserving portability.

### Decision rule

Choose based on the dominant constraint:

| Main need                          | Better starting choice  |
| ---------------------------------- | ----------------------- |
| Frequent travel                    | Laptop                  |
| Easy upgrades                      | Desktop                 |
| Minimal setup                      | Laptop                  |
| Large permanent workspace          | Desktop                 |
| One device for mobile and desk use | Laptop with peripherals |
| Maximum peripheral freedom         | Desktop                 |

### Common mistakes

* Assuming one category is universally better.
* Comparing only purchase price.
* Ignoring repairability and upgrade cost.
* Buying a laptop for portability but leaving it permanently fixed.
* Buying a desktop without accounting for monitor and peripheral costs.
* Assuming a laptop cannot support a large external screen.

### Quick example

A student who works both on campus and at home may use a laptop during the day, then connect it to an external monitor and keyboard at home.

### Recall questions

1. Which trade-off explains many laptop design limitations?
2. When does a laptop effectively become a desktop-like system?
3. Why might a desktop provide better long-term upgrade value?
4. Which factors should be considered beyond initial price?
5. Choose a computer type for a travelling designer and justify the trade-offs.

---

# Operating Systems and Applications

## 14. Operating System

### Core idea

An **operating system**, or OS, is the main software layer that allows users and applications to interact with computer hardware.

### Mental model

The OS acts as an intermediary:

```text
User
  ↓
Applications
  ↓
Operating system
  ↓
Hardware
```

### Main responsibilities

The operating system helps manage:

* user interaction;
* application execution;
* files and folders;
* memory;
* connected devices;
* hardware resources;
* system settings.

### Why it is necessary

Humans do not normally control hardware directly using binary-level instructions.

The OS provides usable abstractions such as:

* icons;
* windows;
* menus;
* files;
* folders;
* touch controls.

### Common operating-system families in the material

| Device category   | Examples                                  |
| ----------------- | ----------------------------------------- |
| Personal computer | Windows, Mac operating systems, Chrome OS |
| Mobile device     | iOS, Android                              |

### Hardware–OS relationship

Together, hardware and the operating system determine much of what the device can do.

The OS must support:

* the device’s hardware;
* required applications;
* expected interaction method.

### Application compatibility

An application may be:

* available on several operating systems;
* available only on one operating system;
* offered in different versions for different systems.

### Rule

Before installing software, verify that it supports the operating system being used.

### Trade-off

Different operating systems may emphasize different:

* interfaces;
* software ecosystems;
* hardware support;
* device integration;
* security models.

### Common mistakes

* Confusing the OS with individual applications.
* Assuming a program works on every device.
* Downloading the wrong software version.
* Thinking the operating system is stored only in RAM.
* Treating visual differences as the only differences between operating systems.

### Quick example

A word-processing application asks the operating system to:

* open a file;
* allocate memory;
* display a window;
* receive keyboard input;
* save changes to storage.

The application does not normally manage all hardware directly.

### Recall questions

1. Why is the operating system described as an intermediary?
2. What would applications need to do without an operating system?
3. Why can an application be unavailable for a particular device?
4. How does the OS simplify hardware interaction?
5. Which OS responsibilities are involved when saving a file?

---

## 15. Applications

### Core idea

An **application**, or app, is software designed to help the user perform a particular task or activity.

### Mental model

* The operating system provides the environment.
* Applications provide task-specific capabilities.

```text
Operating system = platform
Application = tool used on the platform
```

### Common purposes

| Category      | Examples of tasks               |
| ------------- | ------------------------------- |
| Productivity  | Writing documents, spreadsheets |
| Communication | Email, messaging                |
| Navigation    | Maps and location search        |
| Entertainment | Videos, games, music            |
| Web access    | Browsing websites               |
| Creative work | Editing images or media         |

### Mobile and desktop applications

Mobile apps are designed for devices such as:

* smartphones;
* tablets.

Desktop applications are designed for:

* desktops;
* laptops.

The distinction is influenced by:

* screen size;
* touch versus mouse input;
* available hardware;
* operating system.

### Installed applications

Some apps are included with the device.

Others may be:

* downloaded;
* purchased;
* installed later.

### Web applications

Some applications run mainly through a web browser.

These are often called **web apps**.

They may rely on:

* internet access;
* remote servers;
* browser capabilities.

### Relationship with the OS

Applications depend on operating-system services for tasks such as:

* drawing windows;
* accessing files;
* using the network;
* printing;
* receiving input.

### Common mistakes

* Assuming “app” refers only to mobile software.
* Confusing a website with the browser used to access it.
* Assuming every app must be installed locally.
* Downloading an application without checking OS compatibility.
* Treating the application and its files as the same thing.

### Quick example

Microsoft Word is an application used to create documents. The document is user data, while Word is the software that edits it.

### Recall questions

1. How is an application different from an operating system?
2. Why can desktop and mobile versions of the same app behave differently?
3. What operating-system services might a browser use?
4. How is a document different from the application that opens it?
5. Under what conditions would a web app stop working even if the computer is functioning?

# Setting Up a Desktop Computer

## 16. Planning the Workspace

### Core idea

A reliable setup begins with arranging the workspace before connecting cables.

### Mental model

Set up in this order:

```text
Plan → Position → Connect → Verify → Power on
```

### Before connecting anything

* Remove packaging and protective material.
* Identify each component and cable.
* Choose where the computer will be used.
* Leave enough room for ventilation.
* Position cables to avoid pulling and tripping.

### Why it matters

Planning first reduces:

* incorrect connections;
* tangled cables;
* blocked airflow;
* repeated rearrangement;
* accidental equipment damage.

### Common mistakes

* Connecting everything before choosing a layout.
* Placing the computer where ventilation is blocked.
* Stretching cables tightly between devices.
* Putting drinks near the computer.
* Throwing away unidentified adapters or documentation too early.

### Quick example

Place the monitor at a comfortable viewing distance before connecting its cable. Otherwise, the cable may later be too short or routed awkwardly.

### Recall questions

1. Why should workspace arrangement happen before cable connection?
2. Which setup choices affect both safety and cooling?
3. What problems can tightly stretched cables create?
4. How would you arrange a workspace with limited desk space?

---

## 17. Connecting the Monitor

### Core idea

A separate desktop monitor must receive both:

* display data from the computer;
* electrical power.

### Mental model

```text
Computer → Display cable → Monitor
Wall power → Power cable → Monitor
```

### Connection process

1. Identify the monitor’s display connector.
2. Find the matching display output on the computer.
3. Connect the cable without forcing it.
4. Connect the monitor’s power cable.
5. Select the correct monitor input if required.

### Common display connections

* HDMI;
* USB-C;
* DisplayPort;
* VGA;
* DVI.

### Exception

An **all-in-one computer** contains the display and main computer components in one unit, so a separate display cable may not be required.

### Rule

A connector should fit with little force. If it does not:

* check the orientation;
* confirm that the port is correct;
* check whether an adapter is needed.

### Common mistakes

* Connecting the cable to the wrong port.
* Forgetting the monitor’s power cable.
* Selecting the wrong input source on the monitor.
* Forcing a connector.
* Assuming a blank screen means the entire computer is broken.

### Quick example

A monitor connected through HDMI may remain blank if its selected input is DisplayPort. The cable is connected, but the monitor is listening to the wrong source.

### Recall questions

1. Why does a monitor normally require two separate connections?
2. What should you inspect before assuming a blank monitor is faulty?
3. When would a display adapter be required?
4. Why should a connector never be forced?

---

## 18. Connecting Keyboard and Mouse

### Core idea

A keyboard and mouse may connect through:

* USB;
* a wireless receiver;
* built-in wireless technology;
* older legacy ports.

### Wired devices

Most modern wired keyboards and mice use USB.

Basic process:

1. locate an available USB port;
2. insert the connector correctly;
3. wait for the operating system to recognize the device.

Some keyboards contain an extra USB port that can accept a mouse or another low-power device.

### Wireless devices

A wireless keyboard or mouse may use:

* a small USB receiver;
* Bluetooth or another built-in wireless system.

Wireless setup may require **pairing**, which creates a recognized connection between the device and computer.

### Mental model

```text
Physical cable → Direct connection
Wireless device → Wireless link + pairing
```

### Wireless requirements

Depending on the device:

* insert its receiver;
* install or charge batteries;
* turn the device on;
* activate pairing mode;
* select it in system settings.

### Common mistakes

* Forgetting to install batteries.
* Leaving the device switched off.
* Losing the wireless receiver.
* Assuming all wireless devices connect automatically.
* Plugging an older connector in while the computer is running when its instructions require shutdown.

### Quick example

A Bluetooth mouse may receive power correctly but remain unusable until the operating system pairs with it.

### Recall questions

1. What additional step may wireless devices require?
2. Why might a powered wireless keyboard still not work?
3. How does a USB receiver differ from built-in Bluetooth?
4. What would you check first if a newly connected mouse is unresponsive?

---

## 19. Connecting Audio Devices

### Core idea

Speakers, headphones, and microphones require a compatible audio connection.

### Possible connections

| Connection | Typical use                              |
| ---------- | ---------------------------------------- |
| Audio jack | Analog headphones, speakers, microphones |
| USB        | Digital audio devices                    |
| Wireless   | Bluetooth headphones or speakers         |

### Audio jack identification

Desktop computers may use separate audio ports for:

* sound output;
* microphone input.

A green port commonly indicates audio output, but labels and symbols are more reliable than colour alone.

### Why it matters

Using the wrong audio port can produce:

* no sound;
* no microphone input;
* incorrect device detection.

### Common mistakes

* Plugging headphones into a microphone port.
* Assuming every computer uses the same port colours.
* Forgetting to select the new audio device in system settings.
* Connecting speakers without providing separate power when required.

### Quick example

USB headphones may be physically connected but silent because the operating system is still sending sound to the monitor’s speakers.

### Recall questions

1. Why is physical connection not always enough for audio output?
2. How can you distinguish an input port from an output port?
3. Why might external speakers need two connections?
4. What setting should be checked when sound plays through the wrong device?

---

## 20. Connecting Power Safely

### Core idea

Power should be connected after the main components are arranged and connected.

### Typical desktop power connections

A standard desktop setup commonly includes:

* one power cable for the computer case;
* one power cable for the monitor.

Additional devices may also need separate power.

### Recommended sequence

1. Connect component power cables.
2. Plug them into a surge protector.
3. Plug the surge protector into the wall.
4. Turn on the surge protector if it has a switch.
5. Check all connections.
6. Turn on the computer.

### Surge protector

A surge protector helps reduce damage caused by sudden voltage increases.

### Important distinction

A surge protector is not necessarily a battery backup.

| Device                       | Main purpose                                                      |
| ---------------------------- | ----------------------------------------------------------------- |
| Surge protector              | Limits damage from voltage spikes                                 |
| Uninterruptible power supply | Provides temporary battery power and may provide surge protection |

### Common mistakes

* Overloading a power strip.
* Confusing an ordinary power strip with a surge protector.
* Turning on the computer before checking connections.
* Routing power cables across walking paths.
* Using damaged cables or loose sockets.

### Quick example

During a voltage spike, a surge protector may protect components. During a complete power outage, it normally cannot keep a desktop running unless it also contains battery backup.

### Recall questions

1. How does a surge protector differ from a battery backup?
2. Why should power be connected near the end of setup?
3. What risks come from overloaded power strips?
4. Which setup would protect against both voltage spikes and short outages?

---

## 21. Final Setup Check

### Core idea

Before first use, verify the complete system rather than troubleshooting one disconnected component at a time.

### Checklist

* Monitor cable connected.
* Monitor powered.
* Keyboard connected or paired.
* Mouse connected or paired.
* Audio devices connected if needed.
* Computer case powered.
* Surge protector switched on.
* Ventilation openings unobstructed.
* Cables placed safely.
* Workspace comfortable.

### Mental model

Troubleshooting begins with the simplest physical dependencies:

```text
Power → Connection → Device selection → Software
```

### Common mistakes

* Changing software settings before checking power.
* Reconnecting cables randomly without identifying them.
* Ignoring monitor input selection.
* Blocking fans after moving the case into position.

### Quick example

If the keyboard does not respond:

1. check whether it has power;
2. inspect the cable or receiver;
3. confirm wireless pairing;
4. try another port;
5. then investigate software settings.

### Recall questions

1. Why should physical checks come before software troubleshooting?
2. What is the most efficient order for diagnosing a disconnected device?
3. Which setup problems may not appear until the computer begins producing heat?
4. Build a troubleshooting sequence for a monitor that displays “No Signal.”

---

# Internet Connections

## 22. Internet Service Basics

### Core idea

Accessing the internet requires a connection supplied by an **internet service provider**, or ISP.

### Mental model

```text
Device → Home network equipment → ISP → Internet
```

### ISP responsibilities

An ISP may:

* activate the connection;
* provide account access;
* supply or configure a modem;
* send a technician;
* provide technical support.

### Important distinction

The internet is not the same as Wi-Fi.

* **Internet service** connects the home to outside networks.
* **Wi-Fi** connects nearby devices wirelessly to the home network.

A Wi-Fi network can exist without working internet access.

### Common mistakes

* Treating Wi-Fi and internet access as identical.
* Assuming a router creates internet service by itself.
* Buying equipment before checking ISP compatibility.
* Expecting every location to support every connection type.

### Quick example

A phone may show that it is connected to Wi-Fi, but websites may fail because the router has lost its connection to the ISP.

### Recall questions

1. How can Wi-Fi work while internet access does not?
2. What role does the ISP play?
3. Why must equipment compatibility be checked with the provider?
4. Which parts of the connection are inside the home, and which are external?

---

## 23. Types of Internet Connections

### Core idea

Internet connection types differ in:

* speed;
* availability;
* cost;
* transmission medium;
* reliability.

### Main types from the material

| Type     | Transmission medium             | General characteristic          |
| -------- | ------------------------------- | ------------------------------- |
| Dial-up  | Telephone line                  | Very slow                       |
| DSL      | Telephone line                  | Broadband, faster than dial-up  |
| Cable    | Cable television infrastructure | Broadband                       |
| Fiber    | Fibre-optic cable               | Very high speed                 |
| Cellular | Mobile wireless network         | Wireless and location-dependent |

### Dial-up

Dial-up uses a telephone line and is much slower than modern broadband.

It may still exist where other services are unavailable.

### DSL

DSL also uses telephone infrastructure but supports significantly faster data transmission than dial-up.

### Cable

Cable internet uses infrastructure associated with cable television service.

Its performance may vary based on:

* provider;
* local network capacity;
* number of active users.

### Fiber

Fiber transmits data using light through fibre-optic cables.

General advantages:

* high speed;
* high capacity;
* strong performance over distance.

Possible limitations:

* limited availability;
* higher installation or service cost.

### Cellular internet

Cellular internet may use technologies marketed as:

* 4G;
* LTE;
* 5G.

It can provide internet access to:

* smartphones;
* portable hotspots;
* home cellular routers.

Performance depends on:

* network coverage;
* signal strength;
* congestion;
* service plan.

### Mental model

No connection type is universally best.

```text
Best option = Required performance + Local availability + Acceptable cost
```

### Common mistakes

* Assuming the fastest advertised technology is available everywhere.
* Comparing plans only by maximum speed.
* Ignoring data limits, reliability, or installation cost.
* Assuming all cellular connections provide the same speed.
* Believing DSL and dial-up are equivalent because both use phone lines.

### Quick example

A rural user may choose cellular internet because fibre and cable are unavailable, even if cellular performance varies more with signal conditions.

### Recall questions

1. Why can two connections using telephone infrastructure perform very differently?
2. Which factors besides speed should influence the choice of ISP?
3. Why may cellular internet performance change throughout the day?
4. Under what conditions could a slower technology still be the practical choice?
5. Why is fibre not automatically the best choice for every household?

---

## 24. Broadband

### Core idea

**Broadband** refers to internet connections designed to provide much higher data rates than dial-up.

### Examples in the material

* DSL;
* cable;
* fibre;
* some cellular services.

### Why it matters

Higher bandwidth supports activities such as:

* video streaming;
* large downloads;
* online games;
* video calls;
* multiple connected users.

### Mental model

Bandwidth is like the capacity of a road:

* higher capacity allows more data to move at once;
* actual travel time still depends on traffic and route conditions.

### Important distinction

Bandwidth and responsiveness are related but not identical.

| Concept     | Meaning                               |
| ----------- | ------------------------------------- |
| Bandwidth   | Amount of data transferable over time |
| Latency     | Delay before data begins to arrive    |
| Reliability | Consistency of the connection         |

### Common mistakes

* Assuming more bandwidth removes all delay.
* Treating advertised maximum speed as guaranteed speed.
* Ignoring upload speed.
* Assuming one fast connection will perform equally well with any number of users.

### Quick example

A high-bandwidth connection may download a large video quickly but still feel slow in an online game if latency is high.

### Recall questions

1. Why does high bandwidth help households with many devices?
2. How can a connection have high bandwidth but poor responsiveness?
3. Which activities depend strongly on upload performance?
4. Why can actual speed differ from advertised speed?

---

# Modems and Routers

## 25. Modem

### Core idea

A modem connects the home network or computer to the ISP’s service.

### Mental model

The modem is the boundary device between:

* the provider’s connection;
* the customer’s local equipment.

```text
ISP line → Modem → Computer or router
```

### How it works

The modem communicates using the connection technology supplied by the ISP, such as:

* cable;
* DSL;
* fibre-related equipment;
* cellular service.

The exact setup depends on the provider and connection type.

### ISP-provided vs purchased modem

| ISP-provided                 | User-purchased                   |
| ---------------------------- | -------------------------------- |
| Usually easier to support    | May avoid equipment rental costs |
| Likely to be compatible      | Compatibility must be verified   |
| May include maintenance      | User handles replacement         |
| May combine modem and router | Greater equipment choice         |

### Common mistakes

* Purchasing a modem without checking compatibility.
* Calling every network device a modem.
* Assuming a modem automatically provides strong Wi-Fi.
* Resetting provider equipment without knowing its configuration.

### Quick example

A cable modem connects to the cable service line and converts the provider connection into a network connection usable by a router or computer.

### Recall questions

1. What boundary does a modem sit across?
2. Why must a user-purchased modem be approved by the ISP?
3. Can a modem provide internet to one wired computer without a separate router?
4. Why are modem and router often confused?

---

## 26. Router

### Core idea

A router connects multiple devices and directs network traffic between them and the internet.

### Mental model

The router is the local network’s traffic manager.

```text
                    → Laptop
Internet → Router  → Phone
                    → Television
                    → Desktop
```

### Main functions

A home router may:

* create a local network;
* connect several devices;
* provide Ethernet ports;
* broadcast Wi-Fi;
* direct data to the correct device;
* provide basic security functions.

### Modem–router relationship

| Modem                                  | Router                              |
| -------------------------------------- | ----------------------------------- |
| Connects to the ISP                    | Connects local devices              |
| Provides the external internet link    | Shares and manages that link        |
| Usually one provider-facing connection | Supports multiple local connections |

### Combined devices

Some equipment contains both:

* modem functionality;
* router functionality.

Therefore, one physical box may perform both roles.

### Important exception

A router can create a local network without an active internet connection.

Devices may still be able to:

* communicate locally;
* access shared storage;
* use local printers.

### Common mistakes

* Assuming the router is the internet source.
* Buying a second router when the modem already includes one.
* Placing the router where walls and obstacles weaken the signal.
* Connecting equipment to the wrong ports.
* Assuming more signal bars always mean the internet connection itself is healthy.

### Quick example

Two computers may exchange files through the router even when the ISP connection is unavailable.

### Recall questions

1. How does a router differ from a modem?
2. What can still work when the router is operating but the ISP is down?
3. Why may one physical device be called both a modem and router?
4. Which device decides where local network data should go?

---

# Wireless Home Networks

## 27. Wi-Fi Network

### Core idea

A wireless router broadcasts a local network that compatible devices can join through Wi-Fi.

### Setup process

1. Connect the router to the internet connection or modem.
2. Follow the router’s configuration instructions.
3. Choose a network name.
4. Enable strong encryption.
5. Set a strong password.
6. Connect each device to the network.

### Network name

The network name is also called the **SSID**.

Its purpose is to identify the wireless network to nearby devices.

### Mental model

```text
SSID = Network identity
Password = Access credential
Encryption = Protection for transmitted data
```

### Joining the network

On each device:

1. open Wi-Fi or network settings;
2. select the correct SSID;
3. enter the password;
4. confirm the connection.

### Wired alternative

Devices without wireless capability may connect through:

* an Ethernet cable;
* an added wireless adapter or card.

### Why it matters

A wireless home network allows several devices to share one internet connection without being physically tied to one location.

### Common mistakes

* Selecting a neighbour’s similarly named network.
* Leaving the default network name and password unchanged.
* Placing the router in a hidden or obstructed location.
* Assuming a device is connected just because Wi-Fi is enabled.
* Sharing the password without considering network security.

### Quick example

A desktop without Wi-Fi can connect directly to the router through Ethernet while phones and laptops connect wirelessly.

### Recall questions

1. What distinct roles do the SSID and password perform?
2. How can a device without Wi-Fi join the same network?
3. Why can two devices use different connection methods on one router?
4. What would you check if the correct SSID is visible but connection fails?

---

## 28. Wireless Security

### Core idea

Wireless encryption protects data travelling between devices and the router and prevents unauthorized access.

### Recommended standards in the material

* WPA2;
* WPA3, when supported.

### Mental model

A secure Wi-Fi network needs three elements:

```text
Recognized network + Strong authentication + Encryption
```

### Strong password characteristics

A strong network password should generally be:

* long;
* difficult to guess;
* unique;
* different from the router’s administrative password.

### Important distinction

| Password                      | Purpose                            |
| ----------------------------- | ---------------------------------- |
| Wi-Fi password                | Allows devices to join the network |
| Router administrator password | Allows changes to router settings  |

Using the same password for both increases risk.

### Cause and effect

Weak security may allow an unauthorized user to:

* use the internet connection;
* access poorly protected local devices;
* inspect some network activity;
* change settings if administrative access is exposed.

### Common mistakes

* Leaving the router administrator password at its default.
* Using old or weak encryption.
* Reusing a simple personal password.
* Assuming a hidden SSID provides strong security.
* Giving broad access when a guest network would be safer.

### Quick example

A guest network can give visitors internet access without placing them on the same local network as personal computers and storage devices.

### Recall questions

1. Why are the Wi-Fi and administrator passwords separate?
2. What risks arise when a neighbour joins an unsecured network?
3. Why is hiding the SSID not a substitute for encryption?
4. When would a guest network be useful?
5. What should be checked before enabling WPA3 exclusively?

---

## 29. Wired vs Wireless Networking

### Core idea

Ethernet and Wi-Fi connect devices to the same network using different transmission methods.

### Comparison

| Factor            | Ethernet                 | Wi-Fi                                    |
| ----------------- | ------------------------ | ---------------------------------------- |
| Mobility          | Low                      | High                                     |
| Cable required    | Yes                      | No                                       |
| Interference      | Usually lower            | Can be affected by obstacles and signals |
| Stability         | Often high               | Depends on environment                   |
| Setup location    | Limited by cable         | Flexible within coverage                 |
| Security exposure | Requires physical access | Signal extends through surrounding space |

### Mental model

```text
Ethernet optimizes stability.
Wi-Fi optimizes mobility.
```

### Trade-off

The best choice depends on the task.

Ethernet may be preferable for:

* desktop computers;
* online gaming;
* large transfers;
* fixed devices;
* stable video calls.

Wi-Fi may be preferable for:

* phones;
* tablets;
* portable laptops;
* areas where cables are impractical.

### Hybrid approach

A network can use both:

* Ethernet for fixed high-demand devices;
* Wi-Fi for mobile devices.

### Common mistakes

* Treating the choice as all-or-nothing.
* Assuming Wi-Fi performance is identical everywhere in the home.
* Using Wi-Fi for a fixed device located beside the router without considering Ethernet.
* Assuming Ethernet fixes problems caused by the ISP connection itself.

### Quick example

A desktop beside the router may use Ethernet for stability, while a laptop uses Wi-Fi so it can move around the home.

### Recall questions

1. Why is a hybrid network often practical?
2. Which connection would you choose for a fixed gaming computer?
3. Why can Ethernet remain slow when the ISP is having problems?
4. What environmental factors can reduce Wi-Fi performance?

---

# Cloud Computing and Storage

## 30. The Cloud

### Core idea

“The cloud” refers to computing services and data stored on remote internet-connected servers rather than only on the local device.

### Mental model

The cloud is not an invisible place. It is someone else’s networked computing infrastructure.

```text
Local device ↔ Internet ↔ Remote servers
```

### Cloud-based activities

* storing files;
* editing documents;
* synchronizing photos;
* accessing applications;
* backing up devices;
* sharing content.

### Why it matters

Cloud services separate access to information from one specific physical device.

This enables:

* access from multiple devices;
* remote collaboration;
* easier sharing;
* off-device backup;
* synchronized data.

### Important assumption

Cloud access usually depends on:

* internet availability;
* account access;
* service availability;
* correct permissions.

### Common mistakes

* Thinking cloud data exists nowhere physically.
* Assuming cloud storage always means backup.
* Believing internet access is unnecessary after a file is uploaded.
* Assuming every cloud service provides equal privacy and security.
* Forgetting that account loss can block access.

### Quick example

A photo uploaded from a phone can later be viewed from a laptop because both devices access the same remote account and server-stored file.

### Recall questions

1. Why is “the cloud” still physical infrastructure?
2. What dependencies exist when accessing a cloud file?
3. How does cloud storage reduce dependence on one device?
4. What new risks appear when files depend on an online account?

---

## 31. Cloud Applications

### Core idea

A cloud-based application performs some or most of its work using remote servers and may be accessed through a browser or installed app.

### Web applications

A **web app** runs through a web browser.

Examples of possible tasks:

* creating documents;
* editing spreadsheets;
* managing email;
* sharing projects.

### Mental model

```text
Browser or app = Interface
Remote service = Data storage and processing
```

The exact division of work varies. Some cloud apps still perform significant processing locally.

### Advantages

* access from different devices;
* automatic updates;
* easy collaboration;
* reduced need for local installation;
* centralized file storage.

### Limitations

* internet dependency;
* service outages;
* account dependency;
* possible subscription costs;
* privacy considerations;
* reduced control over service changes.

### Exception

Some cloud applications support offline access and synchronize changes when internet connectivity returns.

### Common mistakes

* Assuming all browser-based software works offline.
* Confusing the browser with the cloud application.
* Assuming files automatically synchronize immediately.
* Editing the wrong shared version of a file.
* Believing remote storage removes the need for access control.

### Quick example

A document editor may allow two users to modify the same remote document, reducing the need to exchange multiple email attachments.

### Recall questions

1. What is the difference between a browser and a web app?
2. Why can cloud apps simplify collaboration?
3. What happens when two offline users edit the same file?
4. Which cloud-application limitations come from dependence on a provider?

---

## 32. Cloud Storage

### Core idea

Cloud storage keeps files on remote servers so they can be accessed through an internet-connected account.

### Typical uses

* photos;
* documents;
* music;
* device files;
* shared project folders.

### Mental model

Cloud storage acts like a remotely accessible drive, but with additional dependencies:

```text
File + Provider + Account + Network + Permissions
```

### Main benefits

* access from several devices;
* reduced dependence on one local drive;
* convenient sharing;
* synchronization;
* possible version history.

### Main trade-offs

| Benefit                        | Cost or risk                           |
| ------------------------------ | -------------------------------------- |
| Access anywhere                | Depends on connectivity                |
| Easy sharing                   | Incorrect permissions may expose files |
| Off-device storage             | Provider controls infrastructure       |
| Automatic synchronization      | Mistakes may synchronize too           |
| Reduced local storage pressure | Storage limits or fees may apply       |

### Synchronization

Synchronization keeps copies or representations of files consistent across devices.

Important consequence:

> Deleting a synchronized file on one device may delete it from other devices and the cloud.

### Common mistakes

* Treating synchronization as independent backup.
* Sharing a public link accidentally.
* Assuming deleted synchronized files remain elsewhere.
* Ignoring storage quotas.
* Keeping the only account recovery method on the device being backed up.

### Quick example

A synchronized folder appears on a laptop and desktop. Editing a file on the laptop updates the cloud copy, which then updates the desktop copy.

### Recall questions

1. Why is synchronization not always the same as backup?
2. How can cloud sharing create a privacy risk?
3. What might happen when a synchronized file is deleted locally?
4. Which dependencies must work for cloud storage to remain accessible?

---

## 33. Cloud Backup

### Core idea

A cloud backup service copies important data to remote servers so it can be restored after loss or damage.

### Mental model

Backup is a recovery system, not merely another location where files appear.

```text
Original data → Backup copy → Restore after loss
```

### Automated backup

Some services run continuously or according to a schedule.

Automation helps maintain recent copies without requiring manual action every time.

### Why off-site backup matters

A local computer and a nearby external drive can both be damaged by the same event.

Examples:

* theft;
* fire;
* electrical damage;
* flooding;
* malware;
* physical accident.

Remote backup reduces this shared-location risk.

### Backup vs synchronization

| Synchronization                       | Backup                               |
| ------------------------------------- | ------------------------------------ |
| Keeps active copies consistent        | Preserves recoverable copies         |
| Changes propagate quickly             | May retain historical versions       |
| Deletion may propagate                | Deleted files may remain recoverable |
| Designed for access and collaboration | Designed primarily for recovery      |

### Important exception

Not every cloud storage service provides full backup behaviour.

Check whether the service supports:

* version history;
* deleted-file recovery;
* automatic backup;
* complete device restoration;
* encryption;
* retention periods.

### Common mistakes

* Assuming uploaded files are automatically protected forever.
* Never testing restoration.
* Backing up only some important folders.
* Depending on one backup copy.
* Failing to secure the backup account.
* Ignoring ransomware that may affect synchronized files.

### Quick example

If a laptop fails, a cloud backup can restore documents onto a replacement computer. A synchronized folder may restore only the files that were included and not the entire system.

### Recall questions

1. Why is an off-site backup safer than only a nearby external drive?
2. What feature distinguishes backup from simple synchronization?
3. Why should restoration be tested before an emergency?
4. How could ransomware affect cloud-synchronized files?
5. Which service features would you inspect before calling it a backup solution?

---

## 34. Backup Strategy

### Core idea

Reliable protection requires multiple copies stored in more than one location.

### Mental model

A useful general model is:

```text
Original + Local backup + Off-site backup
```

### Relationship between methods

| Copy            | Strength            | Weakness                                  |
| --------------- | ------------------- | ----------------------------------------- |
| Original device | Immediate access    | Can fail or be lost                       |
| External drive  | Fast local recovery | Can share the same physical disaster      |
| Cloud backup    | Off-site protection | Depends on provider, account, and network |

### Why multiple methods matter

Each backup type protects against different failures.

* External drives help with fast recovery.
* Cloud backups protect against local disasters.
* Version history protects against accidental changes or deletion.

### Rule

A backup is only useful if it can be restored.

### Practical checks

* Does the backup run automatically?
* Is the latest data included?
* Can older versions be recovered?
* Is the backup account secure?
* Has restoration been tested?
* Is at least one copy stored elsewhere?

### Common mistakes

* Connecting an external backup drive permanently without protection.
* Forgetting to back up newly created folders.
* Assuming one backup is sufficient.
* Discovering during an emergency that the backup is incomplete.
* Protecting data but not protecting account recovery credentials.

### Quick example

A student keeps current project files on a laptop, an automatic local copy on an external drive, and an encrypted cloud backup. Failure of any one location does not destroy every copy.

### Recall questions

1. Why do local and remote backups protect against different threats?
2. What does it mean to test a backup?
3. Which failure remains if both the computer and backup drive are stored together?
4. How does version history protect against user mistakes?
5. Design a backup strategy for an important semester project.

---

# Physical Computer Care

## 35. Why Cleaning Matters

### Core idea

Cleaning protects both appearance and function.

Dust and debris can affect:

* keyboard operation;
* mouse movement;
* airflow;
* cooling;
* long-term hardware reliability.

### Cause-and-effect model

```text
Dust buildup → Restricted airflow → Higher temperature → Reduced stability or component life
```

### General safety rule

Before cleaning:

* shut down the equipment;
* unplug it when appropriate;
* follow device-specific instructions.

### Why it matters

Cleaning involves both electrical and mechanical risk. Incorrect liquids or methods can damage:

* circuits;
* coatings;
* sensors;
* moving parts.

### Common mistakes

* Cleaning powered equipment.
* Spraying liquid directly onto a device.
* Using strong chemicals.
* Blocking vents after cleaning.
* Treating every surface with the same cleaning method.

### Quick example

Dust inside a keyboard may stop a key from moving correctly, while dust in a ventilation opening may increase internal temperature. The same contaminant causes different failures in different locations.

### Recall questions

1. How can dust affect both input devices and internal components?
2. Why should devices normally be powered off before cleaning?
3. Why is one cleaning method unsuitable for every component?
4. Which symptoms could indicate restricted airflow?

---

## 36. Cleaning a Keyboard

### Core idea

Keyboard cleaning should remove debris without allowing liquid to enter the electronics.

### Basic cleaning process

1. Shut down or disconnect the keyboard.
2. Turn it upside down.
3. Gently shake loose debris.
4. Use compressed air between keys.
5. Wipe surfaces with a lightly moistened cloth.

### Liquid rule

Never pour cleaning liquid directly onto the keyboard.

Instead:

* apply a small amount to the cloth;
* keep the cloth damp, not dripping.

### Spill response

For an accidental spill:

1. shut down the computer immediately;
2. disconnect the keyboard if possible;
3. turn it upside down;
4. allow liquid to drain;
5. let it dry completely before reconnecting.

### Exception and caution

The source describes rinsing some sticky, separate keyboards and drying them for two days. This is not safe for every keyboard, especially:

* laptop keyboards;
* wireless keyboards with batteries;
* backlit keyboards;
* keyboards with integrated electronics.

Manufacturer guidance should take priority.

### Prevention

The safest spill strategy is keeping drinks away from the computer area.

### Common mistakes

* Reconnecting a keyboard before it is fully dry.
* Using excessive liquid.
* Spraying cleaner directly between keys.
* Applying aggressive force while shaking.
* Treating a laptop keyboard like a detachable keyboard.

### Quick example

A detachable USB keyboard may be replaceable after severe liquid damage. A laptop spill may reach the motherboard, so immediate shutdown is more important.

### Recall questions

1. Why should cleaner be applied to the cloth rather than the keyboard?
2. Why is a laptop keyboard spill more dangerous?
3. What is the first action after a serious spill?
4. Why should generic rinsing advice not be applied to every keyboard?
5. How does prevention reduce both equipment and data risk?

---

## 37. Cleaning a Mouse

### Core idea

The cleaning method depends on whether the mouse is optical or mechanical.

### Optical mouse

An optical mouse uses a light-based sensor underneath.

Cleaning needs are usually limited to:

* removing dust from the sensor area;
* wiping the outer surface;
* checking the surface on which it is used.

### Mechanical mouse

An older mechanical mouse uses a tracking ball and moving rollers.

Cleaning may require:

1. unplugging the mouse;
2. removing the tracking ball;
3. wiping the ball;
4. removing dirt from internal rollers;
5. drying parts before reassembly.

### Mental model

```text
Optical mouse → Sensor must see movement clearly
Mechanical mouse → Moving parts must rotate freely
```

### Common mistakes

* Touching or scratching the optical sensor.
* Using too much liquid.
* Cleaning a mechanical mouse while connected.
* Reassembling damp parts.
* Assuming cursor problems always come from software.

### Quick example

A small piece of dust near an optical sensor can cause irregular pointer movement even when the mouse buttons work normally.

### Recall questions

1. Why does an optical mouse require less internal cleaning?
2. What symptoms can contamination near the sensor produce?
3. Why should a mechanical mouse be unplugged before opening it?
4. How would you distinguish a dirty sensor from a failed USB connection?

---

## 38. Cleaning a Monitor

### Core idea

Monitor screens require gentle cleaning because liquids and chemicals can damage internal components or surface coatings.

### Safe method

1. Turn off and unplug the monitor.
2. Use a soft, lint-free cloth.
3. Lightly moisten the cloth with water or an approved screen cleaner.
4. Wipe gently without pressing hard.

### Rules

* Do not spray directly onto the screen.
* Do not allow liquid to run toward the edges.
* Do not use ordinary glass cleaner unless the manufacturer permits it.
* Do not use abrasive cloths or paper that may scratch the surface.

### Why glass cleaner can be harmful

Some screens contain:

* anti-glare coatings;
* protective surface treatments.

Strong cleaners may damage these layers.

### Screen vs housing

The display surface and outer casing may require different cleaning methods.

A product safe for the casing may still be unsafe for the screen.

### Common mistakes

* Spraying cleaner directly on the display.
* Pressing strongly on an LCD panel.
* Using ammonia-based glass cleaner.
* Cleaning while the screen is warm and powered.
* Using the same cloth after it has collected abrasive dirt.

### Quick example

Liquid sprayed directly onto a monitor can run through the bezel and reach internal circuits even when the screen surface appears sealed.

### Recall questions

1. Why should liquid be applied to the cloth?
2. How can glass cleaner damage a monitor without entering it?
3. Why should screen pressure be minimized?
4. Which part of a monitor may tolerate a different cleaner from the panel?

---

## 39. Cleaning the Case and Ventilation

### Core idea

The computer case must remain clean enough for cooling air to move freely.

### Suitable tools

* lint-free or microfiber cloth;
* compressed air;
* mild cleaner applied to a cloth for suitable external surfaces.

### Ventilation care

Inspect:

* air intake openings;
* exhaust openings;
* fan grilles;
* surrounding space.

### Airflow mental model

```text
Cool air enters → Components release heat → Warm air exits
```

Blocking either side reduces cooling.

### Placement rules

Avoid placing the computer:

* directly against a wall;
* inside a closed compartment;
* beneath stacks of paper;
* on surfaces that block lower vents;
* near heavy dust sources.

### Compressed-air caution

When cleaning internal or fan areas:

* power off and unplug the computer;
* use short bursts;
* follow manufacturer guidance;
* avoid forcing fans to spin excessively.

### Common mistakes

* Treating fan noise as only an annoyance.
* Cleaning visible surfaces but ignoring vents.
* Placing the case inside a closed desk cabinet.
* Allowing paper or fabric to cover openings.
* Spraying cleaning liquid through ventilation holes.

### Quick example

A computer may become slow during heavy use because blocked vents cause overheating, which may trigger automatic performance reduction.

### Recall questions

1. Why can blocked airflow reduce performance before causing failure?
2. Which placements are most likely to obstruct ventilation?
3. Why should cleaner never be sprayed through vents?
4. How would you distinguish an airflow problem from low storage space?
5. Why can fan noise increase when dust accumulates?

---

## 40. Physical Maintenance Model

### Core idea

Good physical maintenance combines:

* regular cleaning;
* safe placement;
* airflow protection;
* spill prevention;
* cable safety.

### Mental model

```text
Cleanliness + Cooling + Safe handling = Longer reliable operation
```

### Relationships

| Practice                     | Prevents or reduces             |
| ---------------------------- | ------------------------------- |
| Removing dust                | Heat buildup and input problems |
| Keeping liquids away         | Electrical damage               |
| Leaving ventilation space    | Overheating                     |
| Managing cables              | Tripping and disconnection      |
| Powering off before cleaning | Electrical and mechanical risk  |

### Important limitation

Cleaning cannot repair:

* worn components;
* failed electronics;
* corrupted software;
* serious liquid damage.

It is preventive maintenance, not a universal repair method.

### Common mistakes

* Cleaning only after failure appears.
* Believing more aggressive cleaning is more effective.
* Ignoring the manufacturer’s instructions.
* Assuming software maintenance can solve physical overheating.
* Assuming visible cleanliness means internal airflow is healthy.

### Quick example

A computer that is externally clean may still overheat if its internal heat sink and fan are clogged with dust.

### Recall questions

1. Why is cleaning mainly preventive rather than corrective?
2. What maintenance actions reduce overheating risk?
3. Why can a visually clean computer still have a cooling problem?
4. Which physical risks are caused by poor cable management?
5. Create a safe monthly computer-cleaning checklist.

# Computer Protection and Maintenance

## 41. Malware

### Core idea

**Malware** is software designed to:

* damage a computer;
* disrupt its operation;
* steal information;
* gain unauthorized access.

### Common categories

| Type                     | General behaviour                                       |
| ------------------------ | ------------------------------------------------------- |
| Virus                    | Attaches to files or programs and spreads when executed |
| Spyware                  | Secretly collects information                           |
| Trojan horse             | Pretends to be legitimate software                      |
| Other malicious software | Performs unauthorized or harmful actions                |

### Mental model

Malware often succeeds through two stages:

```text
Entry → Execution → Harmful action
```

Possible entry points include:

* unsafe downloads;
* malicious attachments;
* deceptive websites;
* unpatched software;
* infected external devices.

### Important fact

No mainstream computer platform is completely immune to malware.

Risk can affect:

* Windows computers;
* Macs;
* Chromebooks;
* mobile devices.

The types and frequency of threats may differ across platforms.

### Why it matters

Malware may cause:

* file loss;
* identity theft;
* account compromise;
* reduced performance;
* unauthorized monitoring;
* loss of access to the computer.

### Common mistakes

* Assuming malware always produces obvious symptoms.
* Believing only Windows computers can be infected.
* Downloading software from unknown sources.
* Disabling security warnings without understanding them.
* Treating every slow computer as malware-infected.

### Quick example

A fake utility appears useful but secretly records account information. Its visible purpose hides its real behaviour, making it a Trojan horse.

### Recall questions

1. Why can malware remain unnoticed for a long time?
2. How does a Trojan horse differ from software that openly appears malicious?
3. Which stages must occur before downloaded malware can cause harm?
4. What signs would justify investigating a possible malware infection?

---

## 42. Antivirus Software

### Core idea

Antivirus software attempts to:

* prevent known threats from running;
* scan files and programs;
* detect suspicious behaviour;
* isolate or remove harmful software.

### Mental model

Antivirus is one defensive layer, not complete protection.

```text
Safe behaviour
+ Updated software
+ Antivirus
+ Backups
= Stronger protection
```

### How it works

Antivirus tools may use:

* known threat signatures;
* behavioural analysis;
* reputation information;
* real-time file scanning;
* scheduled system scans.

### Important limitation

No antivirus system detects every threat.

Reasons include:

* new malware may not yet be recognized;
* attackers modify existing malware;
* users may approve dangerous actions;
* legitimate-looking software can hide malicious functions.

### Protection practices

* Keep antivirus protection enabled.
* Install security updates.
* Scan suspicious files.
* Avoid unknown downloads.
* Do not bypass warnings casually.
* Keep recoverable backups.

### Built-in protection

Some operating systems include security tools, reducing the need to install a separate antivirus product for basic protection.

The effectiveness still depends on:

* correct configuration;
* regular updates;
* user behaviour.

### Common mistakes

* Installing several real-time antivirus products together.
* Assuming antivirus makes unsafe downloads harmless.
* Ignoring detection alerts.
* Using expired or outdated threat definitions.
* Removing a suspicious file without checking whether accounts were compromised.

### Quick example

Antivirus may block a known malicious attachment, but it may not stop a user from voluntarily entering a password into a fraudulent website.

### Recall questions

1. Why is antivirus only one layer of security?
2. How can malware bypass signature-based detection?
3. Why may multiple real-time antivirus products cause problems?
4. What additional actions are needed after spyware steals a password?

---

## 43. Software Updates

### Core idea

Updates fix problems and reduce known security weaknesses in software.

### Mental model

A discovered vulnerability creates a race:

```text
Weakness discovered
        ↓
Developer releases fix
        ↓
User installs update
```

Until the update is installed, the system may remain exposed.

### Software that should be updated

* operating system;
* web browser;
* antivirus tools;
* installed applications;
* device software where applicable.

### Why updates matter

Updates may provide:

* security patches;
* stability improvements;
* bug fixes;
* compatibility changes;
* performance improvements.

### Important distinction

An update is not always a feature upgrade.

| Update type          | Main purpose                          |
| -------------------- | ------------------------------------- |
| Security update      | Fixes vulnerabilities                 |
| Bug-fix update       | Corrects errors                       |
| Feature update       | Adds or changes capabilities          |
| Compatibility update | Supports changed hardware or software |

### Trade-off

Updates improve safety but may occasionally introduce:

* compatibility problems;
* changed interfaces;
* new bugs;
* increased resource requirements.

For most users, delaying security updates indefinitely creates greater risk.

### Common mistakes

* Ignoring update notifications repeatedly.
* Updating the antivirus but not the browser or OS.
* Downloading “updates” from pop-up advertisements.
* Assuming an old application remains safe because it still works.
* Interrupting an important system update by removing power.

### Quick example

A browser vulnerability may allow a malicious website to exploit the computer. Updating the browser closes the known weakness.

### Recall questions

1. Why does a working application still need updates?
2. How does delaying an update increase the attack window?
3. Why should updates come from the application or official provider?
4. When might an organization test updates before deploying them widely?

---

## 44. Storage Cleanup

### Core idea

Temporary and unnecessary files consume storage and can make storage management harder.

### Windows tools mentioned in the material

* Storage Sense;
* Disk Cleanup on older versions.

### How cleanup works

A cleanup tool may identify:

* temporary files;
* cached data;
* old system files;
* recycle-bin contents;
* files that are no longer needed.

### Mental model

Storage cleanup creates capacity; it does not directly increase computing power.

```text
Remove unnecessary files → More free storage
```

### Why free space matters

Very low free storage can interfere with:

* application updates;
* operating-system updates;
* temporary file creation;
* virtual memory;
* file downloads.

### Important caution

Automatic cleanup rules should be reviewed because they may remove:

* old downloads;
* recycle-bin files;
* temporary data still needed for recovery.

### Common mistakes

* Deleting unfamiliar system files manually.
* Treating cleanup as a replacement for adding storage.
* Assuming every large file is unnecessary.
* Emptying the recycle bin before confirming its contents.
* Believing cleanup removes malware automatically.

### Quick example

Deleting temporary installation files may recover space, but deleting an active project folder would cause data loss rather than useful cleanup.

### Recall questions

1. Why can extremely low free space affect normal system operation?
2. What should be reviewed before enabling automatic deletion?
3. Why does deleting files not improve CPU performance directly?
4. How would you decide whether a large file is safe to remove?

---

## 45. Drive Optimization

### Core idea

Storage optimization depends on the type of drive.

### Mechanical hard drives

Files on an HDD may become divided into pieces stored in different physical locations.

Defragmentation rearranges related data to reduce mechanical movement during access.

### Mental model

For an HDD:

```text
Scattered file pieces
→ More head movement
→ Slower retrieval
```

Defragmentation places pieces more efficiently.

### Solid-state drives

SSDs have no mechanical read head, so physical file placement affects them differently.

They use different optimization methods and generally should not be treated like traditional hard drives.

### Rule

Use the operating system’s built-in **Optimize Drives** function rather than manually applying the same process to every drive.

### Why it matters

Correct optimization can help maintain storage performance, while unnecessary manual operations may provide little benefit.

### Common mistakes

* Assuming every drive requires traditional defragmentation.
* Installing aggressive third-party optimization tools unnecessarily.
* Expecting defragmentation to fix malware or hardware failure.
* Interrupting optimization by forcing shutdown.
* Confusing file organization in folders with physical data organization.

### Quick example

Moving a document into a different visible folder changes its logical organization. Defragmentation concerns how file pieces are arranged internally on a mechanical disk.

### Recall questions

1. Why does fragmentation affect HDDs more than SSDs?
2. How does logical folder organization differ from physical storage layout?
3. Why should built-in optimization tools identify the drive type?
4. Which performance problems would defragmentation not solve?

---

## 46. Regular Backups

### Core idea

Security tools attempt to prevent damage; backups support recovery after prevention fails.

### Mental model

```text
Protection reduces probability.
Backup reduces consequence.
```

### What backups protect against

* hardware failure;
* accidental deletion;
* theft;
* severe malware;
* physical damage;
* corrupted files.

### Backup options in the material

| Method                   | Main advantage                  | Main weakness                        |
| ------------------------ | ------------------------------- | ------------------------------------ |
| External drive           | Fast local copying and recovery | Can be lost with the computer        |
| Built-in backup software | Easier automation               | Depends on configured destination    |
| Online backup            | Off-site protection             | May require fees and internet access |

### Important rule

Back up regularly enough that losing changes since the last backup would be acceptable.

### Common mistakes

* Creating one backup and never updating it.
* Keeping the only backup beside the computer.
* Backing up files without checking restoration.
* Leaving an external drive permanently exposed to malware.
* Assuming antivirus eliminates the need for backups.

### Quick example

If a project changes every day but is backed up once a month, a failure could erase nearly one month of work.

### Recall questions

1. How do backups and antivirus solve different problems?
2. How should backup frequency relate to how often files change?
3. Why can a permanently connected backup drive remain vulnerable?
4. What recovery risk remains when the backup is stored beside the computer?

---

# Ergonomics

## 47. Ergonomic Workspace

### Core idea

**Ergonomics** adapts the workspace to the user so tasks can be performed with less strain.

### Mental model

The body should remain close to a neutral position:

```text
Neutral posture + Appropriate distance + Regular movement
→ Lower strain
```

### Problems ergonomics aims to reduce

* eye strain;
* wrist discomfort;
* neck pain;
* back pain;
* fatigue;
* repetitive strain.

### Cause and effect

Poor positioning may create small stresses that accumulate over time.

```text
Awkward posture
+ Repetition
+ Long duration
= Increased discomfort or injury risk
```

### Why it matters

A setup can feel acceptable briefly but become harmful when used for many hours.

### Common mistakes

* Waiting for pain before adjusting the workspace.
* Copying another person’s setup exactly.
* Assuming expensive equipment automatically creates good ergonomics.
* Maintaining one posture all day, even if it initially feels comfortable.

### Quick example

A monitor placed too low may cause repeated downward neck bending. Raising it can reduce the need to hold that posture.

### Recall questions

1. Why can a tolerable posture become harmful over time?
2. How do repetition and duration amplify a small ergonomic problem?
3. Why must ergonomic settings be adjusted to the individual?
4. Which workspace features would you inspect after recurring neck pain?

---

## 48. Keyboard and Wrist Position

### Core idea

The wrists should remain straight and relaxed while typing.

### Mental model

```text
Forearm → Wrist → Hand
```

These should form a relatively neutral line rather than a sharp bend.

### Positioning principles

* Keep elbows near the body.
* Avoid bending wrists upward or sideways.
* Use a comfortable keyboard height.
* Type with relaxed hands.
* Avoid resting heavy pressure on the wrists while actively typing.

### Chair relationship

Chair height affects keyboard position.

If the chair is too low:

* wrists may bend upward;
* shoulders may rise.

If the chair is too high:

* feet may lose support;
* wrists may bend downward.

### Support equipment

Possible aids include:

* adjustable chair;
* ergonomic keyboard;
* keyboard tray;
* wrist support;
* footrest.

Equipment should correct a specific problem rather than being added automatically.

### Common mistakes

* Raising the chair without supporting the feet.
* Resting the wrists on a hard edge.
* Keeping the keyboard too far away.
* Assuming wrist pain should be ignored.
* Using a wrist rest to hold the wrists rigidly while typing.

### Quick example

If raising the chair aligns the wrists but leaves the feet hanging, a footrest can support the lower body without sacrificing keyboard position.

### Recall questions

1. How can chair height affect wrist position?
2. Why may a footrest become necessary after adjusting the chair?
3. What is the difference between supporting the wrists and pressing them down?
4. Which adjustment should be tested before buying a new keyboard?

---

## 49. Sitting Posture

### Core idea

A chair should support a natural, comfortable posture without forcing excessive stiffness or slouching.

### Useful positioning

* Sit back in the chair.
* Support the lower back.
* Keep shoulders relaxed.
* Keep feet supported.
* Avoid leaning toward the screen.
* Adjust the chair rather than adapting the body to a poor setting.

### Mental model

Good posture is supported, not rigid.

```text
Balanced support ≠ Sitting perfectly still
```

### Adjustable chair features

A chair may allow changes to:

* seat height;
* backrest angle;
* lumbar support;
* armrest position;
* seat depth.

### Trade-off

Too little support encourages slouching.

Too much forced correction may create tension.

### Common mistakes

* Sitting on the front edge for long periods.
* Setting armrests high enough to raise the shoulders.
* Leaning toward a distant or hard-to-read screen.
* Sitting extremely upright without back support.
* Treating posture as fixed rather than changing position periodically.

### Quick example

A person may lean forward because the text is too small. Increasing text size may address the real cause more effectively than repeatedly correcting posture.

### Recall questions

1. Why is rigid posture not necessarily healthy posture?
2. How can screen readability affect sitting position?
3. What chair settings influence lower-back support?
4. Why should posture vary during the day?

---

## 50. Monitor Position

### Core idea

The monitor should be positioned so the user can view it without leaning, squinting, or bending the neck excessively.

### Recommended relationship

The material suggests:

* roughly an arm’s length away;
* approximately 20–40 inches;
* top of the screen around eye level.

Exact placement depends on:

* screen size;
* text size;
* vision;
* seating position;
* type of work.

### Mental model

```text
Readable distance + Comfortable height
→ Neutral head and neck position
```

### Monitor height

If the screen is too low:

* the head may tilt downward.

If it is too high:

* the neck may tilt upward;
* the eyes may remain overly open and dry more quickly.

### Multiple monitors

The most frequently used monitor should generally be placed most directly in front of the user.

### Common mistakes

* Applying one exact distance to every monitor size.
* Placing the monitor off-centre while using it continuously.
* Raising the monitor but leaving the keyboard too high.
* Moving closer instead of increasing text size.
* Ignoring glare from windows or lights.

### Quick example

A large monitor may need to be placed farther away than a small laptop screen while still remaining readable.

### Recall questions

1. Why is “arm’s length” only an approximate rule?
2. How does monitor height affect neck posture?
3. Where should the main monitor be placed in a two-monitor setup?
4. What alternatives exist to leaning closer to small text?

---

## 51. Laptop Ergonomics

### Core idea

A laptop combines the screen and keyboard, creating an ergonomic conflict.

### Cause-and-effect model

* A comfortable keyboard height often places the screen too low.
* A comfortable screen height often places the keyboard too high.

```text
Fixed screen–keyboard connection
→ One part is usually poorly positioned
```

### Practical solution

For extended use:

* raise the laptop screen;
* connect an external keyboard;
* connect an external mouse.

This separates display position from input position.

### Trade-off

| Direct laptop use        | External setup                     |
| ------------------------ | ---------------------------------- |
| Portable and simple      | More ergonomic for long sessions   |
| Fewer accessories        | Requires extra space and equipment |
| Suitable for shorter use | Better independent positioning     |

### Common mistakes

* Working for hours with the laptop on the lap.
* Raising the laptop without adding an external keyboard.
* Using a touchpad continuously despite discomfort.
* Treating portability as more important than every long-term posture issue.

### Quick example

Placing a laptop on a stand raises the screen, but typing on the raised keyboard would strain the arms. An external keyboard completes the setup.

### Recall questions

1. What ergonomic conflict is built into laptop design?
2. Why is a laptop stand alone often insufficient?
3. When is direct laptop use reasonable?
4. How can portability and ergonomic comfort be combined?

---

## 52. Screen Brightness and Colour

### Core idea

The screen should not appear dramatically brighter or darker than the surrounding environment.

### Mental model

The eyes should not repeatedly adapt between extreme brightness levels.

```text
Screen brightness ≈ Environmental brightness
```

### Signs of poor brightness

Too bright:

* screen feels like a light source shining at the eyes;
* glare becomes distracting;
* eye fatigue may increase.

Too dim:

* text appears unclear;
* the user may lean closer;
* contrast may be difficult to perceive.

### Reduced blue-light modes

Some systems offer modes such as:

* Night Mode;
* Night Shift;
* warmer colour settings.

These reduce the display’s cooler blue appearance.

### Important limitation

A warmer display is not a replacement for:

* proper brightness;
* good posture;
* breaks;
* suitable room lighting;
* adequate sleep.

### Common mistakes

* Keeping maximum brightness in a dark room.
* Making the display too dim to read comfortably.
* Assuming blue-light mode solves every form of eye strain.
* Ignoring reflections and glare.
* Using automatic brightness without checking whether it is comfortable.

### Quick example

Lowering screen brightness may not solve glare caused by a window behind the user. Repositioning the monitor may be more effective.

### Recall questions

1. Why should screen brightness relate to environmental light?
2. How can a screen that is too dim affect posture?
3. What problems are not solved by a warmer colour mode?
4. How would you distinguish excessive brightness from reflected glare?

---

## 53. Breaks and the 20-20-20 Rule

### Core idea

Frequent short breaks reduce continuous visual and physical strain.

### 20-20-20 rule

Every:

* **20 minutes**;
* look at something about **20 feet away**;
* for approximately **20 seconds**.

### Mental model

Close screen work keeps the eyes focused at one distance.

Looking farther away changes the focusing demand.

### Movement breaks

The material also recommends getting up and moving approximately every hour.

Possible actions:

* walk briefly;
* stretch;
* change posture;
* complete a task away from the desk.

### Important distinction

The 20-20-20 rule mainly targets visual strain.

Movement breaks address:

* prolonged sitting;
* muscular stiffness;
* lack of circulation;
* static posture.

### Exception

Standing desks do not remove the need for movement. Standing motionless for long periods is still static behaviour.

### Common mistakes

* Taking one long break after several uninterrupted hours.
* Looking at a phone during an eye break.
* Standing all day without changing position.
* Waiting for severe discomfort before moving.
* Treating reminders as useful while repeatedly dismissing them.

### Quick example

Looking from a monitor to a phone does not meaningfully change viewing distance, so it is not an effective distance break.

### Recall questions

1. What problem does the 20-20-20 rule mainly address?
2. Why is checking a phone not an ideal eye break?
3. Why do standing-desk users still need movement?
4. How are movement breaks different from visual breaks?

---

## 54. Workspace Clutter and Safety

### Core idea

Clutter can create both ergonomic and physical hazards.

### Possible problems

* restricted movement;
* awkward reaching;
* blocked ventilation;
* falling objects;
* damaged cables;
* tripping hazards.

### Cable management

Loose power cables can:

* catch a foot;
* disconnect equipment;
* pull a device from the desk;
* become damaged.

### Mental model

A clear workspace reduces unnecessary movement and accidental force.

### Practical rules

* Store unused supplies elsewhere.
* Keep frequently used items within easy reach.
* Route cables away from walking areas.
* Do not cover ventilation openings.
* Avoid unstable stacks near equipment.

### Common mistakes

* Organizing for appearance while leaving cables unsafe.
* Placing frequently used items far away.
* Running cables tightly around sharp edges.
* Hiding the computer case in a sealed compartment.
* Allowing paperwork to cover vents.

### Quick example

A printer placed too far away may require repeated twisting. Moving it within a safer reaching range reduces strain.

### Recall questions

1. How can clutter create an ergonomic problem?
2. Which cable arrangements present a tripping risk?
3. Why should frequently used objects be placed within easy reach?
4. How can attempts to hide equipment reduce airflow?

---

# Safer Web Browsing

## 55. Browser Security

### Core idea

Web browsers provide security features, but users must still evaluate websites and warnings.

### Browser protections may include

* malicious-site warnings;
* download warnings;
* secure-connection indicators;
* automatic updates;
* domain highlighting;
* permission controls.

### Mental model

Browser security works as a partnership:

```text
Browser protection + User verification
```

### Important limitation

A browser can identify many known risks but cannot guarantee that every site is honest or safe.

### Common mistakes

* Ignoring browser warnings.
* Assuming no warning means no danger.
* Granting every requested permission.
* Using an outdated browser.
* Trusting a website based only on professional appearance.

### Quick example

A fraudulent website may not contain known malware, so the browser might not block it. The user must still verify the address and purpose.

### Recall questions

1. Why can a harmful website remain accessible without a warning?
2. What decisions remain the user’s responsibility?
3. How do browser updates improve security?
4. Why is visual design weak evidence of legitimacy?

---

## 56. Domain Names

### Core idea

The domain name identifies the main website being visited and should be checked before entering sensitive information.

### Mental model

Attackers often create addresses that look almost correct.

Examples of deception may involve:

* misspellings;
* extra words;
* misleading subdomains;
* similar-looking characters;
* unfamiliar domain endings.

### Address structure

A web address may contain several parts:

```text
https://accounts.example.com/login
```

The important registered domain is based around:

```text
example.com
```

An attacker could instead use something like:

```text
example.account-check.com
```

Here, the actual domain belongs to `account-check.com`, not `example.com`.

### Why browsers emphasize domains

Highlighting the main domain makes deceptive addresses easier to detect.

### Rule

Read the address from the domain outward rather than trusting the first familiar word.

### Common mistakes

* Checking only the beginning of the address.
* Assuming a familiar logo proves identity.
* Ignoring minor spelling differences.
* Trusting a link because it arrived in an expected email.
* Confusing a subdomain with the registered domain.

### Quick example

`bank.security-example.com` belongs to `security-example.com`, not necessarily to the bank named at the beginning.

### Recall questions

1. Why is the first familiar word in an address unreliable?
2. How can a subdomain be used deceptively?
3. What part of an address should be verified before signing in?
4. Why can a correct-looking logo appear on a fraudulent site?

---

## 57. Secure Connections and the Padlock

### Core idea

A padlock generally indicates that the connection between the browser and website is encrypted.

### What encryption helps protect

It reduces the ability of third parties on the network to read data in transit, such as:

* passwords;
* messages;
* payment information.

### Mental model

```text
Padlock = Secure connection to this domain
Padlock ≠ Proof that the domain is trustworthy
```

### Important distinction

The padlock helps answer:

> Is the connection encrypted?

It does not fully answer:

> Is this website honest?

A fraudulent site can also use encryption.

### Why it matters

Before entering sensitive data, verify both:

1. the correct domain;
2. a secure connection.

### Common mistakes

* Treating the padlock as proof of legitimacy.
* Ignoring the domain because HTTPS is present.
* Assuming sites without sensitive data require identical security checks.
* Entering personal data after dismissing certificate warnings.

### Quick example

An encrypted phishing site can securely transmit a stolen password to the attacker. Encryption protects the transmission, not the user’s decision.

### Recall questions

1. What does the padlock actually confirm?
2. Why can a phishing site still use HTTPS?
3. Which two checks should be made before entering banking information?
4. Why should certificate warnings not be dismissed casually?

---

## 58. Browser Updates

### Core idea

An updated browser includes newer protections against known security weaknesses.

### Update process

Browsers may:

* update automatically;
* notify the user;
* provide a manual update check.

### Why browser updates are important

Browsers process complex, untrusted web content.

A vulnerability may be triggered simply by:

* opening a page;
* loading media;
* running scripts;
* processing a download.

### Common mistakes

* Ignoring restart requests after an update.
* Assuming automatic updates always completed.
* Using an unsupported browser version.
* Downloading a fake browser update from a website advertisement.
* Keeping unnecessary browser extensions updated poorly or not at all.

### Quick example

A browser may download an update in the background but continue using the older version until it is restarted.

### Recall questions

1. Why are browsers frequent targets for attackers?
2. Why may restarting be required after an update?
3. How can a fake update message itself become a threat?
4. What additional browser components may also need updates?

---

# Email Safety

## 59. Spam

### Core idea

**Spam** is unsolicited bulk email sent to many recipients.

### Possible purposes

* advertising;
* fraud;
* malware delivery;
* phishing;
* confirmation that an address is active.

### Why spam is difficult to eliminate

Senders can:

* distribute messages at large scale;
* hide or change identities;
* create new sending accounts;
* modify message wording.

### Spam filters

Email providers analyze incoming messages and move suspicious ones into a spam folder.

### Important limitation

Filtering is imperfect.

It may produce:

| Error          | Meaning                         |
| -------------- | ------------------------------- |
| False positive | Legitimate email marked as spam |
| False negative | Spam reaches the inbox          |

### User actions

* Mark missed spam correctly.
* Check the spam folder periodically.
* Avoid replying to suspicious messages.
* Do not download unexpected attachments.
* Delete clear scams.

### Common mistakes

* Assuming every message in the inbox is safe.
* Assuming every message in spam is malicious.
* Replying to demand removal from an obvious scam.
* Opening attachments to discover what they contain.
* Marking unwanted legitimate subscriptions as fraud instead of unsubscribing safely.

### Quick example

A university email may be incorrectly filtered because it contains many links or was sent to a large mailing list.

### Recall questions

1. Why can legitimate mail enter the spam folder?
2. How does marking spam improve future filtering?
3. Why can replying to spam make the problem worse?
4. What should you verify before restoring a message from spam?

---

## 60. External Email Images

### Core idea

Images in emails may be loaded from a remote server, allowing the sender to observe that the message was opened.

### How tracking can work

1. The email contains a remotely hosted image.
2. The recipient opens the message.
3. The email client requests the image.
4. The sender’s server records the request.

### Possible information revealed

Depending on the system, the sender may infer:

* that the address is active;
* approximate opening time;
* repeated openings;
* limited device or network information.

### Mental model

```text
Remote image request → Sender receives a signal
```

### Protection

Some email services:

* block external images;
* ask before displaying them;
* load images through privacy-protecting systems.

### Important distinction

An embedded image stored directly inside the email does not require the same remote request as a remotely hosted image.

### Common mistakes

* Assuming every image is purely decorative.
* Loading images in obvious spam.
* Believing blocking images prevents every form of email tracking.
* Confusing an image tracker with malware infection.

### Quick example

A one-pixel image may be visually invisible but still notify the sender when it is requested.

### Recall questions

1. How can loading an image confirm that an email address is active?
2. Why might blocking remote images reduce tracking?
3. Does blocking images prevent every tracking method?
4. How is tracking through an image different from malware?

---

## 61. Phishing

### Core idea

**Phishing** is a deception technique that impersonates a trusted source to obtain information or trigger a harmful action.

### Common goals

* steal passwords;
* collect payment details;
* obtain personal information;
* install malware;
* persuade the victim to transfer money.

### Mental model

Phishing attacks trust rather than only technology.

```text
Trusted identity
+ Urgent message
+ Requested action
= Pressure to bypass verification
```

### Typical warning signs

* unexpected urgency;
* threats of account closure;
* requests to verify sensitive data;
* unfamiliar sender address;
* suspicious links;
* unusual attachments;
* spelling or formatting problems;
* requests that conflict with normal procedure.

### Important exception

Well-designed phishing may contain:

* correct spelling;
* familiar branding;
* believable account details;
* professional formatting.

Poor grammar is only one possible clue.

### Common mistakes

* Trusting the displayed sender name.
* Clicking before checking the address.
* Assuming a known company logo proves authenticity.
* Acting because the message creates fear or urgency.
* Believing phishing always looks amateurish.

### Quick example

An email claims that a bank account will be locked within one hour unless the user signs in through the included link.

The urgency is designed to prevent careful verification.

### Recall questions

1. Why does urgency make phishing more effective?
2. Which parts of an email can attackers imitate?
3. Why is correct grammar not proof of legitimacy?
4. How does phishing differ from malware?
5. Which verification steps would you use for an unexpected account warning?

---

## 62. Safer Link Handling

### Core idea

Sensitive accounts should be accessed through a trusted route rather than an unexpected email link.

### Safer alternatives

* type the known address manually;
* use a trusted bookmark;
* open the official application;
* contact the organization through verified details.

### Mental model

```text
Untrusted message → Do not use its verification path
```

The sender should not control both:

* the warning;
* the method used to verify the warning.

### Link inspection

Before opening a link:

* inspect the destination;
* check the domain;
* watch for misspellings;
* consider whether the message was expected.

### Important limitation

Manually typing a web address is safer only when the correct address is already known.

Search advertisements and copied addresses can still be misleading.

### Common mistakes

* Clicking because the visible link text looks correct.
* Calling a phone number included in a suspicious message.
* Using contact details supplied by the possible attacker.
* Signing in before noticing a domain mismatch.
* Assuming shortened links reveal their true destination.

### Quick example

Instead of selecting “Verify Account” in an email, open the bank’s official app independently and check for alerts there.

### Recall questions

1. Why should verification use an independent communication path?
2. How can visible link text differ from the actual destination?
3. Why might a phone number in a suspicious email be unsafe?
4. When is manually typing an address still insufficient?

---

## 63. Protecting Sensitive Information

### Core idea

A request for sensitive information should be evaluated based on context and verification, not familiarity.

### Sensitive information includes

* passwords;
* security codes;
* payment-card details;
* government identification numbers;
* account recovery answers;
* home address;
* private personal data.

### Rule

Legitimate organizations generally do not need users to send passwords through email.

### Verification questions

Before responding, ask:

* Was this request expected?
* Is this normal for the organization?
* Can the request be verified independently?
* Is the requested information necessary?
* Is the communication channel appropriate?

### Mental model

```text
Identity claim ≠ Verified identity
```

### Common mistakes

* Sending information because the sender knows personal details.
* Reusing passwords across accounts.
* Sharing one-time authentication codes.
* Believing partial account information proves the sender is genuine.
* Responding before contacting the organization independently.

### Quick example

A caller who knows the last four digits of a card may have obtained them from leaked data. That knowledge does not prove the caller represents the bank.

### Recall questions

1. Why is possession of personal details weak proof of identity?
2. Which information should never be shared through ordinary email?
3. How can a request be independently verified?
4. Why are one-time codes valuable to attackers?

---

# Digital Tracking

## 64. Online Tracking

### Core idea

Websites and advertisers collect signals about browsing activity to build profiles and personalize content or advertising.

### Mental model

A single observation reveals little.

Many connected observations reveal patterns.

```text
Small data points
+ Repeated collection
+ Identity matching
= Behavioural profile
```

### Data that may contribute

* websites visited;
* searches;
* clicked advertisements;
* account activity;
* device characteristics;
* approximate location;
* purchase interests.

### Why it matters

Profiles may influence:

* advertisements;
* recommendations;
* search results;
* content order;
* marketing decisions.

### Privacy concern

Users may not fully understand:

* what is collected;
* who receives it;
* how long it is retained;
* what conclusions are inferred.

### Common mistakes

* Assuming tracking only occurs after logging in.
* Believing one deleted cookie removes every profile.
* Treating all personalization as harmless.
* Assuming collected data is always accurate.
* Confusing anonymity with privacy.

### Quick example

Searching for shoes on one site may contribute to a profile later used to show shoe advertisements on unrelated sites.

### Recall questions

1. Why are many small observations more revealing together?
2. How can tracking occur without a direct purchase?
3. What is the difference between collected data and inferred data?
4. Why might a behavioural profile be inaccurate?

---

## 65. Tracking Cookies

### Core idea

A cookie is a small piece of data that a website asks the browser to store.

Cookies may support legitimate functions or tracking.

### Useful cookie functions

* keeping a user signed in;
* remembering preferences;
* retaining shopping-cart contents;
* maintaining a session.

### Tracking use

An advertising or analytics system may assign an identifier and recognize the browser during later visits.

### Mental model

```text
Cookie identifier → Browser recognized later
```

### First-party vs third-party relationship

| Type                 | General relationship                                     |
| -------------------- | -------------------------------------------------------- |
| First-party cookie   | Created in the context of the site being visited         |
| Third-party tracking | Associated with an outside service embedded across sites |

### Important distinction

Cookies usually identify a browser or session, not directly prove a real-world identity.

They may later be linked to identity through:

* account sign-in;
* email use;
* purchases;
* other matching methods.

### Common mistakes

* Treating all cookies as malicious.
* Assuming deleting cookies permanently prevents recognition.
* Believing private browsing makes the user invisible.
* Blocking cookies without considering effects on login or site functions.
* Assuming one device always has one stable identifier.

### Quick example

A shopping site uses one cookie to remember the cart. An advertising network may use another identifier across several participating sites.

### Recall questions

1. Why are some cookies necessary for normal website functions?
2. How can an anonymous browser identifier become linked to a person?
3. Why may deleting cookies not end tracking permanently?
4. What site functions may break when all cookies are blocked?

---

## 66. Cross-Device Matching

### Core idea

Companies may attempt to determine which phones, tablets, and computers belong to the same person.

### Direct matching

Devices can be linked through shared account use, such as:

* signing in to the same email account;
* using the same social-media account;
* sharing a service account.

### Probabilistic matching

A system may infer relationships using patterns such as:

* similar locations;
* network connections;
* browsing behaviour;
* timing;
* device characteristics.

### Mental model

```text
Same account → Strong direct link
Similar patterns → Inferred probable link
```

### Important distinction

Probabilistic matching is an estimate, not certainty.

This can produce:

* incorrect device associations;
* incorrect advertising assumptions;
* merged profiles involving several household members.

### Common mistakes

* Assuming devices are tracked independently.
* Treating inferred identity as always accurate.
* Believing logging out removes every relationship.
* Ignoring shared-device and shared-network effects.

### Quick example

Several devices using the same home network and visiting similar services may be inferred to belong to one person, even when they belong to different family members.

### Recall questions

1. How does direct account matching differ from probabilistic matching?
2. Why can shared households create incorrect profiles?
3. What makes probabilistic matching uncertain?
4. How can one login connect activity across devices?

---

## 67. Benefits and Costs of Personalization

### Core idea

Data collection can make services more relevant while reducing privacy and user control.

### Trade-off

| Possible benefit            | Possible cost                  |
| --------------------------- | ------------------------------ |
| Relevant recommendations    | Detailed behavioural profiling |
| Faster search results       | Filtered or narrowed exposure  |
| Personalized advertising    | Reduced privacy                |
| Remembered preferences      | Persistent identification      |
| Convenient cross-device use | Broader activity linkage       |

### Mental model

```text
More personalization usually requires more information.
```

### Important questions

Evaluate:

* What data is necessary?
* Is collection transparent?
* Can the user opt out?
* Is data shared?
* How long is it retained?
* What happens if the profile is wrong?

### Common mistakes

* Treating personalization as either entirely good or entirely harmful.
* Assuming recommendations are neutral.
* Giving access without reviewing permissions.
* Believing an opt-out stops all collection.
* Ignoring the effect of profiling on other people using the same device.

### Quick example

A video service may recommend useful content based on viewing history, but the same history reveals interests and habits.

### Recall questions

1. Why does personalization usually require data collection?
2. How can recommendations shape behaviour rather than only reflect it?
3. Which controls would make personalization more acceptable?
4. What harm can occur when a profile is inaccurate?

---

## 68. Limiting Tracking

### Core idea

Tracking can often be reduced, but complete prevention is difficult.

### Possible controls

* block or restrict third-party cookies;
* clear stored site data;
* review browser privacy settings;
* limit account sign-ins;
* restrict application permissions;
* use privacy-oriented browser features;
* avoid unnecessary personal information;
* reset advertising identifiers where supported.

### Mental model

Privacy protection reduces available signals.

It does not guarantee invisibility.

### Trade-offs

Stronger restrictions may affect:

* persistent logins;
* saved preferences;
* embedded media;
* shopping carts;
* website compatibility.

### Common mistakes

* Believing one setting stops every tracking method.
* Installing untrusted “privacy” extensions.
* Granting broad browser-extension permissions.
* Assuming private mode hides activity from websites, employers, or ISPs.
* Blocking cookies while remaining signed into major accounts.

### Quick example

Blocking third-party cookies may reduce some cross-site tracking, but signing into the same advertising platform on several sites can still link activity.

### Recall questions

1. Why can blocking cookies fail to prevent all tracking?
2. What functionality may be lost under strict privacy settings?
3. How can a privacy extension itself become a risk?
4. Why does staying signed in weaken some tracking protections?

# Windows Interface

## 69. Windows Desktop

### Core idea

The **desktop** is the main Windows workspace displayed after signing in.

### Main parts

| Element            | Purpose                                          |
| ------------------ | ------------------------------------------------ |
| Desktop background | Visual background, also called wallpaper         |
| Desktop icons      | Shortcuts to files, folders, or applications     |
| Taskbar            | Access to open and pinned applications           |
| Start button       | Opens applications, settings, and system options |
| Notification area  | Displays status icons and notifications          |

### Mental model

```text
Desktop = Main workspace
Taskbar = Navigation and status area
Start menu = Application and settings directory
```

### How it works

Applications and files generally open inside rectangular areas called **windows**.

The user can:

* open several windows;
* move them;
* resize them;
* switch between them;
* close them.

### Important distinction

The Windows operating system and an application window are not the same thing.

* **Windows** is the operating system.
* A **window** is a visual container used by an application or folder.

### Common mistakes

* Confusing the desktop with the entire operating system.
* Deleting a shortcut and assuming the original file was deleted.
* Filling the desktop with too many files.
* Assuming the desktop is the only location where files can be stored.

### Quick example

Double-clicking a document icon on the desktop opens the document in its associated application.

### Recall questions

1. How does the desktop differ from the taskbar?
2. Why may deleting a desktop icon not remove the application?
3. What problems can result from using the desktop as the main file-storage location?
4. Which interface element would you use to access system settings?

---

## 70. Taskbar

### Core idea

The taskbar provides quick access to applications and shows which applications are currently running.

### Common contents

* Start button;
* pinned application shortcuts;
* icons for open windows;
* system and network status;
* clock;
* notifications.

### Mental model

The taskbar combines two functions:

```text
Launch applications + Switch between running applications
```

### Pinned vs running applications

| State              | Meaning                                                |
| ------------------ | ------------------------------------------------------ |
| Pinned             | Shortcut remains available even when the app is closed |
| Running            | Application is currently open                          |
| Pinned and running | Shortcut also represents an active application         |

### Switching windows

When several windows are open, selecting an application icon on the taskbar can bring its window to the front.

An application may have multiple open windows.

### Why it matters

The taskbar reduces the need to:

* minimize every window;
* search for an application repeatedly;
* return to the desktop to switch tasks.

### Common mistakes

* Assuming a taskbar icon always means the application is running.
* Opening several copies of an application unintentionally.
* Confusing closing a window with removing its pinned shortcut.
* Overloading the taskbar with rarely used applications.

### Quick example

A browser and word processor are both open. Selecting the word-processor icon on the taskbar brings the document window into view.

### Recall questions

1. How does a pinned app differ from a running app?
2. Why can one taskbar icon represent several windows?
3. What happens to a pinned icon when its application closes?
4. How would you switch applications without closing either one?

---

## 71. Start Menu

### Core idea

The Start menu is a central location for finding:

* applications;
* files;
* settings;
* power controls.

### Mental model

```text
Start menu = System-wide launch and navigation point
```

### Typical uses

* launch an application;
* search the computer;
* open Settings;
* access user-account options;
* shut down or restart the computer.

### Version differences

The appearance and behaviour of the Start interface vary across Windows versions.

The source notes that Windows 8 used a full-screen Start screen, while other versions use a menu-style interface.

### Important rule

Interface layout can change, but the underlying purpose remains similar:

> Find and launch system resources.

### Common mistakes

* Searching the desktop manually instead of using system search.
* Using the power button on the computer case instead of the operating system’s shutdown command.
* Assuming an application is uninstalled because it is not pinned.
* Confusing the Start menu with the taskbar.

### Quick example

Typing the name of an application after opening Start can locate it even when no desktop or taskbar shortcut exists.

### Recall questions

1. Why is the Start menu more than an application list?
2. How can system search reduce navigation time?
3. Why should normal shutdown use the operating system’s command?
4. What remains consistent even when the Start interface changes between versions?

---

## 72. File Explorer

### Core idea

**File Explorer** is the Windows tool for viewing and managing files, folders, and storage locations.

### Mental model

```text
Storage device
└── Folders
    └── Subfolders
        └── Files
```

### Main operations

File Explorer allows users to:

* open files;
* create folders;
* move or copy items;
* rename items;
* delete items;
* search storage;
* inspect connected drives.

### Files vs folders

| Item   | Purpose                                         |
| ------ | ----------------------------------------------- |
| File   | Stores data, such as a document or image        |
| Folder | Organizes files and other folders               |
| Drive  | Represents a storage device or storage location |

### Why it matters

File Explorer exposes the computer’s hierarchical storage structure.

Understanding this structure helps prevent:

* lost files;
* duplicate copies;
* accidental deletion;
* saving to the wrong location.

### Common mistakes

* Confusing a file with the application that opens it.
* Saving files without checking the destination folder.
* Moving a file when intending to copy it.
* Creating many duplicate downloads.
* Deleting a folder without checking its contents.

### Quick example

A course assignment might be organized as:

```text
Documents
└── Semester 1
    └── Computer Basics
        └── Assignment.docx
```

### Recall questions

1. How does a folder differ from a file?
2. Why can the same file type open in different applications?
3. When should an item be copied rather than moved?
4. How would a clear folder hierarchy reduce accidental data loss?

---

## 73. Opening Files, Folders, and Programs

### Core idea

Windows commonly uses a **double-click** to open desktop items.

### Typical interaction rules

| Action         | Common result                                 |
| -------------- | --------------------------------------------- |
| Single-click   | Selects an item                               |
| Double-click   | Opens an item                                 |
| Right-click    | Displays available actions                    |
| Click and drag | Moves or selects an item depending on context |

### Mental model

```text
Select first → Act second
```

A single-click usually identifies the target without opening it.

### Important exception

Items on the taskbar and Start menu usually open with a single click.

The required action depends on the interface context.

### Common mistakes

* Double-clicking taskbar buttons and opening duplicate windows.
* Moving an icon accidentally while attempting to open it.
* Clicking repeatedly when an application is loading slowly.
* Assuming every interface uses the same click behaviour.

### Quick example

A folder icon on the desktop usually requires a double-click, while File Explorer on the taskbar usually requires one click.

### Recall questions

1. Why does Windows distinguish selection from opening?
2. Which interface locations commonly use a single click?
3. How can repeated clicks create duplicate application windows?
4. When is right-clicking more useful than double-clicking?

---

## 74. Moving Windows

### Core idea

An application window can usually be moved by dragging its title bar.

### How it works

1. Place the pointer on the title bar.
2. Hold the primary mouse button.
3. Move the mouse.
4. Release the button at the desired location.

### Mental model

```text
Title bar = Handle for the window
```

### Why it matters

Moving windows helps:

* compare information;
* expose content behind another window;
* organize multitasking;
* use multiple monitors.

### Important limitation

A maximized window usually fills the screen and may need to be restored before being freely repositioned.

### Common mistakes

* Dragging inside the document instead of the title bar.
* Accidentally selecting text while trying to move the window.
* Moving a window partly off-screen.
* Assuming the application has closed when its window is hidden behind another.

### Quick example

A browser window can be moved beside a notes application so information can be read and recorded simultaneously.

### Recall questions

1. Why is the title bar treated as a window handle?
2. Why might a maximized window resist normal movement?
3. How can window positioning improve comparison tasks?
4. What would you check if an open application appears to have disappeared?

---

## 75. Maximizing and Restoring Windows

### Core idea

A window can be enlarged to fill the available screen and later returned to its previous size.

### Window states

| State              | Meaning                                            |
| ------------------ | -------------------------------------------------- |
| Normal or restored | Window occupies part of the screen                 |
| Maximized          | Window fills most or all available workspace       |
| Minimized          | Window is hidden from the desktop but remains open |
| Closed             | Window or application session is ended             |

### Mental model

```text
Maximize = Use full workspace
Restore = Return to adjustable size
Minimize = Hide temporarily
Close = End the window
```

### Why it matters

Different states support different tasks:

* maximize for focused work;
* restore for side-by-side work;
* minimize to temporarily clear the screen;
* close when finished.

### Common mistakes

* Confusing minimize with close.
* Reopening an application that is only minimized.
* Closing unsaved work.
* Assuming maximizing increases application performance.

### Quick example

Minimizing a music player hides its window but may allow playback to continue because the application remains active.

### Recall questions

1. How does minimizing differ from closing?
2. When is restoring preferable to maximizing?
3. Why might an application continue working while minimized?
4. Which window action creates a risk of losing unsaved work?

---

## 76. Closing Windows

### Core idea

The **X button** normally closes the current window.

### Important distinction

Closing a window may:

* close only one document;
* close one browser tab or browser window;
* exit the entire application.

The result depends on the program.

### Unsaved changes

Applications may ask whether unsaved work should be:

* saved;
* discarded;
* left open by cancelling the close action.

### Mental model

```text
Close request → Check unsaved state → Save or discard → End window
```

### Common mistakes

* Selecting “Don’t Save” without reading the prompt.
* Assuming closing one window always exits the whole application.
* Turning off the computer to close an unresponsive program immediately.
* Confusing a close button with a minimize button.

### Quick example

Closing one document in a word processor may leave the application running with another document still open.

### Recall questions

1. Why can the same close button produce different results across applications?
2. What should be checked before discarding changes?
3. How can an application remain open after one window closes?
4. Why should forced shutdown not be the first response to a frozen window?

---

# macOS Interface

## 77. macOS Desktop

### Core idea

The macOS desktop is the main workspace and includes:

* desktop background;
* menu bar;
* Dock;
* files and folders placed on the desktop.

### Main parts

| Element  | Location                    | Purpose                          |
| -------- | --------------------------- | -------------------------------- |
| Menu bar | Top of screen               | Application and system commands  |
| Dock     | Usually bottom              | Application and folder shortcuts |
| Desktop  | Main area                   | Workspace and file placement     |
| Finder   | File-management application | Browse files and folders         |

### Mental model

```text
Menu bar = Commands
Dock = Launching and switching
Finder = File navigation
Desktop = Workspace
```

### Important difference from Windows

macOS generally places the active application’s menus in the fixed menu bar at the top of the screen rather than inside each application window.

### Common mistakes

* Looking inside the window for every application menu.
* Treating the Dock as the only way to open an application.
* Saving too many files directly on the desktop.
* Assuming Windows and macOS use identical close behaviour.

### Quick example

When Safari is active, the top menu bar displays Safari-related commands. Switching to Finder changes the available menus.

### Recall questions

1. How does the macOS menu bar differ from many Windows application menus?
2. What determines which commands appear in the menu bar?
3. How do the Dock and Finder serve different roles?
4. Why can desktop clutter create file-management problems?

---

## 78. Dock

### Core idea

The Dock provides quick access to applications, folders, and minimized windows.

### Typical functions

* launch an application;
* switch to a running application;
* open frequently used folders;
* access minimized windows;
* view the Trash.

### Mental model

The Dock is similar to the Windows taskbar but follows macOS conventions.

### Application state

A Dock icon may represent:

* an application that is not running;
* an application that is currently running;
* an application with one or more open windows.

### Important distinction

Closing an application window does not always quit the application.

The application may remain active and visible in the Dock.

### Common mistakes

* Assuming the application has quit when its last visible window closes.
* Clicking a running application repeatedly and expecting new windows.
* Removing an icon from the Dock and assuming the application was uninstalled.
* Confusing a minimized window with its application icon.

### Quick example

Closing a Safari window may leave Safari running. Selecting **Safari → Quit Safari** ends the application.

### Recall questions

1. Why can an app remain running after its window closes?
2. How does removing a Dock icon differ from uninstalling the app?
3. What kinds of items can the Dock contain besides applications?
4. How can you tell whether reopening is unnecessary because an app is already active?

---

## 79. Finder

### Core idea

**Finder** is the primary macOS application for navigating and managing files and folders.

### Main uses

* browse storage;
* open files;
* create folders;
* move and copy items;
* access connected drives;
* search for files;
* manage desktop contents.

### Mental model

Finder performs a role similar to Windows File Explorer.

| Windows       | macOS                           |
| ------------- | ------------------------------- |
| File Explorer | Finder                          |
| Taskbar       | Dock                            |
| Start search  | Spotlight or other search tools |
| Recycle Bin   | Trash                           |

### Important fact

Finder is itself an application, but it is deeply integrated into normal macOS operation.

### Common mistakes

* Assuming Finder only searches for files.
* Confusing Finder with Spotlight.
* Ejecting storage devices by physically removing them immediately.
* Moving a file unintentionally while dragging it.

### Quick example

Opening a folder from the Dock may display its contents in a Finder window.

### Recall questions

1. How does Finder differ from Spotlight?
2. Why is Finder considered an application despite being central to macOS?
3. Which Windows tool is most similar to Finder?
4. What risks arise when removing a storage drive without ejecting it properly?

---

## 80. Launchpad

### Core idea

Launchpad displays installed applications in an icon-based layout.

### Mental model

```text
Launchpad = Visual application catalogue
```

### Main purpose

Launchpad is useful when:

* the desired application is not in the Dock;
* the user prefers visual browsing;
* applications are organized into groups or pages.

### Launchpad vs Dock

| Launchpad                            | Dock                                          |
| ------------------------------------ | --------------------------------------------- |
| Shows a broad application collection | Shows selected shortcuts and running apps     |
| Opened when needed                   | Usually remains visible or readily accessible |
| Useful for discovery                 | Useful for frequent access                    |

### Common mistakes

* Assuming an app absent from the Dock is not installed.
* Treating Launchpad as a file manager.
* Searching visually through many pages instead of using Spotlight.
* Confusing removal from Launchpad with ordinary shortcut removal.

### Quick example

A rarely used utility can be opened from Launchpad even when it is not permanently placed in the Dock.

### Recall questions

1. When is Launchpad more useful than the Dock?
2. Why is Launchpad not suitable for organizing documents?
3. How does Spotlight provide an alternative to Launchpad?
4. Why might a frequently used application still be placed in the Dock?

---

## 81. Menu Bar

### Core idea

The menu bar provides commands for the active application and access to system functions.

### Left side

The left side commonly contains:

* Apple menu;
* active application name;
* application menus such as File, Edit, or View.

### Right side

The right side commonly contains:

* system status icons;
* search;
* notifications;
* time;
* network and sound controls.

### Mental model

```text
Active application changes → Application menus change
System controls remain available
```

### Why it matters

The menu bar separates:

* application-specific commands;
* system-wide status and controls.

### Common mistakes

* Looking for the wrong application’s commands after changing focus.
* Assuming a menu command applies to every open application.
* Confusing closing a window with quitting from the application menu.
* Ignoring status icons that indicate network or battery problems.

### Quick example

Selecting a Finder window makes Finder active, so Finder-specific commands appear beside the Apple menu.

### Recall questions

1. Why do menu-bar commands change when another application becomes active?
2. Which menu-bar elements remain system-wide?
3. Where would you normally find an application’s Quit command?
4. How can the menu bar reveal which application currently has focus?

---

## 82. Apple Menu

### Core idea

The Apple menu provides system-level commands and settings.

### Typical uses

* open system settings;
* view recent items;
* restart;
* shut down;
* log out;
* put the computer to sleep;
* access system information.

### Mental model

```text
Apple menu = System control menu
Application menu = Current app control
```

### Shutdown relationship

Using the Apple menu allows macOS to:

* notify applications;
* offer to save work;
* close processes;
* shut down safely.

### Common mistakes

* Holding the physical power button for normal shutdown.
* Selecting Restart instead of Shut Down.
* Logging out when intending to turn off the computer.
* Using Sleep when the device must be fully powered down.

### Quick example

A system update may require **Restart**, while transporting or servicing the computer may require **Shut Down**.

### Recall questions

1. Why is the Apple menu system-wide rather than application-specific?
2. How does Sleep differ from Shut Down?
3. Why is a software-controlled shutdown safer than cutting power?
4. When would Restart be more appropriate than Shut Down?

---

## 83. Spotlight Search

### Core idea

**Spotlight** searches across applications, files, folders, and other indexed content.

### Mental model

```text
Describe what you need → Spotlight locates possible matches
```

### Possible uses

* launch an application;
* find a document;
* locate a folder;
* search system content;
* access certain quick information or actions.

### Search advantage

Spotlight avoids the need to remember:

* exact folder location;
* Launchpad page;
* Dock placement.

### Limitation

Search quality depends on:

* accurate keywords;
* indexed content;
* permissions;
* file metadata.

### Common mistakes

* Entering overly broad search terms.
* Assuming a missing result means the file does not exist.
* Confusing web results with local results.
* Searching only by filename when content keywords may work better.

### Quick example

Searching for part of a report’s title may locate the document even when its folder is forgotten.

### Recall questions

1. Why can Spotlight find a file without knowing its location?
2. What factors can cause a relevant result to be missing?
3. When is searching by content more useful than filename?
4. How does Spotlight differ from Launchpad?

---

## 84. Notification Center

### Core idea

Notification Center collects alerts and information from applications and system services.

### Possible notifications

* calendar events;
* reminders;
* messages;
* application updates;
* system alerts.

### Mental model

```text
Applications generate alerts → Notification Center organizes them
```

### Customization

Users can control:

* which applications may send notifications;
* how alerts appear;
* whether sounds are used;
* whether previews are shown;
* whether notifications are grouped.

### Trade-off

| More notifications                | Fewer notifications             |
| --------------------------------- | ------------------------------- |
| Lower chance of missing events    | Fewer interruptions             |
| More immediate information        | Greater focus                   |
| More visual and audio distraction | Important alerts may be delayed |

### Common mistakes

* Allowing every application to send alerts.
* Disabling all notifications and missing important events.
* Showing sensitive previews on a visible screen.
* Treating promotional notifications as necessary system alerts.

### Quick example

Calendar alerts may remain enabled while game and shopping notifications are disabled.

### Recall questions

1. What trade-off exists when reducing notifications?
2. Why may notification previews create a privacy risk?
3. Which applications deserve immediate alerts?
4. How can excessive notifications reduce productivity?

---

## 85. Closing and Quitting Applications

### Core idea

In macOS, closing a window and quitting an application are often separate actions.

### Closing a window

The red control in the upper-left corner generally closes the current window.

### Quitting an application

To fully end an application, use methods such as:

* application name → Quit;
* keyboard shortcut `Command + Q`;
* Dock application menu → Quit.

### Mental model

```text
Close window = Remove one visible workspace
Quit app = End the application process
```

### Important exception

Application behaviour varies. Some apps may quit when their last window closes, while many remain active.

### Common mistakes

* Assuming the red button always quits.
* Leaving many unused applications running.
* Quitting when only one document should be closed.
* Closing unsaved work without reading the prompt.

### Quick example

Closing a document window in a text editor may leave the editor active and ready to create another document.

### Recall questions

1. Why does macOS separate window management from application management?
2. When should a window be closed without quitting the app?
3. How can you determine whether an app is still active?
4. What is the effect of `Command + Q`?

---

## 86. Full-Screen Mode

### Core idea

Full-screen mode expands an application to reduce surrounding distractions.

### Mental model

```text
Full screen = Dedicated visual workspace
```

### Benefits

* increased usable area;
* fewer visible distractions;
* better focus;
* useful presentation or media view.

### Limitations

* other windows become less visible;
* menu and Dock access may change;
* not every application supports the same behaviour;
* switching tasks may initially feel less obvious.

### Source-specific command

The source mentions using:

```text
Control + Command + F
```

to enter or leave full-screen mode in supported applications.

### Common mistakes

* Confusing full screen with maximizing a normal window.
* Assuming the application is frozen because controls are hidden.
* Forgetting the keyboard shortcut to exit.
* Using full screen when frequent cross-application comparison is required.

### Quick example

A video application benefits from full screen, while comparing a browser with notes may work better using separate windows.

### Recall questions

1. How does full-screen mode support focus?
2. When can full screen reduce productivity?
3. Why may controls appear to disappear?
4. How does full screen differ from simply enlarging a window?

---

## 87. Natural Scrolling

### Core idea

Natural scrolling treats the content as though it is being physically moved by the user’s fingers.

### Direction model

With natural scrolling:

* moving fingers upward pushes content upward;
* the view moves toward content farther down the page.

Traditional scrolling often feels like moving the scrollbar rather than the content.

### Mental model

```text
Natural scrolling → Move the page
Traditional scrolling → Move the viewing control
```

### Why it may feel unfamiliar

Mouse-wheel users may have learned the traditional model, while touchscreen users are familiar with directly moving content.

### Preference rule

Neither direction is universally correct. The useful setting is the one that produces fewer errors and feels predictable to the user.

### Common mistakes

* Assuming natural scrolling is broken because it feels reversed.
* Changing the setting before understanding the movement model.
* Expecting another user’s preferred direction to feel natural.
* Applying the setting inconsistently across devices without noticing.

### Quick example

A user accustomed to smartphones may find natural trackpad scrolling intuitive because the gesture resembles dragging a page.

### Recall questions

1. What object is mentally being moved in natural scrolling?
2. Why may mouse and touchscreen users prefer different directions?
3. What makes one scrolling direction better for an individual?
4. How could inconsistent settings across devices cause errors?

---

## 88. Multi-Touch Gestures

### Core idea

Multi-touch gestures map finger movements on a trackpad or compatible mouse to system actions.

### Possible gestures

* taps;
* swipes;
* pinches;
* multi-finger movements;
* double taps.

### Mental model

```text
Gesture pattern → Recognized command
```

### Possible tasks

* move between pages;
* zoom;
* switch applications or spaces;
* open system views;
* scroll.

### Hardware dependency

Available gestures depend on:

* trackpad or mouse capabilities;
* operating-system settings;
* application support.

### Trade-off

Gestures can make navigation faster but may be harder to discover or remember than visible buttons.

### Common mistakes

* Performing the correct motion with the wrong number of fingers.
* Assuming every application supports every gesture.
* Triggering gestures accidentally.
* Memorizing many gestures before learning basic navigation.
* Confusing system-level gestures with application-specific gestures.

### Quick example

A two-finger swipe may navigate between pages in one context, while a different finger count may switch between desktops.

### Recall questions

1. Why do gestures depend on both hardware and software?
2. What makes gesture controls less discoverable than buttons?
3. How could accidental gestures be reduced?
4. When is a visible command preferable to a gesture?

---

# Web Browser Basics

## 89. Web Browser

### Core idea

A **web browser** is an application used to request, retrieve, and display web content.

### Examples from the material

* Chrome;
* Safari;
* Firefox;
* Internet Explorer.

### Mental model

```text
User enters request
→ Browser contacts web server
→ Server returns data
→ Browser interprets and displays it
```

### Browser vs internet

| Browser                                 | Internet                                     |
| --------------------------------------- | -------------------------------------------- |
| Application                             | Global network infrastructure                |
| Displays and interacts with web content | Carries data between systems                 |
| One method of using internet services   | Supports the web, email, messaging, and more |

### Browser differences

Browsers generally support similar core functions but may differ in:

* interface;
* performance;
* privacy features;
* extension support;
* operating-system integration.

### Common mistakes

* Calling the browser “the internet.”
* Confusing a search engine with a browser.
* Assuming a website is installed because it opens in the browser.
* Believing all browsers display every site identically.

### Quick example

Chrome is a browser. A search service opened inside Chrome is a website or web service, not the browser itself.

### Recall questions

1. How does a browser differ from the internet?
2. What role does the web server play when a page opens?
3. Why can the same website look slightly different in two browsers?
4. How is a search engine different from a browser?

---

## 90. Address Bar

### Core idea

The address bar is used to enter or display a web address.

### Web address

A web address is also called a **URL**.

It identifies a resource such as:

* website;
* page;
* file;
* web application location.

### Modern browser behaviour

The address bar may also:

* perform web searches;
* suggest visited sites;
* complete addresses;
* display security information.

### Mental model

```text
Known address → Navigate directly
Unknown information → Search
```

### Suggestions

Suggestions may come from:

* browsing history;
* bookmarks;
* search provider;
* currently open tabs.

### Privacy implication

Address-bar suggestions can reveal previous activity to someone viewing the screen.

### Common mistakes

* Typing an address into a webpage search box unnecessarily.
* Selecting a misleading suggested address.
* Ignoring the final domain after autocomplete.
* Assuming every address-bar suggestion comes from browsing history.
* Entering sensitive searches on a shared screen without considering visibility.

### Quick example

Typing the first few letters of a frequently visited website may cause the browser to suggest the complete address.

### Recall questions

1. How does the address bar combine navigation and search?
2. Where can address suggestions come from?
3. Why should an autocompleted domain still be verified?
4. What privacy issue can address-bar history create?

---

## 91. Links

### Core idea

A **link**, or hyperlink, connects one web resource to another.

### How links work

Selecting a link may:

* open another page;
* move to another part of the same page;
* download a file;
* open another application;
* start an email or other action.

### Mental model

```text
Visible link element → Destination or action
```

### Important distinction

The visible text of a link may differ from its actual destination.

Therefore, sensitive links should be inspected before use.

### Common mistakes

* Assuming every underlined item is safe.
* Clicking a link without checking its destination.
* Confusing links with buttons, even though both may trigger navigation.
* Repeatedly clicking while a page is loading.
* Assuming every link opens a webpage.

### Quick example

A link labelled “Download report” may directly retrieve a file rather than display another page.

### Recall questions

1. What actions can a link perform besides opening a page?
2. Why is visible link text not sufficient for security verification?
3. How could a link launch another application?
4. When should a destination be opened in a separate tab?

---

## 92. Back and Forward Navigation

### Core idea

The browser keeps a navigation sequence for each tab.

### Back button

The Back button returns to the previously viewed location in the tab’s history.

### Forward button

The Forward button becomes useful after moving backward and returns toward the more recent location.

### Mental model

```text
Page A → Page B → Page C

Back: C → B → A
Forward: A → B → C
```

### Important distinction

The Forward button does not normally return to the first page. It moves forward through the tab’s navigation history.

### History relationship

Back and Forward operate on the current tab’s recent path, not the browser’s complete stored history.

### Common mistakes

* Expecting Forward to work before using Back.
* Assuming Back always returns to the previous website rather than the previous page state.
* Repeatedly selecting Back and accidentally leaving a form.
* Confusing tab history with global browser history.

### Quick example

After opening a product page from search results, Back returns to the results. Forward can return to the product page.

### Recall questions

1. Why is the Forward button sometimes unavailable?
2. How does tab navigation differ from full browser history?
3. What risk exists when using Back while completing an online form?
4. Why might Back return to a different position on the same website?

---

## 93. Tabs

### Core idea

Tabs allow several web pages to remain open inside one browser window.

### Mental model

```text
Browser window
├── Tab 1
├── Tab 2
└── Tab 3
```

Each tab maintains its own:

* current page;
* navigation history;
* loading state.

### Main operations

* create a tab;
* switch tabs;
* close a tab;
* reopen a recently closed tab;
* move a tab;
* open a link in another tab.

### Why tabs matter

Tabs support:

* comparison;
* research;
* keeping a source open;
* separating tasks;
* avoiding repeated navigation.

### Trade-off

Too many tabs increase:

* visual clutter;
* memory usage;
* difficulty locating information;
* distraction.

### Common mistakes

* Opening duplicate tabs repeatedly.
* Closing the entire window instead of one tab.
* Keeping hundreds of tabs as a substitute for bookmarks.
* Losing unsaved form data when closing a tab.
* Assuming tabs consume no computer resources.

### Quick example

A student can keep a lesson in one tab and a reference page in another without losing either location.

### Recall questions

1. How does each tab maintain independent navigation?
2. When is a new tab preferable to replacing the current page?
3. Why can excessive tabs reduce performance?
4. How do tabs differ from separate browser windows?

---

## 94. Opening a Link in a New Tab

### Core idea

Opening a link in a new tab preserves the current page while loading the destination separately.

### Common methods

* right-click and select **Open link in new tab**;
* use a keyboard or mouse shortcut;
* open a new blank tab and enter the address.

### Mental model

```text
Current page remains
+
Destination opens separately
```

### Useful situations

* comparing several search results;
* reading references;
* keeping an original page open;
* opening background material for later.

### Trade-off

New tabs preserve context but can quickly create clutter if every link is opened separately.

### Common mistakes

* Opening many tabs without reviewing them.
* Forgetting which page was the original source.
* Closing the parent page before dependent work is complete.
* Assuming the new tab automatically becomes active.

### Quick example

From a search-results page, several relevant sources can be opened in background tabs before being evaluated individually.

### Recall questions

1. How does a new tab preserve browsing context?
2. When is replacing the current page more efficient?
3. Why might a new tab open without appearing immediately?
4. How would you manage several sources without losing the original results page?

---

## 95. Closing Tabs

### Core idea

The X on a tab closes that page while leaving the remaining browser window open.

### Important distinction

| Action       | Effect                          |
| ------------ | ------------------------------- |
| Close tab    | Removes one open page           |
| Close window | Removes all tabs in that window |
| Quit browser | Ends the browser application    |

### Recovery

Browsers commonly provide a command to reopen a recently closed tab.

This may restore:

* the page;
* some navigation history;
* page position.

Recovery is not guaranteed for every form or private session.

### Common mistakes

* Closing the whole window accidentally.
* Assuming unsaved form contents will be restored.
* Keeping unnecessary tabs because reopening is unknown.
* Confusing a webpage logout with closing its tab.

### Quick example

Closing an online account’s tab does not necessarily sign the user out. The session may remain active.

### Recall questions

1. How does closing a tab differ from logging out?
2. Why might a restored tab lose form information?
3. What happens when the last tab in a window is closed?
4. Which action should be used to remove one page without ending the browser?

---

## 96. Bookmarks

### Core idea

A **bookmark** stores a convenient reference to a web address for later access.

### Mental model

```text
Bookmark = Saved route to a page
```

A bookmark does not normally save the complete page contents.

### Useful cases

* frequently visited sites;
* important references;
* pages needed later;
* trusted account sign-in pages.

### Organization

Bookmarks may be organized into folders such as:

* Study;
* Work;
* Finance;
* Tools;
* Reading.

### Bookmark limitations

A bookmark may stop working when:

* the page is moved;
* the site removes it;
* access permissions change;
* the address expires.

### Security relationship

A correctly created bookmark can provide a safer route to a sensitive website than an unexpected email link.

However, a bookmark created while already on a fraudulent page preserves the wrong address.

### Common mistakes

* Treating bookmarks as saved offline copies.
* Saving pages without meaningful names.
* Building an unorganized bookmark collection.
* Assuming a bookmarked page will always remain available.
* Bookmarking a login page without first verifying its domain.

### Quick example

A verified university portal can be bookmarked so future access does not depend on searching for it each time.

### Recall questions

1. Why is a bookmark not the same as an offline copy?
2. How can bookmark folders improve retrieval?
3. Why should a sensitive site be verified before bookmarking it?
4. What can cause a bookmark to stop working?

---

## 97. Browser History

### Core idea

Browser history records pages visited over time.

### Possible uses

* return to a page that was not bookmarked;
* identify recent research;
* recover a closed location;
* supply address-bar suggestions.

### Mental model

```text
History = Automatically recorded browsing trail
Bookmark = Intentionally saved destination
```

### Privacy implication

History may reveal:

* interests;
* account usage;
* research topics;
* sensitive websites;
* browsing times.

### Important limitation

History may be absent or incomplete because of:

* private browsing;
* manual deletion;
* browser settings;
* synchronization differences;
* use of another device or browser.

### Common mistakes

* Treating history as permanent storage.
* Assuming deleting local history deletes records held by websites or networks.
* Depending on history instead of bookmarking important resources.
* Forgetting that shared-device users may view it.
* Confusing search history with all browsing history.

### Quick example

A page visited yesterday can often be found in history even when its exact address is forgotten.

### Recall questions

1. How does browser history differ from bookmarks?
2. Why does deleting browser history not erase every record of activity?
3. What privacy risk exists on a shared computer?
4. Under what conditions may a visited page not appear in history?

---

## 98. Browser Suggestions

### Core idea

Browsers use stored and external information to predict what the user may want to open or search.

### Possible sources

* browsing history;
* bookmarks;
* open tabs;
* popular searches;
* search-engine predictions.

### Mental model

```text
Partial input + Stored context + External predictions
→ Suggested completion
```

### Benefit

Suggestions reduce:

* typing;
* repeated searching;
* navigation time.

### Risk

Suggestions may:

* expose private history;
* lead to the wrong site;
* encourage selection without domain verification;
* reflect external popularity rather than user intent.

### Common mistakes

* Pressing Enter before reading the completed address.
* Assuming the first suggestion is the intended site.
* Selecting a search suggestion when direct navigation was intended.
* Allowing sensitive suggestions to appear during screen sharing.

### Quick example

Typing `doc` may suggest:

* a bookmarked document site;
* a previously visited page;
* a search for “document editor.”

The user must identify which type of suggestion is shown.

### Recall questions

1. Why can two visually similar suggestions perform different actions?
2. How can suggestions expose private browsing activity?
3. Why should the completed domain be checked before navigation?
4. When would disabling some suggestions be useful?

---

## 99. Browser Interface Differences

### Core idea

Browsers may arrange controls differently while supporting similar fundamental tasks.

### Common shared functions

Most browsers support:

* address entry;
* tabs;
* Back and Forward;
* bookmarks;
* history;
* downloads;
* privacy settings;
* extensions;
* updates.

### Mental model

```text
Different interface ≠ Different basic web model
```

### Transferable knowledge

Instead of memorizing only button locations, understand each function’s purpose.

For example:

* find the address bar;
* locate tab controls;
* identify browser settings;
* find history and bookmarks.

### Why it matters

Interface details change because of:

* browser choice;
* software updates;
* operating system;
* screen size;
* user customization.

### Common mistakes

* Believing a feature is unavailable because it moved.
* Following outdated instructions mechanically.
* Memorizing icons without understanding their function.
* Assuming identical keyboard shortcuts on every platform.

### Quick example

The bookmark control may appear as a star in one browser and inside a menu in another, but both preserve a page address.

### Recall questions

1. Why is conceptual knowledge more durable than memorizing button positions?
2. Which browser functions remain common despite interface differences?
3. What should you do when instructions refer to an older browser layout?
4. How can operating-system differences change browser shortcuts?

---

# Integrated Mental Models

## 100. Complete Computer-System Model

### Core idea

A useful understanding of computers connects physical components, software, networks, and user actions.

### System relationship

```text
User
  ↓
Application
  ↓
Operating system
  ↓
Hardware
  ↓
Local network
  ↓
Internet service provider
  ↓
Remote server
```

### Example: Opening a cloud document

1. The user selects a browser.
2. The operating system launches the application.
3. The browser sends a network request.
4. Hardware processes and transmits data.
5. The router directs traffic.
6. The modem connects through the ISP.
7. A remote server returns the document.
8. The browser displays it.
9. Active data is held in RAM.
10. Changes may be synchronized to cloud storage.

### Why it matters

Problems can occur at different layers.

A useful troubleshooting approach identifies the layer before changing settings.

### Quick diagnostic model

| Symptom                          | Likely layers to inspect       |
| -------------------------------- | ------------------------------ |
| Computer will not power on       | Power and hardware             |
| One app fails                    | Application or OS              |
| Wi-Fi connected, no internet     | Router, modem, ISP             |
| One website unavailable          | Browser, DNS, or remote server |
| Unsaved work lost after shutdown | RAM and saving behaviour       |
| Computer overheats               | Dust, airflow, fan, workload   |

### Common mistakes

* Blaming the internet when one application fails.
* Reinstalling software for a disconnected cable.
* Replacing hardware before checking settings.
* Treating every symptom as an isolated problem.
* Changing several variables at once during troubleshooting.

### Recall questions

1. Which layers participate when opening a web application?
2. How would you isolate whether a problem is local or server-side?
3. Why should only one troubleshooting variable be changed at a time?
4. Which layers are involved when a wireless printer fails to print?

---

## 101. Security Mental Model

### Core idea

Security depends on reducing both the chance and impact of failure.

### Layered protection

```text
Careful behaviour
+ Verified identities
+ Updated software
+ Antivirus
+ Strong authentication
+ Backups
```

### Relationship between layers

| Layer               | Main role                      |
| ------------------- | ------------------------------ |
| Safe browsing       | Avoid initial exposure         |
| Domain verification | Avoid impersonation            |
| Updates             | Remove known weaknesses        |
| Antivirus           | Detect some malicious software |
| Strong passwords    | Reduce unauthorized access     |
| Backups             | Recover after damage or loss   |

### Rule

No single control is sufficient.

Examples:

* HTTPS does not prove honesty.
* Antivirus does not make every download safe.
* A strong password does not help if it is entered on a phishing site.
* Cloud synchronization does not always provide backup.

### Common mistakes

* Looking for one perfect security product.
* Focusing only on technical controls.
* Ignoring account recovery security.
* Assuming inconvenience means a protection is unnecessary.
* Failing to prepare for controls that eventually fail.

### Quick example

A phishing email bypasses antivirus because it contains no malware. Domain verification and independent account access are the relevant defences.

### Recall questions

1. Why does layered security remain useful when each layer is imperfect?
2. Which defence is most relevant against a fake login page?
3. Why are backups part of security rather than only maintenance?
4. How could one compromised email account weaken several other accounts?

---

## 102. Maintenance Mental Model

### Core idea

Computer reliability depends on maintaining both the physical and software environment.

### Two maintenance categories

| Physical maintenance     | Software maintenance           |
| ------------------------ | ------------------------------ |
| Remove dust              | Install updates                |
| Protect airflow          | Remove unnecessary files       |
| Prevent spills           | Scan for malware               |
| Manage cables            | Optimize storage appropriately |
| Use safe power equipment | Maintain backups               |

### Cause-and-effect relationship

```text
Small preventive actions
→ Lower failure probability
→ Less disruption and recovery cost
```

### Important distinction

Maintenance reduces risk but does not eliminate aging, defects, or unexpected failure.

Therefore:

> Maintenance and backup are complementary.

### Common mistakes

* Maintaining software while ignoring cooling.
* Cleaning hardware while ignoring updates.
* Waiting for failure before creating backups.
* Applying the same optimization method to HDDs and SSDs.
* Assuming a well-maintained device cannot fail.

### Quick example

A clean, updated computer can still experience sudden storage failure. A tested backup provides recovery when prevention is insufficient.

### Recall questions

1. Why must physical and software maintenance be combined?
2. Which maintenance actions reduce overheating?
3. Which actions support recovery rather than prevention?
4. Why can a well-maintained computer still require replacement?

---

## 103. Usability Mental Model

### Core idea

Computer interfaces differ, but most user actions follow recurring patterns.

### Common patterns

* select an object;
* open it;
* change or use it;
* save results;
* close or exit;
* organize it for later access.

### Cross-platform relationships

| Purpose      | Windows example      | macOS example              | Browser example     |
| ------------ | -------------------- | -------------------------- | ------------------- |
| Launch apps  | Start or taskbar     | Dock, Launchpad, Spotlight | Browser icon        |
| Manage files | File Explorer        | Finder                     | Downloads interface |
| Switch tasks | Taskbar              | Dock                       | Tabs                |
| Search       | Start search         | Spotlight                  | Address bar         |
| Close item   | X button             | Red window control         | Tab X               |
| Exit app     | Close or app command | Quit command               | Quit browser        |

### Why it matters

Recognizing functions makes it easier to adapt to:

* another operating system;
* a redesigned interface;
* a new browser;
* unfamiliar applications.

### Common mistakes

* Memorizing exact screen positions only.
* Assuming the same symbol always has identical behaviour.
* Ignoring context when clicking.
* Fearing experimentation instead of observing interface feedback.

### Quick example

A magnifying-glass symbol commonly represents search, but the search scope may be a page, application, device, or internet service.

### Recall questions

1. Why is function-based learning more transferable than position-based learning?
2. How can the same search icon operate on different scopes?
3. Which interaction patterns appear across operating systems and browsers?
4. What contextual clues indicate whether closing affects a file, window, or application?

---

## 104. Troubleshooting Mental Model

### Core idea

Troubleshooting is the process of locating the layer where expected behaviour breaks.

### Recommended sequence

```text
1. Define the exact symptom
2. Check simple physical dependencies
3. Isolate the affected scope
4. Change one variable
5. Test the result
6. Record or reverse the change
```

### Scope questions

Ask:

* Does the problem affect one file?
* One application?
* One device?
* The whole computer?
* The local network?
* Every internet service?
* One remote website?

### Cause isolation

| Observation                     | Possible inference                    |
| ------------------------------- | ------------------------------------- |
| Other websites work             | Internet connection probably exists   |
| Other devices connect           | Router and ISP may be working         |
| One USB port fails              | Device may work; port may be faulty   |
| External monitor works          | Built-in display path may be affected |
| Restart fixes issue temporarily | Software state may be involved        |

### Assumption warning

An observation narrows possibilities but may not prove one cause.

For example, if one website fails, the cause could still be:

* browser cache;
* DNS;
* account issue;
* local blocking;
* remote server failure.

### Common mistakes

* Making several changes simultaneously.
* Starting with destructive actions.
* Assuming correlation proves cause.
* Ignoring recent changes.
* Failing to preserve important data before repair attempts.

### Quick example

If a wireless mouse fails:

1. check power;
2. check its switch;
3. check the receiver or pairing;
4. test another USB port;
5. test the mouse on another computer.

Each step isolates one dependency.

### Recall questions

1. Why should troubleshooting begin by defining the exact symptom?
2. How does testing another device narrow a network problem?
3. Why should one variable be changed at a time?
4. What evidence would distinguish an application problem from an operating-system problem?
5. Design a troubleshooting sequence for a browser that cannot open one specific website.

---

## 105. Final Course Relationships

### Core idea

The course’s major concepts form a connected system rather than separate facts.

### Core relationships

```text
Hardware executes software.
The operating system manages hardware and applications.
Applications help users perform tasks.
Networks connect devices.
Servers provide remote resources.
Browsers access web resources.
Cloud services store or process data remotely.
Security reduces risk.
Backups reduce the impact of failure.
Ergonomics protects the user.
Maintenance protects the equipment.
```

### Key trade-offs

| Choice                        | Main trade-off                            |
| ----------------------------- | ----------------------------------------- |
| Laptop vs desktop             | Portability vs flexibility                |
| Wi-Fi vs Ethernet             | Mobility vs stability                     |
| Local vs cloud storage        | Direct control vs remote access           |
| HDD vs SSD                    | Cost per capacity vs speed and durability |
| Personalization vs privacy    | Relevance vs data collection              |
| Integration vs upgradeability | Compact design vs replaceable parts       |
| Convenience vs security       | Faster access vs stronger verification    |

### Rules to remember

* Save active work to persistent storage.
* Back up important files in more than one location.
* Verify domains before entering sensitive information.
* Treat the padlock as encryption, not proof of trust.
* Keep software updated.
* Never force a connector.
* Protect airflow.
* Use the correct cleaning method for each surface.
* Adjust the workspace to the body.
* Diagnose problems by layer and scope.

### Common mistakes

* Treating memorized terminology as understanding.
* Ignoring relationships between components.
* Assuming one product or setting solves every problem.
* Optimizing convenience without considering risk.
* Waiting for loss or injury before applying preventive measures.

### Quick example

A slow cloud document may involve:

* limited internet performance;
* Wi-Fi interference;
* browser problems;
* insufficient RAM;
* remote server load.

Understanding the full system prevents immediately blaming the CPU.

### Recall questions

1. How do hardware, the operating system, and applications depend on one another?
2. Which trade-off explains why laptops are harder to upgrade?
3. How do antivirus protection and backups address different stages of failure?
4. Why can a cloud application fail even when the computer hardware is healthy?
5. How would you evaluate whether a problem belongs to the device, local network, ISP, or remote server?
6. Design a secure and reliable setup for a student who works from several devices.
7. Which course principles remain useful even when specific technologies change?

END OF NOTES

# QUESTIONs:

# Conceptual Understanding

1. **[Moderate]** Why is a computer better understood as an input–process–output–storage system than simply as a machine that “calculates”?

2. **[Hard]** Hardware can exist without useful software, and software can exist without running hardware. What does this reveal about the dependency between the two?

3. **[Tricky]** A monitor displays a video, but the monitor does not perform all the work required to play it. Trace which components are responsible for storage, processing, temporary data, graphics generation, and final display.

4. **[Moderate]** Why does describing the CPU as the “brain” help beginners but risk creating an incomplete mental model?

5. **[Hard]** A user increases a computer’s RAM but sees no improvement in a calculation-heavy task. Which assumptions about RAM and performance were probably incorrect?

6. **[Hard]** Explain why volatile memory is useful even though its contents disappear when power is lost.

7. **[Tricky]** A computer has a large storage drive but cannot keep many programs open smoothly. What does this show about the difference between capacity and active working memory?

8. **[Hard]** Why is the motherboard important even though it does not independently perform all processing, storage, and display tasks?

9. **[Moderate]** How does a specialized computer, such as one embedded in an appliance, differ from a general-purpose computer without ceasing to be a computer?

10. **[Hard]** A machine acts as a client in one interaction and a server in another. What does this imply about the meaning of “client” and “server”?

11. **[Inference, Hard]** Why is it more accurate to define a server by its role than by its physical size or appearance?

12. **[Tricky]** A laptop and desktop use similar internal concepts but create different user experiences. Which design priorities produce those differences?

13. **[Hard]** How does an operating system reduce the amount of hardware-specific knowledge required by applications?

14. **[Moderate]** Why is an application not the same thing as the file it creates or edits?

15. **[Hard]** A web application appears inside a browser but performs work on remote servers. Which parts belong to the browser, the application, the operating system, and the server?

16. **[Tricky]** Why can a router create a functioning local network even when the household has no working internet connection?

17. **[Hard]** Explain why Wi-Fi and internet access are related but not interchangeable concepts.

18. **[Moderate]** How do an SSID, a Wi-Fi password, and encryption solve three different networking problems?

19. **[Hard]** Why is cloud storage more accurately described as remote physical infrastructure than as an abstract or locationless space?

20. **[Tricky]** A synchronized folder exists on three devices. One file is deleted from one device and disappears everywhere. Why does this behaviour make synchronization different from backup?

---

# Why and How

21. **[Hard]** Why can low free storage space interfere with application updates even when the computer still has enough CPU power and RAM?

22. **[Tricky]** Why does defragmentation help a mechanical hard drive more than a solid-state drive?

23. **[Hard]** How can blocked airflow reduce performance before causing an obvious shutdown or hardware failure?

24. **[Moderate]** Why should a connector that does not fit be rechecked rather than forced?

25. **[Hard]** How can two USB-C ports with identical shapes support different capabilities?

26. **[Tricky]** A monitor is powered on and connected, but it shows “No Signal.” Why is replacing the monitor an unjustified first conclusion?

27. **[Hard]** Why may a wireless mouse have power yet still fail to control the pointer?

28. **[Moderate]** Why does a desktop power supply do more than merely connect the computer to a wall outlet?

29. **[Hard]** Why can a laptop continue working through a brief power outage while a normal desktop immediately shuts down?

30. **[Tricky]** Why may adding a faster SSD improve startup and loading times without improving a long mathematical calculation?

31. **[Hard]** How does a surge protector reduce one kind of electrical risk without providing the same protection as a battery backup?

32. **[Hard]** Why can a high-bandwidth internet connection still feel unresponsive in an online game?

33. **[Moderate]** Why may several devices using one fast connection receive less performance than one device using it alone?

34. **[Tricky]** How can a device display strong Wi-Fi signal while internet access remains unavailable?

35. **[Hard]** Why must a user-purchased modem be checked for ISP compatibility even if its connector physically matches?

36. **[Hard]** How does a router know which local device should receive incoming data?

37. **[Inference, Hard]** Why might Ethernet improve stability without fixing slow performance caused by the ISP?

38. **[Tricky]** Why may cloud access fail even when the local computer, browser, and Wi-Fi are all working correctly?

39. **[Hard]** How does automatic backup reduce human error while still leaving risks that must be managed?

40. **[Moderate]** Why is a backup useful only if restoration is possible?

41. **[Hard]** Why can an external backup drive stored beside a computer fail to protect against some disasters?

42. **[Tricky]** Why may a cloud-synchronized file be vulnerable to ransomware even when a copy exists online?

---

# Connections Between Topics

43. **[Hard]** Connect RAM, storage, the CPU, and the operating system in the process of opening, editing, and saving a document.

44. **[Tricky]** How do portability, battery use, compact cooling, and limited upgradeability arise from the same laptop design goal?

45. **[Hard]** Explain how motherboard compatibility can limit a planned CPU, RAM, or expansion-card upgrade.

46. **[Inference, Creative]** A company wants highly repairable computers but also wants the thinnest possible devices. Which design conflict from the notes does this create?

47. **[Hard]** How do physical maintenance and software maintenance protect against different categories of failure?

48. **[Tricky]** Why do antivirus software, updates, safe browsing, and backups remain useful even though none provides complete protection?

49. **[Hard]** Connect domain verification, HTTPS, phishing, and password security in the process of safely signing in to a bank account.

50. **[Creative]** A fraudulent website uses a correct-looking logo, professional grammar, and HTTPS. Which evidence from the notes still justifies distrust?

51. **[Hard]** How can browser history, bookmarks, and address-bar suggestions support navigation while creating different privacy risks?

52. **[Tricky]** Why can blocking third-party cookies reduce tracking while failing to prevent cross-device profiling?

53. **[Hard]** Connect account sign-ins, probabilistic matching, shared networks, and advertising profiles.

54. **[Inference, Hard]** Why might two people in the same household receive advertisements based partly on each other’s browsing behaviour?

55. **[Hard]** How do ergonomic monitor position, text readability, posture, and eye strain influence one another?

56. **[Tricky]** Why can raising a laptop screen improve neck posture while creating a new problem for the wrists and shoulders?

57. **[Hard]** Connect chair height, wrist angle, foot support, and keyboard position in one ergonomic chain of cause and effect.

58. **[Creative]** A user buys an expensive ergonomic keyboard but keeps the chair and desk badly positioned. Why may the purchase have little effect?

59. **[Hard]** How do dust buildup, fan behaviour, CPU heat, and automatic performance reduction relate?

60. **[Tricky]** A computer is visually clean but still overheats. Which maintenance assumptions should be questioned?

61. **[Hard]** Connect browser tabs, RAM use, multitasking, and perceived computer performance.

62. **[Inference, Hard]** Why may closing many browser tabs improve performance on one computer but produce little change on another?

63. **[Hard]** How do the Windows taskbar, macOS Dock, browser tabs, and application windows all support task switching while representing different levels of activity?

64. **[Tricky]** Why does closing a visible window not always mean that its application has stopped running?

65. **[Hard]** Compare File Explorer, Finder, Spotlight, Launchpad, and browser history by the kind of resource each helps locate.

66. **[Creative]** A user says, “I lost my file because the icon disappeared.” Which distinctions among shortcuts, files, folders, applications, and search tools are needed to investigate the claim?

---

# Application and Scenarios

67. **[Moderate]** A student travels daily but uses a large monitor and keyboard at home. Which setup best satisfies both portability and workspace comfort, and what trade-offs remain?

68. **[Hard]** A desktop powers on, fans spin, and the monitor displays nothing. Design a troubleshooting sequence that follows the notes’ layer-by-layer method.

69. **[Tricky]** A USB-C cable charges a laptop but does not send video to a monitor. Identify at least three assumptions that should be tested.

70. **[Hard]** A user installs more RAM to solve slow file transfers from an old hard drive. Why might the upgrade fail to address the bottleneck?

71. **[Creative]** A computer becomes slow only during long gaming sessions but performs normally after cooling down. Use the notes to construct the most plausible chain of causes.

72. **[Hard]** A wireless keyboard stops responding after being moved to another room. Build a diagnostic sequence that separates power, pairing, receiver, range, and interference problems.

73. **[Tricky]** A monitor remains blank after a cable is connected correctly. Which monitor-side and computer-side settings should be checked before replacing hardware?

74. **[Hard]** A home has fibre internet, but video calls are unstable in one bedroom. Explain why the internet service type alone does not determine the user experience.

75. **[Creative]** A family’s smart television streams well through Ethernet, but phones perform poorly over Wi-Fi. What does this reveal about the likely location of the problem?

76. **[Hard]** A computer shows Wi-Fi connectivity, local file sharing works, but no websites open. Which components are probably functioning, and which layers remain suspect?

77. **[Tricky]** A user buys a second router because Wi-Fi is weak, then discovers the modem already contains routing functions. What conceptual confusion caused the unnecessary purchase?

78. **[Hard]** A visitor needs internet access but should not reach personal computers or shared storage. Which network design from the notes is most appropriate, and why?

79. **[Creative]** A user deletes a project from a synchronized folder and assumes the cloud copy will remain safe. Explain why this expectation may fail and what backup feature would have helped.

80. **[Hard]** A backup service reports success every day, but restoration has never been tested. Why is the system still unproven?

81. **[Tricky]** A laptop and external backup drive are stolen together. Which backup principle was violated?

82. **[Hard]** A user receives a bank email with correct personal details and a link to an HTTPS page. Build a safe verification process without using any contact route supplied in the email.

83. **[Creative]** An email contains no attachment and no malware, yet it still causes account theft. Explain how this can happen.

84. **[Hard]** A browser warns that a certificate is invalid on a familiar banking domain. Why is “I have used this site before” insufficient justification for continuing?

85. **[Tricky]** A user manually types a company name into a search engine and selects the first advertisement. Why does this not provide the same assurance as using a verified bookmark?

86. **[Hard]** A sender knows the last four digits of a user’s card. Why should the user still refuse to provide a one-time authentication code?

87. **[Creative]** An invisible email image confirms that an address is active without installing malware. Trace the mechanism.

88. **[Hard]** A shared computer reveals private websites in address-bar suggestions during a presentation. Which browser features contributed to the exposure?

89. **[Tricky]** A user deletes cookies but remains signed into a major account across several services. Why may personalized tracking continue?

90. **[Hard]** Several family members share one network and occasionally one tablet. Explain how probabilistic matching could create an inaccurate combined profile.

---

# Comparison and Trade-offs

91. **[Moderate]** Compare a laptop and desktop for a user whose priorities are repairability, portability, screen size, and power-outage resilience.

92. **[Hard]** Compare an HDD and SSD for a portable device, a large archive, and a system drive. Explain why one choice may not suit all three roles.

93. **[Tricky]** Why is “more storage” not equivalent to “more memory,” and why does the everyday use of the word *memory* create confusion?

94. **[Hard]** Compare internal expansion cards with external USB accessories as methods of adding capabilities.

95. **[Creative]** A manufacturer removes expansion slots, replaces ports with USB-C, and integrates the display with the computer. What benefits and costs follow from this design direction?

96. **[Hard]** Compare Ethernet and Wi-Fi for mobility, stability, interference, installation effort, and security exposure.

97. **[Tricky]** Under what circumstances could Wi-Fi be the better choice even when Ethernet would provide a more stable connection?

98. **[Hard]** Compare dial-up, DSL, cable, fibre, and cellular service without assuming that maximum speed is the only relevant factor.

99. **[Creative]** A remote location offers slow but stable DSL and fast but inconsistent cellular service. What kinds of users might rationally choose each?

100. **[Hard]** Compare synchronization and backup in terms of purpose, deletion behaviour, version history, and recovery.

101. **[Tricky]** Why can a service be excellent for collaboration but weak for disaster recovery?

102. **[Hard]** Compare local backup, external-drive backup, and cloud backup by recovery speed, physical risk, cost, and dependency.

103. **[Moderate]** Compare minimizing, closing a window, closing a tab, logging out, and quitting an application.

104. **[Hard]** Compare the Windows taskbar and macOS Dock without treating them as exact equivalents.

105. **[Tricky]** Compare Finder and Spotlight by explaining when search is preferable to hierarchical navigation and when it is not.

106. **[Creative]** Compare a bookmark and browser history as two different forms of “remembering” a page.

107. **[Hard]** Compare full-screen mode, maximizing, and arranging windows side by side according to focus, visibility, and multitasking needs.

108. **[Tricky]** Compare natural scrolling and traditional scrolling using the different mental objects the user imagines moving.

---

# Errors and Misconceptions

109. **[Moderate]** A user calls the desktop tower “the CPU.” What misunderstandings about internal components could follow from this language?

110. **[Hard]** “The monitor is the computer because everything appears there.” Identify the processing, storage, and output misconceptions in this claim.

111. **[Tricky]** “My computer has 2 TB of memory, so it should run many programs easily.” Diagnose the terminology and performance assumptions.

112. **[Hard]** “An SSD makes the processor calculate faster.” Explain the limited situations in which the computer may feel faster and the situations in which it will not.

113. **[Creative]** “My laptop has no expansion slots, so nothing can be added to it.” Use internal and external expansion concepts to challenge the claim.

114. **[Hard]** “The Wi-Fi icon is full, so the internet cannot be the problem.” Separate wireless-link quality from internet availability and remote-service health.

115. **[Tricky]** “A router gives me internet, so I do not need an ISP.” Identify the missing dependency.

116. **[Hard]** “The padlock proves the website belongs to my bank.” Explain precisely what the padlock does and does not establish.

117. **[Creative]** “The email contains no spelling mistakes, so it is not phishing.” Explain why this heuristic is weak.

118. **[Hard]** “Antivirus is installed, so suspicious downloads are safe.” Identify the security-layer mistake.

119. **[Tricky]** “I use cloud storage, so I do not need backups.” Under what service behaviours would this claim fail?

120. **[Hard]** “Deleting browser history removes all evidence of browsing.” Identify the other systems or parties that may still retain information.

# Errors and Misconceptions — Continued

121. **[Moderate]** “If the computer is externally clean, overheating cannot be caused by dust.” What hidden areas make this conclusion unreliable?

122. **[Hard]** “Defragmentation is general maintenance that improves every storage device.” Why is applying one optimization model to HDDs and SSDs incorrect?

123. **[Tricky]** “A surge protector will let me finish saving my work during a blackout.” Which two protective devices are being confused?

124. **[Hard]** “The mouse pointer moves badly, so the operating system must be corrupted.” Which simpler physical causes should be tested first?

125. **[Creative]** “The keyboard survived because it started working immediately after a spill.” Why might immediate success be misleading?

126. **[Hard]** “Spraying cleaner directly is more effective because more liquid reaches the dirt.” Explain why this increases risk for monitors and keyboards.

127. **[Tricky]** “Glass cleaner is safe because a monitor has a glass-like surface.” Which hidden screen property makes this assumption unsafe?

128. **[Hard]** “Sitting perfectly upright without moving is ideal posture.” Why does the notes’ ergonomic model reject both slouching and rigid stillness?

129. **[Creative]** “A standing desk removes the health risks of desk work.” Which risk remains unchanged when the user stands without moving?

130. **[Hard]** “Night mode prevents eye strain.” Identify the separate factors that can still produce discomfort.

131. **[Tricky]** “Looking at a phone counts as a 20-20-20 break because it is a different screen.” Why does this fail the purpose of the rule?

132. **[Moderate]** “An application is gone because its shortcut disappeared from the desktop or Dock.” What distinction disproves this conclusion?

133. **[Hard]** “Closing the last visible macOS window always exits the app.” Why does the application–window distinction matter here?

134. **[Tricky]** “A minimized program is closed because I cannot see it.” Which evidence would show that it remains active?

135. **[Hard]** “Forward returns to the first page I visited.” Explain the actual relationship between Back, Forward, and a tab’s navigation sequence.

136. **[Creative]** “Bookmarks are permanent copies of websites.” What kinds of website changes expose this misconception?

137. **[Hard]** “Private browsing prevents websites, employers, and ISPs from observing activity.” Why is this an overgeneralization?

138. **[Tricky]** “Blocking all cookies provides privacy without affecting usability.” Which normal site functions may depend on cookies?

139. **[Hard]** “A hidden SSID makes weak Wi-Fi encryption acceptable.” Why does hiding the network name solve a different and much weaker problem?

140. **[Creative]** “If a file is visible in RAM, it must already be saved.” Construct a situation in which this belief causes data loss.

---

# Edge Cases and Exceptions

141. **[Hard]** A computer has functioning hardware but no usable operating system. Which capabilities may remain technically possible, and which normal user interactions become inaccessible?

142. **[Inference, Tricky]** Could a device with no conventional screen still provide meaningful output? Justify using the broader input–process–output model.

143. **[Hard]** A computer performs one narrow task exceptionally well but cannot install general applications. Why does this limitation not disqualify it as a computer?

144. **[Tricky]** Can the same physical machine act as both server and client during one session? Construct a valid example using only concepts from the notes.

145. **[Hard]** A laptop’s battery no longer holds a charge, but the laptop works when plugged in. Which laptop advantages have been lost, and which remain?

146. **[Inference, Creative]** A desktop is connected to an uninterruptible power supply but its monitor is connected only to a surge protector. What unusual failure experience could occur during a blackout?

147. **[Hard]** A USB-C port supports data and charging but not display output. How should this affect cable and adapter selection?

148. **[Tricky]** An external monitor works through one laptop port but not another visually identical port. Which capability assumption has failed?

149. **[Hard]** A computer has an optical drive but cannot read a particular disc. Why is the presence of a disc slot insufficient evidence of format compatibility?

150. **[Creative]** An old peripheral works through a legacy port but fails through an inexpensive adapter. Which layers of compatibility might the adapter fail to provide?

151. **[Hard]** A desktop has free expansion slots but cannot accept a chosen graphics card. Identify non-space-related compatibility constraints suggested by the system model.

152. **[Inference, Tricky]** Why might a physically smaller graphics card still be unsuitable for a computer with an available slot?

153. **[Hard]** A program opens correctly but fails only when handling very large files. Which resource limits might become relevant only at that scale?

154. **[Creative]** A computer has abundant RAM but becomes slow while opening many cloud documents. Construct two local causes and two network-side causes.

155. **[Hard]** A user saves a document, but the newest changes are missing after reopening it. What possibilities exist besides RAM loss?

156. **[Tricky]** An SSD has plenty of free space but an application launches slowly. Why does storage capacity fail to identify the bottleneck?

157. **[Hard]** A router has working internet access through Ethernet but cannot broadcast Wi-Fi. Which router functions are separable in this case?

158. **[Inference, Creative]** A household can print to a network printer during an ISP outage. What does this reveal about local and external network dependencies?

159. **[Hard]** A Wi-Fi device joins the correct SSID with the correct password but receives no usable connection. Which stages after authentication might still fail?

160. **[Tricky]** A device without wireless hardware must join a network in a room where Ethernet cabling is impossible. Which expansion principle offers a solution?

161. **[Hard]** A cloud document is available on one device but not another using the same account. Which synchronization, application, or permission exceptions should be considered?

162. **[Creative]** A user edits a cloud file offline on two devices and reconnects both later. What conflict does the service need to resolve?

163. **[Hard]** An online backup preserves files but cannot restore applications or system settings. Why may it still be a valid backup while not being a complete-device recovery solution?

164. **[Tricky]** A deleted cloud file can be restored for 30 days but not after 31 days. Which backup property determines this boundary?

165. **[Hard]** A backup drive is disconnected except during scheduled backups. Which risk is reduced, and which new human-factor risk is introduced?

166. **[Inference, Creative]** An automatic cleanup tool removes files from Downloads after a set period. Under what workflow could this reasonable rule cause important data loss?

167. **[Hard]** A mechanical hard drive is regularly defragmented but remains slow because it is nearly full. Why are these two storage issues distinct?

168. **[Tricky]** A browser update is installed but a security flaw remains exploitable until restart. What does this reveal about downloaded versus active software versions?

169. **[Hard]** Antivirus quarantines a malicious program after it has already stolen a password. Why is malware removal not the end of incident recovery?

170. **[Creative]** A phishing message directs the user to the genuine company website but asks them to call a fraudulent phone number afterward. Why does verifying only the domain fail?

171. **[Hard]** An email appears in the spam folder but is a legitimate deadline notice. Which filtering limitation explains this, and what user habit reduces the risk?

172. **[Tricky]** A remotely hosted email image is loaded through a privacy proxy. How might this weaken the sender’s ability to infer the recipient’s exact behaviour without eliminating every signal?

173. **[Inference, Hard]** Why might blocking all external email images reduce tracking but make some legitimate messages harder to use?

174. **[Creative]** A phishing page correctly rejects an incorrect password before accepting the real one. How could this behaviour increase credibility without proving legitimacy?

175. **[Hard]** A fraudulent domain differs from the legitimate one only by a visually similar character. Why are branding and page design especially weak checks in this edge case?

176. **[Tricky]** A website’s HTTPS certificate is valid, but the user reached it through a deceptive shortened link. What has been secured, and what remains unverified?

177. **[Hard]** A browser blocks third-party cookies, but the same user signs into one advertising account on every device. Why can cross-device matching remain strong?

178. **[Creative]** A family shares a browser profile but uses separate website accounts. Which tracking signals may still combine their activity incorrectly?

179. **[Hard]** A user closes every browser tab, yet music continues playing. Which application, tab, or background-process assumptions should be investigated?

180. **[Tricky]** A browser tab is restored after accidental closure, but a completed form is empty. Why is page restoration not equivalent to restoring application state?

---

# Critical Reasoning

181. **[Hard]** Which is the stronger general troubleshooting question: “What component is broken?” or “At which layer does expected behaviour stop?” Justify using several examples.

182. **[Inference, Creative]** Why does changing several settings at once reduce the value of a successful troubleshooting result?

183. **[Hard]** A restart temporarily fixes a recurring problem. What can reasonably be inferred, and what cannot be concluded?

184. **[Tricky]** Other devices access the internet normally, but one laptop cannot. Which network components are less likely to be responsible, and why are they not completely eliminated?

185. **[Hard]** One website fails in every browser on one computer but works on a phone using cellular data. Construct competing explanations at the device, local-network, and remote-service levels.

186. **[Creative]** A web service fails only when the user is signed in. Why does this observation shift attention away from basic internet connectivity?

187. **[Hard]** A computer becomes slow after opening many applications. How would you distinguish insufficient RAM from high CPU load or slow storage using symptoms and controlled tests?

188. **[Tricky]** A new SSD makes the system feel faster, leading the user to conclude that every component was previously too slow. Why is perceived overall improvement weak evidence about individual bottlenecks?

189. **[Hard]** When is replacing hardware an unreasonable first troubleshooting step, and when might it become justified?

190. **[Inference, Creative]** Why should data preservation occur before some repair attempts even when the suspected problem is not related to storage?

191. **[Hard]** A computer shuts down under heavy workload. Compare overheating, power-supply limits, battery problems, and software failure as competing explanations.

192. **[Tricky]** Why does fan noise alone neither prove overheating nor prove healthy cooling?

193. **[Hard]** A user cleans dust from visible vents but performance under load does not improve. What evidence would justify inspecting internal cooling rather than continuing external cleaning?

194. **[Creative]** Design a test that distinguishes an uncomfortable monitor height from text that is simply too small.

195. **[Hard]** Why should ergonomic adjustments be judged over time rather than only by immediate comfort?

196. **[Inference, Tricky]** A wrist rest reduces discomfort while resting but worsens typing posture. What does this reveal about evaluating support equipment by task phase?

197. **[Hard]** A user develops neck strain only when working from a laptop away from home. Which differences between the temporary and permanent setups should be examined?

198. **[Creative]** Why might increasing display scaling solve both an eye-strain problem and a posture problem without moving the monitor?

199. **[Hard]** A company blocks all browser downloads to prevent malware. Analyze the security benefit and the usability cost using the layered-security model.

200. **[Tricky]** Why is “no browser warning appeared” weaker evidence of safety than “the browser displayed a warning” is evidence of danger?

201. **[Hard]** A site requests camera access for a function that appears unrelated to video. Which principle should guide the permission decision?

202. **[Inference, Creative]** Why can a legitimate website become dangerous temporarily without changing its familiar domain?

203. **[Hard]** A user receives an unexpected password-reset email but did not request a reset. Why should the user avoid both the reset link and complete inaction?

204. **[Tricky]** A bank confirms that an account alert is genuine. Why should the user still avoid returning to the original email link?

205. **[Hard]** A password is unique and strong but is entered into a phishing site. Which security property succeeded, and which failed?

206. **[Creative]** Explain why one-time codes are especially attractive to attackers even though they expire quickly.

207. **[Hard]** A user blocks tracking cookies but allows every application broad location access. Why is browser privacy only one part of the overall profile?

208. **[Tricky]** A recommendation system accurately predicts an interest the user never explicitly stated. Distinguish observation from inference.

209. **[Hard]** Why can an inaccurate advertising profile still affect a user even though it does not reflect reality?

210. **[Inference, Creative]** How might personalized recommendations gradually shape the data that later appears to confirm the original profile?

---

# Mixed-Topic Questions

211. **[Hard]** A cloud application is slow only when many browser tabs are open and Wi-Fi signal is weak. Explain how RAM pressure and network quality could combine rather than act independently.

212. **[Tricky]** A laptop on battery power reduces performance while running a cloud backup. Which relationships among power, CPU activity, network use, and battery life may explain this behaviour?

213. **[Creative]** A user moves a desktop into a closed cabinet to improve cable organization. Which maintenance, cooling, and reliability trade-offs were overlooked?

214. **[Hard]** A surge damages the router but not the computer. The computer still connects to the router through Ethernet but cannot reach websites. Use the layered model to explain the partial functionality.

215. **[Tricky]** A file stored in the cloud is unavailable because the browser is outdated. Why does remote storage still depend on local software compatibility?

216. **[Hard]** A student edits an assignment in a browser, closes the tab, and later finds that some work is missing. Analyze possible failures involving saving, synchronization, network access, and tab state.

217. **[Creative]** A user assumes a browser-based application consumes no local resources because it runs “in the cloud.” Use CPU, RAM, display, and network concepts to challenge this.

218. **[Hard]** A laptop is connected to an external monitor, keyboard, Ethernet, and power. In what ways has it become desktop-like, and in what ways does it remain a laptop?

219. **[Tricky]** A docked laptop has better network stability but worse overheating. Which benefits and constraints of the setup may be interacting?

220. **[Hard]** A user backs up files to a cloud service but uses a weak reused password. How can a good recovery design be undermined by poor account security?

221. **[Creative]** A ransomware attack encrypts local files and synchronized cloud copies, but an older version remains recoverable. Which service feature changed the outcome?

222. **[Hard]** A browser bookmark opens a phishing site because the bookmark was created during an earlier attack. Why does a usually safe navigation method become dangerous here?

223. **[Tricky]** A user reaches the correct domain through a bookmark but ignores a certificate warning. Which one security check succeeded, and which failed?

224. **[Hard]** A computer’s clock is incorrect, and secure websites begin showing certificate errors. Using only the system model, why might a local setting affect remote-site validation?

225. **[Inference, Creative]** A user sees targeted advertisements on a smart television after searching on a laptop. Construct two supported paths by which the devices could have been linked.

226. **[Hard]** A household blocks third-party cookies on computers but uses the same social-media account on phones, tablets, and television apps. Why may cross-device personalization remain effective?

227. **[Tricky]** An Ethernet-connected desktop and Wi-Fi-connected laptop both lose internet simultaneously, but local file sharing continues. Which common dependency most likely failed?

228. **[Hard]** A browser cannot download an operating-system update because storage is nearly full. Trace how a storage-maintenance issue becomes a security issue.

229. **[Creative]** A computer has updated antivirus but an outdated browser and operating system. Why does one current security layer not compensate for two outdated ones?

230. **[Hard]** A user avoids phishing links but installs an untrusted browser extension with permission to read every webpage. Which safe-browsing objective has been bypassed through another route?

231. **[Tricky]** A user disables all notifications to improve focus and misses a backup failure warning. What trade-off from notification management becomes visible?

232. **[Hard]** A cloud backup runs continuously on a slow cellular connection with a strict data limit. Analyze reliability, cost, and performance trade-offs.

233. **[Creative]** A traveller uses public Wi-Fi to access a correctly verified HTTPS cloud-storage domain. Which risks are reduced by HTTPS, and which account or device risks remain?

234. **[Hard]** A user’s cloud files are safe after a laptop is destroyed, but the only account-recovery code was stored on that laptop. Which dependency now prevents recovery?

235. **[Tricky]** A user copies important files to an external drive and then moves the originals to the Trash to free space. Under what mistake could this “backup” become the only copy?

236. **[Hard]** A user stores files on the desktop for convenience, synchronizes the desktop to the cloud, and assumes organization no longer matters. Why can poor file structure still cause errors?

237. **[Creative]** A browser’s address-bar suggestion opens an outdated bookmarked page instead of the intended current page. Which browser memory systems interacted?

238. **[Hard]** A user opens the same cloud document in multiple tabs and edits both versions. What kinds of conflict or overwriting risks may follow?

239. **[Tricky]** A macOS user closes a document window, assumes the application quit, and later notices high memory use. Which interface convention explains the surprise?

240. **[Hard]** A Windows user repeatedly launches an application because its minimized window is not visible. How do taskbar state and window state explain the duplicate instances?

# Unusual and Unpredictable Scenarios

241. **[Creative]** A computer has no keyboard, mouse, or screen but receives sensor data and controls a refrigerator’s temperature. Use the course’s model to explain why it is still a complete computing system.

242. **[Tricky]** A smart television can stream videos but cannot run a desktop word processor. Why does this limitation reflect specialization rather than insufficient “computer-ness”?

243. **[Hard]** A server successfully sends a webpage, but the client displays it incorrectly. Which parts of the system have demonstrated correct behaviour, and where could the remaining failure exist?

244. **[Inference, Creative]** A personal computer stores shared files for several family members. At what point does it begin functioning as a server, even if nobody calls it one?

245. **[Hard]** A user upgrades a desktop’s graphics card and then experiences shutdowns during demanding tasks. Which relationships among expansion, power supply, heat, and compatibility should be investigated?

246. **[Tricky]** A faster graphics card produces no improvement in a text-editing task. Why is this outcome consistent with specialized hardware roles?

247. **[Creative]** A laptop is easier to carry after the manufacturer removes ports and upgradeable parts. Explain how the same design change can be both an improvement and a limitation.

248. **[Hard]** A user buys a powerful CPU but installs too little RAM and a slow HDD. Why might the finished system fail to feel powerful during everyday multitasking?

249. **[Inference, Tricky]** A computer becomes responsive immediately after closing a large application, even though no files were deleted. Which temporary resource was probably released?

250. **[Hard]** A document remains visible after the storage drive disconnects unexpectedly. Why might the user still be able to view or edit it temporarily, and what risk remains?

251. **[Creative]** A power failure occurs between editing a file and saving it. Explain why the operating system, RAM, storage, and backup each play different roles in the outcome.

252. **[Hard]** A laptop’s external monitor works, but the built-in screen does not. What does this suggest about the CPU, operating system, and graphics processing path?

253. **[Tricky]** A monitor displays the operating-system startup screen but goes blank after login. Why does this evidence shift attention away from the physical cable alone?

254. **[Hard]** A desktop’s power button works, but USB devices receive no power. Which assumption about “the computer having power” should be reconsidered?

255. **[Creative]** A mouse works on paper but not on a shiny desk. Use the optical-sensor model to explain the difference without assuming hardware failure.

256. **[Hard]** A keyboard intermittently repeats keys after crumbs are removed. Why should both physical contamination and software causes remain under consideration?

257. **[Tricky]** A wireless keyboard responds during startup but not after the operating system loads. Which layer change is significant?

258. **[Hard]** A laptop charges through one USB-C cable but charges very slowly through another. Which cable, charger, and port capabilities may differ despite matching connectors?

259. **[Inference, Creative]** Why might an all-in-one computer reduce visible cable clutter while increasing the consequence of a display failure?

260. **[Hard]** A user wants to replace only the monitor in an all-in-one system. Which integration trade-off complicates this plan?

261. **[Tricky]** A desktop and laptop have the same advertised processor model, but the desktop performs better under sustained load. Which cooling and power factors could explain the difference?

262. **[Hard]** A laptop’s battery provides backup during an outage, but unsaved work is still lost later. Construct a plausible sequence consistent with the notes.

263. **[Creative]** A device continues operating during a blackout but loses internet access immediately. Which equipment likely had battery power, and which likely did not?

264. **[Hard]** A modem is functioning, but replacing the router restores household connectivity. What evidence distinguishes the failed roles of the two devices?

265. **[Tricky]** A router broadcasts an SSID but rejects the correct password after a security-setting change. Which compatibility issue might have been introduced?

266. **[Inference, Hard]** Why might enabling only WPA3 improve security for newer devices while preventing an older device from connecting?

267. **[Creative]** A guest connects successfully to Wi-Fi but cannot print to a household printer. Why could this be intentional rather than a fault?

268. **[Hard]** A wired desktop reaches the internet, but no wireless device can even see the SSID. Which router subsystem is most directly implicated?

269. **[Tricky]** A user places a router inside a closed metal cabinet for neatness and security. Why can this reduce both usability and network reliability?

270. **[Hard]** A home upgrades from cable to fibre but keeps an old router. Why may wireless performance remain almost unchanged?

271. **[Creative]** A family pays for faster internet but notices improvement only when several people stream simultaneously. What does this suggest about bandwidth and prior usage?

272. **[Hard]** A video call is clear in image quality but has noticeable conversational delay. Which network property is more suspicious than download bandwidth?

273. **[Tricky]** A file downloads quickly, yet cloud-document typing feels delayed. Why can these experiences depend on different performance characteristics?

274. **[Hard]** Cellular internet is fast near a window but unreliable elsewhere in the room. Which environmental dependency distinguishes it from a wired service?

275. **[Inference, Creative]** A user with limited cellular data enables continuous cloud backup. What hidden consequence could appear even if the backup works perfectly?

276. **[Hard]** A cloud service makes a file available everywhere but also propagates an accidental edit everywhere. Explain how convenience and risk arise from the same synchronization mechanism.

277. **[Tricky]** A cloud provider experiences an outage while all local devices and networks function normally. Why does storing files remotely create this new failure mode?

278. **[Hard]** A user can view cached cloud documents offline but cannot open a file never accessed before. What does this reveal about local and remote data availability?

279. **[Creative]** A backup account is secure, but the provider retains deleted files for only one day. Which recovery scenarios remain poorly protected?

280. **[Hard]** A user restores an older file version and unintentionally overwrites newer valid work. Why can recovery tools themselves require careful judgment?

---

# Reasoning About Human Behaviour

281. **[Hard]** Why are urgent phishing messages effective even when users know that phishing exists?

282. **[Tricky]** A user correctly identifies a suspicious sender address but clicks the attachment out of curiosity. Which gap exists between recognition and safe behaviour?

283. **[Creative]** A phishing email accurately imitates the user’s normal bank notifications. Which independent checks remain reliable when visual familiarity fails?

284. **[Hard]** Why does asking users to “be careful” provide weaker protection than designing a verification process with trusted routes?

285. **[Inference, Tricky]** Why might a person who understands the padlock symbol still fall for a secure phishing page?

286. **[Hard]** A user reuses one strong password across many services. Explain why strength and uniqueness solve different security problems.

287. **[Creative]** A scammer creates a message that contains no link but persuades the recipient to search for a fake support site. Why does “never click email links” alone fail as a complete rule?

288. **[Hard]** Why can a trusted bookmark reduce phishing risk while also creating false confidence if it is never rechecked?

289. **[Tricky]** A user receives a legitimate security email immediately after a phishing attempt. Why should both messages still be evaluated independently?

290. **[Hard]** Why is checking the sender’s displayed name weaker than checking the actual address and communication context?

291. **[Creative]** A spam message asks the recipient to reply with “STOP.” Explain why responding may provide the sender with useful information.

292. **[Hard]** Why should a spam folder be reviewed periodically but not treated as an ordinary inbox?

293. **[Tricky]** A legitimate newsletter uses remote images for layout and analytics. What privacy–usability trade-off appears when images are blocked?

294. **[Hard]** A user approves every browser permission request to avoid breaking websites. Why does convenience become a security and privacy weakness?

295. **[Inference, Creative]** Why may an interface that hides technical details improve usability while also making security decisions harder to understand?

296. **[Hard]** A user repeatedly postpones updates because the computer is needed for work. Explain how short-term productivity may increase long-term disruption risk.

297. **[Tricky]** Why might automatic updates be safer for an ordinary user but require more controlled deployment in an organization?

298. **[Hard]** A user installs several “optimization” and “security” tools from advertisements. Why can attempts to improve safety and performance create new risk?

299. **[Creative]** A person refuses all cloud services for privacy but keeps one unencrypted laptop as the only copy of important files. Evaluate the trade-off they have chosen.

300. **[Hard]** Why does privacy protection require considering applications, accounts, browsers, and devices rather than only cookies?

301. **[Tricky]** A user blocks personalized advertisements but continues receiving ads related to recent activity. Why does this not necessarily prove that the setting failed?

302. **[Inference, Hard]** How can a recommendation system influence future behaviour and then treat that influenced behaviour as evidence of existing preference?

303. **[Creative]** A household member sees recommendations generated from another member’s activity and begins interacting with them. How can this make an initially inaccurate profile appear increasingly accurate?

304. **[Hard]** Why should an inferred profile be treated differently from information explicitly provided by the user?

305. **[Tricky]** A person believes private browsing protects privacy because history disappears afterward. Which visible result encourages an overly broad conclusion?

306. **[Hard]** Why is the absence of local history compatible with continued logging by websites and network providers?

307. **[Creative]** A user organizes hundreds of browser tabs into many windows but never creates bookmarks. What problem has been rearranged rather than solved?

308. **[Hard]** Why may keeping an application running in the Dock or taskbar be convenient while also consuming resources?

309. **[Tricky]** A user closes windows to “clean the desktop” but does not quit the applications. Why may system resource use remain high?

310. **[Hard]** Why can memorizing interface positions make a user less adaptable after a software update?

---

# Systematic Troubleshooting Scenarios

311. **[Hard]** A desktop shows no signs of power. Create a test sequence covering wall power, surge protector, cable, power supply, and power button without skipping directly to component replacement.

312. **[Tricky]** A computer powers on only after the power cable is moved. What evidence would distinguish a loose connection from a failing power supply?

313. **[Hard]** A monitor intermittently loses signal when its cable is touched. Which hypothesis is stronger than a CPU failure, and how would you test it?

314. **[Creative]** A display works at low resolution but fails at a higher setting. Why might cable or port capability become relevant only under the more demanding configuration?

315. **[Hard]** A USB printer fails on one port but works on another. What conclusions are justified about the printer, cable, operating system, and original port?

316. **[Tricky]** A USB device works after restart but later disconnects again. Why does the temporary fix not identify whether the cause is hardware or software?

317. **[Hard]** A Bluetooth mouse pairs successfully but its pointer remains motionless. Which distinction between connection establishment and usable input should guide diagnosis?

318. **[Creative]** A user replaces a mouse because the pointer jumps, but the new mouse behaves identically. Which environmental factor should now receive greater attention?

319. **[Hard]** A keyboard works in one application but not another. Why does this scope suggest an application-level issue rather than a physical keyboard failure?

320. **[Tricky]** Some keys fail everywhere, while an external keyboard works normally. Which component becomes the leading suspect?

321. **[Hard]** A laptop overheats only when placed on a bed. Explain the evidence connecting surface choice to ventilation.

322. **[Creative]** A desktop runs cooler after being moved toward the front of a desk compartment. What hypothesis has this movement tested?

323. **[Hard]** A fan becomes louder after dust removal but temperatures decrease. Why is increased sound not necessarily evidence that cleaning caused damage?

324. **[Tricky]** A computer remains hot even after workload ends. Which cooling or airflow failures might explain slow heat removal?

325. **[Hard]** A system has adequate free storage but still cannot install an update. Which non-capacity causes remain possible within the notes’ layered model?

326. **[Creative]** A cleanup tool frees significant storage, but startup remains slow. What does this result reveal about the original bottleneck hypothesis?

327. **[Hard]** An HDD optimization completes successfully, but file access remains inconsistent and noisy. Why should hardware condition now be considered?

328. **[Tricky]** A system with an SSD reports that drive optimization is occurring automatically. Why should the user not assume harmful traditional defragmentation?

329. **[Hard]** Antivirus repeatedly detects the same threat after removal. Which persistence, reinfection, or unsafe-behaviour possibilities should be investigated?

330. **[Creative]** Malware is removed, but browser redirects continue. Why may browser configuration or extensions require separate examination?

331. **[Hard]** One browser opens a site correctly while another fails. Which system layers are probably healthy, and which become more suspect?

332. **[Tricky]** Every browser on one computer fails to open a site, but other sites work. Why does this pattern not prove that the remote site is down?

333. **[Hard]** One website works through cellular data but not through home internet on any device. Which common local or ISP-level factors become likely?

334. **[Creative]** A website loads its text but not images. Construct explanations involving browser settings, network transfer, remote resources, and blocked content.

335. **[Hard]** A cloud app works until the user attempts to save. Which distinction between reading data and writing data becomes important?

336. **[Tricky]** A shared cloud file opens but cannot be edited. Why is this consistent with correct connectivity and file availability?

337. **[Hard]** An external backup drive appears connected but new files are absent from the backup. Which difference between connection and configured backup behaviour matters?

338. **[Creative]** A backup job finishes unusually quickly after many files changed. Why should success be verified rather than assumed?

339. **[Hard]** A restored file opens but contains an older version than expected. Which backup-frequency and retention questions should be asked?

340. **[Tricky]** A cloud backup is current, but restoration is extremely slow. Why does backup success not guarantee rapid recovery?

---

# Cross-Platform and Interface Reasoning

341. **[Hard]** A Windows user expects closing the last macOS window to quit the application. Which cross-platform assumption causes the mistake?

342. **[Tricky]** A macOS user searches the Windows taskbar for application-specific menus. Why does understanding interface roles matter more than matching visual positions?

343. **[Creative]** A user removes an application icon from the Dock and later cannot find the app. Which search and launch tools could recover access without reinstalling it?

344. **[Hard]** A user deletes a desktop shortcut on Windows and an application icon from the macOS Dock. What similar action occurred, and what did not occur?

345. **[Tricky]** Why can the same X-shaped control represent closing a tab, window, or application context depending on location?

346. **[Hard]** A user opens many browser windows because they do not understand tabs. Compare the effects on organization, navigation, and task switching.

347. **[Creative]** A researcher uses one window with dozens of tabs and loses track of source relationships. Which organizational weakness remains even though tab functionality is understood?

348. **[Hard]** Why may a browser’s Back button behave unexpectedly after a page dynamically changes without loading a visibly new website?

349. **[Inference, Tricky]** A user presses Back while completing a form and loses entered information. What does this reveal about navigation history versus saved application data?

350. **[Hard]** A browser history entry finds a page that a bookmark does not. Which differences in automatic and intentional recording explain this?

351. **[Creative]** A bookmarked page redirects to a new address. What should the user verify before updating the bookmark?

352. **[Hard]** A Spotlight search finds an application faster than Launchpad, while Finder is better for moving its related files. Explain the different search and management roles.

353. **[Tricky]** A file appears in Spotlight results but not in the folder where the user expects it. Why is the search result not evidence that Finder is malfunctioning?

354. **[Hard]** A notification appears while presenting sensitive work. Which notification settings balance awareness against privacy?

355. **[Creative]** A backup failure alert is hidden among promotional notifications. How can poor notification design indirectly increase data-loss risk?

356. **[Hard]** A full-screen application makes the user think other programs have closed. Which distinction between visibility and execution is being missed?

357. **[Tricky]** Natural scrolling feels correct on a trackpad but wrong on a mouse. Why might one user rationally prefer different movement models across devices?

358. **[Hard]** A gesture works in one application but not another. Which hardware, operating-system, and application dependencies should be separated?

359. **[Creative]** A user disables a gesture after accidental activation but becomes slower at navigation. What trade-off between discoverability, speed, and error rate appears?

360. **[Hard]** Why should interface proficiency be measured by the ability to locate functions in a changed layout rather than by speed in one familiar version?

# Design and Decision-Making

361. **[Hard]** A buyer wants maximum portability, long battery life, a large screen, powerful cooling, and easy upgrades. Which goals conflict, and which compromises are unavoidable?

362. **[Tricky]** Why might a less powerful laptop be more suitable than a high-performance desktop for a user whose main constraint is location?

363. **[Creative]** Design a computer setup for someone who changes workspaces daily but needs a stable, ergonomic environment whenever at home.

364. **[Hard]** A user primarily stores large media archives but wants fast application startup. How could HDD and SSD strengths be combined rather than choosing only one?

365. **[Inference, Tricky]** Why might a computer with replaceable parts cost less to maintain over time even if its initial price is higher?

366. **[Hard]** A user wants an all-in-one because it is compact but also expects to replace individual components easily. Which expectation should be reconsidered?

367. **[Creative]** A school computer lab values easy repair, standardized components, and fixed workstations. Which computer form best aligns with those priorities, and why?

368. **[Hard]** A field worker values durability, battery operation, and minimal cables. Which design priorities become more important than internal expansion?

369. **[Tricky]** Why does selecting a computer by processor model alone ignore system-level performance?

370. **[Hard]** Compare two systems: one has a fast CPU and slow storage; the other has a modest CPU and fast storage. Which may feel faster during ordinary use, and why is the answer task-dependent?

371. **[Creative]** A user wants to upgrade graphics performance but has a weak power supply and limited ventilation. Why can a technically compatible expansion slot still be insufficient?

372. **[Hard]** When choosing an external monitor for a laptop, why must the user evaluate both connector compatibility and supported display capability?

373. **[Tricky]** A cable physically fits both devices. Why does that establish mechanical compatibility but not necessarily functional compatibility?

374. **[Hard]** A workspace has limited power outlets. Which components require power, and how should surge protection and cable safety influence the arrangement?

375. **[Creative]** Design a setup that minimizes the effect of a short power outage on a desktop user’s unsaved work and internet access.

376. **[Hard]** Why must a backup-power plan include the monitor and network equipment rather than only the computer case?

377. **[Tricky]** A desktop is protected by a UPS, but the router is not. Which activities may continue during an outage, and which may fail?

378. **[Hard]** A user chooses Wi-Fi solely to remove cables. Which performance, security, and environmental costs should be considered?

379. **[Creative]** Design a hybrid home network for fixed high-demand devices, mobile devices, visitors, and one device without wireless hardware.

380. **[Hard]** A household must choose between expensive fibre and cheaper cable. Which usage patterns would justify the additional cost?

381. **[Tricky]** Why might the highest advertised internet speed offer little benefit to a single user performing lightweight tasks?

382. **[Hard]** A household frequently uploads large projects and joins video calls. Why is comparing only download speed insufficient?

383. **[Creative]** A rural user has unreliable high-speed cellular service and stable slower DSL. Develop a decision rule based on task sensitivity rather than headline speed.

384. **[Hard]** Why should ISP choice include availability, reliability, cost, latency, and data limits rather than only connection technology?

385. **[Tricky]** Two ISPs use the same technology but provide different real-world performance. Which factors beyond the technology label could explain the difference?

386. **[Hard]** A user buys a router with advanced Wi-Fi capability but connects it to a slow internet plan. Which tasks may improve and which will remain limited?

387. **[Creative]** Explain how a faster router could improve local file transfers without improving external download speed.

388. **[Hard]** A user wants maximum cloud convenience with minimal privacy risk. Which trade-offs must be consciously managed rather than completely eliminated?

389. **[Tricky]** Why can refusing synchronization improve protection against propagated mistakes while reducing convenience?

390. **[Hard]** A project team needs simultaneous editing and reliable recovery from accidental changes. Which combination of collaboration and backup features is required?

---

# Cause-and-Effect Chains

391. **[Hard]** Trace how dust accumulation can eventually lead from restricted airflow to reduced CPU performance.

392. **[Tricky]** How could blocked ventilation cause an application to appear slow even though neither the application nor its files are defective?

393. **[Creative]** Construct a chain in which poor cable placement leads indirectly to data loss.

394. **[Hard]** Trace how low storage space can progress from a minor inconvenience to a security problem.

395. **[Tricky]** How can failing to restart after a browser update leave a user exposed despite the update appearing installed?

396. **[Hard]** Trace how one reused password can turn the compromise of a minor account into access to cloud backups and other critical data.

397. **[Creative]** Build a chain showing how a phishing email with no malware can eventually cause permanent file loss.

398. **[Hard]** How can enabling remote email images lead from a single message opening to increased spam?

399. **[Tricky]** Trace how clicking a legitimate-looking link can bypass the protection offered by a strong password.

400. **[Hard]** Explain how an outdated operating system can weaken otherwise current applications and security tools.

401. **[Creative]** Construct a chain in which excessive browser tabs contribute to poor posture.

402. **[Hard]** How can small text lead indirectly to neck and back discomfort?

403. **[Tricky]** Trace how raising a chair to correct wrist position can create a lower-body ergonomic problem.

404. **[Hard]** How can a monitor positioned too far away cause both visual strain and poor posture?

405. **[Creative]** Construct a chain in which notification overload leads indirectly to failed backups going unnoticed.

406. **[Hard]** Trace how a misleading SSID could cause a user to connect to the wrong wireless network and expose activity.

407. **[Tricky]** How can using the same router administrator password and Wi-Fi password increase the consequences of one disclosure?

408. **[Hard]** Trace how weak router placement can affect cloud-application responsiveness without affecting the cloud provider itself.

409. **[Creative]** Build a cause-and-effect chain in which an inaccurate behavioural profile becomes more influential over time.

410. **[Hard]** How can synchronization turn an accidental local deletion into a multi-device loss?

411. **[Tricky]** Trace how a backup retention limit can turn a recoverable mistake into an unrecoverable one.

412. **[Hard]** How can a permanently connected backup drive increase convenience while increasing malware exposure?

413. **[Creative]** Construct a chain in which failing to test restore procedures causes longer downtime than the original hardware failure.

414. **[Hard]** Trace how poor file organization can increase the chance of backing up the wrong folders.

415. **[Tricky]** How can an incorrectly configured automatic cleanup rule undermine an otherwise sound backup strategy?

416. **[Hard]** Explain how a device can remain connected to Wi-Fi while a failure at the modem or ISP prevents all web access.

417. **[Creative]** Trace a webpage request from a mouse click through the operating system, hardware, router, ISP, server, and back to the monitor.

418. **[Hard]** How can a failure at any one stage of the client–server path produce a similar visible symptom to the user?

419. **[Tricky]** Why can two unrelated failures produce the same “page cannot be displayed” message?

420. **[Hard]** Trace how a cloud-service outage differs causally from a home-network outage, even when both prevent access to the same document.

---

# Assumptions and Implications

421. **[Hard]** What hidden assumption is made when a user says, “The computer is slow, so the CPU must be weak”?

422. **[Tricky]** What assumption is being made when a user concludes that a device is not a computer because it lacks a keyboard?

423. **[Inference, Creative]** If a machine can be both client and server, what does this imply about computing roles in peer-to-peer interactions?

424. **[Hard]** What assumption is hidden in the statement, “The application is installed, so it must work on this operating system”?

425. **[Tricky]** What does a program’s failure on one OS but success on another imply about software compatibility layers?

426. **[Hard]** A user believes a larger drive will make multitasking easier. Which resource relationship has been assumed incorrectly?

427. **[Creative]** What does the existence of external laptop peripherals imply about the boundary between portable and desktop computing?

428. **[Hard]** If a laptop can function as a desktop-like workstation, why do its internal upgrade limits still matter?

429. **[Tricky]** What assumption is made when someone expects an all-in-one to be as modular as a tower desktop?

430. **[Hard]** If a router can support local communication during an internet outage, what does this imply about the independence of local and wide-area networking?

431. **[Inference, Creative]** If several devices fail simultaneously while local networking still works, what does that imply about their shared dependencies?

432. **[Hard]** What assumption is hidden in choosing a cellular plan based only on the label “5G”?

433. **[Tricky]** What does inconsistent cellular performance imply about the relationship between technology name and environmental conditions?

434. **[Hard]** A user assumes fibre will eliminate all latency. Which network property is being confused with bandwidth?

435. **[Creative]** What does an ISP’s advertised “up to” speed imply about variability and guarantees?

436. **[Hard]** If a cloud application supports offline work, what does this imply about the division of data and processing between local and remote systems?

437. **[Tricky]** What assumption is made when a user believes cloud files are protected from every form of deletion?

438. **[Hard]** If synchronization can propagate both useful edits and mistakes, what does this imply about automation more generally?

439. **[Inference, Creative]** Why does version history change the risk profile of collaborative editing?

440. **[Hard]** What assumption is hidden in saying, “The backup completed, so recovery is guaranteed”?

441. **[Tricky]** If a backup is encrypted but account recovery is weak, what does this imply about the weakest-link nature of security?

442. **[Hard]** A user trusts a webpage because it looks familiar. Which assumption about visual identity is unsupported?

443. **[Creative]** What does the existence of encrypted phishing sites imply about the difference between confidentiality and authenticity?

444. **[Hard]** If antivirus cannot prevent voluntary disclosure of a password, what does this imply about the boundary between technical and human security controls?

445. **[Tricky]** What assumption is made when a user believes that a familiar sender name verifies an email’s origin?

446. **[Hard]** If a recommendation system predicts interests from behaviour, what does this imply about privacy even when no sensitive fact is explicitly entered?

447. **[Inference, Creative]** Why can inferred information sometimes be more revealing than individual collected data points?

448. **[Hard]** What assumption is hidden in believing that blocking one tracking method produces anonymity?

449. **[Tricky]** If deleting cookies breaks a login session, what does that imply about legitimate uses of browser storage?

450. **[Hard]** A user assumes the disappearance of local browser history means the activity is forgotten everywhere. Which systems are being incorrectly treated as one?

---

# Exceptions, Boundaries, and Qualifications

451. **[Hard]** Under what circumstances can closing a window terminate an application, and why can this not be assumed universally?

452. **[Tricky]** When might minimizing an application fail to reduce its CPU or network activity?

453. **[Hard]** Why can a browser tab continue consuming resources even when it is not currently visible?

454. **[Creative]** Under what circumstances could closing a tab improve system performance but interrupt an important process?

455. **[Hard]** When is a desktop shortcut deletion harmless, and when could deleting a desktop item remove the actual file?

456. **[Tricky]** Why does the meaning of an icon depend on whether it represents a shortcut, application, folder, or file?

457. **[Hard]** Under what conditions may a browser bookmark provide a safer route than search, and under what condition can it preserve a dangerous route?

458. **[Creative]** When might browser history be more useful than bookmarks despite being less intentional and less organized?

459. **[Hard]** Why may a browser’s Back button not reproduce the exact prior state of a dynamic webpage?

460. **[Tricky]** Under what circumstances can a reopened tab fail to restore authentication, form contents, or page state?

461. **[Hard]** When is natural scrolling best understood as an accessibility or preference choice rather than a correctness issue?

462. **[Creative]** Under what conditions can multi-touch gestures reduce efficiency instead of increasing it?

463. **[Hard]** When may full-screen mode improve work, and when may it hide information required for the task?

464. **[Tricky]** Why can disabling all notifications reduce distraction while increasing operational risk?

465. **[Hard]** Under what circumstances might a spam-folder message deserve urgent attention without making the folder generally trustworthy?

466. **[Creative]** When could loading external email images be necessary despite the privacy cost?

467. **[Hard]** Why may a website without a padlock be acceptable for reading public information but unacceptable for submitting a password?

468. **[Tricky]** Under what conditions can a site with a padlock still be dangerous even when the domain appears correct?

469. **[Inference, Hard]** How could a legitimate site become compromised while preserving both its real domain and valid encryption?

470. **[Creative]** When might manually typing a URL be less safe than using a carefully verified bookmark?

471. **[Hard]** Under what conditions can an external drive be a strong backup, and what conditions make it weak?

472. **[Tricky]** When might a continuously connected backup be justified despite higher malware exposure?

473. **[Hard]** Why may a cloud backup be more resilient to local disasters but less convenient for very large restores?

474. **[Creative]** Under what circumstances could a local backup be preferable for recovery even when an off-site backup also exists?

475. **[Hard]** When may automatic cleanup be beneficial, and what categories of files require cautious rules?

476. **[Tricky]** Why is “never delete temporary files” as flawed as “delete every temporary file”?

477. **[Hard]** Under what conditions can an HDD remain preferable to an SSD despite lower speed and durability?

478. **[Creative]** When may adding RAM be the correct solution to slowness, and what evidence should exist first?

479. **[Hard]** Under what conditions can adding storage capacity solve a problem without improving performance?

480. **[Tricky]** Why must every performance upgrade be evaluated against the actual bottleneck rather than a generally desirable specification?

# Synthesis Across Hardware, Software, and Networks

481. **[Hard]** A computer can open local applications normally but cannot access any cloud service. Which layers have already demonstrated correct operation, and which remain uncertain?

482. **[Tricky]** A browser loads cached pages while the internet connection is down. Why can this create the illusion that external connectivity still exists?

483. **[Creative]** A user replaces a router to fix a browser problem, but only one browser had been failing. Which scope clue should have prevented the unnecessary replacement?

484. **[Hard]** A local document opens instantly, while an identical cloud document opens slowly. Which additional dependencies account for the difference?

485. **[Inference, Tricky]** Why can cloud applications increase the importance of browser maintenance compared with purely local applications?

486. **[Hard]** A web application crashes while the server remains available and other users continue working. Which client-side components may be responsible?

487. **[Creative]** A remote server returns correct data, but an outdated application cannot interpret it. What does this reveal about compatibility across system boundaries?

488. **[Hard]** A network request reaches the server, but the response never appears on-screen. Identify failure points between the server response and visible output.

489. **[Tricky]** Why can a problem that appears to be “the internet” actually originate in display, browser, operating-system, or account state?

490. **[Hard]** A file is downloaded successfully but cannot be opened. Which distinction between transfer success and application compatibility matters?

491. **[Creative]** A browser downloads an application installer that is correct for another operating system. Which stages worked correctly, and where does the process fail?

492. **[Hard]** A laptop connects to a server through Ethernet but loses access after switching to Wi-Fi. Which shared and non-shared layers should be compared?

493. **[Tricky]** A server is reachable by IP address but not by its usual name. Why does this narrow the problem without proving the server itself is healthy?

494. **[Inference, Hard]** Why can an application appear offline when only its authentication service is unavailable?

495. **[Creative]** A cloud document opens but embedded media does not. Construct a layered explanation involving separate servers or resources.

496. **[Hard]** A user can browse websites but cannot send email through an application. Why does working web access fail to prove that every internet service is functioning?

497. **[Tricky]** A browser accesses a site while another application cannot reach the same service. Which differences in permissions, configuration, or software paths may explain this?

498. **[Hard]** A desktop has working local storage but cannot save to a network folder. Which storage and networking concepts must be separated?

499. **[Creative]** A file appears to save successfully to the cloud but later exists only on one device. What local-cache or synchronization failure could explain this?

500. **[Hard]** Why is “the file exists somewhere” insufficient evidence that a backup strategy is complete?

---

# Small Details with Large Consequences

501. **[Moderate]** Why can monitor input selection matter even when every cable is physically connected correctly?

502. **[Hard]** A user connects headphones to the correct-looking port but receives no sound. Why should labels, system output selection, and device power all be checked?

503. **[Tricky]** Why may colour-coded audio ports be helpful without being reliable enough to use as the only identification method?

504. **[Creative]** A USB speaker is connected and powered, but the operating system still sends audio through the monitor. Which hidden software choice overrides the physical setup?

505. **[Hard]** A wireless device uses a USB receiver but is mistakenly configured as Bluetooth. What conceptual difference caused the failure?

506. **[Tricky]** Why can losing a tiny wireless receiver make an otherwise functional keyboard unusable?

507. **[Hard]** A keyboard is plugged into a USB port on another peripheral. Under what conditions might this work, and why might it fail?

508. **[Creative]** A user assumes every monitor cable carries audio because HDMI does. Why is this generalization unsafe across connection types?

509. **[Hard]** A laptop supports an external monitor but not at the monitor’s highest resolution. Which capability limit may exist despite successful basic connection?

510. **[Tricky]** Why can an adapter solve connector-shape differences while failing to solve signal-format differences?

511. **[Hard]** A user disconnects an older PS/2-style device while the computer is running. Why might legacy interfaces require different handling from USB?

512. **[Creative]** A computer contains several ports that look unused. Why is appearance alone insufficient for deciding what hardware can be added?

513. **[Hard]** A user assumes a disc drive can write discs because it can read them. Which capability distinction should be verified?

514. **[Tricky]** Why may an older peripheral remain usable while still being impractical compared with a modern USB or wireless replacement?

515. **[Hard]** A surge protector’s indicator light is off, but connected devices still receive power. Why should this not be interpreted as full protection?

516. **[Creative]** A user places a desktop on thick carpet. Which hidden ventilation design could make this placement risky?

517. **[Hard]** Why can paper stacked near a case cause both cooling and fire-safety concerns without entering the computer?

518. **[Tricky]** A cleaning cloth is only slightly damp, but liquid still reaches the monitor edge. Why does direction of wiping matter?

519. **[Hard]** Why should compressed air be used in short bursts rather than as a continuous high-pressure stream?

520. **[Creative]** A fan spins rapidly while being cleaned with compressed air. Why could movement that looks harmless still be undesirable?

521. **[Hard]** A keyboard dries externally within hours after a spill. Why can reconnecting it still be unsafe?

522. **[Tricky]** Why does a sticky liquid create a different cleaning problem from plain water?

523. **[Hard]** A mechanical mouse is cleaned but remains inaccurate. Which moving parts besides the tracking ball might still contain debris?

524. **[Creative]** An optical mouse fails only on one desk surface. Why is changing the surface a more informative test than reinstalling software?

525. **[Hard]** Why can airflow remain poor even when all visible vents appear unobstructed?

---

# Data, Files, and Storage Reasoning

526. **[Hard]** A user moves a file into a different folder and expects disk performance to improve. Which distinction between logical organization and physical storage invalidates this expectation?

527. **[Tricky]** Why can two files with identical names exist without being the same file?

528. **[Creative]** A user edits the wrong copy of an assignment because duplicates exist in several folders. Which file-management habits would have prevented the confusion?

529. **[Hard]** A file appears in “Recent” but the user cannot locate its folder. Which tools can reveal its actual storage location?

530. **[Tricky]** Why is opening a file successfully not proof that its backup copy is current?

531. **[Hard]** A user copies a folder to an external drive but later discovers hidden or excluded files were omitted. What does this reveal about manual backup assumptions?

532. **[Creative]** A backup includes documents but excludes application settings. Which recovery goals can still be met, and which cannot?

533. **[Hard]** A user renames a file extension to make it open in another application. Why does changing the name not necessarily change the data format?

534. **[Tricky]** A file opens in one application but not another that claims to support the same type. Which compatibility or corruption possibilities remain?

535. **[Hard]** A file is stored on an SSD and synchronized to the cloud. Which risks are reduced by each location, and which risks remain shared?

536. **[Creative]** A user has three copies of a file, but all are synchronized mirrors. Why may the apparent redundancy provide less protection than expected?

537. **[Hard]** A backup preserves only the newest version. Which mistakes remain difficult to recover from?

538. **[Tricky]** Why does version history help with accidental edits but not necessarily with every kind of account loss?

539. **[Hard]** A user restores an entire folder to recover one file and overwrites newer files. Which restoration-scope decision caused the secondary damage?

540. **[Creative]** A cloud provider preserves deleted files, but the user cannot sign in. Which recovery dependency now dominates the technical storage safeguards?

541. **[Hard]** Why should backup account credentials and recovery methods be protected separately from the files they safeguard?

542. **[Tricky]** A local backup is encrypted, but the decryption key exists only on the failed computer. Why is the backup operationally unusable?

543. **[Hard]** A user backs up every week but creates critical files daily. How should acceptable data-loss tolerance influence the schedule?

544. **[Creative]** A semester project changes rapidly near its deadline. Why might the appropriate backup frequency change over the project’s lifetime?

545. **[Hard]** A backup service runs continuously but consumes so many resources that the user disables it. Which design trade-off undermined reliability?

546. **[Tricky]** Why can a backup process succeed technically but fail as a practical system because it is too inconvenient?

547. **[Hard]** A user depends on browser history to recover important research sources. Which deletion, privacy, and synchronization events threaten that strategy?

548. **[Creative]** A bookmark saves the page address, while a local saved copy preserves content. Which future changes affect each differently?

549. **[Hard]** Why might a webpage disappear even though its bookmark remains intact?

550. **[Tricky]** A page remains in browser history but now shows different content. Why does history preserve location rather than historical truth?

---

# Security Under Ambiguity

551. **[Hard]** An email is expected, uses the correct sender address, and contains a link to the correct domain. Which risks still remain before opening an attachment?

552. **[Tricky]** Why should an unexpected attachment from a known contact still be treated cautiously?

553. **[Creative]** A trusted contact’s account is compromised and sends a believable message. Which verification practices remain useful when the sender identity is technically genuine?

554. **[Hard]** A website requests a password immediately after a browser update. Why should timing alone not establish legitimacy?

555. **[Tricky]** A user reaches the correct domain but through an unsecured connection. Which security property is missing?

556. **[Hard]** A site uses encryption but requests information irrelevant to its stated function. Why should data minimization influence trust?

557. **[Creative]** A support message asks for a screenshot that may contain private information. Which review step should occur before sharing it?

558. **[Hard]** A browser warning says a download is uncommon rather than confirmed malicious. How should uncertainty affect the user’s decision?

559. **[Tricky]** Why is “not detected as malware” different from “verified safe”?

560. **[Hard]** A file comes from an official site but is incompatible with the user’s OS. Why is source trust separate from technical suitability?

561. **[Creative]** An antivirus scan finds nothing after a suspicious login event. Why should account-security actions still be considered?

562. **[Hard]** A phishing victim changes the stolen password but not reused passwords on other services. Which remaining attack path stays open?

563. **[Tricky]** Why can changing a password fail to remove an attacker who already has an active session?

564. **[Inference, Hard]** Why should account recovery settings be reviewed after phishing even when the password has been changed?

565. **[Creative]** A user enters a one-time code into a fraudulent page and immediately changes the account password. Which timing question determines whether compromise may already have occurred?

566. **[Hard]** A secure browser blocks known malicious sites, but a newly created fraudulent site is not blocked. Which limitation of reputation-based protection is visible?

567. **[Tricky]** Why can a browser’s domain highlighting feature reduce error without replacing careful reading?

568. **[Hard]** A user recognizes the correct registered domain but overlooks a misleading path after it. When can the path still matter?

569. **[Creative]** A legitimate site hosts user-generated content containing a deceptive link. Why is trusting the main domain insufficient for every action within the page?

570. **[Hard]** A shortened URL comes from a trusted person. Why is destination uncertainty still relevant?

571. **[Tricky]** Why can copying and pasting a visible URL be unsafe if the displayed text differs from the actual link target?

572. **[Hard]** A browser extension changes search results to insert advertisements. Which boundary between browser functionality and third-party software has been crossed?

573. **[Creative]** A privacy extension requests permission to read and modify all website data. Why can a tool designed to reduce tracking become a tracking risk itself?

574. **[Hard]** A user keeps software updated but installs applications only from random download sites. Which security layer is being undermined?

575. **[Tricky]** Why is an operating system’s built-in antivirus not evidence that every installed application is trustworthy?

---

# Ergonomics in Real Workflows

576. **[Hard]** A user’s wrists are neutral, but shoulders remain raised while typing. Which part of the setup is still misaligned?

577. **[Tricky]** Why can correct wrist angle coexist with poor overall posture?

578. **[Creative]** A chair supports the lower back well but prevents the feet from reaching the floor. Which adjustment or accessory resolves the trade-off?

579. **[Hard]** A user moves the monitor closer to read small text and later develops neck discomfort. Which alternative adjustment would preserve distance?

580. **[Tricky]** Why can increasing text size be more effective than changing chair posture?

581. **[Hard]** A user places a second monitor far to the side and frequently reads from it. What repetitive strain pattern may result?

582. **[Creative]** How should monitor placement change when one screen is used continuously and another only occasionally?

583. **[Hard]** A laptop user adds an external keyboard but leaves the laptop flat on the desk. Which ergonomic conflict remains unresolved?

584. **[Tricky]** A user raises the laptop screen but continues using its built-in touchpad. Why might shoulder or arm strain persist?

585. **[Hard]** A user works in a dark room with a bright monitor and Night Mode enabled. Why can eye discomfort remain likely?

586. **[Creative]** A monitor’s automatic brightness repeatedly changes during work. How could a feature intended for comfort become distracting?

587. **[Hard]** A user follows the 20-20-20 rule but never changes body position. Which risks remain?

588. **[Tricky]** Why can one long exercise session fail to compensate for uninterrupted static posture during the workday?

589. **[Hard]** A standing-desk user experiences foot and lower-back discomfort. Which principle about movement and neutral posture applies?

590. **[Creative]** A user’s workspace is visually tidy but requires repeated reaching for frequently used items. Why is it aesthetically organized but ergonomically poor?

591. **[Hard]** A cable is routed safely away from feet but pulls tightly against a laptop port. Which physical risk remains?

592. **[Tricky]** Why can an ergonomic accessory create new problems if it is added without adjusting the rest of the setup?

593. **[Hard]** A user experiences pain only after several hours, not immediately. Why does this pattern support an ergonomic cause?

594. **[Creative]** Design an observation-based test to determine whether eye strain is caused mainly by brightness, glare, distance, or lack of breaks.

595. **[Hard]** Why should persistent pain trigger reassessment rather than repeated small posture corrections alone?

---

# High-Level Integrative Challenges

596. **[Hard]** Design a complete mental model for safely creating, editing, storing, synchronizing, backing up, and recovering an important document.

597. **[Tricky]** In that document workflow, which single failure could bypass both local saving and cloud synchronization?

598. **[Creative]** Design a computer setup for a student that balances portability, ergonomics, backup, security, and reliable networking.

599. **[Hard]** Which parts of that setup protect availability, which protect confidentiality, and which protect physical health?

600. **[Tricky]** Why can improving one dimension—such as convenience, portability, personalization, or automation—create new risks in another?

# Evaluation and Prioritization

601. **[Hard]** A computer has several minor problems: low free storage, an outdated browser, dusty vents, and no recent backup. In what order should they be addressed, and which ordering assumptions affect the decision?

602. **[Tricky]** Why might the most visible problem not be the most urgent problem?

603. **[Creative]** A user can afford either a larger SSD, more RAM, or an external backup drive. What evidence should determine the purchase rather than choosing the most impressive specification?

604. **[Hard]** A laptop is slow, overheats, and has little free storage. How would you determine whether these symptoms share one cause or represent separate problems?

605. **[Tricky]** Why should data backup sometimes happen before performance troubleshooting?

606. **[Hard]** A user wants to improve computer security but currently has weak passwords, delayed updates, no backups, and unsafe email habits. Which changes reduce likelihood of compromise, and which reduce its consequences?

607. **[Creative]** Rank convenience, reliability, privacy, security, and portability for a user who works with sensitive files while travelling. How would the ranking influence equipment and service choices?

608. **[Hard]** A family has a limited budget for either faster internet or better home Wi-Fi equipment. What observations would reveal which upgrade addresses the actual bottleneck?

609. **[Tricky]** Why might buying a faster internet plan worsen disappointment without changing performance in a distant room?

610. **[Hard]** A user has one recent cloud copy and several old local copies of a project. How should they judge which copy is most valuable during recovery?

611. **[Creative]** A backup plan is secure but difficult to use. At what point does inconvenience become a reliability weakness?

612. **[Hard]** A user receives many legitimate notifications but misses critical alerts. How should notification priority be designed around consequence rather than frequency?

613. **[Tricky]** Why can disabling low-value notifications improve the reliability of high-value notifications?

614. **[Hard]** A user is deciding whether to keep a laptop application installed or use its web version. Compare offline access, local resource use, updates, compatibility, and cloud dependence.

615. **[Creative]** When might an older but stable computer be preferable to a newer device with limited ports and repairability?

616. **[Hard]** A user wants to reduce online tracking but also wants persistent logins and personalized recommendations. Which controls permit partial reduction rather than an all-or-nothing choice?

617. **[Tricky]** Why is a privacy decision often a trade-off among multiple conveniences rather than a single setting?

618. **[Hard]** A user can either place the router centrally in view or hide it in a cabinet. Which functional goal should take priority, and why?

619. **[Creative]** A workspace is comfortable but blocks the computer’s vents. Why must ergonomic design include the equipment’s physical needs as well as the user’s?

620. **[Hard]** A user has correct posture but works for five uninterrupted hours. Why is posture quality insufficient by itself?

---

# Counterfactual Reasoning

621. **[Hard]** What would change in everyday computing if RAM were non-volatile and preserved all active data after shutdown?

622. **[Tricky]** Even if RAM retained data without power, why would long-term storage still be necessary?

623. **[Creative]** If every application communicated directly with hardware without an operating system, which compatibility and usability problems would increase?

624. **[Hard]** If all computers had identical hardware, why could software compatibility problems still exist?

625. **[Tricky]** If a laptop had unlimited battery life, which laptop–desktop trade-offs would remain unchanged?

626. **[Creative]** If a desktop had a built-in battery and handle, which design limitations would still prevent it from functioning like a practical laptop?

627. **[Hard]** If every USB-C port supported all possible USB-C functions, which user mistakes would disappear, and which cable-related problems would remain?

628. **[Tricky]** If a monitor required no separate power cable, which troubleshooting checks would become unnecessary, and which would still matter?

629. **[Hard]** If Wi-Fi were perfectly stable and unaffected by distance, why might Ethernet still remain useful?

630. **[Creative]** If internet service never failed, which reasons for maintaining a local network would still exist?

631. **[Hard]** If cloud applications always supported offline access, which cloud dependencies would remain?

632. **[Tricky]** If synchronization never propagated deletions, what new inconsistency problems might appear?

633. **[Creative]** If cloud storage retained every historical version forever, which recovery problem would improve and which storage or privacy concerns might grow?

634. **[Hard]** If antivirus detected every piece of malware, why would phishing and account security still matter?

635. **[Tricky]** If every website used HTTPS, why would domain verification remain essential?

636. **[Creative]** If browser history could never be deleted, how would usability and privacy change?

637. **[Hard]** If cookies were completely unavailable, which legitimate website behaviours would need alternative mechanisms?

638. **[Tricky]** If every user rejected personalized advertising, why might other forms of behavioural profiling still continue?

639. **[Hard]** If computers never overheated, which physical-cleaning practices would still matter?

640. **[Creative]** If screens caused no eye strain, which ergonomic principles would remain necessary?

---

# Diagnosing Competing Explanations

641. **[Hard]** A computer freezes while saving a large file. Compare insufficient RAM, slow storage, application failure, and network delay as possible causes.

642. **[Tricky]** What observation would most strongly distinguish a local-storage save from a cloud save in the previous scenario?

643. **[Hard]** A laptop battery drains quickly during video calls. Separate the possible contributions of display brightness, CPU work, Wi-Fi use, and battery age.

644. **[Creative]** A laptop lasts much longer when disconnected from an external monitor. Construct explanations involving display output, workload, and power configuration.

645. **[Hard]** A desktop restarts during gaming but not during web browsing. Compare thermal, power, graphics, and software explanations.

646. **[Tricky]** Why does workload-specific failure make a loose monitor cable less likely as the primary cause of the restart?

647. **[Hard]** A browser becomes slow only after many hours of use. Compare accumulated tabs, memory usage, extensions, cached state, and network factors.

648. **[Creative]** A browser restart fixes the problem without restarting the operating system. Which layers does this evidence narrow?

649. **[Hard]** A website loads slowly on one device but quickly on another using the same Wi-Fi. Compare device performance, browser state, and account differences.

650. **[Tricky]** Why does identical network access not imply identical application performance?

651. **[Hard]** A cloud backup stalls only when several people stream video. What shared resource is likely constrained?

652. **[Creative]** A cloud backup is fast at night but slow during the day. Construct explanations involving household use, ISP congestion, and service-side demand.

653. **[Hard]** A computer cannot join Wi-Fi but works through Ethernet. Which components and settings have already been partly validated?

654. **[Tricky]** Why does working Ethernet fail to prove that the router’s wireless security settings are correct?

655. **[Hard]** A device connects to Wi-Fi but receives a different local-network experience from other devices. Compare guest-network isolation, permissions, and device configuration.

656. **[Creative]** A visitor can browse the internet but cannot discover household devices. Why is this evidence of deliberate network segmentation?

657. **[Hard]** A user reports that “the cloud deleted my file.” What local actions, synchronization rules, retention limits, and account events should be examined?

658. **[Tricky]** Why should remote deletion not automatically be attributed to the provider?

659. **[Hard]** A spam message appears to come from a real colleague and references a real project. Compare account compromise, spoofing, and leaked information as explanations.

660. **[Creative]** What independent evidence would distinguish a genuine urgent request from an attacker using accurate context?

---

# Concept Boundaries

661. **[Hard]** Where is the boundary between an operating system function and an application function when opening a file?

662. **[Tricky]** Why can the same action—such as printing—involve application, operating-system, hardware, and network responsibilities?

663. **[Creative]** A browser displays a downloaded document internally. Is it acting as a browser, a document viewer, or both? Justify by function.

664. **[Hard]** At what point does local storage become cloud storage from the user’s perspective, and why can the boundary be hidden?

665. **[Tricky]** A synchronized file exists locally and remotely. Why is it misleading to classify it as exclusively local or exclusively cloud-based?

666. **[Hard]** Where is the boundary between a modem and router in a combined device, and why does the distinction still matter for troubleshooting?

667. **[Creative]** A smartphone shares cellular internet with a laptop. Which device is acting as modem-like equipment, router-like equipment, and client?

668. **[Hard]** At what point does a personal computer become part of a network rather than merely an isolated device?

669. **[Tricky]** Why can one physical port carry power, data, and video while those remain conceptually separate functions?

670. **[Hard]** Where is the boundary between a hardware performance problem and a software-efficiency problem?

671. **[Creative]** An inefficient application uses all available CPU power. Why is the CPU simultaneously working correctly and participating in poor performance?

672. **[Hard]** At what point does ordinary personalization become privacy-sensitive profiling?

673. **[Tricky]** Why can the same collected data be harmless in one context and sensitive when combined with other data?

674. **[Hard]** Where is the boundary between useful email tracking and invasive surveillance?

675. **[Creative]** A newsletter records aggregate opening rates but not individual identities. How does this alter the privacy analysis?

676. **[Hard]** At what point does an ergonomic preference become a health-related requirement?

677. **[Tricky]** Why can two users need different monitor positions while both follow the same ergonomic principles?

678. **[Hard]** Where is the boundary between preventive maintenance and repair?

679. **[Creative]** Cleaning a fan restores normal cooling. Was the action maintenance or repair? Defend both interpretations.

680. **[Hard]** At what point does a backup copy become an archive rather than an active recovery tool?

---

# Misleading Evidence

681. **[Hard]** A computer starts quickly, so the user concludes that all hardware is healthy. Why is startup speed weak evidence about graphics, cooling, or battery condition?

682. **[Tricky]** A file opens correctly once. Why does this not prove the storage drive is reliable?

683. **[Creative]** A backup restores one test file successfully. Which broader recovery capabilities remain unverified?

684. **[Hard]** A website displays the expected company name in the browser tab. Why is this weaker evidence than the domain?

685. **[Tricky]** An email arrives at the exact time the user expects a delivery update. Why does timing increase plausibility without proving authenticity?

686. **[Hard]** A browser extension has many users. Why is popularity not sufficient evidence of safe permissions?

687. **[Creative]** Antivirus reports that the system is clean. Which account, privacy, and configuration risks may still remain?

688. **[Hard]** A computer’s fans are quiet. Why can this indicate either efficient cooling or a failed fan?

689. **[Tricky]** A router shows all status lights as normal. Why may internet service still be impaired?

690. **[Hard]** A Wi-Fi speed test is fast near the router. Why does this not establish whole-home performance?

691. **[Creative]** A laptop charges to 100%. Why does this not prove the battery has healthy capacity?

692. **[Hard]** A monitor looks visually clean. Why can airflow and internal dust still affect the overall computer?

693. **[Tricky]** A user feels no discomfort during the first hour at a desk. Why is this insufficient evidence of good ergonomics?

694. **[Hard]** A browser restores all tab titles after a crash. Why does this not prove that every task state was recovered?

695. **[Creative]** A cloud service reports that a file is synchronized. Which device, version, and permission questions remain?

696. **[Hard]** A website has existed for years. Why is age alone insufficient evidence that every current page or download is safe?

697. **[Tricky]** A password-reset message contains no link. Why can it still be part of a social-engineering attack?

698. **[Hard]** A contact confirms sending an attachment. Why should the file type and expected purpose still be checked?

699. **[Creative]** A computer problem disappears after cleaning. Why does this support but not prove dust as the sole cause?

700. **[Hard]** An application works after reinstalling. Which competing explanations besides corrupted application files remain possible?

---

# Strategic “What Would You Do?” Scenarios

701. **[Moderate]** You must submit an assignment in one hour, and the laptop begins overheating. What immediate actions preserve both the device and the work?

702. **[Hard]** You discover that your only project copy is in a synchronized folder and deletion recovery expires tomorrow. What should be secured first, and why?

703. **[Creative]** You receive a convincing account-warning email while travelling and cannot access your usual computer. Construct a safe response using independent verification.

704. **[Hard]** Your home internet fails during an important cloud-based task, but your files are partially available offline. How should you decide what work can continue safely?

705. **[Tricky]** A cloud document shows unsynchronized changes just before you must shut down. Why might closing the device immediately be risky even if the document is visible?

706. **[Hard]** A new router supports stronger security, but one essential older device cannot connect. How would you balance security, compatibility, and network separation?

707. **[Creative]** A user must share internet with guests frequently but has no guest-network feature. Which risks should be explained before sharing the main credentials?

708. **[Hard]** A backup drive begins making unusual sounds during a restore. Which priority—continued recovery or hardware preservation—should guide the next action?

709. **[Tricky]** Why can repeated attempts to read a failing drive increase risk even when the goal is data recovery?

710. **[Hard]** A browser update changes the interface just before an exam. How should conceptual knowledge of browser functions reduce disruption?

711. **[Creative]** You must teach a user to switch from Windows to macOS without overwhelming them. Which functional equivalences and non-equivalences are most important?

712. **[Hard]** A user’s computer is slow, but they cannot afford hardware upgrades. Which maintenance and workflow changes from the notes should be evaluated first?

713. **[Tricky]** Why should closing unnecessary applications be tested before recommending more RAM?

714. **[Hard]** A user wants to improve privacy without breaking essential websites. How should changes be introduced and evaluated?

715. **[Creative]** A browser permission is necessary for one video call but not afterward. What temporary-permission principle should guide the decision?

716. **[Hard]** A workspace causes wrist pain, but the user cannot replace the desk. Which adjustable elements should be used to redesign the relationship among chair, keyboard, and feet?

717. **[Tricky]** Why may adding a footrest solve a problem created by a correct chair-height adjustment rather than represent an unrelated accessory?

718. **[Hard]** A user can afford either a laptop stand or an external keyboard, but not both. Why might buying only one fail to solve the laptop ergonomic conflict?

719. **[Creative]** A user must work temporarily from a poor setup. Which habits can reduce risk when ideal equipment is unavailable?

720. **[Hard]** A system is working now, but its software is unsupported, storage is nearly full, and backups are untested. Why is present functionality a poor measure of future reliability?

# Deep Integration and Transfer

721. **[Hard]** A user understands every component individually but still struggles to troubleshoot. Why is knowledge of relationships more valuable than isolated facts?

722. **[Tricky]** Why can the same visible symptom—such as slowness—result from CPU load, insufficient RAM, storage delay, overheating, network congestion, or remote-server problems?

723. **[Creative]** Build a decision tree for distinguishing local-computer slowness from cloud-service slowness.

724. **[Hard]** A local application and a web application both freeze. Which shared layers become more suspicious than the applications themselves?

725. **[Tricky]** If only one web application freezes while other sites work, why should the home router not be the first suspect?

726. **[Hard]** A browser tab consumes large amounts of RAM while displaying data stored on a remote server. Why does remote storage not eliminate local memory use?

727. **[Creative]** Explain how a cloud application can shift some work away from the device while still depending heavily on local hardware.

728. **[Hard]** A computer with weak hardware feels responsive when using simple cloud tools but struggles with complex local software. Which distribution of processing explains this?

729. **[Tricky]** Why can a powerful computer still perform poorly when the remote service is overloaded?

730. **[Hard]** A remote service is fast, but the user’s browser renders it slowly. Which local bottlenecks remain possible?

731. **[Creative]** A student says, “My file is in the cloud, so my computer does not matter.” Construct a counterexample involving account access, browser compatibility, and local hardware.

732. **[Hard]** Why does a complete recovery plan require both access to backup data and a working environment capable of opening it?

733. **[Tricky]** A backup contains a file format unsupported by the replacement computer. Why is data preservation not identical to data usability?

734. **[Hard]** How can operating-system compatibility become a recovery problem even when every file was backed up successfully?

735. **[Creative]** A user switches operating systems and discovers that some applications are unavailable. Which distinction between files, applications, and platforms helps plan the transition?

736. **[Hard]** A document format is supported on both systems, but its original application is not. What determines whether the user can continue working?

737. **[Tricky]** Why can web applications reduce operating-system compatibility problems without eliminating them?

738. **[Hard]** A cloud service works in one browser but not another. What does this imply about standards, browser implementation, and application support?

739. **[Creative]** Design a cross-platform workflow that reduces dependence on one application without assuming every feature transfers perfectly.

740. **[Hard]** Why does using widely accessible formats improve resilience without guaranteeing identical appearance or behaviour?

---

# Trade-offs Hidden Inside Convenience

741. **[Hard]** A device integrates the monitor, storage, input hardware, and battery into one compact unit. Why does each convenience increase dependency on the whole device?

742. **[Tricky]** Why can fewer cables make setup easier while making individual component replacement harder?

743. **[Creative]** Explain how automatic synchronization can simultaneously reduce forgotten saves and increase the speed of accidental propagation.

744. **[Hard]** Why can automatic updates improve security while creating timing and compatibility concerns?

745. **[Tricky]** Why can automatic backup reduce user effort but encourage false confidence if failures are not surfaced clearly?

746. **[Hard]** A browser remembers logins, history, bookmarks, and suggestions. Which conveniences create the strongest privacy consequences on a shared device?

747. **[Creative]** A user disables all browser memory features for privacy. Which repeated tasks become less efficient?

748. **[Hard]** Why does a single sign-on account improve convenience while increasing the consequences of account compromise?

749. **[Tricky]** A user keeps one account signed in across every device. How does this simplify access while strengthening cross-device tracking?

750. **[Hard]** Why can a personalized interface become less useful when the underlying profile is wrong?

751. **[Creative]** A recommendation system hides unfamiliar content to improve relevance. Which intellectual cost may arise even when the recommendations are accurate?

752. **[Inference, Hard]** Why can convenience features gradually reduce a user’s awareness of where files, applications, and data actually reside?

753. **[Tricky]** A user edits files from “Recent” lists without knowing their folders. Why can this create confusion during backup or recovery?

754. **[Hard]** Why does a simple all-in-one interface sometimes hide distinctions that become important during troubleshooting?

755. **[Creative]** A user expects every connected device to configure itself automatically. Which manual checks remain important when automation fails?

756. **[Hard]** Why can wireless peripherals reduce cable clutter while adding batteries, pairing, interference, and receiver management?

757. **[Tricky]** A wired device is less portable but more predictable. Which dependency has been removed?

758. **[Hard]** Why can a guest network slightly increase setup complexity while greatly simplifying security boundaries?

759. **[Creative]** Explain how a strong default configuration can protect beginners but reduce flexibility for advanced users.

760. **[Hard]** Why is “more convenient” not equivalent to “more reliable”?

---

# Failure Containment

761. **[Hard]** How does separating guest devices from personal devices reduce the effect of one compromised device?

762. **[Tricky]** Why can network segmentation limit damage without preventing the original compromise?

763. **[Creative]** A synchronized folder contains both active work and archival copies. Why does this design allow one mistaken deletion to affect too much?

764. **[Hard]** How does keeping an offline or disconnected backup contain the effects of malware?

765. **[Tricky]** Why can an offline backup still fail if it is outdated?

766. **[Hard]** A user stores all account recovery information in one cloud account. What single point of failure has been created?

767. **[Creative]** A household uses one administrator password for the router, Wi-Fi, email, and cloud backup. Trace how one disclosure could spread across systems.

768. **[Hard]** Why does using distinct credentials reduce the blast radius of a compromise?

769. **[Tricky]** A user has unique passwords but stores them in an unprotected document. Which security property is missing?

770. **[Hard]** Why can local file organization help contain mistakes even when cloud synchronization is enabled?

771. **[Creative]** A project folder contains final work, temporary exports, and downloads. How can poor separation increase cleanup and backup errors?

772. **[Hard]** Why should cleanup rules target categories rather than indiscriminately deleting old files?

773. **[Tricky]** A backup retains historical versions but no deleted files. Which failure type is contained and which is not?

774. **[Hard]** A user restores from backup onto the same failing drive. Why does this fail to contain the original hardware risk?

775. **[Creative]** Explain how replacing a failed device before restoring data can reduce the chance of repeated loss.

776. **[Hard]** Why does a UPS contain the impact of short outages but not long outages or hardware failure?

777. **[Tricky]** A surge protector protects the computer but not the network cable entering the router. Why can electrical risk remain through another path?

778. **[Inference, Hard]** Why does physical separation between backup copies matter even when both are encrypted?

779. **[Creative]** A fire destroys an encrypted laptop and its encrypted external drive. Which security goal succeeded, and which availability goal failed?

780. **[Hard]** Why is resilience about preventing a single event from destroying every path to recovery?

---

# Observation and Evidence

781. **[Hard]** What evidence would distinguish low RAM from a slow storage drive during application startup?

782. **[Tricky]** Why does improvement after closing applications support, but not prove, a RAM-related bottleneck?

783. **[Creative]** Design a simple test to compare Wi-Fi weakness with ISP slowness using one wired and one wireless device.

784. **[Hard]** What observation would distinguish a router failure from a modem or ISP failure?

785. **[Tricky]** Why does local file sharing during an outage provide strong evidence about the router’s local-network functions?

786. **[Hard]** What evidence would distinguish an application crash from an operating-system-wide failure?

787. **[Creative]** A computer becomes unresponsive, but the mouse pointer still moves. What limited inference can be made from that observation?

788. **[Hard]** A monitor shows system menus but not one application’s content. Why does this narrow the failure away from the entire display system?

789. **[Tricky]** Why does successful output to an external monitor validate only part of the graphics path?

790. **[Hard]** What evidence would distinguish a defective keyboard from incorrect application shortcuts?

791. **[Creative]** A key works in a text editor but not in a game. Construct two software-side explanations.

792. **[Hard]** What observation would distinguish a dirty optical sensor from a failing wireless connection?

793. **[Tricky]** Why does testing a mouse on a different surface control one variable more cleanly than reinstalling its software?

794. **[Hard]** What evidence would distinguish weak Wi-Fi signal from incorrect authentication?

795. **[Creative]** A device repeatedly asks for the Wi-Fi password despite correct entry. Which security-mode or saved-configuration issues could explain this?

796. **[Hard]** What evidence would distinguish a remote-server outage from a local browser problem?

797. **[Tricky]** Why does testing the same site in another browser provide useful but incomplete evidence?

798. **[Hard]** What observation would distinguish synchronization delay from permission denial?

799. **[Creative]** A file is visible but read-only. Which evidence suggests access-control problems rather than connectivity failure?

800. **[Hard]** Why should a good diagnosis state both what the evidence supports and what it does not prove?

---

# Teaching and Explanation Challenges

801. **[Moderate]** How would you explain RAM and storage to a beginner without implying that they are interchangeable?

802. **[Hard]** Create an analogy for the operating system that preserves its role as intermediary without suggesting that it performs every task itself.

803. **[Tricky]** Why can analogies such as “CPU equals brain” become harmful when treated literally?

804. **[Creative]** Explain client and server roles using an analogy that allows the same device to play both roles.

805. **[Hard]** How would you explain the difference between Wi-Fi and internet access to someone who uses the terms interchangeably?

806. **[Tricky]** What example best demonstrates that a router can work while the internet is unavailable?

807. **[Creative]** Explain cloud storage without using the phrase “someone else’s computer,” while still preserving the physical-server model.

808. **[Hard]** How would you teach synchronization and backup so that deletion propagation is clearly understood?

809. **[Tricky]** Why should the explanation of backup include restoration rather than only copying?

810. **[Creative]** Explain HTTPS to a beginner without implying that encrypted sites are automatically trustworthy.

811. **[Hard]** How would you explain phishing as a trust attack rather than merely a fake-email problem?

812. **[Tricky]** Why should a phishing lesson include independent verification routes?

813. **[Creative]** Explain browser history, bookmarks, and tabs using one coherent mental model without merging their purposes.

814. **[Hard]** How would you teach the distinction between closing a window and quitting an application across Windows and macOS?

815. **[Tricky]** Why should interface teaching focus on functions before exact icon locations?

816. **[Creative]** Explain ergonomic neutrality without suggesting that users should remain motionless.

817. **[Hard]** How would you teach the 20-20-20 rule while clarifying that it does not address every physical risk?

818. **[Tricky]** Why should maintenance teaching separate cleaning, optimization, updating, and backup?

819. **[Creative]** Explain the troubleshooting principle “change one variable at a time” using a non-technical analogy.

820. **[Hard]** How would you explain that an inference can be reasonable without being certain?

---

# Multi-Constraint Scenarios

821. **[Hard]** A student needs portability, long battery life, strong performance, low cost, easy repair, and a large display. Which three constraints should be clarified first to avoid an impossible recommendation?

822. **[Creative]** Design two different valid setups for that student based on two different priority orderings.

823. **[Tricky]** Why can the same device be a good choice for one user and a poor choice for another despite identical specifications?

824. **[Hard]** A household wants maximum Wi-Fi coverage, minimal visible equipment, strong security, and no additional cabling. Which goals conflict?

825. **[Creative]** Propose a compromise that improves coverage without fully sacrificing appearance or security.

826. **[Hard]** A user wants continuous backup, minimal bandwidth use, instant recovery, no subscription cost, and off-site protection. Which goals cannot all be maximized simultaneously?

827. **[Tricky]** Why does a backup strategy require choosing acceptable costs in time, money, bandwidth, and complexity?

828. **[Hard]** A company wants all browser activity private, fully personalized, permanently synchronized, and accessible on every device. Which requirements conflict?

829. **[Creative]** Design a partial solution that preserves some personalization while limiting cross-device linkage.

830. **[Hard]** A user wants zero notifications but also immediate awareness of security and backup failures. What categorization strategy resolves the conflict?

831. **[Tricky]** Why is selective interruption better than either constant interruption or complete silence?

832. **[Hard]** A laptop user wants the screen at eye level, the keyboard at elbow height, no external accessories, and continuous long-session comfort. Which physical constraint makes this impossible?

833. **[Creative]** Identify the least disruptive compromise for a short session and a different one for an eight-hour session.

834. **[Hard]** A computer must be placed inside a small cabinet, remain quiet, and run demanding workloads. Which thermal constraints must be addressed?

835. **[Tricky]** Why can quieter operation conflict with cooling under sustained load?

836. **[Hard]** A user wants to block all trackers without breaking authentication, shopping carts, or embedded services. Why is site-by-site evaluation more realistic than a universal rule?

837. **[Creative]** Design a staged privacy configuration that allows problems to be identified and reversed.

838. **[Hard]** A user wants every application to update automatically but never wants interface changes during important work. Which scheduling or control trade-off is required?

839. **[Tricky]** Why can postponing all updates until a “convenient time” become equivalent to indefinite delay?

840. **[Hard]** A recovery plan must work after theft, account compromise, hardware failure, and accidental deletion. Why does no single copy or service cover all four perfectly?

---

# Final Synthesis Questions

841. **[Hard]** Which five distinctions from the notes are most important for preventing beginner misconceptions, and why?

842. **[Tricky]** Which concepts appear separate at first but become tightly connected during troubleshooting?

843. **[Creative]** Build one unified model that explains how a user action becomes a saved cloud file visible on another device.

844. **[Hard]** In that model, identify every point where data could be delayed, altered, lost, exposed, or made unavailable.

845. **[Tricky]** Which failures are prevented by maintenance, which are detected by security tools, and which are recovered through backups?

846. **[Hard]** Why is the distinction between prevention and recovery central across hardware care, cybersecurity, and data management?

847. **[Creative]** Identify one example where improving convenience increases risk in each of these areas: networking, storage, browsers, applications, and account access.

848. **[Hard]** Which trade-offs are fundamental physical constraints, and which are mainly consequences of configuration choices?

849. **[Tricky]** Why can a technically correct solution still be unsuitable when human behaviour is ignored?

850. **[Hard]** How do ergonomics, interface design, automation, and security all depend on understanding real user behaviour?

851. **[Creative]** Construct a scenario where the computer, network, and cloud service all work correctly, but the user still cannot complete the task.

852. **[Hard]** Construct the reverse: a scenario where the user follows the correct process, but one hidden system dependency fails.

853. **[Tricky]** Why should a troubleshooting explanation include uncertainty rather than present the first plausible cause as fact?

854. **[Hard]** What makes a troubleshooting step informative rather than merely hopeful?

855. **[Creative]** Design a test sequence that maximizes information gained while minimizing destructive changes.

856. **[Hard]** Why should backup, security, and maintenance be designed before failure rather than added only after an incident?

857. **[Tricky]** Which risks become harder to solve after the event has already occurred?

858. **[Hard]** Why is understanding dependencies more durable than memorizing specific products, versions, or button locations?

859. **[Creative]** Which mental models from the notes are most likely to remain useful even as computer technology changes?

860. **[Hard]** How does the complete course support the broader principle that reliable computing requires coordination among technology, environment, and human decisions?

---

# Unpredictable Final Challenges

861. **[Creative]** A computer passes every diagnostic test but remains unpleasant to use. Which non-performance factors from the notes could explain the failure?

862. **[Tricky]** A user has perfect posture, perfect backups, updated software, and strong passwords but still loses access to work. Construct a supported cause.

863. **[Creative]** A user has no malware, no hardware failure, and no network outage, yet a document is lost. Construct three distinct supported explanations.

864. **[Hard]** A website is genuine, encrypted, and functioning correctly, but using it still creates privacy risk. Explain how.

865. **[Tricky]** A file exists locally, remotely, and in backup, but no copy is useful. Construct a scenario involving format, permissions, or encryption.

866. **[Creative]** A computer becomes more secure after a change but less reliable for the user’s task. Give a supported example and explain the trade-off.

867. **[Hard]** A network becomes faster after a security change. Why should this unexpected result not automatically prove that weaker security caused the previous slowness?

868. **[Tricky]** A user solves a problem by restarting, but the same problem returns daily. Why is the restart a workaround rather than a diagnosis?

869. **[Creative]** A visually identical action has three different meanings: closing a tab, closing a window, and quitting an app. Explain how context determines the result.

870. **[Hard]** A user correctly follows a checklist but still fails because the checklist’s assumptions do not match the device. What does this reveal about rules and exceptions?

871. **[Tricky]** Why can a recommended default be correct for most users and still wrong for a particular case?

872. **[Creative]** A device is both easier and harder to use after an update. Explain how both can be true.

873. **[Hard]** A user’s privacy improves while their exposure to data loss increases. Construct a scenario supported by the notes.

874. **[Tricky]** A backup strategy becomes safer after one copy is made less convenient to access. Explain why.

875. **[Creative]** A network failure improves security temporarily. Construct a plausible explanation.

876. **[Hard]** A browser feature designed to save time causes the user to visit the wrong site. Which feature is involved, and what verification step was skipped?

877. **[Tricky]** A strong password creates false confidence and leads to account compromise. Explain the behavioural chain.

878. **[Creative]** A user reduces eye strain by changing a software setting rather than moving hardware. Which setting could do this, and why?

879. **[Hard]** A hardware problem is partly solved by a software change. Give a supported example and explain why the underlying issue may remain.

880. **[Tricky]** A software problem appears only when hardware becomes hot. Why does this make the hardware–software boundary difficult to diagnose?

881. **[Creative]** A local network remains useful after the internet disappears. List several supported tasks that could still continue and explain the common dependency they avoid.

882. **[Hard]** A cloud service protects data from device failure but increases dependence on identity verification. Why is this a transfer rather than elimination of risk?

883. **[Tricky]** A recommendation system improves relevance but reduces discovery. Explain the tension without assuming either outcome is always harmful.

884. **[Creative]** A user’s attempt to simplify their setup creates a single point of failure. Construct examples involving an all-in-one computer, one cloud account, and one backup location.

885. **[Hard]** A user follows every security warning but ignores physical maintenance. How could a non-malicious failure still cause severe data loss?

886. **[Tricky]** A user maintains hardware perfectly but ignores software updates. Why is physical reliability insufficient for secure operation?

887. **[Creative]** A computer is technically reliable, but the user repeatedly loses work. Which workflow and mental-model problems should be examined?

888. **[Hard]** Which notes-based principles would help distinguish a tool problem from a user-process problem?

889. **[Tricky]** Why can “works for me” be weak evidence when evaluating a setup for another user?

890. **[Creative]** Design one final scenario that requires reasoning about hardware, operating systems, applications, networking, cloud storage, security, backup, and ergonomics at the same time.

```END....```

