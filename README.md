
# Dual boot with Windows (BIOS/MBR) - Archlinux

Simple instructions to install Arch Linux alongside Windows (Legacy-BIOS/MBR). Last update: 16/06/18.

Full documentation: [**Wiki**](https://wiki.archlinux.org/index.php/Installation_guide)

##### Windows UEFI vs BIOS

According to the [**ArchWiki**](https://wiki.archlinux.org/index.php/Dual_boot_with_Windows):

>  The best way to detect the boot mode of Windows is to do the following:
>
>  * Boot into Windows
>  * Press Win key and 'R' to start the Run dialog
>  * In the Run dialog type "msinfo32" and press Enter
>  * In the System Information windows, select System Summary on the left and check the value of BIOS mode item on the right
>  * If the value is UEFI, Windows boots in UEFI/GPT mode. If the value is Legacy, Windows boots in BIOS/MBR mode.

### Pre-installation

##### Windows setup

###### Partition

1. Within the console (`Win + R`) go to `Disk Management` by typing:
    ```
    $ diskmgmt.msc
    ```
2. Right click the partition you would like to expand and then choose `Extend Volume` (e.g. D partition).

###### Fast-boot

2. Open `Power options`.

3. “Choose what the power buttons do”.

4. "Change settings that are currently unavailable.”

5. Under “Shutdown settings” make sure “Turn on fast startup” is disabled.

###### Boot usb

6. Enter `BIOS` (F1, F2 or F10 during boot sequence).

7. Give a USB device boot sequence priority over the hard drive

##### Linux setup

##### Network connection

8. If not using an ethernet conexion:
    ```
    $ wifi-menu
    ```
    *Pick network, enter password.*

    Verify it's working by using [**ping**](https://en.wikipedia.org/wiki/Ping_(networking_utility):
    ```
    $ ping archlinux.org
    ```
9. Update the system clock
    ```
    $ timedatectl set-ntp true
    ```
    To verify use ```timedatectl status```.

#### Installation

###### Partition the disks

10. Identify the disks (e.g. `/dev/sda`) with [**fdisk**](https://wiki.archlinux.org/index.php/Fdisk):
    ```
    $ fdisk -l
    ```
    As you already have installed Windows, the output should look something like this:

    ```
    Disk /dev/sda: 931.5 GiB, 1000204886016 bytes, 1953525168 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 4096 bytes
    I/O size (minimum/optimal): 4096 bytes / 4096 bytes
    Disklabel type: dos
    Disk identifier: 0xd275cc93

    Device     Boot      Start        End   Sectors   Size Id Type
    /dev/sda1  *          2048    1026047   1024000   500M  7 HPFS/NTFS/exFAT
    /dev/sda2          1026048  437999615 436973568 208.4G  7 HPFS/NTFS/exFAT
    /dev/sda3        437999616 1401948159 963948544 459.7G  7 HPFS/NTFS/exFAT
    /dev/sda4       1401948160 1953525167 551577008   263G  5 Unallocated
    ```

    We are going to unallocated partition (i.e. `/dev/sda4`) to install Linux.

11. Enter the disk `/dev/sda`:
    ```
    $ fdisk /dev/sda
    ```
    With this we will enter `fdisk`'s command prompt.

    We will install Linux in 2 partitions (`root` and `home`). To do so, we will make the `Type` of the unallocated partition `Extended` ([**source**](https://askubuntu.com/a/575415)):
    ```
    $ Command (m for help): n
    ```
    Select `Extended` partition type. For the fist and last sector enter the default.

    Create the `root` partition (`n` command). Partition type: Primary. First sector: default. Last sector: +XXXG (i.e. +63G).

    Create the `home` partition. Partition type: Primary. First sector: default. Last sector: default.

    To finalize, enter `w` to write the changes.

    After `fdisk -l`, the output should look something like this:
    ```
    Device     Boot      Start        End   Sectors   Size Id Type
    /dev/sda1  *          2048    1026047   1024000   500M  7 HPFS/NTFS/exFAT
    /dev/sda2          1026048  437999615 436973568 208.4G  7 HPFS/NTFS/exFAT
    /dev/sda3        437999616 1401948159 963948544 459.7G  7 HPFS/NTFS/exFAT
    /dev/sda4       1401948160 1953525167 551577008   263G  5 Extended
    /dev/sda5       1401950208 1534070783 132120576    63G 83 Linux
    /dev/sda6       1534072832 1953525167 419452336   200G 83 Linux
    ```

###### Format the partitions

12. Format both partitions to `ext4`:
    ```
    $ mkfs.ext4 /dev/sda5
    ```
    ```
    $ mkfs.ext4 /dev/sda6
    ```
###### Mount the partitions

13. We will mount `/dev/sda5` to `/mnt` and `/dev/sda6` to `/mnt/home`:
    ```
    $ mount /dev/sda5 /mnt
    ```
    We need to create the directory `/home` first:
    ```
    $ mkdir /mnt/home
    ```
    ```
    $ mount /dev/sda6 /mnt/home
    ```
    Mount them all with `mount`.

###### Install the base packages

14. Use the [**pacstrap**](https://projects.archlinux.org/arch-install-scripts.git/tree/pacstrap.in) script to install the basic configuration:
    ```
    $ pacstrap /mnt base
    ```

###### Configure the system

15. Generate the [**fstab**](https://wiki.archlinux.org/index.php/Fstab). According to the wiki:
> The fstab file can be used to define how disk partitions, various other block devices, or remote filesystems should be mounted into the filesystem.

    ```
    $ genfstab -U /mnt >> /mnt/etc/fstab
    ```
    Look at it:
    ```
    $ cat /mnt/etc/fstab
    ```
16. Use `arch-chroot` to enter `/mnt` and change its privileges to [*root*](https://wiki.archlinux.org/index.php/Change_root):
    ```
    $ arch-chroot /mnt
    ```
17. Install packages using the [**pacman**](https://wiki.archlinux.org/index.php/pacman) command.
    ```
    $ pacman -S grub-bios linux-headers linux-lts linux-lts-headers
    ```
    The `linux-lts` (long term support) packages are *optional*.

18. *OPTIONAL* Install these packages for wireless-card ([**source**](https://youtu.be/oF_Zkq531qA?list=PLT98CRl2KxKHjq4YVsHqp9BbfDhtImhrN&t=200)):
    ```
    $ pacman -S dialog network-manager-applet networkmanager networkmanager-openvpn wireless_tools wpa_supplicant wpa_actiond
    ```

19. Recreate the [**initramfs**](https://wiki.archlinux.org/index.php?title=Initramfs&redirect=no) image (already done with `pacstrap`):
    ```
    $ mkinitcpio -p linux
    ```
    If you installed `linux-lts`, run the previous command again now using the lts version.

20. Uncomment localizations (i.e. en_US.UTF-8 UTF-8) in `/etc/locale.gen` (use `nano`). Generate them with:
    ```
    $ locale-gen
    ```

21. Set the root password with [**paswd**](https://jlk.fjfi.cvut.cz/arch/manpages/man/passwd.1).

###### Grub configuration

22. Run the following command:
    ```
    # grub-install --target=i386-pc --recheck /dev/sdX
    ```
    where `/dev/sdX` is the partitioned disk where grub is to be installed (e.g. `/dev/sda` and not partition `/dev/sda5`).

23. In the file `/etc/grub.d/40_custom` (or `/boot/grub/custom.cfg`) copy the following (for Windows Vista/7/8/8.1/10):

    ```
    if [ "${grub_platform}" == "pc" ]; then
      menuentry "Microsoft Windows Vista/7/8/8.1/10 BIOS/MBR" {
        insmod part_msdos
        insmod ntfs
        insmod search_fs_uuid
        insmod ntldr     
        search --fs-uuid --set=root --hint-bios=hd0,msdos1 --hint-efi=hd0,msdos1 --hint-baremetal=ahci0,msdos1 XXXXXXXXXXXXXXXX
        ntldr /bootmgr
      }
    fi
    ```
    where *XXXXXXXXXXXXXXXX* is the filesystem UUID which can be found with command `lsblk --fs` (in most cases this corresponds to the first partition, i.e. `/dev/sda1`).

24. Copy the locale:
    ```
    $ cp /usr/share/locale/en\@quot/LC_MESSAGES/grub.mo /boot/grub/locale/en.mo
    ```

25. Generate the main configuration file:
    ```
    $ grub-mkconfig -o /boot/grub/grub.cfg
    ```

###### Swap file

26. Create the [**swapfile**](https://wiki.archlinux.org/index.php/Swap#Swap_file):
    ```
    $ fallocate -l XG /swapfile
    ```
    where X (e.g. 2G, 4G) is the size of the file in gigabytes.

27. Change file permissions so that only the owner has privileges (using [**chmod**](https://wiki.archlinux.org/index.php/File_permissions_and_attributes#Changing_permissions)):
    ```
    $ chmod 600 /swapfile
    ```

28. Swap format:
    ```
    $ mkswap /swapfile
    ```
    Add this file to `/etc/fstab`:
    ```
    $ echo '/swapfile none swap sw 0 0' | tee -a /etc/fstab
    ```

29. Exit arch-chroot with `exit`. Unmount all devices with `umount -a`. Reboot (remove the USB).

###### Networking (wifi)

30. Check if you have an IP address with `ip`:
    ```
    $ ip a
    ```
    If not, check the status of `NetworkManager` using [**systemctl**](https://wiki.archlinux.org/index.php/systemd):
    ```
    $ systemctl status NetworkManager
    ```
    To enable/start it use the following command:
    ```
    $ systemctl XXXXX NetworkManager
    ```
    Where *XXXXX* can be start or enable.

31. Having installed `NetworkManager` we can use `nmcli`, a command line interface which we will use to connect to wifi.

    To show a list of UUID's:
    ```
    $ nmcli dev show
    ```
    Choose your corresponding wifi and connect using:
    ```
    $ nmcli dev wifi connect <SSID> password <PASSWORD>
    ```
    Verify you have an IP address with `ip a`.

###### Users

32. Add an user with [**useradd**](https://wiki.archlinux.org/index.php/users_and_groups):
    ```
    $ useradd -m <USER>
    ```
    Verify a user home directory has been created:
    ```
    $ ls /home
    ```
    Add a password to the user with `passwd <USER>`.

###### GUI

33. Install the display server [**Xorg**](https://wiki.archlinux.org/index.php/xorg):
    ```
    $ pacman -S xorg-server
    ```

34. Check which video driver you have with `lspci`. For example:
    ```
    00:00.0 Host bridge: Intel Corporation Xeon E3-1200 v3/4th Gen Core Processor DRAM Controller (rev 06)
    00:01.0 PCI bridge: Intel Corporation Xeon E3-1200 v3/4th Gen Core Processor PCI Express x16 Controller (rev 06)
    00:02.0 VGA compatible controller: Intel Corporation 4th Gen Core Processor Integrated Graphics Controller (rev 06)
    00:03.0 Audio device: Intel Corporation Xeon E3-1200 v3/4th Gen Core Processor HD Audio Controller (rev 06)
    00:14.0 USB controller: Intel Corporation 8 Series/C220 Series Chipset Family USB xHCI (rev 04)
    00:16.0 Communication controller: Intel Corporation 8 Series/C220 Series Chipset Family MEI Controller #1 (rev 04)
    00:1a.0 USB controller: Intel Corporation 8 Series/C220 Series Chipset Family USB EHCI #2 (rev 04)
    00:1b.0 Audio device: Intel Corporation 8 Series/C220 Series Chipset High Definition Audio Controller (rev 04)
    00:1c.0 PCI bridge: Intel Corporation 8 Series/C220 Series Chipset Family PCI Express Root Port #1 (rev d4)
    00:1c.2 PCI bridge: Intel Corporation 8 Series/C220 Series Chipset Family PCI Express Root Port #3 (rev d4)
    00:1c.3 PCI bridge: Intel Corporation 8 Series/C220 Series Chipset Family PCI Express Root Port #4 (rev d4)
    00:1d.0 USB controller: Intel Corporation 8 Series/C220 Series Chipset Family USB EHCI #1 (rev 04)
    00:1f.0 ISA bridge: Intel Corporation HM86 Express LPC Controller (rev 04)
    00:1f.2 SATA controller: Intel Corporation 8 Series/C220 Series Chipset Family 6-port SATA Controller 1 [AHCI mode] (rev 04)
    00:1f.3 SMBus: Intel Corporation 8 Series/C220 Series Chipset Family SMBus Controller (rev 04)
    01:00.0 3D controller: NVIDIA Corporation GK208M [GeForce GT 740M] (rev a1)
    03:00.0 Network controller: Qualcomm Atheros QCA9565 / AR9565 Wireless Network Adapter (rev 01)
    04:00.0 Ethernet controller: Qualcomm Atheros QCA8171 Gigabit Ethernet (rev 10)
    ```
    Check which video driver (`Inter`, `NVIDIA`, `VirtualBox`, etc.) you have under `VGA compatible controller` or `3D controller`. If you have both, I recommend installing the Intel driver.

    Intel:
    ```
    $ pacman -S xf86-video-intel libgl mesa
    ```
    NVIDIA:
    ```
    $ pacman -S nvidia nvidia-lts nvidia-libgl mesa
    ```
    VBOX:
    ```
    $ pacman -S virtualbox-guest-utils virtualbox-guest-modules- arch mesa
    ```
    (Remember to enable the `vboxservice.service` service with `systemctl enable`).

35. Now its time to install a [**display manager**](https://wiki.archlinux.org/index.php/Display_manager). In this example we will install [**SDDM**](https://wiki.archlinux.org/index.php/SDDM).
    ```
    $ pacman -S sddm
    ```
    Enable the service (but not start it yet):
    ```
    $ systemctl enable sddm
    ```

36. We will install the [**desktop environment**](https://wiki.archlinux.org/index.php/Desktop_environmen). I prefer [**KDE**](https://wiki.archlinux.org/index.php/KDE):
    ```
    $ pacman -S plasma-meta
    ```
    If you want the most basic installation, install `plasma` instead (remember to install some terminal like `konsole`).

37. REBOOT!
