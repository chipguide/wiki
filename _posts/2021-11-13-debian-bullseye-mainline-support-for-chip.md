---
title: Debian Bullseye mainline support for CHIP
layout: post
---

Thanks to the hard work of user _macroalpha_ we now have a mainline Debian Bullseye image available for the CHIP and Pocket CHIP. This means the CHIP can finally run modern software and receive future updates without the fear of breaking anything. With _macroalpha_'s Bullseye the CHIP moves from the _4.4.13-ntc_ Linux kernel to an unmodified _5.14_ kernel.

From [_macroalpha_'s release notes thread](https://old.reddit.com/r/ChipCommunity/comments/qnc53k/updated_mainline_debian_11_image/):

> This image should be universal in that it works for both the Hynix and Toshiba CHIPs, and it supports autodetection of the official NTC DIPs and loads the appropriate overlays (though note since I do not have the VGA or HDMI DIPs I "guessed" at what those overlays should look like, feedback would be appreciated if you encounter issues with them). This device has a modified device tree, but otherwise runs an entirely mainline kernel with a patch fixing a bug I reported upstream for the I2C bus and another patch adding the Hynix NAND which has already been accepted for the 5.16 kernel. Mainline U-Boot is also used, however it includes a rather substantial patch adding support for slc-emulation for MLC NAND which is still pending review for mainline acceptance.
>
> The gist of the flashing process is basically
>
> 1. put your CHIP in FEL mode
> 2. run a script
> 3. when your CHIP shuts down take it out of FEL mode and plug in a flash drive to it
> 4. turn it on
> 5. connect to it with a terminal program (such as putty, screen, etc)
> 6. flash the rootfs
> 7. you're done.

## Requirements

1. Either a Linux or Mac host machine, with the latest version of Sunxi tools installed.
2. A FAT32 formatted USB flash drive
3. Micro-USB cable
4. CHIP

## Building Sunxi tools on a Linux host

These instructions are for a Linux host running Ubuntu 20.04.3 (but they should be similar for most versions of Ubuntu/Debian)

```shell
sudo apt install git build-essential libusb-1.0-0-dev libfdt-dev zlib1g-dev
git clone https://github.com/linux-sunxi/sunxi-tools.git
cd sunxi-tools
sudo make install-tools
```

## Building Sunxi tools on a Mac host

These instructions assume you're using [brew](https://brew.sh) to manage your packages.

```shell
brew install u-boot-tools libusb zlib pkg-config git 
git clone https://github.com/linux-sunxi/sunxi-tools.git
cd sunxi-tools
pkg-config --libs libusb-1.0
pkg-config --libs zlib
sudo make install-tools
```

You should now have the `sunxi-fel` command available.

## Grab the flash files

You can download the latest `flash_tools.tar.bz2` and `rootchipbasicuniversal.ubi` from here:

- [https://macromorgan.s3.amazonaws.com/ntc-chip-mainline/flash_tool.tar.bz2](https://macromorgan.s3.amazonaws.com/ntc-chip-mainline/flash_tool.tar.bz2)
- [https://macromorgan.s3.amazonaws.com/ntc-chip-mainline/rootchipbasicuniversal.ubi](https://macromorgan.s3.amazonaws.com/ntc-chip-mainline/rootchipbasicuniversal.ubi)

Extract the `flash_tools.tar.bz2` somewhere, and copy the `rootchipbasicuniversal.ubi` to a FAT32 formatted USB flash drive and plug it into your CHIP for later.

## Put your CHIP in to FEL mode

Connect the FEL and Ground pins on your CHIP with a jumper, and then connect the CHIP to your computer with a USB cable.

![Picture showing how to jumper the FEL pin to GND](../images/{{page.date|date:"%Y-%m-%d"}}/fel-jumper.jpg)

## Identify your CHIP flash type

The CHIP comes with either a Hynix or Toshiba flash. To identify which your CHIP has, look at the following diagram:

![Picture illustrating the differences between Hynix and Toshiba flash chips](../images/{{page.date|date:"%Y-%m-%d"}}/chip-types.jpg)

Most of the flash chip will be covered with QC stickers, but you should be able to make out part of the word "HYNIX" or "TOSHIBA" on either flash chip. The Toshiba flash chip also features a circle dimple in two of it's corners, one of which is larger than the other.

## Run the flash_xxx.sh

Inside the extracted `flash_tool` folder, run either the `flash_hynix.sh` or `flash_toshiba.sh` depending upon which type of CHIP you have. Note that you may have to run this as root. (e.g. `sudo ./flash_toshiba.sh`). 

This tool will run for a while, between 5 and 10 minutes, because uploading the mini-rootfs over FEL takes time.

When the CHIP shuts down, remove the FEL jumper and if you have not already plugged in the flash drive to the USB port that contains the `rootchipbasicuniversal.ubi` please do so now. Make sure the device remains plugged into your computer and power the CHIP back on.

After about 10-20 seconds of booting, the device should be ready to log in. Use your favorite serial terminal program (such as `screen`) and connect to the device as a serial gadget. For example, the device might show up as `/dev/ttyACM0`, so you would connect to it like this:

```shell
screen /dev/ttyACM0 115200
```

Note that if your user is not in the dialup group you will likely need to run this command with sudo as well. If your device does not show up as `/dev/ttyACM0`, check your `dmesg` log to see if it shows up under a different name.

## Write the root image to the CHIP's internal NAND

Log into the CHIP with the username of `root` and no password, then mount the USB flash drive and use `ubiformat` to write the root image to the NAND:

```shell
mount /dev/sda1 /mnt
ubiformat /dev/mtd3 -f /mnt/rootchipbasicuniversal.ubi
```

## All done!

When the device is done flashing (but the power is still off) hook up any DIPs you have to it (e.g. VGA, HDMI or Pocket CHIP) and then turn it on.

Regardless of connected DIPs, the device will boot when you turn it on with the micro USB port acting as a USB Serial gadget, so you can connect to it without any video or keyboard attached.

Use the same command as you did in the initial flashing environment (e.g. `screen /dev/ttyACM0 115200`). Username and password are both `chip`.

## Next steps

The root image provided is very basic, so you will need to install and configure things to your liking. 

First you might want to connect to Wi-Fi. You can do this with the following command:

```shell
sudo nmcli dev wifi connect "YOUR NETWORK SSID" password "YOUR NETWORK PASSWORD"
```

After your Wi-Fi is connected, you can use `apt` to install software.

If you want to build your own root image, you can follow the instructions and configurations provided by _macroalpha_ here: [macromorgan/chip-debroot](https://github.com/macromorgan/chip-debroot)