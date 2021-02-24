### Git Pull Request

> ✔ **OS : macOS Big Sur 11.1**
>
> ✔ **Git version 2.24.3**



#### Background

- 협업을 진행하며 형상관리 툴로 Git 을 널리들 사용함
- 각자 개인의 로컬 저장소에서 Master 브랜치로 임의로 **Merge** 하고 **Push** 하는 것은 위험함
- 관리자에 의해 **다양한 Contributor 간의 저장소 운영**이 원활하게 진행될 수 있어야 함 
- Contributor 들은 로컬 저장소에서 작업한 후 **Pull Request** 를 요청해 패치 내역이 반영될 수 있게 함



#### Git Pull Request

- 일반적인 팀 단위 Work

~~~shell
# 별도의 Branch 생성 후 작업 진행
git branch hotfix/network
...
# 원격 저장소에 반영
git add .
git commit -m "커밋 메세지"
git push origin hotfix/network
# github 에서 Create Pull Request 요청
# -----> 관리자가 수락시 Master 브랜치에 Merge Pull Request
# -----> 관리자가 거절시 해당 Pull Request 는 취소됨
~~~

- Open Source Contribution

~~~shell
# github 에서 해당 레포지토리 Fork
# -----> 내 저장소에 레포지토리가 생성됨
git clone "Fork 레포지토리"
# 각 Feature 별로 Branch 작업 등 작업 진행
# 그 후 변경사항들 모두 로컬의 Master Branch 에 Merge
# -----> Fork 레포지토리의 Master Branch 에 반영
...
git push origin master
# github 의 Fork 레포지토리 ( 내 저장소 ) 에서 Create Pull Request 요청
# -----> 관리자가 수락시 Upstream Master 브랜치에 Merge Pull Request
# -----> 관리자가 거절시 해당 Pull Request 는 취소됨
~~~