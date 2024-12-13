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
sudo mkdir /usr/src
cd /usr/src
```

Look for your kernel in here:
[Index of /pub/linux/kernel/v5.x/](https://mirrors.edge.kernel.org/pub/linux/kernel/v5.x/)

Download the kernel with the same name as yours:
```sh
wget https://mirrors.edge.kernel.org/pub/linux/kernel/v5.x/linux-<your_kernel_name>.tar.gz
```

Now we look for a real-time patch here:
[Index of /pub/linux/kernel/projects/rt/]([https://mirrors.edge.kernel.org/pub/linux/kernel/v5.x/](https://cdn.kernel.org/pub/linux/kernel/projects/rt/5.15/older/))

And we will aslo download it:
```sh
wget https://cdn.kernel.org/pub/linux/kernel/projects/rt/5.15/older/patch-<your_kernel_name>.patch.gz
```

Now we will extract both:

```sh
tar -xzf linux-<your_kernel_name>.tar.gz
```
and:
```sh
gunzip patch-<your_kernel_name>.patch.gz
```

Enter inside your linux folder:
```sh
cd linux-<your_kernel_name>
```
And we will patch our kernel:
```sh
patch -p1 < ../patch-<your_kernel_name>.patch
```

Enable all Ubuntu configurations:
```sh
yes '' | make oldconfig
```

Enable expert mode:
```sh
make nconfig
```

A new window will open with the kernel configuration. Using the "arrow keys", and "enter" to navigate, and the "space" to select and unselect submodules, configure the following options:
```txt
General Setup --->
        [*]  Embedded System
``` 

Then we need to enable rt_preempt in the kernel:

```sh
make menuconfig
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
sudo make -j$(nproc) deb-pkg && \
sudo make modules_install -j$(nproc) deb-pkg && \
sudo make install -j$(nproc) deb-pkg && \
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

Relaunch again the WSL terminal.

Go back to the PowerShell terminal and redo the steps to attach the device:
```
usbipd wsl list
usbipd wsl attach --busid <desired_bus_id> --distribution <name_of_your_WLS_distribution>
```

Go back to the WSL terminal and check if the device is propperly recognised by typing
```
lsusb
ls /dev/video*
```

And we are done and all set to go!
