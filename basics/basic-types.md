# 기본 타입(Basic types)

D 언어는 운영체제나 플랫폼의 영향을 받지 않는 **일관된** 다양한 기본 타입을 제공합니다.

단, `real` 이라고 불리는 높은 정확도를 보여주는 부동소수점 타입은 예외입니다. 사용에 있어 주의가 필요하며, 이는 운영체제나 플랫폼에 따라 영향을 받을 수 있습니다.

정수형 계통의 타입은, 32비트 CPU용과 64비트 CPU용 컴파일러 모두 **일관된** 메모리상의 크기로 지정되어 있습니다.

| 타입                          | 메모리상의 크기
|-------------------------------|------------
|`bool`                         | 8비트
|`byte`, `ubyte`, `char`        | 8비트
|`short`, `ushort`, `wchar`     | 16비트
|`int`, `uint`, `dchar`         | 32비트
|`long`, `ulong`                | 64비트

#### 부동 소수점 타입(Floating point types):

| 타입    | 메모리상의 크기
|---------|--------------------------------------------------
|`float`  | 32비트
|`double` | 64비트
|`real`   | 64비트 이상일 수 있음 (인텔 x86 32비트 CPU에서 80비트가 필요)

일부 타입의 앞에 붙은 `u` 라는 알파벳은 *부호가 없는 (unsigned)* 타입임을 명시합니다.

`char` 타입의 경우 UTF-8 인코딩에 해당하는 문자로 변환되며, `wchar`는 UTF-16, `dchar`는 UTF-32 문자를 보관하게 됩니다.

타입 간의 변환은 숫자나 값의 손실이 발생하지 않는 경우에만 허용됩니다. 같은 계통이면서 크기가 같거나 더 큰 타입으로의 변환은 괜찮지만, 크기가 더 작거나 완전히 다른 타입일 경우 문제가 발생합니다. 그래서 강제로 변환하는 과정이 필요할 수 있습니다. 그러나 부동소수점 타입끼리는 자유롭게 변환할 수 있으며, 자동으로 변환됩니다. (예를 들어 `double` 에서 `float`으로의 대입)

다른 타입으로 변환시 `cast(타입명) 변수명` 구문을 통해 강제로 변환할 수 있습니다. 다만, `cast` 구문에 의해 D 언어가 유지하고 있는 타입 규칙이 깨질 수 있기 때문에 많은 주의가 필요합니다.

`auto` 라는 특별한 타입 키워드는 변수에 대입하려는 값이나 변수로부터 타입을 자동으로 유추하여 타입을 설정하는데 쓰입니다. `auto myVar = 7` 과 같은 선언에서, 등호(=) 우측에 입력된 값이 정수 7이므로 컴파일러는 int라는 타입을 유추할 수 있습니다. 단, `auto` 는 컴파일 시점에 모든 걸 파악하고 실행시점에서 auto 때문에 유추된 타입은 바뀌지 않습니다. 마치 `int myVar = 7` 을 대신 사용한 것과 같아집니다.

### 타입 프로퍼티(Type properties)

자료를 담을 수 있는 타입은 모두 `.init` 라는 프로퍼티를 통해 값이 초기화됩니다.

정수형 계통 타입은 모두 `.init` 를 통해 `0` 으로 초기값이 설정되고, 부동 소수점 계통 타입은 `nan(*Not a number*, 수가 아님)` 으로 설정됩니다.

정수형 계통 타입과 부동 소수점 계통의 타입은 표현 가능한 최대값을 알려주는 `.max` 라는 프로퍼티가 있습니다.

반대로 정수형 계통 타입의 표현 가능한 최소값은 `.min` 이라는 프로퍼티를 통해 확인할 수 있습니다. 한편 부동 소수점 계통 타입에서의 표현 가능한 최소값은 정규화(normalized)된 최소값 프로퍼티인 `.min_normal` 을 사용해 확인해야합니다. 이는 부동 소수점 표현의 특성 때문에 그렇습니다.

