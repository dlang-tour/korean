# 계약에 의한 프로그래밍(Contract programming)

계약, 혹은 약속에 의한 프로그래밍을 하는 기법이 있습니다.

코드를 작성할 때 주어진 입력값에 대해 검증을 하고, 검증된 입력값에 대해 올바른 동작으로 수행하여 계약한대로 결과값을 내놓게 만드는 이 기법을 지원하기 위해 D 언어의 여러 언어요소가 존재합니다. 이런 방법으로 코드를 작상하면 코드의 품질을 향상시킬 수 있습니다.

코드로 표현된 계약 검증 요소들은 테스트와 디버그용으로 컴파일할 때만 기계어로 변환되며, **-release** 옵션을 컴파일러에 주어 릴리즈 모드로 컴파일하게 되면 계약 검증 요소가 모두 제거된 말끔한 릴리즈용 파일이 생성됩니다. 따라서 계약 검증 요소들은 사용자의 입력을 검증하거나 `Exception` 을 대체하여 사용되어서는 안됩니다. 실 배포시에 검증 기능이 동작하지 않기 때문입니다.

### `assert`

D 언어에서 사용할 수 있는 가장 간단한 계약 검증 요소는 `assert(...)` 구문입니다. `assert` 는 구문 내에 입력된 조건이 충족되어 참이 되지 않으면 `AssertionError` 를 발생시켜 프로그램 실행을 중단시킵니다.

```d
    assert(sqrt(4) == 2);
    // AssertionError를 발생시킬 때 표시할 특별한 메시지를 작성할 수 있습니다
    assert(sqrt(16) == 4, "sqrt(16) 이 4가 아닙니다. sqrt 함수가 이상합니다.");
```

### 함수 검증(Function contracts)

`in` 과 `out` 코드 블럭으로 입력값과 반환값이 어떤 약속을 지켜야하는지 정할 수 있습니다. `body` 에 실제 실행될 함수를 작성합니다.

```d
    long square_root(long x)
    in {
        assert(x >= 0);
    } out (result) {
        assert((result * result) <= x
            && (result+1) * (result+1) > x);
    } body {
        return cast(long)std.math.sqrt(cast(real)x);
    }
```

`in` 부분은 사실 실제 실행될 코드가 적힌 함수의 `body` 에 작성해도 됩니다. 하지만 `in` 을 사용하여 입력값에 대한 약속을 검증할 때, 프로그래머가 입력값에 대한 약속을 확인하려는 의도가 더욱 분명히 나타납니다.

`out` 부분을 작성할 때는 함수의 `body` 가 반환할 값을 잡아내기 위해 `out(result)` 형태로 작성합니다. 이렇게 함수의 반환값은 반환과는 별도로 `result` 라는 이름으로 접근할 수 있게 됩니다. 이어서 `result` 를, 약속에 따라 값이 처리되었는지 `out` 코드 블럭에서 검증하면 됩니다.

### 객체 상태 검증(Invariant checking)
`invariant()` 는 `struct` 나 `class` 로부터 만들어진 값이나 객체가, 내부 함수가 호출된 후에도 약속대로 동작하기 위한 정상 상태를 유지하고 있는지 점검하기 위한 특별한 내부 함수입니다.

 * 값이나 객체가 유효하게 메모리에 있는 동안 사용됩니다. 즉, 생성자(constructor)가 호출된 후부터 정리자(destructor)가 호출되기 전까지입니다.
 * `struct` 나 `class` 의 내부 함수가 호출될 때 호출되고, 종료될 다시 `invariant()` 가 호출됩니다. 즉, 함수 하나가 호출되면 총 2회 호출됩니다


### 입력값 검증(Validating user input)

섹션 첫머리에서부터 언급했듯, 이러한 프로그래밍 계약 요소들은 릴리즈 모드로 컴파일될 때 모두 사라집니다. 그리고 `assert` 는 `Exception` 이 아니라 `Error(프로그램 실행 중단)` 를 일으키기 때문에 `nothrow` 로 표시된 함수 내에서도 쓰일 수 있습니다.

