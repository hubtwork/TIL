### Git Commit Message 수정하기

> ✔ **OS : macOS Big Sur 11.1**
>
> ✔ **Git version 2.24.3**



#### Background

- 작업한 내용에 대한 커밋 메세지를 잘못 기입하거나, 오탈자 등을 고치고 싶을 때



#### 1. Commit 내역을 아직 Remote 저장소에 Push 하지 않았을 경우

- 단일 커밋 메세지와 최근 여러 개의 커밋 메세지를 보는 두가지 방식이 존재

~~~bash
git log								# 커밋 메세지 및 깃 상태 확인 
# 가장 최근 커밋 메세지 수정하기
git commit --amend		
# 가장 최근 N 개의 커밋 메세지 수정하기
git rebase -i HEAD~[n]
~~~

- 커밋 메세지 조회 후 아래와 같이 수정, **저장소에 Push** 할 것

~~~bash
# 아래와 같이 커밋 메세지가 조회됨
-------
 1 pick 7dfc2e5 master : Rebuilding Whole Project to Multi Module
 2 pick 4fm132q master : Temp
 ...
-------
# 원하는 커밋 메세지에 대해 'pick' 을 'reward' 로 변경, 내용 변경 후 !wq 로 저장
-------
 1 pick 7dfc2e5 master : Rebuilding Whole Project to Multi Module
 2 reword 4fm132q master : changed Commit
 ...
-------
# 해당 브랜치로 Remote 저장소에 Push
git push "remote" "branch"
~~~



#### 2. Commit 내역을 이미 Remote 저장소에 Push 한 경우

- 이미 Remote 저장소에 Push 되어 있는 경우, Force Push 를 통해 강제로 푸시하는 방법밖에 없음
- <span style="color:red">**Side - Effect !** </span> 커밋 메세지가 변경, Force Push 된 경우 해당 커밋 로그를 가진 Collaborator 들은 이를 가져와 수동 수정해주어야 일관성이 보장됨 ( 되도록 하지 말 것 )

>**In GitHub Docs** (https://docs.github.com/en/github/committing-changes-to-your-project/changing-a-commit-message)
>
>**We strongly discourage force pushing**, since this changes the history of your repository. If you force push, people who have already cloned your repository will have to manually fix their local history. For more information, see "Recovering from upstream rebase" in the Git manual.

~~~bash
# 1번과 같은 Flow 를 통해 커밋 메세지를 로컬 환경에서 수정
git push -f "remote" "branch"
~~~