부동 소수점 계통 타입들은 또한 `.nan` (NaN-value) 프로퍼티, `.infinity` (무한대) 프로퍼티, `.dig` (십진법 표기시 소수점 정확도 자리수) 프로퍼티, `.mant_dig` (가수부의 비트수) 등의 프로퍼티도 가지고 있습니다. 지수부, 가수부 표현에 대해서는 부동 소수점 표준안 IEEE 754 문서를 살펴보시기 바랍니다.

모든 타입은 공통적으로 `.stringof` 라는 프로퍼티를 가지고 있습니다. 이를 통해 해당 타입 이름을 문자열로 받을 수 있습니다.

```d
    int.stringof // -> "int"
```

### D 언어에서의 배열 접근

D 언어에서는 `size_t` 라고 별칭이 붙은 타입을 사용해 0부터 시작하는 배열 접근 인덱스를 씁니다. `size_t` 는 이용 가능한 메모리의 전체 영역의 크기에 따라 달라집니다. 32비트 CPU 환경에서는 `uint` 타입을 `size_t` 로 쓰고 64비트 CPU 환경에서 64비트용으로 컴파일하게 되면 `ulong` 타입을 사용합니다.

### 단언(Assert expression)

`assert` 는 *디버그 모드* 에서 주어진 식(expression)을 실행하고, 그 결과값이 true 혹은 true에 준하는 값이라면 그냥 통과하지만 false 혹은 false에 준하는 값이면 `AssertionError` 를 일으키며 프로그램 실행을 중단시킵니다. 프로덕션 모드로 컴파일하게 되면 `assert` 는 코드 실행에 아무런 영향을 주지 않습니다.

이럴 원리를 이용해 `assert(0)` 과 같은 방식으로, 어떻게 해도 실행될 수 없는 코드 위치를 나타내기도 합니다.

### 더 살펴보기

#### 기본 참고문서 (Basic references)

- [Assignment](http://ddili.org/ders/d.en/assignment.html)
- [Variables](http://ddili.org/ders/d.en/variables.html)
- [Arithmetics](http://ddili.org/ders/d.en/arithmetic.html)
- [Floating Point](http://ddili.org/ders/d.en/floating_point.html)
- [Fundamental types in _Programming in D_](http://ddili.org/ders/d.en/types.html)

#### 심도 있는 참고문서 (Advanced references)

- [Overview of all basic data types in D](https://dlang.org/spec/type.html)
- [`auto` and `typeof` in _Programming in D_](http://ddili.org/ders/d.en/auto_and_typeof.html)
- [Type properties](https://dlang.org/spec/property.html)
- [Assert expression](https://dlang.org/spec/expression.html#AssertExpression)

## {SourceCode}

```d
import std.stdio : writeln;

void main()
{
    // 큰 숫자를 적을 때 "_" 로 천단위나 만단위를 구별해주면
    // 한 눈에 들어와 코드 읽기에 편리해집니다.
    int b = 7_000_000;

    // 타입 변환이 필요합니다.
    // 표현 범위가 달라 정확도에 손실이 발생했을 수도 있습니다
    short c = cast(short) b;
    uint d = b; // int와 uint 모두 표현 가능한 범위입니다
    int g;
    assert(g == 0);

    // 숫자 끝에 f를 붙인 건 float 타입으로
    // 취급해달라고 컴파일러에게 요청하는 것입니다.
    // 덕분에 컴파일러는 auto가 float을 가르키고
    // 있음을 쉽게 유추했습니다.
    auto f = 3.1415f;

    // typeid(VAR) 로 VAR가 어떤 타입인지 알 수 있습니다.
    writeln("type of f is ", typeid(f));

    // double 타입의 표현 범위가 훨씬 넓습니다.
    double pi = f;

    // 앞에서 설명했듯, double과 float 상호 간에는
    // 강제적 타입 변환 없이 자동 변환이 이루어집니다.
    float demoted = pi;

    // 타입 프로퍼티의 값이 무엇인지 확인해봅시다.
    assert(int.init == 0);
    assert(int.sizeof == 4);
    assert(bool.max == 1);
    writeln(int.min, " ", int.max);
    writeln(int.stringof); // int
}
```
