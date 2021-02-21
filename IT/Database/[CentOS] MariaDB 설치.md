### CentOS MariaDB 설치

---

> ✔ **테스트 환경 : CentOS 7.3**
>
> ✔ **Dependency : Yum**



#### 1. Yum Repo 설정

~~~bash
$ vim /etc/yum.repos.d/maria.repo
~~~

>[mariadb]
>name = MariaDB
>baseurl = http://yum.mariadb.org/10.1/centos7-amd64
>gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
>gpgcheck=1

- 따로 원하는 버전을 설치하고 싶으면 baseUrl 의 버전명을 변경
- OS 별로 설정 맞춰야 함

#### 2. Yum install 및 설치 확인

~~~bash
$ yum install MariaDB -y	# MariaDB 설치
$ rpm -qa | grep MariaDB	# MariaDB 설치되었는지 확인
# - compat, client, common, server 총 4개가 확인되어야함
$ mariadb --version	# MariaDB 버전 확인
~~~

#### 3. MariaDB 실행, 상태확인

~~~bash
$ systemctl enable mysql	# MariaDB 자동 재시작 설정
$ systemctl start mysql	# MariaDB 실행
$ systemctl status mysql	# MariaDB 상태확인
~~~

#### 4. 로그인 및 비밀번호 설정

~~~bash
$ mysql -u root	# 초기엔 비밀번호 없음 
# in MariaDB
MariaDB[None] > use mysql;
MariaDB[mysql] > update user set password=password('비밀번호') where user='root';
MariaDB[mysql] > flush privileges;
~~~



