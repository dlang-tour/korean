# 예외(Exceptions)

이 섹션에서는 프로그래머가 처리 가능한 `예외(Exception)` 에 대해서만 다룹니다. D 언어에서의 `에러(Error)` 란 대부분 프로그램 동작에 치명적인 문제가 발생한 상태를 의미하며, __예외 처리를 통해 처리하려고 해서는 안됩니다__. `Exception` 는 보통 프로그램이 멈추지 않아도 되는 문제를, `Error` 는 D 언어 실행 환경이나 운영체제 등에 문제가 있어 프로그램이 멈춰야하는 문제를 가르킵니다.

### 예외 처리(Catching Exceptions)

아마도 가장 흔한 예외 처리는 올바르지 않는 사용자 입력에 대한 처리일 겁니다. 예외가 발생한 경우, 최초의 `try - catch` 구문을 만날 때까지 함수 호출 이력을 거슬러 오릅니다. 함수가 호출 될 때마다 쌓이는 이력을 콜 스택(call stack)이라고 부릅니다. 함수 실행이 종료되면 콜 스택에서 그 함수의 이력이 지워집니다.

```d
try
{
    readText("없는 파일");
}
catch (FileException e)
{
    // ...
}
```

`catch` 는 여러번 나타날 수 있습니다. 여러가지 예외가 발생할 수 있기 때문입니다. 그리고 `finally` 구문을 `try - catch` 에 추가해 어떤 상황에서도 반드시 정리하고 가야할 것들(예를 들어 실행시각 로그 남기기 등)을 작성합니다. 예외는 `throw` 를 사용해 프로그래머가 직접 발생시킬 수 있습니다.

```d
try
{
    throw new StringException("문자열 처리 중 예외 발생");
}
catch (FileException e)
{
    // ...
}
catch (StringException e)
{
    // ...
}
finally
{
    // ...
}
```

`catch` 를 사용하지 않고 `try - finally` 로만 구성하는 것도 가능합니다. 다만, 이런 패턴은 심화 과정에서 다룰 [scope guard](gems/scope-guards) 로 처리하는 게 좀 더 깔끔할 때가 많습니다.

### 직접 만드는 예외 클래스(Custom exceptions)

누구든 프로그램 로직상 적절한 예외 발생이 필요하다면 `Exception` 클래스를 상속 받아 새로운 예외를 만들 수 있습니다.

```d
class UserNotFoundException : Exception
{
    this(string msg, string file = __FILE__, size_t line = __LINE__) {
        super(msg, file, line);
    }
}
throw new UserNotFoundException("D맨이 여행갔습니다");
```

### `nothrow` 로 안전 함수 만들기(Enter a safe world with `nothrow`)

D 언어 컴파일러는 `nothrow` 키워드로 표기된 함수가 프로그램 로직 상에서 큰 문제를 일으키지 않을거라고 신뢰합니다. 이 신뢰를 깨지 않기 위해, `nothrow` 로 선언된 함수 내에서는 예외 발생(throwing)을 컴파일 시점에서부터 금지합니다.

```d
bool lessThan(int a, int b) nothrow
{
    // 아무 생각 없이 잘 쓰던 writeln 이지만, 사실 파이프(pipe)나 입출력 버퍼가 깨진 경우에
    // 예외가 발생할 수 있는 함수입니다.
    // 예외가 발생할 수 있기 때문에 nothrow 내에서는 쓸 수 없습니다.
    writeln("unsafe world");
    return a < b;
}
```

템플릿을 이용해 생성된 코드에서도 해당 함수가 예외를 발생시킬지 예측할 수 있습니다.

### 예외 관련 표준 라이브러리(std.exception)

`assert` 구문처럼 주어진 조건이 맞지 않았을 때 프로그램의 동작을 정지시키는 구문은, 릴리스 모드로 배포된 실행파일이나 라이브러리에서는 전혀 동작하지 않습니다. 따라서 `assert` 를 이용해 사용자의 입력을 검증한다는 건 테스트 목적이 아닌 이상 반드시 피해야할 유혹입니다. 하지만 사용자의 입력을 검증하는 것도 번거롭지만 매우 중요합니다. 그래서 D 언어  [`std.exception`](https://dlang.org/phobos/std_exception.html) 라이브러리에서는 `assert` 처럼 간편하게 사용되면서도 릴리스 모드에서도 동작하며, 프로그램 동작 정지 대신 `Exception` 을 발생하여 프로그래머가 처리할 수 있게 해주는 유용한 함수인 [`enforce`](https://dlang.org/phobos/std_exception.html#enforce) 를 제공합니다.

```d
import std.exception : enforce;
float magic = 1_000_000_000;
enforce(magic + 42 - magic == 42, "컴퓨터에서 소수점을 다루는 건 참 복잡미묘하네요");

// 프로그래머가 만들거나 지정한 예외를 발생시킵니다
enforce!StringException('a' != 'A', "대소문자를 구별하는 함수이기 때문에 체크합니다");
```

더 많은 내용이 있지만, 여기에서 다루기에는 분량이 많기에 다루지 않은 부분도 있습니다. 그 중 하나는, 심각하지 않은 `에러(Error)` 에 대해서 에러 발생 여부를 관찰(opt-in)할 수 있게 해주는 [collect](https://dlang.org/phobos/std_exception.html#collectException) 라는 것입니다.

```d
import std.exception : collectException;
auto e = collectException(aDangerousOperation());
if (e)
    writeln("뭔가 에러가 발생해서 실패했습니다. ", e);
```

프로그램 작성 후 유닛 테스트(unit test)를 작성할 때 예외가 발생했는지 체크하려면 [`assertThrown`](https://dlang.org/phobos/std_exception.html#assertThrown) 을 사용하면 됩니다. `std.exception` 내에 있는 함수입니다.

### 더 살펴보기

- [Exception Safety in D](https://dlang.org/exception-safe.html)
- [std.exception](https://dlang.org/phobos/std_exception.html)
- system-level [core.exception](https://dlang.org/phobos/core_exception.html)
- [object.Exception](https://dlang.org/library/object/exception.html) - 프로그래머 try-catch 로 다룰 수 있는 모든 예외들의 기본 클래스입니다

## {SourceCode}

```d
import std.file : FileException, readText;
import std.stdio : writeln;

void main()
{
    try
    {
        readText("dummyFile");
    }
    catch (FileException e)
    {
        writeln("오류 메시지:\n", e.msg);
        writeln("읽으려 했던 파일: ", e.file);
        writeln("행 번호: ", e.line);
        writeln("함수 호출 이력:\n", e.info);

        // 위처럼 원하는 메시지를 출력할 수도 있지만,
        // 그냥 writeln에 입력값으로 e를 던져주기만 해도 기본 형식으로
        // 예외 내용을 출력해줍니다. 주석을 풀어 직접 실행해보세요.
        // writeln(e);
    }
}
```
