---
description: Create an image from scratch for a Raspberry Pi
---

# Image Creation Steps \[Expert\]

## Setup the OGN receiver

Becoming a super hero is a fairly straight forward process:

```bash
$ give me super-powers
```

{% hint style="info" %}
 Super-powers are granted randomly so please submit an issue if you're not happy with yours.
{% endhint %}

Once you're strong enough, save the world:

```bash
# Ain't no code for that yet, sorry
echo 'You got to trust me on this, I saved the world'
```

### Get latest Raspbian Stretch Lite image and write it to SD card

Get lite version of raspbian from: [https://www.raspberrypi.org/downloads/raspbian/](https://www.raspberrypi.org/downloads/raspbian/) \(We don't need any Desktop related\)

### Enable ssh login

Put a file named ssh in /boot \([https://www.raspberrypi.org/documentation/remote-access/ssh/](https://www.raspberrypi.org/documentation/remote-access/ssh/)\)

{% tabs %}
{% tab title="MacOS" %}
```text
touch ~/ssh
```
{% endtab %}

{% tab title="Windows" %}

{% endtab %}

{% tab title="Linux" %}

{% endtab %}
{% endtabs %}

### Became super user

```bash
sudo su
```

{% hint style="danger" %}
With great power comes great responsibility
{% endhint %}

### Boot RPi with this SD card

Upgrade system:

```bash
apt-get update
apt-get dist-upgrade
rpi-update
```

### Update RPi configuration

1. Update hostname to `ogn-receiver` thanks to `raspi-config` 
2. Set up locale \(More instruction here\)

### Install standard OGN lib & softs + standard config

#### Get dependencies

```bash
apt-get install rtl-sdr libconfig9 libjpeg8 libfftw3-dev procserv telnet ntpdate ntp lynx dos2unix
```

#### Apply DVB-T blacklist

```bash
cat >> /etc/modprobe.d/rtl-glidernet-blacklist.conf <<EOF
blacklist rtl2832
blacklist r820t
blacklist rtl2830
blacklist dvb_usb_rtl28xxu
EOF
```

#### Install service

```bash
wget https://raw.githubusercontent.com/snip/OGN-receiver-RPI-image/master/dist/rtlsdr-ogn -O /etc/init.d/rtlsdr-ogn
wget https://raw.githubusercontent.com/snip/OGN-receiver-RPI-image/master/dist/rtlsdr-ogn-service.conf -O /etc/rtlsdr-ogn-service.conf
chmod +x /etc/init.d/rtlsdr-ogn
update-rc.d rtlsdr-ogn defaults
```

### Manage OGN-receiver.conf at boot time

* [x]  Generate `rtlogn-sdr` config from `OGN-receiver.conf`
* [x]  If exist use `/boot/rtlsdr-ogn.conf` at boot
* [x]  Disable pi user password login \(only ssh key login\)
* [x]  Change pi user password & allow password login
* [x]  Option to run a specific command at each boot
* [x]  Manage `rtlsdr-ogn` auto upgrade =&gt; Download at each `rtlsdr-ogn` startup.

```text
wget https://raw.githubusercontent.com/snip/OGN-receiver-RPI-image/master/dist/OGN-receiver.conf -O /boot/OGN-receiver.conf 
wget https://raw.githubusercontent.com/snip/OGN-receiver-RPI-image/master/dist/OGN-receiver-config-manager -O /root/OGN-receiver-config-manager 
chmod +x /root/OGN-receiver-config-manager
```

### Manage optional remote admin

```bash
apt-get install autossh
ssh-keygen
cat ~/.ssh/id_rsa.pub 
wget "https://raw.githubusercontent.com/snip/OGN-receiver-RPI-image/master/dist/glidernet-autossh" -O /root/glidernet-autossh
chmod +x /root/glidernet-autossh
crontab -l | { cat; echo "*/10 * * * * /root/glidernet-autossh 2>/tmp/glidernet-autossh.log"; } | crontab -
```

Return as `pi` user

```bash
exit 
# you'll see pi@ogn-receiver in the prompt
mkdir ~/.ssh
wget "http://autossh.glidernet.org/~glidernet-adm/id_rsa.pub" -O ~/.ssh/authorized_keys2
# return as super user
sudo su
```

### Manage self-update

Example taken from: [https://github.com/Hexxeh/rpi-update/blob/master/rpi-update\#L64](https://github.com/Hexxeh/rpi-update/blob/master/rpi-update#L64)

### Manage not blocking /etc/init.d/rtlsdr-ogn

{% hint style="warning" %}
This require explanations
{% endhint %}

### Add nightly reboot

```bash
crontab -l | { cat; echo "0 5 * * * /sbin/reboot"; } | crontab -
```

### TODO: how to get local time?

Maybe with [https://ipsidekick.com/json](https://ipsidekick.com/json) or [https://ipapi.co/timezone/](https://ipapi.co/timezone/) ? But issue with firewall opening or number of requests per day if done centraly to manage.

### Manage firstboot ? =&gt; Do we realy need it?

* To create hosts ssh keys on rw SD card. Then activate RO?
* In any cases root's ssh keys need to be the same for autossh remote admin.
* We need to expend FS at first boot =&gt; Do we realy need it? Don't think so.

```bash
raspi-config --expand-rootfs
```

### Add watchdog

```bash
echo "RuntimeWatchdogSec=10s" >> /etc/systemd/system.conf
echo "ShutdownWatchdogSec=4min" >> /etc/systemd/system.conf
```

See: [https://www.raspberrypi.org/forums/viewtopic.php?f=29&t=147501&start=25\#p1254069](https://www.raspberrypi.org/forums/viewtopic.php?f=29&t=147501&start=25#p1254069) And to check if it working well, to generate a kernel panic: `echo c > /proc/sysrq-trigger`

### Disable swap

```bash
systemctl stop dphys-swapfile
systemctl disable dphys-swapfile
apt-get purge dphys-swapfile
```

### Disable fake-hwclock?

```bash
update-rc.d fake-hwclock disable
```

As we are going to be RO file system we will not rely on `/etc/fake-hwclock.data`.

### Force time sync every 10 minutes?

```bash
crontab -l | { cat; echo "*/10 * * * * ( /usr/sbin/ntpdate -u pool.ntp.org ) > /tmp/ntp-sync.log 2>&1"; } | crontab -
```

### Add RO FS

```bash
cd /sbin
wget https://github.com/ppisa/rpi-utils/raw/master/init-overlay/sbin/init-overlay
wget https://github.com/ppisa/rpi-utils/raw/master/init-overlay/sbin/overlayctl
chmod +x init-overlay overlayctl
mkdir /overlay
overlayctl install
cat >> /etc/profile <<EOF
echo "----------------------"
source /dev/stdin < <(dos2unix < /boot/OGN-receiver.conf)
echo "OGN receiver \$ReceiverName"
echo "Read-only file system (overlay) status:"
/sbin/overlayctl status
echo "To manage it (as root): overlayctl disable | overlayctl enable | overlayctl status"
echo "----------------------"
EOF
```

\(from: [http://wiki.glidernet.org/wiki:prevent-sd-card-corruption](http://wiki.glidernet.org/wiki:prevent-sd-card-corruption)\)

### Cleanup installed image

From: [https://github.com/glidernet/ogn-bootstrap\#shrinking-the-image-for-distribution](https://github.com/glidernet/ogn-bootstrap#shrinking-the-image-for-distribution)

```bash
apt-get remove --auto-remove --purge libx11-.*
apt-get autoremove
apt-get autoclean
apt-get clean
```

### Reboot to apply changes

```bash
reboot
```



## 🎉 Congratulations! 🎉

Now you have a working OGN receiver!

You can now [configure it ](configure-the-station.md)and verify if it's [working correctly](debug.md).

### 

## \[Optional\] Creation of distributable image

### Read SD image to file & shrink it & compress it

### Disable overlay and reboot

```bash
sudo overlay disable something-something 
sudo reboot
```

#### Remove history

```bash
# Commands to remove history here
rm - some-bash-history
```

#### Fill not used space with 0 \(allow better compression\)

 This allow better compression. can be done at next step by mounting loopback FS**.**

```bash
dd if=/dev/zero of=file-filling-disk-with-0 bs=1M 
rm file-filling-disk-with-0
```

#### Create the image

Read image from another Linux, then:

```bash
shrink-ogn-rpi 2019-04-08-raspbian-stretch-lite-ognro.img
zip -9 2018-03-13-raspbian-stretch-lite-ognro.zip 2018-03-13-raspbian-stretch-lite-ognro.img
```

