# DeepRacer Upgrade 
This is a step by step walk through to update DR to the latest image needed to used with DREM (Deep Racer Event Manager) 

Follow the steps preferrably on a PC (Windows) machine - please follow carefully - hereby we indemnify ourselves from any damage you might cause to your DR or other properties using the script. 

### Windows PowerShell version
Follow the instructions here to use a Windows computer to prepare the update media for your AWS DeepRacer device.

To erase the USB drive
Open the Windows command prompt, enter diskpart, and choose OK to launch Windows DiskPart.

Once the terminal for Microsoft DiskPart opens, list the available disks to find the USB drive you want to clean by entering list disk after the DISKPART> prompt.

Select the disk corresponding to your USB drive. For example, we entered select Disk 2 after the DISKPART> prompt. Read the output carefully to verify that you have chosen the disk you want to clean because the next step is irreversible.

Once you are sure that you've selected the correct disk, enter Clean after the DISKPART> prompt.

Enter list disk after the DISKPART> prompt again. Find the disk you cleaned on the table and compare the disk size to the free disk space. If the two values match, the cleaning was successful.

Exit the Windows DiskPart console by entering Exit after the DISKPART> prompt.

To partition the USB drive
Open the Windows command prompt, enter diskmgmt.msc, and choose OK to launch the Disk Management console.

From the Disk Management console, select your USB drive.

To create the FAT32 partition with a 4 GB capacity, open the context (right-click) menu on your USB drive's Unallocated space and choose New Simple Volume. The New Simple Volume Wizard will appear.

Once the New Simple Volume Wizard appears, do the following:

On the Specify Volume Size page, set the following parameter and then choose Next.

Simple volume size in MB: 4096

On the Assign Drive Letter or Path page, check the Assign the following drive letter: radio button and select a drive letter from the drop down list, then choose Next. Make note of the assigned drive letter, you will need it later to make the FAT32 partition bootable.

On the Format Partition page, check the Format this volume with the following settings radio button and set the following parameters, then choose Next.

File system: FAT32

Allocation unit size: Default

Volume label: BOOT

Leave Perform a quick format checked.

To create the NTFS partition with the remaining disk capacity, open the context (right-click) menu on your USB drive's remaining Unallocated space and choose New Simple Volume. The New Simple Volume Wizard will appear.

Once the New Simple Volume Wizard appears, do the following:

On the Specify Volume Size page, set the Simple volume size in MB to match the Maximum disk space in MB, then choose Next.

On the Assign Drive Letter or Path page, check the Assign the following drive letter: radio button and select a drive letter from the drop down list, then choose Next.

On the Format Partition page, check the Format this volume with the following settings radio button and set the following parameters, then choose Next.

File system: NTFS

Allocation unit size: Default

Volume label: Data

Leave Perform a quick format checked.

To make the USB drive bootable from the FAT32 partition
Make sure you've downloaded the customized Ubuntu ISO image from the prerequisites section.

After downloading UNetbootin, start the UNetbootin console.

On the UNetbootin console, do the following:

Check the Disk image radio button.

For disk image, choose ISO from the drop-down list.

Open the file picker and choose the custom Ubuntu ISO file.

For Type, choose USB Drive.

For Drive, choose the drive letter corresponding to the FAT32 partition you created. In our case, it's E:\.

Choose OK.
**Requirements**:

