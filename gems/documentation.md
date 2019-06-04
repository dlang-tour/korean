# 문서화(Documentation)

D 언어는 현대 소프트웨어 공학에서 쓰이는 여러 필수 요소를 언어 내부에 통합하려는 여러 시도를 하고 있습니다.

*계약에 의한 프로그래밍(contract programming)* 이나 *유닛 테스트(unittesting)* 요소도 있지만, D 언어는 컴파일러가 소스 코드내에 작성된 코멘트를 문서로 바꿔주는 [문서화](https://dlang.org/phobos/std_variant.html) 요소까지 지원하고 있습니다.

물론 모든 코멘트가 문서화되는 건 아니며, 규정된 형식을 지켜 작성해주면 `dmd -D` 명령어 한번에 소스코드로부터 웹에 게시 가능한 HTML을 생성해줍니다. 가장 가까운 적용 사례는 [Phobos 표준 라이브러리 문서](https://dlang.org/phobos) 입니다. 이 문서는 *DDoc* 을 사용해 만들어졌습니다.

아래의 코멘트 스타일로 작성하면 DDoc 이 일반 코멘트가 아닌 문서화 요소로 파악하게 됩니다.

* `/// struct, class, function 전에 선언합니다`
* `/++ 특수문자 + 2개로 시작하는 코멘트입니다  +/`
* `/** 특수문자 * 2개로 시작하는 코멘트입니다  */`

아래 예제에서 문서화를 진행할 때 지키면 좋을 모범사례를 살펴볼 수 있습니다.

### 더 살펴보기

- [DDoc design](https://dlang.org/spec/ddoc.html)
- [Phobos standard library documentation](https://dlang.org/phobos)

## {SourceCode:incomplete}

```d
/**
  입력된 0이상의 실수로부터 제곱근을 계산합니다.

  이 문단은 위의 간단할 설명을 좀 더 자세히 설명하여, 공동 작업자들에게
  이 함수가 제곱근을 계산할 수 있고, 어떻게 계산하는지에 대한 자세한 설명을
  덧붙일 수 있습니다.

  Example:
  -------------------
  double sq = sqrt(4);
  -------------------
  Params:
    number = 제곱근 계산을 위한 0 이상의 실수

  License: 어떤 목적으로든 자유롭게 사용 가능합니다
  Throws: 어떤 예외도 발생시키지 않습니다
  Returns: 입력한 숫자의 제곱근을 반환합니다
*/
T sqrt(T)(T number) {
}
```
