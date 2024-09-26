# Network Booting

- Network booting, or booting from LAN as it is also called, is a process which allows a computer to start up and load an operating system or other program directly from the network without any locally attached storage device, like a floppy, CDROM, USB stick or hard drive.

- On `Intel` architecture computers this is made possible with the `PXE standard`. PXE extends the features of the BIOS so that it can run software directly from the network. PXE support is now so common that you can expect it to be present in any reasonably modern computer that comes with an Ethernet jack (commonly known as RJ45).

## Core Technologies:

- DHCP protocol, used to initialize network configuration for a client
- TFTP protocol, used to download a network boot program (NBP)
- HTTP protocol, used to download data from a web server
- PXE, a way to boot Intel computers using DHCP and TFTP
- UNDI, an API used by the PXE environment to generalize access to networking hardware

## Other Relevant Protocols

All of these network protocols deal with how to access storage over the network in different ways.

iSCSI - for block devices
AoE - for block devices (non-routable, local network only)
NBD - for block devices
NFS - for files (mostly used on Unix)
SMB/CIFS - for files (mostly used on Windows)


## The BIOS Boot Process

When your computer powers on and starts running your operating system, it goes through a series of operations before it actually starts your operating system. Your operating system is a very sophisticated boot program that takes total control over your computer. But a boot program can also be a fairly simple program, like a memory diagnostics program, a hardware stability checker, or even a simple game like Pong or Tetris.

- Power On

    When you put power on your computer and press the On button on the case.

- Initialize hardware

    The BIOS performs an inventory of all the components in the computer, such as the CPU, memory chips, extension cards, storage controllers, etc.

- Run self-tests

    All of the components discovered goes through a self-test procedure, to ensure they are operating properly. If any of the components fail, and that component is required for basic operation, your computer will usually make a series of beeps and stop functioning. When all problems have been fixed the BIOS will move on to the next step in the process, which is to discover additional option ROMs.

- Computer stopped

    If your computer ends up in this state, it will either hang forever, or it will turn itself off. This depends on how it entered this state, and how your BIOS is configured to react when it reaches this state.

- Discover built-in devices and option ROMs

    During this activity, your BIOS will discover all of the extensions available. BIOS extensions are usually included in the firmware of your BIOS or burned into an EEPROM or flash chip on one of your add-on cards. During booting you can normally notice this as your IDE, SATA, SCSI or other controllers finding the devices that are attached to them. For network cards, this is usually when you see the prompt that lets you specify what kind of boot protocol it should support (like PXE or RPL). Option ROMs should usually not do anything fancy at this point, except initialize hardware, run self-tests and set up boot service (BBS) entry points. Once all extensions have been allowed to run and add their boot service entry points, control goes back to the BIOS. At this point all the add-on cards and internal services of the BIOS have been initialized and are ready to do work. All of the boot service entry points are ordered according to the configuration specified in the BIOS. It is quite common that the user is given a choice of which boot service to try first by pressing a hotkey. F12 is a common key to start a boot service offered by a network card, but this varies among manufacturers. Once the first boot service has been selected, either manually or automatically, control moves to the next step.

- Start first boot service in BIOS boot services list

    During this stage, the program indicated by the boot service entry point is started. At this point, control passes to the boot service program, which starts its discovery process for a boot program.

- Boot service performs discovery for boot program

    Different boot services go about looking for the boot program in different ways. A floppy controller will read the first sector of the floppy and get ready to start that piece of code. A hard-drive (HDD) controller will usually read the master boot record (MBR) of the first attached HDD and designate that as the boot program. A network card using the PXE standard will perform a DHCP request to find out its IP address and location of boot program. If a location is advertised, a TFTP request is performed to fetch the boot program, commonly referred to as a network boot program (NBP). If the boot service was unable to find a valid boot program, the boot service will exit and control returns back to the BIOS, which will try the next boot service. If a boot program was successfully found, control will be handed over to it.

- Remove first boot service or put at bottom of list

    The BIOS needs to cycle to the next boot service in its list. Whether the BIOS discards the current boot service or adds it back at the end of the list varies between BIOS vendors. Both methods have pros and cons. The next step is to figure out if there are any more boot services to try out.

- More boot services available?

    If there are more boot services available, the next one in line will be started. If there are no more boot services the computer will halt and perform its halt operation.

