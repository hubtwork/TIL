### Centos7 use iptables

> ✔ **OS : CentOS 7.3**



#### Background

- CentOS 7 의 경우 **패킷필터링** 에 `iptables` 외에 `firewalld` 를 기본 방화벽으로 사용함 
- 이전 버전의 서버에서 운영하던 설정을 그대로 마이그레이션하고자 `iptables` 로 변경하고자 함



#### Remove firewalld

- 이전버전의 CentOS 와 같이 `iptables` 의 온전한 사용을 위해 `firewalld` 를 종료시키고 제거함

~~~shell
# firewalld 가 active 상태라면 종료
$ systemctl stop firewalld
# firewalld 상태 확인
$ systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; vendor preset: enabled)
   Active: inactive (dead)
     Docs: man:firewalld(1)
# firewalld 서비스 symlink 변경
$ systemctl mask firewalld
Created symlink from /etc/systemd/system/firewalld.service to /dev/null.
# firewalld 시스템 유닛 제거 여부 확인
$ systemctl list-units | grep firewalld
~~~



#### Install iptables

- `iptables` 설치 진행 및 

~~~shell
$ yum -y install iptables-services
$ systemctl enable iptables
Created symlink from /etc/systemd/system/basic.target.wants/iptables.service to /usr/lib/systemd/system/iptables.service.
# iptables Rule-sets 설정
$ vi /etc/sysconfig/iptables
# iptables 서비스 시작 및 확인 
$ systemctl start iptables
$ systemctl status iptables
● iptables.service - IPv4 firewall with iptables
   Loaded: loaded (/usr/lib/systemd/system/iptables.service; enabled; vendor preset: disabled)
   Active: active (exited) since 수 2021-03-24 11:31:40 KST; 26s ago
  Process: 31836 ExecStop=/usr/libexec/iptables/iptables.init stop (code=exited, status=0/SUCCESS)
  Process: 31903 ExecStart=/usr/libexec/iptables/iptables.init start (code=exited, status=0/SUCCESS)
 Main PID: 31903 (code=exited, status=0/SUCCESS)
   CGroup: /system.slice/iptables.service
# iptables 시스템 유닛 제거 여부 확인
$ systemctl list-units | grep iptables
iptables.service         loaded active exited IPv4 firewall with iptables
~~~

