# 유니코드와 D 언어(Unicode in D)

유니코드는 컴퓨터에서 문자를 표현하기 위한 국제 표준입니다. 그리고 D 언어는 언어 수준과 라이브러리 수준 모두에서 유니코드를 완벽히 지원합니다.

## 유니코드란 무엇이고 왜 사용하는가(What and Why)

화면으로 보이는 글자들의 뒤에는 컴퓨터가 있습니다. 각 글자들은 여러 비트의 조합으로 만들어지는데 컴퓨터는 각 비트의 상태만을 알지, 글자가 무엇인지는 잘 모릅니다. 컴퓨터가 보기엔 그냥 0과 1로 표기된 숫자이고, 숫자를 그저 처리해줄 뿐입니다.

따라서 사람이 사용하는 문자는 일정 규칙에 따라 부호(code)로 나타나야하고, 이걸 0과 1의 형태로 바꾸거나 다시 글자로 보여주는 과정이 필요합니다.

이런 규칙을 *부호화 체계(encoding scheme)* 이라고 부르고, 유니코드는 수많은 방법 중 하나입니다. 위의 내용이 믿기지 않는다면, 이 섹션에 첨부된 코드로 실습해보시기 바랍니다.

유니코드는 우리에게 알려진 세상에서 쓰이는 모든 문자를 표현하기 위해 특별히 설계되었고, 그 누구라도 유니코드를 사용한다면 문자로 해석해낼 수 있습니다. 하지만 유니코드 이전에는 컴퓨터 제조사마다 다른 인코딩을 사용했고, 타 기종과의 통신에 많은 어려움을 겪었습니다. 개중에는 호환이 되는 경우도 있었지만, 불가능한 경우도 많았습니다. 따라서 기종이 바뀌면 작성된 문서를 전혀 볼 수 없게 되는 문제가 생겼었습니다.

유니코드에 대한 좀 더 기술적인 설명을 보고 싶다면 위키백과에 작성된 문서를 읽어보시기 바랍니다.

## D에서의 유니코드 활용 가이드(How)

이전 문단에서 언급한 복잡한 문제들은 유니코드를 사용함으로써 상당히 해결되었으며, 지금 이용하는 컴퓨터와 OS에서는 대부분이 유니코드를 지원하고 있습니다.

D 언어는 설계 시점부터 유니코드를 다루지 않아 고통 받았던 다른 언어들의 설계 오점을 고려했습니다. 그리고 그 결과로 **모든** 문자열은 D 언어에서 유니코드 체계를 준수하게 되었습니다. C나 C++에서 아직도 문자열은 단순히 바이트(byte)를 담는 그릇에 불과합니다.

문자열 타입 중 `string`, `wstring` 그리고 `dstring` 은 UTF-8, UTF-16, UTF-32 로 표현된 유니코드 문자열을 담는 타입입니다. 이 타입 내의 각 글자는 `char`, `wchar`, `dchar` 타입으로 표현되며 역시 유니코드입니다.

이런 이유로 올바르지 않은 유니코드 문자를 문자열 변수에 저장하는 건 오류로 간주됩니다. 올바르지 않은 문자열의 경우 프로그램 실행 도중 어떤 형태로든 오류가 발생합니다.

하지만 항상 올바른 유니코드 문자만 다룰 순 없습니다. C나 C++에서처럼 단순히 바이트(byte)들을 모아두는 그릇으로써 문자열이 필요하다면, `ubyte[]` 나 `char*` 타입을 활용하시기 바랍니다.

## 범위 탐색 알고리즘과 문자열(Strings in Range Algorithms)

*심화 강좌의 [범위 탐색 알고리즘](gems/range-algorithms) 문서를 미리 읽어보시길 권합니다.*

D 언어가 유니코드를 사용함에 따라 편리한 점도 있지만, 주의할 점도 있음을 기억하십시오.


먼저, 문자열 타입에 담긴 각 문자를 범위 탐색으로 순회하는 경우입니다. 이때 Phobos 표준 라이브러리는 `string` 과 `wstring` 내의 각 문자를 UTF-32 코드포인트로 변환합니다.

**자동 디코딩(auto decoding)** 으로도 잘 알려진 이 과정은 다음과 같은 결과를 낳습니다.

```d
    static assert(is(typeof(utf8.front) == dchar));
```

`utf8.front` 에서 순회하면 당연히 UTF-8 인코딩인 `char` 를 얻을 것 같았지만 사실은 `dchar` 를 받게 되며, `dchar` 는 UTF-32 입니다.

이런 동작에는 수많은 암묵적 약속이 담겨있습니다. 그 중 하나는 `std.traits.hasLength!(string)` 로 문자열의 트레이트를 검증하면 `False` 가 나온다는 것입니다. 당연히 문자 개수를 셀 수 있는데 왜 길이가 없다고 나오는지 의문을 가질 수 밖에 없습니다.

그 이유는 범위 탐색 API와 연관이 있습니다. `string` 의 `length` 프로퍼티는 **문자열 내에 들어있는 구성 원소 수** 를 반환합니다. 일반적인 `.length` 는 범위 탐색 알고리즘이 원소 하나 하나를 탐색했을 때 몇 개나 탐색했는지 카운트한 숫자를 반환합니다.

이해하기 어렵겠지만, 예제를 참고하여 어떤 상황인지 경험해보시기 바랍니다. 그런 특성을 갖고 있기에 범위 탐색 알고리즘은 모두 `string` 의 `.length` 프로퍼티가 없는 상황을 가정하고 동작하는 편입니다.

자동 디코딩(auto decoding)에 대한 자세한 내용과, D 언어로 작성된 프로그램에 미치는 영향은 아래의 "더 살펴보기" 에 정리된 글을 살펴보면 이해하는데 도움을 받을 수 있습니다.

### 더 살펴보기

- [Unicode on Wikipedia](https://en.wikipedia.org/wiki/Unicode)
- [Basic Unicode Functions in Phobos](https://dlang.org/phobos/std_uni.html)
- [Tools for Decoding and Encoding UTF in Phobos](https://dlang.org/phobos/std_utf.html)
- [An in Depth Look at Auto Decoding](https://jackstouffer.com/blog/d_auto_decoding_and_you.html)
- [An in Depth Essay on Benefits of Using UTF-8](http://utf8everywhere.org/)

## {SourceCode}

```d
import std.range.primitives : empty,
    front, popFront;
import std.stdio : write, writeln;

void main()
{
    string utf8 = "å ø ∑ 😦";
    wstring utf16 = "å ø ∑ 😦";
    dstring utf32 = "å ø ∑ 😦";

    writeln("utf8 에서의 길이: ", utf8.length);
    writeln("utf16 에서의 길이: ", utf16.length);
    writeln("utf32 에서의 길이: ", utf32.length);

    foreach (item; utf8)
    {
        auto c = cast(ubyte) item;
        write(c, " ");
    }
    writeln();

    // 범위 탐색 중 각 글자의 타입은 dchar로 처리됩니다.
    // 따라서 UTF-32 코드포인트로 인코딩하기 위해 항상 자기 앞의 바이트가 어떤 형태인지 살펴보는
    // Look-ahead 기법을 사용합니다.
    // 문자열을 구성 중인 바이트를 살펴보려면 간단한 타입 변환이 필요합니다.
    foreach (dchar item; utf16)
    {
        auto c = cast(ushort) item;
        write(c, " ");
    }
    writeln();

    // 자동 디코딩의 결과로 아래 조건이 성립합니다.
    static assert(
        is(typeof(utf8[0]) == immutable(char))
    );
    static assert(
        is(typeof(utf8.front) == dchar)
    );
}
```
