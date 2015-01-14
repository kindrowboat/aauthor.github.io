---

title:  "Connecting to University of Cincinnati's Securewireless with netctl"
date:   2015-01-13 17:14:00
categories: linux

---

If you've set-up an Arch Linux installation, and you've used `wifi-menu` then
you've used [netctl](netctl) before and didn't even know it.  `netctl` is

> ... a CLI-based tool used to configure and manage network connections via
> profiles. It is a native Arch Linux project for network configuration.

I used to think that the `wifi-menu` dialouge was a cute little installer helper
program, but I learned later, that it can be used to automatically generate
profiles in `/etc/netctl` that you can subsequently use to reconnect to network
later.

I was a little shocked and disappointed when `wifi-menu` failed to
connect to the University of Cincinnati's `Securewireless` network.  This short
guide will discuss the steps needed to connect to `Securewireless` using
`netctl`, and discuss why these extra steps are needed.

<!--more-->

*Connect to Securewireless — tl;dr*

Create and edit the file `/etc/netctl/INTERFACE-Securewireless` as root (using
`sudo`).  *Note that `INTERFACE` should be the name of your wireless interface.
Use `ip link` to find out what it is.  While the interface prefix is not
manditory, it does help you stay organized, and you'll need it below.*

```
Connection='wireless'
Interface=INTERFACE
Security='wpa-configsection'
Description="UC eduroam-like network"
IP='dhcp'
TimeoutWPA=30
WPAConfigSection=(
    'ssid="Securewireless"'
    'proto=RSN WPA'
    'key_mgmt=WPA-EAP'
    'auth_alg=OPEN'
    'eap=PEAP'
    'identity="UC_USER_NAME"'
    'password="UC_CENTRAL_LOGIN_PASSWORD"'
)
```

Where `INTERFACE` is your wireless interface as described above, UC_USER_NAME is
your 6+2 user name without the domain suffix (e.g. smithbb1 **not**
smithbb1@mail.uc.edu), and UC_CENTRAL_LOGIN_PASSWORD is the central login
password that you use for all of your UC services.


**Details**

The secret sauce is in the `wpa-configsection`/`WPAConfigSection`.  This allows
you to step outside of simple WEP/WPA/WPA2 shared passphrase paradigm and set
the security stack exactly how you need as if you were setting up wpa_supplicant
by hand.

University of Cincinnati uses, WPA Enterprise much like other universities.
According to [UC's IT Handbook (last page)](uc_it_handbook), `Securewireless`
uses:

* WPA2 Enterprise Security
* Protected Extensible Authentication Protocol (PEAP)
* No/self-signed security certificate

The problem is, the rest of the "fields" for the wireless setup are "Automatic",
how do you tell `netctl` to automatically figure out / negotiate the
authentication and encryption algorithms?  Setting `proto=RSN WPA` and
`key_mgmt=WPA-EAP` uses WPA or WPA2 using AES or TKIP (whichever the WAP
supports) and opens up the `auth_alg` and `eap` options, important to set `PEAP` and
Open System Authentication. `eap=PEAP` sets up the stack to use PEAP (the only
part of the stack the UC specifies), and `auth_alg=OPEN` enables the Open System
authentication algorithm.  Truth be told, most of these are default in
`wpa_supplicant`, so while I haven't had a chance to try it out, a simpler
profile might work:

```
Connection='wireless'
Interface=INTERFACE
Security='wpa-configsection'
Description="UC eduroam-like network"
IP='dhcp'
TimeoutWPA=30
WPAConfigSection=(
    'ssid="Securewireless"'
    'eap=PEAP'
    'identity="UC_USER_NAME"'
    'password="UC_CENTRAL_LOGIN_PASSWORD"'
)
```

There's even a chance you could leave out the `eap=PEAP` as that can negotiated
by wpa_supplicant too.

[netctl]: https://wiki.archlinux.org/index.php/netctl
[uc_it_handbook]: ucdirectory.uc.edu/studentplanner/ITHandbook.pdf
