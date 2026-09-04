# Sleuth Kit (TSK) Deleted-File Recovery Lab

## Objective

Perform deleted-file recovery using **The Sleuth Kit (TSK)** on
Ubuntu/WSL2 using a dedicated test disk image named `recovery.img`.

The exercise uses a safe forensic lab image and demonstrates:

-   Disk-image identification with `img_stat`
-   Partition identification with `mmls`
-   Filesystem analysis with `fsstat`
-   Deleted-file identification with `fls`
-   Deleted-file metadata inspection with `istat`
-   Specific-file recovery with `icat`
-   Bulk recovery with `tsk_recover`
-   SHA-256 verification of recovered data

> **Safety:** All filesystem modifications in this lab are performed
> only on `recovery.img`. During forensic analysis, the image is treated
> as read-only. Never substitute a real physical device such as
> `/dev/sda` or `/dev/nvme0n1`.

------------------------------------------------------------------------

## 1. Verify The Sleuth Kit Installation

Check whether `img_stat` is installed:

``` bash
which img_stat
```

Expected:

``` text
/usr/bin/img_stat
```

Check the TSK version:

``` bash
img_stat -V
```

Example:

``` text
The Sleuth Kit ver 4.12.1
```

This confirms that The Sleuth Kit is installed and available.

------------------------------------------------------------------------

## 2. Create the Dedicated Lab Directory

Create and enter the lab directory:

``` bash
mkdir -p ~/file-recovery-sleuth-kit
cd ~/file-recovery-sleuth-kit
```

All files created for the exercise remain inside this dedicated lab.

------------------------------------------------------------------------

## 3. Create the Test Disk Image

Create a 100 MB raw image:

``` bash
dd if=/dev/zero of=recovery.img bs=1M count=100
```

This creates the test disk image.

Verify it:

``` bash
ls -lh recovery.img
```

### Important

`recovery.img` is the only disk image used in this exercise. No
real/system disk is used.

------------------------------------------------------------------------

## 4. Create an MBR Partition Table

Open the image with `fdisk`:

``` bash
fdisk recovery.img
```

Inside `fdisk`, create one primary partition:

``` text
n
p
1
Enter
Enter
```

The resulting partition starts at sector `2048` and extends to the end
of the image.

Write the partition table:

``` text
w
```

This modifies only `recovery.img`.

------------------------------------------------------------------------

## 5. Identify the Partition Offset Using `mmls`

Run:

``` bash
mmls recovery.img
```

Example output:

``` text
DOS Partition Table
Offset Sector: 0
Units are in 512-byte sectors

      Slot      Start        End          Length       Description
000:  Meta      0000000000   0000000000   0000000001   Primary Table (#0)
001:  -------   0000000000   0000002047   0000002048   Unallocated
002:  000:000   0000002048   0000204799   0000202752   Linux (0x83)
```

The important value is:

``` text
Start = 2048
```

### Why `mmls` matters

The partition offset should not be guessed or blindly assumed during
forensic analysis.

`mmls` reads the partition table and tells us exactly where the
filesystem partition begins.

We use this value with the other TSK commands:

``` text
-o 2048
```

------------------------------------------------------------------------

## 6. Create the Filesystem

For this lab, a FAT32 filesystem is used because deleted FAT directory
entries retain useful information for a simple metadata-based recovery
demonstration.

Install the FAT filesystem tools if necessary:

``` bash
sudo apt install dosfstools
```

Create FAT32 inside the partition beginning at sector 2048:

``` bash
sudo mkfs.fat -F 32 -S 512 --offset 2048 recovery.img
```

This modifies only `recovery.img`.

------------------------------------------------------------------------

## 7. Verify the Filesystem with `fsstat`

Run:

``` bash
fsstat -o 2048 recovery.img
```

The output should identify the filesystem as FAT32.

### Purpose of `fsstat`

`fsstat` displays filesystem-level information such as:

-   Filesystem type
-   Sector/block information
-   Metadata structures
-   Allocation information
-   Filesystem layout

For file recovery, this helps confirm that TSK is interpreting the
filesystem correctly before examining individual files.

------------------------------------------------------------------------

## 8. Mount the Test Filesystem

Create a mount point:

``` bash
mkdir -p mnt
```

Mount the filesystem using the partition offset obtained from `mmls`:

``` bash
sudo mount -o loop,offset=$((2048*512)) recovery.img mnt
```

The calculation converts the sector offset into bytes:

``` text
2048 × 512 = 1048576 bytes
```

------------------------------------------------------------------------

## 9. Create a Sample File

Create the file that will later be deleted:

``` bash
printf 'Sleuth Kit deleted file recovery test.\nThis file should be recoverable.\n' | sudo tee mnt/recovery_data.txt > /dev/null
```