그래도 프로그램 실행 중에 사용자 입력을 검증하는 것은 중요합니다. 이런 용도로 릴리즈 모드에서도 `assert` 와 비슷한 기능을 이용하고 싶다면 [`std.exception.enforce`](https://dlang.org/phobos/std_exception.html#.enforce) 를 사용하면 됩니다. `enforce` 는 프로그래머가 처리할 수 있는 `Exception` 을 `Error` 대신 발생시키기 때문에, 입력값이 올바르지 않을 경우의 처리를 해줄 수 있습니다.

### 더 살펴보기

- [`assert` and `enforce` in _Programming in D_](http://ddili.org/ders/d.en/assert.html)
- [Contract programming in _Programming in D_](http://ddili.org/ders/d.en/contracts.html)
- [Contract Programming for Structs and Classes in _Programming in D_](http://ddili.org/ders/d.en/invariant.html)
- [Contract programming in D spec](https://dlang.org/spec/contracts.html)
- [`std.exception`](https://dlang.org/phobos/std_exception.html)

## {SourceCode:incomplete}

```d
import std.stdio : writeln;

/**
예제로 쓰기 위해 매우 단순화시킨 날짜 struct입니다.
실제 날짜를 다룰 때에는 표준 라이브러리의 std.datetime 을 대신 사용하시기 바랍니다.
*/
struct Date {
    private {
        int year;
        int month;
        int day;
    }

    this(int year, int month, int day) {
        this.year = year;
        this.month = month;
        this.day = day;
    }

    invariant() {
        assert(year >= 1900);
        assert(month >= 1 && month <= 12);
        assert(day >= 1 && day <= 31);
    }

    /**
    메모리 상의 Date 타입을 적절한 문자열로 바꿔준 결과물이 있습니다.
    이를 시리얼라이즈(serialize, 직렬화)라고 부릅니다.

    이 함수는 시리얼라이즈된 문자열에서 다시 Date 타입 값을 만듭니다.

    입력값:
        date = 문자열로 표현된 Date 타입 값

    Returns: Date 값
    */
    void fromString(string date)
    in {
        // 입력될 문자열은 총 10글자여야만 합니다
        assert(date.length == 10);
    }
    body {
        import std.format : formattedRead;
        // formattedRead 함수는 형식에 맞춰 문자열을 파싱(parsing)합니다
        // %d 자리를 처리할 땐 숫자가 오는지 보고, - 가 오면 - 문자가 오는지 찾습니다.
        formattedRead(date, "%d-%d-%d",
            &this.year,
            &this.month,
            &this.day);
    }

    /**
    메모리 상의 Date 타입을 적절한 문자열로 바꿔준 결과물이 있습니다.
    이를 시리얼라이즈(serialize, 직렬화)라고 부릅니다.

    이 함수는 Date 를 적절한 문자열 표현으로 바꾸는 시리얼라이즈를 수행합니다.

    반환값: 문자열로 표현된 Date
    */
    string toString() const
    out (result) {
        import std.algorithm : all, count,
                              equal, map;
        import std.string : isNumeric;
        import std.array : split;

        // YYYY-MM-DD 형식으로 정확히 반환했는지 검증합니다
        assert(result.count("-") == 2);
        auto parts = result.split("-");
        assert(parts.map!`a.length`
                    .equal([4, 2, 2]));
        assert(parts.all!isNumeric);
    }
    body {
        import std.format : format;
        return format("%.4d-%.2d-%.2d",
                      year, month, day);
    }
}

void main() {
    auto date = Date(2016, 2, 7);

    // 1년은 12월까지 밖에 없습니다.
    // 이대로 날짜 정보를 인식하면 Date가 지켜야할 불변(invariant) 약속들을 깨는 것입니다.
    // 따라서 `invariant` 가 함수 호출 전후로 호출될 때, 오류가 발생합니다.
    // 단, 릴리스 모드로 컴파일될 때는 검증 기능이 동작하지 않습니다.
    // 절대 사용자 입력을 검증하기 위한 용도로 사용하지 마십시오.
    date.fromString("2016-13-7");

    date.writeln;
}
```
