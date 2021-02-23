FreeRTOS SDK v2.9.0 for AR8MXMQ
===
### 1. Download and install toochain for ARM Cortex-M

Download prebuilt toolchain from `developer.arm.com`

```shell=
$ wget https://developer.arm.com/-/media/Files/downloads/gnu-rm/7-2017q4/gcc-arm-none-eabi-7-2017-q4-major-linux.tar.bz2
$ tar xjf gcc-arm-none-eabi-7-2017-q4-major-linux.tar.bz2 -C /opt
```

Install `cmake` version >= 3.0.x

```shell=
$ sudo apt-get install cmake
$ cmake --version
cmake version 3.5.1

CMake suite maintained and supported by Kitware (kitware.com/cmake).
```

Add a new system environment variable for ARMGCC_DIR

```shell=
$ export ARMGCC_DIR=/opt/gcc-arm-none-eabi-7-2017-q4-major
$ export PATH=$PATH:/opt/gcc-arm-none-eabi-7-2017-q4-major/bin
```

Update Linux kernel with M4 support
_[this commit](https://github.com/bcmadvancedresearch/linux-imx6/commit/67d9e9b53282431e54cce984c8ea1cde4be32220)_ on Github [branch](https://github.com/bcmadvancedresearch/linux-imx6/tree/p9.0.0_2.0.0-bcm) `p9.0.0_2.0.0-bcm`

In U-boot, load correct dtb file to test MCU
```shell=
u-boot=> setenv fdt_file bcmtwn-imx8mq-ar8mxm-m4.dtb
```

### 2. Download SDK v2.9 from GitHub

```shell=
$ git clone https://github.com/bcmadvancedresearch/freertos-bcm.git
$ cd freertos-bcm
$ git checkout ar8mxmq_2.9.0
```

### 3. Buil and Run example
#### 3.1 Build and Run `hello_world` example

```shell=
$ cd board/evkmimx8mq/demo_apps/hello_world/armgcc
$ ./build_release.sh
$ tree release
release/
├── hello_world.bin
└── hello_world.elf

0 directories, 2 files
```

Copy the binary `hello_world.bin` to 1st FAT partition (Please refert to this [guide](https://hackmd.io/@frodolai/HJI7Z0pbu))

Load and run MCU manually from eMMC

```shell=
u-boot=> setenv m4image hello_world.bin
u-boot=> setenv m4loadaddr 0x7E0000
u-boot=> fatload mmc 0 $m4loadaddr $m4image
u-boot=> bootaux $m4loadaddr
```

Replace `fatload mmc 1 $m4loadaddr $m4image` if using MicroSD

#### 3.2 Build and Run RPMsg demo `rpmsg_lite_str_echo_rtos`

```shell=
$ cd boards/evkmimx8mq/multicore_examples/rpmsg_lite_str_echo_rtos/armgcc
$ ./build_release.sh
$ tree release
release
├── rpmsg_lite_str_echo_rtos.bin
└── rpmsg_lite_str_echo_rtos_imxcm4.elf

0 directories, 2 files
```

Copy the binary `rpmsg_lite_str_echo_rtos.bin` to 1st FAT partition 

Load and run MCU firmware manually from eMMC, then boot to Linux

```shell=
u-boot=> setenv m4image rpmsg_lite_str_echo_rtos.bin
u-boot=> setenv m4loadaddr 0x7E0000
u-boot=> fatload mmc 0 $m4loadaddr $m4image
u-boot=> bootaux $m4loadaddr
u-boot=> boot
```

Boot to Linux and check RPMsg driver

```shell=
$ dmesg | grep rpmsg
[    0.429711] imx rpmsg driver is registered.
[    0.460938] virtio_rpmsg_bus virtio0: rpmsg host is online
```

Then, load the RPMsg module to communicate between MPU and MCU.

```shell=
$ sudo modprobe imx_rpmsg_tty
$ sudo su
# echo "this is a test" > /dev/ttyRPMSG30
```

See the data received from DB9 (CN28)

```shell=
RPMSG String Echo FreeRTOS RTOS API Demo...

Nameservice sent, ready for incoming messages...
Get Message From Master Side : "hello world!" [len : 12]
Get Message From Master Side : "this is a test" [len : 14]
Get New Line From Master Side
```

### 3.3 Build and Run RPMsg demo `rpmsg_lite_pingpong_rtos`

```shell=
$ cd boards/evkmimx8mq/multicore_examples/rpmsg_lite_pingpong_rtos/linux_remote/armgcc
$ ./build_release.sh
$ tree release
release
├── rpmsg_lite_pingpong_rtos_linux_remote.bin
└── rpmsg_lite_pingpong_rtos_linux_remote.elf

0 directories, 2 files
```

Copy the binary `rpmsg_lite_pingpong_rtos_linux_remote.bin` to 1st FAT partition 

Load and run MCU firmware manually from eMMC, then boot to Linux

```shell=
u-boot=> setenv m4image rpmsg_lite_pingpong_rtos_linux_remote.bin
u-boot=> setenv m4loadaddr 0x7E0000
u-boot=> fatload mmc 0 $m4loadaddr $m4image
u-boot=> bootaux $m4loadaddr
u-boot=> boot
```

Boot to Linux and load the RPMsg module to communicate between MPU and MCU.

```shell=
$ sudo modprobe imx_rpmsg_pingpong
```

See the data received from DB9 (CN28)

```shell=
RPMSG Ping-Pong FreeRTOS RTOS API Demo...
RPMSG Share Base Addr is 0xb8000000
Link is up!
Nameservice announce sent.
Waiting for ping...
Sending pong...
Waiting for ping...
Sending pong...
Waiting for ping...
Sending pong...
.....
Waiting for ping...
Sending pong...
Waiting for ping...
Sending pong...
Ping pong done, deinitializing...
Looping forever...
```

### 4. U-boot load MCU firmware automatically (optional)

TBD