- Start boot program

    At this point the boot program is in full control of the computer, and it will start doing whatever it is supposed to do. If the boot program detects a problem or wishes to, it can return back to the BIOS. This is not a very common thing to do for a boot program, as a lot of BIOSes have buggy implementations for getting control back. The more common method is just to display a message and hang, or reboot instantly. Since the boot program has full control over the computer it can make use of all the other devices the BIOS has detected to perform whatever action it needs to.

- Boot program running

    The final part a boot program normally does, is to hand over control to an operating system kernel. A boot program that performs this kind of action is usually called a boot loader. Common boot loaders for Linux systems are grub and syslinux. Before the boot loader does this, it will usually also fetch additional data from a storage device, e.g. drivers and configuration files. Any program code required to operate the hardware must be in main memory at this point, or you will be unable to access the hardware. This requirement is usually implemented using ramdisks, so that the kernel can be kept modular and flexible. In Linux they are called initial ramdisks (initrds), in Solaris they go by the name boot archives, and in recent Windows versions they are called wim files. The operating system kernel will then perform a complete discovery of the hardware attached to the system (again) and start doing whatever it is programmed to do. At this point, because another piece of code (the kernel) is in direct control of the hardware, it would be very unwise to switch control back to the BIOS, as hardware state has been modified under its feet.

  ![](./imgs/Screenshot%202024-09-23%20at%207.03.42 PM.png)

## How iPXE extends the network boot process

- `iPXE` is a sophisticated boot program that is capable of extending the traditional `PXE` network boot process in several ways. It can be flashed as an add-on card's option ROM, or it can be loaded as a network boot program (NBP) from an existing `PXE` option ROM via `TFTP` (this is called chainloading). It is also possible to include it as an option ROM inside your BIOS or load it from any local storage media, like floppy, USB, CD or HDD.

- Depending on how iPXE is configured, it can load additional boot programs from several different sources in addition to TFTP. The most common way is to use HTTP to load additional content using a standard web server. If your web server supports range requests, you can also use it to boot floppies and CD images (ISOs) directly from the web server. FTP can also be used in the same way. There is even support for encrypted transmission with HTTPS. It is possible to configure it to only allow execution of programs that have been signed. If you combine this with ROM-burning you can have a network boot loader that will load only trusted code.

- One of the most interesting features iPXE enables, is to boot a computer without an iSCSI host bus adapter from an iSCSI volume. This is possible, because iPXE implements a full-featured software-based iSCSI initiator. It even supports CHAP authentication! For operating systems that support it, you can also use AoE (ATA Over Ethernet) in addition to iSCSI.

- The final feature that makes iPXE so impressive, is that it also has a very advanced scripting language and text-based menu system. These features enable you to make dynamic boot environments without the need to know a server-side scripting language like PHP, Perl or Python.

- A network boot application is unique for a number of reasons:

  - It is invoked by the BIOS or UEFI firmware during the initial bootstrap of the computer, before any operating system or disk boot manager.

  - It depends on the presence of a special chip on the network card: the PXE boot ROM.

  - It can communicate with a server over the network because of a programming interface provided by the bootrom. PXE is the informal standard for this kind of programming interface.

  - It is downloaded from a special OS deployment server, and does not depend on any local storage device. It can work on computers without hard disk, CD, or diskette devices. If a local storage device is present on the computer, it is accessible to the network boot application. However, a network boot application does not fail if the storage device is corrupt or broken.

  - It gets its IP and other boot parameters from a central DHCP server.

    ```mermaid
    sequenceDiagram
        participant Client as Booting Device (Client)
        participant BIOS as BIOS/UEFI
        participant NIC as Network Interface (NIC with PXE)
        participant DHCP as DHCP Server
        participant TFTP as TFTP Server

        Client ->> BIOS: Power ON
        BIOS ->> NIC: Check Boot Order for PXE Boot
        NIC ->> DHCP: Broadcast DHCP Discovery (DHCPDISCOVER)
        DHCP -->> NIC: DHCP Offer (Includes IP, Gateway, TFTP Server)
        
        NIC ->> DHCP: DHCP Request (DHCPREQUEST) for Lease and Boot Info
        DHCP -->> NIC: DHCP Acknowledgment (DHCPACK) with Boot File Info
        
        NIC ->> TFTP: Request Boot File (TFTP RRQ)
        TFTP -->> NIC: Send Boot File (TFTP DATA)
        
        NIC ->> Client: Execute Boot File (PXE Bootloader)
        Client ->> TFTP: Request Operating System Files (Kernel, Initramfs)
        TFTP -->> Client: Send OS Files

        Client ->> BIOS: Load Operating System into Memory
        BIOS ->> Client: Hand off Control to Operating System
        Client ->> Client: Operating System Running
    ```

