# Linux Networking Configuration

## Network Configuration Basics

### Configuring a Network Adapter

- In order to be able to participate in an IP network, a network interface needs an IP address.

- This address needs to be one that's valid for the network that it's participating in, an address that's within the network range that other peers on the network expect.

- This address can be provided manually as what we call a static IP address, one that doesn't change over time, or it can be provided by a DHCP service where a system on the network listens for new peers and assigns them and address on the network.

- A server or a system that often needs to be found reliably by other peers using its IP address generally should have a static IP address.

- And client systems like laptops or mobile devices or computers that aren't providing services others rely on. Generally should be configured to have a dynamic or DHCP address.

- On a local private network like we have at home or in many business settings, the range of IP addresses we use will most often be in one of these three ranges reserved for the purpose. These are called private ranges, because by design, they don't exist on the internet. We can use the IP ranges of systems that exist on the internet on our own networks, but doing so is discouraged because it causes problems of various kinds.

- Most notably, if you use the same IP range on your local network that a cloud provider does for their servers, you wouldn't be able to access those servers on the internet and your traffic would go to your local devices instead.

- Each peer on a private network needs to have its own distinct IP address within the range, so it can be found and so data can be sent to it. With just an IP address, a system can use its network interface to communicate with peers on its local network.

- But in order to communicate with hosts on other networks, a system needs a route or a rule that tells it where to send packets that are intended for computers that aren't local network peers. Any system that communicates with the internet has at least one route, and it's called the default route. The host that packets intended for other networks are sent to in the default route is called the default gateway, and it's usually the network's router, which is usually found at the first address in a private network's address range.

- So in a home network or other private network, the default gateway will often be 10.0.1.1 for a network using the 10.0.1.0 network space, 192.168.1.1 for networks using 192.168.1.0 network space, and so on. This can be changed by a network administrator, but these addresses are good starting points if you're trying to find the gateway for your network.

- Right now, we'll focus on setting up a client to use an existing gateway on an existing network though. The default gateway acting as a router is configured to pass data along from one network to another translating or transforming packets of data so that they can travel on to remote hosts, and translating or transforming them again when they come back so they can travel to peers on the local network.

- Once the system has an IP address and a default route, it can communicate with internet hosts around the world. There's just one problem though. It can only communicate using numeric IP addresses, not the host names we're familiar with like linkedin.com, google.com, microsoft.com, and so on.

- In order to allow our system to use these human friendly names, we need to provide it information about a DNS server or a DNS resolver, a service which takes a host name and returns an associated IP address for it. This isn't handled by the network interface. Instead, it's a separate configuration for the system.

### Interface Management Tools

- There are mainly 2 type of tools we need to know about when working with Network Configurations in Linux:

  - First type, tells networking components of the kernel what interfaces to define, what addresses to give an interface, and so on.

  - Second type, are tools that store configuration information and use the interface management tools to apply that configuration for us.

- On modern systems, the tools we will use are part of a package called `iproute` or `iproute2`.

  - Primarily, this package contains the command `ip`, which we can use to work with network links, addresses, the routing table, and so on.
  - We can use the software directly to manually configure an interface.
  - This package of software also includes the `ss` command that we can use to look at network sockets. 
  - `iproute` or `iproute2` replaces an older package called `net-tools` which contain tools that also told the kernel what to do with network interfaces.
  - `net-tools` gives us software such as `ifconfig`, which is used to configure interfaces. `netstat` which is used to find out about network sockets and so on.

- One set of tools are those for controlling Wi-Fi interfaces, tools like `wpa_supplicant`. Whereas an Ethernet connection is pretty much ready to go, at least at the physical connectivity level when we plug it in, in order to use a Wi-Fi connection, we need to tell the wireless interface how to connect to a Wi-Fi base station.

- So, first, we use tools to set up that connection, the radio connection between a wireless interface and a particular base station like plugging in a cable but wirelessly. And once that connection is made, the interface is ready for us to configure and address and route information just like an Ethernet connection.

- The other important tool that we need to know about is the DHCP client, the tool that gets a dynamic IP address from a network's DHCP server.

- DHCP makes it easy for systems to join a network without having to know which static IP address they should use in their configurations.

- Again, there are two primary tools we'll need to be aware of here: `dhclient`, which has been used for a long time on many distros, and `dhcpcd`, the DHCP client daemon, which provides many of the same features but is also widespread and is beginning to replace dhclient in newer releases of distros.

### Configuration Management Tools

- When a system restarts, changes that we made using network interface management tools are lost and they need to be recreated.

- For that reason, and to make it easier to work with existing and new interfaces, there are some network configuration management tools that we're likely to use to define network configurations.

- There are three of these tools and they're called ifupdown, NetworkManager, and systemd-networkd. Many older distributions used `ifupdown`, which is a set of scripts that read configuration files for network interfaces and then act on them. The primary scripts that an `ifupdown` system uses are called `ifup` and `ifdown`, short for interface up and interface down, and they read configuration files that reside in different locations on different distros, usually at `/etc/network/` interfaces on Debian and similar systems and in `/etc/sysconfig/network-scripts` on Red Hat and similar systems.

- These scripts can work with either the `net-tool` software or the IP route software and the scripts available on your distro should be whichever version is appropriate based on what tools your distro uses.

- `ifupdown` scripts can be run by the user or they can be hooked into the system's initialization manager so they run at boot time and can be controlled by the system's service manager, software like init or systemd, depending on what your distro uses.

- `ifupdown` scripts give us the opportunity to run various other commands and scripts alongside bringing a network interface up or down. `ifupdown` is generally being phased out in modern distros in favor of newer tools that provide updated functionality. 

- Another network configuration manager is called `NetworkManager`, and at the time I'm recording this, it's likely to be the most widespread network configuration manager in use on modern distros, especially distros running a desktop environment.

- `NetworkManager` offers both command line and graphical tools to manage a network connection. NetworkManager distinguishes between devices, which are network adapters like ethernet, WiFi, cellular modems, and so on, and connections, which are the collection of settings applied to a device. NetworkManager works with net-tools and IP route, and automatically configures routes based on the settings we provide it. We can also indicate whether we want NetworkManager to manage a particular device or to leave it alone for us to configure in other ways.

- `systemd-networkd` is a module for the systemd system management framework that handles network interface configuration. It's not as widely adopted as `NetworkManager`, but it's also quite a bit newer and is often found primarily on systems where there isn't a desktop environment available. `systemd-networkd` provides us the `networkctl` tool, which we can use to view interfaces and, like NetworkManager, we can tell it whether or not we want it to manage particular devices.

- In the Ubuntu world, there's another tool we should be aware of called `netplan`. We can think of this as a network configuration manager configuration manager, because it lets us define a network configuration using a YAML text file and then that configuration is converted into configuration information, either for `NetworkManager` or `systemd-networkd`. Netplan is more of a high-level configuration tool and it lets us build network configurations that can easily be used with deployment or orchestration tools.