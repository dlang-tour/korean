# 유닛 테스트(Unittesting)

프로그램 작성시 테스트는 버그 없는 안정적인 동작을 완성하는 훌륭한 방법 중 하나입니다.

잘 작성된 유닛 테스트 코드는 마치 코드처럼 실행되는 레퍼런스 문서 같아서, 어떤 기능을 수정할 때도 테스트만 잘 통과하면 되기 때문에 신경 써야할 부분을 많이 줄여줍니다.

D 언어는 `unittest` 라는 예약어와 이에 따르는 코드 블럭 문법을 제공하며, 소스 내 어느 부분에서도 해당 기능을 테스트할 수 있는 코드를 선언할 수 있습니다.

```d
    // unittest 코드 블럭 예시입니다
    unittest
    {
        assert(myAbs(-1) == 1);
        assert(myAbs(1)  == 1);
    }
```

이를 통해 필요한만큼 직관적으로 [테스트 중심 개발](https://en.wikipedia.org/wiki/Test-driven_development) 철학을 실천할 수 있습니다.

### 유닛 테스트 수행하기(Run & execute `unittest` blocks)

`unittest` 코드 블럭에는 DMD(D 언어 표준 컴파일러)에 `-unittest` 옵션이 전달되었을 때 실행될 그 어떤 코드라도 작성할 수 있습니다. 작성된 D 언어 코드가 DUB 패키지 형태의 구조를 갖고 있다면, `dub test` 명령을 실행하는 것만으로 일련의 테스트 과정을 자동으로 실행해줍니다.

### `assert` 로 검증하기(Verify examples with `assert`)

보통의 `unittest` 코드 블록 내에는 `assert` 표현이 많이 작성되며, 이를 통해 테스트하려는 함수나 클래스가 의도대로 동작하는지 검증합니다. `unittest` 코드 블럭을 두는 위치로는 함수 실제 구현부가 아닌, 함수 선언 부분의 근처가 선호됩니다. 그렇기에 보통 소스 코드의 상단부에 유닛 테스트를 위한 코드가 몰려있습니다. `class` 나 `struct` 내의 기능을 테스트하는 코드는 그 안에 선언하기도 합니다.

### 테스트 중 실행된 코드량을 늘리기(Increasing code coverage)

유닛 테스트는 튼튼한 프로그램을 작성하기 위한 훌륭한 도구이지만, 얼마나 프로그램이 테스트되었는지는 어떻게 표현되는지 궁금할 수 있습니다.

많이 사용되는 기준은 _코드 커버리지(Code coverage)_ 인데, 간단히 설명하면 전체 프로그래머가 작성된 코드 줄 수 중에 유닛 테스트에 의해 실행된 코드가 어느 정도인지 그 비율을 계산하는 것입니다. DMD 컴파일러는 `-cov` 옵션을 추가하는 것만으로 간단히 코드 커버리지를 측정할 수 있으며, 각 파일마다 실행 결과 보고서가 담긴 `.lst` 파일이 생성됩니다.

DMD 컴파일러는 `unittest` 코드 블럭에 어노테이션을 선언하는 템플릿 코드를 인식합니다. 그래서 테스트용 코드에 몇가지 어노테이션을 추가할 수도 있습니다.

```d
    unittest @safe @nogc nothrow pure
    {
        assert(myAbs() == 1);
    }
```

### 더 살펴보기

- [Unit Testing in _Programming in D_](http://ddili.org/ders/d.en/unit_testing.html)
- [Unittesting in D](https://dlang.org/spec/unittest.html)

## {SourceCode}

```d
import std.stdio : writeln;

struct Vector3 {
    double x;
    double y;
    double z;

    double dot(Vector3 rhs) const {
        return x*rhs.x + y*rhs.y + z*rhs.z;
    }

    // struct 내에 선언하는 것 또한 괜찮다고 앞에서 설명했습니다
    unittest {
        assert(Vector3(1,0,0).dot(
          Vector3(0,1,0)) == 0);
    }

    string toString() const {
        import std.string : format;
        return format("x:%.1f y:%.1f z:%.1f",
          x, y, z);
    }

    // toString() 을 검증하기 위한 코드, 역시 좋습니다
    unittest {
        assert(Vector3(1,0,0).toString() ==
          "x:1.0 y:0.0 z:0.0");
    }
}

void main()
{
    Vector3 vec = Vector3(0,1,0);
    writeln(`이 Vector3은 테스트되었습니다: `,
      vec);
}

/*
색다른 곳에 선언된 unittest 코드 블럭입니다. (물론 유효한 테스트입니다)

DMD 컴파일러에 -unittest 옵션을 주거나,
dub test 를 실행하지 않는 한 모든 테스트는 컴파일되지 않습니다.

역으로, 테스트를 하고 싶다면
그런 옵션을 주거나 dub test를 사용해야한다는 것입니다.
*/
unittest {
    Vector3 vec;
    // .init 는 어떤 타입의 초기값이 무엇인지 확인하기 위한
    // 프로퍼티입니다. 잘 기억나지 않는다면 기본 강좌의
    // 기본 타입(Basic types) 섹션을 복습하십시오.
    assert(vec.x == double.init);
}
```
