# WSL_realtime_kernel
The following repository contains basic instructions on how to install a real-time kernel in WSL.
Based on the oficial documentation: [Building a real-time Linux kernel](https://docs.ros.org/en/humble/Tutorials/Miscellaneous/Building-Realtime-rt_preempt-kernel-for-ROS-2.html)

First step is to download Linux Kernel Drivers.
To do so we will first check what Kernel are we using by opening our WLS window, an typing the following command:
```
uname -a
```

You can now check if said kernel has an existing repo in the official Windows repository:
[WSL2-Linux-Kernel](https://github.com/microsoft/WSL2-Linux-Kernel/releases)

And we will follow by assigning the version of the kernel to a varialbe
```
VERSION=<kernel_version_number>
# example
# VERSION=5.15.90.1
```


We will install some dependencies before updating the kernel:
```sh
sudo apt update && sudo apt upgrade -y && \
sudo apt install -y  \
        autoconf \
        bc \
        bison \
        build-essential \
        cmake \
        dwarves \
        flex \
        git \
        iputils-ping \
        libelf-dev \
        libgtk2.0-dev \
        libncurses-dev \
        libssl-dev \
        libtool \
        libudev-dev \
        net-tools \
        python3-pip \
        unzip \
        v4l-utils \
        zip
```

We will have to enable the deb sources to install the following packages:
```sh
sudo nano /etc/apt/sources.list
```
And erase the "#" of all the lines with a deb source (should be all of them):

Then install with:
```sh
sudo apt-get update
sudo apt-get build-dep linux
sudo apt-get install libncurses-dev flex bison openssl libssl-dev dkms libelf-dev libudev-dev libpci-dev libiberty-dev autoconf fakeroot
```

Now we create a direcotry to store all the files that we will download and install (if it doesn't exist already)
```
sudo mkdir ~/Downloads
cd ~/Downloads
```
We look for a real-time patch here:
[Index of /pub/linux/kernel/projects/rt/](https://cdn.kernel.org/pub/linux/kernel/projects/rt/5.15/older/)

And we will aslo download it:
```sh
sudo wget https://cdn.kernel.org/pub/linux/kernel/projects/rt/5.15/older/patch-<your_kernel_name>.patch.gz
```

And extract it :
```sh
gunzip patch-<your_kernel_name>.patch.gz
```

Look for your WSL kernel in here:
[WSL2-Linux-Kernel - Releases](https://github.com/microsoft/WSL2-Linux-Kernel/releases)

Download the kernel with the same name as yours:
```sh
sudo git clone -b linux-msft-wsl-${VERSION} https://github.com/microsoft/WSL2-Linux-Kernel.git ${VERSION}-microsoft-standard
```

Patch our kernel:
```sh
sudo patch -p1 < ../patch-<your_kernel_name>.patch
```

Copy the original config:
```sh
sudo cp /proc/config.gz config.gz && \
sudo gunzip config.gz && \
sudo mv config .config
```

Enable all Ubuntu configurations:
```sh
yes '' | sudo make oldconfig
```

Enable expert mode:
```sh
sudo make nconfig
```

A new window will open with the kernel configuration. Using the "arrow keys", and "enter" to navigate, and the "space" to select and unselect submodules, configure the following options:
```txt
General Setup --->
        [*]  Embedded System
``` 

Then we need to enable rt_preempt in the kernel:

```sh
sudo make menuconfig
```

A new window will open with the kernel configuration. Using the "arrow keys", and "enter" to navigate, and the "space" to select and unselect submodules, configure the following options:
```
# Enable CONFIG_PREEMPT_RT
 -> General Setup
  -> Preemption Model (Fully Preemptible Kernel (Real-Time))
   (X) Fully Preemptible Kernel (Real-Time)

# Enable CONFIG_HIGH_RES_TIMERS
 -> General setup
  -> Timers subsystem
   [*] High Resolution Timer Support

# Enable CONFIG_NO_HZ_FULL
 -> General setup
  -> Timers subsystem
   -> Timer tick handling (Full dynticks system (tickless))
    (X) Full dynticks system (tickless)

# Set CONFIG_HZ_1000 (note: this is no longer in the General Setup menu, go back twice)
 -> Processor type and features
  -> Timer frequency (1000 HZ)
   (X) 1000 HZ

# Set CPU_FREQ_DEFAULT_GOV_PERFORMANCE [=y]
 ->  Power management and ACPI options
  -> CPU Frequency scaling
   -> CPU Frequency scaling (CPU_FREQ [=y])
    -> Default CPUFreq governor (<choice> [=y])
     (X) performance
```

And we proceed to build the new kernel and copy it to replace the old one
```
sudo make -j$(nproc) && \
sudo make modules_install -j$(nproc) && \
sudo make install -j$(nproc) && \
sudo mkdir /mnt/c/Sources/ && \
sudo cp -rf vmlinux /mnt/c/Sources/
```

Now we well reboot the WLS sesion by typing:
```
exit
```
or also by simply closing the window.

And we will go back to our PowerShell terminal and type:
```
wsl --shutdown
```

Now we will go to our user folder by pressing "Win+R" and typing
```
%userprofile%
```

We will create an empty text file and rename it to:
```
.wslconfig
```
in case the file didn't exist yet.
And we will paste the direction of the newly created kernel and save it:
```
[wsl2]
kernel=C:\\Sources\\vmlinux
```

Relaunch again the WSL terminal and check if the kernel has been successfully installed:

If you get a kernel with a "rt" in it, you have succesffuly updated your kernel. Example:
```
Linux MY-COMPUTER 5.15.167.4-rt80-microsoft-standard-WSL2+ #1 SMP PREEMPT_RT Fri Dec 13 11:25:30 CET 2024 x86_64 x86_64 x86_64 GNU/Linux
```

Congratulations!
