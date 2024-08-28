# Manual Network Configuration Tools

## Configure a Static IP Address using `ifconfig`

- `net-tools` is a package that contains a collection of tools for configuring and monitoring network interfaces on a Linux system. One of the tools included in the `net-tools` package is `ifconfig`, which is used to configure network interfaces.

- The `ifconfig` command is used to configure network interfaces on a Linux system. It allows you to assign an IP address, subnet mask, and other network configuration parameters to a network interface. In this lesson, we will learn how to configure a static IP address using the `ifconfig` command.

- **Note**: The `ifconfig` command has been deprecated in favor of the `ip` command. However, it is still widely used and supported on most Linux distributions.

- Example:

    ```bash
    > ifconfig
    # This command by itself only shows the network interfaces that are currently up and running.

    > ifconfig -a
    # This command by itself shows all the network interfaces that are currently up and running, and also the inactive ones.

    > sudo ifconfig eth0
    # This command shows the configuration of the eth0 network interface.

    > ifconfig eth0 up
    # This command brings the eth0 network interface up.

    >ifconfig [interface_name] [ip_address_to_be_assigned]
    # This command assigns an IP address to a network interface.

    > route add default gw [gateway_ip_address]
    # This command adds a default gateway to the routing table. The default gateway is where the system will send all traffic, not meant for local network.
    ```

    ![](./imgs/Screenshot%202024-08-28%20at%204.41.34 PM.png)

## Explore interface information using `ifconfig`

- To view details about an interface, I can type ifconfig by itself, which will show all the active interfaces. In this case, my ethernet interface and the loopback interface. Or I can specify one interface by name, like this with `ifconfig eth0`, and that focuses in on just that ethernet interface.

- This is useful if you have many interfaces and want to see just one and it's also useful if you're using the output of this command in a script of some kind.

- The first piece of information we see is the link encapsulation. That is, what kind of interface we're dealing with. This is an ethernet interface, and for other types like the loopback, we'd see appropriate information listed there. After that, we see hwaddr, or hardware address, which is the MAC, or media access control address, of the network interface.

- The interface's IP address is associated with the MAC address and represents this interface. But, at a lower level, it's the MAC address that's really representing this interface on the network. We'll often need to use the MAC address of an interface to allow certain clients to join a network or to reserve an address for them if we're using DHCP reservations.

## Configure a Static IP Address using `ip`

- The `ip` command is a powerful tool for configuring network interfaces on a Linux system. It is part of the `iproute2` package, which is the modern replacement for the `net-tools` package that contains the `ifconfig` command.

- Example:

  ```shell
  > ip addr show
  # This command shows the IP addresses assigned to all network interfaces on the system.
  > ip addr add [ip_address]/[subnet_mask] dev [interface_name]
  # This command assigns an IP address to a network interface.
  > ip link set up [interface_name] 
  # This command brings the network interface up.
  > ip route add [ip_address] dev [interface_name]
  # This command adds a route to the routing table.
  > ip route add default via [gateway_ip_address]
  # This command adds a default gateway to the routing table.
  > ip a
  # This command is a shortcut for ip addr show.
  > ip l
  # This command is a shortcut for ip link show.
  > ip addr delete [ip_address]/[subnet_mask] dev [interface_name]
  # This command deletes an IP address from a network interface.
  > ip link set down [interface_name]
  # This command brings the network interface down.
  > ip link set up [interface_name]
  # This command brings the network interface up.
  ```

  ![](./imgs/Screenshot%202024-08-28%20at%206.09.38 PM.png)

- **Note**: `lower up` means the interface is up, but isn't available for communication electronically.

## Provide DNS Settings

- When we provide address information for a network interface, the interface can communicate using IP networking with any other peer on the local network or on the Internet that's using IP networking.

- And that's how we'll communicate with pretty much anything we need to exchange information with on a local network or the Internet.

- But right now, there's still a lot of normal tasks we can't do. Things like browsing the web or using apps that send data to servers based on host names on the network or on the Internet, because we don't have DNS running.

- Let's we want to ping example.com but we don't have DNS running. We can't do that because we don't have a way to translate the host name example.com into an IP address.

- We can resolve this issue by adding the IP Address to example.com in `resolv.conf` file.

- The `resolv.conf` file is used to configure the DNS resolver on a Linux system. It contains information about the DNS servers that the system should use to resolve domain names to IP addresses.

## Configure an Interface using DHCP

- Assigning a manual address to an interface can be done with either the `ifconfig` or `ip` tools, as we've just seen. And doing this is useful on servers or in situations where we know that a system should have a specific IP address, but that's not always the case.

- In fact, most home networks and most corporate networks run DHCP, dynamic host configuration protocol, a service which listens for computers and other network devices and provides them with an IP address that they can use for a period of time.

