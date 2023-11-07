# Enable Zigbee Dongles for DSM7 

This document helps with the setup of a startup script for DSM 7 to detect and load Zigbee Dongles like Conbee or Sonoff prior to loading Docker Containers.

## Intro

Synology DSM7 does not load, by default, drivers for external USB devices. These drivers can be loaded by the user however when the NAS is restarted - the drivers need to be loaded manually again. But with a couple of tricks - there is a possibility to load the drivers at every restart or boot-up.

DSM7 uses `systemd` for services statup operations. Systemd allows additional scripts and complex chaining of these. To be able to use the ConBee or a Sonoff USB dongle, three drivers are missing: `usbserial`, `ftdi_sio` and `cdc_adm`. These three must be loaded before docker daemon starts and pulls up your Deconz or Zigbee2MQTT container.

## Usage

### Prepare the script

Login via SSH into you synology. Copy the `pkg-zigbeedongle.service` startup script into `/usr/local/lib/systemd/system` by using the below command 

```bash
sudo wget https://raw.githubusercontent.com/EddieDSuza/synology-zigbee-dongle/main/pkg-zigbeedongle.service -P /usr/local/lib/systemd/system
```
Go to the below directory

```bash
cd /usr/local/lib/systemd/system
```

Provide elevated access to the file 

```bash
sudo chmod 777 pkg-zigbeedongle.service
```

Test The Elevated access

```bash
ls -l /usr/local/lib/systemd/system/pkg-conbee.service 
```

You will see the below output
```bash
-rwxrwxrwx 1 root root 340 Nov  6 19:43 /usr/local/lib/systemd/system/pkg-zigbeedongle.service
```

### Test the script

Run the following:

```bash
sudo systemctl start pkg-zigbeedongle.service
```

This will run the script. Now you can check modules in the kernel with `lsmod` as

```bash
lsmod |grep cdc_acm

cdc_acm                18255  2 
usbcore               202075  12 etxhci_hcd,usblp,uhci_hcd,usb_storage,usbserial,ehci_hcd,ehci_pci,usbhid,ftdi_sio,cdc_acm,xhci_hcd,xhci_pci
```

This means modules are loaded.

### Enable the script

The script must run at startup, so you need to tell `systemd` to do it

```bash
sudo systemctl enable pkg-zigbeedongle.service
```

If you now reboot, this script will run before docker and the container will find the old ACM tty.


##  The Startup script

```bash
[Unit]
Description=ConBee Module loader
Requires=pkg-volume.target
Wants=
Requisite=
After=
IgnoreOnIsolate=no
OnFailure=

[Install]
WantedBy=pkg-Docker-dockerd.service

[Service]
Type=simple
ExecStart=/bin/bash -c "modprobe usbserial && modprobe ftdi_sio && modprobe cdc-acm" 
TimeoutStartSec=3600
TimeoutStopSec=3600
RemainAfterExit=true
```

## Disclaimer

USE THIS AT YOUR RISK
