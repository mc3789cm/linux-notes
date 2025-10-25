# What is a File System
A **file system** governs file organization and access. It's the abstraction layer for the stream of raw bytes on the 
disk, so you don't have to memorize the hundreds of annoying little data addresses for each file. There's so many file
systems, from simple file systems like FAT32 to advanced ones like EXT$, but they all have one thing in common - to 
organize those raw bytes on your disk into a format that can be **logically read and grouped**.

# What is a Journaling File System?
A **journaling file system** is a type of file system that keeps track of changes **not yet committed** to the main 
file system by recording them in a special data structure called a **journal**. This aids in preventing file system
corruption and makes recovery time significantly faster and more reliable after crashes, power failures, etc.

# What is the Copy-on-Write (CoW) Principle?
**Copy-on-Write (CoW)** is a strategy/technique used in system design to **delay making copies** of data until 
necessary.

The idea is that if multiple processes, snapshots, etc. are sharing data and nobody modifies it, they simply
reference the same data. Once a process wants to modify that shared data, *only then* is a private copy created for that
process.

1. Share the data

Initially, the data is shared between multiple references. Reads are allowed, while writes are not.

2. Attempt to write
	
When a process wants to modify the shared data, the system intercepts this because the shared data isn't actually
writable yet.

3. Allocate a new block

The system allocates a separate area for the process (basically, a new copy).

4. Copy existing data

The contents from the shared resource(s) are copied into the newly allocated block. Other shared processes still refer
to the original.

5. Redirect writes
	
The process is now directed to operate on the new copy. Metadata is updated to show that the modified version is 
separate from the original, while the old version is remained untouched until/if it is no longer referenced.

# Different File Systems
## EXT (Extended File System)
The **extended file system** family has been a defining struct in Linux file storage solutions, evolving to address 
increasing demands for performance, reliability, and scalability. The next few sections about the evolution of this file
system gives a more extensive view of the features, improvements, and rationale behind their evolution over time.

### EXT (The First Extended File System)
The original extended file system was implemented in 1992 as the first file system created specifically for the Linux
kernel. It was a major improvement over the MINIX file system, which had severe limitations at the time.

EXT has a metadata structure inspired by traditional UNIX file systems. Like, the file system appearing as **one 
rooted tree of directories** and how they don't actually contain files, but rather contain the names of files paired 
with references to **inodes**, which is what contains the files and their respective metadate (permissions, time of last
access, ect).

It could only handle file systems up to 2 GB in size, but back in 1992, this was still a significant amount.

### EXT2 (The Second Extended File System)
EXT2 was developed in 1993 as a replacement for the preceding extended file system. EXT2 had a focus on improved 
performance and functionality while addressing the obvious limitations of the original extended file system.

One of the key features was support for larger files, being able to handle data up to 2 TB, and partitions up to 4 TB.
Another is a much more flexible inode table, allocating files and their metadata to a table, improving efficiency
and scalability.

### EXT3 (The Third Extended File System)
EXT3 was implemented to the Linux kernel 2001 as an evolutionary upgrade to EXT2 while maintaining backwards 
compatibility. The most important of features for this version and what made it revolutionary at this time was the 
addition of [journaling](#what-is-a-journaling-file-system). This reduced recovery times by eliminating the need for
time-consuming file system checks after each crash.

Although, to achieve its backwards compatibility, it had to retain the limitations of EXT2 in terms of file and
partition sizes. The second disadvantage was the performance bottleneck of journaling. While it did vastly improve 
reliability, it also created some performance overhead.

### EXT4 (The Fourth Extended File system)
Introduced in 2006, and declared stable in 2008, EXT4 marked a significant leap in the family; addressing the 
scalability and performance limitations of both EXT2 and EXT3 while adding many new features.

EXT4 added support for file sizes up to 16 TB, and partition sizes up to an entire exabyte (1 EB = 1,000,000 TB).

The additions of extents, delayed allocation, journal checksums, persistent preallocation, 64-bit file system support,
and faster file system checks, truly made this a modern file system still used today and has influenced a few of the
file systems I'll talk about next.

## BTRFS (B-Tree File System)
Developed by Oracle in 2007, BTRFS is a modern file system using the
[Copy-on-Write Principle](#what-is-the-copy-on-write-cow-principle) with the intent of implementing advanced features
while focusing on fault tolerance, repair, and easy administration.

What differs BTRFS from other file systems is mainly its use of CoW; when data is modified, the changes are written 
to new blocks rather than overwriting the original. This improves reliability and enables other features of what makes
this file system so advanced, like check summing and snapshots.

BTRFS can support files and partitions sizes up to 16 exabytes and has native RAID support for RAID 0, 1, 10, 5, and 6,
omitting the need for separate RAID layers.

# Sources
- [Wikipedia - Journaling File System](https://en.wikipedia.org/wiki/Journaling_file_system)
- [Wikipedia - Extended File System](https://en.wikipedia.org/wiki/Extended_file_system)
- [Wikipedia - Copy-on-Write](https://en.wikipedia.org/wiki/Copy-on-write)
- [Linux Foundation - FHS 3.0](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/index.html)

# Appendix
## FHS 3.0 (File-System Hierarchy Standard 3.0)
This section only pertains to the **required** specifications of the FHS 3.0.

The FHS 3.0 provides a set of standards for directories and their structure in the context of a UNIX-like 
operating-system. It lets software (and users) predict the location of installed directories, files, and binaries,
addressing issues where file placements need to be coordinated between many other parties.

It does not pertain to the placement of local files though, as this is a local issue and imposing such restrictions 
on system administrators would be redundant.

| Directory | Purpose                                                |
|:---------:|:-------------------------------------------------------|
|   /bin    | Essential command binaries                             |
|   /boot   | Static files of the boot loader and kernel             |
|   /dev    | Device files                                           |
|   /etc    | Host-specific system configuration data                |
|   /lib    | Essential shared libraries and kernel modules          |
|  /media   | Mount point for removable media                        |
|   /mnt    | Mount point for **temporarily** mounting a file-system |
|   /opt    | Add-on application software packages                   |
|   /run    | Data relevant to running processes                     |
|   /sbin   | Essential system binaries relevant for recovery        |
|   /srv    | Data for services provided by the system               |
|   /tmp    | Temporary files                                        |
|   /usr    | Secondary hierarchy                                    |
|   /var    | Variable data                                          |