- So most laptops, desktops, and even mobile devices when using an IP network use DHCP to get their addresses instead of the addresses being manually set.

- And it's even the case that many servers get their addresses through DHCP as well using DHCP reservations. Where the DHCP server knows the Mac address of a particular interface and reserves an IP address for it.

- Assigning an IP address to an interface manually is fairly straightforward as we've seen, but the DHCP process is a little bit more involved.

- So we'll use software to handle DHCP and the interface configuration process is for us. As with manual address configuration tools, there are many tools to choose from. And two of the most common are `dhclient` and `dhcpcd`.

  - `dhclient` is widespread. Though it's starting to be replaced in new releases of distros with `dhcpcd` both work in the same basic way. We either write `dhclient` or `dhcpcd` and the software advertises its presence on the network, a DHCP server responds with an address and information like a default route and usually a DNS server. And then those settings are applied to the system, using the appropriate tools `ifconfig` or `ip` and `resolve.conf`.

- Example:

  **`dhclient`**
  
  - ```bash
    > dhclient -v [interface_name] # -v here means verbose, this will show more on as the execution takes place
    # This command configures a network interface using DHCP.
    ```

  - This command above using the eth0 interface to send a DHCP request out to the local networks broadcast address. A DHCP service running on that network, in some cases on a router piks up this request, and looks at it's table of IP Addresses, and provides an unused one to the system.

  - Once the IP is assigned, the software takes care of setting up the interface with the appropriate address and other network settings, including the default route provided by the DHCP server.

  - If we want to give up the address that we were just assigned this can be done so using the command:

    ```bash
    > dhclient -r
    # This command releases the IP address assigned to the network interface.
    ```
  
  **`dhcpcd`**

  - ```bash
    > systemctl status dhcpcd
    # This command shows the status of the dhcpcd service.
    > sudo dhcpcd -k
    # This command releases that address that we have back to the DHCP Sever, and remove the address for my interface.
    > dhcpcd [interface_name]
    # This command configures a network interface using DHCP.
    ```

## Connect to Wifi with `iwconfig` and `wpa_supplicant`

- To configure a Wi-Fi connection manually, without the help of network configuration managers, we need to use tools that can set up the radio connection between a wireless adapter and a base station.

- And then we'll apply network settings, like an IP address and so on to the connection. We can think of setting up the radio part of the connection, like plugging in an ethernet cable.

- When we use an ethernet connection, we already take care of that part. The physical part of the connection is made when we click the ethernet cable into the network ports and software, like if config or ip, don't have to worry about it.

- In a similar way, before we can use a wireless connection, we need to ensure that the wireless interface is connected to the wireless access point and that they can communicate using Wi-Fi.

- Only once this connection is established can we then go on to set up a network link using that connection. As with the other manual configuration tools we've seen so far, there's more than one set of tools to configure Wi-Fi connections.

- The `wireless-tools` package contains a collection of tools for configuring and monitoring wireless network interfaces on a Linux system. One of the tools included in the `wireless-tools` package is `iwconfig`, which is used to configure wireless network interfaces, and the other is `iwlist`, which is used to list information about wireless networks in range.

- But, `iwconfig` itself can't configure WPA or WPA2 connection. It can only configure a WEP connection, which has an encryption solution that we shouldn't be using anymore.

- So instead, to connect to a network using more modern security, we'll use a different tool, called `wpa_supplicant` to set up the wireless connection. And then we can use `iwconfig` to work with the wireless network interface. There's another more modern tool called `iwd` as well. That's more aware of modern wireless network standards. Iwd can be installed on most distros, but it's not as widely used yet.

- Example:
  
  ```bash
  > sudo iwlist scan
  # This command scans for wireless networks in range.
  > wpa_passphrase [wifi_name] > /mynetwork.conf
  # This command generates a WPA PSK from a passphrase and saves it to a file.
  ```

  When we run the command, I'll be prompted for the secret password, which in this case is superSecret. That generates a basic configuration file that we can use with WPA supplicant to connect to the network. For me, this configuration file works as is, but if you're troubleshooting or if something about your network that requires it, you can specify other options here.
  
  Use the command `nano mynetwork.conf` to open the file and add the `ssid` and `psk` to the file. The

  `mynetwork.conf` file will look like this:

  ```vim
  network={
    ssid = "wifi_name"
    key_mgmt = WPA-PSK #
    #psk = "superSecret"
    psk = 1a2b3c4d5e6f7g8h9i0j
  }
  ```

  Now, we will use the `sudo wpa_supplicant -i [interface_name] -c /mynetwork.conf` to connect to the network. The `-i` flag is used to specify the interface name and the `-c` flag is used to specify the configuration file.