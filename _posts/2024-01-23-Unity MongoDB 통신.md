---
title: "Unity MongoDB 통신"
date: 2024-01-23 00:00:10 +09:00
category: "Next Reality (컴공 캡스톤)"
tags :
    [
        "Unity", "MongoDB", "데이터베이스"
    ]
description : 오브젝트 데이터 송수신을 위한 MongoDB 연결을 공부함
---
Unity Version : 2022.3.16f1 LTS

## MongoDB 사용 설정

MongoDB를 유니티에서 사용하기 위해 MongoDB Driver가 필요

드라이버는 NuGet 패키지인데 이걸 유니티에서 바로 사용할 수가 없음

그래서 NuGetForUnity를 설치해야함

### NuGetForUnity 설치

1. Window → Package Manager → Add package from git URL… 
2. https://github.com/GlitchEnzo/NuGetForUnity?path=/src/NuGetForUnity 입력

성공하면 아래와 같이 툴바에 NuGet 드롭다운 버튼이 생김

![Untitled](https://cdn.jsdelivr.net/gh/lhr4426/lhr4426.github.io@master/assets/img/posts/2024-01-23-Unity MongoDB 통신/1.png)

### MongoDB Driver 설치

1. NuGet → Manage NuGet Packages 
2. 검색창에 MongoDB 검색 → MongoDB.Driver 설치

스크립트 작성 준비 끝!

## Mongo DB 연결 스크립트 작성

### 연결이 되는지부터 테스트

```csharp
// DBTest.cs

using MongoDB.Bson;
using MongoDB.Driver; // 이거 해줘야됨

public class DBTest : MonoBehaviour
{
		// Connection String이 staatic이 아니면 아래 초기화에서 오류남
    private static string connectionString = "mongodb://localhost:27017"; 
    private string databaseName = "user_db";
    private string collectionName = "user";

    MongoClient client = new MongoClient(connectionString);
    IMongoDatabase database;
    IMongoCollection<BsonDocument> collection;

    private void Awake()
    {

    }

    // Start is called before the first frame update
    void Start()
    {
        database = client.GetDatabase(databaseName);
        collection = database.GetCollection<BsonDocument>(collectionName);

        // 테스트 데이터
        var document = new BsonDocument { { "id", "admin" }, { "email", "admin@test.com" }, { "pw", "admin" } };
        collection.InsertOne(document);
    }

    // Update is called once per frame
    void Update()
    {

    }
}
```

### 싱글톤 타입으로 DBManager 작성

DB에 접근하려는 시도는 앞으로도 많이 있을 것 같아서

전체 게임에서 제어할 수 있도록 싱글톤 타입의 DBManager를 생성함

```csharp
// DBManager.cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using MongoDB.Bson;
using MongoDB.Driver;

public class DBManager : MonoBehaviour
{
    private static string connectionString = "mongodb://localhost:27017";
    private string assetDatabaseName = "asset_db";
    private string assetCollectionName = "assets";

    protected MongoClient client = new MongoClient(connectionString);
    protected IMongoDatabase assetDatabase;
    protected IMongoCollection<BsonDocument> assetCollection;

    public static DBManager instance;

    private void Awake()
    {
        if (DBManager.instance == null)
        {
            DBManager.instance = this;
            DontDestroyOnLoad(gameObject);
        }
        else
        {
            Destroy(gameObject);
        }
    }
    

    // Start is called before the first frame update
    void Start()
    {
        try
        {
            assetDatabase = client.GetDatabase(assetDatabaseName);
            assetCollection = assetDatabase.GetCollection<BsonDocument>(assetCollectionName);
        }
        catch
        {
            Debug.Log("DB Connection Failed");
        }
    }

    // Update is called once per frame
    void Update()
    {
        
    }
}
```

## 참고자료

[유니티 웹서버 통신](https://velog.io/@_hoya_/Unity-Webserver-Networking)

[Quick Start for Unity - .NET SDK](https://www.mongodb.com/docs/realm/sdk/dotnet/unity/)

[[Unity] 유니티에서 Nuget 패키지 사용하기](https://velog.io/@yarogono/Unity-유니티에서-Nuget-패키지-사용하기)

[GitHub - GlitchEnzo/NuGetForUnity: A NuGet Package Manager for Unity](https://github.com/GlitchEnzo/NuGetForUnity/tree/master)
