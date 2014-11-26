# Now HLK-RM04 is supported by OpenWrt (r39237), please check the latest trunk to compile.

---

# OpenWrt-HiLink-HLK-RM04
Openwrt patch and installation guide for HiLink HLK-RM04

## Introduction
HLK-RM04 is a small wifi module which is produced by HiLink.
It is based on the Ralink RT5350 with some GPIOs raised out.  
HLK-RM04 has 4M flash and 16M SDRAM on-package, 1 USB port, 2 UART ports (lite and full), 1 I2C, 2 GPIO (GPIO0, RIN), and 2 Ethernet ports (LAN and WAN). In different configurations, you can get different numbers of GPIO pins (for example 8 GPIOs + 1 serial port).

## Files

- **openwrt-add-support-for-hilink-hlk-rm04.patch** -- patch to add "HILINK HLK-RM04" to OpenWrt
- **openwrt-fix-enable-uartf-kernel-panic.patch** -- patch to fix the kernel panic after enabling UARTF
- **openwrt-hlk-rm04-firwmware-tool.patch** -- this patch was written by Jeff Kent form openwrt forum; this patch allows us to upload firmware from official webUI upload firmware interface
- **openwrt-rt5350-add-AP+STA-support.patch** -- this patch was made by jonsmirl from OpenWrt mailing list. It allows one to use AP and STA modes simultaneously.  For now, the user needs to enable STA first.
- **[hlk-rm04-boot-log.md](./hlk-rm04-boot-log.md)** -- some hardware information and openwrt bootlog of HLK-RM04
- **image/uboot128.img** -- uboot for 16M SDRAM; if you don't modify the HLK-RM04, use this uboot image
- **image/uboot256.img** -- uboot for 32M SDRAM; if you have modified the HLK-RM04, use this uboot image to access the additional memory.
- **image/hlk-rm04-16m-luci-ser2net-usb2serial-r38025.bin** -- image with luci, usb2serial and ser2net. luci runs slowly, sometimes would also run out of memory.

## Patch and Compile Openwrt

In order to build OpenWrt for "HiLink HLK-RM04", you need to:

- download the latest OpenWrt trunk sources from svn
- download the patch
- apply the patch
- choose your target/subtarget/profile for the build
- compile the firmware 

This is achieved using the following code snippet:

    mkdir openwrt
    cd openwrt
    svn co svn://svn.openwrt.org/openwrt/trunk@38333 <--@38333 means force to check out Revision 38333
    git clone https://github.com/JiapengLi/OpenWrt-HiLink-HLK-RM04.git
    cd trunk
    patch -p0 <../OpenWrt-HiLink-HLK-RM04/openwrt-add-support-for-hilink-hlk-rm04.patch
    patch -p0 <../OpenWrt-HiLink-HLK-RM04/openwrt-hlk-rm04-firmware-tool.patch
    patch -p0 <../OpenWrt-HiLink-HLK-RM04/openwrt-fix-enable-uartf-kernel-panic.patch
    patch -p0 <../OpenWrt-HiLink-HLK-RM04/openwrt-rt5350-add-AP+STA-support.patch

Also you may want some extra package(if not, skip then) :

	./scripts/feeds update -a
	./scripts/feeds install -a

Then, empty the ./tmp and configure your openwrt. Run:
	rm -rf tmp
	make menuconfig

In the configuration menu, you need to select the following options:

Target System: Ralink RT288x/RT3xxx
Subtarget: RT305x based boards
Target Profile: HILINK HLK-RM04

If this is the first time you're loading OpenWRT on the HLK-RM04, please select:
*Target Image: ramdisk*  
By selecting this option, you can get a file named `openwrt-ramips-rt305x-hlk-rm04-initramfs-factory.bin`. Use this bin file to load the HLK-RM04 module for first time.

To use OpenWrt with LuCI Web UI, you can additionally select following options:

- LuCI --> Collections --> luci
- LuCI --> Protocols --> luci-proto-3g

