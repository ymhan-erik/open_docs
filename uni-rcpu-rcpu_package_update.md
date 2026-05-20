# INFOCUBE RCPU_Z1G 

### 유니테스트에서 패키지 파일을 제공했을때 보드 이미지에 넣는 방법 

### erik@info-cube.co.kr  
### 2026-05-20

## 준비물 & 환경
 1. 리눅스 
 2. SD_Boot_ext4.zip 
 3. unitest_rcpu_software_package_20260402_1.9.1_1.7.3_for_N208.tar 
 4. sd_card 

## 예제 1)
##   A. 보드이미지 : 
        SD_Boot_ext4.zip
            
##   B. 유니테스트 제공 패키지파일 :  
        unitest_rcpu_software_package_20260402_1.9.1_1.7.3_for_N208.tar



## STEP_1 압축 해제 :
       tar -xvf unitest_rcpu_software_package_20191012_1.8.8_1.7.0_new_sync.tar

## STEP_2 압축 해제
        unzip SD_Boot_ext4.zip

## STEP_3 작업경로 이동
        cd SD_Boot_ext4

## STEP_4 임시 작업 폴더     
        mkdir rootfs_tmp


## STEP_5 root 이미지를 임시 작업 폴더에 복사 & 압축해제

        cp -rf ./ZEROM_ROOT.tar ./rootfs_tmp
        cd ./rootfs_tmp
        sudo tar -xpf ZEROM_ROOT.tar
        sudo rm -rf ./ZEROM_ROOT.tar

## STEP_6 기존 파일 삭제    
        sudo rm -rf ./root/unitest_rcpu_software_package


## STEP_7 새 rcpu 패키지 복사
        sudo cp -rfp /home/f47/iWork/1_UNI_Z010_RCPU_V1/260511/unitest_rcpu_software_package ./root

## STEP_8 원본 백업 , 다시 압축하고 마무리
        cd ..
        mv ./ZEROM_ROOT.tar ./ZEROM_ROOT.tar.old
        sudo tar --numeric-owner -cpf ZEROM_ROOT.tar -C ./rootfs_tmp .

## STEP_9 파일권한 root:roor 에서 f47:f47
        sudo chown f47:f47 ZEROM_ROOT.tar


## STEP_10 마무리
        SD_Boot.ext4.zip로 다시 압축 | sdcard에 복사한다.


