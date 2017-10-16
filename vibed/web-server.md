# 웹 서버(Web server)

vibe.d 를 이용해 HTTP(S) 프로토콜을 처리하는 웹 서버를 즉석으로 만들 수 있습니다.


```d
    auto settings = new HTTPServerSettings;
    settings.port = 8080;
    listenHTTP(settings, &foo);
```

이 예제 코드는 8080포트에서 들어온 접속을 `foo` 라는 함수에게 전달해 HTTP 요청을 처리하도록 합니다.

listenHTTP에 전달될 함수는 `HTTPServerRequest` 와 `HTTPServerResponse` 객체를 입력으로 받습니다. 여기에서는 `foo` 를 예로 들겠습니다.

```d
    void foo(HTTPServerRequest req,
        HTTPServerResponse res) { ... }
```

물론 `foo` 와 같은 델리게이트에서 조건문을 사용해 HTTP 메소드와 경로별로 일일이 처리할 수도 있지만, 번거롭습니다.

이에 따라 HTTP 요청의 메소드(method)가 `GET` 인지 `POST` 인지 혹은 다른 것인지, 어떤 경로로 요청이 온건지에 따라 `.get("path", handler)` 나 `.post("path", handler)` 등 사전에 정의된 적절한 내부함수를 호출해주는 `URLRouter` 클래스가 편의를 위해 제공됩니다. 추가로 웹서버 상의 경로를 내부 함수로 연결해주는 *웹 인터페이스* 또한 등록이 가능합니다.

```d
    auto router = new URLRouter;
    router.registerWebInterface(new WebService);
    listenHTTP(settings, router);
```

`WebService` 는 아래의 간단한 규칙에 따라 웹 서비스로 들어온 요청을 내부 함수로 연결해줍니다.

* `index()` 함수명은 `/index` 경로를 요청할 때 호출됩니다.
* `getName()` 함수명은 `GET (get)` 메소드로 `/name (Name)` 을 요청할 때 호출될 함수명의 형태입니다.
* `postUsername()` 함수명은 `POST  (post)` 메소드로 `/username (Username)` 을 요청할 때 호출될 함수명의 형태입니다.

이러한 규칙에 따르기 힘든 경로라면, 내부 함수를 선언할 때 `@path("/hello/world")` 형태로 속성을 정의해주면 됩니다.
특히, 내부함수 선언시 입력값의 이름 앞에 `_` 를 붙이게 되면 `POST` 메소드로 요청을 처리할 수 있는 내부 함수가 됩니다.
그리고 내부 함수의 입력값으로 받아올 웹 요청 파라미터는 직접 `@path` 속성 내에 아래와 같이 지정할 수 있습니다.

`:id` 로 HTTP 요청에서 받아온 값은 `int _id` 내에 저장됩니다.

```d
    @path("/my/api/:id")
    void foo(int _id)
```

HTTP 요청을 처리할 함수에 모두 `HTTPServerRequest` 와 `HTTPServerResponse` 를 받도록 반드시 정의할 필요는 없습니다. 둘 중 하나만 적어도 되고, 아예 생략해도 됩니다.

vibe.d 프레임워크가 미리 검사하여, 파라미터로 전달할 필요가 없는 경우에는 그냥 넘어갑니다.

## {SourceCode:disabled}

```d
import vibe.d;

class WebService
{
    /*
    세션 변수를 사용합니다.
    세션은 해당 사용자에 대해 서버 상에서만
    유효한 저장공간으로, 세션이 유효할 동안
    모든 요청을 처리할 때 이용할 수 있습니다.
    */
    private SessionVar!(string, "username")
        username_;

    /*
    vibe.d 는 기본적으로 / 경로로 들어오는
    요청을 index 함수로 처리합니다.
    */
    void index(HTTPServerResponse res)
    {
        auto contents = q{<html><head>
            <title>자기소개 시간!</title>
        </head><body>
        <form action="/username" method="POST">
        너의 이름은?
        <input type="text" name="username">
        <input type="submit" value="Submit">
        </form>
        </body>
        </html>};

        res.writeBody(contents,
                "text/html; charset=UTF-8");
    }

    /*
    @path 속성을 이용해 vibe.d 가 처리 가능한
    기본 규칙과는 다른 경로와 요청에 대한
    처리가 가능합니다. (URL Routing)
    여기서는 "/name"으로 들어온 요청을
    getName 함수가 처리하도록 연결했습니다.
    */
    @path("/name")
    void getName(HTTPServerRequest req,
            HTTPServerResponse res)
    {
        import std.string : format;

        // HTTP 요청의 헤더에 담긴 정보를
        // <li> 태그 하나 하나에 묶어
        // 나열합니다.
        // 이걸 통해 HTTP 요청에 어떤 정보가
        // 같이 전달되는지 볼 수 있습니다.
        string[] headers;
        foreach(key, value; req.headers) {
            headers ~=
                "<li>%s: %s</li>"
                .format(key, value);
        }
        auto contents = q{<html><head>
            <title>자기소개 시간!</title>
        </head><body>
        <h1>너의 이름은 %s.</h1>
        <h2>HTTP 헤더 정보</h2>
        <ul>
        %s
        </ul>
        </body>
        </html>}.format(username_,
                headers.join("\n"));

        res.writeBody(contents,
                "text/html; charset=UTF-8");
    }

    void postUsername(string username,
            HTTPServerResponse res)
    {
        username_ = username;
        auto contents = q{<html><head>
            <title>자기소개 시간!</title>
        </head><body>
        <h1>너의 이름은 %s이구나. 반가워.</h1>
        </body>
        </html>}.format(username_);

        res.writeBody(contents,
                "text/html; charset=UTF-8");
    }
}

void helloWorld(HTTPServerRequest req,
        HTTPServerResponse res)
{
    res.writeBody("안녕!");
}

shared static this()
{
    auto router = new URLRouter;
    router.registerWebInterface(new WebService);
    router.get("/hello", &helloWorld);

    auto settings = new HTTPServerSettings;
    // SessionVar 사용을 위해 세션이
    // 서버에 어떻게 저장될지
    // 설정해줘야합니다.
    settings.sessionStore =
        new MemorySessionStore;
    settings.port = 8080;
    listenHTTP(settings, router);
}
```
