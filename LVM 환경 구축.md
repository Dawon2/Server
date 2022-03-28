# LVM 환경 구축

## 구축 목적
: 디스크를 가상으로 분할하여 효율적으로 디스크공간과 용량을 조절하여 사용하기 위해

## LVM 환경 구축

- 먼저 서버에 추가 디스크를 붙여줌 ( +10G )
```
------------------------------------------------------------------------------------
*버전등의 문제로 설치가 안될 경우
# cd /etc/yum.repos.d/
# mv CentOS-Base.repo.org CentOS-Base.repo
# yum clean all
------------------------------------------------------------------------------------

# yum -y install lvm*

# fdisk -l
( 추가한 디스크 잘 들어왔는지 확인 )

# fdisk /dev/xvdb
( 들어온 해당 디스크 설정 )
------------------------------------------------------------------------------------
1.
n -> p
( 파티션 추가 )

2.
p
( 파티션 테이블확인 , 생성한 파티션 System 확인 - Linux로 되어있는걸 Linux LVM 으로 바꿔줘야함 )

3.
t -> L -> 8e
( 파티션 시스템 변경 -> 리스트 확인 -> Linux LVM 설정 )

4.
p -> w
( 파티션 시스템이 정상적으로 변경되었나 확인 후 저장 종료 )
------------------------------------------------------------------------------------
```

### LVM TEST Work
```
★ 목표 : 1. 10G 디스크를 LVM으로 5G, 5G 두개로 나눠서 사용환경 구축
          2. 10G 디스크를 추가로 올려서 LVM 설정 후, lv를 총 5G, 10G, 5G 3개로 구성 후 사용환경 구축


< 목표 1번 >

- LVM 설정 -

# fdisk -l
( LVM 설정 잘 되어있는지 확인 )

# pvcreate /dev/xvdb1
# pvs
( 물리적 디스크에 LVM을 사용할 수 있도록 데이터 구조를 생성함 -> pv )

# vgcreate vg1 /dev/xvdb1
# vgs
( 만든 pv 로 가상 디스크 설정 -> vg )

# lvcreate -L 5G vg1
# lvcreate -l 100%FREE vg1
( 10G의 vg1에서 5G를 나누고 다음 5G 나누려면 공간이 모자르다 나와서 나머지 100% 공간 나누기 )
( 가상 디스크 vg를 파티션처럼 나눠서 사용 -> lv )
# lvs

-> 이제 /dev에 vg1 폴더와 그 안에 lv가 생성됨!


- 파일시스템 탑재 및 마운트 설정 -

# mkfs.xfs /dev/vg1/lvol0 /dev/vg1/lvol1
( xfs 파일시스템으로 생성한 2개의 lv에 파일시스템 탑재 )

# mkdir /data1 /data2
( 마운트할 디렉토리 생성 )

# mount /dev/vg1/lvol0 /data1
# mount /dev/vg1/lvol1 /data2
( lv 하나하나 폴더에 마운트 )

# vi /etc/fstab
( 시스템을 재부팅하면 마운트 정보가 사라지는것을 방지하기 위해 fstab에 마운트정보 저장 )
------------------------------------------------------------------------------------
#
# /etc/fstab
# Created by anaconda on Thu Jun 25 05:13:09 2015
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
UUID=d53d7098-1b1d-4451-963c-62c8d7f46ba7 /                       ext4    defaults        1 1
UUID=8a6c631c-c60a-4370-b015-dadc9c7b4e15 /boot                   ext4    defaults        1 2
UUID=1df276ee-1448-4274-b738-32c7bc5cdb8f swap                    swap    defaults        0 0
/dev/vg1/lvol0                  /data1          xfs     defaults        0 0
/dev/vg1/lvol1                  /data2          xfs     defaults        0 0

순서대로 - 디바이스   마운트위치  파일시스템 타입   옵션   백업동작  파일시스템 체크 순서
------------------------------------------------------------------------------------

★ 목표 1번 달성 ★


< 목표 2번 >
- 10G 디스크 추가로 올린 후 진행

# fdisk -l
# fdisk /dev/xvdc
( 위와 동일하게 진행 )
------------------------------------------------------------------------------------
1.
n -> p
( 파티션 추가 )

2.
p
( 파티션 테이블확인 , 생성한 파티션 System 확인 - Linux로 되어있는걸 Linux LVM 으로 바꿔줘야함 )

3.
t -> L -> 8e
( 파티션 시스템 변경 -> 리스트 확인 -> Linux LVM 설정 )

4.
p -> w
( 파티션 시스템이 정상적으로 변경되었나 확인 후 저장 종료 )
------------------------------------------------------------------------------------

- LVM 설정 -
# fdisk -l

# pvcreate /dev/xvdc1
# pvs

# vgextend vg1 /dev/xvdc1
# vgs
( vgextend 명령어로 기존 vg1에 추가 )

# lvextend -L 10G /dev/vg1/lvol1
( 기존의 5G lv를 10G로 늘리면서 5G 추가 )
# lvcreate -l 100%FREE vg1
( 나머지 5G로 새로운 lv 생성 )


- 파일시스템 탑재 및 마운트 설정 -
# mkfs.xfs /dev/vg1/lvol2
( 새로만든 3번째 lv 파일시스템 탑재 )

# mkdir /data3
( 마운트할 디렉토리 새로 생성 )

# mount /dev/vg1/lvol2 /data3

# vi /etc/fstab
------------------------------------------------------------------------------------
#
# /etc/fstab
# Created by anaconda on Thu Jun 25 05:13:09 2015
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
UUID=d53d7098-1b1d-4451-963c-62c8d7f46ba7 /                       ext4    defaults        1 1
UUID=8a6c631c-c60a-4370-b015-dadc9c7b4e15 /boot                   ext4    defaults        1 2
UUID=1df276ee-1448-4274-b738-32c7bc5cdb8f swap                    swap    defaults        0 0
/dev/vg1/lvol0                  /data1          xfs     defaults        0 0
/dev/vg1/lvol1                  /data2          xfs     defaults        0 0
/dev/vg1/lvol2                  /data3          xfs     defaults        0 0
------------------------------------------------------------------------------------


* xfs_growfs /data2
( 파일시스템에 lv 용량이 적용이 안됬을때 사용! )
```

## TEST
```
1.
# fdisk -l

2.
# df -Th

3.
# mount

4.
# umount -> mount

5.
# rmdir data3 -> mkdir data3

등의 여러가지 방식으로 테스트 확인!
```
***
**★ 정상적으로 작동된다면 LVM 환경 구축 완료 ! ★**
***

## 참고
- https://5log.tistory.com/18
