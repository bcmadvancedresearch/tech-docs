Buld AR8MXMQ U-boot imx_v2018.03_4.14.98_2.2.0-bcm
===


[![hackmd-github-sync-badge](https://hackmd.io/d1Cwode-TrK4s3bsN81vWA/badge)](https://hackmd.io/d1Cwode-TrK4s3bsN81vWA)
1. Download and install Yocto SDK
    * [Yocto 4.14.98](https://engineering.bcmcom.com/CustomerDL/AR8MXMQ/fsl-imx-xwayland-glibc-x86_64-core-image-minimal-aarch64-toolchain-4.14-sumo.sh)
    * [Yocto 5.4.47](https://engineering.bcmcom.com/CustomerDL/AR8MXMQ/fsl-imx-xwayland-glibc-x86_64-core-image-minimal-aarch64-imx8mnevk-toolchain-5.4-zeus.sh)
    
    1.1 Installation (pick up one or both)
    ```shell=
    $ wget https://engineering.bcmcom.com/CustomerDL/AR8MXMQ/fsl-imx-xwayland-glibc-x86_64-core-image-minimal-aarch64-toolchain-4.14-sumo.sh
    $ sh fsl-imx-xwayland-glibc-x86_64-core-image-minimal-aarch64-toolchain-4.14-sumo.sh
    
    $ wget https://engineering.bcmcom.com/CustomerDL/AR8MXMQ/fsl-imx-xwayland-glibc-x86_64-core-image-minimal-aarch64-imx8mnevk-toolchain-5.4-zeus.sh
    $ sh fsl-imx-xwayland-glibc-x86_64-core-image-minimal-aarch64-imx8mnevk-toolchain-5.4-zeus.sh
    ```
    
    1.2 Prepare build environment
    
    ` . /opt/fsl-imx-xwayland/4.14-sumo/environment-setup-aarch64-poky-linux`
    
    or
    
    `. /opt/fsl-imx-xwayland/5.4-zeus/environment-setup-aarch64-poky-linux`

2. Download [imx-mkimage](https://engineering.bcmcom.com/CustomerDL/AR8MXMQ/imx-mkimage.tgz) for u-boot v2018.03
    
    ```shell=
    $ wget https://engineering.bcmcom.com/CustomerDL/AR8MXMQ/imx-mkimage.tgz
    $ tar zxvf imx-mkimage.tgz
    ```
    
   
3. Download and Build u-boot source code
    
    3.1 Download u-boot source from Github
    ```shell=
    $ git clone https://github.com/bcmadvancedresearch/u-boot-imx.git
    $ cd u-boot-imx
    $ git checkout imx_v2018.03_4.14.98_2.2.0-bcm
    ```
    3.2 Configure and build
    |RAM    |Linux   |Android|
    | :---:   | :---:    | :---:   |
    |2GB    |imx8mq_ar8mxm_defconfig    | imx8mq_ar8mxm_android_defconfig |
    |4GB    |imx8mq_ar8mxm_4gb_defconfig    | imx8mq_ar8mxm_4gb_android_defconfig |
    
    ```shell=
    $ make mrproper
    $ make imx8mq_ar8mxm_defconfig
    $ make -j4
    $ cp arch/arm/dts/bcmtwn-imx8mq-ar8mxm.dtb $(imx-mkimage-dir)/iMX8M/
    $ cp u-boot-nodtb.bin $(imx-mkimage-dir)/iMX8M/
    $ cp spl/u-boot-spl.bin $(imx-mkimage-dir)/iMX8M/
    
    $ cd $(imx-mkimage-dir)
    $ make clean; make SOC=iMX8M flash_hdmi_spl_uboot_ar8mxm
    ```
    get `flash.bin` in `$(imx-mkimage-dir)/iMX8M/`
4. Burn the flash.bin to MicroSD card/eMMC offset 33KB (dd for removalbe storage and uuu for USB type-c downlaoding)

    4.1 Use dd
    
    `sudo dd if=flash.bin of=/dev/sd[x] bs=1024 seek=33`

    4.2 Use [uuu](https://github.com/NXPmicro/mfgtools/releases) (preferred)
    
    Move jumper from JP5 to JP4, enter downloader mode
    
    `uuu -b sd flash.bin`
    
    or
    
    `uuu -b emmc flash.bin`
    
    Move jumper back to JP5
    
    4.3 Boot Mode (JP6)
    
    with jumper = 1, without jumper = 0
    |MicroSD    |eMMC    |
    | :---:    | :---:    |
    |1100    |0010    |
    
    