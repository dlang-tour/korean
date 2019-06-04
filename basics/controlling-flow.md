# 프로그램 흐름 제어(Control flow)

프로그램의 흐름은 `if` 와 `else` 문으로 제어할 수 있습니다.

```d
    if (a == 5) {
        writeln("제 1 조건 충족");
    } else if (a > 10) {
        writeln("제 2 조건 충족");
    } else {
        writeln("막다른 조건에 도달");
    }
```

`if` 나 `else` 내에 단 한 건의 함수 호출 또는 변수 대입이 있다면 중괄호를 생략해 간단히 표기할 수 있습니다. 아래의 예시를 참고해주십시오.

```d
    int a = 5;
    int b;

    if (a == 5)
        a = a + 1;

    if (a == 6) {
        writeln("6 발견!");
        writeln("드디어 발견!");
    }
```

D 언어에서는 C/C++ 이나 Java 에서 값을 비교할 때 사용하던 연산자와 동일한 연산자를 쓰고 있습니다.

* `==` 와 `!=` 는 같음과 같지 않음을 확인할 때 사용합니다.
* `<`, `<=`, `>`, `>=`  는 대소비교를 위해 사용합니다.

여러 조건을 이어 붙여 확인할 때 쓰는 `||` 연산자와 `&&` 연산자가 있는데, `||` 는 'A조건 또는 B조건이 참' 을 표현하고 `&&` 는 `A조건과 B조건 모두 참` 을 표현합니다.

D 언어는 `switch`..`case` 구문을 지원하는데, `switch` 문의 변수를 통해 전달될 수 있는 여러 값에 대해, 일치했을 때 무엇을 할지 정의합니다. `switch` 문에 전달할 수 있는 타입은 D 언어의 모든 기본 타입입니다. 이 기본 타입에는 문자열(string)을 포함합니다. 정수 계통 타입에 대해서는 추가로 `case 시작숫자: .. case 끝숫자:` 와 같은 간편 문법을 지원하여 일일이 나머지 숫자에 대한 `case` 를 작성할 필요가 없도록 해줍니다. 아래 소스코드에 예제를 작성해두었습니다.

### 더 살펴보기

#### 기본 참고문서 (Basic references)

- [Logical expressions in _Programming in D_](http://ddili.org/ders/d.en/logical_expressions.html)
- [If statement in _Programming in D_](http://ddili.org/ders/d.en/if.html)
- [Ternary expressions in _Programming in D_](http://ddili.org/ders/d.en/ternary.html)
- [`switch` and `case` in _Programming in D_](http://ddili.org/ders/d.en/switch_case.html)

#### 심도 있는 참고문서 (Advanced references)

- [Expressions in detail](https://dlang.org/spec/expression.html)
- [If Statement specification](https://dlang.org/spec/statement.html#if-statement)

## {SourceCode}

```d
import std.stdio : writeln;

void main()
{
    if (1 == 1)
        writeln("안심하고 D 언어를 사용하십시오!");

    int c = 5;
    switch(c) {
        case 0: .. case 9:
            writeln(c, " 는 0 또는 9 혹은 그 사이입니다.");
            break;
            // 위의 break를 써두지 않으면 폭포수가 떨어지듯
            // 아래로 이동해 다음 `case` 의 코드를 실행하게 됩니다.
        case 10:
            writeln("10! 영어로 Ten이죠!");
            break;
        default:
            // 위의 `case` 에 일치되는 숫자가 없는 경우
            // 여기에 도달하게 됩니다.
            writeln("별 거 아니네");
            break;
    }
}
```
