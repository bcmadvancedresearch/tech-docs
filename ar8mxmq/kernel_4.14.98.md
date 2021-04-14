Build AR8MXMQ Linux Kernel 4.14.98
===
### 1. Download and install Yocto SDK
[Yocto 4.14.98][toolchain-4.14-sumo]
1.1 Installation 

```shell=
$ wget https://engineering.bcmcom.com/CustomerDL/AR8MXMQ/fsl-imx-xwayland-glibc-x86_64-core-image-minimal-aarch64-toolchain-4.14-sumo.sh
$ sh ./fsl-imx-xwayland-glibc-x86_64-core-image-minimal-aarch64-toolchain-4.14-sumo.sh
```
    
1.1.1 Remove `-wl` in LDFLAGS
    
`sudo sed -i 's/-Wl,//g' /opt/fsl-imx-xwayland/4.14-sumo/environment-setup-aarch64-poky-linux`
    
    
    
1.2 Prepare build environment
    
`. /opt/fsl-imx-xwayland/4.14-sumo/environment-setup-aarch64-poky-linux`
    
    

### 2. Download and build Linux kernel source code
2.1 Download Linux kernel source from Github

```shell=
$ git clone https://github.com/bcmadvancedresearch/linux-imx6.git linux-imx8
$ cd linux-imx8
$ git checkout p9.0.0_2.0.0-bcm
```

2.2 Configure and build

```shell=
$ export INSTALL_MOD_PATH=../my-tmp
$ export KERNEL_SRC=$PWD
$ make linux_ar8mxm_defconfig
$ make -j8
$ make modules_install
```
set `CONFIG_MXC_GPU_VIV=m` to build `galcore.ko` outside kernel tree
first type `make menuconfig`, press `/` and type `mxc_gpu_viv` to search, type `1` and then `m` to set `<M> MXC Vivante GPU support`
```shell=
$ make modules_prepare
```

2.3 Download galcore source and build module separatly

```shell=
$ git clone https://github.com/Freescale/kernel-module-imx-gpu-viv.git
$ cd kernel-module-imx-gpu-viv
$ git checkout upstream/6.2.4.p4.2
$ make -j8
$ make modules_install
```



### 3. Copy files to eMMC/SD
3.1 Copy Image and dtb to 1st FAT partition
```
/
├── bcmtwn-imx8mm-ar8mxmm.dtb
└── Image
```
3.2 Copy driver modules to 2nd partition (RFS)
```
/
└── lib
    └── modules
        └── 4.14.98-ge8ad75e24a7a
            ├── build
            ├── extra
            ├── kernel
            ├── modules.alias
            ├── modules.alias.bin
            ├── modules.builtin
            ├── modules.builtin.bin
            ├── modules.dep
            ├── modules.dep.bin
            ├── modules.devname
            ├── modules.order
            ├── modules.softdep
            ├── modules.symbols
            ├── modules.symbols.bin
            └── source

```

3.3 Export eMMC/MicroSD FAT partition for file transfer (Optional)
power on AR8MXMQ board and hit `enter` to u-boot command prompt, use `ums` command to export eMMC(0:1) or MicroSD (1:1)
```shell=
Hit any key to stop autoboot:  0
u-boot=> ums 0 mmc 0:1
UMS: LUN 0, dev 0, hwpart 0, sector 0x4000, count 0x20000
```
Complete file transfer, hit `Ctrl+C` to exit and then `reset` the board
```
CTRL+C - Operation aborted
u-boot=> reset
resetting ...
```

[toolchain-4.14-sumo]: https://engineering.bcmcom.com/CustomerDL/AR8MXMQ/fsl-imx-xwayland-glibc-x86_64-core-image-minimal-aarch64-toolchain-4.14-sumo.sh