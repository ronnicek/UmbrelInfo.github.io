---
layout: default
title: Troubleshooting
nav_order: 10
---

# Troubleshooting
{: .no_toc }

Issues concerning the primary operations of your Umbrel node.

The aim of this troubleshooting guide is to help debug your system, verify all important configuration and diagnose problems that keep you from running the perfect node. Where possible, relevant parts of the guide will be linked.

If you see any discrepancies from the expected output, double check how you set up your node in that specific area.

General questions and further learnings regarding the technology used in Umbrel can be found in the separate [General FAQ](faq.md).

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

### Can I login using SSH?

Yes! Open a terminal on your computer and enter `ssh umbrel@umbrel.local` and enter the password `moneyprintergobrrr`.

### Is my Raspberry Pi compatible?

If you have a Raspberry Pi with at least 4GB of RAM, you can run Umbrel on it.

### My Umbrel node keeps crashing. What can I do to fix the issue?

If you're not using the official power supply, it's probably the power supply.
To detect undervoltage, connect to your node via SSH and run this command: `vcgencmd get_throttled`.
If it doesn't output throttled=0x0, then it's either the power supply or your SSD is using too much power (this can only be the case if you're not using the recommended hardware).

### My Umbrel node doesn't boot. What can I do?

Do you have connected anything to the GPIO pins?
If yes, try to unplug it and reboot the RPi by unplugging the power supply and then plugging it back in.

### I can't access the dashboard at umbrel.local. What can I do?

Check if your router detects your node.
If it doesn't, either you ethernet cable isn't plugged in correctly or the node doesn't boot.
If you think the ethernet cable isn't the issue, follow the answer of the previous question.

If it does detect the node, try to access it with the IP address directly.
If you can't access the dashboard via the IP address either,
try to disconnect the drive from the Raspberry Pi and plug it into the other USB port.
Then SSH into your node and run: `sudo systemctl start umbrel-external-storage`.
After you've run the command, wait for two minutes, then run `sudo systemctl status umbrel-external-storage`.
If the output of that command contains "Exiting the mount script without doing anything", the drive is connected wrongly.
If the output doesn't contain this text, run `sudo systemctl start umbrel-startup`.
You should now be able to access the dashboard.

### I want to connect to my node using ...... over my local network, but it doesn't work. How can I fix this?

If you want to connect to your Umbrel over the local network just replace your onion domain with umbrel.local for any of the connection strings.

### Setting a fixed address on the Raspberry Pi

If your router does not support setting a static ip address for a single device, you can also do this directly on the Raspberry Pi.

This can be done by configuring the DHCP-Client (on the Pi) to advertise a static IP address to the DHCP-Server (often the router) before it automatically assigns a different one to the Raspberry Pi.

1. Get ip address of default gateway (router).
   Run `netstat -r -n` and choose the IP address from the gateway column which is not `0.0.0.0`. In my occasion it's `192.168.178.1`.

2. Configure the static IP address for the Pi, the gateway path and a DNS server.
   The configuration for the DHCP client (Pi) is located in the `/etc/dhcpcd.conf` file:

   ```
   sudo nano /etc/dhcpcd.conf
   ```

   The following snippet is an example of a sample configuration. Change the value of `static routers` and `static domain_name_servers` to the IP of your router (default gateway) from step 1. Be aware of giving the Raspberry Pi an address which is **OUTSIDE** the range of addresses which are assigned by the DHCP server. You can get this range by looking under the router configurations page and checking for the range of the DHCP addresses. This means, that if the DHCP range goes from `192.168.178.1` to `192.168.2.99` you're good to go with the IP `192.168.178.100` for your Raspberry Pi.

   Add the following to the `/etc/dhcpcd.conf` file:

   ```
   # Configuration static IP address (CHANGE THE VALUES TO FIT FOR YOUR NETWORK)
   interface eth0
   static ip_address=192.168.178.100/24
   static routers=192.168.178.1
   static domain_name_servers=192.168.178.1
   ```

3. Restart networking system
   `sudo /etc/init.d/networking restart`

### Using WiFi instead of Ethernet

- Create a file `wpa_supplicant.conf` in the boot partition of the microSD card with the following content.
  Note that the network name (ssid) and password need to be in double-quotes (like `psk="password"`)

  ```conf
  ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
  update_config=1
  country=[COUNTRY_CODE]
  network={
    ssid="[WIFI_SSID]"
    psk="[WIFI_PASSWORD]"
  }
  ```

- Replace `[COUNTRY_CODE]` with the [ISO2 code](https://www.iso.org/obp/ui/#search){:target="\_blank"} of your country (eg. `US`)
- Replace `[WIFI_SSID]` and `[WIFI_PASSWORD]` with the credentials for your own WiFi.

### Using Umbrel on testnet/regtest

This guide is **only for Umbrel OS** and does not work on manual installations.

To change from mainnet to testnet on your Umbrel, [connect to your RPi via SSH](#can-i-login-using-ssh) and run the following commands:

```
cd ~/umbrel
sudo ./scripts/stop
sudo NETWORK=testnet ./scripts/configure
sudo systemctl restart umbrel-startup
```

### Manually accessing `bitcon-cli` and `lncli`

On Umbrel, these binaries are always available in UMBREL_ROOT_DIR/bin/. On Umbrel OS, you can [access them over SSH](#can-i-login-using-ssh) as

```
~/umbrel/bin/bitcoin-cli
```

and

```
~/umbrel/bin/lncli
```

### Reset the web ui password in case you lost it

Do this only if you **do not have any funds** on your LND wallet!

You are going to loose seed / settings / user data / application installed!

```
sudo systemctl stop umbrel-startup && sudo rm -rf ~/umbrel/lnd/!(lnd.conf) && sudo rm ~/umbrel/db/user.json && sudo rm ~/umbrel/db/umbrel-seed/seed && sudo systemctl start umbrel-startup
```

### Manually updating Umbrel

To manually update your node, run these commands [over SSH](#can-i-login-using-ssh):

```
cd ~/umbrel && sudo ./scripts/update/update --repo getumbrel/umbrel#v0.3.1
```

Replace v0.3.1 with the version you want to update to. 

If the update was stuck, run this before the above command:

```
sudo rm statuses/update-in-progress 
```

---

This troubleshooting guide will be constantly updated with findings that have been or will be reported in the issues section. Feel free to contribute via a pull request.

---
