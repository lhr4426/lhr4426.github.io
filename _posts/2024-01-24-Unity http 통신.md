---
title: "Unity http 통신"
date: 2024-01-24 00:00:00 +09:00
category: "Next Reality (컴공 캡스톤)"
tags :
    [
        "Unity", "http", "통신"
    ]
description : 다른 캡스톤 팀원이 웹 서버를 만들어 두어서, 그 서버 메소드를 사용하기 위해 get을 사용하는 과정을 기록했음  
---
Unity Version : 2022.3.16f1 LTS

## 사용자가 입력한 값 받기

우선 사용자가 회원가입할 때 입력한 ID, Email, PW를 스크립트에서 받아야 함

```csharp
// RegisterPopup.cs
public void OnClickRegister()
{
    id = idField.text;
    email = emailField.text;
    pw = pwField.text;
    Debug.Log("Register Clicked");
    Debug.Log("ID : " + id + " Email : " + email + " PW : " + pw);
}
```

### Get으로 ID 있는지 확인

Get을 위한 코루틴 생성

```csharp
// RegisterPopup.cs
IEnumerator RequestGet(string url, Action<string> callback)
{
    UnityWebRequest request = UnityWebRequest.Get(url);
    // response가 올 때까지 턴 넘김
    yield return request.SendWebRequest();
    if (request.result != UnityWebRequest.Result.Success)
    {
        Debug.Log(request.error);
    }
    else
    {
        // response의 내용을 callback에 넣음
        string result = request.downloadHandler.text;
        Debug.Log("Request Text : " + result);
        callback(result);
    }
}
```

버튼을 클릭했을 때, Get으로 사용 가능한 아이디인지 확인할 수 있도록 함

```csharp
// RegisterPopup.cs
public void OnClickRegister()
{
    msgTxt.gameObject.SetActive(false);

    id = idField.text;
    email = emailField.text;
    pw = pwField.text;

    Debug.Log("Register Clicked");
    Debug.Log("ID : " + id + " Email : " + email + " PW : " + pw);

    // ID 중복 검사를 위한 코루틴 생성 및 실행
    StartCoroutine(RequestGet(url + duplicateIDUrl + id, (callback) =>
    {
        Debug.Log("RequestGet Callback : " + callback);
        if (callback == "true")
        {
            msgTxt.gameObject.SetActive(true);
            msgTxt.color = Color.green;
            msgTxt.text = "This ID is UNIQUE! GOOD!";
        }
        else
        {
            msgTxt.gameObject.SetActive(true);
            msgTxt.color = Color.red;
            msgTxt.text = "Register Failed. Check your ID";
        }
    }));
}
```

### 결과

