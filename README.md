# C#에서 적용한 구글 Mail 관련 소스 입니다.
메일 관련 코드 소스

# [메일 관련 에러]
<br/>
22년 5월 30일부터 구글 크롬에서 메일 관련 ‘보안 수준이 낮은 앱’ 권한 처리가 사라졌습니다.
그러므로 기존에 사용 되었던 메일 발송 코드가 에러가 납니다. 해당 오류는 아래와 같이 진행 하면 해결 됩니다.

https://myaccount.google.com/security

구글 계정 > 보안으로 접속하여서 ‘2단계 인증‘을 처리해야 합니다.
![image](https://user-images.githubusercontent.com/34737952/174235777-90c94ed0-55b6-4666-8b77-8a7ed690a5aa.png)

인증 처리 후, 하단에 ‘앱 비밀번호’를 추가해주어야 합니다.

![image](https://user-images.githubusercontent.com/34737952/174236514-f2357831-d82d-4a43-a884-9f178e41993e.png)
![image](https://user-images.githubusercontent.com/34737952/174236521-2eb96513-d046-4e0c-8aef-617c844db795.png)
<br/>
맞춤이름 선택 클릭 원하는 이름으로 생성 > 16자리 비밀번호가 생성되며, 이 번호를 기존에 사용한 이메일 비빌번호에 바꾸어 넣어주면 기존에 잘 작동 되었던 메일 오류가 해결됩니다.
<br/>
![image](https://user-images.githubusercontent.com/34737952/174236530-db8f5947-67c5-4f56-992c-59c078499874.png)
![image](https://user-images.githubusercontent.com/34737952/174236580-8b0df22d-fc3d-48f3-a728-1baad6121ae7.png)
16자리 비번을 기존에 사용한 비번으로 교체

# Web.config
```C#
<appSettings>
	<add key="MailID" value="아이디@gmail.com" />
	<add key="MailPW" value="2차 인증 받은 앱 비밀번호" />
</appSettings>
```

```C#
//비밀번호 찾기 에서 이메일 입력시 메일 발송
[HttpPost]
[Route("Api/Mail/PasswordChange")]
public JsonResult SendMail(string Email)
{
    var result = new ReturnValue();
    string mail_id = ConfigurationManager.AppSettings["MailID"].IsNull("");
    string mail_pw = ConfigurationManager.AppSettings["MailPW"].IsNull("");

    if (!string.IsNullOrWhiteSpace(Email))
    {
	try
	{
	    Member member = this.Db.Select<Member>($"Email='{Email}'").FirstOrDefault();
	    if(member.JoinType =="Normal")
	    {
		if (member != null && member.MemberSeq > 0)
		{
		    result = this.Db.PasswordResetLogSave(Email, false, -1);
		    //메일로 토큰 넘기기
		    var log = this.Db.Select<PasswordResetLog>($"PasswordResetLogSeq='{result.Code}'").FirstOrDefault();
		    result = this.Site.SendMailResetPassword(mail_id, mail_pw, Email, member.UserName, log.Token);

		}
		else
		{
		    result.Error("해당 메일로 가입된 내역이 없습니다.");
		}
	    }
	    else
	    {
		result.Error("SNS 간편가입 회원은 해당 플랫폼에서 확인해주세요.");
	    }
	}
	catch (Exception ex)
	{
	    result.Error(ex);
	    Logger.Current.Error(ex);
	}
    }
    else
    {
	result.Error("이메일 주소를 입력하세요.");
    }

    return Json(result);
}
```

```C#
public virtual ReturnValue SendMailResetPassword(string fromMail, string fromPassword, string toMail, string toName, string authURL)
{
    var result = new ReturnValue();

    try
    {
	string SiteURL = $"{HttpContext.Current.Request.Url.Scheme}://{HttpContext.Current.Request.Url.Host}";

	result = this.SendMail(
			    fromMail,
			    fromPassword,
			    toMail,
			    toName,
			    "[업체명] 비밀번호 재설정 메일입니다.",
			    HttpUtility.UrlDecode(MailHelper.CreateResetPaasswordMail($"{SiteURL}", authURL))
			    );
    }
    catch (Exception ex)
    {
	if (Log != null) Log.Error(ex);
	result.Error(ex);
    }

    return result;
}
```

```C#
public virtual ReturnValue SendMail(string fromMail, string fromPassword, string toMail, string toName, string title, string Body)
{
    var result = new ReturnValue();

    this.Log.Debug("====[ SendMail ]====");
    this.Log.Debug($"fromMail : {fromMail}");
    this.Log.Debug($"fromPassword : {fromPassword}");
    this.Log.Debug($"toMail : {toMail}");
    this.Log.Debug($"toName : {toName}");
    this.Log.Debug($"title : {title}");
    this.Log.Debug($"Body : {Body}");

    try
    {
	using (var client = new System.Net.Mail.SmtpClient("smtp.gmail.com"))
	{
	    client.Credentials = new System.Net.NetworkCredential(fromMail, fromPassword);
	    client.EnableSsl = true;
	    client.Port = 587;
	    client.DeliveryMethod = SmtpDeliveryMethod.Network;

	    System.Net.Mail.MailMessage message = new System.Net.Mail.MailMessage();
	    message.Subject = title;
	    message.From = new MailAddress(fromMail, "업체명", System.Text.Encoding.UTF8);
	    message.To.Add(toMail);
	    message.IsBodyHtml = true;
	    message.Body = Body;

	    client.Send(message);
	}
	result.Success(1);
    }
    catch (Exception ex)
    {
	if (Log != null) Log.Error(ex);
	result.Error(ex);
    }

    return result;
}
```

# 비밀번호 재설정 전 본인 확인 메일 html 폼
해당 폼 안에 AuthURL로 컨트롤러에서 값을 넣어서 메일로 넣어 보내줍니다.
```html
<!DOCTYPE html>
<html lang="ko">
<body style="width: 100%;height: 100%;padding: 0;margin: 0 auto;box-sizing: border-box;font-family: 'Noto Sans KR', sans-serif;-ms-text-size-adjust: 100%;-webkit-text-size-adjust: auto;">
    <table id="Table_wrap" style="width: 700px;border-bottom-left-radius: 10px;border-bottom-right-radius: 10px;border: 0;background-color: #fff;-ms-text-size-adjust: 100%;-webkit-text-size-adjust: auto;height: 600px;margin: 0 auto;">
        <tr style="background: #F9F9F9;">
            <td style="width: 700px;height: 140px;padding: 5px 0 30px 0;background: #F9F9F9;-ms-text-size-adjust: 100%;-webkit-text-size-adjust: auto;">
                <table style="width: 700px;height: 220px;margin-top: 30px;background: #F9F9F9;border: 0;background-color: #fff;-ms-text-size-adjust: 100%;-webkit-text-size-adjust: auto;">
                    <tr style="height: 140px;">
                        <td style="color: #474B52;text-align: center;-ms-text-size-adjust: 100%;-webkit-text-size-adjust: auto; background: #F9F9F9;">
                            <p style="width: 85%;display: inline-block;border-top: 1px solid #CACACA;font-size: 30px;font-weight: 100;padding: 30px 0 0 0;margin: 0;-ms-text-size-adjust: 100%;-webkit-text-size-adjust: auto;">
                                <span style="font-weight: 700;">비밀번호 재설정</span>을 위한 안내 이메일입니다.
                            </p>
                            <p style="width: 85%;display: inline-block;border-bottom: 1px solid #CACACA;color: #474B52;font-size: 14px;font-weight: 100;padding: 10px 0 30px 0;margin: 0;-ms-text-size-adjust: 100%;-webkit-text-size-adjust: auto;">
                                요청하신 비밀번호 재설정을 위한 본인 확인 메일입니다.<br>비밀번호 재설정을 하시려면 아래버튼을 클릭해주세요.
                            </p>
                        </td>
                    </tr>
                </table>
                <table style="width: 100%;background: #F9F9F9;border: 0;background-color: #fff;-ms-text-size-adjust: 100%;-webkit-text-size-adjust: auto;">
                    <tr>
                        <td style="text-align: center;padding-bottom: 15px;-ms-text-size-adjust: 100%;-webkit-text-size-adjust: auto; background: #F9F9F9;">
                            <a href="{SiteURL}/Member/ResetPw/Token={AuthURL}" style="padding: 12px 20px 12px 20px; font-size: 16px; color: #fff; background-color: #5DC6CC; font-weight: 100; display: inline-block; border-radius: 30px; border: 0; outline: 0; text-decoration: none; -ms-text-size-adjust: 100%; -webkit-text-size-adjust: auto;">비밀번호 재설정하기</a>
                        </td>
                    </tr>
                </table>
            </td>
        </tr>
    </table>
</body>
</html>
```