Verify its size:

``` bash
sudo wc -c mnt/recovery_data.txt
```

Expected:

``` text
72 mnt/recovery_data.txt
```

Flush pending filesystem writes:

``` bash
sync
```

------------------------------------------------------------------------

## 10. Delete the Sample File

Delete the test file:

``` bash
sudo rm mnt/recovery_data.txt
```

This is an intentional deletion performed only inside `recovery.img`.

Unmount the image:

``` bash
sudo umount mnt
```

From this point onward, `recovery.img` is treated as read-only for
forensic analysis.

------------------------------------------------------------------------

# 11. Identify the Deleted File with `fls`

Run:

``` bash
fls -o 2048 -r -d recovery.img
```

The important output from the lab was:

``` text
r/r * 5:        recovery_data.txt
```

### Interpretation

-   `fls` = filesystem listing tool
-   `-o 2048` = filesystem starts at sector 2048
-   `-r` = recursively examine directories
-   `-d` = display deleted entries
-   `*` = deleted/not allocated entry
-   `5` = the TSK directory-entry identifier
-   `recovery_data.txt` = original filename

This is the first major recovery step: **finding the deleted file
through filesystem metadata.**

------------------------------------------------------------------------

# 12. Inspect Deleted File Metadata with `istat`

Run:

``` bash
istat -o 2048 recovery.img 5
```

The lab produced:

``` text
Directory Entry: 5
Not Allocated
File Attributes: File, Archive
Size: 72
Name: _ECOVE~1.TXT

Directory Entry Times:
Written:        2026-09-04 17:45:44 (UTC)
Accessed:       2026-09-04 00:00:00 (UTC)
Created:        2026-09-04 17:45:45 (UTC)

Sectors:
3153
```

### Important recovery information

The metadata tells us:

``` text
Status:  Not Allocated
Size:    72 bytes
Sector:  3153
```

The file is deleted, but the metadata still provides enough information
for TSK to locate its contents.

### Purpose of `istat`

`istat` examines the metadata associated with a file entry.

For recovery, it can help determine:

-   Whether the entry is allocated or deleted
-   File size
-   File timestamps
-   Data-sector information
-   Other filesystem metadata

------------------------------------------------------------------------

# 13. Recover a Specific Deleted File with `icat`

Create a separate recovery directory:

``` bash
mkdir -p recovered
```

Recover the file:

``` bash
icat -o 2048 recovery.img 5 > recovered/recovery_data.txt
```

### What `icat` does

`icat` reads the data associated with a specified filesystem metadata
entry and outputs the file contents.

In this case:

``` text
recovery.img
    ↓
partition offset 2048
    ↓
deleted directory entry 5
    ↓
file data
    ↓
recovered/recovery_data.txt
```

Verify the recovered contents:

``` bash
cat recovered/recovery_data.txt
```

Expected:

``` text
Sleuth Kit deleted file recovery test.
This file should be recoverable.
```

Verify the size:

``` bash
wc -c recovered/recovery_data.txt
```

Expected:

``` text
72 recovered/recovery_data.txt
```

At this point, **specific deleted-file recovery using TSK has
succeeded.**

------------------------------------------------------------------------

# 14. Bulk Recovery with `tsk_recover`

`tsk_recover` is used when multiple deleted files need to be recovered
from a filesystem.

Create a separate destination:

``` bash
mkdir -p recovered_all
```

The basic recovery command is:

``` bash
tsk_recover -o 2048 recovery.img recovered_all
```

The command examines the filesystem and exports recoverable files into
the destination directory.

### Difference between `icat` and `tsk_recover`

`icat`:

``` text
Recover one known filesystem entry
```

Example:

``` bash
icat -o 2048 recovery.img 5 > recovered/recovery_data.txt
```

`tsk_recover`:

``` text
Recover multiple files from the filesystem
```

Example:

``` bash
tsk_recover -o 2048 recovery.img recovered_all
```

`tsk_recover` is therefore more convenient when the goal is to recover a
collection of files rather than one specific inode/directory entry.

------------------------------------------------------------------------

# 15. SHA-256 Verification

Hashing allows us to compare the original file with the recovered file.

## 15.1 Hash the original before deletion

In a real lab, the original hash should be calculated **before deleting
the file**.

For this exercise, the original file contents were:

``` text
Sleuth Kit deleted file recovery test.
This file should be recoverable.
```

If the original file is recreated for verification, hash it before
deleting it:

``` bash
sha256sum mnt/recovery_data.txt
```

Example procedure:

``` bash
printf 'Sleuth Kit deleted file recovery test.\nThis file should be recoverable.\n' > original_recovery_data.txt
sha256sum original_recovery_data.txt
```

Then compare it with:

``` bash
sha256sum recovered/recovery_data.txt
```

