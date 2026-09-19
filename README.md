# DiskDetective Storage Audit

## Objective

The objective of this project is to perform an automated storage audit on a RHEL system. The project identifies disk usage, large files, stale files, files owned by a specific user, and demonstrates hard and symbolic links. A second disk is mounted to archive identified stale files.

## Requirements

- Red Hat Enterprise Linux
- Root or sudo access
- Additional disk for archive storage

## 1. Filesystem Survey

The following commands were used to inspect disks and filesystem usage:

    lsblk
    df -h
    du -xh --max-depth=1 / 2>/dev/null | sort -h

## 2. Large Files

Files larger than 100 MB were identified using:

    find / -type f -size +100M -ls 2>/dev/null

The top 10 largest files were identified using:

    find / -type f -printf '%s %p\n' 2>/dev/null | sort -nr | head -10

## 3. Stale Files

Files that had not been modified for more than 180 days were identified using:

    find / -type f -mtime +180 -ls 2>/dev/null

The results were stored in:

    stale_files.txt

Command used:

    find / -type f -mtime +180 -print 2>/dev/null > stale_files.txt

## 4. Files Owned by a User

Files owned by the specified user were identified using:

    find / -type f -user root -ls 2>/dev/null

The username can be changed as required.

## 5. Link Investigation

A test file was created:

    echo "DiskDetective link test" > original.txt

A hard link was created:

    ln original.txt hardlink.txt

A symbolic link was created:

    ln -s original.txt symlink.txt

The inode numbers were compared using:

    ls -li original.txt hardlink.txt symlink.txt

The original file was then deleted:

    rm original.txt

The hard link continued to work because it refers to the same inode.

The symbolic link became broken because it refers to the original file path.

## 6. Archive and Mount

A second disk was partitioned and formatted with XFS.

The partition was mounted at:

    /archive

The mounted filesystem was verified using:

    df -h /archive
    lsblk

Identified stale files were copied to the archive filesystem.

## 7. Persistent Mount

The UUID of the partition was obtained using:

    blkid /dev/sdb1

The filesystem was added to `/etc/fstab` so that it can be mounted automatically after reboot.

The configuration was tested using:

    umount /archive
    mount -a

## 8. Verification

The following commands were used for final verification:

    lsblk
    df -h
    cat stale_files.txt
    df -h /archive
    ls -lh /archive

## Files

- `storage_audit.txt` – storage audit report
- `stale_files.txt` – list of files older than 180 days
- `commands.txt` – command history used during the project
- `screenshots/` – verification screenshots

## Conclusion

The DiskDetective project provides a systematic storage audit by identifying disk usage, large files, stale files, and user-owned files. It also demonstrates hard and symbolic links and provides an archive filesystem using an additional disk.