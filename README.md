# tflight4
Linux Kernel HID Module for Thrustmaster T.Flight HOTAS 4 Joystick.

The joystick's HID Report Descriptor does not provide a mapping for the throttle, stick twist, or throttle paddles.
This driver replaces the HID report descriptor with one that (hopefully works).

All credit goes to the original [authors](https://github.com/walterschell/tflight4).

Forking because I want to incorporate both modes, the default mode with device ID 044f:b67b (red light) and the usb mode
with device ID 044f:b67c (green light), to map them to sane defaults.

The descriptor still seems to be missing some inputs, at least for me, and others are getting mapped to inappropriate
locations as far as I can tell.

### Note
The usb (green light) mode is achieved by plugging in the device with the three buttons on the base of the throttle
pressed.

# WARNING

This project is currently pre-alpha, so although the module will hopefully build and install on your machine, the
results will almost certainly not be what you want.

Releases will be unstable or broken during 0.1.x, but will hopefully improve thereafter, then the v1.0
release will be considered finished when both modes are supported with mappings to some subjectively reasonable
defaults.

# Development

The plan is to release a working version for usb mode, and then ideally to go back and incorporate the red light 
mappings and have the module choose the correct one when it is discovered.

All help is appreciated, so go ahead and open up an issue, then add a PR if you have a solution.

Also, it would be nice to get some sort of high level testing in place in the near future, so feel free to submit some
tests if that's your jam.


# How to Use (manually)

Install whatever package your distro requires for compiling C. ~~On ubuntu this is `build-essential`.~~

    pacman -S base-devel

Now build the module:

    make

Load the module into the current running kernel to test it out for your use. A reboot will undo this change:

    insmod hid-tflight4.ko

If it works well for you, then install it and set it up to be loaded at boot. After this next step, a reboot will not
undo this change:

    sudo cp hid-tflight4.ko /lib/modules/$(uname -r)/kernel/drivers/hid/ \
      && sudo depmod \
      && sudo modprobe hid-tflight4 \
      && grep -qxF 'hid-tflight4' /etc/modules || echo 'hid-tflight4' | sudo tee -a /etc/modules 
    
To undo the previous step, do the following:

    sudo rm -f /lib/modules/$(uname -r)/kernel/drivers/hid/hid-tflight4.ko \
      && sudo depmod \
      && sudo sed -i '/hid-tflight4/d' /etc/modules

# How to Use (DKMS)

The following list should do it:

```
~$ sudo -i
~# cd /usr/src
~# git clone https://github.com/walterschell/tflight4.git tflight4-1.0
~# dkms install tflight4/1.0
```

# Enable twist and seesaw axis separation

Add configuration to the modprobe.d:

```
~$ sudo -i
~# echo 'options hid-tflight4 throttle_seesaw_extra_axis=1' > /etc/modprobe.d/80-tflight4.conf
```

# TODO
Need to test if the optional foot rudders are correctly supported.
