# Working with Virtual Machines

## Creating Disk Images

- WHile it is possible to boot a VM without any storage, just as we can boot a thin client across the network or, boot a system from live CD Environment.

- Most virtual machines will need there own storage, their own virtual hard drive to boot from and save to files. 

- We can attach physical hard drives to VMs, but the most common pattern for virtual machines disks is to represent them with disk image files.

- Disk Image Files are File Systems that exist whitin a standard file on the host's hard drive. To use a disk image with our guest, we will create the file by providing some parammetere for it.

- Once, we boot our Guest System, the installer or user will create partitions as needed.

- To do this, we will use the `qemu-img` utility. We need to tell this utility 3 primary things:

  - The format of the disk image.
  - The path and name of the disk image.
  - The size of the Virtual Disk

- Starting with the formats, there ar 3 formats we have:

  - For creating a brand new image, we might work with `RAW` or `QCOW2` which are the most common formats.

  - The utility does support other types like `VHD` and so on, mostly for work with legacy images or conversions.

    - A `RAW` disk image acts like a regular block device, and on the host it takes whatever amount of space is specified for the disk image.

    - `QCOW2` stands for `QEMU Copy On Write Version 2` and is a more advanced format that allows for snapshots and other features.

      - A Copy-On-Write file system  makes a copy of disk blocks that are changed in a new location, rather than modyfing the existing ones.

      - This has some advantages like allowing easy snapshots.

      - QCOW2 images are also sparse, means they don't take up the space on the host untill data is actually written to them.

- Example:
  
  ```shell
  qemu-img create 0f qcow2 mydisk.qcow2 50G
  # We have a disk image thats ready to act like a 50GB hard drive for my guest
  # It will grow in size on the host as files are added.
  ```

- Many Virtual Guests will have 1 disk image file associated with them as there primary disk, but it is possible to have multiple disk images attached to a single VM.