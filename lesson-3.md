# Network Management

## User-mode Netoworking

- So, far our guest has been using a Private Network Address translation or NAT network that contains only it, and our host. This is called `User-mode networking`.

  The host here acts as a router or gateway and, as a DHCP Server at the address on this private network.

  This provides the guest access to the hosts network and, the internet, but other clients on the hosts network or beyond won't be able to access our guest now.

  Within the guest, we can access the host itself as well, if we need to access the services host is running.

  The Host also acts as a forwarding DNS Server located at some IP Address on the private network. Should, we choose to configure it, the host also can provide SMB file sharing at some other IP Address. To verify all this use this command to bring the guest up:

  ```bash
  qemu-system-x86_64 \
  -enable-kvm \
  -cpu host \
  -smp 4 \
  -m 8G \
  -k en-us \
  -display sdl \
  -vga virtio \
  -usbdevice tablet \
  -drive file=disk1.qcow2,if=virtio \
  -monitor stdio \
  -netdev user,ipv6=off,id=net0 \
  -device rtl8139,netdev=net0
  ```

  In the terminal of the guest, use the command `ip a` to see the IP Address of the guest, and the host. The host will be the gateway for the guest.

  ![](./imgs/Screenshot%202024-08-12%20at%2011.23.47 AM.png)

  Here, we can find the guests IP Address, we use `ip r` to see the routing information, and you will be able to see that the default route goes through the host.

  ![](./imgs/Screenshot%202024-08-12%20at%2011.25.49 AM.png)

  If we run `resolvectl`, we an see our DNS Server, and we can see that the guest can resolve the DNS and, access the internet, with a command like `ping linkedin.com`. The guest can reach the internet.

  ![](./imgs/Screenshot%202024-08-12%20at%2011.28.51 AM.png)

  This configuration is what we get if we don't provide any network configuration to the guest. The host is providing the guest with a private NAT network, and handles routing to it's network and beyond.

  If we start another copy of our guest, or create a different guest without providing any network configuration, that guest will also be on it's own private NAT network, and will be able to access the internet, but not the other guests.

  And, that means without additional configuration, our 2 guests won't be able to communicate with each other, and system's on the host network won't be able to access our guests either. Sometimes, this is what we want, and sometimes not.

  `User-mode networking` is convenient, but it can be a bit slower, than other networking strategies.

## Network Hardware and Settings

- If we want to customize the network connection in user mode, we have a few ways of doing that with a combination of the `-netdev` and `-device` options and the `-nic` option. There's also an option called just `-net`, but that's considered outdated, so we won't cover it here.

- `-netdev` sets up the host network backend, what we might think of as the imaginary router and network that the guest plugs into. It defines what networking mode we're using for a particular network connection and how that network operates. So we can set whether the backend supports IPV6 or IPV4, what the DNS address for DHCP clients should be, what the DHCP range is and so on.

- An important part of defining a connection with `-netdev` is the `id`, which is a name we assign so we can connect a device to the backend. When we use `-netdev`, we pair it with a device using the `-device` option. There we set what kind of emulated adapter we'll be using, the network card that plugs into our host backend. E1000 is common and so is RTL 8139.

- In the device option, we match the ID to the backend with a `-netdev` parameter, kind of like plugging in that adapter to a given backend. And we provide other options that pertain to a network interface, like the hardware model, a Mac address, and so on.

- Let's set up a slightly customized network adapter here. We'll use `-netdev` and tell it to use user mode networking. We'll turn IPV6 off and we'll define the ID as net zero. Then I'll add a device, an RTL 8139 and we'll associate that device with a `-netdev` called `net zero`.

  ```shell
  > qemu-system-x86_64 \
  -enable-kvm \
  -cpu host \
  -smp 4 \
  -m 8G \
  -k en-us \
  -display sdl \
  -vga virtio \
  -usbdevice tablet \
  -drive file=disk1.qcow2,if=virtio \
  -monitor stdio \
  -netdev user,ipv6=off,id=net0 \ # netdev option used here
  -device rtl8139,netdev=net0 # device option used here
  ```

- We'll open the terminal. We'll run `lspci` and here we can see that my ethernet controller now presents as a real tech semiconductor RTL 8139. As with other emulated devices, the default ethernet card is pretty standard and well supported, but as we've seen, we do have other options, like this Real Tech RTL 8139 gigabit adapter and Intel E1000 gigabit adapter.

![](./imgs/Screenshot%202024-08-12%20at%2012.19.24 PM.png)

- Some other specific models, and when we're using KVM paravirtualization, we also have `virtIO` net as an option, which provides a paravirtualized interface that can be faster than emulated ones.

- But not all operating systems have paravirtualization drivers, so you may need to do some research to find what adapters your guest OS supports and provide the right adaptor model or install drivers in the guest.

- We can find which models of adapter QEMU provides with `qemu-system-x86_64 -device help` and then we can scroll to the network devices section.

- The `-nic` option, short for `network interface card` combines both `-netdev` and `-device` into one option, removing the need to assign matching IDs to a `-netdev` and `-device`. This is a newer option and it lets us save a little bit of typing when setting up networking.

- Let's also use paravirtualization here to take advantage of the feature provided by KVM, rather than emulating a specific model of card. As we might expect, we can express both the `-netdev` and `-device` values with `-nic`, though in a slightly different format.

- Notice, for example that we need to say model equals something instead of just putting the model, like in the device option. Like with the device option we saw before we can find a list of the available models with `qemu-system x86_64 -nic model=help`.

- Notice how that's a bit of a parallel construction with how the actual command works. `-nic` versus `-netdev`, `-device` are just two ways to do functionally the same thing. But as I mentioned, the `-nic` is sometimes a bit more concise and the `-device` and `-netdev` options allow us to be more specific in some ways.

- There's a lot of parameters for these options and I'm not going to go through them all because that's what the documentation is for.

  ```shell
  > qemu-system-x86_64 \
  -enable-kvm \
  -cpu host \
  -smp 4 \
  -m 8G \
  -k en-us \
  -display sdl \
  -vga virtio \
  -usbdevice tablet \
  -drive file=disk1.qcow2,if=virtio \
  -monitor stdio \
  -nic user,model=virtio-net-pci # nic option used here
  ```

- We'll shut down my guest and We'll copy my modified command. We'll start the guest back up and in the terminal with `lspci`, there's our slightly different adapter.

    ![](./imgs/Screenshot%202024-08-12%20at%2012.24.31 PM.png)

- Next, Using either `-netdev` and `-device` or `-nic`, we can add one or more network interfaces to our guest by repeating the directives with different values, though in user mode networking those adapters are still within the protected network shared with the host.