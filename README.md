# OpenWrt_NordVPN_WireGuard
A script collection to connect a router with NordVPN via WireGuard

### Synopsis

The purpose of this project is to enable a router liberated by DD-WRT to connect to the service [NordVPN](https://nordvpn.com).
All the clients in the network attached to the router will be able to use NordVPN via one login.
The author of this software is in no way associated with the hosting company NordVPN.
Furthermore, the author of this software does not take on any responsibilities for damages resulting from this software or information.

After the great success of [DD-WRT_NordVPN](https://tobsetobse.github.io/DD-WRT_NordVPN) I became hungry for more.
I thought it should be possible to run the more lightweight WireGuard protocol on a strong router as well and max out bandwidth this way.
Turned out I was right.

### Prerequisites

- a router with a preferably strong CPU where you can run OpenWrt on. Check OpenWrt's [Table of Hardware](https://openwrt.org/toh) **before** buying the router.
- your NordVPN WireGuard private key (that's the tricky part in this installation)

### Obtaining your NordVPN WireGuard key

Unfortunately, NordVPN does not have a web interface to get this private key easily. So you will have to sniff them out of an existing connection. We need to install the NordVPN application, connect and read the private key. There are several ways to accomplish this.

#### Plain Windows way

I haven't tested this yet but you might give it a try. [2-click](https://gist.github.com/2-click/d3267354648bd6175db78ef171472e1d) seems to have found a way to read the private key with PowerShell.

#### Plain Linux way

Pick a Linux distribution for this task (e.g. [Linux Mint](https://www.linuxmint.com/), [Ubuntu](https://ubuntu.com/download) or whatever suits you). Download the ISO file. Use a tool like [Rufus](https://rufus.ie/) to get this ISO on a bootable USB stick, boot from it and select to "try only" (or, if you would like to switch to Linux, really install it).

#### Windows + Linux way

Pick a Linux distribution for this task (e.g. [Linux Mint](https://www.linuxmint.com/), [Ubuntu](https://ubuntu.com/download) or whatever suits you). Download the ISO file. Pick a virtual environment like Oracle's [VirtualBox](https://www.virtualbox.org/) or Broadcom's [VMware](https://www.vmware.com/) and install it. Get the downloaded ISO running in this virtual environment. 

If Linux is involved there are two ways to get your private key:

1. With Access token: Login in a browser to your [Nord Acount](https://my.nordaccount.com/). On the [Products and Services page](https://my.nordaccount.com/dashboard/nordvpn/) there is a [link to get your Access token](https://my.nordaccount.com/dashboard/nordvpn/access-tokens/authorize/). Then open a terminal and execute this (replace <ACCESS_TOKEN> with your Access token):

`curl -s -u token:<ACCESS_TOKEN> https://api.nordvpn.com/v1/users/services/credentials | jq -r .nordlynx_private_key`

2. Without Access token: Install the NordVPN client, connect and read your private key:

```
sudo apt update
sudo apt upgrade
sudo apt install nordvpn wireguard
sudo apt -f install
sudo groupadd nordvpn
sudo usermod -aG nordvpn $USER
sudo nordvpn set technology nordlynx
sudo nordvpn connect
sudo wg show nordlynx private-key
```
Once you have your key, copy it and save it somewhere. We will need it later.

Now install OpenWrt on your router.

### Get the scripts on your router

If you are on Windows you will need a tool like [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest) to connect to your router via SSH. If you are on Linux, just open a terminal and type:

`ssh root@192.168.1.1`

... or whatever is the IP address of your OpenWrt router and login with the password you have chosen during the OpenWrt installation. Once you're logged in, do that:

```
cd /root
mkdir NordVPN
cd NordVPN
wget -O update https://raw.githubusercontent.com/TobseTobse/\
OpenWrt_NordVPN_WireGuard/main/update
chmod +x update
./update
```

This should download all the scripts in this project directly from GitHub. Then install Wireguard with this:

```./install_wireguard```

You will be asked for your NordVPN private key. A file called config/my_config will be created and your private key will be saved there. In this new configuration file are also some other things you could configure at a later point of time if you wish so. Just do not edit maste_config ever! Always edit my_config. To edit my_config you could either use vi (if you know what you're doing) on your router, or simply install an SCP client on your computer and connect as root to your router. I like [WinSCP](https://winscp.net) for such tasks, it runs even fine in [Wine](https://www.winehq.org).

Then use your browser to login to LuCI, your OpenWrt admin backend. Usually this is at https://192.168.1.1 and go to [System > Scheduled Tasks](http://192.168.1.1/cgi-bin/luci/admin/system/crontab). Copy & paste the following lines there (and make sure to end this with an empty line):

```
@reboot      /root/NordVPN/connect
*/5  * * * * /root/NordVPN/auto_set_mtu wg0 $(cat /tmp/currentserver)
*/15 * * * * /root/NordVPN/speedtest reconnect
20   0 * * 2 /root/NordVPN/update

```

The first line regularly finds the best MTU for your connection, the second line checks the speed every 15 minutes and connects to another server if the speed is too low and the third line downloads updates for these scripts once per week from GitHub.

### Usage

Now you're good to go! Give it a try:

```connect```

This script searches for a VPN server itself, connects to it and establishes a killswitch. Should you ever want to disconnect from the VPN again, just do this:

```disconnect```

Have fun!
