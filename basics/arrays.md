# 배열

D에는 **static** 와 **dynamic** 등 두 종류의 배열이 있습니다.
배열에 접근할때는 종류에 상관없이 경계검사(bounds-check)가 수행됩니다.(컴파일러가 경계검사의 불필요성을 증명 할 수 있는 경우는 제외)
경계검사에 실패하면 `RangeError`를 발생시켜 응용프로그램을 중단합니다.
용감한 분들은 다소 안전성을 해치더라도 속도개선을 위해 컴파일러 플래그를 `-boundschecks=off`로 설정할 수 있습니다.

#### 정적(Static) 배열

정적 배열은 함수 내에서 정의된 경우 스택에, 그렇지 않으면 정적 메모리에 저장됩니다.
이것은 컴파일할때 고정된 길이값을 가지고 있습니다.
정적 배열의 타입은 고정크기값을 가지고 있습니다 :

    int[8] arr;

`arr`'의 타입은 `int[8]` 입니다. 배열의 길이를 C/C++과 달리 변수옆이 아닌 타입 옆에 표시한다는 것에 주의하세요.

#### 동적(Dynamic) 배열

동적 배열은 힙(heap)에 저장되어 런타임 중에 확장되거나 축소될 수 있습니다. 동적 배열은 `new`라는 표현식과 길이값으로 생성합니다 :

    int size = 8; // run-time variable
    int[] arr = new int[size];

`arr`의 타입은 `int[]` 이고 이것은 **slice** 입니다. 슬라이스는 [다음장](basics/slices)에서 더 자세히 설명됩니다. 다차원 배열은 `auto arr = new int[3][3]` 구문과 같이 쉽게 만들수 있습니다.

#### 배열 연산과 속성

 
배열은 `~`연산자를 이용해 연결될 수 있습니다. 이때 새로운 동적 배열이 생깁니다.

수학적 연산은 예를 들어 `c[] = a[] + b[]` 구문과 같이 모든 배열에 적용할 수 있습니다. 이것은 `a`와`b`의 모든 요소를`c [0] = a [0] + b [0]``c [1] = a [1] + b [1]`등과 같이 서로 더합니다. 또한 단일 값을 배열과 연산할 수 도 있습니다 :

    a[] *= 2; // 모든 요소에 2를 곱한값
    a[] %= 26; // 모든 요소의 26에 대한 나머지값 계산

컴파일러는 이러한 연산들을 한번에 수행할 수 있는 특별 프로세서 명령어들을 사용하여 최적화 할 수도 있습니다.


, 동적 배열 모두 `.length`속성을 가지고 있는데 이는 정적 배열에서는 읽기 전용값이지만 동적 배열에서는 그 크기를 동적으로 변경하는데 사용될 수 있습니다. `.dup`속성은 배열의 복사본을 만드는데 사용됩니다.

`arr [idx]`구문으로 배열을 색인(indexing) 할 때 특수`$`기호는 배열의 길이를 나타냅니다.
예를 들어,`arr [$ - 1]`는 마지막 요소를 참조하는 `arr [arr.length - 1]`의 축약형 입니다.

### 연습

비밀 메시지를 해독하기 위해 `encrypt` 함수를 완성해야 합니다.
주어진 텍스트를 특정 인덱스를 사용하여 알파벳 문자를 이동하는 방법인 *Caesar encryption*를 사용하여 암호화합니다.
암호화할 (to-be-encrypted) 텍스트가`a-z`범위의 문자 만 포함하기 때문에 어렵지 않을 것입니다.

### 더 깊은 내용

- [Arrays in _Programming in D_](http://ddili.org/ders/d.en/arrays.html)
- [Array specification](https://dlang.org/spec/arrays.html)

## {SourceCode:incomplete}

```d
import std.stdio : writeln;

/**
Shifts every character in the
array `input` for `shift` characters.
The character range is limited to `a-z`
and the next character after z is a.

Params:
    input = array to shift
    shift = shift length for each char
Returns:
    Shifted char array
*/
char[] encrypt(char[] input, char shift)
{
    auto result = input.dup;
    // ...
    return result;
}

void main()
{
    // We will now encrypt the message with
    // Caesar encryption and a
    // shift factor of 16!
    char[] toBeEncrypted = [ 'w','e','l','c',
      'o','m','e','t','o','d',
      // The last , is okay and will just
      // be ignored!
    ];
    writeln("Before: ", toBeEncrypted);
    auto encrypted = encrypt(toBeEncrypted, 16);
    writeln("After: ", encrypted);

    // Make sure we the algorithm works
    // as expected
    assert(encrypted == [ 'm','u','b','s','e',
            'c','u','j','e','t' ]);
}
```
