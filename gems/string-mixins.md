# 컴파일 시점 코드 생성(String Mixins)

`믹스인(mixin)` 구문은 임의의 문자열을 받아 컴파일 시점에 코드로 변환하여 컴파일하는 구문입니다.

`mixin` 은 **컴파일 시점** 에서만 동작하며, 컴파일 시점에 완전한 코드로 인식되는 문자열에 대해서만 코드로 변환하는 메커니즘을 적용합니다. 한번 코드가 컴파일되어 실행파일을 얻은 후에는, 바꾸기 위해 다시 컴파일해야합니다.

첫문장을 보고, 동적으로 입력을 받아 코드로 바꿔 실행하는 JavaScript의 _사악한_ `eval` 함수를 떠올리셨다면 심히 곤란합니다.

```d
    mixin("int b = 5");
    assert(b == 5); // 마치 그냥 int b = 5 를 코드로 쓴 것처럼 컴파일됩니다
```

`mixin` 은 컴파일 이후에 바뀌지 않을 문자열 연산의 결과물에 대해서도 동작합니다. 예를 들자면 템플릿 파라미터(template parameter)로 입력되어 컴파일 시점에 어떤 문자열이 나올지 예상되는 입력에 대해서 알아서 처리하는 걸 들 수 있습니다.

`mixin` 은 이후 섹션에서 다룰 **컴파일 시점 반환값 예측(CTFE, Compile Time Function Evaluation)** 기법을 이용한, [Pegged](https://github.com/PhilippeSigaud/Pegged) 와 같은 인상적인 라이브러리를 작성하게 해주었습니다. Pegged 는 소스 코드 내에 정의된 문자열을 처리해 문법 파서(parser)를 만들어주는 라이브러리 입니다.

### 더 살펴보기

- [Mixins in D](https://dlang.org/spec/template-mixin.html)

## {SourceCode}

```d
import std.stdio : writeln;

auto calculate(string op, T)(T lhs, T rhs)
{
    return mixin("lhs " ~ op ~ " rhs");
}

void main()
{
    // mixin 덕분에 완전히 새로운 방법으로 Hello World를 표시합니다
    mixin(`writeln("Hello World");`);

    // 템플릿 파라미터로 연산자와 입력값을 함께 전달합니다
    // 각 입력값들은 모두 상수(constant)로,
    // 컴파일이 되어도 바뀌지 않습니다
    // 따라서 D 언어 컴파일러는 mixin 내에서
    // 어떤 문자열이 만들어질지 예측할 수 있기에
    // 정상적으로 컴파일됩니다.
    writeln("5 + 12 = ", calculate!"+"(5,12));
    // 5 + 12 와 같음

    writeln("10 - 8 = ", calculate!"-"(10,8));
    // 10 - 8 과 같음

    writeln("8 * 8 = ", calculate!"*"(8,8));
    // 8 * 8 과 같음

    writeln("100 / 5 = ", calculate!"/"(100,5));
    // 100 / 5 와 같음
}
```
