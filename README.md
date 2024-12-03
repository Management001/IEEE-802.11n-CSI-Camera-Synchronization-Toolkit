# IEEE-802.11n-CSI-Camera-Synchronization-Toolkit
* This repository is a part of an RF-based mobility detection and tracking project.
* The goal of our research is to develop technology to recognize and track objects beyond walls using solely WiFi signals in the IEEE standards.
  
The purpose of this repository is to provide support for building the CSI-enabled **TP-Link AC1750** Wi-Fi drivers for Intel Wireless Link **Intel 5300 NIC** adapters on Linux distributions with **Unbuntu 14.01** versions. At this point, this code has been tested on Ubuntu **14.01**, **16.04**, and **18.04**.

The code presented here comprises a modified version of the Linux kernel, such as the firmware command, and error lines of the original baseline. The modifications were made by examining the code provided by [dhalperi/linux-80211n-csitool](https://github.com/dhalperi/linux-80211n-csitool) and adapting them to more recent Linux kernel versions **- iwlwifi** and **wlp1s0** modules. The building and installation instructions were taken from [the original Linux 802.11n CSI Tool website](https://dhalperi.github.io/linux-80211n-csitool/) and adapted accordingly. Moreover, I referred to the synchronization method between CSI and Camera from [CSI-Tool-Camera-Shooting](https://github.com/qiyinghua/CSI-Tool-Camera-Shooting).

Additionally, there have been initial toolkit issues discussed among mobility detection researchers who use WiFi signals and Channel State Information (CSI). To this end, we provide a calculation function **newly_csi_analyzer.m** of amplitude and phase, and signal processing from the acquired CSI samples. Herein, I referred to novel approaches [DensePose From WiFi](https://arxiv.org/abs/2301.00250) and [Can WiFi Estimate Person Pose?](https://arxiv.org/abs/1904.00277).

# Project Members
#### [Youngwoo Oh (M.S. student, Project leader from May 2023 to Feb. 2024)](https://ohyoungwoo.com/)
- Integrated two sensing values including RF signals and captured video from the router and Camera by developing these Linux toolkit codes.
- Signal processing, and project software and hardware configuration.

#### [Jungtae Kang (Undergraduate student, Project follower)](https://kangjeongtae.com/)
- Supporting the generation of CSI samples and Camera images.
- Writing [Winter Conference on Korea Information and Communications Society (KICS) conference](https://conf.kics.or.kr/) papers named *Collection and analysis of CSI in IEEE 802.11n wireless LAN environment for WiFi signal-based human mobility detection* and *Design of WiFi signal-based Multi-modal KNN deep learning approaches for improving anomaly object detection and classification methods*.
- He will follow up on this *Novel multi-modal approaches-based object detection/tracking/recognition methods* in his future research.
  
#### Islam Helemy (Ph.D. student, Project member)
- Responsible for the development of the Multi-modal AI and the pre-processing to generate training data pairs (CSI samples-captured images).
- He will receive a *project leader* position on this future project after Feb. 2024.

#### Iftikhar Ahmad (Ph.D. student, Project member)
- Focused on developing the Teacher network in the multi-modal AI model.

#### Manal Mosharaf (M.S. student, Project member)
- Engaged in developing the Student network in the multi-modal AI model.

# Toolkit Demonstration
Will be upload

# 1. Installation instructions of integrated CSI toolkit
## (1). Kernel version:
Before proceeding further, you need to check the version of your kernel. It should be **4.15**, otherwise, the commands below won't work. The following command will print that information:
```ruby
uname -r
```

## (2). First download the essential packages:
```ruby
sudo apt-get install build-essential linux-headers-$(uname -r) git-core
```
```ruby
sudo add-apt-repository ppa:ubuntu-toolchain-r/test
```
```ruby
sudo apt-get update
```
If you don't need to install this gcc version, skip a below line :)
```ruby
sudo apt-get install gcc-8 g++-8
```
```ruby
sudo apt-get install libgtk2.0-dev
```
```ruby
sudo apt-get install pkg-config 
```
```ruby
sudo apt-get install cmake
```
```ruby
sudo apt update
```

## (3). Check the GCC version:
```ruby
ls -l /usr/bin/gcc /usr/bin/g++
```
```ruby
sudo rm /usr/bin/gcc
```
```ruby
sudo rm /usr/bin/g++
```
```ruby
sudo ln -s /usr/bin/gcc-8 /usr/bin/gcc
```
```ruby
sudo ln -s /usr/bin/g++-8 /usr/bin/g++
```
```ruby
ls -l /usr/bin/gcc /usr/bin/g++
```
```ruby
sudo apt update
```

## (4). Unzip OpenCV for utlizing the USB camera:
```ruby
cd IEEE-802.11n-CSI-Camera-Synchronization-Toolkit/camera_tool
```
```ruby
unzip opencv-2.4.13.6.zip
```
```ruby
cd opencv-2.4.13.6/install
```

## (5). Compile and install OpenCV:
```ruby
cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local ..
```
```ruby
cmake ..
```
```ruby
make
```
```ruby
sudo make install
```

## (6). Configuration OpenCV:
```ruby
sudo gedit /etc/ld.so.conf.d/opencv.conf
```
Then add the /usr/local/lib command to the file.
```
  /usr/local/lib
```
```ruby
sudo ldconfig
```
Then add the following command to the end of the file:
```ruby
sudo gedit /etc/bash.bashrc
```
```
  PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/usr/local/lib/pkgconfig
  export PKG_CONFIG_PATH
```
```ruby
sudo reboot
```

## (7). If a kernel change is needed (optional)
Installing a specific kernel version
```ruby
sudo apt-get install linux-image-4.15.0-20-generic
sudo apt-get install linux-headers-4.15.0-20-generic
sudo apt-get install linux-modules-4.15.0-20-generic
sudo apt-get install linux-modules-extra-4.15.0-20-generic
```

GRUB set
```ruby
sudo gedit /etc/default/grub
```
```
  ... Omitted parts ...

  ## Setting the name of the installed Linux kernel version
  GRUB_DEFAULT='Advanced options for Ubuntu>Ubuntu, with Linux 4.15.0-20-generic'

  ## With the settings below, a countdown screen will appear for 3 seconds.
  ## During the countdown, pressing the 'ESC', 'F4', or 'SHIFT' key will bring up the GRUB boot menu.
  ## At this point, you can choose a kernel version other than the default kernel version.
  GRUB_TIMEOUT_STYLE=countdown
  GRUB_TIMEOUT=3

  ... Omitted parts ...
```

GRUB update
```ruby
sudo update-grub
```
```
  Sourcing file `/etc/default/grub'
  Generating grub configuration file ...
  Found linux image: /boot/vmlinuz-4.15.0-20-generic

  ... Omitted parts ...
```
```ruby
reboot
```

Press [ESC], [SHIFT], or [F4] to select the desired Linux kernel version, such as "Linux 4.15.0-20-generic."
```ruby
sudo apt-get install gcc make linux-headers-$(uname -r) git-core
```
```ruby
sudo apt update
```

## (8). Build and install the modified wireless LAN driver:
```ruby
git clone https://github.com/spanev/linux-80211n-csitool.git
```
```ruby
cd linux-80211n-csitool
```
```ruby
CSITOOL_KERNEL_TAG=csitool-$(uname -r | cut -d . -f 1-2)
```
```ruby
git checkout ${CSITOOL_KERNEL_TAG}
```
```ruby
make -j `nproc` -C /lib/modules/$(uname -r)/build M=$(pwd)/drivers/net/wireless/intel/iwlwifi modules
```
```ruby
sudo make -C /lib/modules/$(uname -r)/build M=$(pwd)/drivers/net/wireless/intel/iwlwifi INSTALL_MOD_DIR=updates modules_install
```
```ruby
sudo depmod
```
```ruby
cd ..
```

## (9). Install the Modified firmware:
```ruby
git clone https://github.com/dhalperi/linux-80211n-csitool-supplementary.git
```
```ruby
for file in /lib/firmware/iwlwifi-5000-*.ucode; do sudo mv $file $file.orig; done
```
```ruby
sudo cp IEEE-802.11n-CSI-Camera-Synchronization-Toolkit/supplementary/firmware/iwlwifi-5000-2.ucode.sigcomm2010 /lib/firmware/
```
```ruby
sudo ln -s iwlwifi-5000-2.ucode.sigcomm2010 /lib/firmware/iwlwifi-5000-2.ucode
```

## (10). Build the userspace logging tool:
Replace the "camera.cpp" and "Makefile" files in the path /CSI-Camera-Synchronization-Toolkit/supplementary/netlink
```ruby
make -C IEEE-802.11n-CSI-Camera-Synchronization-Toolkit/supplementary/netlink
```

## (11). Compile user-application:
```ruby
cd IEEE-802.11n-CSI-Camera-Synchronization-Toolkit/supplementary/netlink
```
```ruby
sudo gedit /etc/ld.so.conf.d/opencv.conf
```
Then add the /usr/local/lib command to the file.
```
  /usr/local/lib
```
```ruby
make
```
Check if CSI data can be received from WiFi (refer to Guide 2-1).

## (12). Install v4l utils and vlc
Check your camera device
```ruby
ls -ltr /dev/video*
```
```ruby
sudo apt-get install v4l-utils -y
```
```ruby
v4l2-ctl --list-devices
```
```ruby
sudo apt install vlc
```
Open the installed vlc media player and select your camera device ex)'/dev/vidio0' in 'Capture Device'. Press the play button to check if the camera is operating normally.

# 2. Guideline Operation of integrated CSI toolkit
## (1). setting of the WiFi router and wireless LAN card:
The below comments (1) only need to be performed once after turning on the computer.
```ruby
iw dev wlp1s0 link
```
```ruby
sudo modprobe -r iwlwifi mac80211
```
```ruby
sudo modprobe iwlwifi connector_log=0x1
```
The below comment returns the MAC address, such as XX:XX:XX:XX:XX:XX.
```ruby
sudo ls /sys/kernel/debug/ieee80211/phy0/netdev:wlp1s0/stations/
```
Check the supported modulation, and transmission rate table from the connected WiFi
```ruby
sudo cat /sys/kernel/debug/ieee80211/phy0/netdev:wlp1s0/stations/XX:XX:XX:XX:XX:XX/rate_scale_table
```
Set the transmit rate from the checked table. In our case, we set it as 0x4007 (16-QAM, 1/2).
```ruby
echo 0x4007 | sudo tee /sys/kernel/debug/ieee80211/phy0/netdev:wlp1s0/stations/XX:XX:XX:XX:XX:XX/rate_scale_table
```
```ruby
echo 0x4007 | sudo tee /sys/kernel/debug/ieee80211/phy0/iwlwifi/iwldvm/debug/bcast_tx_rate
```
```ruby
echo 0x4007 | sudo tee /sys/kernel/debug/ieee80211/phy0/iwlwifi/iwldvm/debug/monitor_tx_rate
```

## (2). Starting this toolkit program (utilized three kernel terminals):
```ruby
sudo dhclient wlp1s0
```
comment under kernel terminal, 

where the camera parameters constructed [*Camera ID*] [*Save Interval*] [*Auto Exit*]

[*Camera ID*]: This parameter controls which camera to use when the computer has multiple cameras. When set to 0, the program will use the first camera. When set to 1, the program will use the second camera.

[*Save Interval*]: This parameter controls the speed at which images are saved. When set to 0, the program will save each frame of the camera. When set to 1, the program will save an image every other frame.

[*Auto Exit*]: This parameter controls whether the program automatically exits when CSI collects stops. When set to 0, This program will always run. When set to 1, This program will automatically exit when no CSI is acquired within 1 second.


Open another kernel terminal, where ping testing is a way to encourage the collection of more CSI samples:
```ruby
ping 192.xxx.xxx.xxx -i 0.3
```

Open the first Linux kernel to execute 'log_to_file':
```ruby
cd IEEE-802.11n-CSI-Camera-Synchronization-Toolkit/supplementary/netlink
sudo ./log_to_file test.dat
```

Open the second Linux kernel and write the below command:
```ruby
cd IEEE-802.11n-CSI-Camera-Synchronization-Toolkit/supplementary/netlink
./camera 0 1 0
```

