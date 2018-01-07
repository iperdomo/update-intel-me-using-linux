
# How to update Intel ME firmware for Lenovo T460 using Linux

_TL;DR_ - Intel and Lenovo release the firmware updates as Windows executables
so you'll have to build a USB drive with
[Windows PE](https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/winpe-intro)
with all the necessary drivers and update utilities.
You boot your PC with the USB and execute the updates from _WinPE_.
If you have an spare PC with Windows, you're lucky and can skip all the
virtual machine part, if you don't, you can use use a virtualized Windows 10
for the USB preparation.

I spent 2 days fiddling with [Windows PE 7 (x86)](https://www.thinkwiki.org/wiki/Windows_PE)
but didn't managed to get it to work. Using Windows 10 (amd64) worked just fine.

## Disclaimer

The contents of this document are provided "AS IS" without warranty of
any kind.  This document could contain technical inaccuracies,
typographical errors and out-of-date information. This document may be
updated or changed without notice at any time. Use of the information
is therefore at your own risk.

## Emulating Windows 10 using VirtualBox

For Windows 10 emulation you'll need [VirtualBox](https://www.virtualbox.org/) and because
we'll emulate a 64bit Windows 10 machine, you must have
[Intel VT-X](https://support.lenovo.com/es/en/solutions/ht500006) enabled.

Instead of creating a virtual machine from scratch, we can use the work done
by Microsoft. Microsoft releases a set of virtual machines with some
combination of operating system and browser version. This machines are _Enterprise_ versions
of Windows and the license is a valid one for 90 days.

Visit [Microsoft Edge](https://developer.microsoft.com/en-us/microsoft-edge/tools/vms/) VM website
and download the file select:

* Virtual machine: __MSEdge on Win10(64bit) Stable (16.16299)__
* Select platform: VirtualBox
* Click: _Download ZIP_

__Note:__ The zip file is a ~4.9GB file.

![VM download](ms-edge-win10-vm.png)

After downloading the `MSEdge.Win10.VirtualBox.zip` you must unzip it to get a final `MSEdge - Win10.ova` file.

### Import the Windows 10 ova file

* File > Import Appliance

  ![Import](import-ova-1.png)

* Select the file `MSEdge - Win10.ova`

  ![Appliance to import](import-ova-2.png)

* Modify the CPU count to 2

  ![Appliance settings](import-ova-3.png)

* Click _Import_ and wait ...

  ![Importing appliance](import-ova-4.png)

### Modify display settings

* Use 128MB of _Video Memory_ and enabled _3D Acceleration_

  ![Display settings](display-settings.png)

### Adding an extra drive to the VM

I didn't manage to make VirtualBox to detect and _share_
a physical USB to the guest machine -- even with Oracle's
[extension pack](https://www.virtualbox.org/wiki/Downloads).
My workaround was to add an extra virtual disk (`.vdi`) and use it as a USB device
on Windows.

Let's create a 1GB additional disk

* Select the machine _MSEdge - Win10_

  ![Select vm](select-vm.png)

* Click _Settings_ button

  ![VM settings](vm-settings.png)

* Click _Storage_ option

  ![VM storage](vm-storage-1.png)

* Under the main IDE controller, click the _Add hard drive_

  ![Add hard drive](vm-storage-2.png)

* Select _Create new disk_

  ![Create new disk](create-new-disk-1.png)

* Select _VDI (Virtual Box Image)_

  ![Select VDI](create-new-disk-2.png)

* Select _Fixed size_

  ![Fixed size](create-new-disk-3.png)

* Save the `usb.vdi` in a known location and use `1.00 GB` as size

  ![1GB vdi](create-new-disk-4.png)

* Click _Create_

## Starting the VM

* Select the _MSEdge - Win 10_ machine
* Click Start

  ![Start VM](start-vm.png)

* Now you have a running Windows 10 VM

### Modifying the VM display resolution

To have a more comfortable workspace, modify the default display
resolution.

* Right-click on the _Desktop_
* Select _Display settings_

  ![Display settings](display-settings-1.png)

* Select _1280 x 960_ option
* Select _Keep changes_

  ![Keep changes](display-settings-2.png)

## Formatting extra drive (usb.vdi)

* Click Windows key

* Type `disk partitions`

* Select _Create and format hard disk partitions_

  ![Select format and hard disk partitions](format-usb-1.png)

* Select __MBR__ for _Disk 1_

  ![Select MBR](format-usb-2.png)

* Select _Disk 1_
* Right-click on the disk and select _New Simple Volume_

  ![New Simple Volume](format-usb-3.png)


* Accept all defaults and assign drive letter __D__
* Format with FAT32 file system

  ![FAT32](format-usb-4.png)

* Format the drive by clicking _Finish_

  ![Format](format-usb-5.png)

* You have now 2 drives and we'll use __D:__ as a USB drive

  ![2 Drives](format-usb-6.png)

## Installing Windows ADK

* Open Microsoft Edge browser
* Visit: http://go.microsoft.com/fwlink/?LinkId=526803

* Click _Download now_

  ![ADK download](win-adk-1.png)

* Select the option: _Run_

  ![Run the installer](win-adk-2.png)

* Select default location and accept the License

  ![Accept license](win-adk-3.png)

* From the default selected packages I removed:

 - Windows Performance Toolkit
 - Microsoft User Experience Virtualization (UE-V) Template
 - Microsoft Application Virtualization (App-V) Sequencer
 - Microsoft Application Virtualization (App-V) Auto Sequencer

 ![ADK features](win-adk-5.png)

* Click _Install_ -- and wait, you'll download ~ 6.4 GB

 ![Win PE download](win-adk-6.png)


## Customizing a Windows PE

The available documentation in
[Microsoft Docs](https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/winpe-intro) gives you all you need
to customize a Windows PE.

* Use the Windows key

* Search for _deployment_

* Right-click in the option _Deployment and Imaging Tools Environment_

* Run as administrator

  ![Deployment tools](custom-winpe-1.png)

* Using the console: `copype amd64 C:\WinPE_amd64`

  ![copype](custom-winpe-2.png)

* We're now ready to load some additional drivers

### Intel Chipset and ME drivers

In order to use the firmware update you need to have some additional
drivers in WinPE environment. You can find the required drivers on
[Lenovo's support portal](https://pcsupport.lenovo.com/es/en/products/laptops-and-netbooks/thinkpad-t-series-laptops/thinkpad-t460/20fn/downloads) under the
__Chipset__ section.

In my case I downloaded:

- Intel Management Engine 11.8 Firmware for Windows 10 (64-bit): https://download.lenovo.com/pccbbs/mobiles/r02rg06w.exe
- Intel(R) Management Engine Interface for Windows 10 (64-bit): https://download.lenovo.com/pccbbs/mobiles/r02mk15w.exe
- Intel(R) Chipset Device Software for Windows 10 (64-bit): https://download.lenovo.com/pccbbs/mobiles/r02ia08w.exe

![Additional drivers](custom-winpe-3.png)

* Save the files and execute them. This are self extractable zip files.
* __NOTE:__ Make sure you don't attempt to install this drivers on the virtual machine.
* Use the default path to extract the content:
  - `C:\DRIVERS\WIN\ME`
  - `C:\DRIVERS\WIN\MEI`
  - `C:\DRIVERS\WIN\Chipset`

### Extracting the content of the driver executables

* The first folder `C:\DRIVERS\WIN\ME` contains the firmware (`.bin` files)
plus the necessary script to update: `MEUpdate.CMD` -- We'll add this folder to the WinPE distribution.

* Using the same console...

* Move to `C:\DRIVERS\WIN\MEI`
 - `cd C:\DRIVERS\WIN\MEI`

* Create a new folder name `Drivers`
  - `mkdir Drivers`

* Execute `SetupME.exe` using the flags `-A` and `-P` to extract the content
of the file
  - `SetupME.exe -A -P C:\DRIVERS\WIN\MEI\Drivers`

  ![MEI extract](mei-extract.png)

* You'll find 2 new folders in under the `Drivers`, `HECI_REL` and `SOL_REL`

  ![HECI_REL](heci-rel.png)

* Move to the `Chipset` folder
  - `cd C:\DRIVERS\WIN\Chipset`

* Create a `Drivers` folder inside
  - `mkdir Drivers`

  ![Chipset drivers](chipset-drivers.png)

* Execute `SetupChipset.exe` with `-extract` flags
  - `SetupChipset.exe -extract C:\DRIVERS\WIN\Chipset\Drivers`

  ![Chipset drivers extract](chipset-extract.png)

* List the files under `Drivers` and you'll find a folder per processor
architecture. I'll use the `skylake` in this case

  ![Chipset drivers list](chipset-drivers-list.png)

## Customizing our WinPE environment with the firmware and drivers

The [Microsoft documentation](https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/winpe-mount-and-customize)
shows how to mount and customize our WinPE.

* Mount the image file

  - `Dism /Mount-Image /ImageFile:"C:\WinPE_amd64\media\sources\boot.wim" /index:1 /MountDir:"C:\WinPE_amd64\mount"`

  ![Dism 1](dism-1.png)

* Add the Intel Chipset driver
  - `Dism /Add-Driver /Image:"C:\WinPE_amd64\mount" /Driver:"C:\DRIVERS\WIN\Chipset\Drivers\skylake\SkylakeSystem.inf"`

  ![Dism 2](dism-2.png)

* Add the Intel ME driver
  - `Dism /Add-Driver /Image:"C:\WinPE_amd64\mount" /Driver:"C:\DRIVERS\WIN\MEI\Drivers\HECI_REL\win10\heci.inf"`

  ![Dism 3](dism-3.png)

  - `Dism /Add-Driver /Image:"C:\WinPE_amd64\mount" /Driver:"C:\DRIVERS\WIN\MEI\Drivers\SOL_REL\mesrl.inf"`

  ![Dism 4](dism-4.png)

* Add the firmware update and script by copying the `C:\DRIVERS\WIN\*` folder to the mount point

  - `xcopy /s C:\DRIVERS\WIN\* C:\WinPE_amd64\mount`
  - NOTE: While is not strictly necessary to copy the drivers binaries I did it for troubleshooting

  ![xcopy](xcopy-1.png)

* Add more temporary space

  - `Dism /Set-ScratchSpace:512 /Image:"C:\WinPE_amd64\mount"`

  ![Dism 5](dism-5.png)

* Unmount the file and __commit__ the changes

  - `Dism /Unmount-Image /MountDir:"C:\WinPE_amd64\mount" /commit`

  ![Dism 6](dism-6.png)

## Create a bootable media

* We'll use the extra drive added to the virtual machine
* `MakeWinPEMedia /UFD C:\WinPE_amd64 D:`

  ![Make media](make-media.png)

## Burning the created media to a USB drive

* Make sure to shutdown the Windows 10 virtual machine
* We have a bootable media under a .vdi format and we need to convert it to be able to _burn_ it
* Using `VBoxManage` we'll convert the `.vdi` in raw format. __NOTE:__ Change the command to the correct path to `usb.vdi`


    VBoxManage clonehd usb.vdi winpe_amd.img --format RAW
	0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
    Clone medium created in format 'RAW'. UUID: b4e20b0b-302d-44d4-8b10-40801fb35b88


* Using `dd` you can copy the resulting `winpe_amd64.img` to your usb.

__WARNING:   Make sure you choose the correct output otherwise you could delete something from your computer__

In my case `/dev/sdb` is the USB device

    sudo dd if=winpe_amd.img of=/dev/sdb bs=4M
    256+0 records in
    256+0 records out
    1073741824 bytes (1.1 GB, 1.0 GiB) copied, 87.5253 s, 12.3 MB/s

## Booting from USB and executing the firmware update

* Hit `<ENTER>` when the Lenovo logo shows up
* Select an alternative boot device - Your USB drive

  ![Boot Menu](boot-menu.png)

* Wait for Windows PE to boot
* Move to `X:\ME\` -- `cd X:\ME`
* Execute `UpdateME.CMD`

  ![MEUpdate](meupdate-cmd.png)

* Wait for the update to finish

  ![MEUpdate 2](meupdate-cmd-2.png)

  ![MEUpdate 3](meupdate-cmd-3.png)

* Use `wpeutil shutdown` to shutdown

  ![wpeutil](wpeutil-shutdown.png)

## Links

* https://www.thinkwiki.org/wiki/Windows_PE
* https://www.thinkwiki.org/wiki/Intel_Active_Management_Technology_(AMT)#Firmware_update
* https://www.reddit.com/r/thinkpad/comments/7ek838/how_to_patch_intel_sa00086_vulnerability_from/
* https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/winpe-intro
* https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/winpe-create-usb-bootable-drive
* https://developer.microsoft.com/en-us/microsoft-edge/tools/vms/
* https://developer.microsoft.com/en-us/windows/hardware/windows-assessment-deployment-kit#winADK
