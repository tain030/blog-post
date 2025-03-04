---
title: 'Rust: 참조를 해제하는 두 가지 방법'
date: '2025-02-16'
category: 'rust'
---

### 목차

1. 패턴 매칭을 이용한 참조 해제 (`&` 패턴)
  - 패턴 매칭으로 참조 해제가 불가능한 경우
2. `*` 연산자를 이용한 명시적인 참조 해제
  - 왜 튜플 내부에서는 `*` 연산자가 허용될까?
  - Deref 트레이트와 `*` 연산자
3. 두 가지 방법 비교

---

https://blog.kakaocdn.net/dn/rAIay/btszbtfDapI/MkIZTYFnJYr9JuOVm5smgK/img.jpg

Rust에서는 참조(reference)를 해제하여 실제 값을 가져오는 방법이 두 가지 있습니다.

1. **패턴 매칭을 이용한 참조 해제 (`&` 패턴)**
2. **`*` 연산자를 이용한 명시적인 참조 해제**

각 방법의 차이점과 활용 방법을 비교하여, 어떤 상황에서 어떤 방법을 사용하는 것이 적절한지 살펴보겠습니다.


## 1. 패턴 매칭을 이용한 참조 해제 (`&` 패턴)

패턴 매칭을 이용하면 `&`를 사용하여 참조를 해제할 수 있습니다. 이는 `match` 구문뿐만 아니라 클로저에서도 유용하게 사용됩니다.

```rust
fn main() {
    let value = 42;
    let reference = &value;

    match reference {
        &v => println!("참조 해제된 값: {}", v),
    }
}
```

위 코드에서 `&v` 패턴을 사용하여 `&i32` 타입에서 `i32` 값을 추출했습니다.

클로저에서도 `|&x|` 패턴을 사용할 수 있으며, 이는 `iter()`를 사용할 때 특히 유용합니다.

```rust
let numbers = vec![10, 20, 30];
let doubled: Vec<i32> = numbers.iter().map(|&x| x * 2).collect();

assert_eq!(doubled, vec![20, 40, 60]);
```

이처럼 `|&x|` 패턴을 사용하면 `&i32` 타입의 값을 직접 `i32`로 변환하여 코드의 가독성을 높일 수 있습니다.

### 패턴 매칭으로 참조 해제가 불가능한 경우

하지만 **튜플 내부의 요소를 직접 참조 해제하려고 하면 오류가 발생**합니다.

```rust
let map = std::collections::HashMap::from([(1, 10), (2, 20)]);
let max = 20;

// ❌ 컴파일 오류 발생
let result: Vec<(_, _)> = map.iter().filter(|(_, &v)| v == max).collect();
```

Rust에서는 **튜플 내부에서 참조를 해제하는 패턴을 허용하지 않기 때문**입니다. 이는 Rust의 패턴 매칭이 구조적으로 **튜플 내부 요소에 대한 부분적인 참조 해제(`&v`)를 금지**하기 때문입니다.

이 경우에는 `*` 연산자를 사용해야 합니다.


## 2. `*` 연산자를 이용한 명시적인 참조 해제

`*` 연산자는 명시적으로 참조를 해제하여 값을 가져올 때 사용됩니다. 하지만 이 연산자를 사용할 수 있으려면 `Deref` 트레이트가 구현되어 있어야 합니다.

### 왜 튜플 내부에서는 `*` 연산자가 허용될까?

튜플 자체는 `Deref` 트레이트를 구현하지 않지만, 개별 요소에 대해 `*` 연산자를 사용하여 직접 참조 해제를 수행할 수 있기 때문입니다. 즉, `|(_, v)| *v == max`에서는 `v`가 `&i32` 타입이므로 `*v`를 사용하여 `i32` 타입으로 변환할 수 있습니다.

### Deref 트레이트와 `*` 연산자

`*` 연산자는 내부적으로 `Deref` 트레이트의 `deref` 메서드를 호출하여 참조를 해제합니다. `Deref` 트레이트는 스마트 포인터에서 주로 사용되지만, 일반적인 참조(`&T` 타입)에도 자동으로 적용됩니다.

```rust
use std::ops::Deref;

struct MyBox<T>(T);

impl<T> Deref for MyBox<T> {
    type Target = T;
    fn deref(&self) -> &T {
        &self.0
    }
}

fn main() {
    let x = MyBox(42);
    println!("참조 해제된 값: {}", *x); // *x는 x.deref()를 호출한 후 참조 해제됨
}
```

위 코드에서 `*x`를 사용할 때 Rust는 `Deref` 트레이트의 `deref()` 메서드를 호출한 후 참조를 해제합니다.


## 3. 두 가지 방법 비교

### 언제 `|&x|`를 쓰는 게 더 좋은가?

- 단순한 타입(예: `&i32`, `&char`, `&bool`)을 처리할 때
- `iter()`를 사용하여 배열, 벡터, 슬라이스 등의 원소를 처리할 때
- 코드의 가독성을 높이고 싶을 때

### 언제 `|x| *x`를 쓰는 게 더 좋은가?

- **튜플 내부에서 참조를 해제해야 할 때** (`|(_, &v)|`가 안 되는 경우)
- **구조체처럼 복잡한 타입이 있는 경우**, 패턴 매칭으로 참조 해제를 하면 문법이 제한될 수 있음
- **스마트 포인터를 사용할 때**, `Deref` 트레이트를 활용하여 값을 가져와야 할 때


## 결론

Rust에서 참조를 해제하는 방법은 크게 두 가지로 나눌 수 있습니다.

1. **패턴 매칭 (`&` 패턴)**: `|&x|`처럼 사용하여 가독성을 높일 수 있으나, 튜플 내부 요소에서는 사용할 수 없음.
2. **`*` 연산자**: `Deref` 트레이트를 통해 동작하며 모든 경우에서 사용할 수 있지만, 패턴 매칭보다 코드가 길어질 수 있음.

각 방법의 특성을 이해하고 적절한 상황에서 사용하면 더욱 가독성 좋은 Rust 코드를 작성할 수 있습니다.


## 참고 문헌

- [The Rust Programming Language - References and Borrowing](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html)
- [Rust Reference - Pattern Matching](https://doc.rust-lang.org/reference/patterns.html)
- [Rust By Example - References and Borrowing](https://doc.rust-lang.org/rust-by-example/scope/borrow.html)
- [Rust Standard Library - Deref](https://doc.rust-lang.org/std/ops/trait.Deref.html)
