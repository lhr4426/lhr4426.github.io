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

![이미 있는 아이디 작성했을 때](https://cdn.jsdelivr.net/gh/lhr4426/lhr4426.github.io@master/assets/img/posts/2024-01-24-Unity http 통신/1.png)

이미 있는 아이디 작성했을 때

![Untitled](https://cdn.jsdelivr.net/gh/lhr4426/lhr4426.github.io@master/assets/img/posts/2024-01-24-Unity http 통신/2.png)

![없는 아이디 작성했을 때](https://cdn.jsdelivr.net/gh/lhr4426/lhr4426.github.io@master/assets/img/posts/2024-01-24-Unity http 통신/3.png)

없는 아이디 작성했을 때

![Untitled](https://cdn.jsdelivr.net/gh/lhr4426/lhr4426.github.io@master/assets/img/posts/2024-01-24-Unity http 통신/4.png)

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
