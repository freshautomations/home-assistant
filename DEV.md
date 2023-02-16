# Developer notes

## System defaults
* The installer uses the default US settings (keyboard, language, locale). Home Assistant can override these at first start.
* The installer expects one (and only one) disk installed in the machine.
* User creation is disabled, only the root is created with a default password.
* Home Assistant was complaining that systemd-resolved is not working. For some reason, it was disabled, so as a workaround
I explicitly enable it in the build script.

## Docker installation
* The get.docker.com was dissected to see requirements.
* The prerequisites are installed in the `pkgsel/include` section.
* Even though the Docker repository (and gpg key) is added in the `apt-setup/local0/...` section, the repository cannot
be used because it is over TLS and ca-certificates is not installed at this point.
* You could try installing it with `anna-install ca-certificates-udeb` in the earlier custom execution step, but then it
gets installed in the installer's environment (`/etc/ssl`) instead of the target environment (`/target/etc/ssl/certs`),
which is what we would need for `apt` to be able to pick it up.
* Hence, doing `pkgsel/include ca-certificates docker-ce` will not work. It can't install the certificates in the same step
where it needs them. There is no custom execution step where `/target` is already mounted but the default packages have
not been installed yet, so the only option is to install docker _after_ the first set of packages have been installed.

## unattended-upgrades
* Even though the `unattended-upgrades/enable_auto_updates` variable is set during installation, it doesn't apply.
(It should create a `/etc/apt/apt.conf.d/20auto-upgrades` file.) This is because the configuration option is
low priority while the automated installation runs at critical priority.
* Temporary workaround: we create the file manually during installation.
