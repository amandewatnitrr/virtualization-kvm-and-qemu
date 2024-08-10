# Understandng QEMU and KVM

## QEMU

- QEMU stands for Quick Emulator is a software that runs on a host computer and acts as a hardware emulator for guest VMs.
- It presents the guest with emulated hardware, it needs to opertae as a computer.
- THe QEMU translates information sent to emulated devices and processes it using host's system hardware.

- The Emulated Devices fall into 2 primry category:
  - Processors and CPU
  - Others

  ![](./imgs/Screenshot%202024-08-09%20at%202.24.01 PM.png)

- QEMU can emulate a huge variety of X86_64 processors models old and new.
- We can allow or block specific CPU features to the guest VMs.
- QEMU can emulate many other processor architectures like ARM, MIPS, PowerPC, and SPARC.

- QEMU handles translating CPU calls into instructions that can be executed on the host CPU, the results are sent back to the guest VM.
- We can select architecture and specific target based on our needs.
- QEMU also emulates any other device that the guest VM might need to operate, this can include mainboards, expansion cards, input devices and more.

- The emulated devices are standard and so well supported means that we can run an unmodified guest OS. That is we can take a installer that would be installed on a bare metal machine and install it as a Virtual Guest Instead.

### Operating Modes

- QEMU can run in two modes:

  - Full System Emulation

    - The whole system is emulated
    - Runs a virtualised guest OS
    - The guest OS is unaware that it is running in a virtualised environment

  - User Mode Emulation

    - It is only available on Linux Host.
    - QEMU runs a single process or program, not a whole OS.

## Virtualization with KVM

- KVM is a feature of the Linux Kernel that enables a linux system to act as a type 1 hypervisor to run virtual machines.
- This feature allows virtual machines direct access to host's processor instead of relying on an emulated processor, providing native performance without the overhead of emulation.
- KVM is available on CPU, that have a CPU that supports virtualization.
- KVM is available on AMD and Intel Systems whose CPU supports virtualization.
- Intel calls it's virtualization feature `VT-x` and AMD calls it `AMD-V`.
- Virtualization can be enabled in System's BIOS or startup environement, and it needs to be turned ON for KVM to work.

- To see whether your linux machine supports kvm, check whether the entry dev KVM exsists, or we can run:
  
  ```bash
  lsmod | grep kvm
  ```

  and check if the mod appears in the output.

- If your system supports virtualisation, but doesn't have KVM enabled, you can enable it by running:

  ```bash
  sudo modprobe kvm
  ```

- Using Virtualization, A Guest Operating System has access to the host's processor and it's features. The Guest OS must be able to run on Host's system architecture.

- We can't run a ARM OS or Power PC guest on a AMD or Intel host which is based X86_64 Architecture, we would need to use QEMU to emulate the processor.

- With a feature called Nested Virtualization, we can run a hypervisor inside a VM, which is running on a hypervisor. This means guest OS can run it's own VMs, and have access to the Host CPU.

- KVM provides guests with access to a protected portion of the Hosts RAM as well.

- In addition to providing access to the Host CPU and Memory, KVM also provides another service called VirtIO(Virtual Input Output), which is a set of drivers that allow the guest to communicate with the host.

- VirtIO enables a virtualization mode called `paravirtualization`, which is different from full virtualization. Becausem in paravirtualization, the guest OS is aware that it is running a virtualized environment, and makes use of the `hypervisor` features.

- In full virtualization, the guest OS is unaware that it is running in a virtualized environment, it just acts like a isolated box.

- Guest OS needs to have support for VirtIO drivers to use the VirtIO devices in order to use it. Though some OS need to have these drivers installed to take advantage of these features.

- In most cases, `paravirtualization` is faster than `full virtualization`, because the guest OS can communicate directly with the host, instead of going through an emulated device.

## Using QEMU & KVM

- KVM provides access to the host's CPU and Memory, and QEMU provides the emulated devices that the guest OS needs to operate with paravirtualization.

