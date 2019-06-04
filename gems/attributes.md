# 함수 속성(Attributes)

함수를 작성할 때 어떤 특성을 갖고 동작하게 될지 속성을 지정하는 방법이 여러가지 있습니다.

이 섹션에서는 잘 알려진 두 가지 기본 속성과 *프로그래머 정의 속성(user-defined attributes)* 를 다룹니다.

이미 기본 강좌에서 다룬 적도 있습니다. `@safe`, `@system` 그리고 `@trusted` 를 함수의 안전성이 어느 정도인지 지정할 때 썼습니다.

### `@property`

`@property` 로 지정된 함수는 마치 함수가 프로퍼티처럼 동작하게 합니다.


```d
    struct Foo {
        @property bar() { return 10; }
        @property bar(int x) { writeln(x); }
    }

    Foo foo;
    writeln(foo.bar); // foo.bar(); 와 동일합니다
    foo.bar = 10; // foo.bar(10); 와 동일합니다
```

### `@nogc`

D 언어 컴파일러가 `@nogc` 속성이 부여된 함수를 컴파일할 때, 절대 해당 함수 내에서 **동적 메모리 할당이 발생하지 않도록** 검사합니다. 동적 메모리 할당은 가비지 컬렉터(Garbage Collector)를 통해 이루어지기 때문인데, `@nogc` 는 GC를 사용하지 않는 속성이기 때문입니다.

`@nogc` 함수는 다른 함수를 호출하더라도 반드시 다른 함수도 `@nogc` 속성을 갖고 있어야합니다.

```d
    void foo() @nogc {
      // 동적 메모리 할당이 발견되었으므로, 컴파일시 오류가 발생합니다
        auto a = new A;
    }
```

### 프로그래머 정의 속성(User-defined attributes (UDAs))

D 언어의 어떤 함수나 타입이라도 D 언어에서 기본 제공하는 속성 외에 직접 작성한 속성도 적용할 수 있습니다.

```d
    struct Bar { this(int x) {} }

    struct Foo {
      @("Hello") {
          @Bar(10) void foo() {
            ...
          }
      }
    }
```

타입에 적용할 경우 기본 타입이든 프로그래머가 작성한 타입이든 가리지 않고 적용 가능합니다.

위의 예제를 살펴보면, `foo()` 라는 함수가 `string` 타입의 `"Hello"` 라는 속성을 받았습니다.

그리고 `foo()` 는 추가적으로 `Bar` 라는 속성에 `10` 이라는 값을 담아 받았습니다. 이때 `10` 이라는 값을 얻으려면 D 컴파일러 *trait* 중 `__traits(getAttributes, Foo)` 를 사용해 [`TypeTuple`](https://dlang.org/phobos/std_typetuple.html) 을 받아와야합니다.

컴파일 시 타입에 따른 특별한 동작을 적용할 수 있는 다른 관점을 제공함으로써, 프로그래머 정의 속성을 이용해 코드를 다양한 측면에서 개선할 수 있습니다.

### 더 살펴보기

- [User Defined Attributes in _Programming in D_](http://ddili.org/ders/d.en/uda.html)
- [Attributes in D](https://dlang.org/spec/attribute.html)

