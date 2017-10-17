# JSON REST 인터페이스 (JSON REST Interface)

vibe.d 프레임워크를 이용해 JSON 형태의 응답을 제공하는 웹 서비스를 단시간에 만들 수 있습니다.

만약 `/api/v1/chapters` 경로로 HTTP 요청이 들어왔을 때 아래처럼 응답을 주고 싶다고 가정해봅시다.

```json
    [
      {
        "title": "안녕",
        "id": 1,
        "sections": [
          {
            "title": "세상",
            "id": 1
          }
        ]
      },
      {
        "title": "고급과정",
        "id": 2,
        "sections": []
      }
    ]
```

먼저 해야할 일은 **일대일** 로 구조가 정확히 대응되는 D 언어 `struct` 자료 구조와
REST API에 필요한 함수가 정의된 인터페이스를 만드는 것입니다.

```d
    interface IRest
    {
        struct Section {
            string title;
            int id;
        }
        struct Chapter {
            string title;
            int id;
            Section[] sections;
        }
        @path("/api/v1/chapters")
        Chapter[] getChapters();
    }
```

`interface` 는 바로 이용할 수 있는 형태가 아니라, 구현이 필요합니다. 실질적 구현을 위해서,
`IRest` 를 상속 받아 나머지 코드를 채웁니다.

```d
    class Rest: IRest {
        Chapter[] getChapters() {
          // 코드를 채워넣어야합니다
        }
    }
```

이제 구현이 완료된 `class` 로부터 객체를 만들어, `URLRouter` 에 REST 인터페이스 구현체로써 등록해주면 모든 게 완료됩니다!

```d
    auto router = new URLRouter;
    router.registerRestInterface(new Rest);
```

vibe.d가 가지고 있는 *REST 인터페이스 생성기* 는 POST 요청으로 전달받은 JSON 데이터로부터,
내부 함수의 각 입력값에 대응하는 값을 자동으로 추출해낼 수도 있습니다.

REST 서버를 만들 때 사용했던 인터페이스(여기서는 `IRest` 가 해당됩니다)는 역으로, REST API에 연결하기 위한 클라이언트를 만들 때 쓰이는 것도 가능합니다. 별도의 모듈이나 변환 과정이 필요하지 않으며, 서버 쪽과 같은 인터페이스를 공유합니다.

```d
    auto api = new RestInterfaceClient!IRest("http://127.0.0.1:8080/");
    // HTTP GET 요청을 /api/v1/chapters 경로로 보내고, 응답값에서 필요한 값을 추출하고,
    // IRest 인터페이스에 정의해둔 Chapter라는 struct 타입의 값으로 변환합니다.
    auto chapters = api.getChapters();
```

## {SourceCode:disabled}

```d
import vibe.d;

interface IRest
{
    struct Section
    {
        string title;
        int id;
    }

    struct Chapter
    {
        string title;
        int id;
        Section[] sections;
    }

    @path("/api/v1/chapters")
    Chapter[] getChapters();

    /*
    POST 요청으로 전달될 데이터 형식은
    아래와 같습니다.
        { "title": "D 언어" }
    */
    @path("/api/v1/add-chapter")
    @method(HTTPMethod.POST)
    int addChapter(string title);
}

class Rest: IRest
{
    private Chapter[] chapters_;

    this()
    {
        chapters_ = [ Chapter("안녕", 1,
                [ Section("세상", 1) ] ),
                 Chapter("고급과정", 2) ];
    }

    Chapter[] getChapters()
    {
        return chapters_;
    }

    int addChapter(string title)
    {
        import std.algorithm : map, max, reduce;
        // 매 챕터가 추가될 때마다
        // 마지막으로 추가된 챕터의 ID보다
        // 하나 큰 숫자를
        // 새 ID로 부여합니다.
        auto newId = chapters_.map!(x => x.id)
                            .reduce!max + 1;
        chapters_ ~= Chapter(title, newId);
        return newId;
    }
}

shared static this()
{
    auto router = new URLRouter;
    router.registerRestInterface(new Rest);

    auto settings = new HTTPServerSettings;
    settings.port = 8080;
    listenHTTP(settings, router);
}
```