![이미 있는 아이디 작성했을 때](https://private-user-images.githubusercontent.com/37866532/302090097-7db2b6df-13f0-4fc3-9998-fc0334bfbd76.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3MDcwMzc1MjgsIm5iZiI6MTcwNzAzNzIyOCwicGF0aCI6Ii8zNzg2NjUzMi8zMDIwOTAwOTctN2RiMmI2ZGYtMTNmMC00ZmMzLTk5OTgtZmMwMzM0YmZiZDc2LnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNDAyMDQlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjQwMjA0VDA5MDAyOFomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWE5ZTFlNjA1OGQyN2NhYjVhZDUyMzMxYjg5OWQ0MjMwMDU5OGNmYWYxNTg4NDQ1M2JhMDc5ZDczYmEwYWE4Y2UmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JmFjdG9yX2lkPTAma2V5X2lkPTAmcmVwb19pZD0wIn0.DhO1i6LYQuh0tnwTLfgRiy41PnraKSuzg3mOJYzu86E)

이미 있는 아이디 작성했을 때

![Untitled](https://private-user-images.githubusercontent.com/37866532/302090101-eb0a24f7-8765-4b0a-b42f-5089f23acd1d.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3MDcwMzc1MjgsIm5iZiI6MTcwNzAzNzIyOCwicGF0aCI6Ii8zNzg2NjUzMi8zMDIwOTAxMDEtZWIwYTI0ZjctODc2NS00YjBhLWI0MmYtNTA4OWYyM2FjZDFkLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNDAyMDQlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjQwMjA0VDA5MDAyOFomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTI2OGIxNTc0NDljMjg4OTk5ZDUyODc5OGNjNzY4Mjk2OGU4M2ViMmE5OGRkMWQ3YjA0YzNmNjU3ZjZjY2YyMTUmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JmFjdG9yX2lkPTAma2V5X2lkPTAmcmVwb19pZD0wIn0.u6AJy0ryBy8dG4RvRbreQYo24-7_Pe4oqdWtpzQb8x4)

![없는 아이디 작성했을 때](https://private-user-images.githubusercontent.com/37866532/302090103-c0809ae7-1aba-47d1-a8e4-a6800131edb1.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3MDcwMzc1MjgsIm5iZiI6MTcwNzAzNzIyOCwicGF0aCI6Ii8zNzg2NjUzMi8zMDIwOTAxMDMtYzA4MDlhZTctMWFiYS00N2QxLWE4ZTQtYTY4MDAxMzFlZGIxLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNDAyMDQlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjQwMjA0VDA5MDAyOFomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWNkYTg0YjBlNTA1ODdlMmVmMzg0YzdlYzgzMDRhZWVhNWQ3NzYxZjI1MTQwZmFjYWFjNDhjMDRkNDRlMWU3NjAmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JmFjdG9yX2lkPTAma2V5X2lkPTAmcmVwb19pZD0wIn0.NFGJ8tjQq7A1T-T1dtSMJlny--4WXvZ8g8AkoIK5Wro)

없는 아이디 작성했을 때

![Untitled](https://private-user-images.githubusercontent.com/37866532/302090104-817c03e8-4987-4c6e-91d0-f0182966683c.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3MDcwMzc1MjgsIm5iZiI6MTcwNzAzNzIyOCwicGF0aCI6Ii8zNzg2NjUzMi8zMDIwOTAxMDQtODE3YzAzZTgtNDk4Ny00YzZlLTkxZDAtZjAxODI5NjY2ODNjLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNDAyMDQlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjQwMjA0VDA5MDAyOFomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWJiZGY4OWExNDEwN2UzM2Q1Yzc5MTZjYzZhYjVjMTk0ZTE0MGM5OGE5NmVjMDllNzhmMGZiMzg3NTI1NWMwZmQmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JmFjdG9yX2lkPTAma2V5X2lkPTAmcmVwb19pZD0wIn0.5_vAcj5Drmj-QhYVeulyZW2K4a5nE9Omc9EBKwLG1ko)

## 참고자료

[유니티 HTTP 통신 구현 핵심 정리1](https://coding-of-today.tistory.com/166)

[[Unity] HTTP REST api 통신 모듈 구현](https://trialdeveloper.tistory.com/65)

[유니티 웹서버 통신](https://velog.io/@_hoya_/Unity-Webserver-Networking)

[[Unity] UnityWebRequest 의 Get, Post 사용하기](https://velog.io/@minjujuu/Unity-UnityWebRequest-사용해서-파일-다운받기)

[[Unity3D] 웹 데이터 파일 다운로드,업로드(GET, POST) 및 저장 (UnityWebRequest)](https://timeboxstory.tistory.com/81)

[How to get Text from TextMeshPro input field](https://discussions.unity.com/t/how-to-get-text-from-textmeshpro-input-field/215584)

[[Unity] UnityWebRequest로 json 데이터를 Http body에  Post로 전달방법](https://timeboxstory.tistory.com/83)

[유니티[Unity3D] TMP_InputFIeld()타입 초기화](https://soopeach.tistory.com/13)

[Unity - [SerializeField],직렬화, [System.Serializable]](https://code-piggy.tistory.com/entry/Unity-SerializeField직렬화-SystemSerializable)

[Unity - using문(using directive , using statement)](https://code-piggy.tistory.com/entry/Unity-using문)
