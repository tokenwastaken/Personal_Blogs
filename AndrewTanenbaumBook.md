# Modern Operating Systems 4th Edition--Andrew Tanenbaum

 
# Notes:
text interface-shell
GUI- graphical user interface
- A memory word is bytes of memory
- OS runs in kernel mode
- User mode cannot do I/O operations
- Shell-GUI are the lowest level of software, allows user to start other programs
- OS software cannot be changed
- Everything running in kernel mode is part of OS but some that do not are also part of it or at least associated with it(password system)
# HISTORY OF OPERATING SYSTEMS
- Babbage and Ada created the first computer.
- Plugboards were used to write data then punch cards were created.
- How the batch system worked was you first wrote code on paper then punch it into cards. Then hand it to operators in the input room.
- In batch system, each job was collected then read from first tape, output was written onto second tape . After each job next job started running.
- Batch system: Bring cards to 1401, which reads cards onto tapes, carry tape into 7094, compute, take output to 1401, print.
- Early 1960s 7094 was for scientific calculations, 1401 for printing, tape sorting mostly.
- OS/360 combined 7094 and 1401 into one but required technology that wasn't around at the time, resulting in many bugs. Even so 360 satisfied user needs fairly good.
- 7094 would stay idle for quite a long time so the had to do something to avoid expensive CPU be idle for this long. While one job would do I/O, other would use CPU. This kept CPU busy almost all the time.
- With spooling C, ability to read jobs into disks as soon as they were brought to the computer room, 1401s were no longer needed.
- Two major versions of UNIX, System V from AT&T and BSD from University of California at Berkeley.
- To make it possible to write programs that can run at any UNIX system IEEE developed the standard POSIX.
- PCs (initially called microcomputers) were similar to minicomputers, just not in price. Minicomputers allowed universities to have their own PCs but microprocessor chip made possible for all to have their own PCs.
- In 1974 Intel created 8080, the first 8-bit CPU but needed OS for it to run.
- Kildall created CP/M for 8080 with floppy disks. But Intel didn't want OS to be run with floppy disks so they refused.
- Kildall created his own company which became company leader for 5 years.
- Intel asked Gates for OS but Gates lead them again to Kildall which refused to make arrangement. Gates hired the creator of DOS and created MS-DOS then sell it to Intel.
- CP/M, MS-DOS were only input.
- Xerox PARC used GUIs.
- Jobs visited PARC and realized the potential of GUI.
- Jobs created Macintosh, user-friendly PC with MAC-OSX like BSD-UNIX
- Timesharing lets programs switch so quickly that we can use all in the same time.
- Multiprogramming is used to keep CPU all the time
- Windows had windows on top of MS-DOS for GUI.
- Win95 came with abilities more like an OS. Then came 98 then XP and so on.
# PROCESSORS/MEMORY/ I/0 DEVICES/BUSES/BOOTING
## Processor
- Stack pointer register points to the top of stack.
- CPU fetches, decodes then executes.
- Pipeline: while process n is being executed , n+1 is decoded and n+2 is fetched.
- Superscalar CPU does pipeline in parallel and holds data in buffer until there is room for execution. hardware handles the FIFO part.
## Memory
- Having a memory both fast and cheap isn't possible in todays standards so another approach is taken.
- Memory is a herarchy of layers. Top layer, higher speed/lower capacity/greater cost per bit.
- Registers are as fast as CPU <1kB
- Register->Cache->RAM->Disk
- When a program needs to read a memory word, hardware checks if the line is in cache, if yes, cache hit if not cache miss, thus request is sent to main memory.
- L1 cache ( built onto microprocessor and is for quick memory access) vs L2(placed on  other chip) cache is timing. L1 is smaller but faster,  L2 cache has delay of one-two clock cycles and is for recently used memory words.
- EEPROM( electrically erasable ROM) is nonvolatile(data not erased when electricity is cut) but can be erased and rewritten. Mostly used as ROM for bug detection.
- CMOS memory for date and time.
- Disks are larger than RAM thus slower.
- SSD are not disks but are flash memory to hold lots of data.
- Virtual memory is to run programs larger than physical memory. By placing programs on the disk and using RAM as cache for most executed parts of program.
## Buses
- A shared bus architecture means that multiple devices use the same wires to transfer data.
- Parallel bus architecture means you send data through multiple wires.
- When you have multiple data you need to decide which data will use the bus. PCIe bus invented by Intel does this by using point to point connections on a serial bus architecture.
- When two network cards use same interrupt it leads to misconceptions. Solution is to inform user to set other interrupts that don't conflict.
## I/O Devices
- I/O devices have device drivers and controllers in them as sort of an interface for OS to communicate with.
- Any device connected to the computer is connected by a plug and socket, and the socket is connected to a device controller.
- Device driver is just a code which helps devices be compatible , recognizable and lets controller communicate with OS.
## BOOTING
- Motherboard has BIOS, low level I/0 software that has instructions on what to load at boot.
- BIOS checks devices attached using PCIe.
# OPERATING SYSTEM ZOO
## -OS Types:
+ Mainframe (room sized computers with  very high I/O capacity. They are created to process many jobs at once. They do batch, transaction processing, and timesharing. OS/390 is  an example but Linux is used nowadays.)
+ Server (They run on servers, which are either very large personal computers, workstations, or even mainframes.)
+ Multiprocessor with multiple CPU
+ PC
+ Handheld(PDA)
+ Embedded
+ Sensor-node (detects fire, forecast, guards national borders. Communicate using wireless communication. They are real computers, have event driven OS, responding to external events or making periodical measurements, TinyOS)
+ Real time OS (guarantees certain action occurs in certain time, soft real-time system can accept some misses. Digital audio or smartphones.)
+ Smart Cards OS (credit card sized devices with CPU chips. Can only do single function like digital payment.)
# OPERATING SYSTEM CONCEPTS
- Two main functions:  providing abstractions to user programs and managing the computer’s resources.
- Each process is associated with an address space, a list of memory locations which process can read and write.
- Address space contains executable program, program's data and its stack. Briefly all the memory process can address.
- Address space is for processes in the memory not interfere with each other
- A process is a container that holds all the data needed to run program.
- Process table holds the latest state of process for later execution. Process table does not hol address space in itself. Process table consists of Process control blocks (PCB) which hold the data( PC counter(next process to be executed), registers,  process state and process number) about processes.
- A suspended process consists of address space and process table entry.
- Interprocess communication is between child and parent processes.
- process and file hierarchies ar both trees, process trees being short lived.
- /home/isken/Downloads first / means home is root.
- Special files are provided in order to make I/O devices look like files.
- Block special files are for disks, character special files are for devices like printer that accept or output stream.
- Special files are located in /dev directory
- Pipes connect two processes, unidirectional.
- files have protection bits rwxr-x--x(first 3 are for user, following 3 are for other groups, last 3 are for those not part of anything former)
# SYSTEM CALLS
- cat file1 file2 file3 | sort > /dev/lp & ( puts files together, sorts them, output is redirected to file, usually printer. putting & at the end immediately pops prompt and command is ran in background.)
- 7094 had roughly over 128kB memory even OS was in assembly to save memory.
- 7094 had no protection, 360 could hold several programs in memory at the same time and let them take turns running.
- 8080 CPU had no protection so again monoprogramming.
- 80286 made multiprogramming possible.
- CDC 6600 in 1964 was the fastest computer for years.
- When microcomputers came out CP/M was leading the market but it too had supported just one directoryon the floppy disk.
- Library procedure puts system calls into register. Then it executes TRAP to switch to kernel mode and start execution. After TRAP kernel code examines system call number then dispatches to correct system call handler. After it's done control returns to user space library procedure then it returns to user program.
## System Calls for Process Management
- Fork creates child process, the same as parent process only distinguishable by fork call. Return value 0 for child PID for parent.
- When program is read from terminal child is forked and waits for it to finish using waitpid.
- Child executes user command using execve command.
- Process has 3 segments: text, data, stack.
## System Calls for File Management
- open, close, read, write.
- To read or write a file you must first open it.
##System Calls for Directory Management
- mkdir/rmdir, link/unlink, mount/unmount,
- link call makes a file appear under two names. Two users at other groups can simultaneously access and write into it. Every directory has an i number located in a table called i-nodes. What link does is it creates a directory with the same i node as the other directory and links them together.
- mount call merges two file systems into one.
## Miscellaneous
- chdir, chmod, kill
- chdir sys call executes cd command.
# WIN32 API
- Win32 is used to get OS services.
- Windows system calls are win32 procedures that are guaranteed to be stable over time.
- no parent child hierarchy on WIN
- Unlike UNIX that has system calls, Windows is event-driven and waits for an event to happen. Win32 does have stable system calls that users expect to access over different versions.
- CreateProcess both forks and executes using execve in the same command.
- WaitForSingleObject is used to wait for an event.
- Win32 interface does not link files, have mounted file systems, security, or signals, so the calls corresponding to the UNIX ones do not exist.
- link, execve, chmod, mount, kill are not supported on Win32.
# OPERATING SYSTEM STRUCTURE
## Monolithic System
- Monolithic systems' entire os runs as single program in kernel mode. The operating system is written as a collection of procedures, linked together into a single large executable program. Single crash takes down entire OS
- Monolithic system structure: Main Procedure->Service Procedures( for system calls)-> Utility Procedures(to help system calls).
- In addition to core OS, many OS let us use shared libraries in UNIX such as I/O drivers and file systems, or DLL(dynamic link libraries) in Windows.

