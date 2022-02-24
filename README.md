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
    head /proc/cpuinfo  | tr a-z A-Z
    ```

```sh
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