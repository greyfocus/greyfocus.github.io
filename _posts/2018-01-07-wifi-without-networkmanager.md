---
layout: post
title: WiFi Networking without NetworkManager
date: 2018-01-07
keywords: linux, networkmanager, wifi
---

The `NetworkManager.service` provides a simple way to configure WiFi networking on Linux systems, but it can cause issues when running in a custom setup (in my case with multiple network adapters).

The instructions below should work for most Linux distributions that use systemd (e.g. Debian, Raspbian, Kali Linux, etc.).

### 1 Disable the NetworkManager service
```bash
systemctl disable NetworkManager
systemctl stop NetworkManager
```

### 2 WiFi configuration
Create `/etc/wpa_supplicant/wpa_supplicant.conf` based on the following template:

```
country=..		# ISO_3166 country code
update_config=1
ctrl_interface=/var/run/wpa_supplicant

network={
        scan_ssid=1
        ssid="Network SSID"     # WiFi SSID to connect to
        psk="password"      # Password
}
```

### 3 Apply the WiFi configuration on boot
Add the following lines to `/etc/rc.local` or create the file if it does not exist.

```bash
#!/bin/bash

(
        # Enable the network interface. Repeat this line for any other wifi interfaces 
        # that you want to use.
        ip link set wlan0 up
        
        # Connect to the configured access point using the wlan0 interface
        wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant/wpa_supplicant.conf
        dhclient -v wlan0
        
) &> /var/log/local_inet.log &
```

If you've just created the `rc.local` file, you will also need to mark it as executable:
```bash
sudo chmod a+x /etc/rc.local
```

### 4 Reboot
```bash
sudo reboot
```
