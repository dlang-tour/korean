# 컴파일 시점 반환값 예측(Compile Time Function Evaluation (CTFE))

CTFE는 **컴파일할 때** 컴파일러가 함수가 어떤 값을 내놓을지 미리 알아내는 메커니즘입니다.

이 메커니즘을 이용하기 위해 특별한 코드를 작성할 필요는 없습니다. D 언어 컴파일러는 컴파일 시점에 알아낼 수 있는 값들을 가능한 파악하여 컴파일 결과물에 반영합니다.

```d
    // 이 함수의 결과물은 너무나 분명합니다.
    // 실행 중에도 sqrt(50)의 값은 바뀌지 않기 때문에 미리 값을 구합니다.
    // 컴파일된 기계어 코드를 살펴보면 sqrt 를 전혀 호출하지 않습니다.
    static val = sqrt(50);
```

`static`, `immutable`, `enum` 과 같은 키워드가 동반되면 컴파일러가 미리 예측할 수 있는 코드에 대하여 CTFE 메커니즘을 적용합니다. 이 메커니즘의 가장 큰 장점은 메커니즘을 위해 코드 일부를 바꾸거나 수정할 필요가 없다는 것입니다. 그냥 짜던 그대로 작성하면 됩니다.

```d
    int n = doSomeRuntimeStuff();
    // 얼핏 보면 비슷해보이지만, 여기서 sqrt(n)의 값은
    // 프로그램이 실행되어야만 예측할 수 있습니다.
    // 그래서 CTFE가 적용되지 않습니다.
    auto val = sqrt(n);
```

가장 이 메커니즘을 잘 활용하는 경우는 D 언어 표준 라이브러리의 [std.regex](https://dlang.org/phobos/std_regex.html) 라는 정규식(Regular Expression)을 다루는 패키지입니다.

`std.regex` 는 `ctRegex` 라는 타입을 제공하는데, `ctRegex` 는 이전 섹션에서 배운 *컴파일 시점 코드 생성(string mixins)* 과 CTFE 메커니즘을 적극적으로 사용합니다. 이 덕분에 문자열을 처리하는데 쓰이는 정규식은 컴파일할 때 최적화된 오토마타(automata) 코드로 변환됩니다. 한편 같은 정규식을 처리하는 `regex` 는 프로그램 실행 중에 정규식을 오토마타(automata) 코드로 해석합니다.

```d
    auto ctr = ctRegex!(`^.*/([^/]+)/?$`);
    auto tr = regex(`^.*/([^/]+)/?$`);
    // ctr과 tr 모두 최종적으로 같은 역할을 수행합니다만,
    // ctr 이 훨씬 빠르게 동작합니다.
```

아직 CTFE 메커니즘으로 최적화되는 D 언어 기능이 그리 많지 않습니다. 하지만 D 언어 컴파일러의 새 버전이 나올수록 더욱 많은 부분에 CTFE 가 적용되어가고 있습니다.

### 더 살펴보기

- [Introduction to regular expressions in D](https://dlang.org/regular-expression.html)
- [std.regex](https://dlang.org/phobos/std_regex.html)
- [Conditional compilation](https://dlang.org/spec/version.html)

## {SourceCode}

```d
import std.stdio : writeln;

/**
뉴턴 근사법을 사용해 임의의 양의 실수로 부터 제곱근을 구해봅시다

Params:
    x = 제곱근을 구할 수

Returns: 제곱근 x
*/
auto sqrt(T)(T x) {
    // epsilon 은 더이상 계산을 반복하지 않아도 되는
    // 시점을 결정해줍니다.
    enum GoodEnough = 0.01;
    import std.math : abs;
    // 적절한 시작값을 고릅니다.
    T z = x*x, old = 0;
    int iter;
    while (abs(z - old) > GoodEnough) {
        old = z;
        z -= ((z*z)-x) / (2*z);
    }

    return z;
}

void main() {
    double n = 4.0;
    writeln("런타임 시점에 계산된 sqrt(4) 는 ",
        sqrt(n));
    static cn = sqrt(4.0);
    writeln("CTFE 메커니즘이 미리 계산한 sqrt(4) 는 ",
        cn);
}
```