The SHA-256 values should be identical.

For example:

``` text
<original SHA-256>  original_recovery_data.txt
<same SHA-256>      recovered/recovery_data.txt
```

### Why hashes are useful

If the hashes match, the recovered file's contents are identical to the
original file's contents.

This provides a stronger verification than simply opening the recovered
file and visually checking it.

------------------------------------------------------------------------

# 16. TSK Commands Used in This Lab

  -----------------------------------------------------------------------
  TSK Command                         Purpose in File Recovery
  ----------------------------------- -----------------------------------
  `img_stat`                          Identifies the disk-image type and
                                      basic image properties

  `mmls`                              Identifies partitions and their
                                      starting sectors

  `fsstat`                            Examines filesystem structure and
                                      confirms filesystem interpretation

  `fls`                               Lists filesystem entries and
                                      identifies deleted files

  `istat`                             Examines metadata for a specific
                                      deleted file

  `icat`                              Recovers the contents of a specific
                                      filesystem entry

  `tsk_recover`                       Recovers multiple files from a
                                      filesystem
  -----------------------------------------------------------------------

The central recovery chain is:

``` text
mmls
  ↓
Find partition start sector
  ↓
fsstat
  ↓
Confirm filesystem
  ↓
fls -d
  ↓
Find deleted file
  ↓
istat
  ↓
Inspect metadata and data location
  ↓
icat
  ↓
Recover specific file
```

For multiple files:

``` text
mmls
  ↓
fsstat
  ↓
tsk_recover
  ↓
Recovered files
```

------------------------------------------------------------------------

# 17. Filesystem-Based Recovery vs Raw File Carving

## Filesystem-Based Recovery

Filesystem-based recovery uses the filesystem's existing metadata.

Typical workflow:

``` text
Deleted file
    ↓
Filesystem metadata still available
    ↓
fls identifies deleted entry
    ↓
istat examines metadata
    ↓
icat/tsk_recover retrieves file data
```

Advantages:

-   Can preserve the original filename
-   Can identify file metadata
-   Can provide timestamps and file size
-   Can directly locate file data
-   Usually provides better context about the deleted file

The recovery performed in this lab is **filesystem-based recovery**.

## Raw File Carving

Raw carving does not depend primarily on filesystem metadata.

Instead, a carving tool searches raw disk/image data for known file
signatures and structures.

For example:

``` text
Raw image
   ↓
Search for file signatures/patterns
   ↓
Identify possible file contents
   ↓
Extract/carve files
```

Carving can sometimes recover files even when filesystem metadata has
been destroyed.

However, it may not recover:

-   Original filename
-   Original directory
-   Original timestamps
-   Complete filesystem context

### Key difference

  -----------------------------------------------------------------------
  Filesystem Recovery                 Raw Carving
  ----------------------------------- -----------------------------------
  Uses filesystem metadata            Searches raw data

  Can preserve filenames              Usually cannot preserve original
                                      filenames

  Uses structures such as directory   Uses file signatures/patterns
  entries/inodes                      

  `fls`, `istat`, `icat`,             Specialized carving tools
  `tsk_recover`                       

  Best when filesystem metadata       Useful when metadata is
  survives                            unavailable/destroyed
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# 18. Safety Rules for the Lab

Always use the dedicated image:

``` text
recovery.img
```

Do not replace it with:

``` text
/dev/sda
/dev/nvme0n1
```

or any unknown physical device.

During setup, commands such as:

``` text
dd
fdisk
mkfs.fat
mount
rm
```

modify the test environment.

During forensic analysis:

``` text
img_stat
mmls
fsstat
fls
istat
icat
tsk_recover
```

should be used against the test image without modifying it.

Recovered output should be written to separate directories such as:

``` text
recovered/
recovered_all/
```

This keeps the original evidence image separate from recovered
artifacts.

------------------------------------------------------------------------

# 19. Final Result

The lab successfully demonstrates the core Sleuth Kit deleted-file
recovery process:

1.  Create a dedicated forensic disk image.
2.  Create a partition table.
3.  Use `mmls` to identify the actual partition offset.
4.  Create a filesystem inside the partition.
5.  Create a sample file.
6.  Delete the sample file.
7.  Unmount the image.
8.  Use `fls` to identify the deleted file.
9.  Use `istat` to inspect its metadata.
10. Use `icat` to recover the specific deleted file.
11. Use `tsk_recover` for bulk recovery.
12. Use SHA-256 hashes to verify recovered data.

The successful recovery demonstrated in this lab was:

``` text
recovery_data.txt
        ↓
deleted
        ↓
fls
        ↓
Directory Entry 5
        ↓
istat
        ↓
72 bytes, sector 3153
        ↓
icat
        ↓
recovered/recovery_data.txt
```
