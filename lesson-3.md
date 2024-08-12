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

## Port Forwarding

- One option available to us with User Mode networking is to expose one or more ports on the guest system through network ports on the host and connect to the guest that way.

- For example, peer will forward port 22 on the guest for SSH to the hosts port 2222, where the host and peers on its network would be able to connect and access the guest.

- To do that, we'd use the `hostfwd` argument to the `-nic` option which we saw a moment ago. With the `-nic` option, we'll provide the user argument to tell qemu that we want to use User Mode networking but we'll also add the host forward argument with a few parameters defining how that forwarding works.

    ```shell
    -nic user,hostfwd=protocol:hostaddr:hostport-guestaddr:guestport
    ```

    ![](./imgs/Screenshot%202024-08-12%20at%201.23.24 PM.png)

- We'll connect to port 22 for SSH purposes and we can use host forward to open a port on the host and forward it to a port on the guest.

  In this example, we'll use Port 2222 on the host. If we don't specify a protocol, it defaults to tcp and if we don't specify an address, it defaults to any of the host or guests available addresses.

  Each host port needs to be dedicated to only one mapping and we can't use ports that are already in use.
  
  For example, port 22 and the host is likely already running its own SSH service and to use host port 1024 and below, we'll need super user privileges. We can also add other host forward arguments and parameters here to individually forward other ports.
  
  For example, if we ran a web server in addition to SSH on the guest, we could also forward port 8080 on the host to port 80 on the guest. We'll briefly install an SSH server and a web server in this guest, so we'll have something to connect to. We'll write `sudo apt update` and then We'll write `sudo apt install openssh-server apache2`. Those are installed and we won't configure these because they're just standing in for real services.
  
  Depending on our distro, we'll need to check our firewall too to ensure that the requisite ports 22 and 80 in this case, are open.

  Now We'll shut down this guest so we can make some changes to its command. Here's the option added onto our command, and once again, create a new tmux pane with control B, C, and then we'll SSH into my guest.

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
  -nic user,hostfwd=::2222-:22,hostfwd=::8080-80 # nic option used here
  ```
  
  We'll write ssh, my guest username at my host and specify the port 22.

  ```shell
  ssh user@localhost -oPort=2222
  ```
  
  We'll accept the key. We'll type in the password, and now We're connected to the guest. Cool, We'll disconnect and close this pane. I'll switch to my browser and open up my host on port 8080 and there's the web server running on the guest. It's common to start up a guest with only port 22 open because nearly any other network service can be tunneled over SSH and it can be used for administration and to send files back and forth as well.
  
  But depending on your requirements, you may need other specific ports opened up to allow connectivity to the guest using the host's address or you may need a different solution as we'll explore shortly.

## Removing Network Connectivity

- Sometimes it's useful to have a guest with no network access. We might use this setup for security work like malware analysis, where we don't want any network communication happening. To tell QEMU not to provide a network interface to a guest, we'll use the option -nic none.

```shell
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
  -nic none
```

We now check the IP Address, and see that we don't even have an ethernet adpater. We can also double check that with `lspci`.

![](./imgs/Screenshot%202024-08-12%20at%201.41.46 PM.png)

![](./imgs/Screenshot%202024-08-12%20at%201.42.13 PM.png)

This guest is all alone in a world of it's own, atleast as far as networking is concerned. We can however still attach a network usb adapter and, establish connectivity for the guest.