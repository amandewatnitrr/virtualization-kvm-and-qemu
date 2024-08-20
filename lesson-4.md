# VM Management Tools

## Exploring `libvert`

- Till now we have been doing everything with our guest configuration, and management manually at the command line. And, this is pretty tedious work.

- If we are working with guests a lot, it can be much nicer to use tools that can handle a lot of detail work for us.

- `libvert` provides an API for management tools like VERS and VERT Manager can use to manage QEMU and to connect with hypervisors.
  
  Use the following command to install the same:

  ```shell
  > sudo apt install libvirt-daemon-system libvirt-clients
  ```

  For changes to take place, on to the machine, you need to logout and back in to the machine.

- `libvirt` stores configurations for guest machines and other entities like networks, storage and so on as XML Files.

- It uses name domains for guest machine definitions and resources. These configuration files are read and managed by various front-end clients, and we can edit the files manually if we're so inclined.

- They are stored in `/etc/libvert/qemu`.

- `libvirt` api handles all the communication with QEMU, and it even sets up it's own network bridge and routing so that guests can participate in a shared NAT Network with the host.

- Out of the box, so to speak , network uses `192.168.122.x` space by default, adn the host is available at `192.168.122.1`. This bridge automatically setup for us is a huge time saver.

- `libvirt` also manages disk images, and snapshot and so on in a centralized way. It has a series of configuration files including one at `/etc/libvert/qemu.conf`, where we can set the defaults for any new guest, saving us from having to specify every little thing about our guests like we do with QEMU system commands.

- If we don't make any changes to this file, `libvert` uses same defaults for our new guests.

## Exploring Virtual Machine Manager

- Many Graphical VM Management Clients can use `libvirt` to manage guests.
- One of this is `virt-manager` or Virtual Machine Manager. We can install it using the command:

  ```shell
  > sudo apt install virt-manager
  ```

  We can open the application from the command line using the command `virt-manager`.

  In many cases, we will use this software to manage guests on our localhost, but we can also add connection to remote hosts, and manage guests on other hypervisors that way.

  ![](./imgs/virt_manager.png)

  We can create a New Guest on this host by clicking the button, with a shiny new monitor. In the interface, where we create a new machine, we have a variety options that look familiar for working with QEMU commands on the terminal. We can choose what to use to install from or boot from, if we alreadt have a pre-made image. If we boot from a pre-made image, like our existing QEMU guest, the image won't be moved automatically to the libvirt storage location.

  We either need to maintain that image where it is or create a volumne and import it manually using tools like `virsh`.

## Exploring `virsh`

- `virsh` allows us ti manage libvirt guests and other resources from the command line.
- It's installed as a part of the `libvirt-clients` package. We can use `virsh` either as a command with arguments following it, or as an interactive shell.
- We can use virsh in the interactive terminal using the command `virsh`.

- Using the `list` command, we can see the running guests. `list --all` we can see all the guests.
- We can use the `edit <guest_name/>` and open up the XML Configuration file associated with it.
- We can find the other commands that `virsh` has using the `help` command.
- We can exit out of this `virsh` interactive shell using the `exit` command.

- We can start an existing guest domain from the command line with the command `virsh start <name_of_the_domain/>`, and we can connect to it's screen with the `virt-viewer` command, and we will see a list of guest to select from. And, there you will have the system. It's the same old VM but it's being managed and accessed with a different tool set.
- We can power the system off safely using the command `virsh shutdown <name-of-the-domain/>` command.