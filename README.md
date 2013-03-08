
**This program (including documentation) is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied**
**warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU General Public License version 3 (GPLv3; http://www.gnu.org/licenses/gpl-3.0.html ) for more details.**

**If you don't know what you are doing, think twice before using this guide.**
**I take absolutely no responsibility for any damage(s) that you might cause to your hard- or software-environment by following the descriptions below!**


Pogoplug_V3_Emdebian
====================

Scripts to create a bootable Emdebian USB-Stick for the Pogoplug V3 devices.

ATTENTION:
==========
**A serial connection to the Pogoplug is highly recommended for this procedure.**

**Most of the descriptions below assume that you are able to connect to the Pogoplug via serial connection!!!**

HOWTO: Emdebian on the Pogoplug V3 (Classic and/or Pro)
------------------

Step-by-step description on how to get Emdebian runnning on your Pogogplug V3:

1. Boot your Pogoplug V3 and install Arch Linux, like explained here: <http://archlinuxarm.org/platforms/armv6/pogoplug-provideov3>
2. Then get your Debian or Ubuntu Host machine ready and get the scipts via git or download them as a zip file: **`'git clone git://github.com/ingmar-k/Pogoplug_V3_Emdebian.git'`** **_OR_** <https://github.com/ingmar-k/Pogoplug_V3_Emdebian/archive/master.zip>
3. Make the file **build_emdebian_system.sh** executable, by running **`'chmod +x build_emdebian_system.sh'`**
4. **VERY IMPORTANT:** Edit the file **general_settings.sh** to exactly represent your host system, Pogoplug-device and network environment.
5. Run the script **with root privileges (su or sudo!)** by typing **`'./build_emdebian_system.sh'`**
6. When the script is done, boot your Pogoplug Classic/Pro with the newly created USB-drive attached.
7. If everything went well, the Plug should boot fine and be accessible via SSH (if you installed the package --> **general_settings.sh** ).



Flashing and testing a new kernel
-----------------

If you want to use a new/newer kernel, you need to replace the original (Arch Linux-)Kernel in NAND.

Here is how to do that:

1. Boot your pogoplug and get (through USB, wget etc. ) the new uImage and kernel modules.
2. Make sure to place the new kernels modules directory into **/lib/modules** !
3. Then run **`'/usr/sbin/flash_erase /dev/mtd1 0xB00000 24'`** to delete the backup image in flash.
4. By running **`'/usr/sbin/nandwrite -p -s 0xB00000 /dev/mtd1 /path/to/new/uImage'`** you write the newer uImage to flash.
5. Reboot the Pogoplug and interrupt the boot process at the Uboot prompt( **CE>>** ).
6. In order to boot the backup kernel image directly, instead of the main image, run **`'run load_custom_nand2 boot'`** (as found in the second half of the **boot_custom** command, shown by running **`printenv`** ) 
7. This will boot the backup kernel image **for one time only**. At the next reboot, the default command will be run again.
8. **Extensively (!!!)** test the kernel before thinking about making it your default kernel!
9. To make this new kernel the default, repeat steps 3. and 4. with the hex adress **0x500000**, INSTEAD OF **0xB00000**.
10. The Pogoplug will then boot to the new kernel by default.



Root-filesystem in NAND
-----------------

Now that the Pogoplug boots Emdebian from USB, the next possible (but optional) step is putting the rootfs into NAND.

1. First, shut down your Pogoplug, remove the USB drive and attach it to your desktop system that was used to create the Emdebian rootfs.
2. Create a new directory on the USB drive (for example named **nand_rootfs**).
3. Extract the created rootfs archive (by default a tar.bz2. file) into the newly created directory.
4. Open the filesystem-table used for mounting the Rootfs ( **../nand_rootfs/etc/fstab** ) with an editor (for example **nano**).
5. Remove the 2 lines **'/dev/root	/	ext3	defaults,noatime	0	1'** and **'/dev/sda2	none	swap	defaults	0	0'**, **replace** them with **'/dev/root	/	ubifs	defaults,noatime	0	0'** and save the file.
6. To delete the contents of the old rootfs in nand, run the command **`'flash_eraseall /dev/mtd2'`** ( with mtd2 being the rootfs partition according to **`cat /proc/mtd`** ).
7. Change into a different directory, that is not part of the **nand_rootfs** dir !!!
8. Create a file called **ubinize.cfg**, with the following content: 

    [ubifs]
    
    mode=ubi
    
    image=ubifs.img
    
    vol_id=0
    
    vol_size=100MiB
    
    vol_type=dynamic
    
    vol_name=rootfs
    
    vol_alignment=1
    
    vol_flags=autoresize
    
9. Check your boot log for the UBI entry called **UBI: available PEBs:** and memorize or write down the number ( should be something like '897' ).
10. Run **`'mkfs.ubifs -r /nand_rootfs -m 2048 -e 129024 -c 897 -x zlib -o ubifs.img'`** with the parameters **fitting your system (very important, the number after '-c' is the one you memorized)**.
11. Then run **`'ubinize -o ubi.img -m 2048 -p 128KiB -s 512 ubinize.cfg'`** to create the final image, ready to flash.
12. To flash that image to NAND, you first need to detach second the partition (**mtd2**) from UBI, by running **`'ubidetach /dev/ubi_ctrl -m 2'`** .
13. Finally flash the created image to NAND by running **`'ubiformat /dev/mtd2 -f ubi.img'`** .
14. Reboot the Pogoplug and again **interrupt the boot process at the Uboot prompt**.
15. To boot the system from NAND, run the commmand **`'setenv bootargs $bootargs_stock'`** , followed by **`'run boot_custom'`**.
16. This change again is only temporary until the next reboot.
17. To make the Pogoplug boot from NAND by default, repeat step 14, followed by running **`'saveenv'`** .
18. Now the Pogoplug should boot to NAND by default.