[![Screenshot-from-2019-07-30-12-00-25.png](https://i.postimg.cc/j2SNcdwT/Screenshot-from-2019-07-30-12-00-25.png)](https://postimg.cc/7GjfY4nt)

## Layered System
- Layered system was a natural evolution of monolithic system as a generalization and first created by Dijkstra and his students in 1968.
- MULTICS generalized the layered system.
- Layered approach let designers have a choice where to put layers to. Having all in kernel could result in crashes but user mode allowed safety.
- That is why they put reset buttons on OS.
5 The operator
4 User programs
3 Input/output management
2 Operator-process communication (console-process)
1 Memory and drum(old memory system) management (make sure pages brought to or removed from memory when needed.)
0 Processor allocation and multiprogramming
- Running each device in user mode will prevent bugs in their software to crash the system.
 
## Microkernel
- Microkernel handles interrupts, processes, scheduling, interprocess communication.
- Microkernel splits the OS into modules which don't run in kernel mode but user mode. This improves reliability of the system since now a crash in a process which would previously run in monolithic system would not crash the entire system, only itself.

[![Screenshot-from-2019-07-30-12-02-24.png](https://i.postimg.cc/k4pXrnYq/Screenshot-from-2019-07-30-12-02-24.png)](https://postimg.cc/bD1jkjjB)

## Client-server System
- On a client-server system either in a single computer or more than one computers, the client sends a message to the server and server takes the message and replies back depending on if it's available or not.
## Virtual Machines
- Official IBM timesharing system of OS/360 is TSS/360 which was abandoned after the development consumed so much money.
- z/VM was created as a radically different system that IBM accepted.
- VM/370 was based on timesharing and multiprogramming and a convenient interface than bare hardware.
- Heart of the system VM monitor(hypervisor) runs on hardware and does multiprogramming.
- CMS ( conversational monitor system) was popular with programmers since it allowed timesharing.
- can run several VMs identical to the hardware, each one can run any OS that will run directly on hardware.
- Type 1 hypervisors were built on top of hardware and could host multiple VMs.
- A Hypervisor also known as Virtual Machine Monitor (VMM) can be a piece of software or hardware that gives an impression to the guest machines(virtual machines) as if they were operating on a physical hardware.
- Type 2 hypervisors were built on top of an existing OS so it could rely on existing OS to handle manage calls to CPU.
 
## Exokernel
- Exokernel lets us save a certain size of disk for virtualization, thus lets us run operating systems on them.
- Exokernel also checks if somebody is trying to use an other's resources.
- On other VM implementations, VM thinks it has its own disk to use, so it needs mapping to reach those disk addresses. With exokernel this is not needed since all exokernel needs is knowing which  VM is assigned to which resource.
Large Programming Projects in C
- To build an OS, each .c file is compiled into an object file which have .o that contain binary instructions for the CPU to execute.
- C preprocessor compiles .c files and all associated include headers as if they were physically linked.
- Compiler extracts the .o files and hands them to the linker.
- Linker gets the .o files and combines them into one executable binary file.
- OS code is directly executed by hardware, with no interpreter.
 
