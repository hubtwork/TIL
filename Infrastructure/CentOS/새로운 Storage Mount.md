### 새로운 Storage Mount

> ✔ **OS : CentOS 7.3**



#### Background

- DB 서버를 운영중에 데이터의 양이 많아 서버 내부 디스크 용량이 급격히 증가
- 이에 추가 스토리지를 마운팅해 데이터를 저장할 소요가 생김



#### Disk Mount

- 스토리지 상태 확인

~~~shell
# 현재 /dev/xvda1~3 디스크들이 추가되어있는 상태이며 /dev/xvdb 디스크 파티셔닝 필요
$ fdisk -l
---
    Device Boot      Start         End      Blocks   Id  System
/dev/xvda1            2048        4095        1024   83  Linux
/dev/xvda2            4096     4198399     2097152   82  Linux swap / Solaris
/dev/xvda3   *     4198400   104857599    50329600   83  Linux

Disk /dev/xvdb: 2147.5 GB, 2147483648000 bytes, 4194304000 sectors
Units = sectors of 1 * 512 = 512 bytes
---
~~~

- 스토리지 파티셔닝

~~~shell
# fdisk 명령을 통해 해당 디스크에 대해 파티션 진행
$ fdisk /dev/xvdb
...
command(m for help): n # n 입력 : 새로운 파티셔닝 진행
command action
	e		extended
	p		primary partition(1-4)
p	# p 선택 ( 보통 Primary partitioning 진행 후, 그래도 부족할 때 Extended partitioning 진행 )
Partition number(1-4): 1
First cylinder (2048-4194304000, default 2048): 2048 # Cylinder 범위 내에서 시작할 Cylinder 지정
Last cylinder ... (2048-4194304000, default 4194304000): 4194304000 # Cylinder 범위 내에서 종료할 Cylinder 지정
command(m for help): w # w 입력 : 파티셔닝 저장 ( 진행 )
The partition table has been altered!
~~~

- 스토리지 포맷

~~~bash
$ mkfs.xfs /dev/xvdb1	# xfs 파티션 ( 고용량 파티션 지원 , EXT : 16TB 까지만 지원 ) 
...
~~~

- 스토리지 마운트 <span style="color:red">**[ 여기까지만 진행시 서버를 재부팅하면 마운팅이 해제됨 ]**</span>

~~~shell
$ mkdir /mnt/v1	# 마운트할 디렉토리 생성
$ mount /dev/xvdb1 /mnt/v1	# 해당 디렉토리에 파티션 된 디스크 마운트
~~~

- 마운트 정보 유지 설정 ( /etc/fstab )

~~~shell
$ vim /etc/fstab
...
# 디스크 이름		마운트 위치	 파티션 포맷			설정			덤프	파일점검옵션(FSCK)
/dev/xvdb1		/mnt/v1				xfs			defaults		0			0
~~~

>fstab 상세 설정
>
>#### 설정
>
>​	**default** : auto , exec , nouser, suid
>
>​	**auto** : 부팅시 자동으로 마운트해줌
>
>​	**exec** : 실행파일 실행 허용
>
>​	**user** : 전체 사용자 마운트 허용
>
>​	**nouser** : root 계정만 마운트 허용
>
>​	**suid** : uid, gid 설정 허용
>
>​	**rw** : 읽기 / 쓰기 모두 허용
>
>​	**ro** : 읽기만 허용
>
>#### 덤프 ( **0** : 불가능 / **1** : 가능 )
>
>#### 파일 점검 옵션 (FSCK)
>
>설정값 1의 파일시스템 점검 후 2 의 파일시스템 점검
>
>​	**0** : 부팅시 fsck 실행 X
>
>​	**1** : 루트파일 시스템 fsck 
>
>​	**2** : 루트파일 시스템 제외 fsck



