Todo: 
Add install script, which takes care of everything from scratch.

# To-Long-Didnt-Read
The image comes shipped with GEF plugin and GDB. The following Qemu command can be used to run the image:
```
qemu-system-aarch64 -M virt -m 1024 -smp 4 -cpu cortex-a53 \
-kernel vmlinuz -initrd initrd.img \
-drive file=rasp.img,if=none,id=drive0,cache=writeback \
-device virtio-blk,drive=drive0,bootindex=0 -append 'root=/dev/vda2 noresume rw' \
-device virtio-net-pci,netdev=mynet -netdev user,id=mynet,hostfwd=tcp::2222-:22 \
-no-reboot -nographic
```
You may have to run it as a one-liner.

This runs the image and opens port 22 for SSH and SCP from your host machine. Tested on `5.4.0-104-generic #118-Ubuntu`

To find the image and the required files look in RELEASES.


# Manual creation:
1. Install the raspberry pi image (see source at bottom), unzip, and give it a name - We'll call it rasp.img

2. Run `sudo fdisk -l rasp.img`
```
Disk rasp.img: 1.8 GiB, 1153433600 bytes, 2252800 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xac8dad98

Device     Boot  Start     End Sectors  Size Id Type
rasp.img1         2048  614399  612352  299M  c W95 FAT32 (LBA)
rasp.img2       614400 2252799 1638400  800M 83 Linux
```

The scond partition starts at 614400 and this multiplied with 512 is 314572800. This is where we'll mount 
this image, to extract the linux kernel and the initrd image. 

3. Make a directory in the mnt directory, eg. `sudo mkdir -p /mnt/raspimg`

4. Mount the image `sudo mount -o offset=314572800 rasp.img /mnt/raspimg/`

5. Now copy the kernelfile called vmlinuz and the initrd.img file into your current directory

6. Run the following command:
```
qemu-system-aarch64 -M virt -m 1024 -smp 4 -cpu cortex-a53 -kernel vmlinuz -initrd initrd.img -drive file=rasp.img,if=none,id=drive0,cache=writeback -device virtio-blk,drive=drive0,bootindex=0 -append 'root=/dev/vda2 noresume rw' -no-reboot -nographic
``` 

7. Now you should be able to log-in with the default username and password
User: root
Password: raspberry

8. Get internet by changing `/etc/network/interfaces.d/enp0s1` and enp0s2, in the same directory. These files need to contain:
```
auto enp0s1

iface enp0s1 inet dhcp
	pre-up iptables-restore < /etc/iptables/rules.v4
	pre-up ip6tables-restore < /etc/iptables/rules.v6

---
and the same but with enp0s2, for the other file
```

9. Now boot again, and you should be able to do `ping 8.8.8.8`

10. I just needed this, so that I could do arm things for ctf's:) That's it!

[ To compile stuff for arm use aarch64-linux-gnu-g++ ] <br>
[ To transfer files ones can do base64, or wget ] <br>
[ To transfer files one can also use scp `scp test.txt root@localhost:/root/`]<br>
<br>To use the SCP add:
```
-device virtio-net-pci,netdev=mynet -netdev user,id=mynet,hostfwd=tcp::2222-:22
```
Right before your `-no-reboot` in your qemu script

Source: https://www.youtube.com/watch?v=Y-FUvi1z1aU
