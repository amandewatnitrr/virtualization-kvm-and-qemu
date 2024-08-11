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
  qemu-img create -f qcow2 mydisk.qcow2 50G
  # We have a disk image thats ready to act like a 50GB hard drive for my guest
  # It will grow in size on the host as files are added.
  ```

- Many Virtual Guests will have 1 disk image file associated with them as there primary disk, but it is possible to have multiple disk images attached to a single VM.

## Creating a VM

- Creating a Virtual Guest with QEMU at the command line can be littile intimidating.

- Guests are made up of primarily 2 things:

  - Configuration
  - Disk Image

- We now know how to create a disk image, so now let's focus on the configuration part.

- We will use the VNC Client to connect to the VM or Guest, to see what's going on. For now, we will dive into the example that we discussed in the previous lesson.

    Example:

    ```shell
    qemu-system-x86_64 \ # Create a guest with specfied architecture we want to use
    -enable-kvm \ # Use KVM instead of QEMU Emulation
    -cpu host \ # Use the host CPU
    -smp 4 \ # Number of Virtual CPU Cores provided to the guest
    -m 8G \ # Amount of RAM provided to the guest
    -k en-us \ # Keyboard Layout
    -vnc :0 \ # Show the guest screen using VNC at host port 5900
    -usbdevice tablet \ # Attach a USB 1.1 tablet device to make mousing better
    -drive file=disk_name.qcow2,if=virtio \ # Set the disk image guest will use
    -cdrom /path/to/iso \ # Attach the installer as CD ROM
    -boot d \ # Boot from the CD ROM
    ```

  - `-enable-kvm`

    - tells QEMU that we want to use KVM and paravirtualization with this guest
    - If this opetion is not specified QEMU will emulate it, instead of virtualizing it.
    - KVM let's us take advantage of the host's support for vitualization which is much faster than emulation.
    - This feature isn't available if we are running an architecture which is different from the host.

  - `-cpu <type/>`

    - tells QEMU to present the guest with a processor of a specific type
    - specifying `host` tells QEMU to use the host's CPU features.
    - If we leave this option out, QEMU will present the guest with a generic QEMU Processor model to the guest.
    - This is useful for taking advantage of the host's CPU features.
    - If we are running a VM on a different architecture, we can specify the architecture we want to use.

  - `-smp`

    - `smp` is short for `Symmetric Memory Multi-Processing` or `Shared Memory Multi-Processing`, it refers to the architecture most modern computers have.
    - It is used to tell QEMU, how many virtual cores to the CPU cores to give to the guest.

      - For very light loads, 1 core is sometimes enough.
      - And, we leave this option off, to provide the guest one Virtual CPU or vCPU core.
      - In the example, above we created a 4 core machine we used `-smp 4`.
      - We can get more granular too like by defining the chip topology, that is where the guest thinks it's vCPUs are in one or more sockets. With one or more cores per socket and so on.
      - Most modern systems benifit atleast from having 2 cores, but it depends on the workload, and we can set this value upto 256 cores, though that won't rflect the reality of most hosts.
      - We can specify more vCPUs than our host hardware has, but request sent to those will just be scheduled across the host's available cores.

    - Another architecture we might consider using instead of SMP is `NUMA` or `Non-Uniform Memory Architecture`, which allows us to specify different amount of memory per processor.

    - If we need this sort of arrangement, feel free to explore the QEMU documentation.

  - `-m <memory_in_GB>G` or `-m <memory_in_MB>M`

    - This option is used to specify the amount of RAM to give to the guest.
    - If we leave this off, QEMU will give the guest 128 MB of RAM, too little to really work with any modern OS.
    - We can set this as a nuber of Megabytes or Gigabytes.
    - This is one of the options that is too little to change easily later.
    - As long as we don't give the guest too little memory, it shpould boot and we can tune the amount if we want to make modifications latero on.

  - `-k <keyboard_layout>`

    - This option is used to specify the keyboard layout for the guest.
    - If we leave this off, QEMU will use the default keyboard layout for the guest.
    - When we connect through VNC we'll usually need to type things, and this options makes us make sure that keyboard behaves correctly.
    - Not all systems require this, but if you're connecting from a Mac or from certain other clients, it helps avoid bugs.
    - This is useful for making sure that the guest has the right keyboard layout for the user.

  - `vnc:`

    - provides a way for us to see the screen of the guest system, if we are working on a host across the network.
    - VNC stands for `Virtual Network Computing` and is a way to see the screen of a remote system.
    - It provides a handy, if not very fast or secure method of showing a remote system screen on our local system.
    - We can use this to see anything on the guest monitor, if it had one.
    - That includes the boot process, any graphical interface, and any text or console interface that guest mmay display.
    - It's not just for graphics even though VNC is usually used for Graphical Desktop sharing, it's better to think of this VNC Server as the guest's video card output.
    - There's no VNC Server running inside the guest.
    - The Server functionality is provided by QEMU on the host. So, this VNC session will still work even if there's no OS on the guest or, if the guest isn't an OS that supports running a VNC server itself.
    - `: 0` tells QEMU to start a VNC server for the guest on port 5900 on the host.
    - The 0 refers to the last digit of 5900. If we use 1, it will start on 5901 and so on.
    - Each guest needs to have it's own VNC port on the host, if we're using this option
  - When the guest starts, we will use the VNC CLient software to connect to the host's address with appropriate port number.
  - By default, there's no VNC password, and if connecting from a Mac using it's buil-in VNC Client, we need to do certain changes to workaround to it's password prompt.
  - There are other display options we can use, fi we're running a guest on the same computer, whose screen we're using.
  - VNC will work with any guest, local or remote in any OS.

- `usbdevice <device/>`

  - changes how the system tracks the mouse cursor and often resolves issues where the host mouse is shown in one place on the screen but the guest mouse is shown on another.
  - This feature is useful when we have a guest that will be using a graphical interface, but we can leave it off in any guest that doesn't need that.

- `-drive file=file_name.file_format, if=<interfac-type/>`

  - tells QEMU which resource to mount as a drive within the guest.
  - In our example it's the disk image file, `disk_name.qcow2` that we created earlier.
  - `if` tells QEMU which interface type to use for the drive. In our example, it's the `virtio` interface, which is generally what we want to use for speed reasons.
  - `virtio` is a paravirtualized driver provided by KVM, that is faster than the default `IDE` or `SATA` interfaces.
  - There are some other options we can use like setting specifially which  bus and device the disk should appear as, various limits we want to impose and so on.

- `-cdrom`

  - The `-cdrom` option is kind of shortcut option for drive that tells QEMU to mount an ISO image as a CD-ROM device.
  - This is what we will use to install OS on the guest.
  - This option will be removed once the installation is done, and we just want to startup the system normally.

- `-boot`

  - The `-boot` option tells QEMU to boot not from the disk image, but from the CD-ROM device with the ISO image.
  - `d` means the first CD-ROM device, `c` would mean the first hard drive.
  - `a` and `b` refer to floppy drives, which are less common these days.

- Because the QEMU command runs in, and takes over a  shell session , we need to be careful about closing terminals or disconnecting SSH Sesions when running the guests.

- We can run `temux` or `screen` to keep the session alive, or we can run the command in the background with `&` at the end of the command. So, if the connection is lost, the command will keep running, and the terminal isn't killed abruptly.

## Installing a Guest OS

- Our Guest is botting up, so let's connect to it and see what's going on.
- To access, we will open a VNC Viewer. Put the address of the QEMU host and the port number we specified in the command, and press enter to connect.

    ![](./imgs/Screenshot%202024-08-11%20at%203.38.48 PM.png)

- The VNC Session sends the mouse and keyboard activity within the VNC Viewer Window to the guest so we can interact with it just we would interact with a regular computer.
- Now, Intall the Ubuntu OS as needed.

    ![](./imgs/Screenshot%202024-08-11%20at%203.43.03 PM.png)

- Once the installation is done, we will shut down the system here. Ordinarilly, We would restart a new installed system but right now, we can't remove the installer disk image without powering the guest down.

- We will close this prompt and wait for the guest to boot into the live CD Environment. Than, we click on the top right choose > power off/logout > power off.

- We need to remove the disk image otherwise the guest will start right back up in the installer, and there's no point in that we already installed.

- Back, in the terminal, once the guest is shutdown we recall our previous command that we used to create the VM, and edit it.

    ```shell
    qemu-system-x86_64 \
    -enable-kvm \
    -cpu host \
    -smp 4 \
    -m 8G \
    -k en-us \
    -vnc :0 \
    -usbdevice tablet \
    -drive file=disk_name.qcow2,if=virtio \
    ```

    <code>We will remove the `-cdrom` option and the `-boot` option.</code>

    We can also switch the boot to `c` for the first hard drive instead of `d` for the Optical drive, but than if this disk image ever moves, we will get an error even though we will not be using it.

    And, we don't need these unused options cluttering up my command. We will run this modified command and the guest will boot.

- Once the guest is booted, we will dismiss the startup screens, and there you have the Ubuntu Desktop running as a QMU Guest with KVM, and we can use it just like a regular computer.

    ![](./imgs/Selection_692-1.png)

- We will open up the terminal here, and use the command `hostnamectl`, and we can use it to see what system knows about how's it running.

    ![](./imgs/Screenshot%202024-08-11%20at%204.02.17 PM.png)

    Here we can clearly see it's showing KVM being used.

- And, if we run `lscpu`, and we can see that the guest processor thinks it's processor is Xeon Silver 4108, which is what the host actually has, and we can also see the 4 cores we specified here.

    ![](./imgs/Screenshot%202024-08-11%20at%204.04.31 PM.png)

- We will run `lspci`, and that will show us the emulated PCI devices that the guest has including the various mainboard parts, the virtio storgae device, and intel network adapter.

    ![](./imgs/Screenshot%202024-08-11%20at%204.08.23 PM.png)

- We can also run `lsusb`, and that will show us the USB devices that the guest has, including the tablet device that we added to make the mouse work better.

  And, when we reun the `lsmem` command, we can see the memory that the guest has, and we can see that it's 8GB, which is what we specified.

  ![](./imgs/Screenshot%202024-08-11%20at%204.10.27 PM.png)

- One thing you might notice is that the display is not very good, we can improve the display a little bit by changing the resolution, but the emulated video adapter is let's say rather basic, and it has only 16MB of video RAM. So, that prevents us from having really smooth graphics or high resolutions.

  VNC limits us as well because it provides quite a less video bandwidth than a real video connection like HDMI or Display Port would have.

  This kind of basic display is fine for administrative tasks, but it's not nice to do a lot of work in.

  QEMU does provide other emulated video adapters, and we change their parameters to a degree or if we're feeling very adventurous, and have a compatible video card, we can pass through a video card to the guest. So, it can use the card, and a real monitor.

- We can make the copy of a disk image using the command:

    ```shell
    cp disk_name1.qcow2 disk_name2.qcow2
    ```

## Control and Debug a guest with QEMU Monitor

- One feature of QEMU is it's text based console or monitor. instead of starting a guest and have nothing happen in our terminal, we can instead connect the monitor console to it with the option `-monitor stdio`.

    ```shell
    qemu-system-x86_64 \
    -enable-kvm \
    -cpu host \
    -smp 4 \
    -m 8G \
    -k en-us \
    -vnc :0 \
    -usbdevice tablet \
    -drive file=disk_name.qcow2,if=virtio \
    -monitor stdio
    ```

- Now, we have a little command line interface available here, that we can use to do all sorts of useful things like add and, remove devices from the guest, pause and resume it, and shut it down by sending various control signals.

- The monitor also let's us deep dive into debugging a guest including things like inspecting a guest's memory contents.

- For example, we can change the vnc password for the guest, if we're using one with the command `change vnc password`.

    Example:

    ```shell
    qemu-system-x86_64 \
    -enable-kvm \
    -cpu host \
    -smp 4 \
    -m 8G \
    -k en-us \
    -vnc : 0, password=on  \
    -usbdevice tablet \
    -drive file=disk_name.qcow2,if=virtio \
    -monitor stdio
    ```

    This is nessecary to set passoword for use with a MacOS built-in VNC Viewer because that software won't connect to our guest's VNC interface without a password.

    In order to set a password we'll need to tell QEMU tho use a VNC Password with an additional parameter `password=on` on the VNC option.

- `system_powerdown` is a command that we can use to shut down the guest, and it's the same as pressing the power button on a physical computer. It sends ACPI power down signal instead of just ending it's process, and terminating it abrutly.

- The Monitor is pretty useful specially when customizing guest's configuration because it often saves me the time of finding a VNC Window, and powering off the guest to change a configuration option.

## Managing and Modifying Disk Images

- One useful way of using disk images with guest involve overlays.
- An Overlay is where we start with one disk image, that we call backing image, and lay another disk image on top of it.
- So, any changes made to the guest filesystsem are stored in that overlay file, and not in the orignal backing image.

    ![](./imgs/Screenshot%202024-08-11%20at%207.19.37 PM.png)

- To use an overlay, we would point the guest to the overlay instead of the orignal.
- Creating an Overlay looks like this:

    ```bash
    qemu-img create -o backing_file=filename.qcow2,backing_fmt=qcow2 -f overlay.qcow2
    ```

- Overlay makes it easy to rollback to a known orignal state quickly by creating a new overlay with the orignal backing image.
- It allows us to use 1 copy of a disk image with a base OS or pre-configured template installed as backing store for more than 1 guest, saving disk space on the host.
- If we have multiple, similar or  identical guests running at the same time. Each guest boots from a combination of the same immutable image, and whatever other changes have been applied in the overlay, and there individual changes are each stored in their own overlay file.

- However, this arrangement can get a bit messy, if we plan to frequently update our guest's OS.
- The Orignal Image will stay unchanged and, the other changes will exist on the overlay and, after a few OS upgrades, we will loose the benefit of having this space efficient overlay setup.
- So, consider using overlays if you need templated systems that won't change much, and generate new base images with any large updates.
- We might also consider using an overlay when we are experimenting and getting things just right, and than condense it into aunified image for regular use.
- If we copy our disk images somewhere else, we will need to copy both the backing image and the appropriate overlay. And, will need to update the overlay with the path to the backing image on the new host, if the backing image is at a different path.

    We would do this with the option `rebase -b /path/tobacking_image overlay .qcow2`.

    But, again we don't need to do that right now. It's also useful to use `qemu-img` with the `info` option to see what's going on with a disk image.

    ```shell
    qemu-img info disk_name.qcow2
    ```

    And, if I am working with an overlay, I can see the overlays backing image.

- While a disk image has a specific maximum capcity that we define, disk images can be resized later. We can do so using the `resize` option.

    ```shell
    qemu-img resize +100G disk_name.qcow2
    # This adds 100GB to the disk image to the existing size.
    # +100 Adds 100 more to the existing
    # 100 would resize it to 100GB
    ```

    This will resize the disk image by adding 100GB to the existing memory, and the disk image will grow in size on the host as files are added to the guest.

    We need tools within the guest to resize our partitions and  file systems to use the new space.

    If we reduce the size of an image, we'll need to start out with resizing file systems and partitions and than resize the image. Again, we can specify a fixed size or use minus sign in an amount to reduce disk capacity that way.

    Disk Operations can put data at risk, so it's a good idea to have a backup of our guest files or, better keep a duplicate copy of disk image file, we're working with just in case.

- The `qemu-img` utility is a powerful tool and provides many more features. `qemu` provides a tool that allows us to mount guest disk images should we need to do that for maintainence or data recovery purposes.

- The `qemu-nbd` utility attaches a disk image to our host as a network block device, and than we can mount it as we would any other network storage device and read and write it, depending on the file system within it.

    But, for this we need to make sure that the Network Kernel Module is loaded with modprobe `nbd`. For this we can use the command:

    ```shell
    sudo modprobe nbd
    ```

    and, we will use `qemu-nbd` to attach the disk image to the first entry for Network Block Devices. If we need to work with more than one at a time, we can mount images on subsequent devices, nbd1, nbd2 and so on.

    ```shell
    sudo qemu-nbd --connect=/dev/nbd0 /path/to/disk_imgs/disk_name.qcow2
    lsblk -f /dev/nbd*
    ```

    `lsblk -f` will show us the block devices on the system, and we can see the disk image mounted as a network block device in our file systems.

    ![](./imgs/Screenshot%202024-08-11%20at%208.53.52 PM.png)

    On `nbd03`, we can see there's an `ext4` partition. That's our guest root file system. We'll create a space for partition to be mounted with the following command:

    ```shell
    sudo mkdir /mnt/my_disk_name
    ```

    and mount it with the command:

    ```shell
    sudo mount /dev/nbd0p3 /mnt/my_disk_name
    ```

    Now, we can explore the filesystem, and work with it the same way We would be working with any other disk. And, than when I am done using that resource, I'll unmount it from my file system with command:

    ```shell
    sudo umount /mnt/my_disk_name
    # We also need to use qemu-nbd and detach the disk image from the device that we created for it.
    sudo qemu-nbd -d /dev/nbd0
    ```

## Guest Graphics and Display Options

- Our guest has been configured, so we can view it screened though VNC on any stream attached to our host's network.
- But, VNC is a little bit limiting when it comes to image quality and responsiveness,especially on a slow or busy network.
- If our QEMU host has a desktop interface, we can use the guest locally inside a desktop window, instead of through VNC.
- The QEMU software will open up it's own window to show the guest. This means we don't need to use VNC Software, or even rely  on the network, to view and use our guest.
- It also means we can get much better video performance out of our VM.
- In many ways, it can behave almost as well as a native system.
- To change the way that QEMU displays the guest, we will use the `-display` option, and we will often combine that with the `-vga` option, which allows us to set which emulated or paravirtualized video device the guest will have.
- `-display` determines where we see the guest's screen, and we think of that as the actual display or monitor, and `-vga` determines what kind of video adapter is used, that's like the guest's video card.
- By default, without any options defining the display or the video adapter, the guest on the linux host will start up a GTK or GNOME toolkit window, and a standard VGA Adapter.
- We've specified VNC in our intial setup here, and that's a shortcut for display VNC.
- Example:

    ```shell
    qemu-system-x86_64 \
    -enable-kvm \
    -cpu host \
    -smp 4 \
    -m 8G \
    -display gtk \
    -k en-us \
    -usbdevice tablet \
    -drive file=disk_name.qcow2,if=virtio \
    -monitor stdio
    ```

    <code> We will remove the `-vnc` option here from our previous example, that we reffered to.</code>

- We can use one type of display at a time. The choices we have available for the display option include GTK, SDL, VNC, curses and none.

  - `none` attaches no display to the VM, while that might sound odd, we will still be able to connect to the guest via the network, if we have that setup, and we can control the guest machine from the QEMU monitor.

  - Many servers and other types of guests won't need a display duing normal operation.
  
  - `curses` option presents the guest as a text interface, which is suitbale for text console based guests, but not for those running a GUI.

  - `vnc` option is what we used in the previous example, and it's a good choice for remote access to the guest, and it's the most common choice for GUI based guests.

  - `sdl` stands for `Simple DirectMedia Layer`, and it's a library that provides a simple way to create graphics and sound in a program. And, it's usually what we might use on a non-Linux Host to draw a local window for te guest, though it works on Linux too.

  - `gtk` is the default option. GTK stands for GNOME Toolkit, and it's a library that provides a way to create graphical user interfaces in a program. It's what we might use on a Linux Host to draw a local window for the guest.

  While it is the default and, we actually don't need to type it out, it's good to know what it is.

- `gtk` and `sdl` are good choices for local guests to draw the guest window with OpenGL, through the `gl=on` or `gl=off` parametetee. OpenGL supports a variety of Video Card vendor.

- `vnc` is a good choice for remote guests.

- `-vga` is important in determining what video card guest has. VGA stands for Video Graphic Adapter. The default is standard VGA, with a supported resolution of upto `1920 * 1080`. This adapter doesn't have a lot of fancy features or a lot of memeory, but it's widely supported.

  We will also discuss about the `virtio` option though. With KVM on linux, we can use paravirtualized video adapter through virtio. This device offers good performance, but it requires a guest to have virtio drivers installed in order to take full advantage of the device.

  If you are using a graphics heavy host, and a local guest display , `virtio` is probably the best choice.

  If you just need a screen to see what's going on, standard VGA is probably fine.

- Examples:

    ```shell
    qemu-system-x86_64 \
    -enable-kvm \
    -cpu host \
    -smp 4 \
    -m 8G \
    -display sdl \
    -vga virtio
    -k en-us \
    -usbdevice tablet \
    -drive file=disk_name.qcow2,if=virtio \
    -monitor stdio
    ```

    <code> `virtio` works just fine with the GTK Display, but like we mentioned, GTK gets a bit weird with scaling. </code>

    It's always good to turn of your machine, if you are using a local video mode.

### Trobulshooting Tip: Black Screen

- If you're modifying graphics and display settings, and guest starts up to a black or grey screen, it may be the case that the resolution is too high for the current display adapter.

- `1024*768` or any lower resolution is a safe starting point, and we can increase it from there.

- The display type and adapter we you will use will be determined by what the guest is intended to do.