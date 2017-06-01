# 기본 자료형

D는 플랫폼에 **관계없이** 항상 동일한 크기를 가진 여러 기본 자료형을 제공합니다. 예외는 가능한 가장 높은 부동 소수점 정밀도를 제공하는 real 자료형입니다. 응용 프로그램이 32 비트 또는 64 비트 시스템 용으로 컴파일되는지 여부에 관계없이 정수의 크기에는 차이가 없습니다.

| 자료형                          | 크기
|-------------------------------|------------
|`bool`                         | 8-bit
|`byte`, `ubyte`, `char`        | 8-bit
|`short`, `ushort`, `wchar`     | 16-bit
|`int`, `uint`, `dchar`         | 32-bit
|`long`, `ulong`                | 64-bit

#### 부동 소수점 자료형:

| 자료형    | 크기
|---------|--------------------------------------------------
|`float`  | 32-bit
|`double` | 64-bit
|`real`   | >= 64-bit (generally 64-bit, but 80-bit on Intel x86 32-bit)

접두사  `u` 는 *부호없는* 자료형을 나타냅니다. `char`는 UTF-8 문자, `wchar`은 UTF-16 문자열, `dchar`는 UTF-32 문자열에서 사용됩니다.

다른 자료형의 변수 간 변환은 정밀도가 손실되지 않을 경우에만 컴파일러에서 허용됩니다. 그러나 부동 소수점 자료형 간의 변환(예 : `double`에서 `float`)은 허용됩니다.

A conversion to another type may be forced by using the
`cast(TYPE) myVar` expression. It needs to be used with great care though,
as the `cast` expression is allowed to break the type system.

The special keyword `auto` creates a variable and infers its
type from the right hand side of the expression. `auto myVar = 7`
will deduce the type `int` for `myVar`. Note that the type is still
set at compile-time and can't be changed - just like with any other
variable with an explicitly given type.

### Type properties

All data types have a property `.init` to which they are initialized.
For all integers this is `0` and for floating points it is `nan` (*not a number*).

Integral and floating point types have a `.max` property for the highest value
they can represent. Integral types also have a `.min` property for the lowest value
they can represent, whereas floating point types have a `.min_normal` property
which is defined to the smallest representable normalized value that's not 0.

Floating point types also have properties `.nan` (NaN-value), `.infinity`
(infinity value), `.dig` (number of decimal digits of precisions), `.mant_dig`
(number of bits in mantissa) and more.

Every type also has a `.stringof` property which yields its name as a string.

### Indexes in D

In D, indexes usually have the alias type `size_t`, as it is a type that
is large enough to represent an offset into all addressable memory - this is
`uint` for 32-bit and `ulong` for 64-bit architectures.

### Assert expression

`assert` is an expression which verifies conditions in debug mode and aborts
with an `AssertionError` if it fails.
`assert(0)` is thus used to mark unreachable code.

### In-depth

#### Basic references

- [Assignment](http://ddili.org/ders/d.en/assignment.html)
- [Variables](http://ddili.org/ders/d.en/variables.html)
- [Arithmetics](http://ddili.org/ders/d.en/arithmetic.html)
- [Floating Point](http://ddili.org/ders/d.en/floating_point.html)
- [Fundamental types in _Programming in D_](http://ddili.org/ders/d.en/types.html)

#### Advanced references

- [Overview of all basic data types in D](https://dlang.org/spec/type.html)
- [`auto` and `typeof` in _Programming in D_](http://ddili.org/ders/d.en/auto_and_typeof.html)
- [Type properties](https://dlang.org/spec/property.html)
- [Assert expression](https://dlang.org/spec/expression.html#AssertExpression)

## {SourceCode}

```d
import std.stdio : writeln;

void main()
{
    // Big numbers can be separated
    // with an underscore "_"
    // to enhance readability.
    int b = 7_000_000;
    short c = cast(short) b; // cast needed
    uint d = b; // fine
    int g;
    assert(g == 0);

    auto f = 3.1415f; // f denotes a float

    // typeid(VAR) returns the type information
    // of an expression.
    writeln("type of f is ", typeid(f));
    double pi = f; // fine
    // for floating-point types
    // implicit down-casting is allowed
    float demoted = pi;

    // access to type properties
    assert(int.init == 0);
    assert(int.sizeof == 4);
    assert(bool.max == 1);
    writeln(int.min, " ", int.max);
    writeln(int.stringof); // int
}
```