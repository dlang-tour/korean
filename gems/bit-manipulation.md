# 비트 연산(Bit manipulation)

믹스인(mixin)을 가장 잘 활용하여 D 언어의 기능을 충분히 끌어내는 경우의 예로 비트 연산을 들 수 있습니다.

### 간단한 연산(Simple bit manipulation)

D 언어에서 제공하는 기본 비트 연산자들입니다.

- `&` 비트 연산용 and
- `|` 비트 연산용 or
- `~` 비트 연산용 negative
- `<<`  비트 연산용 산술(원래 값의 부호가 유지됩니다) 좌향 shift
- `>>`  비트 연산용 산술(원래 값의 부호가 유지됩니다) 우향 shift
- `>>>` 비트 연산용 부호 없는 우향 shift

### 실질적 예시(A practical example)

비트 연산을 쓰는 대부분의 경우는 어떤 값의 특정 비트가 어떤 상태인지 확인하는 일인데, 내장 라이브러리인 `core.bitop.bt` 를 사용하면 처리할 수 있습니다.

그러나, 이번 섹션에서는 번거롭지만 라이브러리 없이 직접 다뤄보겠습니다.

```d
enum posA = 1;
enum maskA = (1 << posA);
bool getFieldA()
{
    return _data & maskA;
}
```

위의 예제는 단 한자리의 비트만 확인할 수 있었지만, 일반화하여 연속된 자릿수의 비트를 읽어봅시다.

읽은 부분만을 추출해야 하기 때문에 자릿수 구간을 나타내줄 특별한 마스크(mask)가 필요합니다. 이 마스크는 읽지 않아야할 부분을 0으로 만들어줄 것입니다.

그리고 원하는 값만 읽기 위해 결과물에서 우측에 남아있는 불필요한 비트를 제거할 필요가 있습니다. (좌측은 0으로 채워집니다)

```d
enum posA = 1;
enum lenA = 3;
enum maskA = (1 << lenA) - 1; // ...0111 인데, & 연산을 하면 _data에서 1로된 부분만 1로 남습니다
uint getFieldA()
{
    return (_data >> posA) & maskA;
}
```

역으로, `_data` 의 특정 비트 자릿수 구간에 원하는 비트를 설정하는 건 마스크를 반전시켜 비트를 설정할 부분을 찾아내는 것입니다.

```d
void setFieldA(bool b);
{
    return (_data & ~maskAWrite) | ((b << aPos) & maskAWrite);
}
```

## 도와줘요 `std.bitmanip` ! (`std.bitmanip` to the rescue)

자기만의 특별한 비트 연산 함수를 정의하고 이용하는 건 재밌는 일이고, D 언어는 이를 위한 충분한 기능을 제공하고 있습니다.

다만, 이렇게 비트 연산 처리하는 코드를 단순히 복사해서 붙여넣어 이용하는 건 코드 유지보수 측면에서 불리할 뿐더라 오류 발생의 원인이 됩니다.

`std.bitmanip` 표준 라이브러리를 이용해 유지보수하기 편리하면서도 코드 가독성을 높인, 하지만 성능에서 전혀 손해보지 않는 비트 연산 코드를 작성해보십시오.

실습 화면을 보십시오. `BitVector` 가 `struct` 로 정의되어있고 다룰 때도 `struct` 처럼 다루면 되지만 실제론 내부에서 비트필드(bitfield)를 이용한 비트 연산이 이루어지고 있습니다.

`std.bitmanip` 와 `core.bitop` 에는 메모리가 극단적으로 부족해 비트 단위로 메모리를 자주 다뤄야할 응용 프로그램을 위한 유용한 도구들을 제공하고 있습니다.

### 메모리 공간 상의 자리채움과 배치(Padding and alignment)

OS 커널이 다루는 메모리 최소 단위(`size_t.sizeof`)보다 더 작은 공간을 차지하는 값을 다루는 경우에, D 언어 컴파일러는 OS 커널에서 처리할 수 있게 원본 값에 더미(dummy) 값을 더 붙여 메모리 최소 단위를 맞춥니다. 만약 4바이트(32비트)를 한 단위로 사용하는 OS라면, `bool`, `byte`, `char` 와 같은 타입을 다룰 때 이런 일이 발생하게 됩니다.

이렇게 되면 자칫 소중한 메모리가 낭비될 수 있으므로 효율적으로 잘 배치될 수 있도록 신경써주는 편이 좋습니다. 예제에서 `struct BadVector` 의 경우를 자세히 보십시오.

## 더 살펴보기

- [std.bitmanip](http://dlang.org/phobos/std_bitmanip.html) - Bit-level manipulation facilities
- [_Bit Packing like a Madman_](http://dconf.org/2016/talks/sechet.html)

## {SourceCode}

```d
struct BitVector
{
    import std.bitmanip : bitfields;
    // 직접 해당 타입으로 변수에 저장하기 전에,
    // 프록시(proxy)를 거쳐 처리되도록 합니다.
    // 믹스인 덕분에 컴파일될 때 각각 x, y, z, flag 변수로 바뀝니다
    mixin(bitfields!(
        uint, "x",    2, // 2비트
        int,  "y",    3, // 3비트
        uint, "z",    2, // 2비트
        bool, "flag", 1)); // 1비트 - 여기까지 해서 총 8비트, 1바이트
}

void main()
{
    import std.stdio : writefln, writeln;

    BitVector vec;
    vec.x = 2;
    vec.z = vec.x - 1;
    writefln("x: %d, y: %d, z: %d",
              vec.x, vec.y, vec.z);

    // 오직 1바이트(8비트)만을 사용합니다
    writeln(BitVector.sizeof);

    struct Vector { int x, y, z; }
    // 위의 Vector는 int를 사용한 벡터입니다.
    // 각 변수 하나당 4바이트를 차지하므로 총 12바이트가 소모됩니다
    writeln(Vector.sizeof);

    struct BadVector
    {
        bool a;
        int x, y, z;
        bool b;
    }
    // 총 14바이트가 필요할 것 같지만, OS의 최소 메모리 단위로 인해
    // bool a는 1바이트만 필요하지만 4바이트가 소모되며,
    // bool b도 1바이트만 필요하지만 4바이트가 소모됩니다.
    // 따라서 a, x, y, z, b 모두 4바이트를 차지해 메모리가 낭비되었습니다
    // 좋지 않은 선언의 예입니다
    writeln(BadVector.sizeof);
}
```