- But using KVM also means that we can only run guest OS that are compatible with the host's CPU architecture.

- We need to install some tool to use QEMU on linux, and these tools will come from a distribution repository. To view the list use the command:
  
    ```shell
    apt list qemu*
    ```

- We will install the `qemu-kvm` package, which is a meta package that installs all the necessary tools to use QEMU with KVM.

  ```shell
  sudo apt install qemu-kvm
  ```

- Once installed type `qmeu-` and press `tab` to see the list of available commands with matching prefix.

  ![](./imgs/Screenshot%202024-08-09%20at%2010.58.32 PM.png)

  - Let's explore some of these commands:

    - `qemu-img` is a tool to creating and working with disk images.

      - It can be used to create disk images of various formats.
      - It can be used to convert disk images from one format to another.
      - It can be used to resize disk images.
      - It can be used to inspect disk images.

    - `qemu-io` is another utility for working with disks.

      - It can be used to test disk performance.
      - It can be used to perform disk operations like read, write, flush, etc.

    - `qemu-nbd` is for working with Network Block Storage.

      - Network Block Storage is a way to access remote storage over a network.
      - It can be used to mount remote disk images on the local system.
      - It can be used to create a network block device.
      - It can be used to export a local disk image over the network.

    - `qemu-system-XXXX` where XXXX represents the architecture of the guest OS, are the commands use to emulate a specific processor type.

      - For example, `qemu-system-x86_64` is used to emulate an X86_64 processor.
      - `qemu-system-arm` is used to emulate an ARM processor.
      - `qemu-system-spice` is used to emulate a spice processor.
      - When using these commands, we can pass options to specify the guest OS, the amount of memory, the number of CPUs, the disk image, and more.
      - We also need to kepp a note in mind that these commands can be extremly long and fairly complex, and they are not something we'll usually type out from beginning to end.
      - Planning these commands take time, and often suggested to be composed in a note document to make sure we have all the things we need.
      - Then we must assemble the parts into a long command string, that we can copy and paste into the terminal.
      - It's also recommended to use shell line continuation character `\` to break the command into multiple lines for better readability.
      - Example:

        ```shell
          qemu-system-x86_64 \
          -m 512M \
          -hda /path/to/disk.img \
          -cdrom /path/to/iso \
          -boot d
        ```

      - The order of the options doesn't really matter, but it's a good idea to keep them in a logical order for easier reading in an inside outward order.

## Exploring QEMU Documentation

- QEMU has a lot of options and features, and it can be overwhelming to try and learn them all at once.

- The best way to learn is to start with the basics and then explore the documentation as needed. You can access the QEMU documentation by running:

  ```shell
  man qemu
  ```

  - It starts with listing what type of peripherals and devices QEMU can emulate. And, than comes the long list of options that we can use with any of the QEMU system binaries.

  - If you want to search anything in the documentation, you can use the `/search_term` key to search for the `search_term`.

  - You can find the same information on the <a href="https://www.qemu.org/docs/master/system/index.html">`website`</a>, under the `invocation` section.

## Planning a Virtualization Soution

- Planning is an important part of setting up a virtualization solution regardless of the size of your virtual machine deployement.

- We need to plan out what resources our virtual machine will have. This includes:

  - The amount of memory the VM will have.
  - The number of CPUs the VM will have.
  - The amount of disk space the VM will have.
  - The type of disk image the VM will use.
  - The type of network connection the VM will have.
  - The type of devices the VM will have.
  - Emulated Hardware like Video and Network Adapters.

- A QEMU lab can quickly take up a lot of space as we work with Operating System installers and disk images that guests run from.

- And, if we use features like snapshot, space can be consumed more easily than we might expect.

- While we are learning, we can store disk images pretty much anywhere, but if we are a deploying a lab or a production solution.

- We'll want to organized these images carefully on dedicated storage and make sure that we have a responsible backup plan in case of a problem. 

- SSDs are a good medium for image storage but they can get expensive if the capacity needs to be large.

