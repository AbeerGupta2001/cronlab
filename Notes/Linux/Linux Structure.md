- Linux used for personal computers, servers and even mobile devices
- An operating system (OS) is a software that manages all the hardware resources of the computer, facilitating communication between software applications and hardware components.

- **History**
	- Linux os started with the ==🟠Unix Operationg System== release by **Ken Thompson and Dennis Ritchie** (whom both worked for AT&T at the time) in **1970**.
	- Linux was a personal project started by Linus Torvalds (1991) a Finnish student.
	- Linux is available in 600 distros.

- **Philosophy**
	- Linux follows five core principles:
		- **Everything is a file**: 
			- All system resources including hardware devices, processes, and network connections are represented as files, allowing them to be read from and written to using the same standard tools and commands
		- **Small, single-purpose programs**:
			- It offers many different tools that we will work with, which can be combined to work together.
		- **Ability to chain program together to perform complex tasks**:
			- The integration and combination of different tools enable us to carry out many large and complex tasks.
		- **Avoid captive user interfaces**:
			- Linux is designed to work mainly with the shell, which gives the user greater control over the OS.
		- **Configuration data stored in a text file**:
			- An example of such a file is `/etc/passwd` file, which stores all users registered on the system.

- **Components**
	- **Bootloader**: A piece of code that runs to guide the booting process to start the OS.
	- **OS Kernel**: The kernel is the main component of an operating system. It manages the resources for system's I/O devices at the hardware.
	- **Daemons**: Background services are called "daemons" in Linux. Their purpose is to ensure that key functions such as scheduling, printing and multimedia are working correctly. These small programs load after we booted or log into to computer.
	- **OS Shell**: 
		- OS Shell or the command language interpreter (also known as command line) is the interface between the OS and the user.
		- This interface allows the user to tell the OS what to do.
		- Commonly used shell are fish, zsh, bash, tcsh etc.
	- **Graphics Server**: This provides a graphical sub-system (server) called "X" or "X server" that allows graphical programs to run locally or remotely on the X-windowing system.
	- **Window Manager**; Also know as a graphical user interface (GUI). There are many options, including GNOME, KDE, MATE, Unity and Cinnamon. 
	- **Utilities**: Applications or utilities are programs that perform particular functions for the user or another program.
- **Linux** **Architecture**:
	- **Hardware**: Peripheral devices such as the system's RAM, hard drive, CPU, and others
	- **Kernel**: 
		- The core of Linux OS
		- Function is to virtualize and control common computer hardware resources like CPU, allocated memory, accessed data, and others.
		- It gives each process its own virtual resources and prevents mitigates conflicts between different processes.
	- **Shell**: CLI (Command line interface)
	- **System** **Utility**: Makes available to the user all of the OS functionality.

### File System Hierarchy

|**Path**|**Description**|
|---|---|
|`/`|The top-level directory is the root filesystem and contains all of the files required to boot the operating system before other filesystems are mounted, as well as the files required to boot the other filesystems. After boot, all of the other filesystems are mounted at standard mount points as subdirectories of the root.|
|`/bin`|Contains essential command binaries.|
|`/boot`|Consists of the static bootloader, kernel executable, and files required to boot the Linux OS.|
|`/dev`|Contains device files to facilitate access to every hardware device attached to the system.|
|`/etc`|Local system configuration files. Configuration files for installed applications may be saved here as well.|
|`/home`|Each user on the system has a subdirectory here for storage.|
|`/lib`|Shared library files that are required for system boot.|
|`/media`|External removable media devices such as USB drives are mounted here.|
|`/mnt`|Temporary mount point for regular filesystems.|
|`/opt`|Optional files such as third-party tools can be saved here.|
|`/root`|The home directory for the root user.|
|`/sbin`|This directory contains executables used for system administration (binary system files).|
|`/tmp`|The operating system and many programs use this directory to store temporary files. This directory is generally cleared upon system boot and may be deleted at other times without any warning.|
|`/usr`|Contains executables, libraries, man files, etc.|
|`/var`|This directory contains variable data files such as log files, email in-boxes, web application related files, cron files, and more.|


