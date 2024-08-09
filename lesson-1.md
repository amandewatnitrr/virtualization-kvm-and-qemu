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

- 