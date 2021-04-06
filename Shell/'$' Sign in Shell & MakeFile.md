### '$' Sign in Shell & MakeFile



#### Background

- 기본적으로 `shell` 에서는 `$` 사인을 이용해 변수를 참조함
- `Makefile` 역시 이와 비슷하다 알고 있어 동일한 방식으로 작성하였으나 지속적인 에러가 발생해 정리해 본다



#### `$` 사인의 의미

- **Shell**

  > `shell` 내에서 변수를 참조하고 싶을 땐 `$` 를 변수명 앞에 붙이면 된다
  >
  > 또한, 명령어 치환 (Command Substitution) 을 통해 명령어의 출력 컨텍스트를 참조하는 용도로도 사용 가능하다

~~~shell
#!/bin/bash
# 기본적인 $ 를 이용한 변수 참조
for it in *;
do echo "i=$it";
done
# Command Substitution 으로 사용한 $ 사인
echo $(ls *.o)
~~~

- **Makefile**

  > `Makefile` 에서 변수를 참조하고 싶다면 `$(var)` 혹은 `${var}` 을 이용한다
  >
  > `Makefile` 에서 명령어 내에 사용하기 위해 `$` 를 리터럴 문자로 이용하고 싶다면 `$$` 로 표현한다

  **< 주의 > ** `Makefile` 에서 변수명이 한 글자인 경우에는 `$` 로도 참조할 수 있지만 자동변수들과 혼동될 수 있으므로 쓰지 말 것 ( **ex_ `$@` - 타겟 이름 , `$^` 타겟 의존 대상 목록 등** )

~~~makefile
#!make
# $(var) 을 이용한 참조
objects = program.o foo.o utils.o
program : $(objects)
		cc -o program $(objects)
# $$ ( 더블 달러 사인 ) 으로 리터럴 문자 $ 로서 사용
my-files:
    for it in *; 
    do echo "it=$$it";
    done
~~~



### REFERENCE

- https://wiki.kldp.org/HOWTO/html/Adv-Bash-Scr-HOWTO/commandsub.html
- https://blog.daum.net/english_100/10