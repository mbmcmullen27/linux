# linux
Book notes: "How Linux Works"


### Chapter 1 - The Big picture
- (4) Because it's common to refer to the state in abstract terms rather than to the actual bits, the term _image_ refers to a particular physical arrangement of bits.

- (5) [The kernel] runs _between_ process time slices during a context switch.

- (7) Other than init, all new user processes on a Linux system start as a result of fork()

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

- (see 97) The LVM example here is an Ubuntu VM that already has LVM installed and configured. According to [Arch Wiki](https://wiki.archlinux.org/title/LVM) the package to install is lvm2, but I suspect the actual configuring of LVM is much more involved
    - (see 102) Constructing a Logical Volume System

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

#### Systemd

- (140) One way that systemd is more ambitious than previous versions of init is that it doesn't joust operate on processes and services; it can also manage filesystem mounts, mmonitor neetwork connection requests, run timers, and more
    - Each capability is called a unit
    - Unit types:
        - **Service Units:** Control the service daemons found on a Unix system.
        - **Target Units:** Control other units, usually by grouping them.
        - **Socket Units:** Represent incoming network connection request locations.
        - **Mount Units:** Represent the attachment of filesystems to the system.
- View systemd dependency graph with `Systemd-analyze dot`
- (150) you can use several _conditional dependency_ parameters to test various operating system states rather than systemd unitts. For example:
    condition|effect
    ---|---
    ConditionPathExists=p        |  True if the (file) path exists in the filesystem
    ConditionPathIsDirectory=p   |  True if _p_ is a directory
    ConditionFileNotEmpty=p      |  True if _p_ is a file and it's not zero-length
    - if you activate a unit that has a conditional dependency and some unit dependencies, systemd attempts to activate those unit dependencies regardless of wheteher the condition is true or false

#### System V init
- uncommon in most modern distros
- (160) The mechanism that System V init uses to run the init.d scripts has found its way into many Linux systems, regardlesss of whether they use System V init.
    - `run-parts --help` 
    - can be used to run all scripts in a file that match a regex pattern

### Chapter 7 - System Configuration: Logging, System Time, Batch Jobs, and Users

#### journald
- search journalctl for a specific unit: `journalctl  -U cron.service`
- list units in the journal: `journalctl -F _SYSTEMD_UNIT`
- list available fields: `journalctl -N`
- filter by boot with offset 1 (last boot): `journalctl -b -1`

#### cron
- /etc/crontab is a system wide cron table where you need to specify the user to run as
    - `crontab -e` edits only the executing user's table

- (187) The cron utility is one of the oldest components of a Linux system; it's bveen around for decades (predating Linux itself)

#### systemd equivalent to `at`
```sh
systemd-run --on-calendar='2022-08-14 18:00' /bin/echo this is a test
```
- can also use `--on-active=30m` for 30 minutes in the future
- (188) systemd has a user manager associated with a logged-in user, and this is necessary to run timer units. You can tell systemd to keep the user manager around after you log out with this command: `loginctl enable-linger`

#### user authentication
- (192) you can move away fro using /etc/passwd for uyour users and use a network service such as LDAP instead by changing only the system configuration
    - I always assumed LDAP worked like SSO, and a user would be created on the target system if it existed on the LDAP server but not the target system, but this seems to suggest you don't use the native login service to authenticate at all?
    - does using a network auth service mean I can't login to my system if I don't have network connectivity? Even if I've logged in in the past?

### Chapter 8 - A Closer Look At Processes and Resource Utilization
- (208) The keernel adds the nice value to the current priority to determine the next time slot for the process
    - change the nice value with renice `renice 20 <pid>`    

```sh
# view page faults in a running process
ps -o pid,min_flt,maj_flt <pid>
```

- `vmstat 2` for I/O and memory use statistics

#### cgroups
- (217) There are two versions of cgroups, 1 and 2 and unfortunately, both are currently in use and can be configured simultaneously on a system
```sh
# list the shell's cgroups
cat /proc/self/cgroup

# view example cgroup fs
ls /sys/fs/cgroup/user.slice/user-1000.slice/session-1.scope

# add a process to a cgroup
sudo echo pid > cgroup.procs

# view cpu utilization of a cgroup
cat cpu.stat
```

### Chapter 9 - Understanding Your Network and Its Configuration

#### Ethernet
- (236) devices on an Ethernet network send messages in frames, which are wrappers around the data sent. Aframe contains the origin and destination MAC addresses
- Frames can't leave a physical Etherenet network without a bridge to take data out of one frame, repackage it, and send it to a host on a different physical network

- (236) network interfaces usually have names that indicate the kind of hardware underneath, such as _enp0s31f6_ (an interface in a PCI slot)
    - a name like this is called a _predictable network interface device name_

```sh
# send traffic to 192.168.45.0/24 through a router at 10.23.2.44
ip route add 192.168.45.0/24 via 10.23.2.44 
```

#### DNS
- (245) \[...\] this is part of the idea behind zero-configuration name service systems such as Multicast DNS (mDNS) and Link Local Multicast Name Resolution (LLMNR). If a process wants to find a host by name on the local netowrk, it just broadcasts a request over the network; if present, the target host replies with its address
    - resolving dns without a nameserver could be useful in the homelab

```sh
# check current DNS settings
resolvectl status
```

- FURTHER READING: _DNS and BIND_, 5th edition, by Cricket Liu and Paul Albitz

#### localhost
- (247) The _lo_ interface is a virtual network interface called the _loopback_ because it "loops back" to itself. \[...\] When outgoing data to localhost reaches the kernel network interface for _lo_, the kernel just repackages it as incoming data and sends it back through _lo_, for use by any server program that's listening

#### tcp
```sh
# well known port numbers
cat /etc/services
```
- [Internet Assigned Numbers Authority](www.iana.org)
- (250) only the superuser can use ports 1-1023, also known as system or privileged ports

#### DHCP
- (253) Upon startup, dhclient stores its process ID in /var/run/dhclient.pid and its lease information in `/var/lib/dhcp/dhclient.leases`
    - Inspecting `/var/run` I can see the docker.pid and sshd.pid and the docker.sock
    - Seems its common practice to dump a pid into this directory, I wonder what the intended use of these is, async bash seems tricky
        - When are pids assigned? How do I read my pid if I'm a running process? Do subshells get a separate pid?
- (253) The IETF took advantage of the large IPv6 address space to devise a new way of network configuration that does not require a central server. This is called _stateless configuration_

**(254) Section 9.21** Configuring Linux as a Router

#### Network Address Translation (IP Masquerading) - NAT
- (257) Hosts on the privateee network need no special configuration the router is their default gateway \[...\]
    1. A host on the internal private network wants to make a connection to the outside world, so it sends its connection request packets through the router.
    2. The router intercepts the connection request packet rather than passing it out to the internet (where it would get lost bcause the public internet knows nothing about private networks).
    3. The router determines the destination of the connection request packet and opens its own connection to the destination.
    4. When the router obtains the connection, it fakes a "connection established" message back to the original internal host.
    5. The router is now the middleman between the internal host and the destination. The destination knows nothing about the internal host; the connection on the remote host looks like it came from the router.

- NAT works on the internet and transport layer, packets need to be examined to determine tcp and udp port numbers

- to setup a NAT router on a linux machine you must enable:
    1. network packet filtering ("firewall support")
    2. connection tracking
    3. iptables support
    4. full NAT
    5. MASQUERADE target support

- Example iptables commands for internal network enp0s2 sharing an external connection at enp0s31f6 (257)

    ```sh
    sysctl -w net.ipv4.ip_forward=1
    iptables -P FORWARD DROP
    iptables -t nat -A POSTROUTING -o enp0s31f6 -j MASQUERADE
    iptables -A FORWARD -i enp0s31f6 -o enp0s2 -m state --state ESTABLISHED,RELATED -j ACCEPT
    IPTABLES -A FORWARD -i enp0s2 -o enp0s31f6 -j ACCEPT
    ```

- (258) Nat \[...\] is essentially a hack that extends the lifetime of the IPv4 address space. IPv6 does not need NAT, thanks to tis larger and more sophisticated address space

#### Routers and Linux
- (258) Linksys, was required to release the source code for its software under the terms of the license of one of its components, and soon specialized linux distributions such as OpenWRT appeared for routers. (The "WRT" in these names came from the Linksys model number.)

```sh
# show iptables
iptables -L

# set DROP policy on FORWARD chain
iptables -P FORWARD DROP

# DROP all packets from host 192.168.34.63
iptables -A INPUT -s 192.168.34.63 -j DROP

# DROP all packets from subnet 192.168.34.0/24 coming into port 25
iptables -A INPUT -s 192.168.34.0/24 -p tcp --destination-port 25 -j DROP

# delete the third rule and reinsert it at the top of the chain
iptables -D INPUT 3
iptables -I INPUT -s 192.168.34.37 -j ACCEPT

# example INPUT chain for ssh host 
# ACCEPT icmp connections (for ping and other utilities)
# ACCEPT connections from 10.1.0.0/16 subnet and localhost
# ACCEPT any non-SYN packet (block  connections)
# ACCEPT UDP connections from DNS server
# ACCEPT ssh connections from anywhere
iptables -P INPUT DROP
iptables -A INPUT -p icmp -j ACCEPT
iptables -A INPUT -s 127.0.0.1 -j ACCEPT
iptables -A INPUT -s 10.1.0.0/16 -j ACCEPT
iptables -A INPUT -p tcp '!' --syn -j ACCEPT
iptables -A INPUT -p udp --source-port 53 -s [nameserver_addr] -j ACCEPT
iptables -A INPUT -p tcp --destination-port 22 -j ACCEPT
```

- _ARP_ - Address Resolution Protocol
- (264) A host using Ethernet as its physical layer and IP as the network layer...
    - Does that mean there alternatives to IP at the network layer?

- (267) **iw** 9.27.1
- _WPA_ - Wifi Protected Access

### Chapter 10 - Network Applications and Services

- example telnet request
```sh
telnet example.org 80
GET / HTTP/1.1
Host: example.org
# enter twice

```

- example curl request
```sh
curl --trace-ascii trace_file http://www.example.org/
```

- ssh_config has an opition `X11Forwarding` that enable X Window System client tunneling

```sh
# copy dir to remote host
tar zcvf - dir | ssh remote_host tar zxvf - 
```
#### Diagnostic Tools
- `lsof -iTCP -sTCP:LISTEN` to show processes listening on tcp ports
- `tcpdump` puts your network interface card into _promiscuous mode_ and reports on every packet that comes accross
- `nmap` for port scanning

#### Security Resources
- [the SANS institute](http://www.sans.org/)
- [The CERT Division of Carnegie Mellon's Software Engineering Institute](http://www.cert.org/)
- [Nmap and other tools](http://www.insecure.org/)

### Chapter 11 - Introduction to Shell Scripts

- Special vars
    - `$#` : number of arguments
    - `$@` : all arguments
    - `$$` : process id
    - `$?` : exit code

```sh
# check for empty argument
if [ "$1" = hi ]; then
if [ x"$1" = x"hi" ]; then
```

```sh
# use grep as a conditional
if grep -q daemon /etc/passwd; then
    echo The daemon user is in the password file
fi
```

(302) File Type Operators
|Operator|Tests for|
|---|---|
|-f|Regular file|
|-d|Directory|
|-h|Symbolic link|
|-b|Block device|
|-c|Character device|
|-p|Named pipe|
|-s|Socket|


(303) File Permission Operators
|Operator|Permission|
|---|---|
|-r|Readable|
|-w|Writeable|
|-x|Executable|
|-u|Setuid|
|-g|Setguid|
|-k|"Sticky"|

- bash case statement can do pattern matching
- I knew about *) but didn't realize it could do full patterns
- (304)
    ```sh
    case $1 in
        bye)
            echo Fine, bye.
            ;;
        hi|hello)
            echo Nice to see you.
            ;;
        what*)
            echo Whatever.
            ;;
        *)
            echo 'Huh?'
            ;;
    esac
    ```

- (305) while loop with grep
    ```sh
    #!/bin/sh
    FILE=/tmp/whiletest.$$;
    echo firstline > $FILE

    while tail -10 $FILE | grep -q firstline; do
        # add lines to $FILE until tail -10 $FILE no longer prints "firstline"
        echo -n Number of lines in $FILE:' '
        wc -l $FILE | awk '{print $1}'
        echo newline >> $FILE
    done

    rm -f $FILE
    ```
    - the exit code of `grep -q firstline` is the test

- Bash also has an until loop that will run until the conditon returns a zero exit code instead of a nonzero exit code

- (306) "you shouldn't need to use the while and until loops very often. In fact, if you find that you need to use while, you should probably be using a language more appropriate to yourr task, such as Python or awk."

- (306) tmpfile w/ trap (deletes tmpfiles when a CTRL+C interrupt signal is recieved)
```sh
#!/bin/sh
TMPFILE1=$(mktemp /tmp/im1.XXXXXX)
TMPFILE2=$(mktemp /tmp/im2.XXXXXX)
trap "rm -f $TMPFILE1 $TMPFILE2; exit 1" INT

cat /proc/interrupts > $TMPFILE1
sleep 2
cat /proc/interrupts > $TMPFILE2
diff $TMPFILE1 $TMPFILE2
rm -f $TMPFILE1 $TMPFILE2

```

### Chapter 12 - Network File Transfer and Sharing

- (316) `python -m SimpleHTTPServer` starts a basic web server that makes the current directory available on port 8000 by default

- (324) 12.4 Sharing Files with Samba
    - Samba allows your network's Windows computers to get to your Linux system
    - is this necesssary if instead you just use WSL?

### Chapter 13 - User Environments

- (337) "Many users create a bin directory of thier own to store shell scripts and programs"
    - why have I not done this?

- (339) "You can't change an environment variable with a shell script, because scripts run as subshells. (but you can instead define shell functions to perfom this task)

- (341) "You also need a .bash_profile if you ever want to log in on the console or remotely,because those login shells don't ever bother with .bashrc"
    - This can't be true can it? I only have functions in a bashrc and they definitely are run when I ssh

### Chapter 14 - A Brief Survey of the Linux Dektop and Printing
- (348) "We'll aslo take a quick look at printing, as derkstop workstations often share a common printer"
    - Feeling like nobody expected printers would ever go out of style


#### Desktop Components (Xorg vs Wayland)
- (348) _framebuffer_: a chunk of memory that graphics hardware reads and transmits to the screen for display
- in Wayland windows have their own memory buffer and software called a _compositor_ combines them for the framebuffer
- in X window system, the xserver handles requests to draw a window

- (351) These two systems are not mutually exclusive. If your system uses Wayland, it is also probably running an X compatibility server. It's also possible to start a Wayland compositor inside X