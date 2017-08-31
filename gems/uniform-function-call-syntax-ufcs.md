# 연쇄 함수 호출 문법(Uniform Function Call Syntax (UFCS))

**UFCS** 로 줄여부르는 *연쇄 함수 호출 문법* 은 D 언어의 핵심 기능 중 하나이자, 코드 재사용성(code reusability)과 확장가능성(scalability)을 실현하는 요소입니다.

UFCS를 사용하면 어떤 함수라도 `fun(a)` 형태로 호출하던 함수를 `a.fun()` 형태로 호출할 수 있습니다. 약간 어색할 수 있습니다. 만약 값 `a` 의 원본 타입에 `fun()` 이라는 내부 함수가 정의되어있지 않다면, D 언어는 해당 프로그램 전역에 이용 가능한 함수 중 `fun` 이라는 이름을 가지면서 `a` 타입의 값을 받을 수 있는 함수를 찾아서 적용합니다.

이 문법은 특히 복잡한 함수 호출이 이어질 때, 가독성을 향상시킬 수 있습니다.


```d
    foo(bar(a))
```

이런 일반적 호출을 아래와 같이 바꿔쓰는 게 가능합니다.

```d
    a.bar().foo()
```

게다가 D 언어에서 입력값이 필요 없는 함수는 소괄호 `( )` 조차 생략이 가능합니다. 프로퍼티(property)에 대해 언급하며 한번 간단히 설명한 적이 있습니다. 다시 정리하자면, 소괄호를 생략함으로써 함수를 마치 프로퍼티처럼 다룰 수 있게 되는 것입니다.

```d
    import std.uni : toLower;
    "D rocks".toLower; // toLower는 함수이지만 마치 프로퍼티처럼 쓰입니다. 결과물은 "d rocks" 입니다.
```

UFCS는 *범위 탐색(range)* 를 다루는 동안 여러 알고리즘 함수를 적용해야할 때도 유용하게 쓰입니다.

```d
    import std.algorithm : group;
    import std.range : chain, retro, front, retro;

    // chain() 을 먼저 적용해 [1, 2, 3, 4]를 만듭니다.
    // 다음에 retro([1, 2, 3, 4]) 를 적용해 [4, 3, 2, 1]을 얻습니다.
    [1, 2].chain([3, 4]).retro; // 4, 3, 2, 1

    // front(dropOne(group([1, 1, 2, 2, 2]))) 로 다시 고쳐쓸 수 있습니다만,
    // UFCS의 연쇄 호출 형태가 좀 더 예쁩니다.
    [1, 1, 2, 2, 2].group.dropOne.front; // tuple(2, 3u)
```

### 더 살펴보기

- [UFCS in _Programming in D_](http://ddili.org/ders/d.en/ufcs.html)
- [_Uniform Function Call Syntax_](http://www.drdobbs.com/cpp/uniform-function-call-syntax/232700394) by Walter Bright
- [`std.range`](http://dlang.org/phobos/std_range.html)

## {SourceCode}

```d
import std.stdio : writefln, writeln;
import std.algorithm.iteration : filter;
import std.range : iota;

void main()
{
    "안녕, %s아".writefln("세상");

    10.iota // 0부터 9까지 숫자를 반환합니다
      // 이들 중 짝수를 거릅니다
      .filter!(a => a % 2 == 0)
      .writeln(); // 결과물을 화면에 출력합니다

    // 위의 코드를 다시 고쳐쓰면 아래와 같습니다.
    writeln(filter!(a => a % 2 == 0)
                   (iota(10)));
}
```
