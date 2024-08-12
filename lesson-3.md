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

## Bridged Network

- We can use a different networking mode to build more complex network setups and that involves network bridges.

- A network bridge allows two or more devices or networks to communicate with each other. Normally, we find bridges inside of routers or switches, allowing those devices to send traffic between our computers and the broader network. But on the Linux system, we can create a virtual bridge that our QEMU guests can connect to in order to communicate.

  When we're using a bridge set up, our guest network interfaces are represented on the host as TAPs, a term for a virtual network adapter that we can plug into our bridge.
  
  Combining a bridge and two or more TAPs, we effectively build a virtual ethernet network we can configure however we like. With QEMU, we create a bridge on the host and then configure our guests to connect to it. We can use this model to create two similar, but functionally different, kinds of virtual network .
  
  One kind is a Private Bridged Network, where we connect guests to our bridge and they operate in a private network environment. We may choose to provide services, like DHCP, within this network, with some additional setup and we might even choose to configure our host to act as a router, to provide the guests with network access.
  
  The other kind of network is a Public Bridged Network, where we add one of the host's ethernet interfaces to the bridge along with the guest's, effectively extending the host's network inside our private network, so that the guests function as regular clients of that network.
  
  If we're feeling very industrious, we can mix and match these modes, as well. For example, bridging one guest to the host's network and adding another interface to it, so it can be configured to provide routing and other services to a second private bridged network of guests. We're going to keep things fairly straightforward here, though.
  
  Working with bridge networking requires a bridge and that's something we have to set up on the host itself. The bridge device is entirely separate from QEMU. To create a bridge, We'll use the ip utility here on Ubuntu, and We'll write, `sudo ip link add br0 type bridge`. This creates a bridge called `br0`.
  
  Then, We'll bring the bridge up, with `sudo ip link set br0 up`. The name can be anything you like. br0 is a bit of a convention and it's short, so I'll use it here. We can also create more than one bridge should our architecture require that. This bridge, created in this way, won't persist through a reboot.
  
  We can use our distribution-specific network configuration tools, like Network Manager, netplan, ifupdown, and so on, to make this bridge permanent. Or we can put these commands in a script. 
  
  Once our bridge is created, we'll need to attach devices to it. There's two paths to doing this. One is to manually create and manage individual TAP interfaces on the host and pass them to each guest by name. This shows how we might do that for two guests that will share a private network. 
  
  ```shell
  # create tap interfaces tap0 and tap1
  ip tuntap add tap0 mode tap && ip tuntap add tap1 mode tap

  # bring up tap interfaces tap0 and tap1
  ip link set tap0 up promisc on && ip link set tap1 up promisc on

  # connect tap interfaces tap0 and tap1 to the bridge br0
  ip link set tap0 master br0 && ip link set tap1 master br0

  # start two guests using tap0 and tap1 (remember to use two different disk images)
  qemu-system-x86_64 ... -netdev tap,id=t0,ifname=tap0,script=no,downscript=no -device e1000,netdev=t0,id=nic0

  qemu-system-x86_64 ... -netdev tap,id=t1,ifname=tap1,script=no,downscript=no -device e1000,netdev=t1,id=nic1
  ```

  But, as you can see, that's a lot of work. We could put this into a script or into our host configuration so it all gets set up automatically. 
  
  But whenever we want to add another guest, we'd need to go through the TAP creation process for that new guest. So helpfully, there's a tool called QEMU Bridge Helper that does the work for us. It creates and manages the TAP interfaces for guests that use them and runs the `qemu-ifup` and `qemu-ifdown` scripts in the host's etc directory to set the TAP interfaces up and down when the guest boots and is shut off. This lets us add guests easily.
  
  Using the helper takes a little bit of setup, though. First, we need to change the permission mode of the helper tool itself to add the set UID bit to it. Alright, `sudo chmod u+s /usr/lib/qemu/qemu-bridge-helper`. This step allows our user to invoke the helper without requiring super user privileges to run the QEMU target itself.
  
  Running QEMU as the super user is generally discouraged, but in order to manipulate the network interfaces on the host, we need super user privileges and that's where this helper tool comes in. It's able to act as the super user to make those changes to create and manage the TAP interfaces in a way that doesn't require individual users to be empowered with root access. The helper needs a little bit of information, though, about which bridges it's allowed to work with. 
  
  We have just one, br0, and to tell the software that users can use that bridge, We'll create a configuration file. We'll write:

  ```shell
  > vi /etc/qemu/bridge.conf
  > allow br0
  > # Now save the file
  > ls -l /etc/qemu
  ```

  Move into insertion mode and write, `allow br0` and save the file, with escape, `:wq`. We created this file as root, so it's owned by root, but other users will be able to read it. This bridge helper configuration will persist across reboots, but the bridge itself won't, unless we tell the system to set it up in the host's network configuration.
  
  Now, we're ready to use our bridge and the helper to start QEMU guests that participate in the same network.