- Run in PowerShell command window (**NOT** run as Administrator, but you will be prompted to elevate with Administrator at some point in the script=
- First make sure you can actually run scripts in your Powershell ISE :
Open PowerShell as an administrator.
Run the following command: ```Set-ExecutionPolicy -ExecutionPolicy RemoteSigned ```
When prompted, type Y and press Enter to confirm the change.

Also set the execution policy of PowerShell to  non-restricted mode, which allows scripts running.

Open PowerShell as an administrator.
Run the following command: ```Set-ExecutionPolicy -ExecutionPolicy RemoteSigned```  then 
```Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass```
When prompted, type Y or Y to all and press Enter to confirm the change.

**Command**:


Please pay attention to the ' around the password and SSID'
```
.\usb-build.ps1 -DiskId <disk number> -SSID 'yourssidhere' -SSIDPassword 'yourpasswoord'
```

Additional switches:

Description                                                    | Switch
---------------------------------------------------------------|---------------------------------------------------
Provide Wifi Credentials                                       | `-SSID <WIFI_SSID> -SSIDPassword <WIFI_PASSWORD>`
Create partitions (default value is True)                      | `-CreatePartition <True/False>`
Ignore lock files (default value is False)                     | `-IgnoreLock <True/False>`
Ignore Factory Reset content creation (default value is False) | `-IgnoreFactoryReset <True/False>`
Ignore Boot Drive creation (default value is False)            | `-IgnoreBootDrive <True/False>`

**Note:**

- The wifi credentials are used to create `wifi-creds.txt` on the `DEEPRACER` partition, upon rebooting after flashing the car will use this file to connect to wifi
- Tested and working on Windows 11 Pro


## tweaks.sh
make sure your script has execution permission - by default it wont run on your Deepracer  you need to run `sudo  chmod +x tweaks.sh`
A script to change a couple of things on the car that I've found useful at events

- Add `deepracer` user to `sudoers`
- Change the hostname
- Change the car console password (**Note:** Update the default password in the script)
- Update Ubuntu
- Update the car software
- Disable IPV6 on network interfaces
- Disable the video stream on the car console by default
- Disable system suspend
- Disable network power saving
- Disable the software update check
- Enable SSH (You've probably already done this)
- Allow multiple logins to the car console
- Increase the car console cookie duration
- Disable Gnome, Bluetooth & CUPS

Command:

```
sudo ./tweaks.sh -h newhostname -p magicpassword
```

## reset-usb.sh

Needs to be run on the car to reset USB in the event of a failure of the front USB hub, faster than a full reboot of the car - works ~70% of the time YMMV

To ensure that car configuration is correct, please run `tweaks.sh` once after flashing the car.

| -   | File                        | Description                                                                                  |
| --- | --------------------------- | -------------------------------------------------------------------------------------------- |
| 1   | `tweaks.sh`                 | Update the car settings.                                                                     |
| 2   | `dev-stack-dependencies.sh` | Installs the dependencies for a custom DR stack. Script is only required to be run one time. |
| 3   | `dev-stack-build.sh`        | Downloads the packages defined in `ws/.rosinstall` and builds them into the `ws` folder.     |
| 4   | `dev-stack-install.sh`      | Installs the stack built in `ws/install` into `/opt/aws/deepracer/lib`.                      |


## usb-build.*

**Recommendations**:

- Try using USB 3.0 usb stick, the process will take about 30 minutes per stick compared to 1.5hrs with USB 2.0 ones.

**Requirements**:

- The custom AWS DeepRacer Ubuntu ISO image `ubuntu-20.04.1-20.11.13_V1-desktop-amd64.iso` in the same directory (will be downloaded if missing)
- The latest AWS DeepRacer software update package `factory_reset.zip` **unzipped** in the same directory (will be downloaded and unzipped if missing)

Both files can be downloaded from here https://docs.aws.amazon.com/deepracer/latest/developerguide/deepracer-ubuntu-update.html (under the "Prerequisites" heading).

### OSX version

**Requirements**:

- https://unetbootin.github.io/ installed

**Command**:

```
./usb-build.sh -d disk2 -s <WIFI_SSID> -w <WIFI_PASSWORD>
```

**Note:**

- The wifi credentials are used to create `wifi-creds.txt` on the `DEEPRACER` partition, upon rebooting after flashing the car will use this file to connect to wifi
- Tested and working on Intel and Apple based Macs
- Unetbootin currently doesn't work on OSX Ventura [#Issue 337](https://github.com/unetbootin/unetbootin/issues/337)

### Ubuntu version

**Requirements**:

- Requires sudo privileges
- Will add current user to sudoer automatically (with the "no password" option enabled)

**Command**:

```
./usb-build.ubuntu.sh -d sdX -s <WIFI_SSID> -W <WIFI_PASSWORD>
```

**Note:**

- The wifi credentials are used to create `wifi-creds.txt` on the `DEEPRACER` partition, upon rebooting after flashing the car will use this file to connect to wifi
- Tested and working on Ubuntu 20.0.4 (like a DeepRacer car)


## VS.Code Dev Container

Files have been added to the repository (`.devcontainer` and `.vscode`) that allows the normal DR packages to be built on another machine, isolated from what is installed on the OS. Tested on Ubuntu 20.04.

Container includes most relevant ROS2 Foxy packages, as well as the OpenVINO release that is used on the car.

Before opening the container run `docker volume create deepracer-ros-bashhistory` to get your commandline history enabled inside the container.
