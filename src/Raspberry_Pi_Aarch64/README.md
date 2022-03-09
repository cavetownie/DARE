The image comes shipped with GEF plugin and GDB. The following Qemu command can be used to run the image:
```
qemu-system-aarch64 -M virt -m 1024 -smp 4 -cpu cortex-a53 \
-kernel vmlinuz -initrd initrd.img \
-drive file=rasp.img,if=none,id=drive0,cache=writeback \
-device virtio-blk,drive=drive0,bootindex=0 -append 'root=/dev/vda2 noresume rw' \
-device virtio-net-pci,netdev=mynet -netdev user,id=mynet,hostfwd=tcp::2222-:22 \
-no-reboot -nographic
```

This opens port 22 for SSH and SCP from your host machine. Tested on `5.4.0-104-generic #118-Ubuntu`

The image file itself is too large to be hosted in this folder, see RELEASES.
