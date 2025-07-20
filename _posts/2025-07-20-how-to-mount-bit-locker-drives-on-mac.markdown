---
title: "How to mount BitLocker-encrypted drives on macOS"
date: 2025-07-20
categories: tutorial
---

## The Problem

Got a BitLocker-encrypted drive that you need to access on your Mac? macOS has no built-in support for Microsoft's encryption, which means you're usually out of luck. Commercial solutions exist but cost money and often have limitations.

There's now a free alternative: `anylinuxfs`. This open-source tool lets you mount NTFS (the primary Windows filesystem) on macOS with full read/write access — whether it's encrypted with BitLocker or not. It also supports Linux filesystems but that's a topic for another time.

## What You'll Need

First, you need [Homebrew](https://brew.sh) installed. If you don't have it yet, open Terminal and run:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

## Installation

Install `anylinuxfs` using these two commands:

```bash
brew tap nohajc/anylinuxfs
brew install anylinuxfs
```

That's it. The tool is now ready to use.

## Finding Your Drive

Connect your BitLocker drive and list all available Microsoft drives:

```bash
anylinuxfs list -m
```

You'll see output like this:

```
/dev/disk9 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *128.0 GB   disk9
   3:       Microsoft Basic Data                         126.6 GB   disk9s3
```

The drive shows as "Microsoft Basic Data" but it's actually encrypted. Running the same command with `sudo` reveals more details:

```bash
sudo anylinuxfs list -m
```

Now you can see it's actually BitLocker:

```
/dev/disk9 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *128.0 GB   disk9
   3:                  BitLocker DESKTOP-7EP4D9Q C: 7... 126.6 GB   disk9s3
```

## Mounting the Drive

To mount your encrypted drive, use the device identifier from the listing above:

```bash
sudo anylinuxfs /dev/disk9s3
```

You'll be prompted for your BitLocker recovery key:

```
Enter passphrase for /dev/disk9s3:
```

Enter your recovery key and the drive will mount automatically. You'll find it in `/Volumes/` with a name based on the original drive label.

### Optional: Decrypt Metadata First

If you want to see what filesystem is inside the encryption before mounting, you can decrypt the metadata:

```bash
sudo anylinuxfs list -m -d /dev/disk9s3
```

This shows:

```
Enter passphrase for /dev/disk9s3: 

/dev/disk9 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *128.0 GB   disk9
   3:            BitLocker: ntfs DESKTOP-7EP4D9Q C: 7... 126.6 GB   disk9s3
```

But this step isn't necessary — you can mount directly without it to avoid typing your recovery key twice.

## Verifying the Mount

Check that your drive mounted successfully:

```bash
anylinuxfs status
```

You should see something like:

```
/dev/disk9s3 on /Volumes/DESKTOP-7EP4D9Q_C__7-15-2025 (ntfs, uid=501, gid=20, mounted by nohajan) VM[cpus: 2, ram: 512 MiB]
```

The drive is now accessible in Finder. Note that mounted drives appear as network drives under "localhost" in the sidebar (similar to when you use "Connect to Server"). This happens because `anylinuxfs` uses a virtual machine behind the scenes to handle the filesystem operations, then shares the files back to macOS through a network protocol (NFS). The drives won't show up on your desktop, but you can access them normally through Finder.

## Unmounting

When you're done, you can unmount the drive in two ways:

1. Use the eject button in Finder (under "localhost")
2. Or use the command line:

```bash
diskutil unmount /Volumes/DESKTOP-7EP4D9Q_C__7-15-2025/
```

## How It Works

`anylinuxfs` bridges the gap between macOS and unsupported filesystems by leveraging Linux's extensive filesystem support. Since Linux can handle BitLocker decryption and NTFS operations natively, the tool creates a lightweight Linux environment to do the heavy lifting, then presents the results to macOS in a way it can understand (NFS).

## Beyond BitLocker

While this tutorial focuses on BitLocker-encrypted drives, `anylinuxfs` also works with regular unencrypted NTFS drives and of course Linux native filesystems such as ext4 or btrfs (with or without LUKS encryption), making it a versatile tool for accessing drives that macOS can't handle natively. You can visit [nohajc/anylinuxfs](https://github.com/nohajc/anylinuxfs) for more information.

## Wrapping Up

`anylinuxfs` solves the BitLocker problem on macOS without requiring expensive commercial software. The installation is straightforward, and once set up, mounting encrypted drives is just a matter of running one command and entering your recovery key.