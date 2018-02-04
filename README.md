Compiling the hello world module for Raspbian:
Figuring this out took a lot of trial and error. The problem is that Raspbian is not released with the typical header files, and for some reason gcc is not able to do the compilation (something about cross-compilation on a 64 bit platform).

http://lostindetails.com/blog/post/Compiling-a-kernel-module-for-the-raspberry-pi-2
AND
https://www.raspberrypi.org/documentation/linux/kernel/building.md

These two sources were basically combined to make this all work.

1. Use the following command to download the toolchain to the home folder:
git clone https://github.com/raspberrypi/tools ~/tools

2. If you are on a 64-bit host system, you should use:
echo PATH=\$PATH:~/tools/arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian-x64/bin >> ~/.bashrc
source ~/.bashrc

3. From home:
git clone https://github.com/raspberrypi/linux

4. export CCPREFIX=/home/INSERT YOUR USER NAME HERE/tools/arm-bcm2708/arm-bcm2708-linux-gnueabi/bin/arm-bcm2708-linux-gnueabi-

5. Set the environment variable KERNEL_SRC to point to the kernel source: export KERNEL_SRC=/home/INSERT YOUR USER NAME HERE/linux

6. On the raspberry pi execute FIRMWARE_HASH=$(zgrep "* firmware as of" /usr/share/doc/raspberrypi-bootloader/changelog.Debian.gz | head -1 | awk '{ print $5 }') to retrieve the firmware hash.

7. execute KERNEL_HASH=$(wget https://raw.github.com/raspberrypi/firmware/$FIRMWARE_HASH/extra/git_hash -O -) to retrieve the kernel hash.

8. On RPI run echo $KERNEL_HASH and copy this value

9. On ubuntu run from linux folder: git checkout INSERT_KERNEL_HASH_HERE

10. execute make mrproper in the linux directory

11. copy /proc/config.gz from the raspberry pi to the linux directory.
e.g. scp pi@ipaddress:/proc/config.gz ./ and unpack it: zcat config.gz > .config

12. make ARCH=arm CROSS_COMPILE=${CCPREFIX} oldconfig

13. Sit back and have some tea:
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- zImage modules dtbs

View the makefile for how to actually compile the .ko file, and note that the make is of the format:

make ARCH=arm CROSS_COMPILE=$(PREFIX) -C /home/[USER]/linux M=$(SRC) modules

Finally, simply scp the hello.ko to the RPI and perform insmod
