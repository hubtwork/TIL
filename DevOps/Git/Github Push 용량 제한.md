### Github Push 용량 제한 해결

> ✔ **OS : macOS Big Sur 11.1**
>
> ✔ **Git version 2.24.3**



#### Background

- Git 을 사용하면서 **Remote Repository** 를 선정할 때 체계화가 잘 되어 있는 `Github` 나 `Bitbucket` 을 사용

- 두 플랫폼 모두 파일 / 프로젝트의 용량 제한이 걸려있어 필요할 경우 업로드 못하는 상황이 발생

  > **Github** - 각 파일 100mb 제한
  >
  > **Bitbucket** - 전체 프로젝트 2GB 제한

- 특히 최근 **인공지능, 머신러닝** 등의 데이터 활용 분야에서 대용량의 파일을 올려야할 소요가 생길 수 있기에 알아둘 필요가 있음



#### Problem

~~~bash
...
> remote: warning: Large files detected.
> remote: warning: File MockDbData.zip is 61.20 MB; this is larger than GitHub's recommended maximum file size of 50 MB
...
> remote: warning: Large files detected.
> remote: error: File MockDbImages.zip is 223.14 MB; this exceeds GitHub's file size limit of 100 MB
...
~~~

- **Github** Remote Repository 에 Push 하는 중, 용량이 큰 파일 들에 대해 위와 같은 `warning` 및 `error` 가 발생
- `warning` : 50mb ~ 100mb 사이의 파일이 확인될 경우
- `error` : 100mb 이상의 파일이 확인될 경우



#### Solution

- **Git LFS** 를 git 에 적용한 후 대용량 파일의 업로드가 가능 ( **Github Recommendation** )
- **LFS Installation**
  - 아래 방법으로 각 운영체제에 **git lfs Extension** 을 설치한 후 적용을 원하는 레포지토리에 적용

~~~shell
# Ubuntu / Debian ( using Apt )
$ curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | sudo bash
$ sudo apt-get install git-lfs

# OSX ( using Hombrew )
$ brew install git-lfs

# RHEL / CentOS ( using yum )
$ curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.rpm.sh | sudo bash
$ sudo yum install git-lfs

--------------

# After Extension Installed ( All OS )
$ cd [ target Repository ]
$ git lfs install
~~~

- **LFS Tracking**
  - **LFS** 는 전체 레포지토리에 적용되는 것이 아니라 트래킹할 파일을 지정해주어야 함
  - **LFS** 트래킹을 적용한 후에는 `.gitattributes` 를 ADD 함으로서 LFS Pointer 들을 커밋에 포함시킬 수 있음

~~~shell
# [ Option ] if already git added large file, unstag that file first.
$ git rm --cahced <file path>
# tracking Large File, and original file will be LFS Pointer file.
$ git lfs track <file path>
# add .gitattributes for containing LFS Pointer
$ git add .gitattributes
$ git commit -m "< Commit Message >"
$ git push <remote> <branch>
~~~

- **LFS Untracking**
  - **LFS** 로 트래킹되고 있는 파일들은 `.gitattributes` 를 통해 관리됨
  - 제한 용량이하로 변경되어 **LFS** 의 사용이 불필요해지면 **Untracking** , **LFS Pointer Dismiss** 를 해줄 것

~~~shell
# Unstagging Target file.
# 1. use 'untrack' command
$ git lfs untrack <file path>
# 2. delete manually in '.gitattributes'
$ vim .gitattributes

# Unstagging Changes on Commit ( Originally LFS Pointer )
# ADD Changes ( Not LFS Pointer ) for files
$ git rm -- cached <file path>
$ git add <file path>
~~~



### REFERENCE

- https://docs.github.com/en/github/managing-large-files/conditions-for-large-files

- https://docs.github.com/en/github/managing-large-files/versioning-large-files