### Steps to Implement Network Booting (PXE Boot)

- Ensure PXE Boot is Enabled on Client:

  - Go to BIOS/UEFI settings on the client machine.
  - Enable PXE boot in the boot order.
  - Set the Network Interface Card (NIC) as the primary boot device.

- Configure a DHCP Server:

  - Set up a DHCP server that provides IP addresses to the clients.
  - Ensure the DHCP server is configured to provide the location of the TFTP server and the boot file (also known as a bootloader).
  - Key DHCP options:
    - Option 66: TFTP server name
    - Option 67: Boot file name (e.g., pxelinux.0 for Linux environments)

- Set Up a TFTP Server:

  - Install and configure a TFTP server to host the boot files.

    **Note**: Trivial File Transfer Protocol (TFTP) is a simple protocol that provides basic file transfer function with no user authentication. TFTP is intended for applications that do not need the sophisticated interactions that File Transfer Protocol (FTP) provides.

  - On Linux systems, you can use dnsmasq or tftpd-hpa as TFTP servers.

  - Store bootloader files like `pxelinux.0` or other bootloaders (for Windows, wdsnbp.com).
  - Make sure the TFTP server is reachable by the client systems.

- Prepare Boot Files:

  - For Linux systems:

    - Download and store the kernel and initramfs files that the PXE client will download to boot.

  - For Windows systems:

    - Use Windows Deployment Services (WDS) to store Windows installation images.

- Set Up a Bootloader:

  - The bootloader (like pxelinux.0 for Linux) is responsible for downloading and launching the OS.

  - Configure the bootloader to point to the kernel and the root filesystem

- Start the PXE Boot Process:

  - Once the setup is complete, boot the client device.

  - The client will:

    - Request an IP address from the DHCP server.

    - Receive the TFTP server’s address and bootloader details.

    - Download the bootloader from the TFTP server.

    - Load the operating system files and boot the system.

## Network Boot Sequence

- Start up: A user or a wake-up event starts the remote-boot computer up.

- Network boot: The BIOS configuration or boot order, a hot key, for example, F12, or the wake-up event or the UEFI boot sequence instructs the computer to boot on the network.

- IP address discovery: The remote-boot target broadcasts a DHCP request for an IP address. Any DHCP server that recognizes the hardware address of the target or that has a pool of freely distributable dynamic addresses, sends an IP address. The target takes the first response and confirms it to the server. In addition to the IP address, the server gives some other network parameters to the target and information about the boot procedure to follow.

- OS deployment server discovery: In the case of PXE remote-boot, the target computer then proceeds to discover the OS deployment server. The OS deployment server is responsible for delivering a network boot program to the target. It is not necessarily the same computer as the DHCP server. The target responds to the first OS deployment server which replies and downloads a small program using a simple multicast protocol (MTFTP).

- Server connection: If the network boot program is the deployment engine, it establishes a secure encrypted connection to the OS deployment server and receives instructions from the server to determine the name of the program to run.

- Pre-OS configuration: The deployment engine then downloads from the file server everything required by the OS configuration specified by the system administrator. These file transfers are done using a secure, robust and efficient unicast or multicast protocol. You can perform many actions at this stage: installing an operating system from scratch, repairing a corrupted operating system, or performing an inventory.

- OS booting: When instructed by the OS configuration, the deployment engine removes itself from memory and allows the computer start the operating system, as if the target is booting normally from the hard disk. This ensures full compatibility with the operating system and avoids all problems of the traditional diskless remote boot.

    ```mermaid
    graph TD;
        A[Power ON] --> B[BIOS/UEFI Initialization];
        B --> C[Check Boot Order];
        C --> D[PXE Enabled Network Card];
        D --> E[DHCP Request];
        E --> F[DHCP Server Response with Boot File Info];
        F --> G[TFTP Request for Boot File];
        G --> H[TFTP Server Response with Boot File];
        H --> I[Bootloader Executed];
        I --> J[Operating System Loaded Over Network];
        J --> K[OS Ready for Use];
    ```