After all the needed options are selected, exit the menu, save the configuration, and proceed to build:

	make

After compiling is done without any error, you'll find the image in `bin/ramips/` which is named `openwrt-ramips-rt305x-hlk-rm04-squashfs-sysupgrade.bin`. 

## Warning
Before installation, you need to make a choice. The memory configure resistors of the HLK-RM04 is in the wrong position, so we can't use it to automatically configure OpenWRT. Once the memory node can be used to force set the memmory size, but now it does not work in the latest trunk, maybe OpenWrt Developer changed the name. Two ways are found to solve this, hardware modification or force setting of the SDRAM size in Kernel\_menuconfig.
	
+ Hardware  
Follow <http://wiki.openwrt.org/toh/hilink/hlk-rm04?s#memory.configuration>, change the 2 memory configure resistors to 16M mode.
+ Software  
Run command:

		make kernel_menuconfig
find **kernel hacking**, press `Enter` and then find **(rootfstype=squashfs,jffs2) Default kernel command string**  
press `Enter` again and manually set SDRAM value `rootfstype=squashfs,jffs2 mem=16M`  
At last, recompile the OpenWrt:

		make


## Installtion
Jeff Kent made the firmware upload much easier. We can upload the OpenWRT firmware to the HLK-RM04 directly through the upload firmware WEBUI interface.
### First Time
Make sure you've gotton the `openwrt-ramips-rt305x-hlk-rm04-initramfs-factory.bin`, be careful with this.  It may brick your hlk-rm04 module, so you may need save your hlk-rm04 firmware first.

1. set your PC's IP address to **192.168.16.100**, Gateway **192.168.16.254**, may be different if you have changed it already.
1. connect HLK-RM04 LAN port to your PC, power up HLK-RM04
1. Navigate to <http://192.168.16.254/HLK_RM04.asp>; replace `192.168.16.254` with the IP address of your HLK-RM04 if you've changed it.
1. click the `Upload Firmware` on the left side, choose the `openwrt-ramips-rt305x-hlk-rm04-initramfs-factory.bin`, **Apply** and wait, then you got openwrt run on your hlk-rm04 module.

