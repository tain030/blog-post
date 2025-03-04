---
title: 'Rust: Result 타입과 표준 라이브러리의 타입 별칭'
date: '2025-03-02'
category: 'rust'
---

### 목차

1. Result 타입의 기본 구조
2. `std::io::Result<T>`는 어떻게 하나의 타입만 가지는가?
3. 타입 별칭의 장점과 제한
    - 코드 가독성과 일관성 향상
    - 모듈별 맞춤형 Result 타입 가능
    - 주의사항: 타입 별칭은 새로운 타입이 아니다

---

https://blog.kakaocdn.net/dn/rAIay/btszbtfDapI/MkIZTYFnJYr9JuOVm5smgK/img.jpg

Rust에서 `Result<T, E>` 타입은 성공적인 결과(`Ok(T)`) 또는 오류(`Err(E)`)를 나타내는 열거형(enum)입니다. 하지만 Rust 표준 라이브러리에서는 특정 상황에서 더 간결한 코드 작성을 위해 `Result<T, E>`에 대해 **타입 별칭(type alias)** 을 제공합니다. 이 글에서는 `std::io::Result<T>`와 같은 별칭이 어떻게 하나의 타입만 가지는 `Result`로 표현될 수 있는지 자세히 살펴보겠습니다.

## Result 타입의 기본 구조

Rust의 `Result` 타입은 다음과 같이 정의되어 있습니다.

```rust
enum Result<T, E> {
    Ok(T),  // 성공적인 결과를 포함
    Err(E), // 오류 정보를 포함
}
```

일반적으로 `Result<T, E>`는 **T(성공 시 반환 값)와 E(에러 타입)** 두 가지 타입을 필요로 합니다. 그러나 표준 라이브러리에서는 특정 모듈에서 자주 사용되는 `Result` 타입을 **타입 별칭**을 이용해 간소화합니다.

## `std::io::Result<T>`는 어떻게 하나의 타입만 가지는가?

Rust 표준 라이브러리에서는 `std::io` 모듈에서 자주 사용되는 `Result<T, std::io::Error>`를 다음과 같이 별칭으로 정의합니다.

```rust
pub type Result<T> = std::result::Result<T, std::io::Error>;
```

이렇게 하면 `std::io::Result<T>`를 사용할 때마다 `std::result::Result<T, std::io::Error>`를 반복해서 작성할 필요가 없습니다. 예를 들어, 파일을 읽는 함수를 정의할 때:

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_file() -> io::Result<String> {
    let mut file = File::open("hello.txt")?;
    let mut content = String::new();
    file.read_to_string(&mut content)?;
    Ok(content)
}
```

위 코드에서 `io::Result<String>`은 실제로 `Result<String, io::Error>`와 동일한 타입입니다. 따라서 `?` 연산자를 사용할 때도 `Err(io::Error)`가 반환됩니다.

## 타입 별칭의 장점과 제한

### 1) 코드 가독성과 일관성 향상

- `std::io::Result<T>`를 사용하면 입출력 관련 코드에서 오류 처리를 명확하게 표현할 수 있습니다.
- 반복적인 `Result<T, std::io::Error>` 작성이 필요 없어 코드가 간결해집니다.

### 2) 모듈별 맞춤형 Result 타입 가능

Rust에서는 특정 도메인에 맞춘 별칭을 직접 정의할 수도 있습니다.

```rust
type DbResult<T> = Result<T, sqlx::Error>;

type AuthResult<T> = Result<T, auth::AuthError>;
```

이렇게 하면 데이터베이스 작업과 인증 작업에서 각기 다른 `Result` 타입을 명확하게 구별할 수 있습니다.

### 3) 주의사항: 타입 별칭은 새로운 타입이 아니다

타입 별칭은 단순히 기존 타입의 다른 이름이므로 **완전히 새로운 타입을 정의하는 것은 아닙니다**. 따라서 `std::io::Result<T>`는 여전히 `Result<T, std::io::Error>`로 동작합니다.

```rust
fn test(r: std::io::Result<()>) {}
fn test2(r: Result<(), std::io::Error>) {}

// 두 함수는 동일한 타입을 받음
test2(Ok(()));
test(Ok(()));
```

이처럼 타입 별칭은 문법적으로는 다른 이름이지만, 실제로는 같은 타입으로 취급됩니다.

## 결론

- Rust의 `Result<T, E>`는 두 개의 타입 매개변수를 가지지만, **타입 별칭(type alias)**을 사용하면 특정 에러 타입을 고정할 수 있습니다.
- `std::io::Result<T>`는 `Result<T, io::Error>`의 별칭이며, 입출력 관련 코드에서 가독성을 높이는 데 도움을 줍니다.
- 필요에 따라 **사용자 정의 타입 별칭**을 만들어 도메인별 `Result`를 정의할 수 있습니다.
- 하지만 타입 별칭은 새로운 타입을 생성하는 것이 아니므로, 여전히 원래의 `Result<T, E>` 타입으로 동작합니다.

## 출처

- [Rust 공식 문서 - Result](https://doc.rust-lang.org/std/result/)
- [Rust 공식 문서 - Error Handling](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html)
- [Rust By Example - Result](https://doc.rust-lang.org/rust-by-example/error/result.html)
- [The Rust Programming Language (The Book) - Error Handling](https://doc.rust-lang.org/book/ch09-00-error-handling.html)
