# 가변성(Mutability)

D 언어는 정적 타입 언어(statically typed language)입니다. 변수에 대한 타입이 선언되면 컴파일 시점에 결정되며, 이후에는 바뀔 수 없습니다.

이러한 타입 체계를 통해 프로그램 실행 전에 버그를 예방할 수 있습니다. 그리고 이러한 안전한 타입 체계가 큰 프로그램을 더욱 안전하고 관리하기 용이하게 만드는데 한 몫하고 있습니다.

D 언어의 타입 수식어(type qualifier)에는 몇가지 종류가 있습니다. 가장 많이 이용하는 두 가지는 `const` 와 `immutable` 입니다.

### `immutable`

정적 타입 체계와 더불어, D 언어는 타입 수식어 문법을 통해 변수의 이용에 제약사항을 둘 수 있습니다. 예를 들어 `immutable` 은 변수가 일단 초기화되면 이후로의 변경을 금지하는 제약사항을 줍니다.

    immutable int err = 5;
    // 위의 코드에서, int를 생략하고 immutable err = 5 형태로도 선언할 수 있습니다.
    // 왜냐면 명백하게 어떤 타입인지 컴파일 시점에 알 수 있기 때문입니다.
    err = 5; // immutable 변수를 수정하려 시도합니다. 컴파일 시 오류가 발생합니다.

`immutable` 변수는 프로그램 실행 중에 바뀌지도 않고 바뀔 이유도 없기 때문에 여러 쓰레드에서 접근하더라도 동기화 제어(synchronization)가 필요하지 않습니다. 또한 이러한 특성으로 인해, 완벽히 메모리에 캐시(cache)될 수 있어 프로그램 성능 향상에도 도움을 줍니다.

### `const`

`const` 도 `immutable` 과 같이, '변경을 할 수 없다'는 점에서는 공통이지만 `const` 는 함수 또는 스코프(scope) 내에서만 영향을 줍니다. `const` 특성을 갖는 포인터는 어느 값으로부터 만들어져도 상관없습니다. 이는 현재 `const` 포인터를 사용하려는 함수 또는 스코프(scope) 내에서만 영향을 주고 있고, 원래 변수의 값을 바꿀 수 있는지 여부와는 무관하기 때문입니다.

`const` 로 지정된 구역 외에 존재하는 다른 쓰레드 혹은 함수에서는 원래 변수의 값을 변경할 수도 있습니다. 주의하시기 바랍니다.

API 등에 선언된 함수 매개변수(arguments)부에 `const` 가 이런 특성을 활용해서 많이 쓰이고 있습니다. 프로그래머가 API를 통해 전달해준 값을, API 내부에서 바꾸지 않겠다고 약속해주는 일종의 보증 서약이기 때문입니다.

    void foo(const char[] s)
    {
        // 아래 줄의 주석이 풀려있다면 오류가 발생할 겁니다.
        // 왜냐면 s라는 변수를 const 수식어를 붙여 받았기 때문입니다.
        // s[0] = 'x';

        import std.stdio : writeln;
        writeln(s);
    }

    // `const` 덕분에 아래 두가지 호출 방법 모두 컴파일 가능합니다.
    foo("abcd"); // 이 문자열은 수정될 수 없는 char 배열입니다.
    foo("abcd".dup); // .dup 프로퍼티는 수정될 수 있는 char 배열을 만듭니다.

`immutable` 과 `const` 모두 _이행형(transitive)_ 타입 수식어입니다. 다시 설명하면, 어떤 타입에 이를 적용하게 되면 그 타입 내에서 사용되는 하위 요소들에 대해서도 `const` 나 `immutable` 속성이 적용된다는 것입니다. 이를 통해 다른 함수에 자신의 변수를 안전하게 전달할 수 있습니다.

### 더 살펴보기

#### 기본 참고문서 (Basic references)

- [Immutable in _Programming in D_](http://ddili.org/ders/d.en/const_and_immutable.html)
- [Scopes in _Programming in D_](http://ddili.org/ders/d.en/name_space.html)

#### 심도 있는 참고문서 (Advanced references)

- [const(FAQ)](https://dlang.org/const-faq.html)
- [Type qualifiers in D](https://dlang.org/spec/const3.html)

## {SourceCode}

```d
import std.stdio : writeln;

void main()
{
    /**
    * 변수는 기본적으로 모두 수정 가능 상태입니다.
    * 값을 새로 대입하거나 바꾸어도 괜찮습니다.
    */
    int m = 100; // 값을 바꿀 수 있음
    writeln("m: ", typeof(m).stringof);
    m = 10; // 정상 동작

    /**
    * 값을 바꿀 수 있는 메모리 공간에 대한 포인터:
    */
    // 값을 바꿀 수 있는 메모리 공간에 대한 const 포인터 선언은 유효합니다
    const int* cm = &m;
    writeln("cm: ", typeof(cm).stringof);
    // 앞에서 보셨던 것처럼,
    // `const` 로 받아온 메모리 공간의 값을 수정할 수는 없습니다.
    // *cm = 100; // 주석을 풀어 오류가 발생함을 확인해보십시오.

    // `immutable` 수식어는 프로그램 실행 중 불변을 보증합니다.
    // 따라서 그 값도 바뀔 수 없으므로
    // 값을 바꿀 수 있는 메모리 공간의 주소를 저장해서는 안됩니다.
    // 주석을 풀어 오류가 발생함을 확인해보십시오.
    // immutable int* im = &m;

    /**
    * 읽기만 할 수 있는 메모리 공간에 대한 포인터:
    */
    immutable v = 100;
    writeln("v: ", typeof(v).stringof);

    // v는 immutable 상태입니다.
    // 주석을 풀어 오류가 발생함을 확인해보십시오.
    // v = 5;

    // `const` 포인터를 만들어 immutable 값 주소를
    // 가르킬 수 있습니다. 읽기만 가능합니다.
    const int* cv = &v;
    writeln("*cv: ", typeof(cv).stringof);

    // v는 immutable 상태입니다.
    // 주석을 풀어 대입시 오류가 발생함을 확인해보십시오.
    // *cv = 10;
}
```