### Sysupgrade
After install openwrt first time, you can use openwrt sysupgrade command to upgrade hlk-rm04. [See wiki](http://wiki.openwrt.org/doc/howto/generic.sysupgrade)

## Installtion(This method is out of date)
So far, I don't make WEBUI upgrade firmware work for flash openwrt image. This installtion need two steps:

- replace the HiLink official uboot 
- use the new uboot to flash Openwrt image

### Prepare
- USB2Serial or USB2TTL tool
- One Network cable
- serial concole(Eg: putty)
- tftp server, (ubuntu tftp-hpa; Windows Tftpd32.exe)

### Step by Step

Replace the uboot

1. set your PC ipadress to **192.168.16.100**, Gateway **192.168.16.254**, may be difference if you have changed it once.
1. connect HLK-RM04 LAN port to your PC, power up HLK-RM04
1. access <http://192.168.16.254/adm/hlk_update_www_hlktech_com.asp>, replace `192.168.16.254` with yours, if you've changed it once.
1. in **Update Bootloader** option, click **Choose File**, then select **uboot128.img**, click **Apply** then click **OK** to make sure.
1. Here, we have replaced the uboot of HLK-RM04 successfully. And now the hlk-rm04 works fine, only thing what we change is let Uboot 'talk' to us, so that we can use uboot to download the openwrt image.

**Flash Openwrt with Uboot**

1. configure a tftp server. 
	- For Ubuntu, See more [Install tftp info][Install tftp]

	```bash
		sudo atp-get install tftpd-hpa 
		sudo service tftpd-hpa 
		cp bin/ramips/openwrt-ramips-rt305x-hlk-rm04-squashfs-sysupgrade.bin /var/lib/tftpboot/	
	```
 	- Download Tftpd32.exe, make `openwrt-ramips-rt305x-hlk-rm04-squashfs-sysupgrade.bin' to be in the same directory with Tftpd32.exe, choose a right server ip.
1. open serial use 57600,8,n,1, make sure you have connect the serial cable.
1. Restart your HLK-RM04 module. Push '2' rapidly to enter the tftp write flash mode.
1. Set device ip `192.168.16.1`, Set server ip `192.168.16.100`.
1. input the file name `openwrt-ramips-rt305x-hlk-rm04-squashfs-sysupgrade.bin`

	```
		2: System Load Linux Kernel then write to Flash via TFTP.
		 Warning!! Erase Linux in Flash then burn new one. Are you sure?(Y/N)
		 Please Input new ones /or Ctrl-C to discard
		        Input device IP (10.10.10.123) ==:192.168.16.1
		        Input server IP (10.10.10.3) ==:192.168.16.100
		        Input Linux Kernel filename (tim_uImage) ==:openwrt-ramips-rt305x-hlk-rm04-squashfs-sysupgrade.bin
	```
1. Wait flashing finished, then you can use Openwrt on HLK-RM04. 

### De-bricking

If you've not overwritten the UBoot bootloader, but you've bricked the rest of the firmware, you can de-brick the system through UBoot:

1. Download the necessary files to de-brick. The firmware in image/hlk-rm04-16m-luci-ser2net-usb2serial-r38025.bin (this repository) should work.  You will also need a TFTP server.  See above section for Linux.  For Windows, get Tftpd32.
1. Attach your HLK-RM04's LAN port to an ethernet switch.
1. Attach your PC to the same ethernet switch.
1. Set your PC's IP address to 10.10.10.3
1. Set Tftpd32 up as a TFTP Server.  You do not need to run a DHCP server.
1. Copy the firmware image that you wish to use into the TFTP root directory as `tim_uImage`.
1. Hold the WPS switch on the HLK-RM04.  If you have the bare module, look at the reference design schematic to find the WPS switch.
1. Press and release the Reset switch on the HLK-RM04.
1. Release the WPS switch on the HLK-RM04.
1. Watch Tftpd32 transfer `tim_uImage` (the firmware) to the device.  Wait five minutes after it completes.
1. Reset the HLK-RM04.

## About Patch

### openwrt-add-support-for-hilink-hlk-rm04.patch
This patch is based on previous work by Squonk42 (<https://github.com/Squonk42/OpenWrt-RT5350>), shmygov (<https://github.com/shmygov/OpenWrt-HAME-MPR-A2>)  and others from OpenWrt forum (https://forum.openwrt.org/viewtopic.php?id=37002&p=19), adapted for the new "Device Trees" structure based on dts files used in the latest OpenWrt trunk. Because need to change the order of `uart` and `uartlite` node in the rt5350.dtsi file, so that linux kernel register `uartlite` and `uart` in `ttyS0` and `ttyS1` sequence, so didn't use the rt5350.dtsi file just create a new one.  
**Note** This patch use GPIO0 for system status LED, you may need solder one on you HLK-RM04. And WPS button is connected to "RIN/GPIO14" pin.

### openwrt-fix-enable-uartf-kernel-panic.patch
After enable uartf we come across a kernel panic, this patch fixes it. Reference [OpenWrt Ticket #13590][ticket]

## End
This patch is made with the help of many guys, Jeff who find the the extra at command, Tao Zhou who help to solve the ttyS* sequence problem, and John Crispin and Sebastian Muszynski from the openwrt mail list.

[Install tftp]: http://www.cyberciti.biz/faq/install-configure-tftp-server-ubuntu-debian-howto/
[ticket]: https://dev.openwrt.org/ticket/13590


[![Bitdeli Badge](https://d2weczhvl823v0.cloudfront.net/JiapengLi/openwrt-hilink-hlk-rm04/trend.png)](https://bitdeli.com/free "Bitdeli Badge")

