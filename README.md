# Mass provisioning of Kismet and Apache MiNiFi in Raspberry Pi using Ansible.

![Mass provisioning of Kismet and Apache MiNiFi in Raspberry Pi using Ansible](https://github.com/chilcano/ansible-raspberrypi-wardriving/blob/master/images/mass-provisioning-kismet-minifi-raspberrypi-ansible-1-arch.png "Mass provisioning of Kismet and Apache MiNiFi in Raspberry Pi using Ansible")

Further details in https://holisticsecurity.io


## 1. Using Ansible Playbooks.


Connect 2 Raspberry Pi to your local LAN, make sure they have IP addresses assigned. You could use `fing` to discover those IP addresses. Both IP addresses will be used when configuring `ansible-raspberrypi-wardriving/inventory` and `ansible-raspberrypi-wardriving/playbooks/kismet/vars.yml`.


![Ansible Playbooks for building/compiling, installing and running Kismet on Raspberry Pi](https://github.com/chilcano/ansible-raspberrypi-wardriving/blob/master/images/mass-provisioning-kismet-minifi-raspberrypi-ansible-2-pkg.png "Ansible Playbooks for building/compiling, installing and running Kismet on Rasoberry Pi")


The first Raspberry Pi will be used to build the Kismet binary from the previously downloaded source code, and the second Raspberry Pi will be where I install and run Kismet. This second one must have an additional WIFI usb connected.

```
$ git clone https://github.com/chilcano/ansible-raspberrypi-wardriving

$ cd ansible-raspberrypi-wardriving
```

Update `ansible-raspberrypi-wardriving/inventory` with both IP addresses, for example, the IP `192.168.0.17` will be used to build/compile Kismet and where I will run Apache HTTPd, and IP `192.168.0.18` will be used to install and run Kismet.
```
$ nano inventory
```
```yaml
[pibuilder]
192.168.0.17

[pibuilder:vars]
ansible_ssh_user=pi

[pikismets]
192.168.0.18

[piminifis]
192.168.0.18

[pis:children]
pibuilder
pikismets
piminifis

[pis:vars]
ansible_ssh_user=pi

[nifis]
192.168.0.100
```

If you want change some values related to Kismet and the installation process, you have to update the file `ansible-raspberrypi-wardriving/playbooks/kismet/vars.yml`, for example:
```
$ nano playbooks/kismet/vars.yml
```
```yaml
---
### used by main_build.yml
kismet_url_tarball_src: "http://www.kismetwireless.net/code/kismet-2016-07-R1.tar.xz"
kismet_version: "2016-07-R1"
kismet_packaging: "tar.xz"
checkinstall_deldoc: true
checkinstall_nodoc: true
localmirror_dir_name: "localmirror"

### used by main_clean.yml and main_install.yml
warpi_start_script: warpi.sh
warpi_service_name: warpi
kismet_wifi_nic: wlan0
kismet_conf_log: /var/log/kismet
kismet_conf_logtypes: pcapdump,netxml
kismet_conf_logdefault: kismet
kismet_conf_configdir: .kismet

### remove dependencies
kismet_remove_dependencies: false
```

And finally run the Ansible Playbook `ansible-raspberrypi-wardriving/main_kismet_install.yml`.
```
$ ansible-playbook -i inventory main_kismet_install.yml -k
```

Now, just verify if Kismet is running and capturing WIFI anonymous traffic.
```
$ ssh pi@192.168.0.18

picuy@rpi18:/var/log/kismet $ sudo systemctl status warpi
● warpi.service - Enable monitor mode and manage Kismet Server as service
   Loaded: loaded (/etc/systemd/system/warpi.service; enabled)
   Active: active (running) since Wed 2017-03-22 20:33:14 UTC; 10min ago
 Main PID: 994 (warpi.sh)
   CGroup: /system.slice/warpi.service
           ├─ 994 /bin/sh /home/picuy/warpi.sh
           └─1003 kismet_server

Mar 22 20:40:00 rpi18.intix.info warpi.sh[994]: INFO: Detected new probe network "<Any>", BSSID 14:10:9F:E5:6E:CF,
```

## 2. Using 3 Ansible Roles.

I've refactored all Ansible Playbooks/Tasks in this [Github repo](https://github.com/chilcano/ansible-raspberrypi-wardriving) and I've created 3 [Ansible Roles](https://galaxy.ansible.com/chilcano). They are in Ansible Galaxy and are:

__Ansible Role Kismet RPi Build (https://galaxy.ansible.com/chilcano/kismet-rpi-build)__

An Ansible Role that builds / compiles from scratch and packs (Debian/Raspbian binary) Kismet on a Raspberry Pi. This Role provides the following features:

- Download the Kismet source code.
- Compile the source code in a Raspberry Pi.
- Generate a Kismet Debian/Raspbian package suitable for Raspberry Pi (ARMv7).

__Ansible Role Kismet RPi Wardriving (https://galaxy.ansible.com/chilcano/kismet-rpi-wardriving)__

An Ansible Role that installs, configures and runs Kismet on a Raspberry Pi. This Role provides the following features:

- Install Kismet and dependencies.
- Configure Kismet by deploying an customized kismet.conf.
- Download MAC Addresses Manufacturer List.
- Enable monitor mode in the Raspberry Pi before starting Kismet.
- Run Kismet as a systemd service.

__Ansible Role Apache MiNiFi (https://galaxy.ansible.com/chilcano/apache-minifi)__

An Ansible Role that installs, configures and runs Apache MiNiFi in tiny devices like a Raspberry Pi, although you can use it on any distro. This Role provides the following features:

- Install Apache MiNiFi and Java SDK.
- Configure Apache MiNiFi.
- Run Apache MiNiFi as a systemd service.

### 2.1. Using just 3 Ansible Roles.

Install all Ansible Roles needed as below or using `requirements.yml` file:
```
$ sudo ansible-galaxy install geerlingguy.apache
$ sudo ansible-galaxy install chilcano.kismet-rpi-build
$ sudo ansible-galaxy install chilcano.kismet-rpi-wardriving
$ sudo ansible-galaxy install chilcano.apache-minifi

$ ansible-galaxy list

- chilcano.apache-minifi, 1.0.1
- chilcano.kismet-rpi-build, 1.0.4
- chilcano.kismet-rpi-wardriving, 1.1.1
- geerlingguy.apache, 2.0.1
- knopki.locale, a1232f836b5514c58da381d9479640e40d874906
- Stouts.hostname, 1.1.0
- Stouts.timezone, 2.2.0
```

...and continue with this:
```
$ git clone https://github.com/chilcano/ansible-raspberrypi-wardriving --branch 2.0.2 --single-branch

$ cd ansible-raspberrypi-wardriving

$ ansible-playbook -i inventory main_all.yml -k
```

And that's it !.


## 3. References.

- (CheckInstall)[https://help.ubuntu.com/community/CheckInstall]
- (Compiling things on Ubuntu the Easy Way)[https://help.ubuntu.com/community/CompilingEasyHowTo]
- (StackOverflow: How to manage multiple package dependencies with checkinstall?)[http://stackoverflow.com/questions/18365600/how-to-manage-multiple-package-dependencies-with-checkinstall]
- (Everything generates data: Capturing WIFI anonymous traffic using Raspberry Pi and WSO2 BAM (Part I))[https://holisticsecurity.io/2016/02/02/everything-generates-data-capturing-wifi-anonymous-traffic-raspberrypi-wso2-part-i]
