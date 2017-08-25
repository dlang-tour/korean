# 인터페이스(Interfaces)

D 언어의 `interface` 는 실질적으로 `class` 타입과 비슷하지만 다른 특성을 가지고 있습니다. `interface` 를 구현하기로 약속한 클래스는 반드시 `interface` 선언에 명시해둔 내부 함수들을 모두 구현해야합니다. 이를 인터페이스를 구현한다, 또는 인터페이스를 상속한다고 합니다. 인터페이스는 상호간의 약속이기도 합니다.

예를 들어 사람을 비롯하여 여러 동물들이 소리를 낼 수 있다는 걸 인터페이스로 표현해보겠습니다.

```d
    interface Animal {
        void makeNoise();
    }
```

현재 `Animal(동물)` 이라는 `interface` 선언은, 이 인터페이스를 구현할 클래스는 반드시 `makeNoise` 라는 함수를 구현해야한다고 약속하고 있습니다. 예를 들어 개(Dog), 고양이(Cat) 등을 표현하려는 클래스를 만들고자하면, `makeNoise` 를 각 동물에 맞춰 구현해야한다는 것입니다. 이러한 모습은 클래스(Classes) 섹션에서 다루었던, 구현 필요 표시 키워드 `abstract` 를 클래스 내부 함수에 붙였을 때의 동작과 비슷합니다. 클래스에서는 상속할 클래스가 `abstract` 를 구현해줘야했습니다.

```d
    class Dog : Animal {
        override makeNoise() {
            ...
        }
    }

    // 여기서 new Dog 을 했기 때문에, auto 의 타입은 Dog 으로 추론됩니다.
    auto dog = new Dog;

    // 인터페이스를 모두 구현하고 있다면, 인터페이스도 하나의 타입으로써 변수를 선언하는데 사용할 수 있습니다.
    Animal animal = dog; 

    // Dog 클래스는 Animal 인터페이스를 구현하고 있기 때문에, 안심하고 짖는 소리를 낼 수 있습니다.
    animal.makeNoise(); // 멍멍!
```

`class` 간의 상속은 단 하나의 클래스로부터만 상속이 가능합니다만, 인터페이스는 다릅니다. 다양한 인터페이스를 구현하기 위해, 여러개의 `interface` 가 추가될 수 있습니다.

```d
    interface Student {
        void carryBook();
        void openBook();
        void closeBook();
        // ...
    }

    interface Human {
        void sleep();
        void wakeUp();
        void walk(int to);
        // ...
    }

    class Charles : Human, Student {
        // ...
    }
```

### NVI (non virtual interface) 패턴

D 언어의 [NVI 패턴](https://en.wikipedia.org/wiki/Non-virtual_interface_pattern) 지원을 통해, `interface` 의 대부분 함수 선언이 실질적 코드가 없는, 빈 상태(virtual)로 작성되는 것과 달리 `interface` 선언 시에도 코드가 있는 함수를 선언할 수 있습니다.

NVI 패턴은 간단히 말해, 인터페이스를 상속해도 바뀌지 않아야할 함수를 미리 구현해두는 걸 의미합니다. D 언어에서는 이를 함수 인터페이스 선언시 `final` 이라는 키워드를 함께 사용해 이 인터페이스 정의만큼은 다른 함수로 대체하거나 새로 선언할 수 없다고 제한하는 방법으로 구현합니다.

```d
    interface Animal {
        void makeNoise();

        // NVI Pattern으로, makeNoise() 가 어떻게 구현되든지 간에 
        // doubleNoise() 는 makeNoise()를 두 번 호출하는 역할만 수행합니다.
        // 이 interface 를 상속받아도, doubleNoise 의 동작 코드는 변경할 수 없습니다.
        final doubleNoise()
        {
            makeNoise();
            makeNoise();
        }
    }
```

### 더 살펴보기

- [Interfaces in _Programming in D_](http://ddili.org/ders/d.en/interface.html)
- [Interfaces in D](https://dlang.org/spec/interface.html)

## {SourceCode}

```d
import std.stdio : writeln;

interface Animal {
    /*
    구현이 필요한 빈 함수(virtual) 선언입니다.
    */
    void makeNoise();

    /*
    앞에서 살펴본 NVI 패턴을 복습해봅니다.
    makeNoise() 함수 인터페이스를 내부적으로 이용해 다른 동작도 만들어봅시다.
    doubleNoise() 와는 달리, 지정한 회수만큼 makeNoise() 를 반복하도록 수정했습니다.
    Params:
        n =  반복 회수
    */
    final void multipleNoise(int n) {
        for(int i = 0; i < n; ++i) {
            makeNoise();
        }
    }
}

class Dog: Animal {
    override void makeNoise() {
        writeln("멍!");
    }
}

class Cat: Animal {
    override void makeNoise() {
        writeln("냥!");
    }
}

void main() {
    Animal dog = new Dog;
    Animal cat = new Cat;
    Animal[] animals = [dog, cat];
    foreach(animal; animals) {
        animal.multipleNoise(5);
    }
}
```
