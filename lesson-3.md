# Network Configuration with `systemd-networkd`

## Exploring `systemd-networkd` and `networkctl`

- The systemd system manager controls many aspects of the system on many Linux distributions.

- The systemd-networkd module can control network interfaces, and it's becoming increasingly popular.

- One advantage of using systemd-networkd instead of another network configuration manager is that it can be used in just the same way that we create, modify, and interact with other systemd units.

- And on systems with limited resources or where network information doesn't vary as much as it might on a system where network manager is used, systemd-networkd gives us a lightweight method of configuring network information.

- A `systemd-networkd` is a primary method of control for some distributions, and it's available to be installed on many others. The systemd-networkd configuration files can be read from a variety of directories, and that can make working with them a little confusing.

- If systemd-networkd is the only software controlling network settings, configuration files are stored in the `etc/systemd/network` directory. If systemd-networkd is being used in conjunction with other software, like we saw with Netplan just a little bit ago, the configuration files are usually found in `run/systemd/network` instead. The files here take precedence over those in the `etc/systemd/network` directory. Regardless of which directory they're in, the files are named beginning with numbers, which are used to set the order in which configurations are read. 

- The software reads them from zero to 99. And if there's more than one configuration file, we'll commonly see names for those files, with numbers that are spaced 10 or 20 values apart, allowing space to insert different configurations, should we need to do so.

- These configuration files end with the extension .network. On most systems and especially most personal computers and basic servers, we'll only have one or two configuration files, usually one for ethernet and sometimes one for a second ethernet connection or for wifi.

- When we're working with systemd-networkd, we can use the network CTL tool to view information about network interfaces.

- `systemd-networkd` works in conjunction with another service called `systemd-resolved`, which is in charge of name resolution on most systems running systemd unless configured otherwise. As with many other network configuration approaches, this keeps DNS separate from addressing and routing, even though DNS information is provided in the `systemd-networkd` configuration files or via DHCP.

- Example:

  ```bash
  > sudo systemctl status systemd-networkd
  # This command will show the status of the systemd-networkd service.
  > sudo systemctl start systemd-networkd
  # This command will start the systemd-networkd service.
  > sudo systemctl stop systemd-networkd
  # This command will stop the systemd-networkd service.
  > sudo systemctl restart systemd-networkd
  # This command will restart the systemd-networkd service.
  > networkctl
  # This command will show information about network interfaces.
  > networkctl status
  # This command will show the status of network interfaces.
  > networkctl list
  # This command will list network interfaces.
  > networkctl show [interface_name]
  # This command will show information about a specific network interface.
  > networkctl reload
  # This command will reload network configurations.
  > networkctl reload [interface_name]
  # This command will reload a specific network interface.
  > networkctl reload-all
  # This command will reload all network configurations.
  > networkctl reload-all [interface_name]
  # This command will reload all network configurations for a specific network interface.
  > networkctl enable [interface_name]
  # This command will enable a specific network interface.
  > networkctl disable [interface_name]
  # This command will disable a specific network interface.
  > networkctl mask [interface_name]
  # This command will mask a specific network interface.
  ```

  Let's for sample creare a file called `/etc/systemd/network/10-static.network` with the following content:

  Being the only file in the directory, it will be the only one read by systemd-networkd.

  ```bash
  # This file will have 2 sections: [Match] and [Network].
  # The [Match] section will contain the MAC address of the network interface, and will tell which interface to apply the configuration to.
  # The [Network] section will contain what settings to apply.
  [Match]
  Name=enp0s3

  [Network]
  Address=10.0.1.149/24
  Gateway=10.0.1.1
  DNS=10.0.1.1
  ```

  Now, Save it, and use the command `sudo systemctl restart systemd-networkd` to apply the changes.

  Similarly, for a DHCP configuration, we can create a file called `/etc/systemd/network/10-dhcp.network` with the following content:

  ```bash
  [Match]
  Name=enp0s3

  [Network]
  DHCP=yes
  ```

  Now, Save it, and use the command `sudo systemctl restart systemd-networkd` to apply the changes.
