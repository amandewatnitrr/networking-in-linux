# Hostname and Firewalls

## Set the System Hostname

- A system's host name is used to identify it in a human readable way, and it can be used by software and other systems to refer to a computer as well.

- A Linux System has many addresses to which it will respond aside from its IP address on a network. The address `127.0.0.1` resolves to the local system. And so does the name local host and some systems also use the address `127.0.1.1` to refer to the local system as well. We can also assign a name to the system.

- In legacy systems, the hostname was stored in the file `/etc/hostname`.

  But in modern systems, the hostname is stored in the file `/etc/hostname` and `/etc/hosts`. The `/etc/hostname` file contains the hostname of the system, and the `/etc/hosts` file contains the IP address and hostname of the system.

  Set hostname in `/etc/hostname` using command:

  ```bash
  echo "[Hostname]" > /etc/hostname
  ```

  `/etc/hosts`

  ```bash
  [IP Address] [Hostname] [Alias]
  ```

- This hosts file can be used to create local static mappings giving names to other IP addresses as well. This is useful if DNS isn't working on a system, or if you want to create local shortcut names for systems on your network, we can also use the nmcli tool to set a system's host name, using an `nmcli general hostname [newHostname]`   , and then the new host name.

- While this changes the host name, it doesn't update the hosts file. And so using this method will sometimes run across strange results. For that reason whenever possible, we should use the third method to change a system's host name.

  We can use `hostnamectl` to set the system's host name. This command will update the host name in the `/etc/hostname` file and the `/etc/hosts` file. We can use the command `hostnamectl set-hostname [newHostname]` to set the host name.

  Andm than when you start the system you can see the changes have taken effect.

## Exploring Firewall

- A firewall is software that determines what network traffic is allowed to come into and go out of a system.

- On Linux systems, the firewall software, or packet filtering software, as it's known, is called `iptables`, and on some more recent distros, iptables is being replaced with software called `nftables`.

- Firewalls operate using chains of rules, which evaluate each packet received by the firewall to decide whether the packet matches rules that allow it to pass through. If not, a packet is either dropped or rejected. These rules can be constructed to filter packets based on many different conditions.

- For example, which port they arrived on, what address they come from, what protocol they represent, and so on. We can also write rules that tell the firewall to send packets from one system to another system and this enables network address translation, a feature that nearly all home networks and many corporate networks rely on to allow client systems secure access to the internet.

- Writing iptables rules directly ourselves is possible and doing so is a good skill for network administrators to have. However, the syntax and construction of these rules can get pretty complicated, so in most cases, on client systems and basic servers, we'll use software that makes working with a firewall a little more user-friendly.

- There's two firewall management programs we'd like to cover here, called `firewalld` and `ufw`. `firewalld` is usually found on systems running Red Hat, CentOS, and Fedora, and UFW, which stands for `uncomplicated firewall`, is installed by default on Ubuntu systems and is available for Debian and other distros as well.

## Firewall Configuration with `ufw`

- To install ssh server on Ubuntu, we can use the command `sudo apt install openssh-server`.

- To allow ssh traffic through the firewall, we can use the command `sudo ufw allow ssh` or `sudo ufw allow 22/tcp`.

- You can validate the rule by using the command `sudo ufw status`.

- Using `less /etc/services` you can see the list of services and their corresponding port numbers.

- To delete the rule, you can use the command `sudo ufw delete allow ssh` or `sudo ufw delete allow 22/tcp`.

## Firewall Configuration with `firewalld`

- Use the command `sudo systemctl status sshd` to check the status of the sshd service.

- Activate the sshd service using the command `sudo systemctl start sshd` and `sudo systemctl enable sshd` command.

- You can check the status of the sshd service using the command `sudo systemctl status sshd`.

- `sudo firewall-cmd --list` command will list all the services allowed through the firewall.

- To allow ssh traffic through the firewall, you can use the command `sudo firewall-cmd --add-service=ssh --permanent` or `sudo firewall-cmd --add-port=22/tcp --permanent`.

- For changes to take effect, you need to reload the firewall using the command `sudo firewall-cmd --reload`.

- If we want to remove the rule, we can use the command `sudo firewall-cmd --remove-service=ssh --permanent` or `sudo firewall-cmd --remove-port=22/tcp --permanent`.

## Monitor Network Port Activity

- When we use IP networking, services communicate using sockets.

- A socket is the combination of an address and a port, and there can only be one socket per combination of port and address.

- So for example, if our SSH service is listening on 10.0.1.101 TCP port 22, nothing else on that system can use that socket, that specific combination of address port and protocol.

- Because sockets are so important to communication, it's useful to know how to see what sockets exist on a given system. The legacy net-tools package gives us the program `netstat` for this purpose. And the more modern IP route toolset gives us the program `ss`.

- Both programs work in a similar way and we'll often use whichever program is at our disposal to explore which sockets are actively connected, which are listening and what process or program is responsible for a given socket.

- Across both programs, the option `-t` shows active TCP sockets, `- u` shows active UDP sockets, `- l` shows listening sockets, ones that don't have an active connection and `-p` shows the program or process responsible for a socket.

- 