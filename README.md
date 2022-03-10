# linux
Book notes: "How Linux Works"


### Chapter 1 - The Big picture
- (page 4)
    
    Because it's common to refer to the state in abstract terms rather than to the actual bits, the term _image_ refers to a particular physical arrangement of bits.

- (page 5)

    [The kernel] runs _between_ process time slices during a context switch.

- (page 7)

    Other than init, all new user processes on a Linux system start as a result of fork()

### Chapter 2 - Basic Commands and Directory Hierarchy
- `echo *` has the same output as `ls`
    - the shell will always replace wildcards/globs with matched files even if the command doesn't expect any filename
    - (19) if no files match the glob, the bash shell performs no expansion and the command runs with literal characters

- [The Jargon File](http://www.catb.org/jargon/html/)
- (25 table 2-2) command line keystrokes

- search for a man page by keyword with `man -k keyword`
- manual pages are sorted into sections based on what the package is used for

- `tr` is translation not trim
    ```sh
    # converts characters to uppercase
    head /proc/cpuinfo  | tr a-z A-Z
    ```

```sh
# redirect stdout to a file 'f'
ls /fffff > f

# redirect stdout and stderr (2 is the stream id) 
ls /fffff > f 2> e

# redirect stderr to the same place as stdout
ls /fffff > f 2>&1
```

### Chapter 3 - Devices

```sh
# print device information
udevadm info --query=all --name=/dev/sda
```

```sh
# copies a single 1,024-byte block from /dev/zero (a continuous stream of zero bytes) to new_file
dd if=/dev/zero of=new_file bs=1024 count=1
```
- (51) dd option format is based on an old IBM job control language (JCL) style
- (52) [considering storage devices /dev/sd*] the sd portion of the name stands for SCSI disk. _Small Computer System Interface_ 

### Chapter 4  - Disks and Filesystems

- (76) exercise 4.1.3 - Creating a Partition Table
- (82) list 4.2.1 - Filesystem Types
    - (83) "At the time of this writing, Btrfs is the default for one major Linux distribution. If this proves a success, it's likely that Btrfs will be poised to replace the Extended series [ext2-4]."

- (87) "There is a difference between Unix and DOS text files, principally in how lines end [...] There have been many attempts at automatic conversion at the filesystem level, but these are always problematic."

- on (88) there is an example fstab file that mounts an iso9660 filesystem from a cdrom. Is it possible to mount iso's that are already on disk? is that what is happening in a container?
- (91) "Aside from hardware problemms, filesystem errors are usually due to a user shutting down the system in a rude way (for example, by pulling out the power cord.)"

```sh
# initialize an empty file as swap and add it to the swap pool
dd if=/dev/zero of=swap_file bs=1024k count=num_mb
mkswap swap_file
swapon swap_file
```

- (97) The LVM example here is an Ubuntu VM that already has LVM installed and configured. According to [Arch Wiki](https://wiki.archlinux.org/title/LVM) the package to install is lvm2, but I suspect the actual configuring of LVM is much more involved
    - (102) Constructing a Logical Volume System

- where /dev/sdb1 is a physical volume
    ```sh
    # print LVM header on a PV
    dd if=/dev/sdb1 count=1000 | strings | less
    ```    
- (113) 4.6.1 inode details
    ```sh
    # view inode numbers for any directory
    ls -i
    ```
    - "A hard link is just a manually created entry in a directory to an inode that already exists"
    - (114) "when you check a filesystem, as described ini Section 4.2.11, the `fsck` program walks through the inode table and directory sttructure to generate new link counts and a new block allocation map (such as the block bitmap), and then compares the newly generated data with the filesystem on the disk."

### Chapter 5 - How The Linux Kernel Boots

- (117) A simplified view of the boot process looks like this:
    1. The machine's BIOS or boot firmware loads and runs a boot loader
    2. The boot loader finds the kernel image on disk, loads it into memory, and starts it.
    3. The kernel initializes the devices and its drivers.
    4. The kernel mounts the root filesystem.
    5. The kernel starts a program called _init_ with process ID of 1. This point is the _user space_ start.
    6. init sets the rest of the system processes in motion.
    7. At some point, init starts a process allowing you to log in, usually at the end or near thee end of the boot sequence.

- (119) Upon startup, the Linux kernel initializes in this general order:
    1. CPU inspection
    2. Memory inspection
    3. Device bus discovery
    4. Device discovery
    5. Auxiliary kernal subsystem setup (networking and the like)
    6. Root filesystem mount
    7. User space start

```sh
# view kernel boot parameters
cat /proc/cmdline
```
- see bootparam(7) man page or reference file kernel-params.txt for an overview on available boot params
- (122) A boot loader's core functionality includes the ability to do the following [...] Select from multiple kernels
    - when would this be necessary? 
    - does this mean you can use different linux distributions on the same system without re-partitioning?

- (132) UEFI makes it releatively easy to support loading other operating systems because you can install multiple boot loaders in the EFI partition [...] you may still have an individual partition with an MBR-style boot loadder [...] Instead of configuring and running a Linux kernel, GRUB can load and run a different boot loader on a specific partition on your disk; this is called _chainloading_
    ```sh
    menuentry "DOS" {
        insmod chain
        insmod fat
        set root=(hdo,3)
        chainloader /io.sys
    }
    ```
    - ^^ Grub's configs look a lot like Hashicorp's HCL

### Chapter 6 - How User Space Starts

- (140) One way that systemd is more ambitious than previous versions of init is that it doesn't joust operate on processes and services; it can also manage filesystem mounts, mmonitor neetwork connection requests, run timers, and more
    - Each capability is called a unit
    - Unit types:
        - **Service Units:** Control the service daemons found on a Unix system.
        - **Target Units:** Control other units, usually by grouping them.
        - **Socket Units:** Represent incoming network connection request locations.
        - **Mount Units:** Represent the attachment of filesystems to the system.
- View systemd dependency graph with `systemd-analyze dot`