## Creating a Private Network

- To create a private network, as with all of the other bridge related network architectures, we can use qemu's netdev and device options or the nic option.

  Let's start at using the `-netdev` and device options for this guest. For the `-netdev` backend, We'll tell qemu to use a bridge. Then We'll say which bridge to use. In my case, that's `br0`. Then we'll provide an id, a name to call that backend, in this case net1. In the device section, We'll use para virtualization and say that this device should attach to the netdev backend with the id net1. Remember that this name can be more descriptive if you need it to be.

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
    -netdev bridge,br=br0,id=net1 \
    -device virtio-net,netdev=net1
    ```

  But there's one important thing we need to be aware of. Our second guest is going to be joining the same network as the first guest, and qemu assigns the same MAC address to each guest's first network interface, so our two guests will both have the same MAC address and well, that's a problem if we want them to communicate.

  For my second guest, We'll specify a different MAC address than the default. In this case, just one digit higher, ending with 57 instead of 56. We could use `-netdev` and `-device` for the second guest too but let's use the `-nic` option instead. We'll write a command very similar to my first guests but notice that we're using the `-nic` option instead of `-netdev` and `-device`.

  Each additional guest we add will need to have a unique MAC. We can increment the default one for each guest or use other MAC addresses. In the past, We've used this generator website, `macaddress.io` but you could come up with your own script or system of generating valid unicast locally administered addresses.

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
  -drive file=disk2.qcow2,if=virtio \
  -monitor stdio \
  -nic bridge,br=br0,mac=52:54:00:12:34:57,model=virtio-net-pci
  ```
  
  Okay, our two guests are up and running, and are virtually plugged into the same bridge. We can see the bridge and our tap devices on the host with a command `ip link`. Here's the bridge, tap0 and tap1.
  
    ![](./imgs/Screenshot%202024-08-12%20at%202.42.45 PM.png)

  The bridge helper set these up for us but we're not quite done yet. Within this private bridge architecture we just set up, there's only two systems. There's nothing providing DHCP information to them and they don't have useful IP addresses. 
  
  So in order for the guests to be able to communicate with each other, we'll need to provide them addresses. We'll move into each guest's settings here and assign addresses within a private range.
  
  For our first guest, We'll change IPv4 to manual and We'll give it the address 10.10.10.2.

  ![](./imgs/Screenshot%202024-08-12%20at%202.46.07 PM.png)
  
  Same way we'll go into its settings of second guest and give it the manual address 10.10.10.3. We'll toggle that connection off and on and see it's connected. 
  
  Back here in my first guest, We'll open up the terminal and We'll try to communicate with that second guest, using the command:

  ```shell
  > ping 10.10.10.3
  ```

  ![](./imgs/Screenshot%202024-08-12%20at%202.48.45 PM.png)
  
  And that works. With this configuration I'll only be able to communicate between these guests but not with the host or any network outside of this one.
  
  I could add other guests to this private network the same way, the bridge helper will connect them to the bridge, but I'll need to assign them IP addresses in order for them to communicate with their peers.

## Creating a Host-Only Network

- Another useful networks topology for QEMU guests is called a host-only network, which is like a private network with no outside connectivity except that the host participates in the private network. This architecture allows guests access to resources on the host, but not to reach out beyond the host to its network or the internet.

- We can convert our private bridge network into a host only network by signing the bridge and address on the private network.

  Here on the host We'll run the command:

  ```shell
  # This must be done on the host
  > sudo ip address add 10.10.10.1/24 dev br0
  ```
  
  To add the address 10.10.10.1 to the bridge called `br0`. And here inside my guest I can communicate with the host now. With this architecture, we can take another step and set up DHCP for the private network so we don't have to worry about manual addresses at all.
  
  With this configuration, any guest we add to the private network will be assigned an IP address automatically by the host. We've already assigned the bridge an IP address and the network range we want to use. So the next step is to tell the DNS mask software here on my Ubuntu host to provide DHCP service only on my bridge interface. 
  
  That's really important to get right because having a rogue DHCP server on your regular network will cause a bunch of problems. 
  
  We'll use DNS mask here on my Ubuntu host to provide a DHCP server only on the bridge interface and give it the IP range `10.10.10.2` to `10.10.10.100`. 
  
  ```shell
  # This must be done on the host
  > sudo dnsmasq --interface=br0 --bind-interfaces --dhcp-range=10.10.10.2, 10.10.10.100
  ```

  That's more than enough addresses for a network I'd need to build. 
  
  And now I'll switch back to my guests and clear out their manual address information. I'll set them back to automatic. After a moment, We can see that the guest has been given an address in my DHCP range. And We can still communicate with the host at 10.10.10.1 as well. And from the host, I can access the guests by their private IP addresses too. But this is still a host only network and doesn't allow access beyond the host. We just don't have to worry about manually setting IPs with DHCP turned on.

