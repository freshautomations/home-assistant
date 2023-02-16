# Home-Assistant x86_64 Installer
(A Debian installer ISO for a supported Home Assistant.)

A better user experience for installing Home Assistant on Intel/AMD CPU (x86_64/amd64) machines.

Note: for ARM-based machines with the best user experience, check out
[Home Assistant Yellow](https://www.crowdsupply.com/nabu-casa/home-assistant-yellow), a Raspberry Pi-based
Home Assistant hardware solution.

### Problem description
From the Home Assistant installation guide:
> The HAOS has no integrated installer.
> This means the Operating System is not copied automatically to the internal disk.

Copying Home Assistant to the target boot medium on regular PCs is cumbersome and not user-friendly:
* some computers have non-serviceable hard drives. (for example the Mac mini)
* Users might not be comfortable taking their PC apart.
* Installing through USB to a fixed drive is not well documented and requires advanced Linux knowledge.

### Proposed solution
The Home Assistant supervisor is supported on Debian through a cumbersome
[installation procedure](https://community.home-assistant.io/t/installing-home-assistant-supervised-on-debian-11/200253).

This project merges that installation procedure into an installable Debian ISO image.

If the user is capable of installing an operating system through a USB disk or a CD-ROM, they will be able to install
Home Assistant (with supervisor in a supported setup) on a regular PC.

### How to use it
1. Find a usable USB drive with at least half a gigabyte capacity. All data will be deleted from this drive.
2. Download the [Home Assistant installation ISO](https://github.com/freshautomations/home-assistant/releases) and use
[balenaEtcher](https://www.balena.io/etcher) or your favorite USB burner to burn the ISO to the USB drive.
3. Boot from the USB drive on your PC and go through the installation. After reboot, Home Assistant should be available
on the usual ip:8123 address of the machine.

The process is similar with a CD-ROM. Find a tutorial on how to burn ISOs to CD-ROM.

### Which image shall I choose?
If you have a fresh **new (or freshly repurposed) hardware** with no valuable data on it, choose the standard "home-assistant"
image. This image will ask no questions during the install process, it will automatically partition the machine's disk drive
and set up networking through your home network's DHCP service. (If you have advanced knowledge, reserve an IP address
on your router for this machine.)

If you would rather go through the **Debian installer** as you would normally do, you can choose the `debian-assistant` image.
This image has no automated responses set so you will get a few questions during installation (which you might already know
from previous Debian installations) and it will also install an SSH server for remote access. This _technically_ makes the
installation unsupported from a Home Assistant perspective, but it does not seem to break anything.

### Additional security
Security is hard, and this is doubly true in the IoT space where you have underpowered devices who can not verify a
certificate properly (I'm looking at you ESP8266 with limited memory for a certificate chain...).

Still, it is highly recommended to secure your home assistant web page with TLS (http**s**). Unfortunately, this is not
built-in and Home Assistant does not support pre-configuration. So, for now, there are a few extra steps that should be
executed after the installation finished.

You can either follow some TLS guides, like [Duck DNS](https://www.home-assistant.io/integrations/duckdns/ but if you
plan to use the web interface internally, you can get away with a self-signed certificate, described below.

A self-signed certificate is automatically generated under the docker-mounted `/ssl` folder with the name
`home-assistant`. If this is not the name you want, you need to
[create a new certificate](https://stackoverflow.com/questions/10175812/how-to-generate-a-self-signed-ssl-certificate-using-openssl)
and install it. (You can use the File editor mentioned below to install it, you just need to disable the "Enforce basepath"
setting in its configuration.)


Step 1: 
You need to either have some internal DNS server or you need to edit your `/etc/hosts` file to add the below entry:
```
<IP_of_home-assistant> home-assistant
```
(or the name you chose in your custom certificate)

Step 2: (allow redirected incoming connections)
* Install the `File editor` addon from `Settings -> Add-ons -> Add-on Store`
([detailed instructions](https://github.com/home-assistant/addons/blob/master/configurator/DOCS.md)) and start it.
(Or use your favorite way to edit the configuration.yaml [described here](https://www.home-assistant.io/docs/configuration/#editing-configurationyaml).)
Set the `Show in sidebar` option to true for easier access.
* Open up the `/config/configuration.yaml` file using the File editor and add the below lines:
(this will redirect secure incoming requests to our local Home Assistant web interface) ([detailed description](https://github.com/home-assistant/addons/blob/master/nginx_proxy/DOCS.md))
```yaml
http:
  use_x_forwarded_for: true
  trusted_proxies:
    - 172.30.33.0/24
```
* Restart the core services from `Developer Tools -> Check and Restart section, Restart button` so the above setting activates.

Step 3: (create a secure endpoint for incoming connections)
* Install the `NGINX Home Assistant SSL proxy` addon from `Settings -> Add-ons -> Add-on Store` ([detailed instructions](https://github.com/home-assistant/addons/blob/master/nginx_proxy/DOCS.md))
* In the configuration section of the addon, set the domain to `home-assistant`.
* Start the add-on.

Note: if you see "400 Bad Request" when you try to use the address `https://home-assistant` page at this point, then you
didn't do Step 2 properly.

### Troubleshooting
If you receive an error message during installation, the fourth console screen (CTRL-ALT-F4) will show you the results
of the last few commands run.

On a successful installation, your server console will print the machine IP address on the login screen. If you do not
see a valid IP address, the installation failed somewhere.
See the `/var/log/installer/syslog` file for any indication of trouble. On a successful installation you should have a line
"Finished custom home-assistant preseed late_command execution." near the end.

On the supported `debian-...-home-assistant.iso` image, you can log in as root to the machine with the password
`home-assistant!default.password`. If you do so, change the password to something else that fits your needs.
(There is no SSH access available on this installation by default.)
