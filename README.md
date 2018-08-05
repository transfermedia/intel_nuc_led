# DISCLAIMER

Use at your own risk! In the event that your computer breaks due to use of this software, the author will not be held responisble for any damages caused!

# Intel NUC8i7HVK LED Control

This is a simple kernel module to control the LEDs on Intel NUC8i7HVK (Hades) kits.

This module is intended as a demonstration/proof-of-concept and may not be maintained further.  Perhaps
it can act as a jumping off point for a more polished and complete implementation.  For testing and basic
manipulation of the LEDs, it ought to work fine, but use with caution none the less. This
has only been tested on 4.18 RC kernels.


## Requirements

Requirements:

* Intel NUC8i7HVK
* Latest BIOS
* ACPI/WMI support in kernel
* LED(s) set to `SW Control` in BIOS

## Building

THe `nuc_led` kernel module supports building and installing "from source" directly or using `dkms`.

### Installing Build Dependencies

Ubuntu:

```
apt-get install build-essential linux-headers-$(uname -r)

# DKMS dependencies
apt-get install debhelper dkms
```

Redhat:

```
yum groupinstall "Development Tools"
yum install kernel-devel-$(uname -r)

# Install appropriate EPEL for DKMS if needed by your RHEL variant
yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

yum install dkms
```

### Building and Installing "from source"

```
make clean
make install
```

### Building and Installing Using DKMS

Build and install without system packaging:

```
make dkms-install
```

Uninstall without system packaging:

```
make dkms-uninstall
```

Build and install using system packaging:

```
# Ubuntu
make dkms-deb

# RHEL
make dkms-rpm

# Install generated DEB/RPM from the folder specified in the output using system package manager
```

## Usage
    
This driver works via '/proc/acpi/nuc_led'.  To get current LED state:

```
cat /proc/acpi/nuc_led
```
    
To change the LED state:

```
 echo '<action>,<led id>,<indicator id>,[setting],[value]' | sudo tee /proc/acpi/nuc_led > /dev/null
```

|Action              |Description                  |
|--------------------|-----------------------------|
|set_indicator       |Change LED indicator type.   |
|set_indicator_value |Change LED indicator setting.|


Example execution to set Front 2 LED (5) to Wifi indicator type (3):

    echo 'set_indicator,5,3' | sudo tee /proc/acpi/nuc_led

Example execution to set the Skull LED (2), Indicator type Power state (0), setting S0 RGB color red (3) to 10:

    echo 'set_indicator_value,2,0,3,10' | sudo tee /proc/acpi/nuc_led > /dev/null
    
Errors in passing parameters will appear as warnings in dmesg.
**NOTE** Not all warnings are implemented and may send unsupported data, which can inactivate the LEDs (or worse!)


You can change the owner, group and permissions of `/proc/acpi/nuc_led` by passing parameters to the nuc_led kernel module. Use:

* `nuc_led_uid` to set the owner (default is 0, root)
* `nuc_led_gid` to set the owning group (default is 0, root)
* `nuc_led_perms` to set the file permissions (default is r+w for group and user and r for others)

Note: Once an LED has been set to `SW Control` in the BIOS, it will remain off initially until a color is explicitly set, after which the set color is retained across reboots.
