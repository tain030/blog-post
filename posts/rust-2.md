---
title: 'Rust: 유연한 타입 설계를 위한 추상화된 타입들'
date: '2025-02-09'
category: 'rust'
---

### 목차

1. 슬라이스 (`&[T]`)
2. `&str`과 `String`
3. `AsRef<T>` 트레이트
4. `Into<T>` 트레이트
5. `Cow<T>` (Clone-On-Write)

---

https://blog.kakaocdn.net/dn/rAIay/btszbtfDapI/MkIZTYFnJYr9JuOVm5smgK/img.jpg

Rust에서는 함수의 파라미터나 타입을 설계할 때, 다양한 데이터 구조에서 공통으로 사용할 수 있도록 추상화된 타입을 활용할 수 있습니다. 이렇게 하면 함수나 구조체가 특정 타입에 종속되지 않고 더 범용적으로 동작할 수 있습니다. 이번 글에서는 Rust에서 자주 사용되는 추상화된 타입과 그 활용법, 그리고 주의할 점을 설명하겠습니다.

## 1. 슬라이스 (`&[T]`)

`&[T]`는 여러 종류의 컬렉션을 받을 수 있도록 하는 추상화된 타입입니다. 이를 사용하면 `Vec<T>`, 배열 `[T; N]`, 다른 슬라이스 `&[T]`를 동일한 함수에서 처리할 수 있습니다.

```rust
fn sum(slice: &[i32]) -> i32 {
    slice.iter().sum()
}

fn main() {
    let array = [1, 2, 3];
    let vec = vec![4, 5, 6];

    println!("{}", sum(&array)); // 배열 지원
    println!("{}", sum(&vec));   // Vec 지원
}
```


## 2. `&str`과 `String`

Rust에서 문자열을 다룰 때 `&str`을 사용하는 것이 일반적입니다. `&str`은 `String`의 내부 데이터에 대한 참조를 제공하므로, `String`과 `&str`을 모두 지원하는 함수를 만들 수 있습니다.

```rust
fn greet(name: &str) {
    println!("Hello, {}!", name);
}

fn main() {
    let s = String::from("Alice");
    greet(&s);   // &String -> &str
    greet("Bob"); // &str 그대로 사용
}
```

### 주의사항
- `&str`은 읽기 전용 참조이므로, 소유권이 필요할 경우 `String`을 사용해야 합니다.


## 3. `AsRef<T>` 트레이트

`AsRef<T>` 트레이트를 사용하면 특정 타입을 참조 형태(`&T`)로 변환할 수 있습니다. 이를 활용하면 여러 타입을 지원하는 범용 함수를 만들 수 있습니다.

```rust
fn print_length<T: AsRef<str>>(s: T) {
    println!("Length: {}", s.as_ref().len());
}

fn main() {
    let s = String::from("Hello");
    print_length(s);   // String 지원
    print_length("World"); // &str 지원
}
```


## 4. `Into<T>` 트레이트

`Into<T>` 트레이트를 사용하면 특정 타입을 자동 변환할 수 있도록 설계할 수 있습니다. 이는 특히 제너릭 함수에서 유용합니다.

```rust
fn to_string<T: Into<String>>(s: T) -> String {
    s.into()
}

fn main() {
    let s1 = to_string("hello"); // &str -> String
    let s2 = to_string(String::from("world")); // String 그대로 사용

    println!("{} {}", s1, s2);
}
```


## 5. `Cow<T>` (Clone-On-Write)

`Cow<T>`는 `&T`(참조)와 `T`(소유권)을 동시에 다룰 수 있도록 설계된 타입입니다. 기본적으로 참조를 유지하지만 필요하면 소유권을 가진 값으로 변경할 수 있습니다.

```rust
use std::borrow::Cow;

fn process<'a>(input: Cow<'a, str>) {
    if input.contains("modify") {
        let mut owned = input.into_owned();
        owned.push_str(" (modified)");
        println!("{}", owned);
    } else {
        println!("{}", input);
    }
}

fn main() {
    process("hello".into()); // 참조 사용
    process(String::from("modify me").into()); // 소유권 사용
}
```

### 주의사항
- 변경이 필요한 경우 `.into_owned()`를 호출해야 하므로, 불필요한 소유권 이동을 방지해야 합니다.


## 결론
Rust에서는 `&[T]`, `&str`, `AsRef<T>`, `Into<T>`, `Cow<T>` 등 다양한 추상화된 타입을 제공하여 함수나 타입 설계를 더욱 유연하게 만들 수 있습니다. 이러한 추상화를 잘 활용하면 코드의 재사용성을 높이고, 불필요한 복사를 줄여 성능을 최적화할 수 있습니다.


## 참고 자료
- [Rust 공식 문서 - Slices](https://doc.rust-lang.org/std/primitive.slice.html)
- [Rust 공식 문서 - AsRef 트레이트](https://doc.rust-lang.org/std/convert/trait.AsRef.html)
- [Rust 공식 문서 - Into 트레이트](https://doc.rust-lang.org/std/convert/trait.Into.html)
- [Rust 공식 문서 - Cow](https://doc.rust-lang.org/std/borrow/enum.Cow.html)