- Spinning or Magnetic Disks are a good alternative for storing disk images, but they are slower than SSDs.

- Various tools in the QEMU system will allow us to backup the image files via snapshots or other means.

- Guests will all compete for memory on host, so it's important to think about how much RAM each guest will need, and how to reserve for host's own operations.

- When we allocate memory to a guest, that memory isn't all take my the guest at once. The guest only takes what it needs at any given time.

- So, we can have 2 guests running on a host with 8GB RAM, and provide them each 8 GB of RAM.

- As long as the guest and the host don't do anything memory-intensive, QEMU will handle this situation just fine.

- But, if the machine starts competing for that over-provisoned memory, both the guests and the host will suffer in performance.

- So, it's usually best to better think of part of the host's overall  memory as being available to allocate the guests.

- At least a few gigabytes of memory for the host, outside of what's allocated to the guests.

- We also need to determine what processor model and, how many processor cores a guest should have.

- QEMU provides emulated hardware that can be the same regardless of which host a guest is running on.

- The proessor however might vary b/w different virtualization hosts, and so instead of using a direct pass-thorugh to whatever model processor a particular host has, whih is common practice on a single machine or in a lab, we may need to choose an emulated processor model, that will allow guests to have same CPU across a fleet of hosts.

- But, if we have 2 or more hosts with different CPU models, and plan to run guests in a way where they could start on any machine in the cluster.

- A very basic Linux Guest with no desktop environement like a clean install of Alpine Linux may only need 1 GB RAM and 1 CPU core.

- But, a desktop distribution or another more general purpose OS like windows will need more space for disk image certainly will need more RAM, probably 4 GB RAM at minimum, and will benifit from having 2 or more core processors.

- And depending on the role of the guest, will decide which hardware it has, what its network topology is and, whether it needs additional devices connected, either emulated ones or real hardware from the host.

- All these resources need to be planned, and tuned to what's appropriate. Guests will widely vary depending on what they need to do.

- We also need to decide on how we will be acdessing the guest, we can connect to the guest and, send it mouse and keyboard input either through a local window or through VNC across the network.

- And, as we configure our guest, we might choose to establish other kinds of remote access, such as SSH.

- Luckily, we can change all of the guest's parameter except for the target architecture, after we create the guest.

- If we find we have underestimated or overestimated what a guest actually needs.

- It's useful to think of QEMU guests as having 2 primary parts, the disk image from which they run, and the configuration that determines the virtual hardware and configuration of the guest system.

- The configuration of a machine can be defined at the command line, and can also be stored in files in various ways to be managed manually or through Guest Management Solutions like virsh and Virt-Manager.

- And, for larger deployments we can use orchestration tools to mange the configuration and,deployment of guests across a series of host systems.

- The QEMU default emulated hardware is usually a responsible starting point for guests unless we have other specific needs or optimizations to make.

- linux generally have a good support for these devices, as mentioned earlier, and some other OS do as well, though if we make changes to the defaults we will need to consider whether a guest needs drivers to  support those different devices.

- In a lab we can virtualizae pretty much any system we might want to, but in a production deplyment, there are roles that we don't want to virtualize.

  - For example, we shouldn't have all of our core infrastructure virtualised, specially things like Primary Domain Controllers, Core Routing, or Primary DNS.

  - If our virtualization solution goes down for some reason, this can create a circular problem for getting our network back up and running.

  - It's also improtant to consider licensing when we are creating guest systems.

  - While Operating Systems like Linux and FreeBSD don't require per-system licenses, other OS like Windows do.

  - Just because a system is running in a virtualizaed environment doesn't mean we don't have to pay for the license.

  - And, sometimes running a guest on a non-vendor approved hardware can violate OS Licenses.
  
- Sometimes,  we will virtualize systems to take better advantage of hardware resources, and in many cases managing system easier and save on costs, and sometimes for resiliency and recoverability.

- Having virtual machines to be started up quickly on a new hardware, if a host system is having problems is a lot easier than re-building a bare metal server.