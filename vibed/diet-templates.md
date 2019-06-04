# DIET 템플릿 (DIET Templates)

[DIET 템플릿](https://vibed.org/templates/diet) 지원으로 인해 vibe.d 상에서 동작하는 웹 페이지를 작성하는 게 간단해집니다.

DIET는 [Jade](http://jade-lang.com/) 를 기반으로 한 템플릿 언어로, HTML 페이지를 간단한 문법에 따라 생성할 수 있게 해줍니다.

```
    doctype html
    html(lang="en")
      head
        // D 코드로부터 pageTitle 값을 받습니다
        title #{pageTitle}
        // 태그의 속성은 이렇게 부여합니다
        script(type='text/javascript')
          if (foo) bar(1 + 5)
        // HTML 엘리먼트 ID는 body-id 로,
        // html 엘리먼트 클래스는 the-style 로 지정합니다
        body#body-id.the-style
          h1 DIET template
```

DIET 템플릿 코드는 파이썬(python) 언어처럼 들여쓰기가 중요합니다. 들여쓰기를 강제한 대신, 닫는 태그는 자동으로 처리됩니다.

DIET 템플릿은 최상의 성능을 위해 메모리에 올릴 수 있게 컴파일된 상태로 이용됩니다. 그리고 DIET 템플릿 내에서 D 언어 코드를 함께 사용할 수 있습니다. `#{ 1 + 1 }` 과 특별한 문법 내에 코드를 감싸서 한 줄짜리 함수 호출의 결과물이나 수식 결과물을 HTML 문서에 함께 출력할 수 있습니다. `#{ }` 를 사용하지 않는 경우, 모든 D 언어 코드는 DIET 내에서 `-` 로 시작되어야합니다.

```
    - foreach(title; titles)
      h1 #{title}
```

`-` 로 시작하는 줄은 D 언어 코드로 처리되며, 여러줄의 코드를 작성할 때 좋습니다. 심지어 이런 식으로 HTML 생성에 도움을 줄 D 언어 함수를 정의하는 것도 가능합니다.

DIET 템플릿 파일들은 **컴파일 시점 반환값 예측(CTFE)** 기법을 적용한 코드로 변환되며, 표준 vibe.d 폴더 구성을 기준으로 했을 때 `views` 폴더 내에 있어야합니다. 템플릿은 그 자체로는 HTML이 될 수 없지만, URL 핸들러 내에서 제공되는 `render` 함수를 사용해 최종적으로 보여질 HTML로 만들어집니다.

```d
    void foo(HTTPServerResponse res) {
        string pageTitle = "Hello";
        int test = 10;
        res.render!("my-template.dt", pageTitle, test);
    }
```

모든 D 언어 값/변수는 `render` 에 사용될 파라미터로 쓰일 수 있습니다.

## {SourceCode:disabled}

```d
doctype html
html
  head
    title DIET 템플릿 테스트
  body
    - import std.algorithm : min;
    p Four items ahead:
    - foreach( i; 0 .. 4 )
      - auto num = i+1;
      p Item
        b= num
    p Prints 8:
    p= min(10, 2*6, 8)
```
