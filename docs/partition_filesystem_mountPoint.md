## Partition:

#### First stage:

- partition table:
    - data about partitions: number, location, size, type, etc.
    - written in a particular sector (Master Boot Record, MBR) at the very begining of the disk 
    - boot loader: the boot program included in the MBR + partition table

- MBR structure: 
    - magic number
    - partition table
    - Primary Boot Loader(PBL): look at partiton table for boot flag (boot partition)

#### Second stage:

- Volume Boot Record, VBR (in the boot partition)
    - allow loading of the operating system
    - Boot Manager (if multiple os, otherwise directly in the MBR)
        - GRUB
        - Lilo

#### 3 type of partitons

- primary partition
    - partitions of old, can be four of them (1,2,3,4)
- extended partition (a special primary partition, for holding more (logical) partitions)
    - can only be one, numbered after the highest primary partition (3 or 4)
    - contain all of the remaining space, otherwise space is wasted
- secondary partition
    - created inside the extended partition, numbers are always 5 and higher

#### GPT partition

- legacy partitioning/dos partitioning (old)
- GPT (GUID Partition Table) (new)
    - GUID: Globally Unique Indentifier
- UEFI: Unified Extensible Firmware Interface (extend the old BIOS firmware)
    - interface between the firmware and the operating system


## Filesystem

#### files in Linux

> In Linux, almost everything is a file; if something is not a file, it is a process.

- File categories:
    - regular file
    - directories
    - special files
        - the mechanism used for input and output
        - placed in /dev directory
    - sockets
        - special files for inter-process communication through the network
    - links
        - mechanism to make visible file in multiple locations of the file system
    - pipes
        - allows inter-process communication with a lean semantics relatively to the sockets

#### Structure of the filesystem

- In Linux, all files and folders are placed in a single root tree '/'

- Devices files or special files (discovered devices)
    - a kind of interface between the device driver and hardware
    - placed usually in the /dev directory
    - /dev/hda1, /dev/sda1

- high level partitioning: putting a branch of the filesystem
    - command: mkfs.<type> (mkfs.ext4)

#### setting up the filesystem: mounting/unmounting of partitions

- what to do with the partitions (before installation)
    - The choices made by the administrator during the installation process are stored into the /etc/fstab file in the real filesystem
- After installation
    - Case of Removable Media
        - The kernel notifies the udev daemon the availability of a new device and udev creates the device file.
        - As for the mounting, the HAL daemon gets an signal from udev telling him that the a new device file has been created. HAL uses the /media directory to accomplish the mounting. 
    - Case of Irremovable Media

#### standard organization of the filesystem

- / is the root. It is the highest level of the filesystem. At the very beginning of the filesystem setting up, this directory — as some other sub directories — is purely virtual — residing in RAM —, then the partition that contains the final root of the filesystem is mounted there read-only. Thus the kernel is able to find the tools necessary to the initialization of disks and the mounting of the other partitions. After this job is done, the partition is remounted read write.
- /home, it the place where the files of the users will be placed. Generally, but it is not mandatory, a separate partition is mounted there.
- /etc is the place for installed application configuration files e.g. /etc/fstab, /etc/hosts, etc.
- /lib is the place for shared libraries and the kernel modules.
- /media is a mounting point for removable devices such as CDs, DVDs, USB stick or drives, etc.
- /bin is a place for essential command binaries such as cat, ls, mount…
- /boot is a place for static files of the boot loader.
- /dev is for the device files.
- /mnt is a place where to mount filesystems temporarily.
- /opt is for additional programs.
- /run is for data related to running processes.
- /sbin is for essential commands.
- /srv is for the data of service supported by the system e.g. the file of a web server.
- /tmp is for temporary files
- /usr is a secondary hierarchy.
- /var is for variable data
- /root is for the files belonging to the super user (root)

## Reference

- [Basics of partitions, filesystems, mount_points](https://en.opensuse.org/SDB:Basics_of_partitions,_filesystems,_mount_points#Notion_of_partition)