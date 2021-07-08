---
title: '인증(Authentication)'
excerpt: 인증에 대해 이해하기
categories:
    - Web

tag:
    - TIL
    - web
    

comments: true
last_modified_at: 2021-07-08T11:00:00+09:00
toc: true
---

![post-thumbnail](https://media.vlpt.us/images/suasue/post/de0a0d0f-8f0c-4fd0-bb89-8a23611f6248/authentication-software2.png)

[사진출처](https://managementmania.com/en/authentication-software)

백엔드 개발자로서 회원가입과 로그인 기능 구현 시 꼭 알아야 할 인증의 개념과 비밀번호 암호화 방식에 대해서 알아보도록 하겠다.

## 🤙 인증(Authentication)

### 인증이란?

- 인증이란 유저가 누구인지(identification)를 식별하는 절차
- 아이디와 비밀번호를 확인하는 절차
- 회원가입과 로그인

### 언제 인증을 사용할까?

- 우리 서비스를 누가 쓰는지, 어떻게 쓰는지 추적하고 싶을 때

### 어떤 과정으로 이루어질까?

1. 유저 아이디와 비번 생성
2. 유저 비번 암호화 해서 DB에 저장
3. 유저 로그인 -> 아이디와 비밀번호 입력
4. 유저가 입력한 비밀번호 암호화한 후 암호화되서 DB에 저정된 유저 비밀번호와 비교
5. 일치하면 로그인 성공
6. 로그인 성공하면 `access token`을 클라이언트에게 전송
7. 유저는 로그인 성공후 다음부터는 `access token`을 첨부해서 request를 서버에 전송함으로서 매번 로그인 해도 되지 않도록 한다.

------

## 🔑 비밀번호 암호화

![img](https://media.vlpt.us/images/suasue/post/bef08a5e-0ca8-4810-b043-9613905d5d96/image.png)

[사진출처](https://devlog-wjdrbs96.tistory.com/212)

- 인증에 필요한 것은 아이디, 이메일주소, 비밀번호 등이 있는데, 이 중 가장 중요한 것은 **비밀번호**. 비밀번호를 알면 그 사람의 개인정보에 접근할 수 있기 때문에
- 비밀번호는 꼭 **`암호화`**해야 한다.

### 비밀번호를 암호화해서 저장해야 하는 이유

- 개인정보보호법에서 그렇게 하라고 정해놓았다.
- DB가 해킹을 당하면 유저의 비밀번호도 그대로 노출 된다.
- 외부 해킹이 아니더라도 내부 개발자나 인력이 유저들의 비밀번호를 볼 수 있다.

(페이스북에선 2019년까지도 사용자의 비밀번호를 평문으로 저장해왔다고 한다.. [참고 기사](https://www.boannews.com/media/view.asp?idx=78058&page=1&kind=1))

### 비밀번호 암호화 시기

1. DB에서 비밀번호 저장할 때 암호화
   Databases에 저장 시 개인 정보를 해싱하여 복원할 수 없도록 한다.
   (비밀번호가 이미 백엔드로 넘어갔을 때)
2. 통신 시 개인 정보를 주고받을 때 SSL을 적용하여 암호화(HTTPS) --> 이 부분은 따로 포스팅할 예정
   (프론트에서 백으로 텍스트가 전달될 때)

------

## 🔒 비밀번호를 저장할 때 사용하는 암호화 방식

### 단반향 해쉬 함수

![img](https://media.vlpt.us/images/suasue/post/494aa426-81de-42c5-bb29-ddc00710c86e/image.png)

[사진출처](https://velog.io/@nameunzz/단방향-해시-함수)

- 단방향 해시 함수는 수학적인 연산을 통해 원본 메시지를 변환하여 암호화된 메시지인 다이제스트를 생성한다. 원본 메시지를 알면 암호화된 메시지를 구하기는 쉽지만 암호화된 메시지로는 원본 메시지를 구할 수 없어야 하며 이를 '단방향성'이라고 한다.
- 예를 들어 `army`라는 패스워드를 흔히 쓰이는 해시 알고리즘 SHA-256으로 인코딩하면 다음과 같은 결과를 얻을 수 있다

```null
b2fd5cdba2b1484db46c26a3426f70878d2b31f44f18f806fd262f34887a6320
```

- 대부분의 해시 함수는 입력 값의 일부가 변경되었을 때 다이제스트가 완전히 달라지도록 설계되어 있다. `army2`라는 값의 SHA-256 다이제스트는 아래와 같으며 위의 `army`와는 완전히 달라진 것을 확인할 수 있다.

```null
4d6ba57a1cbffa2fd4def7c2a0b74f1151971454659b78a833103c4e60d7da0d
```

- 원본 메시지를 살짝만 바꾸어도 결과가 크게 달라지는 특징을 avalanche 효과라고 하며, 사용자의 원본 패스워드를 추론하기 어렵게 만드는 중요한 요소이다.

### 단방향 해시 함수의 문제점

#### 인식 가능성(recognizability)

- 단반향 해쉬에는 같은 값을 해싱하면 항상 같은 다이제스트가 나온다는 허점이 존재한다. 이 특성을 이용해서 가능한 경우의 수를 미리 해시값으로 만들어 놓은 테이블을 레인보우 테이블(rainbow table)이라고 한다. 레인보우 테이블을 이용한 공격 방식을 **레인보우 테이블 공격(rainbow table attack)**이라 한다.

#### 속도(Speed)

- 해시 함수는 원래 짧은 시간에 데이터를 검색하기 위해 설계된 것이다. 해시 함수의 빠른 처리 속도로 인해 공격자는 매우 빠른 속도로 임의의 문자열의 다이제스트와 해킹할 대상의 다이제스트를 비교할 수 있다. 이를 **무차별 대입공격(brute-force attack)**라고 한다.
- 이런 방식으로 패스워드를 추측하면 패스워드가 충분히 길거나 복잡하지 않은 경우에는 그리 긴 시간이 걸리지 않는다.

### 단방향 해시 함수 보완하기

![img](https://media.vlpt.us/images/suasue/post/d74b8b99-0e10-4c37-9780-19f429596b3c/image.png)

#### 솔팅(Salting)

- 솔팅(Salting)은 실제 비밀번호 이외에 추가적으로 랜덤 데이터(Salt)를 더해서 해시값을 계산하는 방법이다.
- 이 방법을 사용하면, 공격자가 실제 패스워드의 다이제스트를 알아내더라도 솔팅된 다이제스트를 대상으로 패스워드 일치 여부를 확인하기 어렵다. 또한 사용자별로 다른 솔트를 사용한다면 동일한 패스워드를 사용하는 사용자의 다이제스트가 다르게 생성되어 인식 가능성 문제가 크게 개선된다.
- 레인보우 테이블 공격(rainbow table attack)을 막을 수 있다.

#### 키 스트레칭(Key Stretching)

- 키 스트레칭(Key Stretching)은 해쉬 값을 여러번 반복해서 해시 하는 행위이다.
- 입력한 패스워드를 동일한 횟수만큼 해시해야만 입력한 패스워드의 일치 여부를 확인할 수 있다.
- 일반적인 장비로 1초에 50억 개 이상의 다이제스트를 비교할 수 있지만, 키 스트레칭을 적용하여 동일한 장비에서 1초에 5번 정도만 비교할 수 있게 한다.
- 무차별 대입공격(brute-force attack)을 막을 수 있다.

> ### 양방향 암호화는 언제 쓰이나?
>
> 양방향 암호화는 백엔드 개발자가 암호화해서 저장해야 하는 민감한 정보이나 다시 복호화해서 사용해야 하는 경우에 사용된다.
> 예를 들어 병원 시스템에서 주민등록번호는 민감한 정보이긴 하나 자주 사용해야 하므로 복호화할 수 있어야 한다. 또한, 쿠팡에서 쉽게 결제할 수 있는 이유도 신용카드 정보를 양방향 암호화로 저장해두었기 때문이다. 엄격하게 하는 곳은 주소도 양방향 암호화 해두기도 한다.

------

## 🔏 bcrypt

- bcrypt는 단방향 해쉬 함수로 암호화할 때 사용하는 대표적인 라이브러리이다. Salting과 Key Stretching 적용을 편하게 해준다.
- bcrypt는 hash결과값에 소금값과 해시값 및 반복횟수를 같이 보관하기 때문에 비밀번호 해
  싱을 적용하는데 있어 DB설계를 복잡하게 할 필요가 없다.
- bcrypt를 통해 해싱된 결과 값(Digest)의 구조는 아래와 같다.
  ![img](https://media.vlpt.us/images/suasue/post/7bba9af7-84c8-4ee0-9c01-05543bef9e26/image.png)

### bcrypt 사용하기

bcrypt를 설치한다.

```python
pip install bcrypt 
```

bcrypt를 import한다.

```python
import bcrypt
```

#### 비밀번호 암호화하기

bcrypt는 `str`데이터가 아닌 `bytes` 데이터를 암호화한다.
`str` 을 encode해서 `bytes`로 만들어 준다.(반대는 decode이다.)
encode 시에는 우리가 인식할 수 있는 형태로 변환하기 위해 `UTF-8` 유니코드 문자 규격을 사용한다.

```python
password = '1234'
encoded_password = password.encode('utf-8')
```

`bcrypt.hashpw()` 메소드는 (bytes 형식의 패스워드, 소금값)을 인자로 받아 해싱된 비밀번호를 만든다.
`bcrypt.gensalt()` 메소드로 랜덤한 소금값을 생성할 수 있다.

```python
hashed_password = bcrypt.hashpw(encoded_password, bcrypt.gensalt())
```

`print()`로 확인해보자. 암호화된 데이터의 타입은 Bytes이다.

```python
print(hashed_password)
b'$2b$12$YFs9rh.1LgJwZuf9ibyjpuLvBoCaGX0MzedFWF2Jo0zU3lMZurZ4a'

type(hashed_password)
<class 'bytes'>
```

#### 비밀번호 확인하기

`bcrypt.checkpw()` 메소드로 입력된 비밀번호가 맞는지 확인할 수 있다. (입력받은 패스워드, 저장된 암호화된 패스워드)를 각각 인자로 받는다. 일치하면 True를 반환한다.

```python
new_password = '1234'
bcrypt.checkpw(new_password.encode('utf-8'),hashed_password)
True
```

> 참고사이트
> https://d2.naver.com/helloworld/318732
> https://st-lab.tistory.com/100
> https://k0102575.github.io/articles/2020-03/hash
> https://velog.io/@nameunzz/단방향-해시-함수
> https://velog.io/@hwang-eunji/backend-django-비밀번호-암호화-구현