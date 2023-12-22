# linux-poseidon
A series of patches against Vanilla Linux Stable to establish the same or similar functionality to "linux-neptune" on Steam Deck devices.

### Supported Upstream Kernel Version: v6.6.8

# Before you begin:
#### Having at least 5 GB of accessible, usable, space is mandatory for this project.
#### Even then, I would say you should double that just to be safe.

# What you will need:
1. The "git" program.
2. The "patch" program.
3. The ability to compile the Linux kernel.
    1. You will need bison, flex, libssl-dev (OpenSSL), gcc, make, and a slew  of other things I can't remember at the moment.
    2. You will need a config file for the kernel. You can use the one provided here, edit it, or make your own by running 'make menuconfig' in the kernel source code directory AFTER you apply the patches from this repo you want.
        - Personally, rather than rolling your own entirely, I think you will have a better experience pulling a working kernel config from Steam OS to use as a base.  In Desktop Mode, open a terminal and copy "/proc/config.gz" to your home directory.  From there, uncompress it and it is now useable in kernel building.
4. The linux-firmware-neptune tarball from Valve Software. I provide one version here, but if you want to pull newer firmware, go to [the Jupiter repository in Valve's ArchLinux mirror](https://steamdeck-packages.steamos.cloud/archlinux-mirror/sources/jupiter-3.5/) and download your preferred version. Fair warning: Valve, apparently, does bulk staggered releases like Ubuntu, and looks like they create new repositories in their mirror for each release.  This means the link provided here may be out of date.  Just go one directory up on the site and you should see the list of repositories, including the current Jupiter release.

5. A copy of the Linux Kernel codebase to patch.
    - Use Git to clone the stable codebase and manually set the local checkout to reflect the supported stable version:
      ```
      git clone https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git linux-stable-steamdeck-hacks
      cd linux-stable-steamdeck-hacks
      git checkout v6.6.8
      ```
The Linux kernel codebase is huge. This will take a while. Go get yourself some coffee or a Coke.

# Ok, I have all of that (and my drink). What now?
1. Untar the linux_firmware_neptune tarball and set it aside for now.
2. Now, make sure you can successfully run a vanilla kernel compile.
    1. Navigate into the codebase directory and run
    ```
    make -j '<max_threads_on_your_computer_minus_2>'
    make -j '<max_threads_on_your_computer_minus_2>' modules
    ```
    being sure to replace ```<max_threads_on_your_computer_minus_2>``` with the appropriate value for your computer. I always leave 2 threads available to reduce the chances that the UI locks up when running the compile, which I have found to be a problem on some lower-end computers.
   
    2. If Step 1 fails, it is probably because you are missing some required software. Did you install Bison? Flex? What about "libssl-dev"? I've been caught up in this dance before as well. The only generic advice I can give is to pay attention to the output and check to see if anywhere you see that some file or program or other could not be found. Then, use your system's package manager to find out what packages provide the files in question and install them. After, start over from Step 1. Rinse and repeat until the build completes successfully.

3. Next, run
```
make clean
```
4. Still in the codebase directory, run
```
patch -ruN -p1 -d . < '<path_to_the_patch_file_you_want>'
```
being sure to replace ```<path_to_the_patch_file_you_want>``` with the path to the patch from this project you want to apply.

5. Last, run the commands in Step 1 again.

That's it. You're finished with the kernel compile. Keep in mind that preparing the kernel for ACTUAL USE is another story.

# Ok, How do I use the kernel I just built?
1. You need a Linux distro install put on a storage medium with which you can boot your Steam Deck.
   - Your install medium needs to be fairly large. At least 32 GB.
   - I use an SD card for my storage.
   - For my Linux distro, I went with Gentoo because nothing in Gentoo cares what kernel version you are running as long as it is recent by a few years.

3. You need to mount this install on to the computer you used to build the kernel.
   - Mount everything. Rootfs, var, boot, efi. Everything. BUT, do so while ensuring everything gets mounted to its proper place.
       - Mount the rootfs before anything else.
           - Make a place to mount it. I use "/mnt/sdcard".
     
       - Mount the other partitions into their proper places.
           - Say you mounted the rootfs to "/mnt/sdcard". Mount "boot" to "/mnt/sdcard/boot", "efi" to "/mnt/sdcard/boot/efi", "var" to "/mnt/sdcard/var", etc.
           - When in doubt, read "/etc/fstab" to see where  your install mounts these partitions at boot.  Mount them to the rootfs in the same say, except under wherever you mounted the install's rootfs and not the rootfs on your build host.
            
4. Become "root" and run the following:
   ```
   cp -r <path_to_patched_codebase> <your_install_rootfs>/usr/src/linux-stable-steamdeck-hacks
   cp -r <path_to_untarballed_linux_neptune_firmware> <your_install_rootfs>/usr/src/linux_firmware_neptune
   ```
   This copies the entire patched codebase, and the firmware, into the install's rootfs, which seems weird, but bear with me.
   
6. Now, while still "root", run the following:
   ```
   cd <your_install_rootfs>
   mount -t proc /proc proc
   mount --rbind /dev dev
   mount --make-rslave dev
   mount --rbind /sys sys
   mount --make-rslave sys
   mount --rbind /run run
   mount --make-rslave run
   cp --dereference /etc/resolv.conf etc
   ```
   This sets up mounts for some configuration filesystems that the Linux install will need for next few steps as well as copies the "resolv.conf" file into the install's rootfs so that we can have functioning DNS resolution when we …
   
7. Chroot into the install's rootfs! Heh. Stay "root" and run ```chroot .```.  You should now see a whole new shell prompt. Congrats! In this prompt your computer will (mostly) behave as if you had booted into the Steam Deck install on your build host.
   
8. Now, we navigate to the codebase directory we copied over in Step 4.
   ```
   cd /usr/src/linux-stable-steamdeck-hacks
   ```
   
9. Once there, run:
    ```
    make install
    make modules_install
    ```
    These commands will install the kernel, system map files, and the config file into the "/boot" directory of the install, and the kernel modules into the module directory of the install (usually "/lib/modules").
   
10. Unfortunately, it is at this point where I have to stop providing specific instructions because what comes next depends on what exactly you wanted to do with this new kernel build. It also is highly dependent on your choice of distro. I'll continue, but I can only give vague instructions on what to do next. It is up to you to discover the specific commands you need to run, and in what order, for your specific Linux distro and purpose.

# WORK_IN_PROGRESS