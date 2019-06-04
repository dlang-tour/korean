# 서브타입(Subtyping)

`struct` 는 다른 `struct` 를 확장하여 구현할 수 없습니다.

즉 상속이 불가능하다는 건데, 대신 D 언어에서는 `struct` 의 확장을 위해 상속이 아닌 **서브타입(subtype)** 을 지원하고 있습니다.

`struct` 타입 내의 내부 값들 중 하나에 대해서 `alias this` 라는 정의를 할 수 있습니다.

```d
    struct SafeInt {
        private int theInt;
        alias theInt this;
    }
```

위에서 선언된 `SafeInt` 라는 `struct` 타입 자체에 선언되지 않은 모든 연산이나 내부 함수 호출은 `alias this` 로 지정된 `theInt` 에게 대신 전달됩니다.

`SafeInt` 에 대한 사칙연산 적용이 정의되어있지 않더라도, `theInt` 에게 대신 전달되어 마치 `SafeInt` 를 정수처럼 계산할 수 있게 되는 것입니다. 해당 연산의 순간에는 `SafeInt` 를 그냥 `int` 타입으로 간주하게 됩니다.

이렇게 다른 타입에 새로운 기능을 확장하면서도, 서브타입은 프로그램 실행 성능에 전혀 손해를 끼치지 않습니다. D 언어 컴파일러가 `alias this` 구문이 정확히 동작하도록 컴파일할 것입니다.

`class` 에도 동일하게 적용이 가능합니다.

## {SourceCode}

```d
import std.stdio : writeln;

struct Vector3 {
    private double[3] vec;
    alias vec this;

    double dot(Vector3 rhs) {
        return vec[0]*rhs.vec[0] +
          vec[1]*rhs.vec[1] + vec[2]*rhs.vec[2];
    }
}

void main()
{
    Vector3 vec;
    // double[] 타입이라 생각합시다
    vec = [ 0.0, 1.0, 0.0 ];
    assert(vec.length == 3);
    assert(vec[$ - 1] == 0.0);

    auto vec2 = Vector3([1.0,0.0,0.0]);
    // double[3]로 서브타입을 갖고 있기 때문에 이 연산이
    // 잘 동작합니다.
    writeln("vec dot vec2 = ", vec.dot(vec2));
}
```
