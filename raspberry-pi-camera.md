# raspberry pi camera module notes

The new raspberry pi camera setup wants to use libcamera instead of v4l2. I'd rather use v4l2.
To switch back to v4l2:

Edit /boot/firmware/config.txt:

Disable `camera_auto_detect=1`

Add `start_x=1\ngpu_mem=256`

e.g.
```
sed -re 's/^camera_auto_detect=1/#camera_auto_detect=1\n# pms back to v4l\nstart_x=1\ngpu_mem=256\n# end pms\n/' -i /boot/firmware/config.txt
```

Driver is

```
[    9.891813] bcm2835_v4l2-0: V4L2 device registered as video0 - stills mode > 1280x720

bcm2835_v4l2           49152  0
bcm2835_mmal_vchiq     36864  3 bcm2835_codec,bcm2835_v4l2,bcm2835_isp
videobuf2_vmalloc      20480  1 bcm2835_v4l2
videobuf2_v4l2         32768  5 bcm2835_codec,bcm2835_v4l2,rpivid_hevc,v4l2_mem2mem,bcm2835_isp
videodev              299008  6 bcm2835_codec,videobuf2_v4l2,bcm2835_v4l2,rpivid_hevc,v4l2_mem2mem,bcm2835_isp
videobuf2_common       81920  9 bcm2835_codec,videobuf2_vmalloc,videobuf2_dma_contig,videobuf2_v4l2,bcm2835_v4l2,rpivid_hevc,v4l2_mem2mem,videobuf2_memops,bcm2835_isp
```


bcm2835-mmal-vchiq


ubuntu@ubuntu:~$ modprobe --show-depends bcm2835_v4l2
insmod /lib/modules/6.2.0-1004-raspi/kernel/drivers/media/mc/mc.ko
insmod /lib/modules/6.2.0-1004-raspi/kernel/drivers/media/common/videobuf2/videobuf2-common.ko
insmod /lib/modules/6.2.0-1004-raspi/kernel/drivers/media/v4l2-core/videodev.ko
insmod /lib/modules/6.2.0-1004-raspi/kernel/drivers/media/common/videobuf2/videobuf2-v4l2.ko
insmod /lib/modules/6.2.0-1004-raspi/kernel/drivers/media/common/videobuf2/videobuf2-memops.ko
insmod /lib/modules/6.2.0-1004-raspi/kernel/drivers/media/common/videobuf2/videobuf2-vmalloc.ko
insmod /lib/modules/6.2.0-1004-raspi/kernel/drivers/staging/vc04_services/vc-sm-cma/vc-sm-cma.ko
insmod /lib/modules/6.2.0-1004-raspi/kernel/drivers/staging/vc04_services/vchiq-mmal/bcm2835-mmal-vchiq.ko
insmod /lib/modules/6.2.0-1004-raspi/kernel/drivers/staging/vc04_services/bcm2835-camera/bcm2835-v4l2.ko



things you don't need:
  215  sudo modprobe -r bcm2835-codec
  219  sudo modprobe -r bcm2835-isp
  225  sudo modprobe -r rpivid_hevc




need:
VIDEO_BCM2835

CONFIG_VIDEO_BCM2835_UNICAM=m
CONFIG_VIDEO_BCM2835=m



bcm2835_mmal_vchiq 36864 0 - Live 0xffffd59b99fe1000 (C)
vchiq 327680 1 bcm2835_mmal_vchiq, Live 0xffffd59b9a2a8000 (C)
uvcvideo 102400 0 - Live 0xffffd59b99ff1000
uvc 12288 1 uvcvideo, Live 0xffffd59b99fe7000
videobuf2_v4l2 28672 1 uvcvideo, Live 0xffffd59b99fbd000
videobuf2_vmalloc 16384 1 uvcvideo, Live 0xffffd59b99fd0000
videobuf2_memops 16384 1 videobuf2_vmalloc, Live 0xffffd59b99fc4000
videobuf2_common 57344 4 uvcvideo,videobuf2_v4l2,videobuf2_vmalloc,videobuf2_memops, Live 0xffffd59b99fa4000
brcmfmac_wcc 12288 0 - Live 0xffffd59b99fb5000
brcmfmac 266240 1 brcmfmac_wcc, Live 0xffffd59b99f5f000
brcmutil 16384 1 brcmfmac, Live 0xffffd59b99f58000



