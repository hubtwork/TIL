### Git Branch

> ✔ **OS : macOS Big Sur 11.1**
>
> ✔ **Git version 2.24.3**



#### Background

- 협업을 진행하며 형상관리 툴로 Git 을 널리들 사용함
- 각자 담당하는 모듈 단위 혹은 기능 단위로 개발을 진행 후 필요에 따라 **작은 Work 의 단위로 검수 및 전체 코드의 패치**를 진행
- 각 Work 마다 **Branch** 를 만들어 



#### Git Branch Commands

- Git 에서 Branch 는 말 그대로 소스코드의 관리 입장에서 **새로운 가지** 를 만드는 것을 의미함
- 각 **Work 의 특성** ( 핫픽스, 이슈체킹 등 ) 혹은 **팀의 업무** ( 서버팀, 배치팀 등 ) 작업의 특성에 따른 **독립적 작업공간** 을 보장해줌

~~~shell
# 현재 브랜치 목록 확인
git branch -l
# 해당 이름으로 브랜치 생성
git branch "브랜치 명"
# 해당 브랜치 삭제
git branch -d "브랜치 명"
# 해당 브랜치로 전환 ( 해당 브랜치가 없을 경우 에러 )
git checkout "브랜치 명"
# 해당 브랜치 생성 후 전환 
git checkout -b "브랜치 명"
~~~



#### Merge Commit

- Branch 간에 **서로 다른 Commit** 에 의해 파일의 현황이 다를 경우
- Git 은 각 Branch 헤드의 **변경사항을 비교, 체크해 자동 반영하고 Commit** 을 진행함
- **패치** 라고 생각할 수 있음

~~~shell
# feature/ui 브랜치에서 작업 후 커밋
git branch feature/ui
... ( work )
git add .
git commit -m "feature/ui : Work Something"
# master 브랜치 전환 후 작업, 커밋
git branch master
... ( work )
git add .
git commit -m "master : Rewrite README"
# ----> master 브랜치와 feature/ui 브랜치의 커밋 내역이 달라짐
# master 브랜치에서 feature/ui 브랜치 작업 병합 진행
git merge feature/ui
# ----> feature/ui 의 커밋을 master 의 커밋에 Merge
# ----> feature/ui 에서의 커밋은 전부 master 의 커밋에 반영됨 
# Merge 가 끝난 브랜치의 데이터 삭제
git branch -d feature/ui
~~~



