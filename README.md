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