# import 구문과 모듈(Imports and modules)

D 언어의 Hello World 예제를 보셨겠지만, 아주 간단한 예제에서도 `import` 문이 필요했습니다.

`import` 구문은 가져오려는 모듈 내에 존재하는 모든 공개적으로 접근 가능한(public) 함수와 타입/타입 별칭을 현재 파일에서 쓸 수 있게 가져오는데 쓰입니다.

[Phobos](https://dlang.org/phobos/) 라고 불리는 D 언어 표준 라이브러리는 `std` 라는 **패키지** 안에 들어있고, 패키지 내의 모듈들은 `import std.모듈명` 과 같은 형식으로 가져올 수 있습니다.

모듈 내의 일부 요소만 필요하다면, 선택적으로 가져오는 것도 가능합니다. 아래는 예제 코드입니다.

```d
    import std.stdio : writeln, writefln;
```

이런 방식으로 필요한 요소만을 가져오게 되면, 어떤 함수나 타입이 어느 패키지와 모듈에서 온 건지 쉽게 알 수 있으며 다른 패키지와 이름이 겹치는 불상사를 피할 수도 있습니다.

`import` 구문은 반드시 소스 코드 최외곽의 상단에 위치하지 않아도 됩니다. 외부 모듈의 요소가 필요한 함수의 내부나 다른 스코프(scope) 내에서 작성하면 됩니다.

## {SourceCode}

```d
void main()
{
    import std.stdio;
    // 또는 import std.stdio : writeln; 와 같이
    // import할 요소를 지정할 수 있습니다.
    writeln("안녕 세상아!");
}
```
