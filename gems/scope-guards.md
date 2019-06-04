# 스코프 가드(Scope guards)

스코프(scope)는 어떤 함수의 시작부터 끝, 또는 코드 블럭(중괄호로 감싸져 코드가 작성된 부분)을 의미합니다. 어떤 기능을 하는 코드를 작성할지에 따라 다르지만, 스코프를 벗어날 때 반드시 정리해야할 작업들이 있기 마련입니다. D 언어에서는 스코프 가드를 통해 스코프에 벗어나는 상황에 따라 적절한 코드가 실행되어 마무리 될 수 있도록 합니다.

* `scope(exit)` 선언부는 반드시 스코프를 벗어날 때 호출됩니다
* `scope(success)` 선언부는 아무런 `Exception` 이 발생되지 않고 무사히 실행된 후 스코프에서 벗어나는 경우에 호출됩니다
* `scope(failure)` 선언부에는 스코프에서 벗어나기 전에 `Exception` 이 발생했을 때 호출됩니다.

스코프 가드를 사용해 사용 중 자원 정리 등을 깔끔히 처리할 수 있습니다. 이러한 스코프 가드들이 조건이 충족될 경우(`exit` 은 언제나 호출됨) *반드시* 호출되며, 프로그램이 더욱 안전하게 돌아갈 수 있도록 프로그래머가 코드 작성하는 걸 돕습니다. C++에서 RAII(Resource Acquisition Is Initialization의 줄임말, 자원 확보는 곧 해당 자원의 사용 준비 완료를 의미)를 구현하기 위해 특별한 도구들을 사용했던 복잡함을 D 언어는 스코프 가드로 간편히 대체합니다.

스코프 가드의 실행 순서는, 아래에서 위로, 즉 선언된 순서의 역순입니다.

### 더 살펴보기

- [`scope` in _Programming in D_](http://ddili.org/ders/d.en/scope.html)

## {SourceCode}

```d
import std.stdio : writefln, writeln;

void main()
{
    writeln("<html>");

    // 스코프 가드 실행 순서에 따라,
    // 코드에 가장 먼저 선언되었기에 가장 나중에 호출됩니다.
    scope(exit) writeln("</html>");

    {
        // 여기서부터 또 다른 스코프가 시작되었습니다.
        writeln("\t<head>");
        scope(exit) writeln("\t</head>");
        "\t<title>%s</title>".writefln("Hello");
    }
    // 위에 있는 스코프를 나갈 때,
    // 블럭 내 밑에서 두번째의 scope(exit) 부분이 호출됩니다.

    writeln("\t<body>");

    // 스코프 가드 실행 순서에 따라, 스코프에서 벗어날 때
    // 마지막에서 두번째로 호출됩니다.
    scope(exit) writeln("\t</body>");

    writeln("\t\t<h1>Hello World!</h1>");

    // 스코프 가드 문법 덕분에, 메모리 등의 자원 정리 코드를
    // 반드시 마지막에 작성할 필요 없이, 선언/할당부
    // 바로 밑에 작성할 수 있습니다.
    // 아래 코드에서는 malloc() 으로 받은 메모리가
    // 스코프 종료시 해제될 수 있도록
    // 곧바로 아래에 스코프 가드로
    // 해제 코드를 작성하였습니다.
    import core.stdc.stdlib : free, malloc;
    int* p = cast(int*) malloc(int.sizeof);
    scope(exit) free(p);
}
```
