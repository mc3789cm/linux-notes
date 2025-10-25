# What is a Partition?
All a **partition** is, is a **logically separated section of the physical disk**. We're basically just slicing the cake
into different pieces, so each of which can be use separately for their own purposes. This lets operating systems treat
each partition as its own independent volume. Each partition is then further defined with a
[file system](./file-systems.md).

# What is Partitioning?
When planning out, provisioning, and managing storage for a computer, a fundamental aspect is **disk partitioning**. To
*"partition"* a disk is to **divide the hard drive into multiple logical sections** (partitions), each partition acting
as its own standalone volume. This allows for dual-booting, disk encryption, backups and better organization.

# What are Partitioning Schemes?
**Partitioning schemes** (aka *partition table formats*) define **how data about the disk partitions are stored on the
disk**. They tell the system where a partition begins and ends, what type of partitions they are, and how many
partitions can actually exist.

There is two primary partitioning schemes, MBR and GPT.

## MBR (Master Boot Record)
Introduced in 1983, MBR is the older of the two, though still used today for legacy support.

### Main Characteristics:
- Stores partitioning info in the first 512 bytes of the disk
- Supports up to 4 primary partitions
    - One of those can be an extended partition, which can hold multiple logical partitions
- A maximum disk size of 2 TB
- The bootloader is stored in the same area as the partition table, but this makes it more fragile
- Can only be used in BIOS (legacy) boot mode

## GPT (GUID Partition Table)
The GPT was developed in the late 1990s as part of the UEFI specification developed by Intel. GPT was designed to
overcome the initial limitations of MBR and support more modern computational needs.

### Main Characteristics:
- Supports virtually unlimited partitions
- Uses 64-bit addressing, theoretically giving support for disks up to 9.4 zettabytes (9,400,000,000 TB).
- Stores multiple copies of the partition table for redundancy
- Each partition has a Globally Unique Identifier (GUID) and a type code, improving structure and recovery
- Required for the UEFI boot mode

# Sources
- [Wikipedia - Master Boot Record](https://en.wikipedia.org/wiki/Master_boot_record)
- [Wikipedia - GUID Partition Table](https://en.wikipedia.org/wiki/GUID_Partition_Table)