# 구글 Mail api 관련
메일관련 소스 코드 입니다.

# C#에서 적용한 메일 관련 소스 입니다.
메일 관련 코드 소스

# [메일 관련 에러]
<br/>
22년 5월 30일부터 구글 크롬에서 메일 관련 ‘보안 수준이 낮은 앱’ 권한 처리가 사라졌습니다.
그러므로 기존에 사용 되었던 메일 발송 코드가 에러가 납니다. 해당 오류는 아래와 같이 진행 하면 해결 됩니다.

https://myaccount.google.com/security

구글 계정 > 보안으로 접속하여서 ‘2단계 인증‘을 처리해야 합니다.
![image](https://user-images.githubusercontent.com/34737952/174235777-90c94ed0-55b6-4666-8b77-8a7ed690a5aa.png)

인증 처리 후, 하단에 ‘앱 비밀번호’를 추가해주어야 합니다.






# Web.config
```C#
<appSettings>
	<add key="MailID" value="아이디@gmail.com" />
	<add key="MailPW" value="2차 인증 받은 앱 비밀번호" />
</appSettings>
```

