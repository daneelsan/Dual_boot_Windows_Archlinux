
# Dual boot with Windows (BIOS/MBR) - Archlinux

Simple instructions to install Arch Linux alongside Windows (Legacy-BIOS/MBR).

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


#### Installation

1. If not using an ethernet conexion:
    ```
    $ wifi-menu
    ```
    *Pick network, enter password.*

    Verify it's working by using [**ping**](https://en.wikipedia.org/wiki/Ping_(networking_utility):
    ```
    $ ping archlinux.org
    ```
2. Update the system clock
    ```
    $ timedatectl set-ntp true
    ```
    To verify use ```timedatectl status```.
3.
