Access first FAT partition on AR8MXMQ
===
### Export eMMC/MicroSD FAT partition for file transfer 
Power on AR8MXMQ board and hit `enter` to u-boot command prompt, use `ums` command to export eMMC(0:1) or MicroSD (1:1)

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