bcm2835_v4l2 40960 0 - Live 0xffffd59b9a00a000 (C)
bcm2835_mmal_vchiq 36864 1 bcm2835_v4l2, Live 0xffffd59b99fe1000 (C)
vchiq 327680 1 bcm2835_mmal_vchiq, Live 0xffffd59b9a2a8000 (C)
uvcvideo 102400 0 - Live 0xffffd59b99ff1000
uvc 12288 1 uvcvideo, Live 0xffffd59b99fe7000
videobuf2_v4l2 28672 2 bcm2835_v4l2,uvcvideo, Live 0xffffd59b99fbd000
videobuf2_vmalloc 16384 2 bcm2835_v4l2,uvcvideo, Live 0xffffd59b99fd0000
videobuf2_memops 16384 1 videobuf2_vmalloc, Live 0xffffd59b99fc4000
videobuf2_common 57344 5 bcm2835_v4l2,uvcvideo,videobuf2_v4l2,videobuf2_vmalloc,videobuf2_memops, Live 0xffffd59b99fa4000
brcmfmac_wcc 12288 0 - Live 0xffffd59b99fb5000
brcmfmac 266240 1 brcmfmac_wcc, Live 0xffffd59b99f5f000
brcmutil 16384 1 brcmfmac, Live 0xffffd59b99f58000



[video4linux2,v4l2 @ 0x6b501c0] Raw       :     yuv420p :     Planar YUV 4:2:0 : {32-3280, 2}x{32-2464, 2}
[video4linux2,v4l2 @ 0x6b501c0] Raw       :     yuyv422 :           YUYV 4:2:2 : {32-3280, 2}x{32-2464, 2}
[video4linux2,v4l2 @ 0x6b501c0] Raw       :       rgb24 :     24-bit RGB 8-8-8 : {32-3280, 2}x{32-2464, 2}
[video4linux2,v4l2 @ 0x6b501c0] Compressed:       mjpeg :            JFIF JPEG : {32-3280, 2}x{32-2464, 2}
[video4linux2,v4l2 @ 0x6b501c0] Compressed:        h264 :                H.264 : {32-3280, 2}x{32-2464, 2}
[video4linux2,v4l2 @ 0x6b501c0] Compressed:       mjpeg :          Motion-JPEG : {32-3280, 2}x{32-2464, 2}
[video4linux2,v4l2 @ 0x6b501c0] Raw       : Unsupported :           YVYU 4:2:2 : {32-3280, 2}x{32-2464, 2}
[video4linux2,v4l2 @ 0x6b501c0] Raw       : Unsupported :           VYUY 4:2:2 : {32-3280, 2}x{32-2464, 2}
[video4linux2,v4l2 @ 0x6b501c0] Raw       :     uyvy422 :           UYVY 4:2:2 : {32-3280, 2}x{32-2464, 2}
[video4linux2,v4l2 @ 0x6b501c0] Raw       :        nv12 :           Y/UV 4:2:0 : {32-3280, 2}x{32-2464, 2}
[video4linux2,v4l2 @ 0x6b501c0] Raw       :       bgr24 :     24-bit BGR 8-8-8 : {32-3280, 2}x{32-2464, 2}
[video4linux2,v4l2 @ 0x6b501c0] Raw       :     yuv420p :     Planar YVU 4:2:0 : {32-3280, 2}x{32-2464, 2}
[video4linux2,v4l2 @ 0x6b501c0] Raw       : Unsupported :           Y/VU 4:2:0 : {32-3280, 2}x{32-2464, 2}
[video4linux2,v4l2 @ 0x6b501c0] Raw       :        bgr0 : 32-bit BGRA/X 8-8-8-8 : {32-3280, 2}x{32-2464, 2}
