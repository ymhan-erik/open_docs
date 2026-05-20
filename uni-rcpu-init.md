# INFOCUBE RCPU_Z1G 복구  
### Program Flash, Recovery root-file system  
### erik@info-cube.co.kr  
### 2019-10-17  

# STEP_1

##### INFOCUBE에서 제공한 파일을 SD카드에 복사하고 보드에 삽입한다.  
##### RCPU의 부트점퍼(J18)를 제거하고 전원을 올린다.(SD부트)  
##### 아래 메시지가 나오면, STEP_2로 넘어간다.  
##### 아래 메시지가 나오지 않으면, 수 차례 시도해보고,  
##### 나오지 않을 경우 H/W 손상에 의한 고장처리.  

expect {

    U-Boot 2013.10-00002-ge7773ea-dirty (Oct 19 2015 - 19:22:18)

    I2C:   ready
    Memory: ECC disabled
    DRAM:  1 GiB
    MMC:   zynq_sdhci: 0, zynq_sdhci: 1
    SF: Detected S25FL256S_64K with page size 256 Bytes, erase size 64 KiB, total 32 MiB
    In:    serial
    Out:   serial
    Err:   serial
    Net:   Gem.e000b000
    Hit any key to stop autoboot:  0
    zynq-uboot>
}



# STEP_2
##### 반드시 아래 라인을 한 줄씩 친다.

    setenv kernel_size 500000
    setenv devicetree_size 4000
    setenv bootargs 'console=ttyPS1,115200n8 root=/dev/mmcblk0p1 rootfstype=ext4 rootdelay=3 earlyprintk;'
    setenv qspiboot 'sf probe 0 0 0;sf read 0x3000000 0xA00000 ${kernel_size};sf read 0x2A00000 0xF00000 ${devicetree_size};bootm 0x3000000 - 0x2A00000;'
    setenv sdboot 'setenv bootargs console=ttyPS1,115200n8 root=/dev/ram0 rootfstype=ext4 rootdelay=3 earlyprintk;fatload mmc 0 0x3000000 uImage_sd;fatload mmc 0 0x2A00000 devicetree_sd.dtb;fatload mmc 0 0x2000000 ${ramdisk_image};bootm 0x3000000 0x2000000 0x2A00000;'
    saveenv   
   
##### 아래 메시지가 나오면, STEP_3으로 넘어간다.  
##### 아래 메시지가 나오지 않으면, 수 차례 다시 시도해보고,  
##### 나오지 않을 경우 H/W 손상에 의한 고장처리.  

expect {

        Saving Environment to SPI Flash...
        SF: Detected S25FL256S_64K with page size 256 Bytes, erase size 64 KiB, total 32 MiB
        Erasing SPI flash...Writing to SPI flash...done
        zynq-uboot>        
}


# STEP_3
##### 반드시 아래 라인을 한 줄씩 친다.
    sf probe 0 0 0;
    fatload mmc 0 1000000 ${kernel_image}; && sf update 1000000 0xA00000 ${kernel_size};
    fatload mmc 0 1000000 ${devicetree_image}; && sf update 1000000 0xF00000 ${devicetree_size};
    fatload mmc 0 0x8000 BOOT.BIN; && sf update 0x8000 0 0x300000
    run sdboot

##### 부팅이 완료되면 expect-1 또는 expect-2가 나온다.
#####
##### expect-1이 보이면 자동으로 emmc를 업데이트하고, 재부팅을한다.
##### 부팅이 완료되고, expect-2 메세지가 보이면, STEP_4로 넘어간다.
#####
##### 메시지가 나오지 않으면 수 차례 다시 시도해보고,
##### 나오지 않을 경우 H/W 손상에 의한 고장처리.
#####
##### 일일이 명령어를 치는 방법은 STEP 5를 보면된다.

expect-1 {

    [unitest_rcpu /] eMMC Init~!
    / #
}
##### --- 자동으로 작업을 하고 재부팅한다 ---

expect-2 {

    [unitest_rcpu /] eMMC Ready~!
    / #
}
##### expect-2 메세지가 보이면  
    rm ./mnt/emmc/.emmc_ready
    rm ./mnt/emmc/.qspi_ready
    /etc/init.d/S60update start

##### 파티션 잡는 작업을 자동으로 수행한다. 완료되면 스텝 4로 간다


# STEP_4

##### 부트점퍼(J18)를 1-2번에 연결한다. (플래시부트)
##### 리셋 또는 POR을 한다.
##### 부팅완료하고 터미널에 아래 메시지가 출력되면, 작업완료

##### 파워를 온-오프-온 한다.

expect {

    S50...Starting dropbear sshd: OK
    S60...Starting NFS mount...
    S70...Starting vsftpd: OK
    S90...Start RCPU Manager Program...
    Buildroot 2013.05
    linux login: root (automatic login)

    [S/W Debug] Wiat FM Reconfig Start ----------------------------
    [S/W Debug] Parsed FM version informaion file.
    [S/W Debug] Read FM Version : 1
    [S/W Debug] File FM Version : 257
    [S/W Debug] Read FM Type : 4353
    [S/W Debug] File FM Type : 4353
    [S/W Debug] ----------------------------------------------
    [S/W Debug] Read FM Version : 1
    [S/W Debug] File FM Version : 257
    [S/W Debug] Read FM Type : 4353
    [S/W Debug] File FM Type : 4353
    [root@linux ~]# [S/W Debug] ----------------------------------------------
    [S/W Debug] Read FM Version : 1
    [S/W Debug] File FM Version : 257
    [S/W Debug] Read FM Type : 4353
    [S/W Debug] File FM Type : 4353
    [S/W Debug] ----------------------------------------------
    .
    .
    .
    .
    .
    .
}



# STEP_5    
#### emmc가 있는지 확인한다.

ls /dev/mmcblk*

expect{

    /dev/mmcblk0       /dev/mmcblk0boot1  /dev/mmcblk0rpmb   /dev/mmcblk1p1
    /dev/mmcblk0boot0  /dev/mmcblk0p1     /dev/mmcblk1       /dev/mmcblk1p2
}

#### 반드시 아래 라인을 한 줄씩 친다. << EMMC format 작업

    umount /dev/mmcblk0p1
    fdisk /dev/mmcblk0 < /mnt/sdcard/fdisk_input
    mkfs.ext4 -F /dev/mmcblk0p1
    sync

    mkdir -p /mnt/emmc
    mount -t ext4 /dev/mmcblk0p1 /mnt/emmc
    sync

    tar -xf /mnt/sdcard/ZEROM_ROOT.tar -C /mnt/emmc/
    rm /mnt/emmc/etc/macaddr_store
    sync

    touch /mnt/emmc/.qspi_ready
    touch /mnt/emmc/.emmc_ready
    sync

    umount /dev/mmcblk0p1
    sync

    e2fsck -p -f -v /dev/mmcblk0p1
    sync

    reboot

#### 파워를 온오프 한다.
#### 스텝 4로 간다.










